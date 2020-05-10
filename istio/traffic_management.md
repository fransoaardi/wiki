# intro
- Istio `Traffic Management` 부분 정리, 번역 
> https://istio.io/docs/concepts/traffic-management/

## Traffic Management

### Virtual Services
- Istio service mesh 에서 request 들이 어떻게 routing 되는지에 대한 설정 
- routing 에 대한 세부적인 설정이 가능함, 예를들면 20%만 새 버전을 호출하거나, 이러한 사용자에게는 v2 를 호출하거나, canary 적용.
- kubernetes 는 instance scaling 으로만 traffic distribution 이 가능한데, istio 를 사용하면 가능.
- gateway 를 같이 이용해서 ingress/egress traffic 제어 가능

### Destination Rules
- virtual service: 어떻게 destination 으로 traffic 을 route 할것인가
- destination rules:  destination 에 도달한 traffic 에 어떠한 일이 발생하는가?
- destination rule 은 virtual service 에 의해 evaluate 된 이후에 적용됨

#### Load balancing options
- 기본은 Round-robin 
- Random
- Weighted
- Least Requests 

### Gateways
- inbound, outbound traffic 과리하는데 사용 
- sidecar 로 떠있는 Envoy 에 적용되는것이 아니라, 단독으로 떠있는 Envoy 에 적용됨 
- layer 4-6 load balancing 설정을 할 수 있음
- ingress 뿐만아니라 egress 설정 가능, exit node 설정하는 경우에 사용.
- demo 설정으로 설치하면 기본으로 istio-ingressgateway, istio-egressgateway 가 있고, 
default 로 설정하면 ingress gateway 가 있음. 따로 만들어도됨 
- gateway 설정 한 다음에 앞에 virstualservice 로 연결해줘야됨

### Service Entries
- Istio 가 내부적으로 관리하는 service registry 에 등록 가능.
- service entry 에 등록하면, Envoy 는 mesh 안에 있는것 처럼 호출함 

- 이용할때
  - 외부 API, legacy infrastructure 에 있는 서비스 
  - retry, timeout, fault injection 설정 
  - VM 을 mesh 에 등록해서 이용
  - Kubernetes 상의 multicluster Istio mesh 설정을 위해서 

- 기본적으로 mesh 밖에 있는 서비스를 이용하는데 제약은 없음, Envoy proxy 통해서 외부 서비스 호출 가능
- mesh 에 등록되지 않은 서비스에 Istio feature 을 사용할 수 없음

### Sidecars
- Istio 는 기본으로 Envoy proxy 가 모든 포트의 traffic 을 받아들이도록 한다.

- 아래와 같이 설정 가능
  - Envoy proxy 가 accept 하는 포트와 프로토콜 설정 
  - Envoy 가 호출할 수 있는 services 제한 

- 큰 application 에서 sidecar reachablity 를 설정하고 싶을 수 있지만, 모든 다른 서비스에 설정하면 높은 메모리 사용량으로 잠재적으로 mesh performance 에 영향줄 수 있음 
- workloadSelector 를 이용해서 특정한 workload 를 지정하거나, namespace 로 지정해서 설정을 적용 가능. 


### Network resilience and testing 
- runtime 에 동적으로 설정 가능한 여러 기능들이 있음 
- 노드 연결 실패를 견디고, 다른 노드로 영향이 번지는것을 방지함
- Istio 의 failure recovery features 는 application 과 완전히 독립적이다, 따라서 설정이 충돌할수도 있다. 
- 예를들면, application 에 2 초 타임아웃, Istio 에 3초 timeout + retry 를 설정했으면 app 에서 timeout 발생하고, envoy 는 한번 더 호출하게된다, retry 가 무의미해진다.
- 모든 load balancing pool 에 있는 instance 들이 fail 했다면, Envoy 는 503 error 를 return 하기 때문에, 503 에 대한 에러처리가 잘 되어있어야됨.


#### Timeouts 
- 주어진 서비스로부터 응답을 받을때까지 기다려야되는 시간
- HTTP request 기본은 15초
- 서비스 코드를 변경하지 않고, virtual services 를 이용해서 동적으로 타임아웃 설정을 바꿀 수 있음 

#### Retries
- 첫번째 호출이 실패하면 최대 몇번까지 Envoy proxy 가 연결을 시도할지에 대한 설정
- 일시적으로 서비스나 네트워크에 과부하가 걸리는 상황에 대한 가용성을 높임 
- 기본으로 Envoy 는 실패한 서비스에 대해 재시도 하지 않음
- retry interval 은 Istio 에 의해 자동으로 결정되고, retry request 에 의해 과부하 걸리는것을 막음 
- 서비스 코드를 변경하지 않고, virtual services 를 이용해서 각각 설정 가능함

#### Circuit breakers
- circuit breaker pattern 이용하면 client 가 과부하됐거나 문제생긴 host 에 연결하려고 노력하는것 대신 빠르게 실패하게함 
- destination rules 에 적용할 수 있음 

#### Fault injection
- 전체 application 에 failure recovery capacity 를 테스트 해보기 위한 fault injection 가능 
- 에러 상황에서 견뎌내거나 복구할 수 있는지를 확인하는 방법 
- failure recovery 정책이 적절하지 않거나 너무 과하거나 하진 않은지 확인할 수 있음 
- packet 을 지연시키거나 network layer 에 pod 를 죽이거나 하는 대신, Istio 는 application layer 에 faults 를 inject 함
- HTTP error code 를 주거나 함

- virtual services 를 이용해서 두가지 방식의 fault 를 inject 가능
  - Delays: timing failure, network 지연 증가 혹은 upstream service 의 과부하 상태를 가정함
  - Aborts: crash failure, upstream service 의 실패, HTTP error code 를 주거나 TCP connection failure 상황을 가정함 

- 예를들면, 0.1% 의 request 에 5초 지연을 줌 




# references
- https://istio.io/docs/concepts/traffic-management/
