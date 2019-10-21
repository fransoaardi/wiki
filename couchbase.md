# Couchbase

## intro

기존 Couchbase 4.x 사용중이었는데 Couchbase 5.x 으로 업데이트 진행해보았다.

## install

### docker로 간편하게 설치
docker run -d --name db -p 8091-8094:8091-8094 -p 11210:11210 couchbase

### CentOS-7 용 couchbase community version 다운로드
curl -O https://packages.couchbase.com/releases/5.0.1/couchbase-server-community-5.0.1-centos7.x86_64.rpm

## test

아래의 소스에서 cluster.Authenticate 부분이 4.x 와는 다르게 추가되었다.
참고
- https://developer.couchbase.com/documentation/server/5.5/sdk/go/start-using-sdk.html

```go
package main

import (
	"fmt"
	"github.com/couchbase/gocb"   // 예시와 같이 gopkg. 하면 패키지 자체 에러가 난다.
                                // 추가로 gopath/src/gopkg.in/{couchbase, couchbaselabs} 싹다 삭제해주었다.
)

func main() {
        cluster, _ := gocb.Connect("couchbase://localhost")
        cluster.Authenticate(gocb.PasswordAuthenticator{  // 4.x와 달리 5.x에서 Authenticate 부분이 추가되었다. (4.x에선 필요없다)
            Username: "userid",
            Password: "password",
        })
        bucket, _ := cluster.OpenBucket("bucketname", "")
        
        bucket.Manager("", "").CreatePrimaryIndex("", true, false)
        
       // ...  나머지는 동일
```

## more

- dashboard 자체기능이 강화되었다
bucket 내 document 내용이 2.5k 넘어가는 경우에는 dashboard 내 read/write 가 불가능했는데, 이젠 가능하다.

- ui 개선은 당연

## trouble shooting

- 통검 lineup db 문제 (4.x->5.x 과는 무관하다)

1. cluster 중 일부 node 가 disk 혹은 memory full 이 발생함 
2. cluster 한대는 stop, 나머지 3대는 pending 상태로 남았음 
3. 한대 failover 처리 후, 나머지 3대 rebalancing 시도하였음
4. rebalance 실패하는 경우, 문제가 남아있는 일부 node 다시 확인해서 강제 failover 처리하였음 (pending 이지만 disk full 상태였음)
5. primary index 생성 중 발생한 오류로, cbq 직접 접근해서 primary index drop 진행 
  
```
$ cd /opt/couchbase/bin
$ ./cbq
cbq> select * from system:indexes;
{
    "requestID": "b69a9a92-b953-48f4-be2b-759fbd202b59",
    "signature": {
	"*": "*"
    },
    "results": [
    ],
    "status": "success",
    "metrics": {
	"elapsedTime": "30.54725ms",
	"executionTime": "30.503126ms",
	"resultCount": 0,
	"resultSize": 0
    }
}
# 지금은 없지만, 
# primary index 를 drop 진행 

cbq> drop primary index on `lineup`;
```
7. 삭제 후 cluster 에 failover 처리한 node 붙이는 작업시도
8. 특정 node 가 add server 가 안되고, `service couchbase-server restart` 도 제대로 동작을 안해서, yum remove 시도 
9. yum list installed 확인 후 yum remove `COUCHBASE 설치된 pkg 명` 해서 삭제
10. yum 으로 couchbase install 다시 진행 
11. add server 하여 다시 cluster 
12. cluster 구성 후 bucket 을 flush 하였음 (cluster 구성 후 하는것이 안전한것같음)
  
### reference

 - https://docs.couchbase.com/server/6.0/n1ql/n1ql-language-reference/dropprimaryindex.html
