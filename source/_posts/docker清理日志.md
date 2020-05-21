---
title: docker清理日志
date: 2020-05-19 22:14:34
tags:
---

# docker清理日志
------

## 发现服务器空间不够

今天通过Gitlab CI部署后端服务时，出现卡死，Gitlab也打不开了。
进入服务器发现Gitlab runner的容器一直再运行，打开容器日志显示空间不够，运行df命令查看空间。
```shell
df -h
```
发现系统根文件已经占用了100%，仅剩余43M空间。
不断追踪使用df命令，还可以使用
```shell
du -sh*
```
发现`/var/lib/docker/comtainners`中有一个文件占了24G的空间，根据container的名称，发现是Gitlab的container。
大文件名为'6f60a89f274d51e7482340a9725207a0f5057ce75c673db93b540da16627243c_json.log'，是docker的容器日志文件，一个文件就有24G。

## 清理docker容器日志[1]

### rm 删除日志文件

如果docker容器正在运行，那么使用rm -rf方式删除日志后，通过df -h会发现磁盘空间并没有释放。
原因是在Linux或者Unix系统中，通过rm -rf或者文件管理器删除文件，将会从文件系统的目录结构上解除链接（unlink）。
如果文件是被打开的（有一个进程正在使用），那么进程将仍然可以读取该文件，磁盘空间也一直被占用。
如果要采用rm -rf 命令，需要重启docker。

### 直接删除日志文件内容

直接删除日志中的内容，可用如下命令
```shell
cat /dev/null > *_json.log
```
写入后，空间被释放了。

## 提前设置容器日志大小

在docker-compose的配置文件中，加上日志的大小限制，例如：
```shell
nginx: 
  image: nginx:1.12.1 
  restart: always 
  logging: 
    driver: “json-file” 
    options: 
      max-size: “5g”
```
这样就给该容器的日志文件做了5G大小的限制。

------

作者 [EnxZhou][2]

[1]: https://blog.csdn.net/yjk13703623757/article/details/80283729
[2]: https://enxzhou.github.io