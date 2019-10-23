# mysql 

## mysql setting 에 관한 정리 

- 접속 
```bash
$ sudo -i
$ cat ~/.my.cnf #비밀번호는 .my.cnf에 있다

$ mysql -h localhost -u os_admin -p
```

- spring 으로 gradle build시 

```
java.sql.SQLException: null, message from server: "Host '~~ (내 로컬 ip)' is not allowed to connect to this MySQL server"
```

- 에러메세지 발생, 테스트를 위해서 `$ nc -v ${MySQL서버} ${포트}` 해보니 같은 메세지가 발생했다.  ( mysql 접속권한문제로 진단 )

```mysql
mysql> create database db_example; -- Create the new database
mysql> create user 'springuser'@'localhost' identified by 'ThePassword'; -- Creates the user
mysql> grant all on db_example.* to 'springuser'@'localhost'; -- Gives all the privileges to the new user on the newly created database
```
> reference 1)

## ip 허용 
> reference 2)

- 모든 IP 허용

```sql
INSERT INTO mysql.user (host,user,authentication_string) VALUES ('%','root',password('패스워드를 입력한다'));
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
FLUSH PRIVILEGES;
```

- IP 대역 허용 ( 예: 111.222.xxx.xxx )

다음과 같이 설정하면 111.222로 시작하는 모든 IP가 허용된다.

```sql
INSERT INTO mysql.user (host,user,authentication_string) VALUES ('111.222.%','root',password('패스워드를 입력한다'));
GRANT ALL PRIVILEGES ON *.* TO 'root'@'111.222.%';
FLUSH PRIVILEGES;
```

- 특정 IP 1개 허용 ( 예: 111.222.33.44 )

```sql
INSERT INTO mysql.user (host,user,authentication_string) VALUES ('111.222.33.44','root',password('패스워드를 입력한다'));
GRANT ALL PRIVILEGES ON *.* TO 'root'@'111.222.33.44';
FLUSH PRIVILEGES;
```

- 원래 상태로 복구
모든 IP를 허용한 경우 다음과 같이 원래 상태로 복구할 수 있다.

```
DELETE FROM mysql.user WHERE Host='%' AND User='root';
FLUSH PRIVILEGES;
```

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


### reference
1) https://github.com/spring-guides/gs-accessing-data-mysql
2) https://zetawiki.com/wiki/MySQL_%EC%9B%90%EA%B2%A9_%EC%A0%91%EC%86%8D_%ED%97%88%EC%9A%A9
