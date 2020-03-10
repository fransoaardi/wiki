# intro
- `NGINX` blog 에서 읽은 `Microservices: From Design to Deployment` 정리
> by Chris Richardson
- 7부로 되어있음

# 2부, Building Microservices: Using an API Gateway
> https://www.nginx.com/blog/building-microservices-using-an-api-gateway/

# introduction 
- `monolithic application` 에서는 `single REST call` 할것이고, `load balancer` 가 N 개의 동일한 `application instances` 로 라우팅할것 
- 반면에 `microservices` 는 필요한 `service` 에 요청할것

# direct client-to-microservice communication 
- `client` 가 `microservices` 로 직접 `request` 할 수는 있다. 
하지만
- `client` 가 각각의 `request` 를 요청하는건 복잡하다. (client code complexity 올라감)
- `microservices` 가 `web-friendly` 하지 않은 `protocol` 을 사용할 수도 있다. 
  - `thrift binary RPC`, `AMQP` 은 `browser/firewall friendly` 하지 않아 internal 용도로만 쓰인다. 
  - `firewall` 밖에는 `HTTP`, `WebSocket` 을 사용해야된다.
- `microservices` refactor 힘들다. 
  - 한 서비스를 2개로 나누고 싶은데, client 가 직접 호출하고있었으면 다시 연동해야될것 
  
# using an api gateway
## API Gateway 란 
> 다 중요해서 복사 후 붙여넣었다
- API Gateway is a server that is the single entry point into the system
- It is similar to the Facade pattern from object‑oriented design.
- The API Gateway encapsulates the internal system architecture and provides an API that is tailored to each client.
- It might have other responsibilities such as authentication, monitoring, load balancing, caching, request shaping and management, and static response handling.
- The API Gateway is responsible for request routing, composition, and protocol translation.
- All requests from clients first go through the API Gateway.
- It then routes requests to the appropriate microservice. 
- The API Gateway will often handle a request by invoking multiple microservices and aggregating the results.
- It can translate between web protocols such as HTTP and WebSocket and web‑unfriendly protocols that are used internally.

## netflix api gateway 
> `device-specific adapter code` 를 제공하고, 평균 6-7개 `backend services` 를 호출, 하루에 10억 request 정도를 처리
- Today, they use an API Gateway that provides an API tailored for each device by running device‑specific adapter code.
- An adapter typically handles each request by invoking on average six to seven backend services.
- The Netflix API Gateway handles billions of requests per day.

# Benefits and Drawbacks of an API Gateway
## Benefit
- encapsulates the internal structure of the application 
- simplifies the client code

## Drawabacks
- API gateway 가 개발사이클의 병목지점이 될 수도 있다
  -  It is important that the process for updating the API Gateway be as lightweight as possible
  
# Implementing an API Gateway
## Performance and Scalability
- 10억 request 를 받는 기업은 많지 않겠지만, 대부분의 어플리케이션에게 api gateway 의 performance, scalability 는 중요하다

## Using a Reactive Programming Model
- It handles other requests by invoking multiple backend services and aggregating the results
- In order to minimize response time, the API Gateway should perform independent requests concurrently
- The API Gateway might first need to validate the request by calling an authentication service, before routing the request to a backend service
- 전통적인 비동기 콜백 방식으로 개발할 수 있지만, `callback hell` 이 되기 쉽다.
  - A much better approach is to write API Gateway code in a declarative style using a `reactive approach`.

## Service Invocation
- API Gateway will need to support a variety of communication mechanisms
- `IPC(inter-process communication)`에는 `asynchronous`, `messaging-based` 이용하는 방식과 `synchronous(HTTP, Thrift)` 방식이 있다.
- 보통 둘다 쓰고, `api gateway` 는 다양한 `communication mechanisms` 를 지원해야된다.

## Service Discovery
- traditional application 에서는 endpoint 를 hardwire 해도 됐음
- `microservice` 의 endpoint는 `dynamically assigned locations` 를 가지고, autoscaling, upgrades 영향으로 계속 변함
- `server-side discovery` 나 `client-side discovery` 가 필요함

## Handling Partial Failures
- service 가 다른 service 호출하는데 답변이 늦거나 response 받을 수 없는 경우에 발생 
- 계속해서 downstream service 가 결과를 줄거라고 기다리면 안된다
- 결과를 받지 못하면, 나머지 정보라도 사용자에게 유용하다고 판단되면 결과를 줘야된다.
- 미리 만들어놓은 response 로 대체할 수도 있다.
- 캐시 데이터를 대신 내려줄 수도 있다.
- `Hystrix` 같은 `circuit-breaker tool` 을 이용해서, failure 상태일때 fallback action 을 하게 하는 방법도 있다.


# Summary
- 시스템으로의 단일 진입점인 API gateway 를 만드는건 의미가 있다
- gateway 는 `request routing`, `composition`, and `protocol translation` 의 기능을 한다
- mask failures in the backend services by `returning cached or default data(fallback caching)`
