# docker 镜像使用

## 列出镜像列表

```shell
docker images ls
```

## 获取一个新的镜像

```shell
docker pull ubuntu
```

## 查找镜像

```shell
docker search node
```

## 删除镜像

```shell
docker rmi hello-world
```

## 从镜像创建容器

```shell
~$ docker run -i -t ubuntu:latest /bin/bash
root@2069543a26b6:/#
```

参数说明：

- `-i`: 交互式操作。
- `-t`: 终端。
- `ubuntu:latest`: 这是指用 ubuntu 最新版本镜像为基础来启动容器。
- `/bin/bash`: 放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

## 创建镜像

当我们从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

- 从已经创建的容器中更新镜像，并且提交这个镜像
- 使用 Dockerfile 来创建一个新的镜像

### 更新镜像

更新镜像之前，我们需要使用镜像来创建一个容器。

```shell
~$ docker run -it ubuntu:latest /bin/bash
root@56715f5ac67f:/# ls tmp/

root@56715f5ac67f:/# echo hello-world > tmp/index.html
root@56715f5ac67f:/# ls tmp/
index.html
```

然后修改容器中的内容，修改完成输入 `exit` 命令来退出这个容器。

> 如果修改的时候没有vim，需要先执行 `apt-get update` 用于同步 `/etc/apt/sources.list` 和 `/etc/apt/sources.list.d` 中列出的源的索引。
>
> 如果update速度过慢，可以修改为国内镜像源：
>
> - [中科大开源镜像站](http://mirrors.ustc.edu.cn)
>
> - [清华大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn)
>
> - [北京外国语大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn)
>
> - [上海交大开源镜像站](https://mirrors.tuna.tsinghua.edu.cn)
>
> ```shell
> sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
> ```

更改的容器。我们可以通过命令 `docker commit` 来提交容器副本：

```shell
~$ docker commit  -m "update file" -a xiamu33 56715f5ac67f ubuntu:test
sha256:fae557168bf469b6204ce0d3edc60f24e828c910dc584915c2c8a2ab517b4ebe
```

参数说明：

- `-m`: 提交的描述信息
- `-a`: 指定镜像作者
- `56715f5ac67f`：容器 ID，这里也可以使用自己指定或docker自动生成的NAMES
- `ubuntu:test`: 指定要创建的目标镜像名

```shell
~$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
ubuntu                test                fae557168bf4        17 seconds ago      167MB
ubuntu                latest              f643c72bc252        3 days ago          72.9MB
```

可以看到我们已经在 `ubuntu:latest` 的基础上更新了一份 id 为 `fae557168bf4` 的新镜像 `ubuntu:test`

使用新镜像来创建容器：

```shell
~$ docker run -it ubuntu:test /bin/bash
root@628d103aed32:/# ls tmp/
index.html
root@628d103aed32:/# cat tmp/index.html
hello-world
```

可以看到新镜像里已经有了我们之前的修改

### 构建镜像

我们使用 `docker build` 命令从零开始创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。Dockerfile 文件之后再谈。

```shell
docker build -t node-demo:1.0.0 .
```

参数说明：

- `-t` ：指定要创建的目标镜像名，这里即为 node-demo:1.0.0
- `.` ：Dockerfile 文件所在目录，可以指定 Dockerfile 的绝对路径

```shell
~$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
node-demo             1.0.0               3b7cf97f35ea        38 seconds ago      922MB
node                  latest              2d840844f8e7        4 days ago          935MB
```

### 设置镜像标签

```shell
docker tag node-demo:1.0.0 xiamu33/node-demo:1.0.0
```

```shell
~$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
node-demo             1.0.0               3b7cf97f35ea        5 minutes ago       922MB
xiamu33/node-demo     1.0.0               3b7cf97f35ea        5 minutes ago       922MB
node                  latest              2d840844f8e7        4 days ago          935MB

```

可以看到现在两个 `IMAGE ID` 一致的镜像，实际上它们是一个镜像的两个别名，在想重命名某个镜像时可以先修改镜像标签，再将旧标签删除
