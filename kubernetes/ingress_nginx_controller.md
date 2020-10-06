# ingress-nginx-controller 

## troubleshoot
### `stream terminated by RST_STREAM with error code: PROTOCOL_ERROR` 
#### introduction
- ingress-nginx-controller 를 이용해서 gRPC 를 적용했는데, bidi-stream 을 열고, 60 초만 되면 client 에서 serverStream recv 하는 순간 `stream terminated by RST_STREAM with error code: PROTOCOL_ERROR` 에러가 발생했다.
- bidi-stream 에서 server to client 는 주기적으로 message 를 보내고 있었고, client to server 는 아무런 message 를 보내고 있지 않았다.
- 처음에는 tls termination 이 일어나면서 발생하는 문제라고 생각했다. 하지만 결과적으론 아니다. (client 는 tls enabled client, server 는 중간에 ingress-nginx 에서 tls-termination 이 일어나서 server 는 tls 없었다)
- (결정적인 힌트) client to server 를 주기적으로 message 를 보내기 시작하니 끊어지지 않았다, timeout 이 문제일것이라고 생각했다.
#### progress
- `ingress-nginx` 는 결국 ingress 로 nginx 를 띄우는것과 동일한 것이라, 모르는사이에 nginx 의 기본 config 들이 적용된다. 
> 아래 문서가 도움이 되었다.
- https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/grpc#notes-on-using-responserequest-streams
```
If you do both response and request streaming with an open stream longer than 60 seconds,
you have to change all three timeouts: grpc_read_timeout, grpc_send_timeout and client_body_timeout.

Values for the timeouts must be specified as e.g. "1200s".
On the most recent versions of nginx-ingress, changing these timeouts requires
using the `nginx.ingress.kubernetes.io/server-snippet annotation`. 
There are plans for future releases to allow using the Kubernetes annotations to define each timeout seperately.
```
- `grpc_read_timeout`, `grpc_send_timeout` 등을 적용할때는 k8s annotation 이 별도로 제공되진 않고, `server-snippet` annotation 을 이용해서 별개로 적용해줘야된다.

#### special thanks to
> jontro 의 thread 에서 해답을 찾을 수 있었다, lucky3leaf 이 삽질을 함께 도와주었다. 감사합니다.
- https://github.com/jontro
