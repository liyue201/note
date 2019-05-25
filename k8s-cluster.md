## 3 master+1 haproxy + n worker 的k8s集群部署
###  1.准备工作

节点配置
```
master1:10.102.20.119
master2:10.102.20.120
master3:10.102.20.121
haproxy:10.102.20.122
worker1:10.102.20.123
worker2:10.102.20.124
...
```

设置hostname （master、haproxy、worker每台机器上分别执行）

```
$ hostnamectl set-hostname master1
$ hostnamectl set-hostname master2
$ hostnamectl set-hostname master3
$ hostnamectl set-hostname haproxy
$ hostnamectl set-hostname worker1
$ hostnamectl set-hostname worker2
...
```

修改/etc/hosts （每台master上）
```
10.102.20.119     master1
10.102.20.120     master2
10.102.20.121     master3
```

后面有很多scp操作，为了方便，需要配置master1到master1, master2, master3的免密登陆（master1上）
```
$ ssh-keygen
$ scp .ssh/id_rsa.pub master1:
$ scp .ssh/id_rsa.pub master2:
$ scp .ssh/id_rsa.pub master3:
```

在master1, master2, master3操作（每台master上分别执行）
```
$  mkdir .ssh && cat id_rsa.pub >> .ssh/authorized_keys
```

设置 net.bridge （master、haproxy、worker）
```
$ cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```
$ sysctl --system 
```

禁用Selinux disabled， 这里要重启电脑生效 （master、haproxy、worker）
```
$ vim /etc/sysconfig/selinux

SELINUX=disabled
```

关闭防火墙 （master、haproxy、worker）
```
$ systemctl disable firewalld && systemctl stop firewalld 
```

关闭 swap 分区 （master、haproxy、worker）
```
$ swapoff -a
```
要永久禁掉swap分区，打开如下文件注释掉swap那一行
```
$ vi /etc/fstab
```



### 2. 安装haproxy

在haproxy上执行
```
master1=10.102.20.119
master2=10.102.20.120
master3=10.102.20.121

yum install -y haproxy
systemctl enable haproxy
cat << EOF >> /etc/haproxy/haproxy.cfg
listen k8s-lb *:6443
        mode tcp
        balance roundrobin
        server s1 $master1:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
        server s2 $master2:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
        server s3 $master3:6443 weight 1 maxconn 1000 check inter 2000 rise 2 fall 3
EOF
```
```
$ service haproxy start
```

### 3.安装ETCD集群
在3台master上安装etcd
```
$ yum install -y etcd
$ systemctl enable etcd
```

修改etcd 数据目录权限

```
$ chmod 777 -R /jdata
```

在master1上操作

```
etcd1=10.102.20.119
etcd2=10.102.20.120
etcd3=10.102.20.121

TOKEN=abcd1234
ETCDHOSTS=($etcd1 $etcd2 $etcd3)
NAMES=("infra0" "infra1" "infra2")
for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
cat << EOF > /tmp/$NAME.conf
# [member]
ETCD_NAME=$NAME
ETCD_DATA_DIR="/jdata/etcd"
ETCD_LISTEN_PEER_URLS="http://$HOST:2380"
ETCD_LISTEN_CLIENT_URLS="http://$HOST:2379,http://127.0.0.1:2379"
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://$HOST:2380"
ETCD_INITIAL_CLUSTER="${NAMES[0]}=http://${ETCDHOSTS[0]}:2380,${NAMES[1]}=http://${ETCDHOSTS[1]}:2380,${NAMES[2]}=http://${ETCDHOSTS[2]}:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="$TOKEN"
ETCD_ADVERTISE_CLIENT_URLS="http://$HOST:2379"
EOF
done
```

覆盖每台master的etcd配置

```
for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
scp /tmp/$NAME.conf $HOST:
ssh $HOST "\mv -f $NAME.conf /etc/etcd/etcd.conf"
rm -f /tmp/$NAME.conf
done
```

在每台master节点上启动etcd
master1上执行service etcd start，会一直pending状态，等master2的etcd启动以后就会完成。
```
$ service etcd start
```

任意节点验证集群
```
$  etcdctl member list
$  etcdctl cluster-health
```

### 4.安装docker

每台master上
```
$ yum install docker -y

$ systemctl enable docker 
$ systemctl start docker
```

### 5. 安装kubelet kubeadm kubectl

配置源（每台master上）
```
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装
```
$ yum install -y kubelet-1.12.8 kubeadm-1.12.8 kubectl-1.12.8
$ systemctl enable kubelet
$ systemctl start kubelet
```

