---
title: "Centos_install_kubernetes"
date: 2019-06-14T15:47:05+08:00
---

# CentOS 7.5(minimal) 安装Kubernetes (kubeadmin方式)

1. 安装基础常用包
```
yum install net-tools bash-completion yum-utils
```
2. 关闭firewalld

```
systemctl stop firewalld && systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
3. 设置hostname

```
hostnamectl set-hostname k8s
```

4. 设置系统参数，

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
5. 加载模块

```
lsmod|grep br_netfilter
modprobe br_netfilter

```
6. 安装Kubeadm  Kubectl   Kubelet

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install  kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

4. 安装docker

   ```
   [root@k8s ~(keystone_admin)]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```
5. 禁用docker-c-edge 源 edge是开发版，不稳定，下载stable版本

```
[root@k8s ~(keystone_admin)]# yum-config-manager --disable docker-ce-edge
```
6. 安装docker-ce 相应版本

```
[root@k8s ~(keystone_admin)]# yum install docker-ce 
```
7. 配置Docker代理(可选)

```
方法1. 如果是docker 版本有该文件(docker版本不同，显示不同)
[root@k8s ~(keystone_admin)]# vim /etc/sysconfig/docker
HTTP_PROXY=http://127.0.0.1:39999

方法2. docker 版本18.09.6
[root@k8s ~(keystone_admin)]# vim /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:39999/"

[root@k8s ~(keystone_admin)]# systemctl restart docker

[root@k8s ~(keystone_admin)]# docker info
Containers: 1
 Running: 0
 Paused: 0
 Stopped: 1
Images: 17
Server Version: 18.09.6
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: bb71b10fd8f58240ca47fbb579b9d1028eea7c84
runc version: 2b18fe1d885ee5083ef9f0838fee39b62d653e30
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-862.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 8
Total Memory: 9.607GiB
Name: k8s
ID: JJKA:BTVA:PLYX:A5KP:MLOR:XB2B:OEEA:H4OR:MG4X:AFR3:GZKU:D43N
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
HTTP Proxy: http://127.0.0.1:39999/
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
Product License: Community Engine
```

8. 测试docker 

```
docker run hello-world
```
9. 下载所需要的镜像
```
kubeadmin config image pull
这里下载的镜像 根据kubeadmin的版本不同 下载不同的镜像tag
```
10. 初始化master

参考：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/
```
kubeadm init --apiserver-advertise-address=10.85.44.223 --kubernetes-version=v1.14.3 --pod-network-cidr=10.200.0.0/16
* 如果是IPV6网络： --apiserver-advertise-address=fd00::101

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
# 安装pod所需要的网路flannel


网络的创建必须是在k8s的所有应用之前, kubeadmin仅仅支持基于CNI(Container Network Interface)的网络,目前支持的情况查看https://kubernetes.io/docs/concepts/cluster-administration/addons/   这里用flannel举例说明

https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md
## flannel 说明
1.	如果设置 flannel的网络话，如果不进行修改即可使用，必须设置--pod-network-cidr=10.244.0.0/16 to kubeadm init
2.	设置系统参数使flannel cni可以去工作
```
sysctl net.bridge.bridge-nf-call-iptables=1
```
3. 如果开启了防火墙，请确保允许UDP 8285 8472对于所有的host node节点。

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml

也可以将上述的 kube-flannel-legacy.yml wget下来后 进行更改 以适配自身的环境，与kubeadmin的网络相匹配

4. 查看是否正常
```
kubectl get pods --all-namespaces
```
# 增加节点
1. 获取目前的token
```
kubeadmin token list
```
2. 创建新的token
默认创建的tokne 24小时会直接过过期，然后可以在master节点上重新创建一个新的即可
```
kubeadm token create
```

3. 加入master集群
```
kubeadm join --token <token> <master-ip>:<master-port>
kubeadm join --tpken 8ewj1p.9r9hcjoqgajrj4gi 10.85.44.224:6443
```
4.  验证
```
kubectl get cs
kubectl get nodes
kubectl get pods --all-namespaces
```