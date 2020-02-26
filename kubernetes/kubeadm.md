# intro
- `EKS` 를 사용하지 않고, `EC2` 만을 이용한 결국 실패한 기록. 
- `kubeadm`, `kubelet`, `kubectl` 설치가 중요한게 아니라 아래에 설명을 줄였음
- `EC2` (master x 1, worker x 2) 구성을 의도함
- network 는 `calico` 이용
- network 관련 다수의 문제가 발생
  - master node 에서 pod 하나 실행해서 worker 에 띄우면, master 의 clusterIP 로 접근이 불가함 
  - `LoadBalancer` 는 pending 상태가 유지됨 
  - `ClusterIP` 인 경우, master 에서 접근 안되고, 다른 worker 에서도 접근이 안되고, pod 가 뜬 exact node 에서만 접근이 가능함

# 기록

- `admin` : (v)cpu 2개 이상 필요함 
- `worker` : 상관없었음

**AMI (amazon image) 썼다가 낭패, RHEL 8로 깔끔하게 성공하였음.**

- `kubeadm` 설치 진행 (아래 링크 참고)

> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## AMI 실패의 기록 (AMI package repository 설정문제로 고생함)
``` 
sudo yum install -y kubelet kubeadm kubectl

하고나면, gpg check 안되서 에러난다. 그 이후에 아래 부분을 추가해준다.. (AMI repo 는 gpg check 안하고, 추가한 kubernetes repo 에서 check 하는 과정에서 꼬이는것으로 추정됨)

> `packages.cloud.google.com_yum_repos_kubernetes-el7-x86_64.repo` file 에 아래 내용 append

gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

위 설정하고 다시 설치한다. 
sudo yum install -y kubelet kubeadm kubectl

다 설치한 이후 yum repo 에서 빼준다. 
sudo yum-config-manager --disable packages.cloud.google.com_yum_repos_kubernetes-el7-x86_64

sudo systemctl enable --now kubelet
```

## rhel8 에 docker , kubeadm, kubectl, kubelet 설치 

> docker install 
```
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf list docker-ce
dnf install docker-ce --nobest -y
```

> docker restart
```
systemctl start docker
systemctl enable docker
```
### kubectl/kubeadm/kubelet install

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### kubeadm 관련

`kubeadm init` 하고 나면 `/var/lib/kubelet` dir 에 각종 설정파일들이 clone 된다. 
> `/var/lib/kubelet/kubeadm-flags.env` 파일에 `--cloud--provider` 옵션 줌 
```
KUBELET_KUBEADM_ARGS="--cloud-provider=external --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1"
```

- 이후 `kubelet` 재실행
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

- `calico` 를 `network` 로 사용할때는 `--pod-network-cidr` 옵션을 줘야된다.
> sudo kubeadm init --pod-network-cidr=192.168.0.0/16

- 이후 calico 설치한다
> kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml

#### 참고: kube-apiserver, kube-controller-manager, kube-scheduler 의 설정은 어디서 오는가?
- `kubeadm init` 하는 시점에 `/etc/kubernetes/manifests` 에 `yaml` 파일이 clone 되어온다, 이 설정파일에 각종 flag option 들을 줄 수 있다.
- `kubelet` 이 실행되는 시점에 `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` 이 실행되는것으로 보인다. 
- `kubeadm-flags.env` 적용 후 `kubelet restart` 하니 `kube-apiserver`, `kube-controller-manager` 설정에 이미 적용되어 재실행된것으로 확인된다.
> `kubelet` 이 `kube-apiserver`, `kube-controller-manager` 를 실행시키는것으로 보임
```
# sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml 에 아래 option 추가
--cloud-provider=external

# sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml 에 아래 option 추가 
--cloud-provider=external
```