### 6. 安装master集群

在master1上初始化集群
```
$ vim /etc/sysconfig/kubelet 
KUBELET_EXTRA_ARGS="--v=2 --fail-swap-on=false --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/k8sth/pause-amd64:3.0"
KUBELET_NETWORK_ARGS="--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
KUBELET_DNS_ARGS="--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
KUBELET_KUBEADM_EXTRA_ARGS="--cgroup-driver=cgroupfs"

```
生成配置集群配置文件
```
proxy=10.102.20.122
 
etcd1=10.102.20.119
etcd2=10.102.20.120
etcd3=10.102.20.121

master1=$etcd1
master2=$etcd2
master3=$etcd3

cat << EOF > kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1alpha3
kind: ClusterConfiguration
kubernetesVersion: v1.12.8
apiServerCertSANs:
- "$proxy"
controlPlaneEndpoint: "$proxy:6443"
etcd:
  external:
    endpoints:
    - "http://$etcd1:2379"
    - "http://$etcd2:2379"
    - "http://$etcd3:2379"
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
networking:
    podSubnet: "10.244.0.0/16"
EOF
```

初始化集群
```
$ kubeadm init --config kubeadm-config.yaml

成功会输出类似下面这串文字，保存下来，后面其他节点加入集群时用到
  //  kubeadm join 10.102.20.122:6443 --token sjj4lo.gjdfg4u41147rwks --discovery-token-ca-cert-hash sha256:4adb61e299374dbe2cd27289eb2014213dbc88785b3373d9c556048745221469
  
```

执行这个才能使用kubectl
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

使用kubectl查看集群状态
```
$ kubectl get nodes --all-namespaces
$ kubectl get cs --all-namespaces
$ kubectl get pods -o wide  --all-namespaces 
```

接下来需要安装flannel组件，这样master节点才能变成Ready状态

```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


拷贝集群需要的证书到其它master节点
```
$ cat << EOF > certificate_files.txt
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/sa.pub
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/front-proxy-ca.key
EOF
```
```
$ tar -czf control-plane-certificates.tar.gz -T certificate_files.txt

$ CONTROL_PLANE_IPS="$master2 $master3"
$ for host in ${CONTROL_PLANE_IPS}; do
    scp control-plane-certificates.tar.gz $host:
done
```

配置其它master节点
到master2, master3上执行如下脚本
```
$ mkdir -p /etc/kubernetes/pki
$ tar -xzf control-plane-certificates.tar.gz -C /etc/kubernetes/pki --strip-components 3
```

执行master1上生成的kubeadm join指令，在指令最后加入参数"–experimental-control-plane"，指令最后类似
```
 kubeadm join 10.102.20.122:6443 --token sjj4lo.gjdfg4u41147rwks --discovery-token-ca-cert-hash sha256:4adb61e299374dbe2cd27289eb2014213dbc88785b3373d9c556048745221469 --experimental-control-plane
```

配置时，经常出现出错的情况，可能需要reset并清理生成的文件，以免下次init时出错或干扰

```
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
rm -rf /etc/kubernetes/
systemctl stop kubelet
find /var/lib/kubelet | xargs -n 1 findmnt -n -t tmpfs -o TARGET -T | uniq | xargs -r umount -v

service etcd stop
rm -rf /var/lib/kubelet /jdata/etcd/*

service etcd start
etcdctl member list
etcdctl cluster-health
```

如何从集群中移除Node
在master节点上执行：
```
kubectl drain k8s2 --delete-local-data --force --ignore-daemonsets
kubectl delete node k8s2
```
在k8s2上执行：
```
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
```

### 6. 安装worker节点

参照上面准备工作进行如下操作
设置每台机器的hostname
设置 net.bridge
禁用Selinux
关闭 swap 分区
禁用firewalld
安装docker
设置docker运行根目录
```
$ cat << EOF > /etc/docker/daemon.json
{
  "graph": "/jdata/docker"
}
EOF
```
安装kubelet kubeadm kubectl

下载镜像,由于墙的原因需要提前下载这镜像，再改名
```
$ docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
$ docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
```

加入集群
```
$ kubeadm join 10.102.20.122:6443 --token sjj4lo.gjdfg4u41147rwks --discovery-token-ca-cert-hash sha256:4adb61e299374dbe2cd27289eb2014213dbc88785b3373d9c556048745221469
```
