---
title: fabric ca 在生产环境的使用和配置
category: fabric
tags:
  - fabric
  - blockchain
---

Fabric CA为Hyperledger Fabric行使证书机构的功能。

Fabric CA提供以下功能：

1. 身份注册，或者将连接到LDAP作为用户注册；
2. 颁发登录证书(ECerts)；
3. 颁发交易证书(TCerts)，保证链上交易的匿名性与不可连接性；
4. 证书续期与撤销

Fabric CA 包含一个服务端组件和一个客户端组件，稍后会进行介绍。



[TOC]







# 一、生成root CA、tlsCA 证书 

一个组织对所有成员都要有自己的证书为了保证安全隔离，fabric规划了数据证书和传输证书，数据证书就是对数据进行加密、加签的时候使用的，传输证书就是所有数据传输过程中的私有证书。

为了实现数据、传输证书分离，每个组织的CA 也要分离，所以每个组织要有有自己的ROOT CA 和ROOT tlsCA。

准备目录：ca tlsca

## 1. 生成椭圆加密根证书步骤

进入ca目录

- 生成私钥

```bash
openssl ecparam -name prime256v1 -genkey -out ca-key_.pem
```

- 私钥转换成pk8格式

```bash
openssl pkcs8 -inform PEM -outform PEM -topk8 -nocrypt -in ca-key_.pem -out ca-key.pem
```

- 做好配置文件

  生成一个openssl.conf

  ```properties
  [ req ]
  default_md              = sha256
  default_keyfile         = ca-key.pem
  distinguished_name      = req_distinguished_name
  prompt = no
  x509_extensions = v3_ca
  req_extensions  = v3_req
  x509_extensions = usr_cert
  [ req_distinguished_name ]
  C=CN
  ST=BJ
  L=BJ
  O=my118.com
  CN=ca.my118.com
  [ usr_cert ]
  basicConstraints=critical,CA:true,pathlen:1
  keyUsage=critical, keyCertSign,cRLSign
  subjectKeyIdentifier=hash
  [ v3_req ]
  basicConstraints=critical,CA:true,pathlen:1
  keyUsage=critical, keyCertSign,cRLSign
  subjectKeyIdentifier=hash
  ```

  

- 创建生成证书请求

```bash
openssl req -config openssl.cnf -new -key ca-key.pem -out cert-req.csr
```

- 生成自签名证书

```shell
openssl x509 -req   -extfile openssl.cnf -extensions usr_cert  -in cert-req.csr -out ca-cert.pem -signkey ca-key.pem  -CAcreateserial -days 3650
```

### 查看

- 查看cert-req.csr

```bash
openssl req -in cert-req.csr -noout
```

- 查看ca-cert.pem

```bash
openssl x509 -in ca-cert.pem -text -noout 
```

## 2. 生成椭圆加密TLS根证书步骤

进入tlsca目录

- 生成私钥

```bash
openssl ecparam -name prime256v1 -genkey -out tlsca-key_.pem
```

- 私钥转换成pk8格式

```bash
openssl pkcs8 -inform PEM -outform PEM -topk8 -nocrypt -in tlsca-key_.pem -out tlsca-key.pem
```

- 做好配置文件

  生成一个openssl.conf

  ```properties
  [ req ]
  default_md              = sha256
  default_keyfile         = tlsca-key.pem
  distinguished_name      = req_distinguished_name
  prompt = no
  x509_extensions = v3_ca
  req_extensions  = v3_req
  x509_extensions = usr_cert
  [ req_distinguished_name ]
  C=CN
  ST=BJ
  L=BJ
  O=my118.com
  CN=tlsca.my118.com
  [ usr_cert ]
  basicConstraints=critical,CA:true,pathlen:1
  keyUsage=critical, keyCertSign,cRLSign
  subjectKeyIdentifier=hash
  [ v3_req ]
  basicConstraints=critical,CA:true,pathlen:1
  keyUsage=critical, keyCertSign,cRLSign
  subjectKeyIdentifier=hash
  ```

  

- 创建生成证书请求

```bash
openssl req -config openssl.cnf -new -key tlsca-key.pem -out tlscert-req.csr
```

- 生成自签名证书

```shell
openssl x509 -req   -extfile openssl.cnf -extensions usr_cert  -in tlscert-req.csr -out tlsca-cert.pem -signkey tlsca-key.pem  -CAcreateserial -days 3650
```

### 查看

- 查看 tlscert-req.csr

```bash
openssl req -in tlscert-req.csr -text
```

- 查看 tlsca-cert.pem

```bash
openssl x509 -in tlsca-cert.pem -text -noout
```



# 二、启动 CA 服务器 

使用 docker-compose 来启动

创建ca-server目录，然后新建 docker-compose.yaml

```yaml
fabric-ca-server:
   image: hyperledger/fabric-ca
   container_name: fabric-ca-server
   ports:
     - "7054:7054"
   environment:
     - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
   volumes:
     - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
   command: sh -c 'fabric-ca-server start -b admin:adminpwd'
```

创建 fabric-ca-server，把ca-cert.pem ca-key.pem文件复制到fabric-ca-server目录中。



在  fabric-ca-server 目录下新建 fabric-ca-server-config.yaml 文件

