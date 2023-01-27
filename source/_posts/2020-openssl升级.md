---
title: openssl升级
date: 2019-08-30 14:20:34
tags: 
  - openssl
  - 安全
---

openssl升级

编译安装了

```
./config --prefix=/usr/local/openssl // 指定安装路径
make && make install
```

最后替换当前系统的旧版本 openssl 「先保存原来的」

```
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/lib64/openssl /usr/lib64/openssl.old
mv /usr/lib64/libssl.so /usr/lib64/libssl.so.old
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so /usr/lib64/libssl.so
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v 
```

最后查看当前系统 openssl 版本

```
openssl version
```