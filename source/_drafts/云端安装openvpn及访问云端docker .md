### **需求：**

云端机房有几台 ECS 做开发服务器，有 RDS 、MQ、ES、MongoDB、Nacos ，大家用spring cloud微服务做开发，为了保证大家顺利开发，需要在保证安全的情况下让大家可以随时连接云端组件。

### **方案：**

1、买云服务商的vpn网关：花点钱的事情；

2、自己在ECS中安装vpn，每个开发人员接入vpn后开发。

下面主要讲第二种方案。

### **网络情况：**

云端vpc的网段是10.0.0.0/24

服务器一：安装ES、MongoDb、Doris等组件；IP：10.0.0.11

服务器二：安装nacos、redis、docker服务；IP：10.0.0.10

服务器三：docker服务；IP：10.0.0.12

### **计划：**

1、在服务器一上安装openvpn，接入客户端的ip段规划为10.8.0.0/24，让所有vpn接入的客户端可以直接访问服务器二、三；

2、服务器一、二、三上可以直接连通所有vpn接入的客户端；

3、vpn接入的客户端可以连通服务器二、三上的docker容器中的服务；

1是 openvpn的基本功能；

2需要在openvpn 服务端给vpn客户端推送路由信息；

3需要服务器二三上需要被访问的docker容器接入直接的docker network，然后在openvpn服务端给vpn客户端推送到每个docker network的路由。

### 先处理docker network

在服务器二、服务器三分别创建单独的docker network：

服务器二： `docker network create --subnet=10.10.0.0/24 cem-network`。

服务器三：`docker network create --subnet=10.12.0.0/24 cem-network`。

然后这两个服务器中需要被vpn客户端访问的容器都需要创建在cem-network中。

### 安装openvpn服务

先启动防火墙，需要防火墙做转发（也可以用iptables，这个最近这些年很少用了）

```
sudo systemctl enable firewalld
sudo systemctl start firewalld
```

安装openvpn软件

```
yum -y install epel-release
yum install openvpn
wget https://github.com/OpenVPN/easy-rsa-old/archive/2.3.3.tar.gz
tar xfz 2.3.3.tar.gz
mkdir /etc/openvpn/easy-rsa
cp -rf easy-rsa-old-2.3.3/easy-rsa/2.0/* /etc/openvpn/easy-rsa
cp /usr/share/doc/openvpn-2.4.12/sample/sample-config-files/server.conf /etc/openvpn

```

需改/etc/openvpn/server.conf配置文件，大致需要修改如下几点：

然后在openvpn配制中增加到10.10.0.0/24的路由。

```
client-to-client
#下面这行注释掉
;tls-auth ta.key 0  
#添加这个
tls-crypt myvpn.tlsauth

#添加到云端服务器的路由
#所有 10.0.0/24的都通过openvpn服务器来访问
push "route 10.0.0.0 255.255.255.0"
#访问10.10.0/24 容器的都通过10.0.0.10来路由
push "route 10.10.0.0 255.255.255.0 10.0.0.10 1"
#访问10.12.0/24 容器的都通过10.0.0.12来路由
push "route 10.12.0.0 255.255.255.0 10.0.0.12 1"
```

```
openvpn --genkey --secret /etc/openvpn/myvpn.tlsauth
mkdir /etc/openvpn/easy-rsa/keys
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
./build-ca
./build-key-server server
./build-dh
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
cd /etc/openvpn/easy-rsa

./build-key wanghongxing
```

然后修改防火墙配置

```


firewall-cmd --zone=public --add-port=1194/tcp --permanent
firewall-cmd --zone=public --add-port=1194/udp --permanent
firewall-cmd --zone=trusted --add-service openvpn --permanent

firewall-cmd --list-services --zone=trusted
firewall-cmd --add-masquerade
firewall-cmd --permanent --add-masquerade
firewall-cmd --query-masquerade
firewall-cmd --permanent --add-interface=tun0
firewall-cmd --permanent --add-service=openvpn
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
firewall-cmd --reload

echo "net.ipv4.ip_forward = 1" >>/etc/sysctl.conf
systemctl restart openvpn@server.service

```



### 制作vpn客户端文件

把 下面这几个文件复制下载到客户端电脑

```
/etc/openvpn/ca.crt 
/etc/openvpn/myvpn.tlsauth
/etc/openvpn/easy-rsa/keys/wanghongxing.crt
/etc/openvpn/easy-rsa/keys/wanghongxing.csr
/etc/openvpn/easy-rsa/keys/wanghongxing.key
```

然后编辑生成 whx.ovpn

```
client
dev tun
proto udp
remote 114.116.201.xxx
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert wanghongxing.crt
key wanghongxing.key
remote-cert-tls server
tls-crypt myvpn.tlsauth
verb 3

```

### 下载openvpn客户端

mac用户下载 Tunnelblick ，windows用户下载.

然后倒入whx.ovpn

### 在服务器二三设置路由

其服务器二三设置访问vpn客户端的路由

```
ip route add 10.8.0.0/24 via 10.0.0.11 dev eth0
#这样以后重启了也有效
echo "10.8.0.0/24 via 10.0.0.11 dev eth0" >> /etc/sysconfig/network-scripts/route-eth0

```

在服务器二设置访问服务器三容器的路由

```
ip route add 10.12.0.0/24 via 10.0.0.12 dev eth0
echo "10.12.0.0/24 via 10.0.0.12 dev eth0" >> /etc/sysconfig/network-scripts/route-eth0
```

在服务器三设置访问服务器二容器的路由

```
ip route add 10.10.0.0/24 via 10.0.0.10 dev eth0
echo "10.10.0.0/24 via 10.0.0.10 dev eth0" >> /etc/sysconfig/network-scripts/route-eth0
```

在服务器一设置访问服务器二三容器的路由

```
ip route add 10.12.0.0/24 via 10.0.0.12 dev eth0
ip route add 10.10.0.0/24 via 10.0.0.10 dev eth0
echo "10.12.0.0/24 via 10.0.0.12 dev eth0" >> /etc/sysconfig/network-scripts/route-eth0
echo "10.10.0.0/24 via 10.0.0.10 dev eth0" >> /etc/sysconfig/network-scripts/route-eth0

```





ip route add 172.18.0.0/24 via 10.0.0.10 dev eth0

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --add-port=8848/tcp --permanent
firewall-cmd --zone=public --add-port=9848/tcp --permanent
firewall-cmd --permanent --zone=trusted --change-interface=docker0

firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --zone=public --add-port=9876/tcp --permanent

```



docker network create --subnet=10.10.0.0/24 cem-network

linux中手动添加路由的方法：

ip route add 10.10.0.0/24 via 10.0.0.10 dev eth0



macos中手动添加路由的方法：

```
sudo route -n add -net 10.10.0.0/24 10.0.0.10
```



为了让spring cloud微服务注册到nacos的时候使用特定ip段，需要在bootstrap中设置 preferred-networks ，比如内网用户设置为：

```
spring:
  cloud:
    inetutils:
      preferred-networks:
        - 10.8

```

比如cem容器中设置为：

```
spring:
  cloud:
    inetutils:
      preferred-networks:
        - 10.10
```

