## trouble shooting

- unique constarint 를 제거하고 싶다

> unique constraint 는 결국 index 이므로, 해당 테이블의 index 정보를 확인한다

```mysql
show index from TABLE_NAME;
```

> index 를 drop 한다

```mysql
alter table TABLE_NAME drop index INDEX_NAME;
```
