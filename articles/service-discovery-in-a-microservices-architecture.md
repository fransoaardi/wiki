# intro
- `NGINX` blog 에서 읽은 `Microservices: From Design to Deployment` 정리
> by Chris Richardson
- 7부로 되어있음

# 4부, `Service Discovery in a Microservices Architecture`
> https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/

# Why Use Service Discovery?

- REST API/ Thrift API 를 사용하는 경우에, network location (IP, port) 정보를 알아야됨
- 물리장비에서 실행되던 전통적인 application 의 network locations 는 정적이고, 가끔씩 업데이트 되는 config 파일에서 정보를 읽으면 됨

cloud-based msa 에서는 app 의 주소가 동적으로 할당되어, network location 관리가 힘듬
(autoscaling, failure, upgrade 발생하면서 동적으로 바뀜)

> 그러므로 service discovery 를 적용해야됨 

- client-side discovery & server-side discovery 가 있음 

## The Client‑Side Discovery Pattern

- 사용가능한 network location 을 판단하고, load balance 하는것이 client 의 책임
- client ->(query)-> service registry 해서 available server instances 알게됨
- client 는 load-balancing algorithm 을 사용하고 request 를 생성 

- service start up 할때 service registry 에 network location 을 등록함 
- service instance terminate 할때 service registry 에 network location 지움 
- heartbeat 을 활용해서 service registry 저장된 값을 주기적으로 refresh 함

e.g.
- Netflix OSS (client-side discovery pattern)
- Netflix Eureka (serice registry) 
> service-instance registration 관리하는 REST API를 제공함 
- Netflix Ribbon: Eureka 를 이용하여 load balance 하는 IPC client 

client-side discovery pattern 은 장/단점이 다양함
장점
- 직관적인 패턴
- service registry 를 제외하고 바뀌는 부분은 없음 
- client 가 사용가능한 service instance 정보를 알고 있기 때문에, application-specific load-balancing 이 가능함 (consistent hasing 등..)
단점
- client, service-registry 를 coupling 함 
- client-side service discovery logic 을 clilent 에 구현해야됨 

## The Server‑Side Discovery Pattern
- client 는 load-balancer 를 통해 service 에 request 함 
- load balancer 는 service registry 에 query 하고, 사용가능한 service instance 에 route 함 
- client-side discovery 와 동일하게 service registry 에 service instances 들을 등록/해제 함 

e.g.
AWS Elastic Load Balancer(ELB):

- Internet 으로 오는 외부 트래픽을 loadbalance 하는데 사용됨, 
Virtual Private Cloud(VPC) 로 들어오는 내부 traffic loadbalance 에도 사용됨 
- client 는 HTTP/TCP request 를 DNS 를 사용하여 ELB 로 요청함 
- ELB 는 Elastic Compute Cloud(EC2) 혹은 EC2 Container Service(ECS) container 로 loadbalance 함.
- 따로 service registry 를 갖고있지는 않지만 ELB 자체에 EC2, ECS 정보가 등록되어있음 


NGINX Plus, NGINX 
- Consul Template 를 이용해서 NGINX reverse proxying 설정을 주기적으로 재생산함 
- Consul service registry 에 저장된 설정 데이터를 가지고 주기적으로 가변적인 설정파일을 재생산함 


Kubernetes, Marathon
- cluster 내 각각의 host 에 proxy 를 실행함, proxy 는 server-side discovery load balancer 로 동작함 
- client 는 host 의 IP address 를 이용하고 proxy 를 통해서 service 의 할당된 port 로 request 를 route 한다. 
- proxy 는 cluster 내에 있는 사용가능한 service 로 request 를 있는 그대로(transparently) 전달한다. 

장점:
- discovery 과정이 client 에는 추상화 되어있다. (client 는 단순히 loadbalancer 로 request 할뿐)
- 따라서 client 에 loadbalancer 로직을 구현할 필요가 없다. 
단점:
- 결국 이것도, 관리해줘야될 시스템 component 이다

# The Service Registry
- service instances 의 network locations 를 담은 database 
- service registry 는 highly available 하고 up to date 해야됨 
- client 가 network locations 를 캐시할 수 있지만, 결국 그 정보는 만료가 될것
- 따라서, service registry 는 일관성을 유지하는 서버 클러스터로 구성되야함

- Netflix Eureka 는 service instances 를 검색, 등록하는 REST API 를 제공함- 
- POST request 로 register 함 
- 30초마다 PUT request 로 등록을 새로고침해야됨
- DELETE 명시적 호출, 혹은 등록 timeout 일어나면 제거됨 
- GET 으로 가져올 수 있음 

