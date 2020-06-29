# intro

- kafka 를 사용하기 위한 initial setting 관련
- 큰 내용은 없고 커맨드 위주이다.

# howto

- aws msk 를 이용, kafka cluster 를 생성하였다.
- 아래와 같이 awscli 를 이용, zookeper string 을 얻어내야된다. 
```
aws kafka describe-cluster --region ap-northeast-2 --cluster-arn "arn:aws~~~"
```
- 별도의 vpc 생성, subnet 생성 과정이 있었는데, 복잡했다.  (생략)
  - vpc 에 internet gateway 설정 하고, routing table 수정해주지 않으면 외부에서 ec2 에 connect 가 불가능했다. 
  - vpc 에 external internet gateway 설정을 했는데, 정확히는 확인하지 않았다. (ec2 -> internet 으로 외부로 나가는 gw 라고 예상한다)
  
- java, kafka 를 설치했다. 
- 위에서 얻은 zookeeper-connect-string 을 이용, kafka cluster 와 동일한 vpc 에 있는 ec2 instance 에서 아래와 같이 topic 생성을 한다.
```
bin/kafka-topics.sh --create \
--zookeeper ${zookeeperConnectString} \
--replication-factor 3 --partitions 1 --topic message-create
```

- 아래와 같이 timeout error 발생하며 topic create 가 실패했다
```
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
[2020-06-09 04:09:46,023] WARN Client session timed out, have not heard from server in 10023ms for sessionid 0x0 (org.apache.zookeeper.ClientCnxn)
[2020-06-09 04:09:56,140] WARN Client session timed out, have not heard from server in 10014ms for sessionid 0x0 (org.apache.zookeeper.ClientCnxn)
[2020-06-09 04:10:06,241] WARN Client session timed out, have not heard from server in 10000ms for sessionid 0x0 (org.apache.zookeeper.ClientCnxn)
Exception in thread "main" kafka.zookeeper.ZooKeeperClientTimeoutException: Timed out waiting for connection while in state: CONNECTING
	at kafka.zookeeper.ZooKeeperClient.$anonfun$waitUntilConnected$3(ZooKeeperClient.scala:242)
	at scala.runtime.java8.JFunction0$mcV$sp.apply(JFunction0$mcV$sp.java:23)
	at kafka.utils.CoreUtils$.inLock(CoreUtils.scala:251)
	at kafka.zookeeper.ZooKeeperClient.waitUntilConnected(ZooKeeperClient.scala:238)
	at kafka.zookeeper.ZooKeeperClient.<init>(ZooKeeperClient.scala:96)
	at kafka.zk.KafkaZkClient$.apply(KafkaZkClient.scala:1825)
	at kafka.admin.TopicCommand$ZookeeperTopicService$.apply(TopicCommand.scala:262)
	at kafka.admin.TopicCommand$.main(TopicCommand.scala:53)
	at kafka.admin.TopicCommand.main(TopicCommand.scala)
```

- kafka subnet 의 default-security-group 의 inbound 규칙에 client 의 security-group id 로 부터 오는 모든 traffic 을 추가해서 해결

- kafka consumer 를 이용한 topic 에 데이터가 잘 들어갔는지 확인 
```
./kafka-console-consumer.sh \
--bootstrap-server "${serverString}" \
--topic message-create \
--group message-create-group \
--from-beginning
```

- kafka consumer group 목록 확인
```
./kafka-consumer-groups.sh \
--bootstrap-server "${serverString}" \
--list
```

- kafka cluster 정보 확인 (awscli)
```
aws kafka describe-cluster --region ap-northeast-2 --cluster-arn "${arnString}"
```

- kafka topic partition 갯수 변경 
```
./kafka-topics.sh --bootstrap-server "${serverString}" --alter --topic translate-request --partitions 5
```
