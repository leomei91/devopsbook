[toc]

# 基本概念
## docker的分层特性
docker是容器的一种，docker是分层的。最顶层叫做容器层，是可写的，下面的叫做镜像层，是只读的。

## docker镜像的缓存机制
Docker 会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在，就直接使用，无需重新创建。

如果我们希望在构建镜像时不使用缓存，可以在 docker build 命令中加上 --no-cache 参数。

Dockerfile 中每一个指令都会创建一个镜像层，上层是依赖于下层的。无论什么时候，只要某一层发生变化，其上面所有层的缓存都会失效

说明：

除了构建时使用缓存，Docker 在下载镜像时也会使用。例如我们下载 httpd 镜像。

# 架构原理
## docker镜像的原理
docker镜像分为多层：kernel->base image->image1->container（由低到高）

其中只有container层是可写的，所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。

# 安装部署
## yum
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
```
yum install -y docker-ce
```
```
systemctl enable docker
systemctl start docker
```

# 配置使用
## 常用命令
- 在容器外指定用户执行命令
```
sudo docker exec --user username image_name cmd
```

- 查看所有容器状态
```
sudo docker ps -a
```

- 启动docker
```
sudo docker run -dit --name cname1 cname:v1 /bin/bash
```

- 交互式进入docker
```
sudo docker exec -it cname1 /bin/bash
```

- 将现有容器导出到docker仓库
```
sudo docker commit -a "mei" -m "cname" 47c7ce271d12 cname1:v2
```

- 停止容器
```
sudo docker stop cname1
```

- 删除容器
```
sudo docker rm cname1
```

- 在容器外部执行命令
```
sudo docker exec cname1 cmd
```

- 挂载共享卷
```
sudo docker -v local:container
```

- 拷贝镜像到另一个节点
```
docker save -o xxx.tar.gz image
scp
docker load < xxx.tar.gz
```
或者
```
docker export  
docker import
```

检验
```
docker imaegs
```

总结一下docker save和docker export的区别：
```
docker save保存的是镜像（image），docker export保存的是容器（container）；
docker load用来载入镜像包，docker import用来载入容器包，但两者都会恢复为镜像；
docker load不能对载入的镜像重命名，而docker import可以为镜像指定新名称。
```

- 查看详细信息
```
docker inspect image|container
```

- 查看镜像构建过程（Dockerfile信息）
```
docker history centos:7
```

- 给镜像打tag
```
docker tag centos 11.76.32.1/common/centos_httpd
```

- 查看版本
```
docker version
```

- 查看docker信息
```
docker info
```

- docker命令参考

https://www.runoob.com/docker/docker-command-manual.html

- 一张图搞定docker命令
![](https://res.cloudinary.com/dkkg9pm0i/image/upload/v1557556679/k8s/upload-ueditor-image-20170608-1496889671778038814.png)

## 第三方docker镜像源
- 阿里
```
docker pull nginx --image-repository registry.aliyuncs.com/google_containers 
```

## Dockerfile
### 最佳实践
- 1.编写`.dockerignore`文件，忽略不必要的文件
- 2.容器只运行单个应用
- 3.尽量将多个RUN指令合并成一个RUN指令
- 4.基础镜像要指定标签，没必要用最新的
- 5.每个 RUN 指令后删除多余文件，避免容器过大
- 6.选择合适的基础镜像(alpine 版本最好)
- 7.设置 WORKDIR 和 CMD
- 8.在 entrypoint 脚本中使用 exec 运行程序

### 调试Dockerfile
构建镜像的时候，如果有错误会打印出来。这时，需要执行：
```
docker run -it 出错的镜像ID
```
进入出错的镜像内部进行调试。

### 指令讲解
- COPY

拷贝文件到容器内，支持linux通配符。比如，拷贝abc开头的目录下的所有文件到容器的/abc目录下：
```
COPY abc* /abc/
```
- WORKDIR

设置服务运行的根目录。
```
WORKDIR /data/logs
```

- RUN

在容器中运行指定的命令。

- CMD

容器启动后运行指定的命令。
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。
```
CMD ["/bin/sh","/data/shell/restart_service.sh"]
```

- ENTRYPOINT

设置容器启动时运行的命令。
Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT。

- CMD，RUN和ENTRYPOINT

https://www.cnblogs.com/CloudMan6/p/6875834.html

### java应用Dockerfile示例
- Dockerfile
```
FROM centos:7

MAINTAINER xxx@163.com

ADD jdk1.8.0_211 /usr/local/jdk1.8.0_211

COPY restart_service.sh /data/shell/
COPY register-1.0.0.jar /data/jars/
COPY application-prod.properties /data/conf/

RUN rm -rf /etc/localtime && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN yum install -y iproute telnet kde-l10n-Chinese glibc-common
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

RUN echo "Asia/Shanghai" > /etc/timezone \
    && mkdir -p /data/logs > /dev/null 

WORKDIR /data/logs

ENV LC_ALL zh_CN.utf8
ENV JAVA_HOME /usr/local/jdk1.8.0_211
ENV PATH $PATH:$JAVA_HOME/bin
ENV CLASSPATH $JAVA_HOME/lib/

USER root:root

CMD ["/bin/sh","/data/shell/restart_service.sh"]
```

- restart_service.sh
```
#!/bin/bash
#coding:utf-8
#Usage: sh restart_service.sh
#

#常量
RUN_JAVA=/usr/local/jdk1.8.0_211/bin/java

JAR_HOME=/data/jars
JAR=register-1.0.0.jar
JAR_FILE=$JAR_HOME/$JAR

CONFIG_HOME=/data/conf
CONFIG=application-prod.properties
CONFIG_FILE=$CONFIG_HOME/$CONFIG

IP2=`hostname -I`
IP=`echo $IP2|sed -r 's/^[ \t]+(.*)[ \t]+$//g'`
LOG=`echo $JAR|sed -n 's/jar/log/p'`
LOG_FILE=/data/logs/$IP-$LOG
GC_FILE=`echo $JAR|sed -n 's/jar/gc/p'`

#启动命令
$RUN_JAVA -jar -Xms8G -Xmx8G $JAR_FILE \
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause -XX:+PrintGCApplicationStoppedTime \
-Xloggc:/data/logs/$GC_FILE.log \
--spring.profiles.active=prod \
--spring.config.location=$CONFIG_FILE > $LOG_FILE 2>&1
```

- 目录结构
```
.
├── application-prod.properties
├── Dockerfile
├── register-1.0.0.jar
├── jdk1.8.0_211
└── restart_service.sh
```

- 构建镜像
```
docker build -t register:v1.0.0 .
```

- 启动容器
```
docker run -d --name register1 register:v1.0.0
```

# 故障解决
- 1.`kernel:unregister_netdevice: waiting for lo to become free. Usage count = 1` 

解决：
```
systemctl stop rsyslog
systemctl disable rsyslog
```

# 最佳实践
- 停止所有容器
```
docker stop `docker ps -qa`
```

- 删除所有容器
```
docker rm `docker ps -qa`
```

- 删除容器镜像
```
docker rmi <image id>
```

- 删除所有镜像
```
docker rmi `docker images -q`
```

- 修改docker0网桥的默认网段
```
vim /etc/docker/daemon.json
```
```
{
  "registry-mirrors": ["https://vcmd7xzk.mirror.aliyuncs.com"],
  "bip": "172.27.1.1/24"
}
```
```
systemctl restart docker
```