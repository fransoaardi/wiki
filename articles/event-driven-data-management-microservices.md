# intro
- `NGINX` blog 에서 읽은 `Microservices: From Design to Deployment` 정리
> by Chris Richardson
- 7부로 되어있음

# 5부, Event-Driven Data Management for Microservices
> https://www.nginx.com/blog/event-driven-data-management-microservices/

# Microservices and the Problem of Distributed Data Management
- monolithic app 은 전형적으로 단일 RDB 를 가진다.
- 따라서 ACID 트랜잭션을 사용할 수 있다.
```
Atomicity – Changes are made atomically
Consistency – The state of the database is always consistent
Isolation – Even though transactions are executed concurrently it appears they are executed serially
Durability – Once a transaction has committed it is not undone
```

그 결과, application 은 단순히 transaction 시작하고, 변경을 가하고, 다시 transaction 을 commit 하면 된다.

- RDB 를 사용하니 표현이 풍부하고, 명시적이고, 표준화된 쿼리 언어인 SQL을 사용할 수 있다. 
- 다양한 테이블의 데이터를 조합하는 쿼리를 쉽게 할 수 있다.
- RDBMS query planner 는 가장 최적의 방식을 결정해준다.
- DB 접근 방식에 대한 low-level details 는 신경쓸 필요가 없다. 

- microservices architecture 에서는 데이터가 복잡해졌다.
- microservice 가 갖고있는 데이터는 private 하고, API 를 통해서만 접근가능하다.
- data encapsulation 은 coupling 을 느슨하게 했고, 독립적으로 진화할 수 있도록 했다.
- 여러개의 서비스가 동일한 데이터에 접근한다면 오랜 시간이 걸릴것임.


- 서로 다른 microservices 는 다른 종류의 DB 를 사용하기도 한다.
- 항상 RDB 가 최적의 선택이 아니다
- 어떤 상황에서는 NoSQL DB 가 사용하기 더 쉽고, 성능과 scalability 측면에서 더 낫다.

- 예를들면, text search 를 하는데 ES 를 쓰거나, 사회연결망을 저장하는 Neo4j 와 같은 그래프DB 를 사용하는게 좋다. 
- microservices 기반 app 은 SQL 과 NoSQL DB 를 혼합해서 사용하고, 이것은 `polyglot persistence approach` 라고 한다.

- 분리된, `polyglot-persistent architecture` 데이터 저장은 장점이 많다.
- loosely coupled services
- better performance and scalability.

- 그러나, 분산된 데이터를 관리의 어려움이 있다. 
1. 다양한 서비스들 간에 일관성을 유지하는 business transsaction 구현의 어려움 
  - online B2B store 의 예시.
    - 구매 하기 전에 고객서비스의 신용승인내역 한도 초과했는지 확인을 하고싶은데, monolithic 이었으면 transaction 생성하고 확인 다 하고 주문하고 transaction 닫으면 됐지만, microservices 끼리는 데이터 직접접근이 막혀있음.
    - 구매서비스는 two-phase commit(2PC) 라고 불리는 분산트랜잭션을 사용할 수 있지만, 보통 유효하지 않다.
    - CAP theorem 에 따르면 availability 와 ACID-style consistency 중에 선택하기를 강요하지만, 보통 availability 를 선택한다. 
    - 대부분의 NoSQL db 는 2PC 를 지원하지 않는다. 
> data consistency 는 중요하므로 이를 해결하기 위한 다른 방식이 필요하다. 

2. 다양한 서비스들로 부터 데이터를 가져오는 query 구현의 어려움.
  - 고객과, 그의 최근 구매내역을 보여줘야되는 application 의 예시.
    - 구매서비스가 고객의 데이터를 제공한다면 application-side join 을 하면됨. 
    - 그러나, 구매서비스가 primary key 를 통한 데이터 조회만 가능하다면, 다른 명시적인 방법이 없음.

# Event‑Driven Architecture

- 해결책은 `event-driven architecture` 을 사용하는것이다.
- 주목할만한 이벤트가 생기면 microservices 는 event 를 publish 한다. 
- 다른 microservices 는 event 를 subscribe 하고있다가 event 를 받으면 자신의 business entity 를 갱신한다.
- event 를 multiple services 의 business transaction 구현에 이용할 수 있다.
- 여러 스텝의 transaction 으로 구성되고, 각각의 스텝은 business entity 를 갱신하고, 다음 스텝이 일어나게 하는 event 를 publish 한다. 
예시)
1. 구매서비스는 NEW status 의 구매를 생성하고, `Order Created` event 를 publish 한다.
2. 고객 서비스는 `Order Created` event 를 consume, 구매를 위한 credit 을 예약해두고, `Credit Reserved` event 를 publish gㅏㄴ다. 
3. 구매서비스가 `Credit Reserved` event 를 consume, status 를 NEW 에서 OPEN 으로 변경한다. 

(a) 각 서비스들은 db 를 atomically 갱신하고, event 를 publish 함 
(b) Message Broker 는 event 가 적어도 한번은 전달되도록 보장해서, 여러 서비스에 전달되는 business transaction 을 구현할 수 있게함. 
> 이런 방식은 ACID transaction 은 아니다. 훨씬 약한 보장인 `eventual consistency` 와 같은 방식이고, 이런 모델은 `BASE model` 이라고 불린다. 

