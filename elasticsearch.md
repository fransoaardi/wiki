# API

- Elasticsearch 에서 검색기능을 제공한다.
- query 를 잘 작성하여 Kibana 사용하지 않고 로그확인이 가능하고, api 를 잘 이용하면 로그 확인용 페이지 등을 작성 가능할 것 같다.

## 참고

- https://www.elastic.co/guide/en/elasticsearch/reference/5.4/search-uri-request.html

## 예시와 
http://url_here/index_here-*/doc/_search?q=(user:*)AND(phase:cbt)AND(time%3E=%222018-09-07%22)AND(path=%22/api/path%22)

- index_here-* 
  - index 명 
- doc 
  - type 명 (없어도 되는것같다)
- _search 
  - 조회 해옴
- q
  - query 할것 
    - (user:*) : user 필드는 모든값.
    - AND : AND 조건 가능 (OR 하면 OR 가능)
    - (phase:cbt) : phase 필드는 cbt에 해당하는 값만 
    - (time>="2018-09-07") : time 값이 2018-09-07 보다 큰 값일때
