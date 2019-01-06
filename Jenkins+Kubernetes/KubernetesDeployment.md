# Kubernetes部署
## 部署环境
### master： Ubuntu Server 16.04.1 LTS 64位 2核 4GB  
### work1、work2：Ubuntu Server 16.04.1 LTS 64位 1核 1GB  
### docke-ce: 18.09
### kubernetes: v1.13.1

## Master节点配置
### 安装Docker-ce
```
$ sudo apt-get remove docker docker-engine docker.io
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
 
$ sudo docker version
```
### 安装kubelet，kubeadm，kubectl

```
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
由于google和kubernetes的官方源都无法直接获取到，这边选用了国内的aliyun镜像来安装kubelet、kubeadm和kubectl这三个工具。

### 设置网桥并重启kubelet
```
$ sudo sysctl net.bridge.bridge-nf-call-iptables=1
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```
由于网桥工作于数据链路层，在iptables没有开启 bridge-nf时,数据会直接经过网桥转发，结果就是对FORWARD的设置失效（此处其实不是很懂）
### 初始化kubeadm部署master节点
```
kubeadm init \
  --kubernetes-version=v1.13.1 \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=0.0.0.0
```
此处init kubeadm时会默认从国外的源pull镜像，但由于无法科学上网会导致pull失败。好在它会先查看本地docker镜像中有没有需要的镜像再判断是否要从国外的源pull，所以只要先从国内阿里云的镜像仓库pull下来对应的镜像，再打上相应的tag就可以解决这一问题。
### 添加api认证
```
$ sudo mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 安装flannel network add-on
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

这一步完成之后，使用命令:
```
sudo kubectl get pod --all-namespaces
sudo kubectl get nodes
```
便能看到当前master节点的pod及node信息
```
root@VM-0-10-ubuntu:~# kubectl get nodes
NAME             STATUS   ROLES    AGE     VERSION
vm-0-10-ubuntu   Ready    master   3h33m   v1.13.1
```
```
root@VM-0-10-ubuntu:/home/ubuntu# kubectl get pods -n kube-system -o wide
NAME                                     READY   STATUS    RESTARTS   AGE     IP            NODE          
coredns-86c58d9df4-m45lz                 1/1     Running   1          4h49m   10.244.0.18   vm-0-10-ubuntu
coredns-86c58d9df4-wcvcv                 1/1     Running   1          4h49m   10.244.0.17   vm-0-10-ubuntu
etcd-vm-0-10-ubuntu                      1/1     Running   1          4h48m   172.17.0.10   vm-0-10-ubuntu
kube-apiserver-vm-0-10-ubuntu            1/1     Running   2          4h48m   172.17.0.10   vm-0-10-ubuntu
kube-controller-manager-vm-0-10-ubuntu   1/1     Running   1          4h48m   172.17.0.10   vm-0-10-ubuntu
kube-flannel-ds-amd64-hpqmx              1/1     Running   6          134m    172.17.0.5    work2         
kube-flannel-ds-amd64-m9j45              1/1     Running   0          4h21m   172.17.16.3   work1         
kube-flannel-ds-amd64-t76wt              1/1     Running   2          4h48m   172.17.0.10   vm-0-10-ubuntu
kube-proxy-8cwcf                         1/1     Running   0          134m    172.17.0.5    work2         
kube-proxy-j4mzr                         1/1     Running   1          4h49m   172.17.0.10   vm-0-10-ubuntu
kube-proxy-ttct9                         1/1     Running   0          4h21m   172.17.16.3   work1         
kube-scheduler-vm-0-10-ubuntu            1/1     Running   1          4h48m   172.17.0.10   vm-0-10-ubuntu
```
## Work节点配置
与master节点相同，需要先安装docker-ce，kubelet，kubeadm，kubectl  
但似乎不需要pull所有kubernetes相关的镜像，只需要其中的pause和proxy
```
root@work1:~# docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy               v1.13.1             fdb321fd30a0        3 weeks ago         80.2MB
quay.io/coreos/flannel              v0.10.0-amd64       f0fad859c909        11 months ago       44.6MB
k8s.gcr.io/pause                    3.1                 da86e6ba6ca1        12 months ago       742kB
``` 

当master节点init成功后会生成相应的 ```kubeadm join``` 指令，每个work节点只需要简单地执行这条指令就能加入master所在集群中
```
kubeadm join 172.17.0.10:6443 --token 619lcn.m0matbodlbm2ozpo --discovery-token-ca-cert-hash sha256:f1da4bddf7a2dba3a7d1fc104bfa2b6d48fbd07588fbf687c24be6659ed1a1c2
```
执行完成后回到master节点再执行 ```kubectl get nodes``` 指令就能看到work节点已经加入到当前集群中

```
root@VM-0-10-ubuntu:~# kubectl get nodes
NAME             STATUS   ROLES    AGE     VERSION
vm-0-10-ubuntu   Ready    master   3h33m   v1.13.1
work1            Ready    <none>   3h5m    v1.13.1
work2            Ready    <none>   58m     v1.13.1
```

## 搭建dashboard
配置文件安装kubernetes-dashboard
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
查看dashboard的POD是否正常启动
```
root@VM-0-10-ubuntu:/home/ubuntu# kubectl get pods -n kube-system -o wide
NAME                                     READY   STATUS    RESTARTS   AGE     IP            NODE  
...        
kubernetes-dashboard-57df4db6b-cf777     1/1     Running   2          4h42m   10.244.0.16   vm-0-10-ubuntu
...
```
配置外网访问，修改service配置将type: ClusterIP改为NodePort
```
kubectl edit service  kubernetes-dashboard --namespace=kube-system
```
```
spec:
  clusterIP: 10.96.24.198
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31893
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
```
查看外网暴露端口,这里是31893
```
root@VM-0-10-ubuntu:~# kubectl get service --namespace=kube-system
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   24h
kubernetes-dashboard   NodePort    10.96.24.198     <none>        443:31893/TCP   24h
metrics-server         ClusterIP   10.108.107.242   <none>        443/TCP         24h
```
配置文件创建dashboard用户并获取登录token
```
kubectl create -f admin-token.yaml 
kubectl describe secret/$(kubectl get secret -nkube-system |grep admin|awk '{print $1}') -nkube-system
```
接着按理说访问 https://服务器外网IP:暴露端口 就能进入dashboard界面，然后使用token就能正常登录。但是尝试了各种方法都无法访问到dashboard界面，猜想是由于证书的问题，但目前仍未找到有效的解决方式。
