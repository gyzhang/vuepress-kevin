---
title: 离线安装Docker
date: 2024-12-24 10:11:22
permalink: /pages/de01b5/
sidebar: auto
categories:
  - 随笔
tags:
  - Docker
author: 
  name: Kevin Zhang
  link: https://github.com/gyzhang
---
# 离线安装Docker

在特定的测试环境下，测试服务器可能并不会连接公网，无法在线下载 Docker，并且通过 yum 源在线安装的 Docker 版本可能会较低，无法提供某些特性。

Docker 官方提供的下载包，在主要的 Linux 发行版上都可以直接解压配置后运行。本文提供了简要的安装配置指导，供简单测试场景下的 Docker 的安装配置指引。

> [!NOTE]
>
> 本文档给出的安装配置指引，通过 `Kylin V10SP1 aarch64`、`Kylin V10SP2 x86_64` 和 `Asianux 7(Core) x86_64` 版的验证。

## 1. 下载

官方提供的 Docker 和 Docker Compose 下载地址如下，请按照 CPU 架构下载对应的安装包。

### 1.1 x86_64

[https://download.docker.com/linux/static/stable/x86_64/docker-26.1.3.tgz](https://download.docker.com/linux/static/stable/x86_64/docker-26.1.3.tgz)

[https://github.com/docker/compose/releases/download/v2.31.0/docker-compose-linux-x86_64](https://github.com/docker/compose/releases/download/v2.31.0/docker-compose-linux-x86_64)

[https://download.docker.com/linux/static/stable/x86_64/docker-rootless-extras-26.1.3.tgz](https://download.docker.com/linux/static/stable/x86_64/docker-rootless-extras-26.1.3.tgz)

### 1.2 aarch64

[https://download.docker.com/linux/static/stable/aarch64/docker-26.1.3.tgz](https://download.docker.com/linux/static/stable/aarch64/docker-26.1.3.tgz)

[https://github.com/docker/compose/releases/download/v2.31.0/docker-compose-linux-aarch64](https://github.com/docker/compose/releases/download/v2.31.0/docker-compose-linux-aarch64)

[https://download.docker.com/linux/static/stable/aarch64/docker-rootless-extras-26.1.3.tgz](https://download.docker.com/linux/static/stable/aarch64/docker-rootless-extras-26.1.3.tgz)

## 2. 安装

上传 `docker-26.1.3-linux-x86_64.tgz` 并解压，如 `/root/docker/` 目录。

上传 `docker-compose-2.31.0-linux-x86_64` 到 `/root/docker/` 目录。

按照如下步骤，安装Docker：

```bash
# 重命名docker-compose-2.31.0-linux-x86_64
mv /docker/docker-compose-2.31.0-linux-x86_64 /docker/docker-compose
# 为docker-compose添加执行权限
chmod +x /docker/docker-compose
# 调整docker-compose所属的用户和组
chown root:root /docker/docker-compose
# 移动到/usr/bin/目录
mv docker/* /usr/bin/
```

将 Docker 注册为服务：

```bash
vi /etc/systemd/system/docker.service
=======================================================
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
=======================================================

chmod +x /etc/systemd/system/docker.service
systemctl daemon-reload 
systemctl enable docker.service
systemctl start docker
systemctl status docker
systemctl stop docker
# 查看docker-compose信息
docker-compose --version
```

上传一个 Niginx 离线镜像测试验证：

```bash
# 恢复测试验证镜像
docker load -i nginx-1.26.2-alpine-slim.tar
# 新建一个测试用的docker compose脚本
vi test-compose.yml
=======================================================
services:
  nginx-test:
    image: nginx:1.26.2-alpine-slim
    container_name: nginx-test
    ports:
      - "80:80"
=======================================================
# 使用docker compose启动测试脚本
docker-compose -f ./test-compose.yml -p test-docker up -d
# 查看容器情况
docker ps
# 获取nginx首页
curl http://localhost
# 下线测试容器
docker-compose -p test-docker down
# 删除测试镜像
docker rmi nginx:1.26.2-alpine-slim
```

## 3. 获取镜像

为 Docker 设置国内镜像源。执行命令 `vi /etc/docker/daemon.json` 添加如下内容：

```bash
{
    "registry-mirrors":[
        "https://huecker.io",
        "https://docker.m.daocloud.io",
        "https://dockerhub.timeweb.cloud",
        "https://hub-mirror.c.163.com",
        "https://registry.aliyuncs.com",
        "https://registry.docker-cn.com",
        "https://docker.mirrors.ustc.edu.cn"
    ]
}
```

> 如果没有 `/etc/docker/` 目录则 `mkdir -p /etc/docker/` 创建这个目录。

添加完国内镜像源后，需要重启 Docker 服务：`systemctl restart docker`，然后按照如下命令拉取镜像。

```bash
# 拉取arm架构的jdk8镜像
docker pull --platform linux/arm64 eclipse-temurin:8u412-b08-jdk-centos7
docker save eclipse-temurin:8u412-b08-jdk-centos7 -o eclipse-temurin_8u412-b08-jdk-centos7_arm64.tar

docker pull --platform linux/amd64 eclipse-temurin:8u412-b08-jdk-centos7
docker save eclipse-temurin:8u412-b08-jdk-centos7 -o eclipse-temurin_8u412-b08-jdk-centos7_amd64.tar
```

> 拉取镜像命令中的 `--platform` 参数取值，常用的有两个：`linux/amd64` 和 `linux/arm64`

## 4 打包镜像

创建 Dockerfile（初始命令经 ChatGPT 指导优化后的结果）：

```dockerfile
FROM eclipse-temurin:8u412-b08-jdk-centos7

# 创建用户组和用户，确保主目录存在
RUN groupadd appgroup && \
    useradd -r -g appgroup -d /home/appuser -s /sbin/nologin appuser && \
    mkdir -p /home/appuser && \
    chown -R appuser:appgroup /home/appuser

# 设置工作目录
WORKDIR /opt/ncpm2.5

# 拷贝应用文件
COPY *.jar app.jar
COPY *.yml app.yml

# 修改应用目录权限以确保非 root 用户可访问
RUN chown -R appuser:appgroup /opt/ncpm2.5

# 切换到非 root 用户
USER appuser

# 设置启动命令
CMD ["java", "-jar", "app.jar", "--spring.config.location=app.yml"]
```

优化后的 Dockerfile：

1. 减小镜像体积；
2. 遵循容器日志管理最佳实践；
3. 提高安全性，避免以 root 身份运行；
4. 提高构建效率，减少构建层数；
5. 保证镜像构建的一致性。

创建镜像打包命令：

```bash
#!/bin/bash

docker build -t ncmp/customer:v2.5 .
```

启动容器命令：

```bash
docker run -d --name ncpm2.5-customer --network host ncmp/customer:v2.5
```

操作容器命令：

```bash
docker logs -f ncpm2.5-customer
docker exec -it ncpm2.5-customer bash
```

重新打包步骤：

```bash
cd /root/kevin/pack/
# 失败后重新打包：删除容器、删除镜像
=======================================================
docker rm -f ncpm2.5-customer
docker rmi ncmp/customer:v2.5
./build.sh
docker run -d --name ncpm2.5-customer --network host ncmp/customer:v2.5
docker logs -f ncpm2.5-customer
=======================================================
```

Kevin@HangZhou#20241223
