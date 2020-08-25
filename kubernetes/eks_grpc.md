# introduction 
- gRPC 를 사용하는 app 을 aws EKS 를 사용하여 deploy, gRPC 를 이용한 API call 을 성공하는 과정을 다룬다. 
- EKS 와 함께 사용하는 fargate 를 이용하진 않았다. 

# important notes
- gRPC 연결에 있어, tls termination 부분이 주요하게 작용함
- 

# architecture
| source | destination | comment |
| --- | --- | --- |
| caller | LB(load balancer) | aws NLB (network load balancer) |
| LB | EKS cluster nodes | 논리적 요소 | 
| EKS cluster | cluster nodes | cluster nodes 는 EC2 instance 로 구성 |
| cluster nodes | ingress | nginx-ingress 사용 |
| ingress | service | |
| service | deployment | service 는 label selector 로 pods 연결 |

```
$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
ip-10-0-0-54.ap-northeast-2.compute.internal   Ready    <none>   5d17h   v1.17.9-eks-4c6976
ip-10-0-0-83.ap-northeast-2.compute.internal   Ready    <none>   5d17h   v1.17.9-eks-4c6976
```