```yaml
version: 1.4.3-snapshot-3e29f1a
port: 7054
cors:
    enabled: false
    origins:
      - "*"
debug: false
crlsizelimit: 512000
tls:
  enabled: false
  certfile:
  keyfile:
  clientauth:
    type: noclientcert
    certfiles:
ca:
  name:
  keyfile:
  certfile:
  chainfile:
crl:
  expiry: 24h
registry:
  maxenrollments: -1
  identities:
     - name: admin
       pass: adminpw
       type: client
       affiliation: ""
       attrs:
          hf.Registrar.Roles: "*"
          hf.Registrar.DelegateRoles: "*"
          hf.Revoker: true
          hf.IntermediateCA: true
          hf.GenCRL: true
          hf.Registrar.Attributes: "*"
          hf.AffiliationMgr: true
db:
  type: sqlite3
  datasource: fabric-ca-server.db
  tls:
      enabled: false
      certfiles:
      client:
        certfile:
        keyfile:

affiliations:
   com:
      - my118
signing:
    default:
      usage:
        - digital signature
      expiry: 8760h
    profiles:
      ca:
         usage:
           - cert sign
           - crl sign
         expiry: 43800h
         caconstraint:
           isca: true
           maxpathlen: 0
      tls:
         usage:
            - signing
            - key encipherment
            - server auth
            - client auth
            - key agreement
         expiry: 8760h
csr:
   cn: fabric-ca-server
   keyrequest:
     algo: ecdsa
     size: 256
   names:
      - C: CN
        ST: "BJ"
        L:
        O: my118
        OU: tech
   hosts:
     - 4aaf61d0f70f
     - localhost
   ca:
      expiry: 131400h
      pathlength: 1
idemix:
  rhpoolsize: 1000
  nonceexpiration: 15s
  noncesweepinterval: 15m
bccsp:
    default: SW
    sw:
        hash: SHA2
        security: 256
        filekeystore:
            keystore: msp/keystore
cacount:
cafiles:
intermediate:
  parentserver:
    url:
    caname:
  enrollment:
    hosts:
    profile:
    label:
  tls:
    certfiles:
    client:
      certfile:
      keyfile:
cfg:
  identities:
    passwordattempts: 10
operations:
    listenAddress: 127.0.0.1:9443
    tls:
        enabled: false
        cert:
            file:
        key:
            file:
        clientAuthRequired: false
        clientRootCAs:
            files: []
metrics:
    provider: disabled
    statsd:
        network: udp
        address: 127.0.0.1:8125
        writeInterval: 10s
        prefix: server

```



```shell
docker-compose up -d
```

# 三、启动 tlsCA 服务器 

创建tlsca-server目录，然后新建 docker-compose.yaml使用 docker-compose 来启动

编辑docker-compose.yaml

```yaml
fabric-tlsca-server:
   image: hyperledger/fabric-ca
   container_name: fabric-tlsca-server
   ports:
     - "8054:7054"
   environment:
     - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
   volumes:
     - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
   command: sh -c 'fabric-ca-server start -b admin:adminpw'
```

创建 fabric-ca-server目录，把 tlsca-cert.pem tlsca-key.pem文件复制到fabric-tlsca-server目录中，然后改名为ca-cert.pem ca-key.pem。

在  fabric-ca-server 目录下新建 fabric-ca-server-config.yaml 文件，内容同CA。



```shell
docker-compose up -d
```



**Farbic CA 客户端**

下一部分讲解如何使用fabric-ca-client的命令。

Fabric CA客户端的根目录定义规则如下：

- 如果存在环境变量 `FABRIC_CA_CLIENT_HOME` ，则使用这个值
- 否则，如果存在环境变量 `FABRIC_CA_HOME` ，则使用这个值
- 否则，如果存在环境变量 `CA_CFG_PATH` ，则使用这个值
- 否则，使用`$HOME/.fabric-ca-client`

下面的指引假设客户端的配置文件存在于客户端根目录。



# 四、orderer节点证书生成

```shell
export DOMAIN_NAME="my118.com"
export DOMAIN_NAME_REVERT="com.my118"

export CA_HOST_NAME="localhost"
export TLS_CA_HOST_NAME="localhost"
export CA_HOST_PORT="7054"
export TLS_CA_HOST_PORT="8054"

export CA_HOST="${CA_HOST_NAME}:${CA_HOST_PORT}"
export TLS_CA_HOST="${TLS_CA_HOST_NAME}:${TLS_CA_HOST_PORT}"

export ADMIN_NAME="admin"
export ADMIN_PWD="adminpw"

```



## 1、生成ordrer组织的msp

### 登陆管理员用户

只有先登录管理员用户，才能执行注册别的用户、客户端。

首先，你需要修改配置文件 fabric-ca-client-config.yaml 来设置注册时一些字段的默认值。

这个也可以在登录后修改，然后重新登录。

```
url: http://localhost:7054

mspdir: msp
tls:
  certfiles:
  client:
    certfile:
    keyfile:

csr:
  cn: admin2
  keyrequest:
    algo: ecdsa
    size: 256
  serialnumber:
  names:
    - C: CN
      ST: BJ
      L:
      O: my118.om
#      OU: Fabric
  hosts:
    - ubuntu-xenial

id:
  type: user
  affiliation:
  maxenrollments: 0
  attributes:

enrollment:
  profile:
  label:

caname:

bccsp:
    default: SW
    sw:
        hash: SHA2
        security: 256
        filekeystore:
            keystore: msp/keystore
```

运行`fabric-ca-client enroll`命令来登录一个身份。比如，下面的命令向一个本地运行在7054端口的Fabric CA服务端，登录了一个ID为admin，password为adminpw的身份。



