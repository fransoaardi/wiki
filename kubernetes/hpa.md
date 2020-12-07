# introduction
- hpa (Horizontal Pod Autoscaler) 적용 관련

# progress
- `kubectl get all -n istio-system` 을 수행해봤다가, hpa 가 `istiod`, `istio-ingressgateway` 에 적용되어있다는것을 우연히 발견하게 되었다.
```
$ kubectl get all -n istio-system
(...)
NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   4%/80%    1         5         1          21d
horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 0%/80%    1         5         1          21d
```

- 위의 `TARGETS` 필드에 `4%/80$` 라고 되어있지만, 처음에는 `UNKNOWN/80%` 라고 되어있었다.
    - hpa 가 적용되고 정상적으로 동작하기 위해서는, 해당 pod 의 resource 사용 상황을 알아야되고, 그러한 metric 을 알기 위해서는, metrics-server pod 가 필요하다. 
    - 보통 cpu 사용량을 이용하는데, custom resource (prometheus 를 이용해서 scrape 한)를 이용할 수도 있다.
    - mem 사용량을 이용할 수 있지만, mem 사용량은 app 구현 오류로 release 안될 수도 있고 하여, 잘 사용하지 않는다고 한다.
    > `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

    > https://github.com/kubernetes-sigs/metrics-server
```
$ kubectl get po -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
(...)
metrics-server-866b7d5b74-jjl8l   1/1     Running   0          16d
```

- `kubectl describe hpa -n istio-system` 
  - 아래와 같이 min 1개, max 5개로 설정이 되어있고, cpu 사용량에 따라 유동적으로 변경된다. 
  > 논외이지만, istio 1.5 에서 `istio-ingressgateway` 가 사용량이 늘어, pod 가 1개 초과로 늘어났을때, 동작이 이상해서 강제로 hpa target 을 1로 제한했다는 이야기를 들은적이 있다. 
```
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  4% (4m) / 80%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       1 current / 1 desired
```

- hpa 를 deployment 에 적용하고, stresstest 를 진행했을때, pod 가 늘어나는것을 확인할 수 있었다. 다만 문제점이 몇가지 있다
    - issue 1: hpa 의 기준이 될 metric 의 기준값 선정이 필요한데, 평소에 어느정도 resource 를 사용하는지에 대한 측정값을 알고있는건 상당히 어려운 일이라고 생각한다.
    - issue 2: resource 사용을 정말 정교하게 할것이 아니라면(가용할수 있는 리소스에 여유가 있는 상태), 굳이 hpa 를 적용할 필요는 없다고 생각한다. 물론 k8s 의 주요한 기능중 하나지만, 자칫하면 over engineering 이 될것이라고 생각한다. (평소 resource 사용량을 줄여 performance 를 낮추는 설정일수 있다)
    - issue 3: hpa 가 생성되어있는지 깜빡하고, deployment 를 생성했는데, 계속해서 replicas 설정을 1로 해놨음에도 2개가 생성되었다, 익숙하지 않다 보니, hpa 가 적용되었는지 잘 관리하는것도 중요해보인다.

# references
- doc 
> https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/

- apache 를 이용한 예제
> https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
