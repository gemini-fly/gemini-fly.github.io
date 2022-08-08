---
title: k8s-kubeadm安装K8S1.22.2
layout: post
categories: k8s
tags: k8s
abbrlink: 12126
date: 2021-09-24 11:25:09
password: ddppff
abstract: 有东西被加密了，请输入密码查>看
message: 请输入密码
wrong_pass_message: 抱歉, 这个密码看着>不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能>被校验, 不过您还是能看看解密后的内容.
---
### 一、在生产环境中，我们k8s集群需要多master实现高可用，所以下面介绍如何通过kubeadm部署k8s高可用集群（建议生产环境master至少3个以上）

<!--more-->

### 二、master部署

1、三台maser节点上部署etcd集群
2、本次是通过阿里云服务器进行部署，由于阿里云服务器不支持VIP，所以通过SLB做负载均衡

### 三、环境准备；

|节点主机： | IP地址 | 操作系统 |
| --- | --- | --- |
| test-k8s-master-001 | 10.111.241.148 | CentOS Linux release 7.8.2003 (Core) |
| test-k8s-master-002 | 10.111.241.149 | CentOS Linux release 7.8.2003 (Core) |
| test-k8s-master-003 | 10.111.241.147 | CentOS Linux release 7.8.2003 (Core) |
| test-k8s-node-001 | 10.111.241.150 | CentOS Linux release 7.8.2003 (Core) |
1、配置yum源、安装相关的依赖包以及相关主件、配置内核优化参数等，这里我使用一个脚本直接安装（所有节点上面都需要执行）
cat init.sh
```
echo '安装必要软件'
yum install -y vim wget yum-utils device-mapper-persistent-data lvm2 bash-completion epel-release vim screen bash-completion mtr  wget telnet zip unzip sysstat  libcurl openssl bridge-utils nethogs dos2unix htop nfs-utils ceph-common git ipvsadm ipset


echo '更新yum源'
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
wget http://mirrors.aliyun.com/repo/epel-7.repo -O /etc/yum.repos.d/epel.repo
cat >>/etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum clean all
yum makecache

yum install -y kubelet-1.22.2 kubeadm-1.22.2 kubectl-1.22.2 docker-ce
systemctl restart docker && systemctl enable docker
systemctl enable kubelet

cat >>/etc/docker/daemon.json <<EOF
{
"log-driver": "json-file",
"log-opts": {
"max-size": "100m",
"max-file": "1"
},
"storage-driver": "overlay2",
"storage-opts": [
"overlay2.override_kernel_check=true"
],
"registry-mirrors": [
"https://w6pljua0.mirror.aliyuncs.com"],
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
systemctl daemon-reload
systemctl restart docker

echo '/etc/security/limits.conf 参数调优，需重启系统后生效'


#设置网桥：
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
EOF
# 执行命令使其修改生效
modprobe br_netfilter && sysctl -p /etc/sysctl.d/k8s.conf
```
关闭交换分区：
```
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak | grep -v swap > /etc/fstab
rm -rf /etc/fstab_bak
```
添加ipvs模块
```
cat >>/etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_lc
modprobe -- ip_vs_wlc
modprobe -- ip_vs_lblc
modprobe -- ip_vs_lblcr
modprobe -- ip_vs_dh
modprobe -- ip_vs_sh
modprobe -- ip_vs_nq
modprobe -- ip_vs_sed
modprobe -- ip_vs_ftp
modprobe -- ip_tables
modprobe -- ip_set
modprobe -- xt_set
modprobe -- ipt_set
modprobe -- ipt_rpfilter
modprobe -- ipt_REJECT
modprobe -- ipip
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules
bash /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```
提前下载镜像(3台master主机上下载)：
```
#查看所需镜像
[root@master01 tools]# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.22.2
k8s.gcr.io/kube-controller-manager:v1.22.2
k8s.gcr.io/kube-scheduler:v1.22.2
k8s.gcr.io/kube-proxy:v1.22.2
k8s.gcr.io/pause:3.5
k8s.gcr.io/etcd:3.5.0-0
k8s.gcr.io/coredns/coredns:v1.8.4
```
更换镜像
```
#!/bin/bash
images=(
	kube-apiserver:v1.22.2
	kube-controller-manager:v1.22.2
	kube-scheduler:v1.22.2
	kube-proxy:v1.22.2
	pause:3.5
	etcd:3.5.0-0
)
for imageName in ${images[@]};
do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName       k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
docker pull coredns/coredns:1.8.4
docker tag coredns/coredns:1.8.4 k8s.gcr.io/coredns/coredns:v1.8.4
docker rmi coredns/coredns:1.8.4   
```
在master节点保存镜像；
```
docker save -o kube-proxy.tar k8s.gcr.io/kube-proxy:v1.22.2
docker save -o coredns.tar k8s.gcr.io/coredns/coredns:v1.8.4
docker save -o pause.tar k8s.gcr.io/pause:3.5
```
在node节点上导入镜像：
```
docker load  kube-proxy.tar k8s.gcr.io/kube-proxy:v1.22.2
docker load  coredns.tar k8s.gcr.io/coredns:v1.8.4
docker load  pause.tar k8s.gcr.io/pause:3.5
```

