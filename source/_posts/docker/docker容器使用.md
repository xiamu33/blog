---
title: docker 容器使用
date: 2020-12-01 00:37:37
tags: docker
categories: Docker
---

## 启动容器

```shell
~$ docker run -t -i ubuntu /bin/bash
root@11a6df69c3cf:/#
```

参数说明：

- `-t`: 让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上。
- `-i`: 让容器的标准输入保持打开。
- `ubuntu:latest`: 这是指用 ubuntu 最新版本镜像为基础来启动容器。
- `/bin/bash`: 放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash

> 启动容器若本地没有改镜像，会自动从[docker hub](https://hub.docker.com/)上拉取

## 查看运行的容器

```shell
~$ docker ps -a
CONTAINER ID        IMAGE         COMMAND                CREATED             STATUS                               PORTS        NAMES
11a6df69c3cf        ubuntu        "/bin/bash"            6 minutes ago       Exited (127) 3 minutes ago                        competent_hodgkin
```

参数说明：

- `-a`: 可以查看已停止的容器。

```shell
~$ docker start 11a6df69c3cf
11a6df69c3cf
```

## 后台运行

```shell
docker run -it -d --name node-demo node-demo:1.0.0 /bin/bash
```

参数说明：

- `-d`: 在后台运行容器。

## 停止容器

```shell
docker stop node-demo
```

## 删除容器

```shell
docker rm [OPTIONS] node-demo
```

OPTIONS：

- `-f`(`--force`): 强制删除容器（即使容器还在运行）。
- `-l`(`--link`): 删除指定的链接。
- `-v`(`--volumes`): 删除与容器关联的匿名卷。

## 进入启动中的容器

```shell
docker exec -it node-demo /bin/bash
```

## 查看容器资源使用情况

```shell
docker stats [OPTIONS] [CONTAINER...]
```

OPTIONS：

- `-a`(`--all`): 显示所有容器。
- `--format`: 格式化打印的图像。
- `--no-stream`: 禁用流统计信息，仅输出当前资源使用情况。
- `--no-trunc`: 不截短输出信息，也会显示完整的CONTAINER ID。

## 导出导入容器

### 导出容器快照

```shell
~$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS                 PORTS          NAMES
4163d0029b09        node-demo:1.0.0        "docker-entrypoint.s…"   16 seconds ago      Up 15 seconds          3000/tcp       node-demo
~$ mkdir docker
~$ docker export node-demo > ~/docker/node-demo.tar
~$ ls docker/
node-demo.tar
```

### 导入容器快照为镜像

```shell
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

#### 从url导入

```shell
docker import http://example.com/exampleimage.tgz example/imagerepo
```

#### 从本地快照文件导入

```shell
docker import docker/node-demo.tar - node-demo-import:1.0.0
```

#### 通过管道导入

```shell
cat docker/node-demo.tar | docker import - node-demo-import:2.0.0
```
