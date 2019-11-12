# 安装部署
## kubeadm方式
### 环境说明
- 系统信息
```
centos7.4
3.10.0-693.el7.x86_64
8C32G
```
- 软件信息
```
docker 19.03.4
etcdctl version: 3.3.11
API version: 3
k8s v1.16.2
calico v3.8.4
```
- 节点信息
```
10.43.75.131 k8s-m1
10.43.75.132 k8s-m2
10.43.75.133 k8s-m3
```

### 系统初始化
```
#关闭交换空间
swapoff -a  &&  sed -i 's/.*swap.*/#&/' /etc/fstab
```
```
#开启bridge-nf 允许二层的网桥在转发包时会被iptables的FORWARD规则所过滤
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
#不使用swap
vm.swappiness=0
EOF
```
```
sysctl --system
```
```
#加载ipvs模块（为了kube-proxy使用ipvs模式）
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
```
- 配置仓库
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF
```

- 同步时间
```
ntpdate time1.aliyun.com
```

- 安装docker-ce

安装步骤省略。

- 安装依赖包
```
yum -y install ipvsadm ipset
```

- 安装基本组件
```
yum install -y kubectl kubeadm kebelet
```
- 加入开机自启
```
systemctl enable kubelet
```

### 配置证书
- 安装cfssl工具
```
cd /opt
mkdir k8s
cd k8s
wget -O /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget -O /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget -O /bin/cfssl-certinfo  https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
for cfssl in `ls /bin/cfssl*`;do chmod +x $cfssl;done;
```
```
mkdir ssl
cd ssl
```

- 配置etcd的证书
```
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```
```
cat > etcd-ca-csr.json << EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
EOF
```
```
cat > etcd-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "10.43.75.131",
      "10.43.75.132",
      "10.43.75.133"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shanghai",
            "L": "Shanghai",
            "O": "etcd",
            "OU": "Etcd Security"
        }
    ]
}
EOF
```
- 生成证书
```
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca
cfssl gencert -ca=etcd-ca.pem -ca-key=etcd-ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
```
- 分发证书
```
mkdir -pv /etc/etcd/ssl
mkdir -pv /etc/kubernetes/pki/etcd
cp etcd*.pem /etc/etcd/ssl
cp etcd*.pem /etc/kubernetes/pki/etcd
```
```
scp -r -P52222 /etc/etcd k8s-m1:/etc/
scp -r -P52222 /etc/etcd k8s-m2:/etc/
scp -r -P52222 /etc/etcd k8s-m3:/etc/
```

### 安装etcd
```
yum install etcd -y
```
- 配置
```
vim /etc/etcd/etcd.conf
```
etcd1
```
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://10.43.75.131:2380"
ETCD_LISTEN_CLIENT_URLS="https://localhost:2379,https://10.43.75.131:2379"
ETCD_NAME="etcd1"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.43.75.131:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://localhost:2379,https://10.43.75.131:2379"
ETCD_INITIAL_CLUSTER="etcd1=https://10.43.75.131:2380,etcd2=https://10.43.75.132:2380,etcd3=https://10.43.75.133:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
```
etcd2
```
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://10.43.75.132:2380"
ETCD_LISTEN_CLIENT_URLS="https://localhost:2379,https://10.43.75.132:2379"
ETCD_NAME="etcd2"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.43.75.132:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://localhost:2379,https://10.43.75.132:2379"
ETCD_INITIAL_CLUSTER="etcd1=https://10.43.75.131:2380,etcd2=https://10.43.75.132:2380,etcd3=https://10.43.75.133:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
```
etcd3
```
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://10.43.75.133:2380"
ETCD_LISTEN_CLIENT_URLS="https://localhost:2379,https://10.43.75.133:2379"
ETCD_NAME="etcd3"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.43.75.133:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://localhost:2379,https://10.43.75.133:2379"
ETCD_INITIAL_CLUSTER="etcd1=https://10.43.75.131:2380,etcd2=https://10.43.75.132:2380,etcd3=https://10.43.75.133:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
```
- 启动
```
chown -R etcd:etcd /etc/etcd/
```
```
systemctl enable etcd
systemctl start etcd
```
- 注意点

因为最新的k8s集群只支持etcd api v3，所以使用的时候要注意添加环境变量：
```
ETCDCTL_API=3 etcdctl version
etcdctl version: 3.3.11
API version: 3.3
```
- 检查etcd集群
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert="/etc/etcd/ssl/etcd-ca.pem" --cert="/etc/etcd/ssl/etcd.pem" --key="/etc/etcd/ssl/etcd-key.pem"   member list
```
```
5018a20e9dd0a52e, started, etcd1, https://10.43.75.131:2380, https://10.43.75.131:2379,https://localhost:2379
743b11be54fa9c3a, started, etcd3, https://10.43.75.133:2380, https://10.43.75.133:2379,https://localhost:2379
f2896e201f815f09, started, etcd2, https://10.43.75.132:2380, https://10.43.75.132:2379,https://localhost:2379
```
- 测试写入和读取

