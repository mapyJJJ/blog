
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