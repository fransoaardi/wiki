# introduction
- 주의: 아래 메모는 EKS 에 istio 설정을 하는 실패의 기록을 담습니다. 
- 아직 미완의 글입니다.

aws eks --region ap-northeast-2 update-kubeconfig --name ht-k8s

kubectl config set-context --current --namespace=dev

# istio 설치 at k8s
$ istioctl install --set profile=demo

# kubectl namespace dev 에다가 istio injection enable  함 

kubectl apply -f dev-namespace.yaml 
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager
```

# istio injection enable
$ kubectl label namespace dev istio-injection=enabled

helm repo add jetstack https://charts.jetstack.io

helm repo update

> installCRDs 를 설정 안했더니, Certificate resource 를 찾지 못해서 certmanager 를 지웠다가 새로 설치함 
$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.0.2 \
  --set installCRDs=true

$ helm --namespace cert-manager delete cert-manager

> certmanager create 하면 아래와 같이 에러가 나서 아래 링크를 보고 실행함. (어떻게 동작하는진 모르겠음)
```
$ kubectl apply -f cert-dev.yaml
Error from server (InternalError): error when creating "cert-dev.yaml": Internal error occurred: failed calling webhook "webhook.cert-manager.io": Post https://cert-manager-webhook.cert-manager.svc:443/mutate?timeout=10s: x509: certificate signed by unknown authority
```
- https://github.com/jetstack/cert-manager/issues/2602
```
$ kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook
$ kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io cert-manager-webhook
```

> kiali install (error 나면 한번 더 실행한다.)
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/kiali.yaml
> prometheus install 
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/prometheus.yaml
> run kiali dashboard
$ istioctl dashboard kiali


helm install istio-init \
--wait \
--namespace istio-system \
install/kubernetes/helm/istio-init

