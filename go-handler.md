## TL;DR

- http.Handler는 ServeHTTP를 구현하는 interface이다.
- http.HandlerFunc 는 type이면서 ServeHTTP를 구현하는 Handler interface이다.
- mux도 Handler interface 이다
- mux는 request path에 해당하는 Handler를 map에서 찾아 그 Handler의 ServeHTTP를 실행한다.
 
 ---
 
http.Handler는 ServeHTTP를 구현하는 interface이다.
```go
type Handler interface {
   ServeHTTP(ResponseWriter, *Request)
}
```

http.HandlerFunc 는 type이다. 
```go
type HandlerFunc func(ResponseWriter, *Request)
``` 

http.HandlerFunc는 ServeHTTP를 구현하므로 http.Handler interface 이기도 하다.
```go
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
   f(w, r)
}
``` 

아래의 HandlerFunc(func(ResponseWriter, *Request) 는 결국, func를 HandlerFunc로 type변환 하는 것이다.

http.HandleFunc는 DefaultServeMux를 ,  mux.HandleFunc는 미리 정해놓은 mux를 이용한다. 

```go
// Handle registers the handler for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
// func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
http.HandleFunc("/", HandlerFunc(func(ResponseWriter, *Request))
  
// HandleFunc registers the handler function for the given pattern.
// func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
//    mux.Handle(pattern, HandlerFunc(handler))
// }
mux.HandleFunc("/", HandlerFunc(func(ResponseWriter, *Request))
```

결국  func (mux *ServeMux) Handle (pattern string, handler Handler) 를 호출하고, Handler 를 map으로 저장한다.

```go
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
   mux.mu.Lock()
   defer mux.mu.Unlock()
 
   if pattern == "" {
      panic("http: invalid pattern")
   }
   if handler == nil {
      panic("http: nil handler")
   }
   if _, exist := mux.m[pattern]; exist {
      panic("http: multiple registrations for " + pattern)
   }
 
   if mux.m == nil {
      mux.m = make(map[string]muxEntry)
   }
   mux.m[pattern] = muxEntry{h: handler, pattern: pattern}
 
   if pattern[0] != '/' {
      mux.hosts = true
   }
}
``` 

mux도 ServeHTTP를 구현하므로 handler 이다.

```go
// ServeHTTP dispatches the request to the handler whose
// pattern most closely matches the request URL.
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
   if r.RequestURI == "*" {
      if r.ProtoAtLeast(1, 1) {
         w.Header().Set("Connection", "close")
      }
      w.WriteHeader(StatusBadRequest)
      return
   }
   h, _ := mux.Handler(r)
   h.ServeHTTP(w, r)
}
``` 

mux는 결국 request pattern에 가장 적합하게 등록된  mux 내의 Handler를 찾고, 그 Handler 에 매핑되어있는 Handler의 ServeHTTP를 호출한다.
```go
h, _ := mux.Handler(r)
h.ServeHTTP(w, r)
``` 
