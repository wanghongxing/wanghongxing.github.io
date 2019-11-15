---
title: 'Fabric production env make msp and tls by CA '
date: 2019-11-11 17:13:35
tags:
---





Fabric kafka生产环境部署的基础上部署Fabric CA，使用Fabric CA进行生成公私钥和证书等文件，全部替换cryptogen工具，包括生成TLS相关的私钥和证书等文件。

​       Fabric kafka生产环境部署有三个组织，分别为orderer（排序）组织和两个Peer(节点)组织，对应的ID为example.com、org1.example.com和org2.example.com。为了让生产环境Fabric CA具有扩展性和安全性，存在一个逻辑的根CA（RootCA）和三个中间CA（Intermedia CA），三个中间CA（Intermedia CA）都隶属根CA（RootCA）。
​       三个中间CA（Intermedia CA）分别负责orderer（排序）组织和两个Peer(节点)组织的公私钥和证书生成。当有新的组织加入，只需再生成一个中间CA（Intermedia CA）接入到根CA（RootCA）下，不会影响其它中间CA（Intermedia CA），生产环境CA网络拓扑图如下：

​       ![img](https://img2018.cnblogs.com/blog/544703/201810/544703-20181014130346794-1289725875.jpg)

​       根据生产环境CA网络拓扑图，实现生产环境CA的部署及生成上一章：Fabric kafka生产环境部署所需要公私钥、证书及TLS证书等文件。
生产环境CA部署到上一章：Fabric kafka生产环境部署的kafka3（192.168.235.6）服务器上；由于四CA都在同一台电脑，端口号不能使用同一个，对应的端口号如下表：


![img](https://img2018.cnblogs.com/blog/544703/201810/544703-20181014130210755-99031849.jpg)

运行和配置步骤如下：

(一) CA服务启动
1.	RootCA启动
	1）	创建目录

```
# cd $GOPATH/src/github.com/hyperledger/fabric-ca/bin
# mkdir ca-server
# cd ca-server
```

2） 初始化CA服务

```
# fabric-ca-server init -b admin:adminpw --home ./rootca
```

3） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin:adminpw --home ./rootca --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-rootca.yml到ca-server目录

```
# docker-compose -f docker-rootca.yaml up -d
```

2. IntermediaCA1启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./intermediaca1
# vi ./intermediaca1/fabric-ca-server-config.yaml
修改
port: 7055
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./intermediaca1 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca1.yml到ca-server目录

```
# docker-compose -f docker-intermediaca1.yaml up -d
```

3. IntermediaCAtls1启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls1
# vi ./intermediacatls1/fabric-ca-server-config.yaml
修改
port: 8055
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls1 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca1.yml到ca-server目录

```
# docker-compose -f docker-intermediacatls1.yaml up -d
```

4. IntermediaCA2启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./intermediaca2
# vi ./intermediaca2/fabric-ca-server-config.yaml
修改
port:7056
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./intermediaca2 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca2.yml到ca-server目录

```
# docker-compose -f docker-intermediaca2.yaml up -d
```

5. IntermediaCAtls2启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls2
# vi ./intermediacatls2/fabric-ca-server-config.yaml
修改
port:8056
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls2 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca2.yml到ca-server目录

```
# docker-compose -f docker-intermediacatls2.yaml up -d
```

6. IntermediaCA3启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin3:adminpw3 -u http://admin:adminpw@localhost:7054 --home ./intermediaca3
# vi ./intermediaca3/fabric-ca-server-config.yaml
修改
port: 7057
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin3:adminpw3 -u http://admin:adminpw@localhost:7054 --home ./intermediaca3 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca3.yml到ca-server目录

```
# docker-compose -f docker-intermediaca3.yaml up -d
```

7. IntermediaCAtls3启动
   1）	初始化CA服务

```
# fabric-ca-server init -b admin3:adminpw3 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls3
# vi ./intermediacatls3/fabric-ca-server-config.yaml
修改
port: 8057
```

2） 启动CA服务
【命令行启动】

```
# fabric-ca-server start -b admin3:adminpw3 -u http://admin:adminpw@localhost:7054 --home ./intermediacatls3 --cfg.affiliations.allowremove --cfg.identities.allowremove
```

【docker启动】
拷贝文件docker-intermediaca3.yml到ca-server目录

```
# docker-compose -f docker-intermediacatls3.yaml up -d
```

(二) IntermediaCA1生成证书
1.	生成example.com的msp
	1）	登记example.com

```
# cd /opt/gopath/src/github.com/hyperledger/fabric-ca/bin/ca-server
# fabric-ca-client enroll -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```



2. 生成Admin@example.com的msp
   1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@example.com --id.type client --id.affiliation "com.example" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=example.com --csr.hosts=['example.com'] -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```



2） 登记Admin@example.com

```
# fabric-ca-client enroll -u http://Admin@example.com:123456@localhost:7055 --csr.cn=example.com --csr.hosts=['example.com'] -M ./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/admincerts
# mkdir ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/msp/admincerts
```



3. 生成orderer0.example.com的msp和tls
   1）	注册orderer0.example.com

```
# fabric-ca-client register --id.name orderer0.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer0.example.com --csr.hosts=['orderer0.example.com'] -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```

2） 登记orderer0.example.com

```
# fabric-ca-client enroll -u http://orderer0.example.com:123456@localhost:7055 --csr.cn=orderer0.example.com --csr.hosts=['orderer0.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/admincerts
```



4. 生成orderer1.example.com的msp
   1）	注册orderer1.example.com

```
# fabric-ca-client register --id.name orderer1.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer1.example.com --csr.hosts=['orderer1.example.com'] -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```

2） 登记orderer1.example.com

```
# fabric-ca-client enroll -u http://orderer1.example.com:123456@localhost:7055 --csr.cn=orderer1.example.com --csr.hosts=['orderer1.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp/admincerts
```



5. 生成orderer2.example.com的msp
   1）	注册orderer2.example.com

```
# fabric-ca-client register --id.name orderer2.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer2.example.com --csr.hosts=['orderer2.example.com'] -M ./crypto-config/ordererOrganizations/example.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```

2） 登记orderer2.example.com

```
# fabric-ca-client enroll -u http://orderer2.example.com:123456@localhost:7055 --csr.cn=orderer2.example.com --csr.hosts=['orderer2.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/msp/admincerts
```



(三) IntermediaCAtls1生成证书
1.	生成example.com的msp
	1）	登记example.com

```
# cd /opt/gopath/src/github.com/hyperledger/fabric-ca/bin/ca-server
# fabric-ca-client enroll -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```



\2. 生成Admin@example.com的tls
1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@example.com --id.type client --id.affiliation "com.example" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=example.com --csr.hosts=['example.com'] -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```



2） 登记Admin@example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@example.com:123456@localhost:8055 --csr.cn=example.com --csr.hosts=['example.com'] -M ./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/tlsintermediatecerts/tls-localhost-8055.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/client.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/tls/client.key
```



\3. 生成orderer0.example.com的msp和tls
1）	注册orderer0.example.com

```
# fabric-ca-client register --id.name orderer0.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer0.example.com --csr.hosts=['orderer0.example.com'] -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```

2） 登记orderer0.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://orderer0.example.com:123456@localhost:8055 --csr.cn=orderer0.example.com --csr.hosts=['orderer0.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/tlsintermediatecerts/tls-localhost-8055.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/server.key
```



\4. 生成orderer1.example.com的msp
1）	注册orderer1.example.com

```
# fabric-ca-client register --id.name orderer1.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer1.example.com --csr.hosts=['orderer1.example.com'] -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```

2） 登记orderer1.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://orderer1.example.com:123456@localhost:8055 --csr.cn=orderer1.example.com --csr.hosts=['orderer1.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/tlsintermediatecerts/tls-localhost-8055.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.key
```



\5. 生成orderer2.example.com的msp
1）	注册orderer2.example.com

```
# fabric-ca-client register --id.name orderer2.example.com --id.type orderer --id.affiliation "com.example" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer2.example.com --csr.hosts=['orderer2.example.com'] -M ./crypto-config/ordererOrganizations/example.com/tls -u http://admin1:adminpw1@localhost:8055 --home ./fabric-ca-client
```

2） 登记orderer2.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://orderer2.example.com:123456@localhost:8055 --csr.cn=orderer2.example.com --csr.hosts=['orderer2.example.com'] -M ./crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/tlsintermediatecerts/tls-localhost-8055.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.key
```



(四) IntermediaCA2生成证书
1.	生成org1.example.com的msp
	1）	登记org1.example.com

```
# fabric-ca-client enroll --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example.org1 -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
```



\2. 生成Admin@example.com的msp
1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@org1.example.com --id.type client --id.affiliation "com.example.org1" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
```



2） 登记Admin@example.com

```
# fabric-ca-client enroll -u http://Admin@org1.example.com:123456@localhost:7056 --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/admincerts
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/msp/admincerts
```



\3. 生成peer0.org1.example.com的msp
1）	注册peer0.org1.example.com

```
# fabric-ca-client register --id.name peer0.org1.example.com --id.type peer --id.affiliation "com.example.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org1.example.com --csr.hosts=['peer0.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
```

2） 登记peer0.org1.example.com

```
# fabric-ca-client enroll -u http://peer0.org1.example.com:123456@localhost:7056 --csr.cn=peer0.org1.example.com --csr.hosts=['peer0.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/admincerts
```



\4. 生成peer1.org1.example.com的msp
1）	注册peer1.org1.example.com

```
# fabric-ca-client register --id.name peer1.org1.example.com --id.type peer --id.affiliation "com.example.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer1.org1.example.com --csr.hosts=['peer1.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/msp -u http://admin2:adminpw2@localhost:7056 --home ./fabric-ca-client
```

2） 登记peer1.org1.example.com

```
# fabric-ca-client enroll -u http://peer1.org1.example.com:123456@localhost:7056 --csr.cn=peer1.org1.example.com --csr.hosts=['peer1.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp/admincerts
```



(五) IntermediaCAtls2生成证书
1.	生成org1.example.com的msp
	1）	登记org1.example.com

```
# fabric-ca-client enroll --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example.org1 -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
```



\2. 生成Admin@example.com的msp
1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@org1.example.com --id.type client --id.affiliation "com.example.org1" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
```



2） 登记Admin@example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@org1.example.com:123456@localhost:8056 --csr.cn=org1.example.com --csr.hosts=['org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/tlsintermediatecerts/tls-localhost-8056.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/client.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/tls/client.key
```



\3. 生成peer0.org1.example.com的msp
1）	注册peer0.org1.example.com

```
# fabric-ca-client register --id.name peer0.org1.example.com --id.type peer --id.affiliation "com.example.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org1.example.com --csr.hosts=['peer0.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
```

2） 登记peer0.org1.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://peer0.org1.example.com:123456@localhost:8056 --csr.cn=peer0.org1.example.com --csr.hosts=['peer0.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlsintermediatecerts/tls-localhost-8056.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
```



\4. 生成peer1.org1.example.com的tls
1）	注册peer1.org1.example.com

```
# fabric-ca-client register --id.name peer1.org1.example.com --id.type peer --id.affiliation "com.example.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer1.org1.example.com --csr.hosts=['peer1.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/tls -u http://admin2:adminpw2@localhost:8056 --home ./fabric-ca-client
```

2） 登记peer1.org1.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://peer1.org1.example.com:123456@localhost:8056 --csr.cn=peer1.org1.example.com --csr.hosts=['peer1.org1.example.com'] -M ./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/tlsintermediatecerts/tls-localhost-8056.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.key
```



(六) IntermediaCA3生成证书
1.	生成org2.example.com的msp
	1）	登记org2.example.com

```
# fabric-ca-client enroll --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example.org2 -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
```



\2. 生成Admin@example.com的msp
1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@org2.example.com --id.type client --id.affiliation "com.example.org2" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
```



2） 登记Admin@example.com

```
# fabric-ca-client enroll -u http://Admin@org2.example.com:123456@localhost:7057 --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/admincerts
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/msp/admincerts
```



\3. 生成peer0.org2.example.com的msp
1）	注册peer0.org2.example.com

```
# fabric-ca-client register --id.name peer0.org2.example.com --id.type peer --id.affiliation "com.example.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org2.example.com --csr.hosts=['peer0.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
```

2） 登记peer0.org2.example.com

```
# fabric-ca-client enroll -u http://peer0.org2.example.com:123456@localhost:7057 --csr.cn=peer0.org2.example.com --csr.hosts=['peer0.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/admincerts
4.    生成peer1.org2.example.com的msp
```



1） 注册peer1.org2.example.com

```
# fabric-ca-client register --id.name peer1.org2.example.com --id.type peer --id.affiliation "com.example.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer1.org2.example.com --csr.hosts=['peer1.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/msp -u http://admin3:adminpw3@localhost:7057 --home ./fabric-ca-client
```

2） 登记peer1.org2.example.com

```
# fabric-ca-client enroll -u http://peer1.org2.example.com:123456@localhost:7057 --csr.cn=peer1.org2.example.com --csr.hosts=['peer1.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp --home ./fabric-ca-client
```

3） 生成msp



```
# mkdir ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp/admincerts
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp/admincerts
```



(七) IntermediaCAtls3生成证书
1.	生成org2.example.com的msp
	1）	登记org2.example.com

```
# fabric-ca-client enroll --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
```

2） 添加联盟成员



```
# fabric-ca-client affiliation list -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org1 -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
# fabric-ca-client affiliation remove --force org2 -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
# fabric-ca-client affiliation add com.example.org2 -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
```



\2. 生成Admin@example.com的msp
1）	注册Admin@example.com



```
# fabric-ca-client register --id.name Admin@org2.example.com --id.type client --id.affiliation "com.example.org2" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
```



 

2） 登记Admin@example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@org2.example.com:123456@localhost:8057 --csr.cn=org2.example.com --csr.hosts=['org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls --home ./fabric-ca-client
```

1） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/tlsintermediatecerts/tls-localhost-8057.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/client.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/tls/client.key
```



\3. 生成peer0.org2.example.com的tls
1）	注册peer0.org2.example.com

```
# fabric-ca-client register --id.name peer0.org2.example.com --id.type peer --id.affiliation "com.example.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org2.example.com --csr.hosts=['peer0.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
```

2） 登记peer0.org2.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://peer0.org2.example.com:123456@localhost:8057 --csr.cn=peer0.org2.example.com --csr.hosts=['peer0.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlsintermediatecerts/tls-localhost-8057.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
```



\4. 生成peer1.org2.example.com的tls
1）	注册peer1.org2.example.com

```
# fabric-ca-client register --id.name peer1.org2.example.com --id.type peer --id.affiliation "com.example.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer1.org2.example.com --csr.hosts=['peer1.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/tls -u http://admin3:adminpw3@localhost:8057 --home ./fabric-ca-client
```

2） 登记peer1.org2.example.com

```
# fabric-ca-client enroll -d --enrollment.profile tls -u http://peer1.org2.example.com:123456@localhost:8057 --csr.cn=peer1.org2.example.com --csr.hosts=['peer1.org2.example.com'] -M ./crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls --home ./fabric-ca-client
```

3） 生成tls



```
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/tlsintermediatecerts/tls-localhost-8057.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/signcerts/cert.pem ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt
# cp ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/keystore/xxxxxxx_sk ./fabric-ca-client/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key
```



 