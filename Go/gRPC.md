# gRPC

내가 먼저 접한 것은 Go가 아닌 gRPC이다. 2002년도 리눅스코리아 소속 당시 XML-RPC기술을 사용해서 프로젝트를 진행하면서 RPC의 매력에 빠졌다고 보는 것이 맞을 것이다.

XML를 통해 멀리 떨어져 있는 컴퓨터의 Function을 마음대로 콜을 할수 있다는 사실이 너무 충격적이였다. 이때의 충격은 C을 처음 접하고 Function이라는 힘을 느낀 것 처럼 너무 획기적인 것이였다.

중간에 Json형식의 RPC도 있었지만 구조는 XML-RPC와 대동소이함으로 패스...!!!

아무튼 gPRC는 프로토콜, 구조부터 달랐고 양방향이라는 메리트는 마치 신문물처럼 느껴졌다.

구구절절한 gRPC 이론얘기는 인터넷을 검색하면 쉽게 확인할수 있으니 이곳에 기술하지는 않겠다.

1. [Proto](#Proto)

1. [Server](#Server)

1. [Client](#Client)

### Proto

```proto
syntax = "proto3";

// --go_out에 지정된 디렉토리 밑에 디렉토리가 생성된다.
option go_package = "config/proto";

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
```

일부러 Parameter가 없는 Sample을 사용했다. 인터넷을 찾아보면 Parameter가 있는 Sample들이 대부분이므로 비교하면서 보면 도움이 될 것이다.

```bash
$ go get -u google.golang.org/grpc
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

gRPC를 사용하기 위한 모듈 설치. 모듈이 설치되어 있다면 생략해도 된다.

```bash
$ protoc -I config config.proto --go_out=plugins=grpc:.
```

protobuf를 사용해서 go파일을 생성한다.

`-I config config.proto` : proto파일이 위치한 디렉토리와 proto파일을 지정해 준다

`--go_out=plugins=grpc:.` : Output위치 지정해 준다. (현재 디렉토리에 'option go_package = "config/proto";'의 영향으로 config/proto디렉토리가 생성되고 여기에 config.pb.go가 위치한다.)

### Server

```go
ppackage main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"log"
	"net"
	"tagmemo.com/snoopy_kr/config/proto"
	"time"
)

type server struct {
	// UnimplementedConfigureServer 당황스럽겠지만 안심해도 된다 gRPC를 사용하면 자주보게 될 단어다.
	proto.UnimplementedConfigureServer
}

// 서버가 클라이언트에게 제공할 RPC의 함수다.
func (s *server) SetConfigure(ctx context.Context, in *proto.ConfigRequest) (*proto.ConfigResponse, error) {
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
		return &proto.ConfigResponse{}, nil
	}
}

func main() {
	lis, err := net.Listen("tcp", ":8088")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// 하는 일에 비해 소스는 너무 단순하다...
	s := grpc.NewServer()
	proto.RegisterConfigureServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

Sample내에 Comment처리를 했기 때문에 Sample를 파악하는 것은 어렵지 않을 것이다.

### Client

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"log"
	"tagmemo.com/snoopy_kr/config/proto"
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

	// 설정...!!!
	c := proto.NewConfigureClient(conn)

	// 5초 타임아웃 설정...!!!
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()

	// 서버에서 처럼 호출하는 방식도 단순하다...
	r, err := c.SetConfigure(ctx, &proto.ConfigRequest{})

	// 예외사항 발생 처리...
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
```

Client 관련 Sample도 Comment처리 했으니 참조해서 파악하기 바란다.

Sample을 테스트 하려면 먼저 Server를 구동하고 Client를 실행하고 바로 Client를 죽이게 되면 Server에서 Exception이 발생된다.

그리고 Server를 'case <-time.After(10 * time.Second):'와 같이 Response 타임을 10후로 수정하면 Client에서 Timeout Exception이 발생되는 것을 확인할수 있다.
