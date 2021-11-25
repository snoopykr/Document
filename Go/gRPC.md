gRPC 활용
-----

내가 처음 접한 것은 Go가 아닌 gRPC이다. 2002년도 리눅스코리아 소속 당시 XML-RPC기술을 사용해서 프로젝트를 진행하면서 RPC의 매력에 빠졌다고 보는 것이 맞을 것이다.

XML로 정해진 형식만 맞추면 멀리 떨어져 있는 컴퓨터에 평션을 마음대로 콜을 할수 있다는 사실이 너무 충격적이였다. 이때의 충격은 C을 처음 접하고 function이라는 힘을 느낀 것 처럼 너무 획기적인 것이였다.

중간에 Json형식의 RPC도 있었지만 내부 구조는 XML-RPC와 동일함으로 패스...!!!

아무튼 gPRC는 프로토콜, 내부 구조부터 달랐고 양방향이라는 메리트는 마치 신문물을 접하는 것 같았다.

이런 이론적인 부분은 인터넷이 많이 있으니 이곳에 기술하지는 않겠다.

[ Proto ]

<pre>
<code>
syntax = "proto3";

option go_package = "tagmemo.com/snoopy_kr/config";

package config;

// 설정 조회 서비스
service Configure {
  rpc SetConfigure (ConfigRequest) returns (ConfigResponse) {}
}

// 요청
message ConfigRequest {
}

// 응답
message ConfigResponse {
}
</code>
</pre>

일부러 Parameter가 없는 Sample을 사용했다. 인터넷을 찾아보면 Parameter가 있는 Sample들이 대부분이므로 비교하면서 보면 도움이 될 것이다.

[ Server ]

<pre>
<code>
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"log"
	"net"
	"tagmemo.com/snoopy_kr/config"
	"time"
)

type server struct {
	// UnimplementedConfigureServer 당황스럽겠지만 안심해도 된다 gRPC를 사용하면 자주보게 될 단어이다.
	config.UnimplementedConfigureServer
}

// 서버가 클라이언트에게 제공할 RPC의 함수이다.
func (s *server) SetConfigure(ctx context.Context, in *config.ConfigRequest) (*config.ConfigResponse, error) {
	log.Printf("Received profile")

	// 예외사항이 발생되지 않는 다면 SetConfigure()함수는 return &config.ConfigResponse{}, nil 이렇게 한줄로 마무리 된다.
	// 하지만 중간에 발생될 예외 사항을 처리하기 context라는 것을 사용한다.
	// context에 대해서는 앞으로 만들 context.md를 확인하기 바란다.
	canceled := make(chan string, 1)
	go func() {
		for {
			if ctx.Err() == context.Canceled {
				canceled <- "Client canceled...!!!"
			}
			time.Sleep(time.Millisecond)
		}
	}()

	// go에서는 다른 언어들 처럼 switch도 있지만 select라는 것도 지원한다.
	// 단순히 select는 멀티프로세스를 위해 만들어 진 것이고 채널을 제어하기 위해 사용한다.
	select {
	// 클라이언트가 중간에 예외사항을 발생시키면 반은하는 부분이다.
	case <-canceled:
		fmt.Println("Client canceled the request...!!!")
		return nil, status.Error(codes.Canceled, "Client canceled the request")
	// 클라이언트의 취소를 이끌어 내기 위해 3초를 활당했다.
	case <-time.After(3 * time.Second):
		fmt.Println("Server response the gRPC call...!!!")
		return &config.ConfigResponse{}, nil
	}
}

func main() {
	lis, err := net.Listen("tcp", ":8088")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// 하는 일에 비해 소스는 너무 단순하다...
	s := grpc.NewServer()
	config.RegisterConfigureServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
</code>
</pre>

Sample내에 Comment처리를 했기 때문에 Sample를 파악하는 것은 어렵지 않을 것이다.

[ Client ]

<pre>
<code>
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"log"
	"tagmemo.com/snoopy_kr/config"
	"time"
)

func main() {
	// grpc.WithBlock()이 있으면 타임아웃이 활성화 되지 않는다.
	// grpc.WithBlock()를 사용하게 되면 단일프로세스로 처리가 되기 때문에 프로세스 관리가 되지 않는다.
	// conn, err := grpc.Dial("localhost:8088", grpc.WithInsecure(), grpc.WithBlock())

	conn, err := grpc.Dial("localhost:8088", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	// 서버에서 처럼 호출하는 방식도 단순하다...
	c := config.NewConfigureClient(conn)

	// 5초 타임아웃 설정...!!!
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()

	// 예외사항 발생 처리...
	r, err := c.SetConfigure(ctx, &config.ConfigRequest{})
	if err != nil {
		sError, ok := status.FromError(err)
		if ok {
			code := sError.Code()
			rpcErr := sError.Err()

			if code == codes.DeadlineExceeded {
				fmt.Println("Deadline Over...!!!", rpcErr)
			} else if code == codes.Canceled {
				fmt.Println("Client Cancel...!!!", rpcErr)
			} else {
				fmt.Println("unexpected gRPC Error", rpcErr)
			}
		} else {
			fmt.Println("Error while calling gRPC", err)
		}
		return
	}

	log.Printf("Config: %v", r)
}
</code>
</pre>

Client 관련 Sample도 Comment처리 했으니 참조해서 파악하기 바란다.
