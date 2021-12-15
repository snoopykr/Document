# context

context는 go에서 사용되는 패키지의 이름이다. context를 인터넷에서 찾아보면 우리나라말로 쓰여 있어도 뭔 말인지 도대체 이해가 가지 않는다. (본인의 국어 실력이 낮아서...)

그냥 간단하게 정리하면 context는 프로세스, API 또는 RPC처럼 뭔가를 호출하고 답을 기다려야 할때 사용되며 예외사항이 발생되는 경우 관련 처리를 원활하게 해준다.

Browser를 통해 Server를 호출 했는데 Server가 너무 바빠서 제대로 된 처리를 시간내에 request를 처리하지 못하는 경우(t`imeout`, `deadline`) 또는 Browser가 다운되서 더이상 response할 필요가 없는 경우(`Cancel`) 등 예외사항이 발생된 경우를 처리하기 위해 필요한 패키지이다.

참고로 contest에는 위에 말한 Cancel, timeout, deadline외에 value를 처리하는 부분도 포함이 되어 있다.

1. [Cancel 처리](#cancel-처리)

1. [Timeout, Deadline 처리](#timeout-deadline-처리)

1. [Value](#Value)

## Cancel 처리

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	// 예외사항을 위한 준비...
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	// ctx 생성...
	ctx, cancel := context.WithCancel(context.Background())

	// 예외사항 체크...
	go func() {
		fmt.Println("Signal:", <-exit)
		cancel()
	}()

	// 시작...
	start := time.Now()
	result, err := longFuncWithCtx(ctx)
	if err != nil {
		log.Fatal(err)
	}

	// 예외사항이 발생되면 go루틴에 위해 cancel()이 실행되기 때문에 이 부분은 출력이 되지 않는다.
	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)

	// 출력
	// 정상 종료...!!!
	// duration:5.0135392s result:Success

	// 예외사항
	// Signal: interrupt
	// 예외사항 발생...!!!
	// 2021/11/29 11:45:51 context canceled
}

func longFuncWithCtx(ctx context.Context) (string, error) {
	done := make(chan string)

	// 정상 종료 체크...
	go func() {
		done <- longFunc()
	}()

	select {
	// 예외사항 처리...
	case <-ctx.Done():
		fmt.Println("예외사항 발생...!!!")
		return "Fail", ctx.Err()
	// 정상 종료...
	case result := <-done:
		fmt.Println("정상 종료...!!!")
		return result, nil
	}
}

func longFunc() string {
	// 이렇게 사용해도 문제가 없다.
	// time.Sleep(time.Second * 5)

	<-time.After(time.Second * 5)
	return "Success"
}
```
이 Sample은 실무에서 쓰기보다는 이해를 돕기위해 준비한 Source이다. 단일 프로세스지만 예외사항을 체크하고 어떻게 처리가 되는지 확인하기에는 안성맞춤 Sample이다.

## Timeout, Deadline 처리

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"
)

const maxDuration = time.Second * 2

func main() {
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	// 타임아웃 1초를 지정해준다.
	ctx, cancel := context.WithTimeout(context.Background(), maxDuration)

	go func() {
		fmt.Println("Signal:", <-exit)
		cancel()
	}()

	// 결과보다 먼저 타웃아웃이 발생되기 때문에 예외처리를 하면 된다.
	start := time.Now()
	result, err := longFuncWithCtx(ctx)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)

	// 출력
	// 예외사항 발생...!!!
	// 2021/11/29 13:35:09 context deadline exceeded
}

func longFuncWithCtx(ctx context.Context) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-ctx.Done():
		fmt.Println("예외사항 발생...!!!")
		return "Fail", ctx.Err()
	case result := <-done:
		fmt.Println("정상 종료...!!!")

		return result, nil
	}
}
func longFunc() string {
	// 3초후에 Success를 전달하지만 이미 타임아웃이 된 상태가 된다.
	<-time.After(time.Second * 3)
	return "Success"
}
```
이 Sample에서는 3초의 딜레이 동안 Timeout를 발생시켜 예외사항을 만드는 Source이다. 1초만에 Timeout이 걸리게 되어 모든 처리가 완료 되기 이전에 예외사항이 발생하는 것이다.

Deadline도 Timeout과 거의 동일하기 때문에 Sample은 생략하겠다...

단순히 ctx, cancel := context.WithTimeout(context.Background(), maxDuration)에서 WithDeadline으로 변경하고 Now() + timeout시간으로 변경하면 된다.

즉 Timeout은 after개념이고 Deadline은 when개념이다.

## Value

```go
package main

import (
	"context"
	"errors"
	"fmt"
)

type User struct {
	Name string
}

func main() {
	currentUser := User{Name: "snoopy_kr"}

	// 컨텍스트 생성
	ctx := context.Background()

	// 컨텍스트에 값 추가 - context.WithValue 함수를 사용하여 값을 전달.
	ctx = context.WithValue(ctx, "current_user", currentUser)

	// 시작.
	myFunc(ctx)

}

func myFunc(ctx context.Context) error {
	var currentUser User

	// 컨텍스트에서 'current_user'값을 추출...
	if v := ctx.Value("current_user"); v != nil {

		// ctx.Value로는 interface를 리턴 받게 된다.
		// User struct로 cascading...!!!
		u, ok := v.(User)
		if !ok {
			return errors.New("Not authorized")
		}
		currentUser = u
	} else {
		return errors.New("Not authorized")
	}
	
	fmt.Println(currentUser)

	// 출력
	// {snoopy_kr}

	return nil
}
```
context에 값을 지정해서 전달하는 Sample이다. 다른 Source보다 복잡하지 않아서 쉽게 이해될 것이다.

myFunc에서 Interface를 사용하고 Cascading해 주었는데... 관련 해서는 Interface를 참조하면 Source를 좀더 쉽게 이해할수 있을 것이다.