二、初始化集群(master01操作)
使用kubeadm config print init-defaults > kubeadm-init.yaml 打印出默认配置，然后在根据自己的环境修改配置； 需要修改advertiseAddress、controlPlaneEndpoint、imageRepository、serviceSubnet；
```
  kubeadm init --config kubeadm-init.yaml

apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.183.90.133 #master01地址
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: test-k8s-master-0001
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
controlPlaneEndpoint: test-k8s-master:6443 #slb地址
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io #仓库地址
kind: ClusterConfiguration
kubernetesVersion: 1.22.2 #k8s版本
networking:
  dnsDomain: cluster.local
  podSubnet: 172.117.0.0/12 #podIP池
  serviceSubnet: 172.17.0.0/12 #serviceIP池
scheduler: {}
```
配置kubectl认证信息
```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```
分发证书
```
for node in prod-ops-k8s-master-02 prod-ops-k8s-master-03; do
  ssh $node "mkdir -p /etc/kubernetes/pki/etcd; mkdir -p ~/.kube/"
  scp /etc/kubernetes/pki/ca.crt $node:/etc/kubernetes/pki/ca.crt
  scp /etc/kubernetes/pki/ca.key $node:/etc/kubernetes/pki/ca.key
  scp /etc/kubernetes/pki/sa.key $node:/etc/kubernetes/pki/sa.key
  scp /etc/kubernetes/pki/sa.pub $node:/etc/kubernetes/pki/sa.pub
  scp /etc/kubernetes/pki/front-proxy-ca.crt $node:/etc/kubernetes/pki/front-proxy-ca.crt
  scp /etc/kubernetes/pki/front-proxy-ca.key $node:/etc/kubernetes/pki/front-proxy-ca.key
  scp /etc/kubernetes/pki/etcd/ca.crt $node:/etc/kubernetes/pki/etcd/ca.crt
  scp /etc/kubernetes/pki/etcd/ca.key $node:/etc/kubernetes/pki/etcd/ca.key
  scp /etc/kubernetes/admin.conf $node:/etc/kubernetes/admin.conf
  scp /etc/kubernetes/admin.conf $node:~/.kube/config
done
```
其余节点进行添加

添加kubelet额外配置文件（所有节点）
```
cat >/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf <<EOF
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# # This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# # This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# # the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
EOF
```
创建kubelet额外配置文件（所有节点）
```
cat >/var/lib/kubelet/kubeadm-flags.env <<EOF
KUBELET_KUBEADM_ARGS="--network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.5"
EOF
```
```
mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
查看一下集群状态，确认个组件都处于healthy状态，结果出现了错误
root@prod-ops-k8s-master-01 opt]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Healthy     ok
etcd-0               Healthy     {"health":"true","reason":""}

```
controller-manager和scheduler为不健康状态，修改/etc/kubernetes/manifests/下的静态pod配置文件kube-controller-manager.yaml和kube-scheduler.yaml，删除这两个文件中命令选项中的- --port=0这行，重启kubelet，再次查看一切正常。
```
systemctl daemon-reload
systemctl restart kubelet

查看当前ipvs调度算法
```
ipvsadm -ln
```
修改ipvs模式（master01操作）
```
kubectl edit cm kube-proxy -n kube-system
增加以下几行(搜索metricsBindAddress)
mode: "ipvs"
ipvs:
  scheduler: "wrr"
```
资源预留（所有节点）
```
vim /usr/lib/systemd/system/kubelet.service
修改
[Service]
ExecStart=/usr/bin/kubelet  \
--enforce-node-allocatable=pods,kube-reserved,system-reserved \
--kube-reserved=cpu=300m,memory=300mMi,ephemeral-storage=2Gi \
--system-reserved=cpu=500m,memory=500Mi,ephemeral-storage=10Gi \
--eviction-hard=memory.available<5%,nodefs.available<10%,imagefs.available<10% \
--eviction-max-pod-grace-period=20 \
--max-pods=100
Restart=always
StartLimitInterval=0
RestartSec=10
```
安装calico
```
wget https://docs.projectcalico.org/manifests/calico.yaml  
修改CALICO_IPV4POOL_CIDR下的ip段为service ip
kubectl apply -f calico.yaml
```
2.5 验证k8s DNS是否可用
If you don't see a command prompt, try pressing enter.
[ root@curl:/ ]$
kubectl run curl --image=radial/busyboxplus:curl -it
进入后执行nslookup kubernetes.default确认解析正常:

Server:    172.16.0.10
Address 1: 172.16.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 172.16.0.1 kubernetes.default.svc.cluster.local
至此kubeadm安装K8S1.22.2安装完成
