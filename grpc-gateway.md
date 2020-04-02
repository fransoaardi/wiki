# gRPC gateway 테스트 

## intro 
테스트를 위해 gRPC gateway 를 구현해봤다. 

## references
- grpc gateway 구현 관련 
  - https://github.com/grpc-ecosystem/grpc-gateway
- swagger serving 관련
  - https://medium.com/@ribice/serve-swaggerui-within-your-golang-application-5486748a5ed4

## how-to
### grpc-gateway 관련 binary version 관리 

```golang
// +build tools

package tools

import (
    _ "github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway"
    _ "github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger"
    _ "github.com/golang/protobuf/protoc-gen-go"
)
```

```bash
$ go install \
    github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway \
    github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger \
    github.com/golang/protobuf/protoc-gen-go
```

이 결과 `protoc-gen-grpc-gateway`, `protoc-gen-swagger`, `protoc-gen-go` 가 $PATH 에 잡혀야됨

### protobuf gen 
`test.pb.go`(gRPC 파일), `test.pb.gw.go`(gRPC gateway 파일) 생성한다.  

> gRPC & gateway protoc gen
```bash
protoc -Iapi/proto/v1/ \
    -I/usr/local/include \
    -I$GOPATH/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
    --go_out=plugins=grpc:api/proto/v1 \
    api/proto/v1/test.proto

protoc -I/usr/local/include \ 
    -I. \
    -I$GOPATH/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
    --grpc-gateway_out=logtostderr=true:. \
    api/proto/v1/test.proto
```

> optional: swagger.json gen
```bash
protoc -I/usr/local/include -I. \
  -I/Users/hmc/go/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --swagger_out=logtostderr=true:. \
  api/proto/v1/XXX.proto
```

### 구조 설명
- gRPC gateway 는 http.Handler 이고, 내부적으로 gRPC client conn 을 갖고있다. 
- HTTP 요청을 받는 server(이하 gateway srv), gRPC server(이하 grpc srv) 를 각각 띄운다. 
- HTTP 요청인 경우, http srv 는 받은 요청을 gRPC 요청에 맞게 변경, client 입장에서 grpc srv call 한다.
- gRPC 요청인 경우, grpc srv 가 바로 처리한다. 

### .pb.gw 를 생성하면 생기는 일 
```go
pb.RegisterXXXServiceServer()   // gRPC server 의 rpc 를 정의하고, serving 과 연동하는 과정

// RegisterXXXServiceHandlerFromEndpoint is same as RegisterXXXServiceHandler but
// automatically dials to "endpoint" and closes the connection when "ctx" gets done.
pb.RegisterXXXServiceHandlerFromEndpoint(ctx, mux, endpoint, dialoptions)
> endpoint, dialoptions 로 conn 을 생성, 결국 pb.RegisterXXXServiceHandler(ctx, mux, conn) 를 호출함 

// RegisterXXXServiceHandler registers the http handlers for service XXXService to "mux".
// The handlers forward requests to the grpc endpoint over "conn".
pb.RegisterXXXServiceHandler(ctx, mux, conn)
> conn 으로 NewXXXServiceClient(conn) 생성, 결국 pb.RegisterXXXServiceHandlerClient() 호출함

// RegisterXXXServiceHandlerServer registers the http handlers for service XXXService to "mux".
pb.RegisterXXXServiceHandlerServer()

RegisterXXXServiceHandlerServer, RegisterXXXServiceHandlerClient 
크게 server, client 가 있는데 구현이 거의 동일한데, 전달받은 server의 RPC 호출과 client 의 RPC 호출하는 것만 다르다. 
grpcGateway 는 결국 두가지 방법 모두 HTTP 요청을 localhost 의 gRPC 요청으로 바꿔서 호출하게 될것이다.
```

## 구현
### gateway code
> `https://github.com/grpc-ecosystem/grpc-gateway/blob/master/examples/internal/gateway/gateway.go` 참고
```go
func Gateway(ctx context.Context, conn *grpc.ClientConn, opts []runtime.ServeMuxOption) (http.Handler, error) {
	mux := runtime.NewServeMux(opts...)

	for _, f := range []func(context.Context, *runtime.ServeMux, *grpc.ClientConn) error{
		pb.RegisterXXXServiceHandler,
	} {
		if err := f(ctx, mux, conn); err != nil {
			return nil, err
		}
	}
	return mux, nil
}
```

