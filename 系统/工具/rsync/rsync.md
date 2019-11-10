# 基本概念
## rsync
rsync是一个文件同步工具。通常用于不同节点间的文件同步，也可以用于本地文件同步。

## 特性
1. rsync只同步远程节点不存在的文件
2. 如果文件内容有更新，rsync只同步更新的部分

# 配置使用
- 选项说明
```
-a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
-z, --compress              compress file data during the transfer
-v, --verbose               increase verbosity
-P                          same as --partial --progress
--bwlimit=RATE          limit socket I/O bandwidth
--exclude=PATTERN       exclude files matching PATTERN
--progress              show progress during transfer
--partial               keep partially transferred files
```

- 文件同步，并改变文件属主
```
rsync -avh --chown=USER:GROUP /foo user@remote-host:/tmp/
```
- ssh协议，非22端口文件同步
```
rsync -avh '-e ssh -p 52222' local-file user@remote-host:remote-file
```