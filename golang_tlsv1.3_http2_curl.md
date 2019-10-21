# intro
- golang 으로 tlsv1.3, http2 지원하는 서버를 띄우고, curl 을 이용해서 테스트 해보기 위한 과정 및 결과

# tl;dr
- golang 서버는 tls 설정으로 자연스러운 http2 지원이 되고, tlsv1.3 설정은 go 1.12 에 추가되었다. 
- osx 에 기본 설치되어있는 curl 로는 tlsv1.3 테스트가 되지 않아, 새로 빌드가 필요하다.

# step

- openssl v1.1.0 다운로드
- key, crt 파일 생성 with openssl
- golang tlsv1.3, http2 적용된 서버 생성 
- curl build
- test 

## openssl v1.1.0 다운로드 

tlsv1.3 지원을 하는 openssl v1.1.0 이상을 다운로드한다.

```bash
$ brew install openssl@1.1
```

## key, crt 파일 생성 with openssl

```sh
$ openssl req -new -x509 -sha256 -key server.key -out server.crt -days 3650
```

## golang tlsv1.3, http2 적용된 서버 생성

* go 1.12 에서 설정이 추가되었다. 
- 참고: https://golang.org/pkg/crypto/tls/ 

```golang
  os.Setenv("GODEBUG", os.Getenv("GODEBUG")+",tls13=1")
```

```golang
// curl 'https://localhost:9000/test' --tlsv1.3 --http2 -kIv
// needs -k flag because certificate and key file is self-made which is insecure.
// server using tls configuration in golang transparently serves http2

package main

import (
	"fmt"
	"net/http"
	"os"
)

func init() {
	os.Setenv("GODEBUG", os.Getenv("GODEBUG")+",tls13=1")
}

func main(){
	mux := http.NewServeMux()
	mux.HandleFunc("/test", TestHandler)

	server := http.Server{
		Addr: "localhost:9000",
		Handler: mux,
	}

	if err := server.ListenAndServeTLS("keys/server.crt", "keys/server.key"); err != nil{
		fmt.Println(err)
	}
}

func TestHandler(w http.ResponseWriter, r *http.Request){
	fmt.Println("Test called")
	fmt.Println(r)
	w.Write([]byte("hi hello"))
}
```

## curl build 

현재 osx 에 설치되어있는 curl 버전은 아래와 같고, tlsv1.3 지원이 안된다. 

```bash
$ curl --version                                                                                    ✘ 2 master
curl 7.54.0 (x86_64-apple-darwin18.0) libcurl/7.54.0 LibreSSL/2.6.5 zlib/1.2.11 nghttp2/1.24.1
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz HTTP2 UnixSockets HTTPS-proxy
```

```bash
$ curl 'https://localhost:9000/test' --tlsv1.3 --http2 -kIv                                             master
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 9000 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9000 (#0)
* LibreSSL was built without TLS 1.3 support
* Closing connection 0
curl: (4) LibreSSL was built without TLS 1.3 support
```

따라서, openssl v1.1.0 을 이용하는 curl 의 build 가 필요하다. 

- curl 소스 코드를 받는다. 
```bash
$ git clone https://github.com/curl/curl.git
```
https://github.com/curl/curl

GIT-INFO 파일을 열어보면 매뉴얼이 적혀있다. 
```bash
$ cd curl
$ cat GIT-INFO

... 생략

To build in environments that support configure, after having extracted
everything from git, do this:

./buildconf
./configure
make

```

위의 매뉴얼 대로 따라한다.
> automake 설치가 필요했다. 
```bash
$ brew install automake 
```

configure 시 ssl option 을 따로 줘야된다. 
위에서 설치한 openssl v1.1.0 의 경로를 입력해주고, 아래와 같이 순서대로 실행한다.  

```bash
$ ./buildconf
$ ./configure --with-ssl=/usr/local/opt/openssl@1.1
$ make
```

make 결과 파일은 ./src/curl 에 생성된다. 
새로 빌드된 curl 의 version 을 확인하면 아래와 같다. (openssl 1.1.1, nghttp2(따로 설정해주진 않았지만) 적용 확인 가능하다)
```bash
$ ./src/curl --version                                                                                                                                 master
curl 7.66.0-DEV (x86_64-apple-darwin18.6.0) libcurl/7.66.0-DEV OpenSSL/1.1.1c zlib/1.2.11 brotli/1.0.7 nghttp2/1.39.1 librtmp/2.3
Release-Date: [unreleased]
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS brotli HTTP2 HTTPS-proxy IPv6 Largefile libz NTLM NTLM_WB SSL TLS-SRP UnixSockets
```

## 새로 build 한 curl 을 이용한 test 진행 

```bash
$ ./curl 'https://localhost:9000/test' --tlsv1.3 --http2 -kIv                                                                                      master
*   Trying ::1:9000...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 9000 failed: Connection refused
*   Trying 127.0.0.1:9000...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9000 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_128_GCM_SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd
*  start date: Aug  2 05:19:45 2019 GMT
*  expire date: Aug  1 05:19:45 2020 GMT
*  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fdf79812200)
> HEAD /test HTTP/2
> Host: localhost:9000
> User-Agent: curl/7.66.0-DEV
> Accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* Connection state changed (MAX_CONCURRENT_STREAMS == 250)!
< HTTP/2 200
HTTP/2 200
< content-type: text/plain; charset=utf-8
content-type: text/plain; charset=utf-8
< content-length: 8
content-length: 8
< date: Mon, 05 Aug 2019 02:38:21 GMT
date: Mon, 05 Aug 2019 02:38:21 GMT

<
* Connection #0 to host localhost left intact
```


