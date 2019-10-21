# GO multiple mod in one repository


## intro

사용중인 module 중에 특이하게 정의된 모듈이 있음
보통 `go.mod` 는 `repository/go.mod` 위치에 존재하지만
`go.mod` 가 repository 의 내부 directory (depth 존재) 에 정의되어있음

- **결론적으로, `go.mod` 는 한 repository 안에 여러개 정의 가능하고**

- **repo 의 root 가 아닌, 다른 위치에 정의 가능하다**

## 상황제시 

- `fransoaardi/gomod-parent`
- `fransoaardi/gomod-child`
- `apache/thrift`

위 3개의 repository 가 있고, 
> `gomod-parent` 는 `gomod-child` dependency 를 가지고있고,

> `gomod-child` 는 `apache/thrift` 를 dependency 로 가지고있는 상황이다. 
```bash
~/gomod-parent ❯❯❯ go mod graph                                                                                
github.com/fransoaardi/gomod-parent github.com/fransoaardi/gomod-child/client/go@v1.1.0
github.com/fransoaardi/gomod-child/client/go@v1.1.0 github.com/apache/thrift@v0.12.0
github.com/fransoaardi/gomod-child/client/go@v1.1.0 github.com/fransoaardi/gomod-child@v0.1.0
```
`gomod-child` 는 go client 뿐만 아니라, python, ruby 등의 client 를 제공해야되서 

`gomod-child/client/go` 에 `go.mod` 를 정의하고 go code 가 들어있다. 

- `gomod-child` release, tagging 진행해도, `gomod-child/client/go` 의 버전을 지정할 수 없었다. 

## directory 구조 
- `gomod-child`
```
~/gomod-child ❯❯❯ tree                                                                                          
.
├── README.md
├── client
│   └── go
│       ├── cmd
│       │   └── model.go
│       ├── go
│       ├── go.mod
│       ├── go.sum
│       └── main.go
└── go.mod
```
- `gomod-parent`
```
~/gomod-parent ❯❯❯ tree                                                                                   
.
├── README.md
├── go.mod
├── go.sum
├── gomod-parent
└── main.go

0 directories, 5 files
```

## 해결 

- `gomod-child` 의 root 에 빈 껍데기 격의 `go.mod` 를 생성한다
- `gomod-child/client/go` 밑에 `go.mod` 를 또 하나 생성한다
- `gomod-child/client/go/go.mod` 에 
```bash
$ go mod edit -require github.com/fransoaardi/gomod-child@v0.1.0
$ go mod edit -replace github.com/fransoaardi/gomod-child@v0.1.0=../../
```
와 같이 dependency 를 걸어버리고, 상대경로로 replace 를 한다.
- `gomod-child` repo 에서 `v0.1.0` 으로 release 한다. 
- **`gomod-child` repo 에서 `client/go/v1.0.0` 으로 release 한다.**
- **`gomod-parent` repo 에서 위에서 release 한 버전으로 require 한다.**
```bash
$ go mod edit -require github.com/fransoaardi/gomod-child/client/go@v1.0.0
```

## 참고 (go.mod 파일)

- `fransoaardi/gomod-parent/go.mod` 
```go
module github.com/fransoaardi/gomod-parent

go 1.13

require github.com/fransoaardi/gomod-child/client/go v1.1.0
```

- `fransoaardi/gomod-child/go.mod`
```go
module github.com/fransoaardi/gomod-child

go 1.13
```

- `fransoaardi/gomod-child/client/go/go.mod`
```go
module github.com/fransoaardi/gomod-child/client/go

go 1.13

require (
	github.com/apache/thrift v0.12.0
	github.com/fransoaardi/gomod-child v0.1.0
)

replace github.com/fransoaardi/gomod-child v0.1.0 => ../../
```

## Reference
- https://github.com/golang/go/wiki/Modules#is-it-possible-to-add-a-module-to-a-multi-module-repository