### grpc server code
```go
func GrpcServer() *grpc.Server {
	server := grpc.NewServer()
	service := new(xxxGrpcServer)
	pb.RegisterXXXServiceServer(server, service)

	return server
}
```

### main code 
```go
func main() {
	grpcListen, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatal("8080 port in use")
	}

	gatewayListen, err := net.Listen("tcp", ":8081")
	if err != nil {
		log.Fatal("8081 port in use")
	}

	grpcServer := server.GrpcServer()

	go func() {
		_ = grpcServer.Serve(grpcListen)
		defer grpcServer.GracefulStop()
	}()

	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	grpcConn, err := grpc.Dial(":8080", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("failed to connect: %v", err)
	}
	defer grpcConn.Close()

	gatewayMux, err := server.Gateway(ctx, grpcConn, []runtime.ServeMuxOption{})
	if err != nil {
		log.Fatalf("")
	}

	mux := http.DefaultServeMux
	mux.Handle("/", gatewayMux)
    // adding swagger json serving handler
	fs := http.FileServer(http.Dir("external/swaggerui"))
	mux.Handle("/swaggerui/", http.StripPrefix("/swaggerui/", fs))

	_ = http.Serve(gatewayListen, mux)
}
```

### protobuf 명세 관련
- `import "google/api/annotations.proto";` 추가 

- option 부분을 추가했다. 아래와같이, api path 생성할때, request param 을 추가할 수 있다.
예를들어 {phase} 는 enum 으로 "DEV", "SPRINT", "RELEASE" 의 대문자값으로 경로가 생성됐는데, 결과적으로 api path 가 /v1/api/DEV/A/12345 와 같이 소문자였으면 좋았을뻔했지만, 
이 부분이 마음에 들지 않았다. 실제로 enum 이라 valid 한 값이 아니면 grpc-gateway 차원의 error 를 발생했다. 
```proto3
service XXXService {
  rpc ReadItem(GetItemRequest) returns (Item) {
    option (google.api.http) = {
      get: "/v1/api/{phase}/{service}/{id}"
    };
  }

  rpc CreateItem(Item) returns (Item) {
    option (google.api.http) = {
      post: "/v1/api"
      body: "*"
    };
  }
}
```

```proto3
syntax = "proto3";
package xxx;

import "google/api/annotations.proto";

enum Phase {
    DEV = 0;
    SPRINT = 1;
    RELEASE = 2;
}

enum Service {
    UNDEFINED = 0;
    A = 1;
    B = 2;
    C = 3;
}

message GetItemRequest {
  Phase phase = 1;
  Service service = 2;
  string id = 3;
}

message Item {
  Phase phase = 1;
  Service service = 2;
  string id = 3;
  string payload = 4;
  string timestamp = 5;
}

service XXXService {
  rpc ReadItem(GetItemRequest) returns (Item) {
    option (google.api.http) = {
      get: "/v1/api/{phase}/{service}/{id}"
    };
  }

  rpc CreateItem(Item) returns (Item) {
    option (google.api.http) = {
      post: "/v1/api"
      body: "*"
    };
  }
}
```

## swagger 관련

- `npm install swagger-ui-dist` 를 하면 아래와같이 그대로 serving 만 하면 swagger.json 파싱 가능한 ui 가 제공되는 dir 가 생성된다.
```
$ ls swagger-ui-dist
README.md                           swagger-ui-bundle.js
absolute-path.js                    swagger-ui-bundle.js.map
favicon-16x16.png                   swagger-ui-standalone-preset.js
favicon-32x32.png                   swagger-ui-standalone-preset.js.map
index.html                          swagger-ui.css
index.js                            swagger-ui.css.map
oauth2-redirect.html                swagger-ui.js
package.json                        swagger-ui.js.map
```

- 아래와같이 경로를 잘 잡고, fileserve 한다. 
```go
fs := http.FileServer(http.Dir("external/swaggerui"))
mux.Handle("/swaggerui/", http.StripPrefix("/swaggerui/", fs))
```

- `swagger.json` 파일을 dir 에 옮기고, `swagger-ui-dist/index.html` 에 아래와같이, 기본 url 을 바꿔준다. 
```javascript
window.onload = function() {
    // Begin Swagger UI call region
    const ui = SwaggerUIBundle({
    url: "./xxx.swagger.json", // << 이부분
    dom_id: '#swagger-ui',
    ...
}
```
