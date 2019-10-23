# 디스커버리 Go 언어: 염재현 지음

> 코드 한줄 한줄이 대단했다.

- 소스 위치 : https://github.com/jaeyeom/gogo


```go
//97p 
func hasDupeRune(s string) bool{
    runeSet := map[rune]struct{}{}
    for _, r := range s {
        if _, exists := runeSet[r]; exists{
            return true
        }
        runeSet[r] = struct{}{} // 빈 구조체를 사용해서 불필요한 bool 값에 의한 메모리 차지를 방지
    }
    return false
}
``` 

```go
//249p ~ 276p
func Example_simpleChannel() {
    c:= func() <- chan int {
            go func() {
                defer close(c)
                c <- 1
                c <- 2
                c <- 3
            }()
        return c
    }()
    for num := range c {
        fmt.Println(num)
    }
    // Output:
    // 1
    // 2
    // 3
}
```

- close 는 defer 를 이용하는것이 깔끔합니다.
- 단방향 채널을 반환하여 이 채널을 이용하는 고루틴이 받아가기만 할 수 있게 제한하였습니다. 그렇지 않고 받아가야 하는 곳에서 이 채널에 값을 보내려고 시도하면 그 자료를 받아가는 고루틴이 없어서 영원히 프로그램이 멈추어 있을 수도 있기 때문입니다.

- 채널을 이용하는 방법에는 몇 가지 장점이 있습니다.

1) 생성하는 쪽에서는 상태 저장 방법을 복잡하게 고민할 필요가 없다.
2) 받는 쪽에서는 for의 range를 이용할 수 있다.
3) 채널 버퍼를 이용하면 멀티 코어를 활용하거나 입출력 성능상의 장점을 이용할 수 있다.
 
- 버퍼 없는 채널로 동작하는 코드를 만들고 필요에 따라 성능 향상을 위하여 버퍼 값을 조절해주는 것이 좋습니다.

 

## pipelining pattern

```go
PlusTwo := Chain(PlusOne, PlusOne) 
```
과 같이 만든다.

 
## FanIn, FanOut 

```go
package concurrency
import (
    "sync"
    "golang.org/x/net/context"
)
 
// PlusOne returns a channel of num + 1 for nums received from in.
func PlusOne(ctx context.Context, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for num := range in {
            select {
            case out <- num + 1:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}
 
type IntPipe func(context.Context, <-chan int) <-chan int
 
func Chain(ps ...IntPipe) IntPipe {
 
    return func(ctx context.Context, in <-chan int) <-chan int {
        c := in
        for _, p := range ps {
            c = p(ctx, c)
        }
        return c
    }
}
 
var PlusTwo = Chain(PlusOne, PlusOne)
 
func FanIn(ins ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    wg.Add(len(ins))
    for _, in := range ins {
        go func(in <-chan int) {
            defer wg.Done()
            for num := range in {
                out <- num
            }
        }(in)
    }
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
 
func Distribute(p IntPipe, n int) IntPipe {
    return func(ctx context.Context, in <-chan int) <-chan int {
        cs := make([]<-chan int, n)
        for i := 0; i < n; i++ {
            cs[i] = p(ctx, in)
        }
        return FanIn(cs...)
    }
}
 
func FanIn3(in1, in2, in3 <-chan int) <-chan int {
    out := make(chan int)
    openCnt := 3
    closeChan := func(c *<-chan int) bool {      // 채널을 nil로 만든다. nil 채널에는 보내기 및 받기가 모두
                                                // blocking 된다.
        *c = nil
        openCnt--
        return openCnt == 0
    }
    go func() {
        defer close(out)
        for {
            select {
            case n, ok := <-in1:
                if ok {
                    out <- n
                } else if closeChan(&in1) {
                    return
                }
            case n, ok := <-in2:
                if ok {
                    out <- n
                } else if closeChan(&in2) {
                    return
                }
            case n, ok := <-in3:
                if ok {
                    out <- n
                } else if closeChan(&in3) {
                    return
                }
            }
        }
    }()
    return out
}
```
 
- 모든 자료를 소진시키지 않으면 해제되지 않은 고루틴들이 메모리에 남아 메모리 누수를 일으킨다.