```shell
fabric-ca-client enroll -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca
```




登录命令会存储一个登录证书（ECert），相对应的私钥，还有CA证书链PEM文件。这些存储在Fabric CA客户端的msp目录的子目录下，你会看到信息提示PEM存储在哪里。




## 2、生成Admin用户证书

```shell
export USERNAME="Admin"
```



### 注册用户Admin

```shell

fabric-ca-client register --id.name ${USERNAME}@${DOMAIN_NAME} --id.type client --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca

```

密码会被打印出来，登录这个新注册的身份的时候，需要用到这个密码。这允许一个管理员注册身份，然后把这个身份的ID和密码给别人来登陆。

```shell
2019/11/12 14:55:51 [INFO] Configuration file location: /home/vagrant/fabric-ca-client-config.yaml
Password: 123456
```

登录证书：

```shell
fabric-ca-client enroll -u http://${USERNAME}@${DOMAIN_NAME}:123456@${CA_HOST} --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp --home ~/fabric-ca
```

查看证书：

```
openssl x509 -text -noout -in ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem

```



###  生成Admin msp目录

```shell
mkdir ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp/admincerts

mkdir ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USERNAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/msp/admincerts
```



## 生成其它用户证书

```shell
export USERNAME="otheruser"
```

执行以上注册登记步骤即可。



## 3、生成orderer0 的 MSP

```shell
export ORDERERNAME="orderer0"
```



### 注册 orderer0 

```shell
fabric-ca-client register --id.name ${ORDERERNAME}.${DOMAIN_NAME} --id.type orderer --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=${ORDERERNAME}.${DOMAIN_NAME} --csr.hosts=["${ORDERERNAME}.${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca
```

### 登记orderer0 

```shell
fabric-ca-client enroll -u http://${ORDERERNAME}.${DOMAIN_NAME}:123456@${CA_HOST} --csr.cn=${ORDERERNAME}.${DOMAIN_NAME} --csr.hosts=['${ORDERERNAME}.${DOMAIN_NAME}'] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/msp --home ~/fabric-ca
```

###   生成msp 目录

```shell
mkdir ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/Admin@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/msp/admincerts
```



## 生成orderer1 的msp

```shell
export ORDERERNAME="orderer1"
```

然后重复执行上面的步骤



# 五、orderer节点tls证书生成



## 1、生成orderer组织的tls

### 登记orderer组织

```shell
fabric-ca-client enroll -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```




## 2、生成Admin的tls

```shell
export USER_NAME="Admin"
```



### 注册Admin

```shell
fabric-ca-client register --id.name ${USER_NAME}@${DOMAIN_NAME} --id.type client --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```



### 登记Admin

```shell
fabric-ca-client enroll -d --enrollment.profile tls -u http://${USER_NAME}@${DOMAIN_NAME}:123456@localhost:8054 --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls --home ~/fabric-ca
```




### 生成Admin 的 tls 目录

```shell
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/tlscacerts/tls-localhost-8054.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/ca.crt
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/signcerts/cert.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/client.crt
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/keystore/*_sk ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/client.key
```

## 3、生成orderer0的tls

```shell
export ORDERERNAME="orderer0"
```



### 注册orderer0

```shell
fabric-ca-client register --id.name ${ORDERERNAME}.${DOMAIN_NAME} --id.type orderer --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=${ORDERERNAME}.${DOMAIN_NAME} --csr.hosts=["${ORDERERNAME}.${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```




### 登记orderer0

```shell
fabric-ca-client enroll  --enrollment.profile tls -u http://${ORDERERNAME}.${DOMAIN_NAME}:123456@localhost:8054 --csr.cn=${ORDERERNAME}.${DOMAIN_NAME} --csr.hosts=["${ORDERERNAME}.${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls --home ~/fabric-ca
```




### 生成tls目录



```shell
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/tlscacerts/tls-localhost-8054.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/ca.crt
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/signcerts/cert.pem ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/server.crt
cp ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/keystore/*_sk ~/fabric-ca/crypto-config/ordererOrganizations/${DOMAIN_NAME}/orderers/${ORDERERNAME}.${DOMAIN_NAME}/tls/server.key
```



## 生成orderer1的msp

```shell
export ORDERERNAME="orderer1"
```

然后执行以上步骤



## 清理没用的文件

最后阶段数据清理,删除所有的 IssuerPublicKey  IssuerRevocationPublicKey

```shell
find ~/fabric-ca/crypto-config/ -name "IssuerPublicKey" --exec rm {} \;
find ~/fabric-ca/crypto-config/ -name "IssuerRevocationPublicKey" -exec rm {} \;
```



---



# 六、联盟成员 peer节点的证书生成

```shell
export DOMAIN_NAME="customs.gov.cn"
export DOMAIN_NAME_REVERT="cn.gov.customs"

export DOMAIN_NAME="my118.com"
export DOMAIN_NAME_REVERT="com.my118"

export CA_HOST_NAME="localhost"
export TLS_CA_HOST_NAME="localhost"
export CA_HOST_PORT="7054"
export TLS_CA_HOST_PORT="8054"

export CA_HOST="${CA_HOST_NAME}:${CA_HOST_PORT}"
export TLS_CA_HOST="${TLS_CA_HOST_NAME}:${TLS_CA_HOST_PORT}"

export ADMIN_NAME="admin"
export ADMIN_PWD="adminpw"
```



