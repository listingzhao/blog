---
title: Docker实践
date: 2019-05-16 19:41:35
tags:
---

# Docker实践

标签（空格分隔）： Docker

------

## 背景知识

Linux容器作为一类操作系统层面的虚拟化技术成果，旨在立足于单一Linux主机交付多套隔离性Linux环境。与虚拟机不同，容器系统并不需要运行特定的访客操作系统。相反，容器共享同一套主机操作系统内核，同时利用访客操作系统的系统库以交付必要的系统功能。由于无需借助于专门的操作系统，因此容器在启动速度上要远远优于虚拟机。
目前整个技术行业已经越来越多地将软件应用程序部署基础由虚拟机转移向容器。之所以出现这种趋势，一大重要原因在于容器能够提供远优于虚拟机的灵活性与使用成本。
Docker 是操作系统级虚拟化，它虚拟出来的环境一般被称为 Docker 容器，而不是虚拟机。Docker 容器直接运行在宿主系统的操作系统内核之上，启动一个新的 Docker 容器能在秒级完成。

## 安装 Docker

[Docker 官方文档][1]详尽的列出了各个系统下的Docker安装说明，请直接点过去，本文不做搬运。
对于 Windows/Mac 用户而言，推荐安装 Docker for Window/Mac，而不是 Docker Toolbox。前者可以直接利用宿主系统的虚拟化机制，拥有更好的性能；后者需要借助 VirtualBox 运行的 Linux 虚拟机。

## 镜像和容器

Docker 基于 Docker 镜像运行容器，通常我们所需大部分镜像都可以在 [hub.docker.com][2] 找到。
确保装好 Docker 打开终端，运行以下命令就可以启动容器：

```
$ docker run ubuntu uname -a
```

不出意外，可以看到这样的输出：

```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
b3e1c725a85f: Pull complete
4daad8bdde31: Pull complete
63fe8c0068a8: Pull complete
4a70713c436f: Pull complete
bd842a2105a8: Pull complete
Digest: sha256:7a64bc9c8843b0a8c8b8a7e4715b7615e4e1b0d8ca3c7e7a76ec8250899c397a
Status: Downloaded newer image for ubuntu:latest
Linux 1e22b7feec86 4.4.27-moby #1 SMP Wed Oct 26 14:21:29 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

`docker run` 命令用来从指定镜像启动容器。由于我本地没有 ubuntu 镜像，Docker 首先会从官方 Hub 下载它；然后启动容器并执行 `uname -a` 命令。这个命令是在 Docker 容器内执行，输出的是容器系统信息。
查看和管理 Docker 镜像及容器，主要有这些命令：

- `docker ps` 查看当前运行的容器 `-a` 列出所有
- `docker rm CONTAINER`  删除容器  `-f` 强制删除
- `docker logs CONTAINER` 查看容器logs `-f` 遵循日志输出
- `docker images` 查看所有镜像
- `docker rmi CONTAINER` 删除镜像

使用 Docker 的最佳实践是保持职责单一，一个容器只提供一个服务。官网主要的服务：

- Nginx (80/443)
- NodeJs（3000）
  考虑到Nginx会修改的频繁些，还是选择把它留在宿主系统，剩余NodeJs服务则改用 Docker 容器来运行。

## 安装 Docker

使用官方 Node.js 镜像存在 npm 版本太低的问题。遇到这种情况，可以在 Docker Hub 看看有无第三方 Docker 镜像能够满足需求，也可以构造自己的镜像。我选择后者。
要构建自己的 Docker 镜像，一般都会选定一个已有的镜像做为基础，再在上面增加自己的修改。我的 DockerFile 如下：

```
FROM marcbachmann/libvips
MAINTAINER Listing Zhao <listing.zhao@gmail.com>

RUN apt-get update

# 修改时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

# 安装依赖
RUN apt-get install -y \
    python \
    curl \
    build-essential

# 安装 Node.js v6.x.x LTS
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash -
RUN apt-get install -y nodejs

# 安装 pm2
RUN npm install -g pm2

# 解决 npm 在 docker 下经常 rename 失败的问题。详见：
    # https://forums.docker.com/t/npm-install-doesnt-complete-inside-docker-container/12640/3
RUN cd $(npm root -g)/npm \
    && npm install fs-extra \
    && sed -i -e s/graceful-fs/fs-extra/ -e s/fs\.rename/fs.move/ ./lib/utils/rename.js
```

这份 DockerFile 作用是在 marcbachmann/libvips 镜像上增加了我需要的 Node.js，将 npm 升级到了 最新，还安装了 pm2。
在 DockerFile 所在目录，执行以下命令就可以构建镜像，并将其推送至 Docker Hub（这里略过注册和登录过程）：

```
docker build -t listing/xzz-node .
docker push listing/xzz-node
```

## Docker Compose

Docker Compose 是一个小工具。我们可以在一个文件里定义多个容器，使用 docker-compose 命令让它们全部运行就绪。Docker Compose 非常适合用来部署 WEB 系统这种需要多个容器配合工作的服务。
如果你使用的是 Docker for Windows/Mac，docker-compose 命令应该直接可用。对于 Linux 平台，请参考[官方文档][1]安装 Docker Compose。
官网目录结构如下：

```
├── code
│   ├── app
│   ├── bin
│   ├── config
│   ├── node_modules
│   ├── package.json
│   ├── process.yml
│   ├── views
├── docker-compose.yml
└── shell
    └── install_app_package.sh
```

我将所有需要持久化存储的文件都放在了宿主系统，例如代码目录（code）,安装脚本(shell),这样数据更加安全，也更易于管理。
shell 目录下的 install_app_package.sh 用来安装网站 npm 依赖，我的宿主系统没有安装 Node.js，运行 npm install 也需要借助 Docker 容器，一行命令搞定：

```
docker run -it --rm -v "$PWD/../code":/app -w "/app" listing/xzz-node npm install --registry=http://registry.npm.taobao.org --production
```

这行命令首先基于前面构建好的镜像运行了一个拥有 Node.js 和 npm 的容器；然后将宿主系统的 `code` 目录映射为容器的 `/app` 目录；再将容器的工作目录设置为 `/app`；最后执行 `npm install` 安装依赖。由于指定了 `--rm` 参数，这个容器在完成工作之后就会被彻底销毁，不留任何痕迹。
`docker-compose.yml` 文件内容如下，它定义了每个容器基于什么镜像运行，映射哪些目录，开放哪些端口：

```
version: '2'
services:
  web:
    ports:
     - "3000:3000"
    image: listing/xzz-node
    container_name: xzz-node-web
    volumes:
     - ./code:/app
    restart: always
    working_dir:
      /app
    entrypoint:
      - pm2
      - start
      - process.yml
      - --no-daemon
```

使用宿主系统的 3000 端口可以访问到 node-web 容器提供的 WEB 服务。
通过 docker-compose up -d 命令就可以在后台启动所有容器。docker ps 可以用来查看各个容器的运行状态：

```
IMAGE                 COMMAND                  PORTS                      NAMES
listing/xzz-node  "pm2 start process.y"   0.0.0.0:3000->3000/tcp       xzz-node-web
```

[1]: https://www.docker.com/products/overview#/install_the_platform
[2]: https://hub.docker.com/
