# intro
- nginx blog 에서 읽은 `Microservices: From Design to Deployment` 정리
> by Chris Richardson
- 7부로 되어있음

# 1부, introduction to microservices
> https://www.nginx.com/blog/introduction-to-microservices/

`monolithic application` 은 
- simple to develop (IDE 는 단일 app 만드려는 목표로 쓴다)
- simple to test (end to end test 를 쉽게 할 수 있다)
- simple to scale (loadbalancer 뒤에 여러 서버 띄우면 된다)

하지만 
- overwhelmingly complex (한명의 개발자가 전체를 이해할 수 없다)
- slow down development (start up 하는데 40분씩 걸린다)
- continuous deployment 못한다 (한개 배포하면 전체를 다시 재실행해야됨)
- conflicting resource requirement (A 기능은 memory intensive, B 기능은 computing intensive 일때 두 개 조건을 모두 만족하는 머신에서 실행해야됨 )
- lack in reliability (한 모듈 문제생기면(memory leak) 다같이 영향받음)
- difficult to adopt new frameworks and languages (framework 갈아타거나, 새 기술 적용이 힘듬, 처음 적용한 기술만 쓰게됨)
> agile development and delivery of applications is impossible 

그래서 `microservices architecture pattern` 을 적용해서 복잡성을 낮춘다. 

- `scale cube` (from `the art of scalability`)
  - x axis: horizontal duplication: `scale by cloning`
  - y axis: functional decomposition: `scale by splitting different things`
  - z axis: data partitioning: `scale by splitting similar things`

- 앞단에 load balancer 를 놓고, 
  - caching 
  - access control 
  - API metering 
  - monitoring 
등을 해결함

- 다른 서비스들과 단일 db schema 를 공유하는것 대신 서비스별로 각자 db schema 를 가짐
- duplication of some data 를 일으키기도 하지만, loose coupling 을 하기때문에 중요함 
- 각 서비스들은 각 db 를 갖고, database adapter 를 통해 연결함 (loose coupling)
- 각자 db 를 갖기때문에, 필요에 맞는, 가장 적합한 db 를 활용함 (polyglot persistent architecture): (geoquery 필요하면 mongodb 등을 쓸듯)

- MSA 는 SOA 와 비슷하지만, MSA 는 더 경량화된 프로토콜(REST) 를 이용하고, ESB(Enterprise Service Bus), WS(web service specifcations) 를 사용하지 않음 

## MSA 장점 

- 잘 정의된 RPC, message-driven API 를 가진다 
- monolithic code base 에서 불가능한 모듈화를 강요한다 
> much faster + easier to develop/maintain/understand

- 서비스에 특화된 팀에의해 독립적으로 개발될 수 있음 
> 기술 선택의 자유, 더이상 버려진 기술 쓰지 않아도됨, 새로 개발해도됨 

- 독립적으로 배포할 수 있음. 
> 다른 모듈에 영향을 적게줌

- 독립적으로 scale 할 수 있음 
> resource 효율적으로 씀 

## MSA 단점 

> there is no silver bullets

- service size 가 작기를 강조하지만, 꼭 무조건 작아야 될 필요는 없다. 
> it's important to remember that they are a means to an end and not the primary goal.
> The goal of microservices is to sufficiently decompose the application in order to facilitate agile application development and deployment.

- 분산처리 시스템이 갖는 복잡성 증가 
> messaging, RPC 중의 내부 통신방법(inter-process communication)을 사용해야됨 

- db 가 나뉘어있어 distributed transaction 을 사용하게되고, eventual consistency based approach 를 사용할것이라 관리하기 어려움 

- 테스팅이 힘듬

- A > B > C 로 영향을 준다고 하면 조심해서 배포해야됨.(이런일이 상대적으로 적긴함)

- 배포가 복잡함, automation 이 필요함 (PaaS, kubernetes, docker 등..)


## summary 

> A Monolithic architecture only makes sense for simple, lightweight applications.

> The Microservices architecture pattern is the better choice for complex, evolving applications despite the drawbacks and implementation challenges.

