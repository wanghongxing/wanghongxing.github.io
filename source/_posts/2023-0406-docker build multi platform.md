---
title: 2023-0406-docker build multi platform
---

docker build multi platform

macbook arm版生成镜像时候是arm64 架构的，没法在服务器运行，为了给服务器打镜像。

改成 如下命令可以生成服务器运行的镜像

```
docker   build --platform linux/amd64 .
```

但是不能在本机运行，改成如下命令生成多架构的镜像

```
docker buildx build --platform linux/amd64,linux/arm64
```

但是提示

ERROR: multiple platforms feature is currently not supported for docker driver. Please switch to a different driver (eg. "docker buildx create --use")

然后学习后发现需要

```
docker buildx create --name mybuilder --driver docker-container
```

然后就可以了。