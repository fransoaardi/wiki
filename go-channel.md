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
