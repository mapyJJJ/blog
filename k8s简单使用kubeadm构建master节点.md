
> 以下都是在root用户下安装

```bash
# 先安装 Docker
apt-get install docker.io

# 安装 kubeadm, kubelete, kubectl
# 参考官网教程 https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
# 需要翻墙
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```

```bash
# 开始安装

kubeadm init --apiserver-advertise-address=10.190.31.7 --pod-network-cidr=10.244.0.0/16 
# 会卡在 kubeadm config images pull 这里，拉镜像有点慢
# apiserver-advertise-address 指定与其他节点通信的interface
# pod-network-cidr指定pod的网络范围， 这里的10.244 是因为我们使用flannel网络方案，

kubeadm reset  # 如果没成功，先reset

# 以下是未成功可能的解决方案
swapoff -a

# 一直失败最好换个本地网络环境


```

```bash
# 以下是完成后的提示
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.190.31.7:6443 --token xxxx.xxxxxxxxxxxx \
        --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx


# 配置 KUBECONFIG 环境变量为 /etc/kubernetes/admin.config
# 使用 kubectl 

```

```
# 安装flannel网络插件安装
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

---

更新:

### 安装机器说明
- ubuntu
- 2h16g
只有一台机器

### 开始安装

#### 一,安装kubectl, kubeadm, kubelete

```
sudo curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
sudo apt-get install kubectl, kubeadm kubelet # 安装工具
```

#### 二, 拉取镜像(containerd，注意这里用的不是Docker)
先通过 `kubeadm config images list` 获取所需的所有镜像以及版本，填入下面， 需要注意，pause是个坑, 只能是3.6版本, 如果有什么镜像安装不上，手动补上即可
```
#!/bin/bash
images=(
    kube-apiserver:v1.26.1
    kube-controller-manager:v1.26.1
    kube-scheduler:v1.26.1
    kube-proxy:v1.26.1
    pause:3.6
    etcd:3.5.6-0
    coredns/coredns:v1.9.3
)

for imageName in ${images[@]} ; do
    ctr image pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    ctr -n k8s.io image tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName registry.k8s.io/$imageName
done
```

#### 三，开始安装
```
swapoff -a # 一定主要要执行！！
kubeadm init --apiserver-advertise-address=10.190.31.7 --pod-network-cidr=10.244.0.0/16 

# 如果中途出错，请执行reset命令，然后寻找解决办法
kubeadm reset

# 执行成功后，
export KUBECONFIG=/etc/kubernetes/admin.conf
然后就可以使用kubectl管理集群了
```

#### 四，安装网络插件flannel
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

done



