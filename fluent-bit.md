# intro 

- `fluentbit` 사용과 conf 설정 경험 정리

# why

- `filebeat` -> `elasticsearch` 방식으로 데이터를 입력하고 있었음
- 입력하는 데이터의 type 이 너무 dynamic 해서 elasticsearch 의 dynamic mapping 을 이용했을때 index conflict 가 발생하는 일이 빈번함 
- 따라서 elasticsearch 의 dynamic mapping 이 발생하지 않게, 미리 elasticsearch 의 index 를 만들어놓고, predefined index 에 적합한 데이터를 올리는 방식으로 진행하려고 함
- link(elasticsearch dynamic mapping 관련)

- filebeat 는 elasticsearch 의 predefined index 에 올리는것이 불가능함 ( filebeat 자체 index 를 매일 생성해서 저장함 )
- 따라서 아래와 같은 방법을 생각함 (2번을 시도해보진 않음)
    1) filebeat 대신 fluentbit 를 사용해봄
    2) filebeat -> logstash -> elasticsearch 와 같이, 중간에 logstash 를 두고 데이터를 입력

# 2) 대신 1)을 택한 이유
- logstash 를 추가로 둬서 single point of failure 를 만들고싶지 않았음 
- filebeat 의 동작이 불안한 부분이 있어서 (elasticsearch 의 X-Pack 관련 설정에 영향을 받음, elasticsearch 와 filebeat 의 유료 버전 여부가 동일해야됨) 완벽히 무료인 opensource 로 바꾸고 싶었음


# howto
- fluentbit 설치 진행 (docker)
fluentbit docker run command 를 보니 config 파일 1개 설정 필요함
```
$ docker inspect fluentbit -f "{{ .Config.Cmd }}"
[/fluent-bit/bin/fluent-bit -c /fluent-bit/etc/fluent-bit.conf]
```

- fluentbit 설정 관련
1) `[SERVICE]` 는 fluentbit daemon 의 기본 설정
2) `[INPUT]` 중에 보통 docker 를 사용해서 docker container 의 log 를 streaming 하는 방식으로 많이 사용하는 것 같은데, 물리 log 를 남기고 tailing 하는식으로 구현함
3) parser 를 적용하기 위해서는 `[FILTER]` 의 parser 를 적용해야됨, parser 는 별도의 conf 를 설정하고, [SERVICE] 의 parserconf 경로 설정이 필요함
4) es 의 버그로, @timestamp 가 임의로 추가되어, record_modifier 로 삭제를 하려고 해도 되지 않았다. (아직 고쳐지지 않은 issue)
  - https://github.com/fluent/fluent-bit/issues/628
5) `[OUTPUT]` 은 es 로 하고, 다른 부분은 특이사항이 없다.
  - type 은 _doc 으로 했다 (elasticsearch 의 type 은 이전버전(6)까지 사용했던 개념인것 같은데, index 생성 후, 저장된 document 를 보면 type 에 _doc 가 세팅되어있는것을 확인할 수 있었어서, _doc 으로 입력하였다 )
  - X-Pack 혹은 aws 의 elasticsearch 사용시 필요한 Authorization 관련 설정을 추가로 할 수 있지만, 필요하지 않아 설정하지 않았다. 

- fluentbit.conf
```
[SERVICE]
    Flush           5
    Daemon          Off
    Log_Level       info
    HTTP_Server     Off
    Parsers_File    /fluent-bit/etc/fluent-bit-parsers.conf

[INPUT]
    Name            tail
    Tag             custom_tag_name
    Path            /path/to/log.log

[FILTER]
    Name            parser
    Parser          json_parser
    Match           *
    Key_Name        log
    # Input.Name 이 tail 일때 "log" 라는 필드로 모든 데이터가 뭉쳐서 온다, 따라서 Key_Name 지정이 필요하다

# log 에 "phase" field 를 추가한다
[FILTER]
    Name            record_modifier
    Match           *
    Record          key_to_append {{ value_using_j2_template }}

# elasticsearch 에 올릴때 @timestamp 필드가 duplicate 되지만, 해결이 되지 않는다.
#[FILTER]
#    Name            record_modifier
#    Match           *
#    Remove_key      @timestamp

[OUTPUT]
    Name            es
    Match           *
    Host            hostname.elasticsearch
    Port            9200
    Index           elasticsearch-index-name
    Type            _doc
    Retry_Limit     False
```

# references
- https://docs.fluentbit.io/manual

## fluent-bit tutorial 

이전에, fluent-bit 관련 tutorial 개념으로 테스팅 진행해봤던 기록

### intro

fluent-bit를 이용해 작성중인 log 파일을 tail하며 읽고 새 파일에 write 하려고 한다.

### tutorial, reference

- official documentation
https://fluentbit.io/documentation/current/

- tutorial
https://logz.io/blog/fluent-bit-tutorial/

### download

https://hub.docker.com/r/fluent/fluent-bit/

### 실행, 설정

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

### fluent-bit 실행
```
# 기본 실행, -c 는 config
/fluent-bit/bin/fluent-bit -c fluent-bit.conf

# sosreport 하면 적용될 conf 내용 및 환경설정된것 출력 
/fluent-bit/bin/fluent-bit -c fluent-bit.conf --sosreport
```

### Docker 연동

위의 tutorial 참고.

--log-driver=fluentd 를 추가해주면 24224 포트로 중개하고, fluent-bit input을 forward로 consume 가능

```
docker run --log-driver=fluentd --name {container name} -p 9000:9000 -dit {docker image}
```
