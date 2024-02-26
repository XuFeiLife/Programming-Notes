# Centos安装Kubernetes

## 基本环境
每个机器使用内网ip互通
每个机器配置自己的hostname，不能用localhost  
ks8-master,k8s-node01分别为master和node的hostname
```
#设置每个机器自己的hostname
hostnamectl set-hostname xxx

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
## 安装kubelet、kubeadm、kubectl
```
#配置k8s的yum源地址
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


#安装 kubelet，kubeadm，kubectl
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9

#启动kubelet
sudo systemctl enable --now kubelet

#所有机器配置master域名 172.31.0.4是你的master的ip地址
echo "172.31.0.4  k8s-master" >> /etc/hosts
```
## 下载各个机器需要的镜像
```shell

sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
```

## 初始化master节点
### 初始化
172.31.0.4为master节点ip，注意各个网段不能重复，pod-network-cidr该网段地址默认是192.168.0.0/16  
如果需要修改需要同步修改calico网络中的网段地址，关键字CALICO_IPV4POOL_CIDR
``` shell
kubeadm init \
--apiserver-advertise-address=172.31.0.4 \
--control-plane-endpoint=k8s-master \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16

#所有网络范围不重叠

```
### 记录关键信息
```shell
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

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-master:6443 --token 3vckmv.lvrl05xpyftbs177 \
    --discovery-token-ca-cert-hash sha256:1dc274fed24778f5c284229d9fcba44a5df11efba018f9664cf5e8ff77907240 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master:6443 --token 3vckmv.lvrl05xpyftbs177 \
    --discovery-token-ca-cert-hash sha256:1dc274fed24778f5c284229d9fcba44a5df11efba018f9664cf5e8ff77907240
```
### 设置.kube/config，根据提示复制上面的命令即可
```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 安装calico网络，需要下载镜像时间较长
```shell

curl https://docs.projectcalico.org/v3.20/manifests/calico.yaml -O

kubectl apply -f calico.yaml
```

## 加入worker节点
新令牌可以使用kubeadm token create --print-join-command命令生成 
```shell
kubeadm join k8s-master:6443 --token x5g4uy.wpjjdbgra92s25pp \
  --discovery-token-ca-cert-hash sha256:6255797916eaee52bf9dda9429db616fcd828436708345a308f4b917d3457a22
```

## 验证集群状态
需要都为ready状态
```shell
kubectl get nodes
```
