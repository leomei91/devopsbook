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
--bwlimit=RATE              limit socket I/O bandwidth
--exclude=PATTERN           exclude files matching PATTERN
--progress                  show progress during transfer
--partial                   keep partially transferred files
```

- 同步本地目录A下的文件到本地目录B
```
rsync -avH /A/ /B/
```

- 拷贝目录A到本地目录B下
```
rsync -avH A /B/
```

- 同步本地目录A下的文件到本地目录B，同时排除a,c文件
```
rsync -avH --exclude={a,c} /A/ /B/
```

- 将目录`A`拷贝到`11.76.32.1`的`B`目录下
```
rsync -avP '-e ssh -p 52222' A 11.76.32.1:/B/
```