联盟组织成员需要生成自己的CA、tlsCA证书，然后启动CA、tlsCA服务，然后才能执行证书生成过程。



## 1、生成组织证书

### 登陆管理员用户

```shell
fabric-ca-client enroll -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca
```

## 2、生成Admin证书

你需要修改配置文件 fabric-ca-client-config.yaml 来设置注册时一些字段的默认值。

```yaml
url: http://localhost:7054

mspdir: msp
tls:
  certfiles:
  client:
    certfile:
    keyfile:

csr:
  cn: admin2
  keyrequest:
    algo: ecdsa
    size: 256
  serialnumber:
  names:
    - C: CN
      ST: BJ
      L:
      O: example.com
#      OU: Fabric
  hosts:
    - ubuntu-xenial

id:
  type: user
  affiliation:
  maxenrollments: 0
  attributes:

enrollment:
  profile:
  label:

caname:

bccsp:
    default: SW
    sw:
        hash: SHA2
        security: 256
        filekeystore:
            keystore: msp/keystore

```

### 生成Admin用户

```shell
export USER_NAME="Admin"
```



### 注册用户Admin

如果节点组织是orderer组织， fabric-ca-client register 操作就直接忽略，因为Admin用户已经注册。

```shell
fabric-ca-client register --id.name ${USER_NAME}@${DOMAIN_NAME} --id.type client --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca

```

密码会被打印出来，登录这个新注册的身份的时候，需要用到这个密码。这允许一个管理员注册身份，然后把这个身份的ID和密码给别人来登陆。

```shell
2019/11/12 14:55:51 [INFO] Configuration file location: /home/vagrant/fabric-ca-client-config.yaml
Password: 123456
```

登录证书：

```shell
fabric-ca-client enroll -u http://${USER_NAME}@${DOMAIN_NAME}:123456@${CA_HOST} --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp --home ~/fabric-ca
```

查看证书：

```shell
openssl x509 -text -noout -in ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem

```



###  生成Admin msp目录

```shell
mkdir ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp/admincerts

mkdir ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/msp/admincerts
```



## 3、生成peer0 的 MSP

```shell
export PEERNAME="peer0"
```



### 注册 peer0

```shell
fabric-ca-client register --id.name ${PEERNAME}.${DOMAIN_NAME} --id.type peer --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=${PEERNAME}.${DOMAIN_NAME} --csr.hosts=['${PEERNAME}.${DOMAIN_NAME}'] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/msp -u http://${ADMIN_NAME}:${ADMIN_PWD}@${CA_HOST} --home ~/fabric-ca
```

### 登记peer0

```shell
fabric-ca-client enroll -u http://${PEERNAME}.${DOMAIN_NAME}:123456@${CA_HOST} --csr.cn=${PEERNAME}.${DOMAIN_NAME} --csr.hosts=["${PEERNAME}.${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/msp --home ~/fabric-ca
```

###   生成msp 目录

```shell
mkdir ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/msp/admincerts
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/Admin@${DOMAIN_NAME}/msp/signcerts/cert.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/msp/admincerts
```



## 生成peer1的msp
```shell
export PEERNAME="peer1"
```
重复以上步骤



# 七、联盟成员 peer节点tls证书生成

## 1、生成组织msp

### 登记组织

```shell
fabric-ca-client enroll -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```



## 2、生成Admin的tls

```shell
export USER_NAME="Admin"
```



### 注册Admin

如果节点组织是orderer组织， fabric-ca-client register 操作就直接忽略，因为Admin用户已经注册。

```shell
fabric-ca-client register --id.name ${USER_NAME}@${DOMAIN_NAME} --id.type client --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,role=admin:ecert' --id.secret=123456 --csr.cn=${DOMAIN_NAME} --csr.hosts=["${DOMAIN_NAME}"] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```



### 登记Admin

```shell
fabric-ca-client enroll  --enrollment.profile tls -u http://${USER_NAME}@${DOMAIN_NAME}:123456@${TLS_CA_HOST} --csr.cn=${DOMAIN_NAME} --csr.hosts=['${DOMAIN_NAME}'] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls --home ~/fabric-ca
```



### 生成Admin 的 tls 目录

```shell
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/tlscacerts/tls-localhost-8054.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/ca.crt
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/signcerts/cert.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/client.crt
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/keystore/*_sk ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/users/${USER_NAME}@${DOMAIN_NAME}/tls/client.key

```



## 3、生成peer0的tls

```shell
export PEERNAME="peer0"
```

### 注册peer0

```shell
fabric-ca-client register --id.name ${PEERNAME}.${DOMAIN_NAME} --id.type peer --id.affiliation "${DOMAIN_NAME_REVERT}" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=${PEERNAME}.${DOMAIN_NAME} --csr.hosts=['${PEERNAME}.${DOMAIN_NAME}'] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/tls -u http://${ADMIN_NAME}:${ADMIN_PWD}@${TLS_CA_HOST} --home ~/fabric-ca
```



### 登记peer0

```shell
fabric-ca-client enroll --enrollment.profile tls -u http://${PEERNAME}.${DOMAIN_NAME}:123456@${TLS_CA_HOST} --csr.cn=${PEERNAME}.${DOMAIN_NAME} --csr.hosts=['${PEERNAME}.${DOMAIN_NAME}'] -M ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls --home ~/fabric-ca
```

### 生成tls目录

