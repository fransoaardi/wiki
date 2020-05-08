# intro
- `concurrency in go` 책을 읽고, 필요한(앞으로 참고해볼만한) concurrency pattern 을 정리했음.
- 아래 코드의 출처는 철저히 `concurrency in go` 책에서 참고함

# references
- https://github.com/kat-co/concurrency-in-go-src

# patterns
## repeater (159p)
- 받은 value 를 계속 반복하는 channel 

```go
repeat := func( done <-chan interface{}, values ...interface{}) <-chan interface{} {
    valueStream := make(chan interface{})
    go func() {
        defer close(valueStream)
        for {
            for _, v := range values{
                select {
                case <-done:
                    return
                case valueStream <- v:
                }
            }
        }
    }()
    return valueStream
}
```

## take (160p)
- given channel 에서 n 개를 취하는 channel return 

```go
take := func(done <-chan interface{}, valueStream <-chan interface{}, num int) <-chan interface{} {
    takeStream := make(chan interface{})
    go func() {
        defer close(takeStream)
        for i:=0; i<num; i++ {
            select {
            case <-done:
                return
            case takeStream <- <- valueStream:
            }
        }
    }()
    return takeStream
}
```

## fan-in, fan-out 채널 (169p)

```go
fanIn := func( done <- chan interface{}, channels ...<-chan interface{} ) <-chan interface{} {
    var wg sync.WaitGroup
    multiplexedStream := make(chan interface{})

    multiplex := func(c <- chan interface{}){
        defer wg.Done()
        for i := range c{
            select {
            case <- done:
                return 
            case multiplexedStream <- i:
            }
        }       
    }

    wg.Add(len(channels))

    for _, c:= range channels{
        go multiplex(c)
    }

    go func(){
        wg.Wait()
        close(multiplexedStream)
    }()

    return multiplexedStream
}
```

## or-done 채널 (172p)

```go
orDone := func(done, c <- chan interface{}) <-chan interface{} {
    valStream := make(chan interface{})
    go func() {
        defer close(valStream)
        for {
            select {
            case <-done:
                return
            case v, ok := <-c:
                if ok == false{
                    return
                }
                select {
                case valStream <- v:
                case <- done:
                }
            }
        }
    }()
    return valStream
}
```

## tee 채널 
- 읽어들일 채널을 전달할 수 있으며, 동일한 값을 얻어오는 별개의 채널 두 개를 리턴한다.

```go
tee := func(done <-chan interface{}, in <-chan interface{}) ( _, _ <-chan interface{}){
    out1 := make(chan interface{})
    out2 := make(chan interface{})
    go func(){
        defer close(out1)
        defer close(out2)
        for val := range orDone(done, in){
            var out1, out2 = out1, out2
            for i := 0; i < 2; i++ {
                select {
                case <-done:
                case out1<-val:
                    out1 = nil
                case out2<-val:
                    out2 = nil
                }
            }
        }
    }()
    return out1, out2
}
```

## bridge 채널

```go
bridge := func(done <-chan interface{}, chanStream <-chan <-chan interface{}) <-chan interface{
    valStream := make(chan interface{})
    go func(){
        defer close(valStream)
        for {
            var stream <-chan interface{}
            select {
            case maybeStream, ok := <-chanStream:
                if ok == false{
                    return
                }
                stream = maybeStream
            case <-done:
                return
            }
            for val := range orDone(done, stream) {
                select {
                case valStream <- val:
                case <-done:
                }
            }
        }
    }()

    return valStream
}
```

## rate limiter 
- golang 내부 rate package 를 이용해서 ratelimiter 구현 
- 사용량 (1초에 1번 등..) 제어 
> https://github.com/kat-co/concurrency-in-go-src/blob/master/concurrency-at-scale/rate-limiting/fig-multi-rate-limit.go

## heartbeat 
- goroutine 실행시, goroutine 의 상태를 주기적으로 확인할 수 있는 heartbeat 채널(time.Ticker)을 채널과 함께 return 해서, 확인하는데 이용함.
> https://github.com/kat-co/concurrency-in-go-src/blob/master/concurrency-at-scale/heartbeats/concurrent_test.go
