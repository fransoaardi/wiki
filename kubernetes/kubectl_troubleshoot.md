# introduction
- `kubectl` 이용 관련 시행착오를 기록함

# table of contents
1. `kubectl config` 설정


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
