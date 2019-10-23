# intro 

- `go 1.13` 이 release 되었고, `goproxy`, `gosumdb`, `goprivate` 등 몇 옵션이 추가됨

- `goprivate` 가 설정되어있으면, `goproxy`, `gosumdb` 값을 `goprivate`로 덮어쓴다. 상세스펙은 다음에 정리..

```go
GOPROXY   = envOr("GOPROXY", "https://proxy.golang.org,direct")
GOSUMDB   = envOr("GOSUMDB", "sum.golang.org")
GOPRIVATE = Getenv("GOPRIVATE")
GONOPROXY = envOr("GONOPROXY", GOPRIVATE)
GONOSUMDB = envOr("GONOSUMDB", GOPRIVATE)
```
- https://github.com/golang/go/blob/4be6b4a73d2f95752b69f5b6f6bfb4c1a7a57212/src/cmd/go/internal/cfg/cfg.go#L246-L250

- env 를 쉽게 줄 수 있게끔 `go env` 에 `-w` 옵션이 추가됐습니다.


```bash
$ go env -w GOPATH=/Users/fransoaardi/gogo 
```

와 같이 env 설정이 가능함

Russ Cox 가 작성한 proposal 에 따르면 어딘가에 `env` 가 저장되고, 그 설정이 overriding 되는 구조라고 함
그 `env` 파일은 `os.UserConfigDir()/go/env` 에 저장된다고 proposal 되어있는데

`os.UserConfigDir()` 은 OSX 기준 `$HOME/Library/Application Support`.

```
UserConfigDir returns the default root directory to use for user-specific configuration data.
Users should create their own application-specific subdirectory within this one and use that.

On Unix systems, it returns $XDG_CONFIG_HOME as specified by 
https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html

if non-empty, else $HOME/.config. On Darwin, it returns $HOME/Library/Application Support.
On Windows, it returns %AppData%. On Plan 9, it returns $home/lib.

If the location cannot be determined (for example, $HOME is not defined), then it will return an error.
```
실제로 `$HOME/Library/ApplicationSupport/go` 에 `env` 라는 파일이 있습니다.

`$HOME`은 `/Users/fransoaardi` 인 경우

```bash
$ go env -w GOPATH=$HOME/gogo
$ cat $HOME/Library/ApplicationSupport/go/env
GOPATH=/Users/fransoaardi/gogo
$ go env GOPATH
/Users/fransoaardi/gogo
```

`GOENV=off` 라는 옵션이 있음. 그러면 `env` 저장했던것을 overriding 하지 않음

```bash
~/L/A/go ❯❯❯ go env GOPATH
/Users/fransoaardi/gogo
~/L/A/go ❯❯❯ GOENV=off go env GOPATH
/Users/fransoaardi/go
```

- Reference

Russ Cox 의 go env 관련 proposal
- https://go.googlesource.com/proposal/+/master/design/30411-env.md

`os.UserConfigDir()` 관련 `godoc`
- https://golang.org/pkg/os/#UserConfigDir
