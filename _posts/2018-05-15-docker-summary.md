---
layout: post
title: docker使用速览
date: 2018-05-15 20:34
tags: [docker, linux]
feature-img: "https://images.pexels.com/photos/163130/keyboard-black-notebook-input-163130.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=650&w=940"
---


使用 *docker* 也有一段时间了，刚好有空做个简单的总结?。

<!--more-->


## 基础

- 安装

借助相关平台的软件管理包进行安装，例如，[ubuntu安装docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/#uninstall-old-versions) 的官方文档。

安装完毕，必须开启服务后，方能使用 `docker` 命令。

```bash
# 开启服务
$ sudo systemctl start docker

# 设置开机启动
$ sudo systemctl enable docker
```

- 配置

旧有 `/etc/default/docker` 已弃用，改为 `/etc/docker/daemon.json`，该文件默认不存在，需要手动创建。

鉴于天朝特殊的国情，使用时必须手动指定为国内镜像源

```bash
# 创建配置
$ cat > /etc/docker/daemon.json << EOF
{  
	"registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF

# 重启服务
$ sudo systemctl restart docker
```

- 用户与群组

默认仅有 `root` 用户可以使用 **docker** 命令，这对日常使用不太友好，官方推荐增加群组来解决，详细请参考 [docker安装后必做的的设置](https://docs.docker.com/install/linux/linux-postinstall/)。

```bash
# 创建 docker 群组
$ sudo groupadd docker

# 把当前用户添加到 docker 组
$ sudo usermod -aG docker $USER
```


## 使用

主要是命令的使用，分两种类型：`Management Commands` 和 `Commands`，详情可执行 `docker --help`查看。

- 创建容器

创建名为 *test* 的最新版 *ubuntu* 交互式容器

```bash
$ docker container run --name test -it ubuntu /bin/bash
```

创建名为 *test2* 的 *ubuntu16.04* 守护式容器

```bash
$ docker container run --name test2 \
  -d ubuntu:16.04 /bin/sh \
	-c "while true; do echo hello world; sleep 1; done"
```

创建名为 *test3* 的 *nginx* 容器，映射80端口至主机的8080端口，且退出时自动删除

```bash
$ docker container run --rm --name test3 -p 8080:80 -d nginx
```

- 管理容器

查看现有容器

```bash
# 查看运行中的
$ docker container ls

# 查看所有容器
$ docker container ls -a
```

开启、关闭容器

```bash
$ docker container start 容器名/ID

$ docker container stop 容器名/ID
```

查看容器资源使用状态

```bash
$ docker container stats 容器名/ID
```

查看容器内的进程

```bash
$ docker container top 容器名/ID
```

查看容器明细

```bash
$ docker container inspect 容器名/ID
```

删除容器

```bash
# 删除单个
$ docker container rm 容器名/ID

# 删除所有容器
$ docker container rm $(docker container ls -qa)

# 强制删除运行中的容器
$ docker container rm -f 容器名/ID
```

文件拷贝

```bash
# 拷贝到容器
$ docker container cp 本地文件 容器名/ID:容器文件

# 拷贝到host
$ docker container cp 容器名/ID:容器文件 本地文件
```

在现有容器执行命令

```bash
# 打开交互式终端
$ docker container exec -it 容器名/ID bash
```

查看容器日志

```bash
$ docker logs -ft 容器名/ID
```

仅对有日志输出的容器有效，如果想指定容器日志重定向到系统日志，可在创建时，添加 `--log-driver="syslog"`。

## 数据

容器启动后，数据如何持续的保存呢？

- [绑定挂载](https://docs.docker.com/storage/bind-mounts/)

**Bind mounts** ，绑定挂载，使用上较为简单，类似 *linux* 的 `mount` 命令，举个栗子

```bash
$ docker container run --rm \
  --name test3 \
	-v /data:/usr/share/nginx/html \
	-p 8080:80 -d nginx
```

把主机的 `/data` 与 容器内的 *nginx* web主目录 `/usr/share/nginx/html` 绑定，访问 *web* 服务时，则显示主机内 */data* 目录下的内容。

绑定挂载 可同时挂载多个，其优势是简单易用，支持挂载 **目录** 和 **文件**；缺点是多个容器挂载同一个主机目录时，会有冲突产生，对数据的完整性无法保证。


- [数据卷](https://docs.docker.com/storage/volumes/)

**volumes**，数据卷，较之绑定挂载，有以下优点：

1. 易于备份与迁移
2. 可通过命令行、API进行管理
3. 支持 linux 与 windows 容器
4. 多容器间能安全共享
5. 支持保存在远程主机
6. 支持容器创建时自动初始化

同样支持 `-v` 和 `--mount` ，推荐使用后者，除了易用性更高，它还支持更详细的挂载参数设定。

数据卷可单独创建

```bash
# 创建 my-vol 数据卷
$ docker volume create my-vol

# 查看数据卷
$ docker volume ls
local               my-vol

# 查看数据卷明细
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

> 调整数据卷存储位置，添加键值 {'data-root': '你指定的目录'} 到 /etc/docker/daemon.json，重启服务。

创建时容器时指定数据卷

```bash
# -v 参数形式
$ docker container run --name test3 \
  -v my-vol:/usr/share/nginx/html \
	-p 8080:80 -d nginx

# --mount flag形式
$ docker container run --name test3 \
  --mount source=myvol,target=/usr/share/nginx/html \
	-p 8080:80 -d nginx
```

如果事先没手动创建数据卷，则创建容器时会自行帮你创建数据卷。

当容器关闭时，数据卷仍然存在，如不需要了，手动 `docker volume rm 数据卷名` 删除之。

容器间共享时，则需要指定从哪个容器共享，使用 `volumes-from 容器名/ID`。


- [临时挂载](https://docs.docker.com/storage/tmpfs/)

对于敏感数据，既不想保存在 *Host* 也不想保存在数据卷，则可以使用 临时挂载，仅支持 Linux 系统。

```bash
# --mount 形式
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest

# --tmpfs
$ docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

数据仅保存在主机的内存中，当容器关闭时，数据也随之从内存中清除。


## 网络

解决存储问题后，那容器间如何进行通信呢？默认提供4种网络模式，想快速了解可参考 [docker网络概览](https://docs.docker.com/network/)。

- [Bridge网络](https://docs.docker.com/network/bridge/#use-the-default-bridge-network)

网桥模式，默认指定的模式，若创建新容器时，不手动指定网络，则默认的都是 「网桥模式」。

```bash
# 查看 docker 网络
$ docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
34c23f6311e0        bridge               bridge              local
7eb09c7af16d        host                 host                local
6a7aa40ee24f        none                 null                local

# 查看 bridge 网络详细信息
$ docker network inspect bridge
```

用户可创建 [自定义网桥](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks)，与默认网桥的区别是：

1. 提供更好的隔离与通信
2. 提供自动化的DNS解析，使用容器名即可连接
3. 支持网络热插拔，随时连接与断开
4. 可单独定制网桥参数

docker 服务启动后，默认创建 `docker0` 的网卡，即为默认网桥的设定；如有 *自定义网桥* ，则使用 `br-xxxx` 命名的网卡。

使用默认网桥的所有容器，均可通过 **ip地址** 进行通信；自定义网桥的，额外可以通过容器名访问，秘密就在于容器内的 `/etc/hosts` 文件。

如需要修改 *docker* 服务的某些参数，如 [开启IPV6](https://docs.docker.com/config/daemon/ipv6/)，[开启防火墙](https://docs.docker.com/network/iptables/)，[定义容器IP](https://docs.docker.com/config/containers/container-networking/#proxy-server) 和 [使用代理](https://docs.docker.com/network/proxy/) 等等，在 `/etc/docker/daemon.json` 配置文件中修改即可。


- [Host网络](https://docs.docker.com/network/host/) 与 None网络

主机网络，与 `virtualbox` 虚拟机的 **Host-Only** 类似，容器网络不与主机网络隔离，而是直接相连，并接管主机网络的端口。如

```bash
# 创建 nginx 容器
$ docker container run -d --name nginx --network host nginx
```

此时直接访问主机的 `80` 端口，则显示容器 *nginx* 的 *web* 内容。

在此模式下，修改容器内的部分网络设置，会直接影响到主机的网络。更详细的实践参考 [这里](https://docs.docker.com/network/network-tutorial-host/)。

**None网络** 则非常简单，容器内只有 `lo` 网卡，无法与主机、其他容器通信，适用于使用敏感数据的容器。

- Overlay网络

允许在多个 *docker* 服务端之间进行通信，即跨主机交流。 *docker* 默认使用 `socket`，如需要跨主机使用时，则必须更改为 `tcp` 形式，具体操作参考 [Daemon socket option](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option)。

阅读 [Overlay概览](https://docs.docker.com/network/overlay/) 与 [Overlay示例](https://docs.docker.com/network/network-tutorial-overlay/) 了解更多吧。


- Macvlan网络

为解决某些应用需要监控网络流量，使用 *Macvlan* 网络给予容器一个 *MAC* 地址来实现之；详情见 [概览](https://docs.docker.com/network/macvlan/) 与 [示例](https://docs.docker.com/network/network-tutorial-macvlan/) 。

- 传统的容器间连接

先来看一个 *nginx* 容器，暴露指定端口到主机

```bash
# 指定某个端口
$ docker container run -d -p 8080:80 nginx

# 指定端口范围，随机选一个
$ docker container run -d -p 8080-8088:80 nginx

# 指定IP
$ docker container run -d -p 127.0.0.1:8080:80 nginx

# 指定协议
$ docker container run -d -p 8080/tcp:80 nginx
```

使用默认网桥，暴露指定端口，其实现原理是在主机设定 `iptables` **NAT** 来实现

```bash
$ iptables -L -n | grep 80
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.2           tcp dpt:80
```

如果多容器间需要相互访问端口，都通过上述方式，暴露给主机，再通过主机网络来访问，这就有点蛋疼了。

幸好，*docker* 提供了另一种连接方式，即 `--link`，详细操作参考 [这里](https://docs.docker.com/network/links/#connect-with-the-linking-system)，以下是精简示例

```bash
# 创建一个 db 容器
$ docker container run -d --name db training/postgres

# 链接该 db 容器至 web 容器
$ docker run -d -P --name web \
  --link db:db training/webapp python app.py

# --link <name or id>:alias
```

`--link` 除了能够使容器间端口可以相互访问，链接时亦能自动创建系统变量；可以使用 `-e` 和 `--env` 指定额外的系统变量。

`--link` 这种方式将会被弃用，官方推荐使用 `自定义网桥` 替代之，由此带来新的问题，不支持系统变量在容器间共享，需要借助 **数据卷** 或者是其他方式实现。


## 镜像

一如 `git` 有公共仓库 `github`, **docker** 也有公共仓库 `docker hub`。当创建新容器时，程序会自动检测本地主机是否有对应的镜像，如果没有，则从公共仓库下载，之后再以该镜像为蓝本，创建容器。

镜像的命令较为简单，大概罗列下

```bash
# 查看本地镜像
$ docker images

# 搜索 docker hub 公共仓库镜像
$ docker search nginx

# 删除镜像
$ docker rmi 镜像名/ID
```

- 定制镜像

使用 `Dockerfile` 来定制镜像，举个栗子

```bash
# 创建一个 dev_ub 目录，以建立 ubuntu 16.04 测试专用镜像
$ cd /data/ && mkdir -pv dev_ub

# 创建 Dockerfile
$ cat > Dockerfile << EOF
FROM ubuntu:16.04
RUN apt-get -qyy update && apt-get -qyy install vim curl wget w3m git proxychains && apt-get clean all
EOF

# 编译成本地镜像，注意要在 dev_ub目录，以及最后的那个点
$ docker build -t dev_ub:v1 .
```

查询本地镜像是否有新编译的 **dev_ub** 镜像吧，更多的 *Dockerfile* 的参数解释，参考 [Dockerfile最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)。

- Docker Hub

在 *Docker Hub* 注册账号后，可将定制后的本地镜像推送至公共仓库，具体操作可参考 [这里](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)。

或者是设置从 `github` 上读取 *Dockerfile* 后自动编译定制镜像，具体操作可参考 [这里](https://docs.docker.com/docker-hub/github/)。

- 私人仓库

如果定制的镜像涉及敏感信息，可自动部署 私人仓库，具体的操作可参考 [Deploy a registry server](https://docs.docker.com/registry/deploying/)。

## 小结

实际使用中，还可以进行更多的设置，如 [添加自定义的元数据](https://docs.docker.com/config/labels-custom-metadata/), [docker daemon的启动设置](https://docs.docker.com/config/daemon/systemd/)，以及针对容器进行更细致的设置，如 [限定使用硬件资源](https://docs.docker.com/config/containers/resource_constraints/)。

进阶应用如，`API`, `docker compose` 以及 `docker cloud` 之类的，暂时不表，有空再补充。

书籍推荐阅读 「Docker Cookbook」英文原版?。