```shell
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/tlscacerts/tls-localhost-8054.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/ca.crt
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/signcerts/cert.pem ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/server.crt
cp ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/keystore/*_sk ~/fabric-ca/crypto-config/peerOrganizations/${DOMAIN_NAME}/peers/${PEERNAME}.${DOMAIN_NAME}/tls/server.key
```



## 生成 peer1 的msp
```shell
export PEERNAME="peer1"
```
重复以上步骤



# 八、用每个组织的证书生成网络创世块







# 九、附录

## 重新登陆一个身份

假设你的登陆证书快过期了，你可以重新登陆来替换你的登陆证书（ECert）。

```shell
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer
fabric-ca-client reenroll
```



## 撤销一个证书或身份

身份和证书都能被撤销。撤销一个身份会撤销该身份拥有的所有证书，该身份也不能再获得新的证书。撤销一个证书会使该证书失效。

为了撤销一个证书或身份，发起者必须有hf.Revoker属性。发起者只能撤销与自己的affiliation相同的证书或身份，或者发起者的affiliation是被撤销者的affiliation的前缀。

举个例子，一个“orgs.org1”的发起者只能撤销orgs.org1或者orgs.org1.department1的身份，而不能撤销orgs.org2的身份。



下面的命令撤销一个身份。将来所有发自该身份的请求都会被Fabric CA服务器拒收。

```bash
fabric-ca-client revoke -e <enrollment_id> -r <reason>
```

下面是`-r`选项支持的理由：

1. unspecified
2. keycompromise
3. cacompromise
4. affiliationchange
5. superseded
6. cessationofoperation
7. certificatehold
8. removefromcrl
9. privilegewithdrawn
10. aacompromise

举个例子，有着根affiliation的admin可以回收peer1身份：

```bash
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client revoke -e peer1
```

An enrollment certificate that belongs to an identity can be revoked by specifying its AKI (Authority Key Identifier) and serial number as follows:

一个身份可以撤销自己的登陆证书（ECert），需要指定ECert的AKI和序列号：

```bash
fabric-ca-client revoke -a xxx -s yyy -r <reason>
```

For example, you can get the AKI and the serial number of a certificate using the openssl command and pass them to the revoke command to revoke the said certificate as follows:

举个例子，你可以通过openssl命令来获取一个证书的AKI和序列号：

    serial=$(openssl x509 -in userecert.pem -serial -noout | cut -d "=" -f 2)
    aki=$(openssl x509 -in userecert.pem -text | awk '/keyid/ {gsub(/ *keyid:|:/,"",$1);print tolower($0)}')
    fabric-ca-client revoke -s $serial -a $aki -r affiliationchange

### 启用TLS

这一部分介绍如何为Fabric CA客户端配置TLS。

The following sections may be configured in the `fabric-ca-client-config.yaml`.

下面的可以配置在`fabric-ca-client-config.yaml`中。

```yaml
tls:
    # Enable TLS (default: false)
    enabled: true
    certfiles:
        - root.pem
    client:
        certfile: tls_client-cert.pem
        keyfile: tls_client-key.pem
```

*certfiles*是该客户端信任的根证书集合。一般这都会是Fabric CA服务端根目录下的ca-cert.pem。

The client option is required only if mutual TLS is configured on the server.

只有在服务器配置了双向TLS的情况下，*client*选项才需要。

----





## 体验 Fabric CA 命令行

这一部分提供 fabric-ca-server 和 fabric-ca-client 命令行的使用说明。另外的使用信息会在接下来的内容中提供。

fabric-ca-server

```
Hyperledger Fabric Certificate Authority Server

Usage:
    fabric-ca-server [command]

Available Commands:
    init        Initialize the Fabric CA server
    start       Start the Fabric CA server

Flags:
        --address string                         Listening address of Fabric CA server (default "0.0.0.0")
    -b, --boot string                            The user:pass for bootstrap admin which is required to build default config file
        --ca.certfile string                     PEM-encoded CA certificate file (default "ca-cert.pem")
        --ca.chainfile string                    PEM-encoded CA chain file (default "ca-chain.pem")
        --ca.keyfile string                      PEM-encoded CA key file (default "ca-key.pem")
    -n, --ca.name string                         Certificate Authority name
    -c, --config string                          Configuration file (default "fabric-ca-server-config.yaml")
        --csr.cn string                          The common name field of the certificate signing request to a parent Fabric CA server
        --csr.hosts stringSlice                  A list of space-separated host names in a certificate signing request to a parent Fabric CA server
        --csr.serialnumber string                The serial number in a certificate signing request to a parent Fabric CA server
        --db.datasource string                   Data source which is database specific (default "fabric-ca-server.db")
        --db.tls.certfiles stringSlice           PEM-encoded list of trusted certificate files
        --db.tls.client.certfile string          PEM-encoded certificate file when mutual authenticate is enabled
        --db.tls.client.keyfile string           PEM-encoded key file when mutual authentication is enabled
        --db.type string                         Type of database; one of: sqlite3, postgres, mysql (default "sqlite3")
    -d, --debug                                  Enable debug level logging
        --ldap.enabled                           Enable the LDAP client for authentication and attributes
        --ldap.groupfilter string                The LDAP group filter for a single affiliation group (default "(memberUid=%s)")
        --ldap.url string                        LDAP client URL of form ldap://adminDN:adminPassword@host[:port]/base
        --ldap.userfilter string                 The LDAP user filter to use when searching for users (default "(uid=%s)")
    -p, --port int                               Listening port of Fabric CA server (default 7054)
        --registry.maxenrollments int            Maximum number of enrollments; valid if LDAP not enabled
        --tls.certfile string                    PEM-encoded TLS certificate file for server's listening port (default "ca-cert.pem")
        --tls.clientauth.certfiles stringSlice   PEM-encoded list of trusted certificate files
        --tls.clientauth.type string             Policy the server will follow for TLS Client Authentication. (default "noclientcert")
        --tls.enabled                            Enable TLS on the listening port
        --tls.keyfile string                     PEM-encoded TLS key for server's listening port (default "ca-key.pem")
    -u, --url string                             URL of the parent Fabric CA server
```

