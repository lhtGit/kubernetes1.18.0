**基于centOS7，安装kubernetes1.8.0**

### 环境准备

需要最少2个cpu和4g内存

1. **修改hostname**

如果hostname是localhost，需要修改，比如：master  node。

2. **禁用防火墙**

```bash
systemctl stop firewalld
systemctl disable firewalld
```

   

3. **禁用SELINUX**

执行如下命令：

```bash
vim /etc/sysconfig/selinux
```


修改文件中的SELINUX为： 

```bash
SELINUX=disabled
```



4. **关闭swap内存**

执行命令：

```bash
swapoff -a
```

5. **调整内核参数**

执行命令

```sh
vi /etc/sysctl.d/k8s.conf
```


在新建的k8s.cof文件中增加：

```bash
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

执行如下命令使之生效

```bash
sysctl --system
```

6.  **配置yum源镜像**

kubernetes yum源 

```bash
vi /etc/yum.repos.d/kubernetes.repo
```

内容为

```bash
[kubernetes] 
name=Kubernetes 
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/ 
enabled=1 
gpgcheck=0
```

### 安装阶段

1. **安装docker(根据docker官方文档安装就可以了)**

```bash
sudo yum remove docker \
                 docker-client \
                 docker-client-latest \
                 docker-common \
                 docker-latest \
                 docker-latest-logrotate \
                 docker-logrotate \
                 docker-engine
```

```bash
sudo yum install -y yum-utils
```

```bash
sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

```bash
sudo systemctl start docker
```

```bash
systemctl enable docker.service
```

2. **安装kubernetes**

```bash
setenforce 0
```

```bash
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

```bash
systemctl enable kubelet.service
```

安装完成后

```
已安装:
 kubeadm.x86_64 0:1.18.2-0                                    kubectl.x86_64 0:1.18.2-0                                    kubelet.x86_64 0:1.18.2-0                                   

作为依赖被安装:
 conntrack-tools.x86_64 0:1.4.4-5.el7_7.2           cri-tools.x86_64 0:1.13.0-0                  kubernetes-cni.x86_64 0:0.7.5-0    libnetfilter_cthelper.x86_64 0:1.0.0-10.el7_7.1   
 libnetfilter_cttimeout.x86_64 0:1.0.0-6.el7_7.1    libnetfilter_queue.x86_64 0:1.0.2-2.el7_2    socat.x86_64 0:1.7.3.2-2.el7      

完毕！

```

修改docker配置

```bash
vim /etc/docker/daemon.json
```

修改或者添加内容

```bash
{
 "exec-opts": ["native.cgroupdriver=systemd"]
}
```
重启docker

```
systemctl daemon-reload
systemctl restart docker.service 
```


**初始化kubernetes**

```bash
kubeadm init --image-repository=registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address 192.168.92.132 --kubernetes-version=v1.18.0
```

- `--image-repository`：指定了镜像位置，原始的镜像在google服务上，需要科学上网
- `--pod-network-cidr`：指定 Pod 网络的范围。Kubernetes 支持多种网络方案，而且不同网络方案对 --pod-network-cidr 有自己的要求，这里设置为 10.244.0.0/16 是因为我们将使用 flannel 网络方案，必须设置成这个 CIDR。有其他方案，在这不做详细表述
- `--apiserver-advertise-address`：指明用 Master 的哪个 interface 与 Cluster 的其他节点通信。如果 Master 有多个 interface，建议明确指定，如果不指定，kubeadm 会自动选择有默认网关的 interface。



**初始化成功**

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
# 下面的命令是配置常规用户如何使用kubectl访问集群
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config


You should now deploy a pod network to the cluster. 
#提示如何安装 Pod 网络
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
 https://kubernetes.io/docs/concepts/cluster-administration/addons/


Then you can join any number of worker nodes by running the following on each as root:
#将节点加入集群的命令
kubeadm join 192.168.92.132:6443 --token jhzf5z.tvsp926nqrhjj9gi \
   --discovery-token-ca-cert-hash sha256:ef6a1ced3cccbe27f1f3e6220ad5c3ea75c603072382521a5386d2ee88fdb0ec 

```

**在/etc/profile末尾增加**

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

刷新

```bash
source /etc/profile
```
