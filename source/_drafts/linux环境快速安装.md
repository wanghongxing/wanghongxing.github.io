**linux服务器快速安装**



### centos环境





 创建免密钥登录

```
 ssh-copy-id -i ~/.ssh/id_rsa.pub root@nps-1
```





  - 如果服务器需要秘钥

  ```
 生成密钥:
 # echo -e "\n"|ssh-keygen -t rsa -N "" >/dev/null 2>&1
 
  ```



```shell
#使用管理员来做这些操作
#切换成管理员账号
su


#为了方便添加软件源，支持 devicemapper 存储类型，安装如下软件包
yum update
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

添加 yum 软件源

添加 Docker 稳定版本的 yum 软件源

```shell
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装 Docker

更新一下 yum 软件源的缓存，并安装 Docker。

```shell
yum update
yum install docker-ce-19.03.12-3.el7
```

如果弹出 GPG key 的接收提示，请确认是否为 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35，如果是，可以接受并继续安装。 

至此，Docker 已经安装完成了，Docker 服务是没有启动的，操作系统里的 docker 组被创建，但是没有用户在这个组里。

\# Install docker-compose

 

```shell
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

```

**注意:**

 默认的 docker 组是没有用户的（也就是说需要使用 sudo 才能使用 docker 命令）。

您可以将用户添加到 docker 组中（此用户就可以直接使用 docker 命令了）。

加入 docker 用户组命令

```shell
whoami
```

记录下当前用户名

```shell
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

\#创建docker需要的网络

```shell
docker network create tradeblock
```

