# intro 
- Istio 공부 중 architecture 부분 번역
> https://istio.io/docs/ops/deployment/architecture/

## istio architecture
- service mesh 는 논리적으로 data plane, control plane 으로 나뉨
- data plane 은 sidecar 로 배포된 envoy proxy 로 구성됨, 이런 proxies 가 micro service 사이의 network communication 을 중재하고 제어함, mesh 간 traffic 을 기록함 
- control plane 은 traffic route 설정하고 관리함

- data plane: business logic 부분
- control plane: 설정 부분 

### components
- core components 로는 envoy, pilot, citadel, galley 가 있음.

#### envoy
- c++ 로 개발된 고성능 proxy, 자세한 설명은 생략.

#### pilot
- service discovery 담당.
1. 서비스가 시작되면 platform adapter 에 새 시작을 알림 
2. platform adapter 가 pilot abstract model 에 등록함
3. envoy api 를 이용해서 pilot 이 새로운 traffic rule, configuration 을 배포함

#### citadel
- service-toservice & end-user authentication with built-in identity and credential management. 
- 보안담당, layer3, 4 가 아닌 service identity 기반의 정책을 적용가능. 

#### galley
- Galley is Istio’s configuration validation, ingestion, processing and distribution component


# references
- https://istio.io/docs/ops/deployment/architecture/
