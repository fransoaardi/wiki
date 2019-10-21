# fluent-bit

## intro

fluent-bit를 이용해 작성중인 log 파일을 tail하며 읽고 새 파일에 write 하려고 한다.

## tutorial, reference

- official documentation
https://fluentbit.io/documentation/current/

- tutorial
https://logz.io/blog/fluent-bit-tutorial/

## download

https://hub.docker.com/r/fluent/fluent-bit/

## 실행, 설정

- 읽을 log파일 mount하며 docker run

```
docker run --name fluent -v {from_host}:{to_container} -p 127.0.0.1:24224:24224 -dit fluent/fluent-bit:latest
```

- /fluent-bit/etc/fluent-bit.conf

주의: 4 spaces indent

```
[SERVICE]
    Flush        1
    Daemon       Off
    Log_Level    info
    Log_File     /fluent-bit/log/fluent-bit.log

[INPUT]
    Name tail
    Refresh_Interval 1
    Path /fluent-bit/access.log
    Tag  tglog.*
    Buffer_Chunk_Size 1k

[OUTPUT]
    Name  file
    Match tglog.*
    Path john.log
    Retry_Limit False
```

## fluent-bit 실행

```
# 기본 실행, -c 는 config
/fluent-bit/bin/fluent-bit -c fluent-bit.conf

# sosreport 하면 적용될 conf 내용 및 환경설정된것 출력 
/fluent-bit/bin/fluent-bit -c fluent-bit.conf --sosreport
```


## Docker 연동

위의 tutorial 참고.

--log-driver=fluentd 를 추가해주면 24224 포트로 중개하고, fluent-bit input을 forward로 consume 가능

```
docker run --log-driver=fluentd --name {container name} -p 9000:9000 -dit {docker image}
```
