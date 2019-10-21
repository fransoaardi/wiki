더 자세한건 go wiki(https://github.com/golang/go/wiki/Modules)와 `go help mod` 를 참고해주세요. 

go mod 를 간단히 설명을 드리면, $GOPATH 밖에서도 빌드 가능하고, $GOPATH 를 '덜쓴다' 입니다. 
이전에는 go build 할때는 glide 혹은 dep으로 vendor 내에 받아놓은 경로 참조가 가능했지만, go test 시에는 $GOPATH를 찾게되어 
문제가 좀 있었습니다. 이번에 go mod 는 위 문제가 해결된것으로 보입니다. 

이전에는 GOPATH 에 버저닝 상관없이 go get 하는 시점의 최신 repo를 내려받는거였습니다.
예전에 toml 을 받아놨었다면, 현재의 toml 이 최신인지 아닌지 상관없이 get 하지 않을것이고 없었다면 새로 받습니다.

이제는 $GOPATH/pkg/mod 에 pkg 를 버저닝해서 관리합니다. 
두 가지 경우로 나뉠텐데요,
```
1) 현재 소스가 GOPATH/src 에 있는 경우
2) GOPATH 밖에 있는경우
```

1) 의 경우에는 `GO111MODULE=on go build` 와 같이 `GO111MODULE=on` 값을 줘야 강제할 수 있습니다. 
현재는 `GO111MODULE=auto` 로 되어있고, `auto`는 1)의 상황에는 `GO111MODULE=off` 로 동작하여 
`go build` 가 go1.11 미만 버전과 동일하게 동작합니다 (쓰시던대로 쓰면 됩니다)

2)의 경우에는 `GO111MODULE=on` 으로 동작하여 `go mod` 가 동작하게됩니다. 
`GO111MODULE=on go mod init` 
을 하게되면, `go.mod` 파일이 생성되고, `GO111MODULE=on go build` 하게 되면 `go.mod` 에 써져있는 명세대로 버저닝 된 pkg 다운로드가 시작됩니다. 
$GOPATH/pkg/mod 에서 확인할 수 있습니다. 
`go build` 이후에 `go.sum` 파일은 `go mod verify` 할때 사용됩니다. (go mod verify, tidy 등 go help mod 를 참고해주세요)

`go.mod` 가 생성된 이후에는, `go build` 할때 이전에 go dep, glide 가 그랬듯 버전이 명세된 대로 pkg 를 다운받아 build 할때 사용합니다.
이후에 `go bulid` 할때는 `go.mod` 에 명세된 대로만 다운받고 사용합니다.

`go.mod` 에 명세된 pkg 업데이트가 필요할때는 `GO111MODULE=on go get -u` 와같이 사용할 수 있습니다. (`dep ensure -update` 와 비슷하게 동작합니다)
`go get -u github.com/fransoaardi/pkghere` 와 같이 사용하시면 한 pkg 만 업데이트 가능합니다. 
이 경우 `go.mod` 에 명세된 전체 pkg 를 확인하고 사용가능한 minor 버전을 업그레이드 합니다. (major 버전업그레이드 관련해서는 아직 잘 모르겠습니다. 문서를 참고해주세요)
