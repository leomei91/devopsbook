[toc]

# 基本概念
## 集群内部DNS解析
### service
主机名完整格式：
```
<service-name>.<namespace-name>.svc.cluster.local
```
通常，后面的 `svc.cluster.local` 可以省略。

对于同一个namespace，使用 `service-name` 即可。

## Pod的生命周期
Pod的状态有如下几种：
```
Pending: api server已经发出创建Pod的指令，但是Pod中的容器还没有全部创建完成，有可能还在拉取镜像
Running: Pod中的容器已经全部创建完成，并且至少有个容器处于运行或者正在启动，或者正在重启状态
Succeeded: Pod中的容器全部成功执行并退出，而且不会重启
Failed: Pod中的容器已经退出，但是至少有个退出失败
Unknown: 由于某种原因，无法获取Pod的信息
```

## nodePort,port和targetPort
- nodePort: 宿主机的端口
- port: service(cluster)的端口，
- targetPort: 容器(Pod)端口，1 to 65535

## Label
通过`spec.selector`字段来指定这个RC管理哪些Pod。新建的RC会管理所有拥有`app:xxxLabel`的Pod。这样的`spec.selector`在Kubernetes中被称作Label Selector。

## PV和PVC
PV是提供存储的，PVC是消费存储的，PVC可以在Deployment中使用。

## Deployment
### Deployment的特性
1.Deployment相当于RC的升级版

2.Deployment内部使用了Replica Set

### Deployment对比RC
Deployment对比RC，最大的一个升级是：能够随时了解当前Pod的部署状态。

### Deployment的应用场景
1.通过创建deployment对象来创建replica set对象，最终创建pod副本对象

2.通过检查deployment来查询部署进度

3.通过deployment来实现pod的更新和回滚（扩容，缩容）

## Service
### 特性
1.service定义了一个服务的访问入口

2.service相当于微服务架构中的一个微服务

3.service和后端pod集群交互是通过label selector实现的

### 总结
传统服务通过k8s的service实现了微服务，不同的service提供不同业务的服务，它们之间彼此独立，一起组成强大而灵活的微服务系统。

## kube proxy
### 作用
kube proxy将来自service的请求转发给后端的pod。

### 特性
service被创建，系统会分配给它一个独立的IP（cluster IP），这个IP是固定的

## Annotation
### 特性
1.中文为注解

2.用来定义资源对象的特殊信息

### 常见的annotation
1.build,release信息

2.docker信息

3.日志库，监控库等资源库的地址信息

## Volumn
### 分类
```
宿主机: emptyDir、HostPath
集群存储: configmap、secret
云存储: awsElasticBlockStore、gcePersistentDisk等
外部存储: glusterfs、Ceph等
```

# 原理架构
## master节点
master是k8s集群的控制节点，每个k8s集群都至少有一个master节点。

master节点由三个组件构成：api server,controller manager,scheduler。

### api server
api server是集群的控制入口，也是资源对象操作的唯一入口。提供了rest风格API供客户端调用。

**api server最核心的功能就是提供了操作资源对象的http restful接口。**

#### 特殊的proxy接口
api server提供的接口中，有一种接口比较特殊，就是proxy接口。发送到proxy接口的请求，会被转发到kubelet进程，由kubelet负责处理请求并响应。

### controller manager
controller manager是集群的控制中心。相当于资源对象的“管家”。

### scheduler
scheduler是集群的调度中心。负责Pod的调度。

## node节点
node节点时集群的工作节点。负责接收master节点的指令并在本地执行。

node节点由三个组件构成：kubelet,proxy,docker

### kubelet
kubelet负责Pod的创建，启动，销毁。

### kube proxy
kube proxy负责service资源对象的实现，提供Pod的负载均衡。

### docker 
docker负责容器的创建和管理。

## etcd
etcd是k8s集群的存储中心，负责存储k8s各种资源的信息。

## k8s客户端
用户通过k8s客户端和k8s集群进行交互。常见的客户端有：kubectl工具,编程语言的sdk。

# 配置使用
## 默认的NodePort范围
```
30000-32767
```

## 从外部访问k8s内的服务
### kube-proxy方式
master节点执行：
```
kubectl proxy --address=0.0.0.0 --port=8888 --accept-hosts='^*$'
```
接口格式：
```
http://master-ip:port/api/v1/namespaces/<namespace>/services/<service-name>:<service-port-name>/proxy/
```
示例
```
http://11.76.32.31:8888/api/v1/namespaces/default/services/nginx:nginx-port/proxy/
```

### NodePort
> NodePort服务是让外部流量直接访问服务的最原始方式。NodePort，顾名思义，在所有的节点（虚拟机）上开放指定的端口，所有发送到这个端口的流量都会直接转发到服务。

yaml文件示例：
```
apiVersion: v1
kind: Service
metadata:  
 name: my-nodeport-service
selector:   
 app: my-app
spec:
 type: NodePort
 ports:  
 - name: http
  port: 80
  targetPort: 80
  nodePort: 30036
  protocol: TCP
```
缺点：
```
1.一个端口只能供一个服务使用；

2.只能使用30000–32767的端口；

3.如果节点 / 虚拟机的IP地址发生变化，需要进行处理。
```

### Ingress
- `traefik-rbac.yaml`
```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
    - extensions
    resources:
    - ingresses/status
    verbs:
    - update
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```

- `traefik-ds.yaml`
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
      name: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik:v1.7
        name: traefik-ingress-lb
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
          hostPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
