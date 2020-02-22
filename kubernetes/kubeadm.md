admin : cpu 2개 이상.. 
node : 모름 


- ansible-playbook  setup --tag common,docker 진행함.. 

- kubeadm 설치 진행 (아래 링크 참조)

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum-config-manager --add-repo https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64


sudo yum install -y kubelet kubeadm kubectl

하고나면, gpg check 안되서 에러난다. 그 이후에 아래 부분을 추가해준다.. (왜 그래야될까?)

> `packages.cloud.google.com_yum_repos_kubernetes-el7-x86_64.repo` file 에 아래 내용 append
```
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```
위 설정하고 다시 설치한다. 
sudo yum install -y kubelet kubeadm kubectl

다 설치한 이후 yum repo 에서 빼준다. 
sudo yum-config-manager --disable packages.cloud.google.com_yum_repos_kubernetes-el7-x86_64

sudo systemctl enable --now kubelet

---

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system



sudo systemctl daemon-reload
sudo systemctl restart kubelet

---- 설치됐다고 가정 

sudo kubeadm init


결과가 이렇게 나옴. .
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 123.123.123.1:6443 --token tebnik.umyhdp621ej8cuv3 \
    --discovery-token-ca-cert-hash sha256:8555ee3f9b9892ea34dffdd7cbb67e6ad22bb55c6e414670d4653744e5ce5d7d
```

얘기한대로 아래와 같이 실행함. -> kubectl 이 non root user 에게 동작하게 하기 위함. 
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

kubeadm token list 해보면 위에 발급된 토큰값을 확인은 가능한데, 
 discovery-token-ca-cert-hash 에 해당하는 sha256:<hash> 값은 확인할수가 없다.. 잘 적어놓도록 한다. 

cluster 의 pod network 설치해야되는데, calico 로 설치하였다. 

 kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml



 
--- 
노드도 kubeadm 설치하고, 
위에 sudo kubeadm join  ... 실행한다. 

--- 
[admin]$ kubectl get nodes -o wide
NAME                                               STATUS   ROLES    AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION                  CONTAINER-RUNTIME
ip-123-123-123-1.ap-northeast-2.compute.internal   Ready    master   114m   v1.17.3   234.234.234.1   <none>        Amazon Linux 2   4.14.165-131.185.amzn2.x86_64   docker://18.9.9
ip-123-123-123-2.ap-northeast-2.compute.internal    Ready    <none>   13m    v1.17.3  234.234.234.2    <none>        Amazon Linux 2   4.14.165-131.185.amzn2.x86_64   docker://18.9.9


$ kubectl run kubia --image=luksa/kubia --port 8080 --generator=run-pod/v1

$ kubectl expose rc kubia --type=LoadBalancer --name kubia-http

kubectl patch svc kubia-http -p '{"spec": {"type": "LoadBalancer", "externalIPs":["234.234.234.1"]}}'

---

# rhel8 에 docker , kubeadm, kubectl, kubelet 설치 

#docker install 
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf list docker-ce
dnf install docker-ce --nobest -y

#docker restart
systemctl start docker
systemctl enable docker

# kubectl/kubeadm/kubelet install

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

#kubeadm init (master)
sudo kubeadm init

#kubeadm network setting  (using calico)
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml

# kubeadm join (worker)
sudo kubeadm join 
