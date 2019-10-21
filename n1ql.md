# couchbase N1QL 

## intro
- n1ql (nickel, 니켈, 닉클 ...) 이라고 발음 한다 
- noSQL 인 couchbase 에서 RDB 처럼 querying 을 할 수 있다.  

## how to 

`Query Editor` 에 입력하고 `Execute` 버튼을 누르면 된다.  

## example 

### case 1

쉬운 ver. (성능도 좋다)
첫번째 탭의 쿼리가 '오픈채팅' 이고, 두번째 탭의 타입이 기본탭이 아닌 커스텀탭인 유저의 숫자를 구하는 쿼리

**Note**

\`user\`.\`user\`[*].query[0] 은  \`user\` 버킷의 \`user\`필드 (array) 중 0번째 item 의 \`query\`의 값이다 
```sql
SELECT count(*)
FROM `user`
WHERE `user`.`user`[*].query[0] = '오픈채팅' 
AND `user`.`user`[*].type[0] = 'native'
AND `user`.`user`[*].type[1] <> 'native'
```

### case 2

어려운 ver. (성능도 안좋다)
첫번째 탭의 쿼리가 '오픈채팅' 이고, 두번째 탭의 타입이 기본탭이 아닌 커스텀탭인 유저의 숫자를 구하는 쿼리
```sql
SELECT count(*) 
FROM (
  SELECT FIRST q FOR q IN `user`.`user`[*].query 
         WHEN  `user`.`user`[*].query[0] = '오픈채팅' 
         AND `user`.`user`[*].type[0] = 'native'
         AND `user`.`user`[*].type[1] <> 'native' END as fq
  FROM `user`) nested
WHERE nested.fq
```

### case 3

기본탭이 아닌 커스텀탭으로 '오픈채팅' 탭을 등록한 사람의 숫자를 구하는 쿼리
```sql
SELECT count(*) from `user`
WHERE ANY u in `user`.`user` 
SATISFIES (u.query = '오픈채팅' and u.type <> 'native') END;
```