```
Use "fabric-ca-server [command] --help" for more information about a command.
```

fabric-ca-client

```
# fabric-ca-client
Hyperledger Fabric Certificate Authority Client

Usage:
    fabric-ca-client [command]

Available Commands:
    enroll      Enroll an identity
    getcacert   Get CA certificate chain
    reenroll    Reenroll an identity
    register    Register an identity
    revoke      Revoke an identity

Flags:
    -c, --config string                Configuration file (default "$HOME/.fabric-ca-client/fabric-ca-client-config.yaml")
        --csr.cn string                The common name field of the certificate signing request
        --csr.hosts stringSlice        A list of space-separated host names in a certificate signing request
        --csr.serialnumber string      The serial number in a certificate signing request
    -d, --debug                        Enable debug level logging
        --enrollment.hosts string      Comma-separated host list
        --enrollment.label string      Label to use in HSM operations
        --enrollment.profile string    Name of the signing profile to use in issuing the certificate
        --id.affiliation string        The identity's affiliation
        --id.attr string               Attributes associated with this identity (e.g. hf.Revoker=true)
        --id.maxenrollments int        The maximum number of times the secret can be reused to enroll
        --id.name string               Unique name of the identity
        --id.secret string             The enrollment secret for the identity being registered
        --id.type string               Type of identity being registered (e.g. 'peer, app, user')
    -M, --mspdir string                Membership Service Provider directory (default "msp")
    -m, --myhost string                Hostname to include in the certificate signing request during enrollment (default "$HOSTNAME")
        --tls.certfiles stringSlice    PEM-encoded list of trusted certificate files
        --tls.client.certfile string   PEM-encoded certificate file when mutual authenticate is enabled
        --tls.client.keyfile string    PEM-encoded key file when mutual authentication is enabled
    -u, --url string                   URL of the Fabric CA server (default "http://${CA_HOST}")

Use "fabric-ca-client [command] --help" for more information about a command.
```

Note that command line options that are string slices (lists) can be specified either by specifying the option with space-separated list elements or by specifying the option multiple times, each with a string value that make up the list. For example, to specify host1 and host2 for csr.hosts option, you can either pass –csr.hosts “host1 host2” or –csr.hosts host1 –csr.hosts host2

注意在命令行中需要给某个选项输入列表时，可以用空格分割，或者多次使用该选项。例如，指定`host1`和`host2`给csr.hosts选项，你可以用–csr.hosts “host1 host2”或者–csr.hosts host1 –csr.hosts host2





## Fabric CA 服务端配置文件格式