- Netflix 는 Eureka 서버를 Elastic IP 가 할당된 EC2 인스턴스에 띄운다.
- Eureka 서버가 실행되면, Eureka cluster 설정을 얻어오기위해 DNS query 하고, peers 를 찾고, 미사용된 Elastic IP 를 자신에게 세팅함 

- Client 는 같은 availability zone 에 있는 Eureka 를 사용하는걸 선호하지만, 불가능한 경우에 다른 zone 에 있는 Eureka 를 사용한다. 

다른 service registry 방식 
- etcd: 
  - service discovery 와 공통 설정을 위한 고가용성, 분산, 일관된 k-v store. 
  - Kubernetes, Cloud Foundary 에서 사용중

- consul:
  - service 설정을 discovery 하는데 사용하는 툴 
  - client 에게 service 등록/discover 하는 API 제공
  - health check 기능 있음

- Apache Zookeeper:
  - Hadoop sub-project 였는데 이제는 단독 프로젝트
  - 분산 application 의 동기화를 위한 고성능 서비스. 
  - 널리 쓰임 
  
- Kubernetes, Marathon, AWS 는 service discovery 를 밖으로 빼놓지 않고, 
built-in 으로 제공함 

# Service Registration Options
- service 등록/해지 여러가지 방법들이 있음 
  - self-registration pattern: 스스로 관리하는것 
  - third-party registration pattern: 다른 시스템 component 가 관리하는것

## The Self‑Registration Pattern
- service instance 가 등록/해지의 책임을 가짐
- 자신의 등록이 만기되는것을 방지하려고 heartbeat request 를 보냄 
- e.g.
  - Netflix OSS Eureka client 
  - Sprint Cloud project (@EnableEurekaClient annotation 으로 쉽게 설정함)

- 장점
  - 다른 system component 필요 없고, 상대적으로 간단함 
- 단점 
  - service 를 service registry 와 결합함 

## The Third‑Party Registration Pattern
- service instance 는 register 관리의 책임이 없음 
- service registrar 가 등록을 관리함 
- service registrar 가 polling/ subscribing events 방식을 이용해서 실행중인 인스턴스들의 변경을 추적함 
- 새로운 service instance 발견시 등록함 
- 종료된 instance 는 제거함 
- registrar 이 service instance healthcheck 진행하고 service registry 에 변경을 수정하는 방법, 
  
e.g.
  - Registrator project
    - Docker container 로 deploy 된 service instances 를 register/deregister 함 
    - etcd, Consul 지원함 
  - NetflixOSS Prana 
    - non-JVM language service 를 의도함 
    - service instance 와 같이 실행하는 sidecar application 
    - Prana 는 Netflix Eureka 를 이용해서 service register/deregister 함 

  - AWS EC2 with ELB 
    - Autoscaling Group 으로 생성된 instances 는 자동으로 등록됨 

  - Kubernetes

- 장점
  - service 는 service registry 와 decouple
  - service registration logic 을 직접 구현하지 않아도 됨 
  - 대신 중앙집중해서 처리 가능함 

- 단점 
  - 고가용성을 유지해줘야되는 새로운 시스템 컴포넌트임. (관리포인트 추가됨)

# Summary 
- MSA 에서는 service instances 는 동적으로 바뀜 
- 동적으로 할당되는 network locations 를 가지기 때문에 무조건 service-discovery 방식을 이용해야됨

- service registry 가 핵심 
  - database of available service instances
  - management API 와 query API 를 제공함 
  - management API 는 register/deregister 를 함 
  - query API 는 사용가능한 service instance 발견에 활용 

Query API 관련
- client side discovery 패턴
  - client 가 service registry 에게 query, 사용가능한 instance 고르고, request 함 
- server-side discovery 패턴
  - client 가 router 에게 요청, router 가 service registry 에 query, 사용가능한 instance 로 forward

Manage API 관련
- self-registration pattern 
  - service 가 직접 등록함
- third-party registration pattern   
  - 다른 시스템이 대신해서 관리함 

- 어떤 환경에서는 Netflix Eureka, etcd, Apache Zookeeper 와 같은 service-discovery infrastructure 를 직접 구축해야됨 
- Kubernetes, Marathon 에서는 registration/deregistration 을 관리해주고, server-side discovery router 로 동작하는 proxy 를 갖고있음

- HTTP reverse proxy + load balancer 를 이용하고 Consul Template 등을 이용해서 동적 설정을 구현할 수 있음.
> 솔직히 좋은방법 같지 않음 

