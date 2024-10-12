centos7服务器安装docker



修改 yum 软件源

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```



```
yum update
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

添加 Docker 稳定版本的 yum 软件源

 

```shell
yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
    
    
sed -i 's+https://download.docker.com+https://mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo

```

安装 Docker

更新一下 yum 软件源的缓存，并安装 Docker。

```shell
yum update
yum install docker-ce
```

 

至此，Docker 已经安装完成了，Docker 服务是没有启动的，操作系统里的 docker 组被创建，但是没有用户在这个组里。

\# Install docker-compose

 

```shell
curl -L https://github.com/docker/compose/releases/download/2.29.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

**注意**

 默认的 docker 组是没有用户的（也就是说需要使用 sudo 才能使用 docker 命令）。

您可以将用户添加到 docker 组中（此用户就可以直接使用 docker 命令了）。

加入 docker 用户组命令

```shell
whoami
##记录下当前用户名
usermod -a -G docker USERNAME # Add current user to the docker group
```

 

启动 Docker

如果想添加到开机启动

```
systemctl enable docker
```

启动 docker 服务

```shell
systemctl start docker
```

