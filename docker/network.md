# intro 
Docker network 관련해서 모르고 이용했던것이 많았고, 우연한 기회에 문서를 찾아 읽었다. 
docker network 는 `docker0` 인터페이스가 `osx` 에는 없고 여러 유틸(iptable 등.. )등을 고려하여, linux 를 기준으로 작성한다.

# 

- docker 가 설치된 linux host 에서 ifconfig 명령어를 입력시 docker0 network interface 가 생성되어있다. 
> ifconfig
```
ifconfig
docker0: 
eth0: 
```

- docker network ls 해보면 default bridge, host, none network 가 이미 만들어져있다. 
> docker network ls
```
NETWORK ID          NAME                DRIVER              SCOPE
3c8be4b70b6d        bridge              bridge              local
5e62d92497d8        host                host                local
571e2458615e        none                null                local
```

- docker container 생성시 --network 값을 지정하지 않으면 bridge 로 자동지정된다, 2개의 컨테이너를 사용중이었는데 이미 bridge 에 종속되어있다. 

> docker network inspect bridge -f "{{json .Containers}}"
```
{
    "4dcebe0b0b6c08fee3a05b6e4c21cd7aa5d251e9c8984acbdf3e8f7ef5b54a13": {
        "Name": "container-1",
        "EndpointID": "...",
        "MacAddress": "...",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    },
    "f2f4793ca08aa27ad2695ecbe9f3f507d27b97f5513da2a987601106d37c3122": {
        "Name": "container-2",
        "EndpointID": "...",
        "MacAddress": "...",
        "IPv4Address": "172.17.0.3/16",
        "IPv6Address": ""
    }
}
```

- default bridge vs user-defined bridge

user-defined bridge 를 구성하려면 아래와같이 network create 하고 사용하면 된다. 

> docker network create user-bridge
> docker network ls
```
NETWORK ID          NAME                DRIVER              SCOPE
3c8be4b70b6d        bridge              bridge              local
5e62d92497d8        host                host                local
571e2458615e        none                null                local
4f1876f97493        user-bridge         bridge              local
```

- user-defined bridge 와 docker default bridge 사이의 성능차이는 이해하기 힘들다.
ifconfig 해보면 방금 생성한 bridge 가 host 의 새로운 network interface 로 생성되어있다.

새로 생성한 user-bridge 에 새로운 container alpine1 을 설정해보고 brctl (bridge control) 을 통해서 확인해보았다. 

```
$ docker run -dit --name alpine1 --network user-bridge alpine ash
1cc46571a3c95281a78d567c7cdd744c1096a904e4ce56f37f74bfc8c6cf0d99

brctl show docker0
bridge name	bridge id		STP enabled	interfaces
docker0		8000.02421a0751ea	no		veth75bcd24
							vethb437c84
              
brctl show br-4f1876f97493
bridge name	bridge id		STP enabled	interfaces
br-4f1876f97493		8000.0242f74574ba	no		veth47ad8dd
```

default vs user-defined hop 을 예상해보면

host eth0 -> host docker0 -> host 

```
br-4f1876f97493: 
docker0:
```

# references
docker official docs network overview
- https://docs.docker.com/network/
docker official docs using user-defined + default bridge network tutorial
- https://docs.docker.com/network/network-tutorial-standalone/
docker network 관련
- https://bluese05.tistory.com/15 
- https://jeongchul.tistory.com/636
