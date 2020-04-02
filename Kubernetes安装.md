|作者|王亮|
|---|---
|公众号|。。。。。。
## 目录
* [设置主机名](#设置主机名)
* [关闭防火墙](#关闭防火墙)
* [关闭selinux](#关闭selinux)
* [关闭swap](#关闭swap)
* [安装Docker](#安装Docker)
* [安装Kubeadm&Kubelet](#安装Kubeadm&Kubelet)
* [部署KubernetesMaster](#部署KubernetesMaster)
* [测试命令](#测试命令)
* [外网无法访问nodePort问题](#外网无法访问nodePort问题)

### 设置主机名线
[root@localhost ~]# vi /etc/hostname
hostnamectl set-hostname k8s-master

然后，更改hosts文件添加主机名与IP映射关系

 vi /etc/hosts

192.168.37.100 k8s-master


192.168.37.101 k8s-node1


192.168.37.102 k8s-node2




### 关闭防火墙
systemctl stop firewalld


systemctl disable firewalld
### 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config


setenforce 0
### 关闭swap
K8S中不支持swap分区


#vi /etc/fstab


#/dev/mapper/centos-swap swap                    swap    defaults        0 0


　*.编辑etc/fstab将swap那一行注释掉或者删除掉



echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables 


echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables


cat > /etc/sysctl.d/k8s.conf << EOF


net.bridge.bridge-nf-call-ip6tables = 1


net.bridge.bridge-nf-call-iptables = 1


EOF



sysctl --system
### 安装Docker
yum -y install wget


#wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O


/etc/yum.repos.d/docker-ce.repo


#yum -y install docker-ce-18.06.1.ce-3.el7


#systemctl enable docker && systemctl start docker


#docker --version


Docker version 18.06.1-ce, build e68fc7a


### 安装Kubeadm&Kubelet
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl


yum install kubectl --nogpgcheck


yum install kubeadm --nogpgcheck


systemctl enable kubelet

### 部署KubernetesMaster
以下步骤请在k8s-master节点上操作：


kubeadm init \
--apiserver-advertise-address=192.168.37.100 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

要开始使用群集，您需要以常规用户身份运行以下命令:


  mkdir -p $HOME/.kube


  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config


  sudo chown $(id -u):$(id -g) $HOME/.kube/config


将Pod网络部署到群集:
 kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml

  kubeadm join 192.168.31.26:6443 --token ltqgvv.klswxmvn5vyozq3e \
    --discovery-token-ca-cert-hash sha256:c0677b3c690dfc83cd1081cd10689807fe62878833a5961fae6b0b8b953338a1

当你的token忘了或者过期，解决办法如下：


1.先获取token


#如果过期可先执行此命令kubeadm token create    #重新生成token#列出token


kubeadm token list  | awk -F" " '{print $1}' |tail -n 1

#生成ca的sha256 hash值
[root@node1 flannel]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

### 测试命令


kubectl get nodes


kubectl get pods -n kube-system


kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml


kubectl get pod --all-namespaces
### 外网无法访问nodePort问题
解决k8s 外网无法访问nodePort问题


iptables -P FORWARD ACCEPT