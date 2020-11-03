# introduction
- `kubectl` 이용 관련 시행착오를 기록함

# table of contents
1. `kubectl config` 설정
2. 미사용 `kube config` 삭제

## 1. `kubectl config` 
- `kubectl get svc` 했을때 멈추는 증상 발견 
- 이전에 설정 잡아놓은 kubernetes cluster 를 바라보고있어서 timeout 이 나고있는거였음

> 이전에 gke 에 만들었던 kube cluster 을 지우고 다시 접근하려니 당연히 안됨
```bash
$ kubectl config current-context
gke_edith-270602_asia-northeast3-c_istio-sample
```

> 이번에는 aws 에 새로 만든 eks 에 접근하려는데, awscli 에서 지원하는 내용이 있어서 아래 문서 참고함
- https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-cluster-connection/
```bash
$ aws eks --region ${region} update-kubeconfig --name ${cluster_name}
```

- 결과:
```
$ kubectl get svc

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   17h
```

## 2. 미사용 `kubectl config` 정리
- cluster 더이상 사용하지 않게 되어 삭제가 필요함
- `~/.kube/config` 에 `contexts`, `clusters`, `users` 필드가 나뉘어있음. 

> https://stackoverflow.com/questions/37016546/how-do-i-delete-clusters-and-contexts-from-kubectl-config
- 직접 kube config 구성요소 하나하나 삭제하는 방법
```bash
$ kubectl config unset users.gke_project_zone_name
$ kubectl config unset contexts.aws_cluster1-kubernetes
$ kubectl config unset clusters.foobar-baz
```

- 명령어를 이용한 방법(users 는 이전 방법 그대로..)
```bash
$ kubectl config delete-cluster my-cluster
$ kubectl config delete-context my-cluster-context
# users 는 따로 지우는 방법이 없다고 한다. (rework 가 예정되어있다고 한다)
$ kubectl config unset users.my-cluster-admin
```

- 이후 확인
```bash
$ kubectl config get-contexts
$ kubectl config delete-context Cluster_Name_1
```
```
