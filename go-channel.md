- close closed channel : panic 
```go
ch := make(chan struct{})
close(ch)
close(ch)  // panic: close of closed channel
```

- send to closed channel : panic
```go
ch := make(chan struct{})
close(ch)
ch <- struct{}{} // panic: send on closed channel
```

- select 의 case 는 순서를 보장하지 않는다.
아래 `ctx1.Done()` 을 `ctx2.Done()` 보다 listening 을 먼저 하고 먼저 응답을 받을 것 같지만, 실행 해보면 값이 자주 바뀐다. 
```golang
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx1, cancel1 := context.WithCancel(context.Background())
	ctx2, cancel2 := context.WithCancel(context.Background())

	cancel1()
	cancel2()

	time.Sleep(900 * time.Millisecond)

	select {
	case <-ctx1.Done():
		fmt.Println("ctx 1 done")
	case <-ctx2.Done():
		fmt.Println("ctx 2 done")
	}
}
```
