# 安装Docker
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 containerd.io
systemctl enable docker
systemctl start docker
```

# 配置k8s的yum源
```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

# 关闭防火墙、SeLinux、swap
```shell
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
swapoff -a
\cp -a /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak | grep -v swap > /etc/fstab
```

# 修改/etc/sysctl.conf
```shell
cat <<EOF >> /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl -p
```

#安装kubelet、kubeadm、kubectl
yum install -y kubelet-1.15.1 kubeadm-1.15.1 kubectl-1.15.1

# 设置镜像
``` shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

# 重启 docker，并启动 kubelet
```shell
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet
```

# 配置 apiserver.demo 的域名
```shell
echo "x.x.x.x  apiserver.demo" >> /etc/hosts
```

# 创建 ./kubeadm-config.yaml
```shell
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: v1.15.1
imageRepository: registry.cn-shanghai.aliyuncs.com/anynya
controlPlaneEndpoint: "apiserver.demo:6443"
networking:
  podSubnet: "10.100.0.1/20"
EOF
```

# 初始化 apiserver
```shell
kubeadm reset
kubeadm init --config=kubeadm-config.yaml --upload-certs
```

# k8s单机
```shell
kubectl taint node master node-role.kubernetes.io/master-
```

# 再次检索 Join
```shell
kubeadm token create --print-join-command
```

# 可能报错解决方案
![报错](https://img-blog.csdnimg.cn/20200117182730950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FwcGxlXzE5MDA=,size_16,color_FFFFFF,t_70)
```shell
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
```

> Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
>
```shell
rm -rf $HOME/.kube
```

> The connection to the server localhost:8080 was refused - did you specify the right host or port?
>
```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```
