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


# issues
## tls 관련
- dns 적용해서 호출하고 싶은데, tls 세팅을 해놓지 않으면 호출이 불가능한 이슈가 생김 
- 

### NLB tls termination 적용 
- aws NLB 에서 tls termination 을 지원함 
- 인증서를 NLB 에 적용하고 client <-> NLB 구간 tls 적용
  - 장점: aws 통해서 인증서 관리 가능, NLB 이후 구간 인증서 없이 적용 가능 
  - 문제점: gRPC 전달이 안됐음 
  - 따라서, NLB 에 tls 적용하는건 제외하고 ingress 이후 구간 tls 암호화를 적용하도록함 (단, NLB 가 :443 listen 하여 forward 함 )

### certmanager 를 이용한 인증서 발급
#### references
- k8s 에 cert-manager 설치: https://cert-manager.io/docs/installation/kubernetes/
- cert-manager 로 인증서 발급받고, ingress 에 설정: https://cert-manager.io/docs/tutorials/acme/ingress/

> cert 관련 yaml
```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: mail@to.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
      - http01:
          ingress:
            class:  nginx
```

> ingress yaml 최종
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ht-gateway-ingress
  namespace: dev
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - hostname.abc.com
    secretName: secret-name-you-want

  rules:
  - host: hostname.abc.com
    http:
      paths:
      - backend:
          serviceName: service-to-call
          servicePort: grpc
```
 
 
# howto call from client
- tls 적용이 되어있기 때문에, client 에서 호출할때도 insecure 로 하면 안된다.
- 이전코드:
```

```
- 이후코드:
```

```


- 
