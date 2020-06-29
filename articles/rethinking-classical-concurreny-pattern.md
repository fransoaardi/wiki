- Reference
    - Rethinking classical concurrency pattern 
        https://drive.google.com/file/d/1nPdvhB0PutEJzdCq5ms6UI58dp50fcAN/view

- future pattern
```go
func Fetch(name string) <-chan Item {
    c := make(chan Item, 1)
    go func() {
        [...]
        c <- item
    }()
    return c
}
```
- buffered (length: 1), 나중에 꺼내쓰는 asynchronous API

```go
a := Fetch("a")
b := Fetch("b")
fmt.Println(<-a, <-b)
// fmt.Println(a,b)  // Don't!
```

- producer-consumer queue pattern 
```go
// Glob finds all items with names matching pattern
// and sends them on the returned channel.
// It closes the channel when all items have been sent.
func Glob(pattern string) <-chan Item {
    c := make(chan Item)
    go func() {
        defer close(c)
        for [...] {
            [...]
            c <- item
        }
    }()
    return c
}
```
- typically unbuffered(!)
- 아래와 같이, range 로 호출함
```go
for item := range Glob("[ab]*") {
    [...]
}
```

- A producer–consumer queue also returns a channel, but the channel receives any number of results and is typically unbuffered.

- ** `Caller side concurrency` 를 구현 하라
- API 는 synchronous 하게 구현하고, 호출하는사람이 concurrent 하게
```go
var a, b Item
g, ctx := errgroup.WithContext(ctx)
g.Go(func() (err error) {
    a, err = Fetch(ctx, "a")
    return err
})
g.Go(func() (err error) {
    b, err = Fetch(ctx, "b")
    return err
})
err := g.Wait()
[...]
consume(a, b)
```

```go
func Glob([...]) ([]Item, error) {
    [...] // Find matching names.
    c := make(chan Item)
    g, ctx := errgroup.WithContext(ctx)
    for _, name := range names {
        name := name
        g.Go(func() error {
            item, err := Fetch(ctx, name)
            if err == nil {
                c <- item
            }
            return err
        })
    }

    go func() {
        err = g.Wait()
        close(c)
    }()
    var items []Item
    for item := range c {
        items = append(items, item)
    }
    if err != nil {
        return nil, err
    }
    return items, nil
}
```
- Keeping the channel local to the caller function makes its usage much easier to see.


- no leaked go worker pool, using semaphore
```go
sem := make(chan token, limit)
for _, task := range hugeSlice {
    sem <- token{}
    go func(task Task) {
        perform(task)
        <-sem
    }(task)
}
for n := limit; n > 0; n-- {
    sem <- token{}
}
```

- tranditional go worker pool
```go
work := make(chan Task)
for n := limit; n > 0; n-- {
    go func() {
        for task := range work {
            perform(task)
        }
    }()
}
for _, task := range hugeSlice {
    work <- task
}
```