写入
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert="/etc/etcd/ssl/etcd-ca.pem" --cert="/etc/etcd/ssl/etcd.pem" --key="/etc/etcd/ssl/etcd-key.pem" put /name1 "leo"
```
读取
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert="/etc/etcd/ssl/etcd-ca.pem" --cert="/etc/etcd/ssl/etcd.pem" --key="/etc/etcd/ssl/etcd-key.pem" get /name1
```

### master节点（k8s-m1）
- 编写配置文件
```
cd /opt/k8s
vim kubeadm-config.yml
```
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
apiServer:
  certSANs:
  - "10.43.75.131"
  - "10.43.75.132"
  - "10.43.75.133"
  - "127.0.0.1"
etcd:
  external:
      endpoints:
      - https://10.43.75.131:2379
      - https://10.43.75.132:2379 
      - https://10.43.75.133:2379 
      caFile: /etc/kubernetes/pki/etcd/etcd-ca.pem 
      certFile: /etc/kubernetes/pki/etcd/etcd.pem 
      keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem 
controlPlaneEndpoint: "10.43.75.131:6443"
networking:
  podSubnet: 10.244.0.0/16
imageRepository: registry.aliyuncs.com/google_containers
```
- kubeadm初始化
```
kubeadm init --config kubeadm-config.yml
```
```
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities 
and service account keys on each node and then running the following as root:

  kubeadm join 10.43.75.131:6443 --token 7zppep.jn98shsqab6rmwae \
    --discovery-token-ca-cert-hash sha256:efe814413debd9c764f2e9f207fe6f6ee1b9decaf04f9642b94cedd6723818c2 \
    --control-plane       

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.43.75.131:6443 --token 7zppep.jn98shsqab6rmwae \
    --discovery-token-ca-cert-hash sha256:efe814413debd9c764f2e9f207fe6f6ee1b9decaf04f9642b94cedd6723818c2 
```
- k8s认证配置
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- 配置网络插件
```
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```
- 使用ipvs
```
#1.修改ConfigMap的kube-system/kube-proxy中的config.conf，mode: “ipvs”：
kubectl edit cm kube-proxy -n kube-system
#2.删除pods 重新创建
kubectl get pod -n kube-system | grep kube-proxy | awk '{system("kubectl delete pod "$1" -n kube-system")}'
```

### node节点
- 加入node节点到集群
```
kubeadm join 10.43.75.131:6443 --token 7zppep.jn98shsqab6rmwae \
    --discovery-token-ca-cert-hash sha256:efe814413debd9c764f2e9f207fe6f6ee1b9decaf04f9642b94cedd6723818c2 
```
- 检查节点信息
```
kubectl get nodes
```
```
NAME     STATUS   ROLES    AGE    VERSION
k8s-m1   Ready    master   41m    v1.16.2
k8s-m2   Ready    <none>   3m1s   v1.16.2
k8s-m3   Ready    <none>   2m8s   v1.16.2
```

### 测试集群是否正常工作
- 测试用例
```
cd /opt/k8s
```
```
cat << EOF > nginx.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
  - port: 80
    nodePort: 31000
    name: nginx-port
    targetPort: 80
    protocol: TCP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```
```
kubectl apply -f nginx.yaml
```
- 检查域名解析
```
kubectl run curl --image=radial/busyboxplus:curl -i --tty
$ nslookup nginx
$ nslookup kubernetes
```
nginx域名解析正确，表示集群工作正常。

### 安装Rancher
rancher是k8s管理的平台。
```
mkdir /data/rancher
mkdir /var/log/rancher/
```
```
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /data/rancher:/var/lib/rancher/ \
-v /root/var/log/rancher/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
rancher/rancher:stable
```
- 访问web
```
https://10.43.75.131
```
- 添加一个集群

- 在master节点执行下面命令：
![](https://res.cloudinary.com/dkkg9pm0i/image/upload/v1573550572/websites/ecarx/QQ%E6%88%AA%E5%9B%BE20191112172057.png)
