---
layout: post
title: "Docker 命令集"
date: 2020-09-18 16:23
comments: true
reward: false
top: false
brief: ""
tags: 
	- docker
key: "docker"
---
### 获取镜像
* 从Docker镜像仓库获取镜像的命令是docker pull。其命令格式为：
```markdown

docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

```
> *注意 这里的仓库名是两段式名称，即<用户名>/<软件名>。对于Docker Hub，如果不给出用户名，则默认为library，也就是官方镜像。

### 列出镜像
* 可以通过 docker images [ls] 列出已经下载的所有顶层镜像。其命令格式为：
```markdown
docker images [选项] [仓库名[:标签]]

选项：
    -a, --all           希望显示包括中间层镜像在内的所有镜像的话，需要加 -a 参数。
        --digests       摘要
    -f, --filter value  根据提供的条件过滤输出
                        - dangling =(true|false)     显示*[虚悬镜像]:详细解释在下方
                        - label =<key> or label=<key>=<value>    如果镜像构建时，定义了label，可以通过label来过滤
                        - before =(<image-name>[:tag]|<image-id>|<image@digest>)     希望看到某个镜像之后建立的镜像
                        - since =(<image-name>[:tag]|<image-id>|<image@digest>)      希望查看某个位置之前的镜像
        --format string 以特定格式显示
        --help
        --no-trunc      不要截断输出
    -q, --quiet         仅显示IMAGE ID，‘注意可以和-f搭配组合以完成很强大的功能，看到过滤器后，可以多注意一下它们的用法’
```
* 镜像体积：显示的是镜像下载到本地后，展开后的各层所占空间的总和。需注意镜像体积总和并非是所有镜像实际硬盘消耗，因为docker镜像是多层存储结构，并可以继承、复用相同的基础镜像。可以用下面的命令
```
docker images ls [仓库名[:标签]]
```
* 虚悬镜像（dangling image）：镜像原本是有镜像名和标签的，如ubuntu:16.04，随着官方镜像维护，发布新版本后，重新docker pull ubuntu:16.04时，这个镜像名被转移到了新下载的镜像身上，而旧的镜像上这个名称则被取消，从而成为了<none>.这类无标签镜像也被称为 虚悬镜像。可以用下面的命令
```
docker images ls -f dangling=true
```
* 一般来说虚悬镜像已经失去存在必要，可以删除，可以用下面的命令
```
docker images prune
```
* 中间层镜像：为了加速镜像构建、重复利用资源，docker会利用中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。可以用下面的命令
```
docker images ls -a
```
* 列出部分镜像
```
docker images ls [仓库名[:标签]]

docker images ls -f since=仓库名:标签

docker images ls -f label=com.example.version=0.1
```
* 以特定格式显示，我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这就用到了 Go 的模板语法。
```
docker images ls --format "{{.ID}}: {{.Repository}}"

docker images ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

