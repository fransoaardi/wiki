# intro
- `NGINX` blog 에서 읽은 `Microservices: From Design to Deployment` 정리
> by Chris Richardson
- 7부로 되어있음

# 5부, `Choosing a Microservices Deployment Strategy`
> https://www.nginx.com/blog/deploying-microservices/


# Motivations
- monolithic application deploy 는 여러 동일한 application 을 실행하는것. 
- monolithic application 을 deploy 하는게 microservices application 을 deploy 하는것 보다 훨씬 단순함.
- 각각의 service 에 적절한 CPU, memory, I/O resources 를 제공해야됨
- 복잡하더라도 빠르고, 정확하고, cost-effective 하게 deploy 해야됨 

# Multiple Service Instances per Host Pattern

- 하나 혹은 하나 이상의 물리/가상 호스트를 띄우고, 여러개의 service 인스턴스를 각 호스트에 띄운다. 
- 각 서비스들은 하나 혹은 하나 이상의 호스트에서 well known port 에 뜬다. 
- host machines 는 treated like pets.(msa 의 서버들과는 다르게 소중하게 다뤄짐)

- 각각의 서비스 인스턴스가 process 그룹을 이루는것 (parent process 와 여러개의 child processes)
- 같은 process/process group 에 여러 service instances 를 deploy 하는것 ( Tomcat 에 여러개의 Java web application 을 띄우고, OSGI bundle 여러개를 띄우는것)

- 장점: 
  - resource usage 효율적 (여러 인스턴스들이 server 와 os 를 공유함)
    - 여러 웹 어플리케이션이 톰캣과 JVM 을 공유함 
  - 배포가 상대적으로 빠름
    - service 를 host 에 복사해서 실행하면됨 
    - JAVA 였으면 JAR, WAR 복사하면 됨 

  - overhead 가 없어서 service 실행이 빠름 
    - 각각의 process 가 있어서 단순히 실행하면됨

- 단점:
  - service sinstancs 간 isolation 이 없음 
    - 잘못된 service 가 host 의 모든 CPU 혹은 memory 를 다 잡아먹을 수 있음.
    - 같은 JVM 상에서 heap 을 다 사용할 수 있음 
    - service instance 별로 resource 사용량을 모니터링 할 수 없음

  - 운영팀이 service deploy 하는 것을 세세하게 알아야됨 
  - 여러 언어/framework 가 실행될 수 있고 여러 세세한 것들을 개발팀이 운영팀과 나눠야됨 
    - 복잡도가 올라가서 문제가생길 소지가 많다.

## Service Instance per Virtual Machine Pattern 

- host 로부터 VM 을 만들어서 instance 별로 격리함
- Amazon EC2 AMI 등 VM image 를 만듬. 
- 만들어진 이미지로 각각의 서비스 인스턴스를 띄움 
- Netflix 가 video streaming service 를 deploy 할때 사용했던 방식, Netflix 는 각각의 서비스를 Animator 를 이용해서 EC2 AMI 로 묶었고, 이를 각각의 EC2 에 띄웠음.

- CI 서버에서 service 를 EC2 AMI 로 package 하도록 할 수 있고, Packer.io 를 쓸 수도 있다.
- Boxfuse 는 Java application 을 작은 VM image 로 묶고, CloudNative 는 Bakery 라는 EC2 AMI 만드는 SaaS 를 제공하고, 이용하면 편하다. 

- 장점:
  - 고정된 CPU, memory 를 갖고, 다른 서비스로부터 뺏어올 수 없다. 
  - AWS 등을 이용하면 loadbalancing, autoscaling 사용할 수 있다.
  - service 를 encapsulate 한다 

- 단점:
  - 비효율적인 resource 사용 
    -  IaaS 가 고정된 VM 을 제공해서, resource 가 덜 사용될 가능성이 있음 
  - busy/idle 상관없이 돈이 나감 
  - autoscaling 을 제공하지만 빠르게 제공하진 않음 > 그동안 돈이 많이 나감 
  - VM image 빌드가 오래걸려서, 새로운 버전 deploy 가 느림
  - business 말고도 다양한 부분에 대한 책임이 생김(인프라적인것, VM 제작/관리 등..)


## Service Instance per Container Pattern 

- host 로부터 container 를 만들어서 instance 별로 격리함
- 컨테이너의 memory, CPU 를 제한할 수 있음, I/O rate limiting 이 가능한 경우도 있음 
- 예시: Docker, Solaris Zones 

- Java service 를 deploy 하려면 Java runtime(톰캣), compiled java application 을 가진 image 를 빌드해야됨
- 보통 여러 컨테이너를 물리/가상 host 에 실행함 
- Kubernetes/ Marathon 과 같은 cluster manager 를 사용할 수 있음 
- cluster manager 는 host 들을 resource pool 로 다루고, 각각의 컨테이너에서 필요한 resource 를 가용할 수 있는 각각의 host 에 할당

- 장점: 
  - Service Instance per VM pattern 과 비슷 
  - VM 과는 다르게 container 는 가볍다. 
  - image 빌드하는데 오래 안걸림, 시간이 오래걸리는 OS boot mechanism 이 없어서 빠르게 실행된다. 

- 단점:
  - VM 보다 안전하지 않음 (host OS 의 커널을 공유하기때문에)
  - 컨테이너 이미지 관련 business 말고도 다양한 부분에 대한 책임이 생김 
  - load spike 칠때 overprovisioning 이 발생해서 돈이 많이 나갈 수 있음 

# Serverless Deployment

- deploy 하기 위해서는 zip 파일로 묶어서 AWS Lambda 에 upload 한다. 
- 이벤트 이름, 함수의 이름 등을 갖는 metadata 를 제공할 수 있다. 
- AWS Lambda 는 request 를 처리하기 위해 충분한 인스턴스를 자동으로 실행한다.
- 각각의 request 의 처리시간/ 메모리사용량에 따라 간단하게 과금된다. 
- 물론 AWS Lambda 는 제약이 있지만, 서버/가상머신/컨테이너를 신경쓰지 않아도 되는것이 좋다.

- Lambda function 은 stateless service 이고, 보통 AWS service 를 호출해서 요청을 처리한다.
  - S3 bucket 에 이미지가 업로드되면, lambda function 이 동작하고, DynamoDB 이미지 테이블에 이미지를 입력하고, Kinesis stream 으로 메세지를 publish 해서 이미지 처리를 trigger 한다. 
  - third-party 웹 서비스를 호출할 수도 있다. 

- Lambda function 을 호출하는 4가지 방법
1. web service request 로 직접호출
2. S3, DynamoDB, Kinesis, Simple Email Service 등 AWS service 의 event 로 자동으로 동작
3. client 로 부터 AWS API Gateway 를 통해서 자동으로 
4. cron 등 일정한 스케쥴로 

- Lambda 의 제한 
  - 300 초 안에 request 가 끝나야됨 (message consume 안됨)
  - service 는 stateless 해야됨 (이론적으로, request 마다 다른 instance 에서 실행될거라서)
  - 제공되는 언어로만 개발되야됨
  - 빠르게 실행될 수 있어야 됨 (아니면 timed out, terminated 발생)

# Summary 
- microservices application deploy 는 힘들다. 
  - 다양한 언어/프레임워크로 만들어진 서비스들이 많다
  - 각각은 배포, 리소스, 스케일링, 모니터링 요구사항을 갖는 작은 app 이다. 
  - Service Instance per Virtual Machine, Service Instance per Container 과 같이 다양한 패턴이 있다.
  - serverless 방식인 AWS Lambda 를 이용하는 매력적인 방법도 있다. 


