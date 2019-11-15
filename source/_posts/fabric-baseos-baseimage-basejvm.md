---
title: fabric baseos baseimage basejvm
date: 2019-10-30 14:59:07
tags:
---

# fabric-baseimage主要内容



fabric-baseimage主要内容

fabric-baseimage主要是build出三个基本image(baseos, basejvm, baseimage)，还包括三个dependent-images(couchdb, kafka, zookeeper)。

这里我们不讨论dependent-images，只关注三个基本image，即baseos, basejvm, baseimage

这三个基本images是继承关系：

1. baseos是最基础的，基本上只包括各个不同平台os image的一个wrapper。
2. basejvm是baseos的扩展，增加了JDK。
3. baseimage是basejvm的扩展，增加了一些库和工具，例如curl, make, g++, golang, nodejs, python等。



# baseos

基于指定的os image更新到最新库版本。
然后安装wget工具，因为wget在后面的image安装中会被用到。

```ruby
$ cat config/baseos/Dockerfile.in
FROM _DOCKER_BASE_                              # defined in Makefile: DOCKER_BASE_amd64=ubuntu:xenial
COPY scripts /tmp/scripts
RUN cd /tmp/scripts && \
    common/init.sh && \                         # update to latest using apt-get
    docker/init.sh && \                         # intall wget, it's used by later script
    common/cleanup.sh && \
    rm -rf /tmp/scripts
```

# basejvm

在baseos的基础上，安装java环境。

```ruby
$ cat config/basejvm/Dockerfile.in
FROM _NS_/fabric-baseos:_TAG_
COPY scripts /tmp/scripts
RUN cd /tmp/scripts && \
    common/jvm.sh && \                          # install openjdk-8-jdk
    common/cleanup.sh && \
    rm -rf /tmp/scripts
```

# baseimage

在basejvm的基础上，增加了一些工具包，包括curl,sudo, git, make, zip, golang, nodejs, python, etc.

```ruby
$ cat config/baseimage/Dockerfile.in
FROM _NS_/fabric-basejvm:_TAG_
COPY scripts /tmp/scripts
RUN cd /tmp/scripts && \
    common/packages.sh && \                     #install software-properties-common curl sudo
    common/setup.sh && \                        # install common tools(git, net-tools, make, g++, unzip, etc)/Golang/NodeJS/python2.7/protocol buffer support
    docker/fixup.sh && \                        # prepare /opt/gopath and /var/hyperledger
    common/cleanup.sh && \
    rm -rf /tmp/scripts
ENV GOPATH=/opt/gopath
ENV GOROOT=/opt/go
ENV GOCACHE=off
ENV PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```



