# intro 
- istio 를 공부해보기 위해, demo 를 따라해보았다. 

# environment
- GKE (nodes x 4)

# howto 
- GKE 생성
- `brew install istioctl`
- kubectl 에 gke 로 생성한 cluster 연동 
- `istioctl manifest apply --set profile=demo`
> kubernetes cluster 에 istio 를 설치함 
```
$ kubectl get crd 
# 해보면 상당히 많은 custom resource definitions 가 설치됨
```

- `kubectl label namespace default istio-injection=enabled`
> Envoy sidecar proxy 를 자동으로 inject 시키기 위해서라는데.. 잘 모르겠음

- `kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`
> istio 를 homebrew 로 설치해서 samples 를 따로 받진 않아서, github raw 링크(https://raw.githubusercontent.com/istio/istio/release-1.5/samples/bookinfo/platform/kube/bookinfo.yaml)를 직접 이용함, 이후 생략

- `kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml`
> istio gateway 실행

- `INGRESS_HOST`, `INGRESS_PORT`, `SECURE_INGRESS_PORT` 설정
```
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

$ echo $INGRESS_HOST $INGRESS_PORT $SECURE_INGRESS_PORT
a.b.c.d 80 443
```
- `GATEWAY_URL` 설정 
> export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

# references
- https://istio.io/docs/examples/bookinfo/

