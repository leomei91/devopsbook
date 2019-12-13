
# 安装部署
## yum方式
1.下载最新的yum源
```
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2.安装nginx
```
yum install nginx -y
```

3.加入开机自启
```
systemctl enable nginx
```

4.检查nginx配置
```
nginx -t
```

5.启动
```
systemctl enable nginx
systemctl start nginx
```

6.查看nginx状态
```
systemctl status nginx
# or
ps -ef|grep -v grep|grep nginx
```