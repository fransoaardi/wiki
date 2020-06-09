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
  


