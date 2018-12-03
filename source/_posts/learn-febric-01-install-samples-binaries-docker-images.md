---
title: learn febric install samples, Binaries and Docker Images
date: 2018-11-30 22:30:47
tags:
---

[Febric GetStart](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)



新建一个试验项目目录，我起名 febric ,在这个目录里执行如下命令
```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0
```

我自己curl执行错误，就手动下载 
```
wget https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh
```
保存为bootstrap.sh,然后手动执行

```
chmod +x bootstrap.sh
./bootstrap.sh  -s 1.3.0
```
应该是下载了好多tools在bin目录，这个目录需要设置的path中
下载速度比较慢...

> The command above downloads and executes a bash script that will download and extract all of the platform-specific binaries you will need to set up your network and place them into the cloned repo you created above. It retrieves the following platform-specific binaries:
- configtxgen,
- configtxlator,
- cryptogen,
- discover,
- idemixgen
- orderer,
- peer, and
- fabric-ca-client
```
Finally, the script will download the Hyperledger Fabric docker images from Docker Hub into your local Docker registry and tag them as ‘latest’.

然后最好把～/febric/bin 添加到系统 PATH 中，方便执行