- 여러 microservices 의 데이터들을 pre-join 한 view 를 유지하는데 event 를 사용할 수 있다.
- view 를 관리하는 service 는 고나련된 event 를 subscribe 하고, view 를 갱신한다.
- 예를들면, 고객-주문 view 갱신 서비스는 고객서비스, 주문서비스가 publish 하는 event 를 subscribe 한다. 

- Customer Order View 가 있고, 이를 갱신하는 Customer Order View Updater 가 있음, 
- Order Service, Customer Service 가 publish 하는 event 를 customer order view updater 가 subscribe 하면서 view 데이터 갱신.
- Customer Order View Query service 는 Customer Order View 만 보고 query 함

Event-driven architecture
장점:
- eventual consistency 를 제공
- view 를 유지할 수 있음 

단점:
- ACID transaction 이용할때보다 복잡함 
- application-level failure 발생으로부터 회복하는 보상 transaction 구현이 필요함 (신용확인 실패하면 구매를 취소해야됨)
- incosistent data 인 경우 처리 필요.
- subscriber 가 중복된 event 를 감지하고 무시해야됨

# Achieving Atomicity
- atomically updating the database + publish event 문제 
- 위 두 operation 이 atomically 실행되어야되는데, db 업데이트는 했는데 event publish 하지 못했다면 inconsistent 해짐.

## Publishing Events Using Local Transactions

- `multi‑step process involving only local transactions` 방법으로 event publish 
- message queue 처럼 동작하는 EVENT table 에 business entity 의 상태를 저장함.
- local database transaction 으로 시작, EVENT 테이블에 event 저장, transaction commit 함. 
- 별개의 application thread 는 EVENT 테이블을 읽어서 message broker 에 event publish 함. 
- local transaction 을 이용해서 event 가 publish 됐다고 표시함. 

장점:
  - 2PC 없이 event publish 를 보장함
  - application 이 business-level event 를 publish 함 

단점:
  - 개발자가 event publish 하는것을 기억해야되므로, error-prone 함 
  - local transaction 이 필요한데 NoSQL 은 지원하지 않는 경우가 있어, 구현못할 수 있음.

## Mining a Database Transaction Log
- thread 가 database transaction/commit log 를 mining 해서 event publish 하는 방법
- DB transaction log miner thread 가 log 를 읽고 message broker 에 event 발행함 
- e.g. LinkedIn Databus 
  - oracle transaction log 를 mine 하고, 변경에 대응하는 event 를 publish 
  - LinkedIn 은 Databus 를 이용해서 data store consistency 를 유지한다. 
- AWS DynamoDB 의 streams mechanism
  - 최근 24시간동안 발생한 시간순의 변경을 갖고있다.
  - application 은 그 변경점을 stream 에서 읽어서, event publish 한다.

장점:
  - 2PC 없이 event publish 보장.
  - event publishing 과 application business logic 을 분리하여 단순화함 

단점:
  - transaction log 는 db 고유의 것이고, db version 에 따라 변경될 수 있다.
  - transaction log 에 저장된 low-level update 를 high-level business event 로 변경하는 reverse engineering 은 힘들다.

## Using Event Sourcing
- `Event sourcing` 은 2PC 없이, 전혀 다른 `event-centric` 접근으로 business entity 를 보존한다.
- entity의 현재 상태를 저장하는게 아니라, event 의 상태변화를 저장한다.
- application 은 event 를 재실행해서 entity 의 현재상태를 재건한다. 
- business entity 상태 변화가 생기면, 새로운 이벤트가 이벤트 리스트에 추가된다. 
- event 를 저장하는건 단일 operation 이라, atomic 하다.

- Event Store 에 Event 는 영구저장된다. 
- event store 는 message broker 처럼 동작함 
- store 는 entity event 를 저장, 반환하는 API 가 있다. 
- services 가 event subscribe 하도록 하는 API 도 제공한다. 
- Event Store 는 event-driven microservices architecture 의 중추역할을 한다.

장점:
- 상태가 바뀔때마다 신뢰할수 있게 event publish 를 가능하게함. 
  - data consistency issue 를 해결함 
- domain object 가 아닌 event 를 영구저장해서, `object‑relational impedance mismatch problem` 가 안생김.
- business entity 에 가해진 변경에 대한 100% 신뢰 가능한 audit log 제공, 어느 상황에서든 임시적인 상황에 대한 business entity 의 상태를 확인 할 수 있음.
- event 교환을 하기때문에, business logic 의 결합이 약해짐. 

단점:
  - 익숙하지 않아서 러닝커브가 있음
  - event store 는 pk 를 이용한 ㅠusiness entity 조회만 직접적으로 제공함
  - 반드시 CQRS(Command Query Responsibility Segregation) 를 이용해야됨 
  - `eventually consistent data` 를 다뤄야됨

# Summary

- microservice architecture 에서 각각의 microservice 는 private datastore 를 가진다.
- microservices 는 서로 다른 SQL, NoSQL db 를 사용할 수 있다.
- database architecture 는 큰 장점이 있지만, 분산 데이터 관리 문제가 생긴다. 
  1. 여러 서비스들 사이에서 consistency 를 유지하는 business transaction 을 구현하는 방법
  2. 다양한 서비스들로부터 데이터를 가져오는 방법 구현 

- 대부분의 경우 event-driven architectrue 를 사용하는게 답이다. 
- 어떻게 atomically 상태를 update 하고 event를 publish 하는지가 어렵다. 
- db 를 message queue 처럼 사용하는것, transactioni log mining, event sourcing 등 다양한 방법이 있다.




