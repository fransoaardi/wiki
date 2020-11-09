# connection-reuse 
## introduction
- golang server 의 file descriptor 고갈이 일어나서, request 를 못받고 hang 이 발생한 경우가 있어, connection 관련 실험을 해보기로 했다.
- http2, gRPC 인 경우는 다르게 동작하겠지만 일단은 http1.1 을 기준으로 테스트함

## environment
- osx 
- client: local, server: local
- client: go, server: go
- http1.1

## process
- `watch -n1 'netstat -an | grep "9000"`

## variable
### server 
- server keep alive
```golang
srv.SetKeepAlivesEnabled(false)
```

- server code:
```golang
func main() {
	mux := http.DefaultServeMux
	mux.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		_, _ = w.Write([]byte("hello world"))
		time.Sleep(time.Second)
	})
	srv := http.Server{
		Addr:    ":9000",
		Handler: mux,
	}

	//srv.SetKeepAlivesEnabled(false)

	srv.ListenAndServe()
}

```

### client
- `op1`: client 호출시마다 매번 생성
    - no 0
    - yes 1
- `op2`: `resp.Body` read 
    - none[읽지않음]    0
    - partial[일부 read] 1
    - all[전체 read (ioutil.ReadAll)] 2
- `op3`: `resp.Body.Close()` 여부 
    - not close 0
    - close 1

- client code:
```golang
func main() {
	op1, op2, op3 := os.Args[1], os.Args[2], os.Args[3]
	var c http.Client
	req, _ := http.NewRequest(http.MethodGet, "http://localhost:9000/hello", nil)

	num := 200
	for i := 0; i < num; i++ {
		if op1 == "1" {
			c = http.Client{}
		}

		resp, err := c.Do(req)
		if err != nil {
			continue
		}

		switch op2 {
		case "0": // don't read resp.Body, leave

		case "1": // read partially (5 out of 11 bytes)
			b := make([]byte, 5) // 다 못읽는 경우
			fmt.Println(resp.Body.Read(b))
		case "2": // read all using (ioutil.ReadAll)
			_, _ = ioutil.ReadAll(resp.Body) // 다 읽는 경우
		}

		if op3 == "1" {
			_ = resp.Body.Close()
		}
	}
}
```


## result
### server keep alive on
```
- 000   est 늘어남
- 001   est 안늘고, time_wait 로 변함
- 010   est 늘어남
- 011   est 안늘고, time_wait 로 변함
- 020   est/time_wait 안늘어남
- 021   est/time_wait 안늘어남
- 100   est 늘어남
- 101   est 안늘고, time_wait 늘어남
- 110   est 늘어남
- 111   est 안늘고, time_wait 늘어남
- 120   est/time_wait 안늘어남 
- 121   est/time_wait 안늘어남
```

### server keep alive off
```
- 000  est 늘고 fin, close_wait 하는데 time_wait 로 못변함
- 001  est 안늘고, time_wait 로 변함
- 010  est 늘고 fin, close_wait 하는데 time_wait 로 못변함
- 011  est 안늘고, time_wait 로 변함
- 020  est 안늘고, time_wait 로 변함
- 021  est 안늘고, time_wait 로 변함 
- 100  est 늘고, fin, close_wait 하는데 time_wait 못변함
- 101  est 안늘고, time_wait 로 변함  
- 110  est 늘고, fin, close_wait 하는데 time_wait 못변함 
- 111  est 안늘고, time_wait 로 변함   
- 120  est 안늘고, time_wait 로 변함    
- 121  est 안늘고, time_wait 로 변함    
```

## lessons learned
- 요청시 establish 하고, 최대한 재활용하고, 사용안하게 될때 fin, close_wait 이후 time_wait 으로 변경되어 리소스 정리되는걸 기대함.
- server keep alive 끄면 connection reuse 없고 매번 새로 establish 함
- client 를 매번 재생성하고 안하고는 실험 결과에 영향을 주지 않음
    - client 의 RoundTripper 는 defaultRoundTripper 로 계속 재활용 되긴 했을거라??
- partial read, read none 은 동일하게 동작한다. 
    - `readAll` 을 통해서 다 읽어냈을때 (EOF 까지) reuse 가 일어남 
    - `io.Discard` 까지 호출할필요는 없었던걸까??
- 다 읽는 경우는 close 하나 안하나 재활용이 일어났다
    - 다 읽는 경우에, 다 읽은 body close 되는 로직이 있는것같다.
- 다 안읽는 경우 close 하면 time_wait 으로 변했고, close 안하면 established 로 남아있었다
