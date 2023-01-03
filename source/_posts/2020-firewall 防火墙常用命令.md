---
title: firewall 防火墙常用命令
date: 2019-03-30 11:20:34
---

firewall

配置firewalld

```

firewall-cmd --version  # 查看版本
firewall-cmd --help     # 查看帮助

# 查看设置：
firewall-cmd --state  # 显示状态
firewall-cmd --get-active-zones  # 查看区域信息
firewall-cmd --get-zone-of-interface=eth0  # 查看指定接口所属区域
firewall-cmd --panic-on  # 拒绝所有包
firewall-cmd --panic-off  # 取消拒绝状态
firewall-cmd --query-panic  # 查看是否拒绝

firewall-cmd --reload # 更新防火墙规则
firewall-cmd --complete-reload
# 两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务


# 将接口添加到区域，默认接口都在public
firewall-cmd --zone=public --add-interface=eth0
# 永久生效再加上 --permanent 然后reload防火墙
 
# 设置默认接口区域，立即生效无需重启
firewall-cmd --set-default-zone=public

# 查看所有打开的端口：
firewall-cmd --zone=dmz --list-ports

# 加入一个端口到区域：
firewall-cmd --zone=dmz --add-port=8080/tcp
# 若要永久生效方法同上
 
# 打开一个服务，类似于将端口可视化，服务需要在配置文件中添加，/etc/firewalld 目录下有services文件夹，这个不详细说了，详情参考文档
firewall-cmd --zone=work --add-service=smtp
 
# 移除服务
firewall-cmd --zone=work --remove-service=smtp

# 显示支持的区域列表
firewall-cmd --get-zones

# 设置为家庭区域
firewall-cmd --set-default-zone=home

# 查看当前区域
firewall-cmd --get-active-zones

# 设置当前区域的接口
firewall-cmd --get-zone-of-interface=enp03s

# 显示所有公共区域（public）
firewall-cmd --zone=public --list-all

# 临时修改网络接口（enp0s3）为内部区域（internal）
firewall-cmd --zone=internal --change-interface=enp03s

# 永久修改网络接口enp03s为内部区域（internal）
firewall-cmd --permanent --zone=internal --change-interface=enp03s


```



服务管理

```
# 显示服务列表  
Amanda, FTP, Samba和TFTP等最重要的服务已经被FirewallD提供相应的服务，可以使用如下命令查看：

firewall-cmd --get-services

# 允许SSH服务通过
firewall-cmd --new-service=ssh

# 禁止SSH服务通过
firewall-cmd --delete-service=ssh

# 打开TCP的8080端口
firewall-cmd --enable ports=8080/tcp

# 临时允许Samba服务通过600秒
firewall-cmd --enable service=samba --timeout=600

# 显示当前服务
firewall-cmd --list-services

# 添加HTTP服务到内部区域（internal）
firewall-cmd --permanent --zone=internal --add-service=http
firewall-cmd --reload     # 在不改变状态的条件下重新加载防火墙


```



端口管理

```shell
# 打开443/TCP端口
firewall-cmd --add-port=443/tcp  --permanent

# 永久打开3690/TCP端口
firewall-cmd --permanent --add-port=3690/tcp  --permanent

# 永久打开端口好像需要reload一下，临时打开好像不用，如果用了reload临时打开的端口就失效了
# 其它服务也可能是这样的，这个没有测试
firewall-cmd --reload

# 查看防火墙，添加的端口也可以看到
firewall-cmd --list-all


```





all



```
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --add-port=443/tcp  --permanent
firewall-cmd --add-port=80/tcp  --permanent
firewall-cmd --add-port=7002/tcp  --permanent
```

zone

增加一个zone

```
firewall-cmd --new-zone=zgcw-access --permanent
```

增加ip和端口

gateway

```
firewall-cmd --zone=zgcw-access --add-source=172.17.100.0/24 --permanent
firewall-cmd --zone=zgcw-access --add-port=9939/tcp  --permanent
firewall-cmd --zone=zgcw-access --add-port=7001/tcp  --permanent
firewall-cmd --zone=zgcw-access --add-port=7002/tcp  --permanent
firewall-cmd --zone=zgcw-access --add-port=8091-8094/tcp  --permanent
firewall-cmd --reload
```

work1、work2



```
firewall-cmd --zone=zgcw-access --add-source=172.17.100.0/24 --permanent
firewall-cmd --zone=zgcw-access --add-port=8091-8094/tcp  --permanent
firewall-cmd --reload
```



ip段

```
172.17.100.218
172.17.100.222 work1
172.17.100.223 work2
172.17.100.220 es
172.17.86.0
```