配置文件可以通过 `-c` 或者 `--config` 选项来指定。如果 `--config` 选项使用了，而指定的文件不存在，默认的配置文件（如下）会在指定的位置被创建。然而，如果没有指定配置文件，默认配置文件会在 fabric-ca-server 的 home 目录下创建。（参考 [Fabric CA 服务端](#fabric-ca-服务端) 了解更多）

```yaml
# Server's listening port (default: 7054)
port: 7054

# Enables debug logging (default: false)
debug: false

#############################################################################
#  TLS section for the server's listening port
#############################################################################
tls:
    # Enable TLS (default: false)
    enabled: false
    certfile: ca-cert.pem
    keyfile: ca-key.pem

#############################################################################
#  The CA section contains the key and certificate files used when
#  issuing enrollment certificates (ECerts) and transaction
#  certificates (TCerts).
#############################################################################
ca:
    # Certificate file (default: ca-cert.pem)
    certfile: ca-cert.pem
    # Key file (default: ca-key.pem)
    keyfile: ca-key.pem

#############################################################################
#  The registry section controls how the Fabric CA server does two things:
#  1) authenticates enrollment requests which contain identity name and
#     password (also known as enrollment ID and secret).
#  2) once authenticated, retrieves the identity's attribute names and
#     values which the Fabric CA server optionally puts into TCerts
#     which it issues for transacting on the Hyperledger Fabric blockchain.
#     These attributes are useful for making access control decisions in
#     chaincode.
#  There are two main configuration options:
#  1) The Fabric CA server is the registry
#  2) An LDAP server is the registry, in which case the Fabric CA server
#     calls the LDAP server to perform these tasks.
#############################################################################
registry:
    # Maximum number of times a password/secret can be reused for enrollment
    # (default: 0, which means there is no limit)
    maxEnrollments: 0

    # Contains identity information which is used when LDAP is disabled
    identities:
        - name: <<<ADMIN>>>
        pass: <<<ADMINPW>>>
        type: client
        affiliation: ""
        attrs:
            hf.Registrar.Roles: "client,user,peer,validator,auditor,ca"
            hf.Registrar.DelegateRoles: "client,user,validator,auditor"
            hf.Revoker: true
            hf.IntermediateCA: true

#############################################################################
#  Database section
#  Supported types are: "sqlite3", "postgres", and "mysql".
#  The datasource value depends on the type.
#  If the type is "sqlite3", the datasource value is a file name to use
#  as the database store.  Since "sqlite3" is an embedded database, it
#  may not be used if you want to run the Fabric CA server in a cluster.
#  To run the Fabric CA server in a cluster, you must choose "postgres"
#  or "mysql".
#############################################################################
db:
    type: sqlite3
    datasource: fabric-ca-server.db
    tls:
        enabled: false
        certfiles:
            - db-server-cert.pem
        client:
            certfile: db-client-cert.pem
            keyfile: db-client-key.pem

#############################################################################
#  LDAP section
#  If LDAP is enabled, the Fabric CA server calls LDAP to:
#  1) authenticate enrollment ID and secret (i.e. identity name and password)
#     for enrollment requests
#  2) To retrieve identity attributes
#############################################################################
ldap:
    # Enables or disables the LDAP client (default: false)
    enabled: false
    # The URL of the LDAP server
    url: ldap://<adminDN>:<adminPassword>@<host>:<port>/<base>
    tls:
        certfiles:
            - ldap-server-cert.pem
        client:
            certfile: ldap-client-cert.pem
            keyfile: ldap-client-key.pem

#############################################################################
#  Affiliation section
#############################################################################
affiliations:
    org1:
        - department1
        - department2
    org2:
        - department1

#############################################################################
#  Signing section
#############################################################################
signing:
    profiles:
        ca:
            usage:
            - cert sign
            expiry: 8000h
            caconstraint:
            isca: true
    default:
        usage:
            - cert sign
        expiry: 8000h

###########################################################################
#  Certificate Signing Request section for generating the CA certificate
###########################################################################
csr:
    cn: fabric-ca-server
    names:
        - C: US
            ST: North Carolina
            L:
            O: Hyperledger
            OU: Fabric
    hosts:
        - <<<MYHOST>>>
    ca:
        pathlen:
        pathlenzero:
        expiry:

#############################################################################
#  Crypto section configures the crypto primitives used for all
#############################################################################
crypto:
    software:
        hash_family: SHA2
        security_level: 256
        ephemeral: false
        key_store_dir: keys
```

## Fabric CA 客户端配置文件格式

A configuration file can be provided to the client using the -c or --config option. If the config option is used and the specified file doesn’t exist, a default configuration file (like the one shown below) will be created in the specified location. However, if no config option was used, it will be created in the client’s home directory (see Fabric CA Client section more info).

配置文件可以通过 `-c` 或者 `--config` 选项来指定。如果 `--config` 选项使用了，而指定的文件不存在，默认的配置文件（如下）会在指定的位置被创建。然而，如果没有指定配置文件，默认配置文件会在 fabric-ca-client 的 home 目录下创建。（参考 [Fabric CA 客户端](#fabric-ca-客户端) 了解更多）

```yaml
#############################################################################
# Client Configuration
#############################################################################

# URL of the fabric-ca-server (default: http://${CA_HOST})
URL: http://${CA_HOST}

# Membership Service Provider (MSP) directory
# When the client is used to enroll a peer or an orderer, this field must be
# set to the MSP directory of the peer/orderer
MSPDir:

#############################################################################
#    TLS section for secure socket connection
#############################################################################
tls:
    # Enable TLS (default: false)
    enabled: false
    certfiles:   # Comma Separated (e.g. root.pem, root2.pem)
    client:
        certfile:
        keyfile:

#############################################################################
#  Certificate Signing Request section for generating the CSR for
#  an enrollment certificate (ECert)
#############################################################################
csr:
    cn: <<<ENROLLMENT_ID>>>
    names:
        - C: US
        ST: North Carolina
        L:
        O: Hyperledger
        OU: Fabric
    hosts:
    - <<<MYHOST>>>
    ca:
        pathlen:
        pathlenzero:
        expiry:

#############################################################################
#  Registration section used to register a new user with fabric-ca server
#############################################################################
id:
    name:
    type:
    affiliation:
    attrs:
        - name:
        value:

#############################################################################
#  Enrollment section used to enroll a user with fabric-ca server
#############################################################################
enrollment:
    hosts:
    profile:
    label:
```



## 配置优先级说明

Fabric CA 提供3种方式来配置 fabric-ca-server 和 fabric-ca-client 。优先级如下：

1. 命令行参数
2. 环境变量
3. 配置文件

接下来，我们试着对配置文件进行修改。配置文件的修改可以被环境变量或者命令行参数覆盖。

举个例子，假设我们有以下客户端的配置文件：

```yaml
tls:
    # Enable TLS (default: false)
    enabled: false

    # TLS for the client's listenting port (default: false)
    certfiles:   # Comma Separated (e.g. root.pem, root2.pem)
    client:
        certfile: cert.pem
        keyfile:
```



可以用以下环境变量来覆盖配置文件中 `cert.pem` 的配置

`export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem`

如果我们想同时覆盖环境变量和配置文件，可以通过命令行参数

`fabric-ca-client enroll --tls.client.certfile cert3.pem`

以上方法对fabric-ca-server同样适用，区别是在环境变量的前缀，把`FABIRC_CA_CLIENT`替换为`FABRIC_CA_SERVER`。

## 关于路径的一些说明

fabric-ca-server 和 fabirc-ca-client 的配置文件里的所有属性都支持相对路径和绝对路径。相对路径是相对于配置目录，即配置文件所在的目录。比如，如果配置目录是 `~/config` ，而 tls 部分的配置如下所示，那么Fabric CA 服务端或者客户端会在 `~/config` 目录下查找 `root.pem` ，在 `~/config／certs` 下查找 `cert.pem` ，在 `/abs/path` 目录下查找 `key.pem`

```
tls:
    enabled: true
    certfiles:   root.pem
    client:
        certfile: certs/cert.pem
        keyfile: /abs/path/key.pem
```

## Fabric CA 服务端



这一部分说明Fabric CA服务端。

在启动Fabric CA服务端之前，你可以先初始化Fabric CA服务端。通过初始化，程序会自动生成一份默认配置文件，方便用户在启动前自行修改一些配置项。

Fabric CA服务端的根目录通过以下方式决定：

- 如果存在环境变量 `FABRIC_CA_SERVER_HOME` ，则使用这个值
- 否则，如果存在环境变量 `FABRIC_CA_HOME` ，则使用这个值
- 否则，如果存在环境变量 `CA_CFG_PATH` ，则使用这个值
- 否则，使用当前的工作目录作为根目录

本章接下来的部分，我们假设你已经设置了环境变量 `FABRIC_CA_HOME` ，并且值设置为 `$HOME/fabric-ca/server`。

接下来的内容都默认服务端配置文件存在于服务端根目录下。

## 初始化服务端

用以下命令初始化Fabric CA服务端

```shell
fabric-ca-server init -b ${ADMIN_NAME}:${ADMIN_PWD}
```



初始化时必须提供 `-b` (bootstrap identity: 引导身份) 选项。启动服务端时，至少需要提供一个引导身份。服务端配置文件包含一个可以配置的证书签名请求 (Certificate Signing Request, CSR) 部分。下面是一个CSR的例子。

如果你想通过TLS远程连接Fabric CA服务端，把下面的CSR配置内容中的"localhost"替换成你将会运行Fabric CA服务端的域名。

```
cn: localhost
key:
    algo: ecdsa
    size: 256
names:
- C: US
    ST: "North Carolina"
    L:
    O: Hyperledger
    OU: Fabric
```

上面所有的字段都符合X.509签名与证书规范，可以由`fabric-ca-server` init命令来生成。服务端配置文件中提到的`ca.certfile`和`ca.keyfile`这两个文件也符合X.509规范。字段如下：

- cn 通用名
- key 指明算法和密钥长度
- O 组织
- OU 组织单位
- L 地址或城市
- ST 州（省）
- C 国家

如果需要修改CSR里面的值，你可以修改配置文件，然后把配置中由`ca.certfile`和`ca.keyfile`这两项指明的文件删除。然后再运行一次 `fabric-ca-server init -b ${ADMIN_NAME}:${ADMIN_PWD}` 命令。

`fabirc-ca-server init` 命令生成一个自签名的CA证书，除非使用了 `-u <parent-fabric-ca-server-URL>` 选项。如果使用了 `-u` 选项，本服务端CA证书会由父Fabric CA服务端签名。`fabirc-ca-server init` 命令同时也会在服务端的根目录生成一个默认配置文件*fabric-ca-server-config.yaml*。

算法和密钥长度

可以通过自定义CSR来生成支持ECDSA和RSA的X.509证书和密钥。以下是一个椭圆曲线数字签名算法（ECDSA）的设置，采用的曲线是`prime256v1`，签名算法是`ecdsa-with-SHA256`。

```
key:
    algo: ecdsa
    size: 256
```

对算法和密钥长度的选择取决于你对安全的考量。

ECDSA提供以下选项：

| size |  ASN1 OID  | Signature Algorithm |
| :--: | :--------: | :-----------------: |
| 256  | prime256v1 |  ecdsa-with-SHA256  |
| 384  | secp384r1  |  ecdsa-with-SHA384  |
| 521  | secp384r1  |  ecdsa-with-SHA521  |

RSA提供以下选项：

| size | Modulus (bits) |   Signature Algorithm   |
| :--: | :------------: | :---------------------: |
| 2048 |      2048      | sha256WithRSAEncryption |
| 4096 |      2096      | sha512WithRSAEncryption |

## 启动服务端



如果服务器之前没有初始化过，它会在第一次启动的时候进行初始化。在初始化的过程中，它会如果ca-cert.pem和ca-key.pem这两个文件不存在，它会生成这两个文件；如果配置文件不存在，它会生成默认配置文件。参考[初始化服务端](#初始化服务端)。

除非Fabric CA服务端配置了使用LDAP，否则它必须配置至少一个预注册引导身份来允许你登录其他身份。`-b`选项指明引导身份的用户名和密码。

可以通过`-c`选项来指明其他的配置文件。

```
# fabric-ca-server start -c <path-to-config-file> -b <admin>:<adminpw>
```

为了使fabric ca server监听`https`而不是`http`，配置`tls.enabled`为`true`。

为了限制登录时相同密码的次数，可以在配置文件中设置`registry.maxEnrollments`为恰当的值。如果你设置这个值为1，Fabric CA服务端只允许一个密码被一个登录ID使用（即不会出现多个ID有相同的密码）。如果你设置这个值为0，Fabric CA服务端不会限制密码的重复使用次数。默认值为0。

Fabric CA服务端现在应该正在监听7054端口。


