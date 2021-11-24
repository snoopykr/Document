gRPC 활용
-----

Server

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
	config.UnimplementedConfigureServer
}

func (s *server) SetConfigure(ctx context.Context, in *config.ConfigRequest) (*config.ConfigResponse, error) {
	log.Printf("Received profile")

	// 이곳에서 클라이언트의 취소가 발생하는지를 체크...
	canceled := make(chan string, 1)
	go func() {
		for {
			if ctx.Err() == context.Canceled {
				canceled <- "Client canceled...!!!"
			}
			time.Sleep(time.Millisecond)
		}
	}()

	select {

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

	s := grpc.NewServer()
	config.RegisterConfigureServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
</code>
</pre>

Client

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
	// grpc.WithBlock()이 있으면 데드라인이 활성화 되지 않는다.
	// conn, err := grpc.Dial("localhost:8088", grpc.WithInsecure(), grpc.WithBlock())

	conn, err := grpc.Dial("localhost:8088", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := config.NewConfigureClient(conn)

	// 타임아웃 설정...!!!
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

Proto

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