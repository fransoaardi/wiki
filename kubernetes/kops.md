# kubernetes on AWS EC2 using kops

# how-to 
1. kops 설치 
```shell-script
$ brew update && brew install kops
```

2. kubectl 설치

3. s3 할당 
```shell-script
$ aws s3 mb s3://kube-clusters.abc.def
```
4. KOPS env 설정
```shell-script
$ export KOPS_STATE_STORE=s3://kube-clusters.abc.def
```

5. ssh-keygen (있다면 생략: kubectl secret 으로 사용)
```
$ ssh-keygen
```

6. secret 생성
```shell-script
$ kops create secret --name kube-pypi.abc.def sshpublickey admin -i ~/.ssh/id_rsa.pub
```

7. cluster create (config 생성)
```shell-script
$ kops create cluster --zones=ap-northeast-2a kube-pypi.abc.def
```

8. cluster update (실제 ec2 instance 등 생성, yes tag 없으면 test 용도)
```shell-script
$ kops update cluster kube-pypi.abc.def --yes
```

9. cluster delete (ec2 instance, vpc 등 termination)
```shell-script
$ kops delete cluster kube-pypi.abc.def --yes
```

# reference
- https://kubernetes.io/ko/docs/setup/production-environment/tools/kops/