- 그렇다고 채널을 억지로 닫을 수도 없다. 채널닫기는 언제나 보내는 쪽에서만 하는것으로 패턴을 정형화 하는것이 좋다.

- done 채널을 하나 더 두고, close(done)으로 신호를 주고, 이 채널에서 신호가 감지되면 보내는것을 중단하고 채널을 닫는다.

 

## 채널사용시 주의점 (277p)

```go
// WARNING! This is a bad example.
c := make(chan int)
done := make(chan bool)
go func() {
    for i := 0; i < 10; i++ {
        c <- i
    }
    done <- true
}()
go func() {
    for {
        fmt.Println(<-c)
    }
}()
<- done
```
- 첫 번째 고루틴은 생산자, 두 번째 고루틴은 소비자로 이루어진 코드.

- 두 번째 고루틴은 끝나지 않는다.

- 첫번재 고루틴에서 생산이 끝난 뒤 done 채널에 true 값을 넣는데, 소비가 끝나기 전에 ←done 으로 메인 goroutine 이 끝나버릴 가능성이 있다.

1) 자료를 보내는 채널은 보내는 쪽에서 닫는다.
2) 보내는 쪽에서 반복문 등을 활용해서 보내다가 중간에 return을 할 수 있으므로 닫을 때는 defer 를 이용하는 거이 좋다. 그렇지 않으면 중간에 return했을때  채널을 닫지 않고 종료할 수 있다.
3) 받는 쪽이 끝날때까지 기다리는 것이 모든 자료의 처리가 끝나는 시점까지 기다리는 방법으로 더 안정적이다. 위의 예제에서 생산자가 아닌 소비자 쪽에서 done ← true 를 했어야 한다. 물론 위의 예제에서는 소비자 쪽에서 언제 끝났는지 알 수 없다. 그것을 생산자에서 채널을 닫는 것으로 신호를 줬어야 했다.
4) 특별한 이유가 없다면 받는 쪽에서 range를 이용하는 것이 좋다. 생산자가 채널을 닫은 경우에 반복문을 빠져나오기 때문이다.
5) 루틴이 끝났음을 알리고 다른쪽에서 기다리는 것은 sync.WaitGroup을 이용하는 것이 나은 경우가 많다.
6) 끝났음을 알리는 done 채널은 자료를 보내는 쪽에서 결정할 사항이 아니다. 자료를 보내는 쪽에서는 채널을 닫아서 자료가 끝났음을 알리는 것이 더 낫다. 그러면 done 채널은 받는 쪽에서 보내는 쪽에서 자료 전송이 끝났거나 끝나지 않았으면 더 이상 자료를 보내지 말아달라는 cancel 요청으로 보는 것이 더 낫다.
7) done 채널에 자료를 보내어 신호를 주는 예제가 있는데, close(done)으로 채널을 닫는 것이 더 나은 방법인 경우가 많다.
 

## 281p  atomic과 sync.WaitGroup
```go
for cnt > 0 {
    time.Sleep(100 * time.Millisecond)
}
를 atomic을 이용하여 아래와 같이 변경하면 go race 문제를 해결 할 수 있다.
  
for atomic.LoadInt64(&cnt) > 0 {
    time.Sleep(100 * time.Millisecond)
}
```

## 284p 간단한 concurrent map 구현
```go
package conmap
import "sync"
type Resource int
type Accessor struct {  // Accesor 라는 struct 를 만들고 하나는 Resource(map[string]interface{})
                        // 하나는 sync.Mutex를 갖고 있으면 concurrent한 객체 생성 가능
    R *Resource
    L *sync.Mutex
}
 
func (acc *Accessor) Use() {
    // do something
    acc.L.Lock()
    // Use acc.R
    acc.L.Unlock()
    // Do something else
}
 
type ConcurrentMap struct {
    M map[string]string
    L *sync.RWMutex
}
 
func (m ConcurrentMap) Get(key string) string {
    m.L.RLock()
    defer m.L.RUnlock()
    return m.M[key]
}
 
func (m ConcurrentMap) Set(key, value string) {
    m.L.Lock()
    m.M[key] = value
    m.L.Unlock()
}
```