```

- `traefik-ui.yaml`
```
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  - host: traefik-ui.minikube
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

## 常用命令
- 查看服务器和客户端版本
```
kubectl version
```

- 给node打tag
```
kubectl label nodes k8s-m2 node=w2 
```

- 删除node的tag
```
kubectl label nodes <node-name> <label-key>-
```

- 查看node的标签
```
kubectl get nodes --show-labels
```

- 查看节点列表详细信息
```
kubectl get nodes -o wide
```

- 查看节点详细信息
```
kubectl describe node k8s-m2
```

- 查看日志
```
kubectl logs -n work podName
```
`-f` 查看实时日志。

- 查看所有可用的机器IP
```
ps -ef|grep kube-apiserver
```
查看`service-cluster-ip-range`

- 查看可用的端口范围
```
ps -ef|grep kube-apiserver
```
默认`30000-32767`。

- 查看所有名称空间
```
kubectl get namespaces
```

- 创建名称空间
```
kubectl create namespace dev
```

- 查看集群各组件状态
```
kubectl get cs
```

- 编辑正在运行的deployment
```
kubectl edit deployment/service_name
# service_name可以通过 `kubectl get deployments` 获取
```

- 查看当前的leader
```
kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml
```

- 查看支持的api版本
```
kubectl api-versions
```

- 创建令牌
```
kubectl create sa dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')

DASHBOARD_LOGIN_TOKEN=$(kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}')

echo ${DASHBOARD_LOGIN_TOKEN}
```

## Pod绑定主机名
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: busybox-subdomain
  containers:
  name: busybox
  - image: busybox
```
注意这个hostname和kubectl get pod获取的Name无关。

## 使用 HostAliases 添加hosts解析
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: task-backend
  name: task-backend
  namespace: dev
spec:
  selector:
    matchLabels:
      app: task-backend
  replicas: 2
  template:
    metadata:
      labels:
        app: task-backend
    spec:
      nodeSelector:
        node: w2
      hostAliases:
      - ip: 11.76.32.22
        hostnames:
        - "11-76-32-22"
      - ip: 11.76.32.23
        hostnames:
        - "11-76-32-23"
      hostname: task-backend
```

## 添加外部DNS服务器
### 使用resolv.conf指定
- 编辑文件`/etc/resolv.conf`
```
nameserver 11.76.32.6
```
注意：Pod重新创建后，DNS才生效。

- 查看coredns的配置
```
kubectl -n kube-system get configmap coredns -o yaml
```

- 在线修改coredns的配置
```
kubectl -n kube-system edit configmap coredns
```
注意：添加外部DNS，不重启coredns是不生效的。

### 直接指定上游DNS服务器
```
data:
  #upstreamNameservers: |
  #  ["11.76.32.2"]
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            upstream
            fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . 11.76.32.6
        cache 30
        loop
        reload
        loadbalance
    }
```

## 设置镜像拉取策略
```
      containers:
      - image: traefik
        name: traefik-ingress-lb
        imagePullPolicy: IfNotPresent
```

## IPVS
- 查看ipvs规则
```
ipvsadm -Ln
```

- 检查内核是否支持ipvs
```
lsmod | grep ip_vs
```
```
p_vs_sh               12688  0 
ip_vs_wrr              12697  0 
ip_vs_rr               12600  188 
ip_vs                 141092  194 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          133387  7 ip_vs,nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_ipv4
libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```

- 检查ipvs是否编译进了内核
```
grep -e ipvs -e nf_conntrack_ipv4 /lib/modules/$(uname -r)/modules.builtin
```
如果编译成内核，则得到如下结果
```
kernel/net/ipv4/netfilter/nf_conntrack_ipv4.ko
kernel/net/netfilter/ipvs/ip_vs.ko
kernel/net/netfilter/ipvs/ip_vs_rr.ko
kernel/net/netfilter/ipvs/ip_vs_wrr.ko
kernel/net/netfilter/ipvs/ip_vs_lc.ko
kernel/net/netfilter/ipvs/ip_vs_wlc.ko
kernel/net/netfilter/ipvs/ip_vs_fo.ko
kernel/net/netfilter/ipvs/ip_vs_ovf.ko
kernel/net/netfilter/ipvs/ip_vs_lblc.ko
kernel/net/netfilter/ipvs/ip_vs_lblcr.ko
kernel/net/netfilter/ipvs/ip_vs_dh.ko
kernel/net/netfilter/ipvs/ip_vs_sh.ko
kernel/net/netfilter/ipvs/ip_vs_sed.ko
kernel/net/netfilter/ipvs/ip_vs_nq.ko
kernel/net/netfilter/ipvs/ip_vs_ftp.ko
```

## 使用nfs做共享盘
1.所有节点都要安装nfs-utils
```
yum install -y nfs-utils
```

2.编写yaml文件
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: register
  name: register
  namespace: prod
spec:
  selector:
    matchLabels:
      app: register
  replicas: 1
  template:
    metadata:
      labels:
        app: register
    spec:
      nodeSelector:
        node: w1
      containers:
      - name: register
        image: register:v1.0.1
        ports:
        - containerPort: 13001
        volumeMounts:
        - mountPath: /data/nfs
          name: nfs-server
        - mountPath: /data/logs
          name: register-log
      volumes:
      - name: nfs-server
        nfs:
          server: 11.76.32.10
          path: /data/nfs/register
      - name: register-log
        hostPath:
          path: /data/logs/register
```

# 文档资料
http://docs.kubernetes.org.cn/537.html