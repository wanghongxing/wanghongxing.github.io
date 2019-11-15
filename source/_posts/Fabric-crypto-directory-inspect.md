---
title: Fabric CA Practice探索目录结构
date: 2019-11-12 09:48:03
tags:
---

首先生成所有企业的证书。

进入orderer证书目录

```
cd crypto-config/ordererOrganizations/my118.com/
tree
```

结果：

```
tree
.
├── ca
│   ├── 6ebc1d3414ee1448013d4f6aef49815177a6ba385cd50a4374ac44b73c8213db_sk
│   └── ca.my118.com-cert.pem
├── msp
│   ├── admincerts
│   │   └── Admin@my118.com-cert.pem
│   ├── cacerts
│   │   └── ca.my118.com-cert.pem
│   └── tlscacerts
│       └── tlsca.my118.com-cert.pem
├── orderers
│   ├── orderer0.my118.com
│   │   ├── msp
│   │   │   ├── admincerts
│   │   │   │   └── Admin@my118.com-cert.pem
│   │   │   ├── cacerts
│   │   │   │   └── ca.my118.com-cert.pem
│   │   │   ├── keystore
│   │   │   │   └── ec64443141408b15f94ab8ea364ce2669d442d33208f14dae7a7c0dab5dbd49d_sk
│   │   │   ├── signcerts
│   │   │   │   └── orderer0.my118.com-cert.pem
│   │   │   └── tlscacerts
│   │   │       └── tlsca.my118.com-cert.pem
│   │   └── tls
│   │       ├── ca.crt
│   │       ├── server.crt
│   │       └── server.key
│   └── orderer1.my118.com
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@my118.com-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.my118.com-cert.pem
│       │   ├── keystore
│       │   │   └── c3e47367fc9d6d16b56eb61c80ed0f59758235c5d6e147425d9e743693e88093_sk
│       │   ├── signcerts
│       │   │   └── orderer1.my118.com-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.my118.com-cert.pem
│       └── tls
│           ├── ca.crt
│           ├── server.crt
│           └── server.key
├── tlsca
│   ├── 30a9aab0ad7155e3a54d5839f6a80fbe472a0ab8a2103e0b763c5aa1b85d64fd_sk
│   └── tlsca.my118.com-cert.pem
└── users
    └── Admin@my118.com
        ├── msp
        │   ├── admincerts
        │   │   └── Admin@my118.com-cert.pem
        │   ├── cacerts
        │   │   └── ca.my118.com-cert.pem
        │   ├── keystore
        │   │   └── 82b660a4e12a2285c926a11e3356258b161cb3c1c7edb84711b3db968e83d075_sk
        │   ├── signcerts
        │   │   └── Admin@my118.com-cert.pem
        │   └── tlscacerts
        │       └── tlsca.my118.com-cert.pem
        └── tls
            ├── ca.crt
            ├── client.crt
            └── client.key
```

**查看ca证书**

```
cd ca/
openssl x509 -text -in ca.my118.com-cert.pem
```

结果：

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            e8:42:b4:2b:42:c7:6b:5e:ca:04:2c:d1:7e:a1:26:6a
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=CN, ST=TJ, L=TJ, O=my118.com, CN=ca.my118.com
        Validity
            Not Before: Nov 11 14:03:00 2019 GMT
            Not After : Nov  8 14:03:00 2029 GMT
        Subject: C=CN, ST=TJ, L=TJ, O=my118.com, CN=ca.my118.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:e9:ff:1b:61:6c:83:ef:1e:92:b8:f1:f5:80:25:
                    14:5d:cd:14:dc:3b:37:50:8d:c7:cb:17:42:f7:5b:
                    63:8a:43:9f:ff:6c:92:dd:85:88:f6:2e:a0:7e:1a:
                    28:55:f2:61:7d:c7:f4:46:74:6f:c2:e6:71:d1:c6:
                    c0:96:2d:bc:18
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign, CRL Sign
            X509v3 Extended Key Usage:
                TLS Web Client Authentication, TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                6E:BC:1D:34:14:EE:14:48:01:3D:4F:6A:EF:49:81:51:77:A6:BA:38:5C:D5:0A:43:74:AC:44:B7:3C:82:13:DB
    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:63:59:6a:ca:f3:2f:0d:80:db:79:a3:a4:21:cd:
         aa:75:98:7f:58:f1:40:f9:a0:44:94:c0:7e:6e:c5:d2:9f:65:
         02:20:00:91:7f:e8:63:77:b6:60:fa:eb:60:bf:74:07:59:82:
         c4:40:ef:05:4c:e9:82:64:1a:26:03:fc:bd:70:ae:6e
```

说明ca证书是一个字签名的证书。



**然后查看tlsca证书**

```
cd tlsca
ls
openssl x509 -text -in tlsca.my118.com-cert.pem
```

结果：

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            83:bb:13:c5:5f:79:ff:33:85:9f:6a:7b:7c:fc:8b:a7
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=CN, ST=TJ, L=TJ, O=my118.com, CN=tlsca.my118.com
        Validity
            Not Before: Nov 11 14:03:00 2019 GMT
            Not After : Nov  8 14:03:00 2029 GMT
        Subject: C=CN, ST=TJ, L=TJ, O=my118.com, CN=tlsca.my118.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:97:39:49:f2:19:1e:7c:50:c3:57:d8:e8:87:b4:
                    fb:d2:0a:57:dd:61:21:3e:6d:18:11:71:a9:79:56:
                    0d:37:e6:9d:07:b0:d1:19:74:3e:ee:10:37:8d:97:
                    cf:43:3a:52:70:70:6a:69:ae:cc:9f:56:b4:82:3b:
                    c1:72:a7:77:af
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign, CRL Sign
            X509v3 Extended Key Usage:
                TLS Web Client Authentication, TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                30:A9:AA:B0:AD:71:55:E3:A5:4D:58:39:F6:A8:0F:BE:47:2A:0A:B8:A2:10:3E:0B:76:3C:5A:A1:B8:5D:64:FD
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:b0:07:80:21:5f:ab:8e:6c:2a:91:66:9f:bb:
         d2:31:77:28:7b:e5:48:69:1a:74:df:de:36:d0:30:cc:aa:9d:
         82:02:20:16:b3:32:9a:3b:b5:23:88:6f:79:94:36:07:15:9c:
         6a:bb:e9:59:bd:fb:61:d0:99:73:ff:62:21:78:86:50:2c
```

说明tlsca也是一个自签名的证书。



查看users目录

```
# cd users/Admin@my118.com
# tree
├── msp
│   ├── admincerts
│   │   └── Admin@my118.com-cert.pem
│   ├── cacerts
│   │   └── ca.my118.com-cert.pem
│   ├── keystore
│   │   └── 82b660a4e12a2285c926a11e3356258b161cb3c1c7edb84711b3db968e83d075_sk
│   ├── signcerts
│   │   └── Admin@my118.com-cert.pem
│   └── tlscacerts
│       └── tlsca.my118.com-cert.pem
└── tls
    ├── ca.crt
    ├── client.crt
    └── client.key

```



然后进入msp目录，查看所有证书，很明显keystore目录下放的是admin@my118.com私钥，cacerts tlscacert目录是orderer组织跟目录的ca和tlsca的证书。

比对admincerts和signcerts目录，这两个目录内容一致，都是被orderer ca签署的admin证书。

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                3f:08:d6:79:f0:56:27:9b:65:ea:0d:14:f8:78:44:91
        Signature Algorithm: ecdsa-with-SHA256
            Issuer: C = CN, ST = TJ, L = TJ, O = my118.com, CN = ca.my118.com
            Validity
                Not Before: Nov 11 14:03:00 2019 GMT
                Not After : Nov  8 14:03:00 2029 GMT
            Subject: C = CN, ST = TJ, L = TJ, CN = Admin@my118.com
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:ea:c4:07:f7:e7:52:82:ef:c5:ef:52:0f:02:2b:
                        c2:94:97:50:47:8c:16:32:89:48:9b:b5:2e:b4:50:
                        e3:6e:8c:1b:28:86:77:d6:e9:ff:c9:76:98:6e:c1:
                        92:84:20:ef:b4:ad:08:b4:d2:1c:2e:40:af:b4:d2:
                        cb:4d:0c:4f:68
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            X509v3 extensions:
                X509v3 Key Usage: critical
                    Digital Signature
                X509v3 Basic Constraints: critical
                    CA:FALSE
                X509v3 Authority Key Identifier:
                    keyid:6E:BC:1D:34:14:EE:14:48:01:3D:4F:6A:EF:49:81:51:77:A6:BA:38:5C:D5:0A:43:74:AC:44:B7:3C:82:13:DB
    
        Signature Algorithm: ecdsa-with-SHA256
             30:44:02:20:23:29:f3:7a:dd:d7:b1:c9:46:73:bd:b4:b4:16:
             cb:3c:5d:d5:d8:e6:85:08:b5:68:38:0f:93:43:77:be:0d:e0:
             02:20:61:07:93:ca:cc:37:a0:16:ff:bf:82:63:e8:14:11:ac:
             fc:24:2f:9e:34:81:52:55:47:ac:ac:df:64:7f:49:b7
继续查看Admin的tls目录

比对后发现，ca.crt就是orderer的tlsca证书。

client.key是为admin专门生成的tls传输用的私钥。

```

# openssl x509 -text -noout -in client.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            d9:32:95:b1:62:4e:e8:43:da:87:83:f2:23:c4:de:06
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C = CN, ST = TJ, L = TJ, O = my118.com, CN = tlsca.my118.com
        Validity
            Not Before: Nov 11 14:03:00 2019 GMT
            Not After : Nov  8 14:03:00 2029 GMT
        Subject: C = CN, ST = TJ, L = TJ, CN = Admin@my118.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:c6:c5:65:1e:62:6a:c5:53:ee:c5:a3:c8:6a:cd:
                    40:f7:00:30:7d:e8:e4:8c:88:b8:4b:05:b2:e2:3c:
                    82:32:09:19:34:f2:40:ba:a8:30:95:ff:b5:89:8b:
                    0c:e2:f3:c0:3b:0f:58:b8:46:74:a9:2b:d6:77:5c:
                    24:41:a2:74:d6
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:30:A9:AA:B0:AD:71:55:E3:A5:4D:58:39:F6:A8:0F:BE:47:2A:0A:B8:A2:10:3E:0B:76:3C:5A:A1:B8:5D:64:FD

    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:cb:73:df:c5:8e:23:4b:30:bb:ab:43:28:d4:
         4e:ec:26:4e:e8:38:a3:e2:03:a8:13:74:58:60:95:08:28:96:
         82:02:20:64:26:0c:1d:26:1a:a8:36:3d:6d:44:1c:01:0b:59:
         7e:ce:0a:c5:64:93:3a:06:c0:d7:13:7f:f6:ad:8f:f7:e4
```

说明client.crt是用orderer tlsCA签署的client证书，就是为了tls使用的。



查看msp目录，比对后发现cacerts  是复制ca目录，然后删除了私钥；

tlscacerts是复制了 tlsca目录，然后删除了私钥；

admincerts目录下放了Admin被CA签署的证书；**这个干啥不知道**

估计这三个个目录是要放到所有orderers的子目录中发放的，所以要删除私钥。

---

**每个组织的 msp 目录的作用**

经过查看configtxgen 过程，发现在configtx.yaml文件中，每个组织的msp目录都有配置

```yaml

    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg

        # ID to load the MSP definition as
        ID: OrdererMSP

        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: crypto-config/ordererOrganizations/my118.com/msp
```



```yaml
    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/my118.com/msp
        AnchorPeers:
            - Host: peer0.my118.com
              Port: 6051
        Policies:


```

然后在生成网络创世块的时候会读取每个组织的msp。

其中对于msp子目录的名称规定如下：

```

  cacerts              = "cacerts"
	admincerts           = "admincerts"
	signcerts            = "signcerts"
	keystore             = "keystore"
	intermediatecerts    = "intermediatecerts"
	crlsfolder           = "crls"
	configfilename       = "config.yaml"
	tlscacerts           = "tlscacerts"
	tlsintermediatecerts = "tlsintermediatecerts"
	
	
	
```

代码中对于cacerts 、admincert要求是必须有的，没有就提示出错。



在生成**网络**创世块的时候需要所有 Consortiums 组织的  msp。网络创世块也是用来创建一个特殊的系统通道，系统通道名字可以自定义。

~~在生成**通道**创世块的时候需要所有 Consortiums 组织的  msp。~~这个通道块里没有组织签名。

每个组织的msp其实就是组织的ca 自签名的公钥、tlsca自签名的公钥、被ca签名的管理员的公钥。这三个公钥应该是放到创世块中，当组织加入的时候来验证组织的合法性。

**这样就可以得出一个结论：**

每个组织的 msp 目录，需要发给整个网络的技术运维人员，技术人员用来在生成通道的时候做背书用。



---



**每个orderer的目录**

查看orderer0目录：

```
# cd orderer0.my118.com/
# tree
.
├── msp
│   ├── admincerts
│   │   └── Admin@my118.com-cert.pem
│   ├── cacerts
│   │   └── ca.my118.com-cert.pem
│   ├── keystore
│   │   └── ec64443141408b15f94ab8ea364ce2669d442d33208f14dae7a7c0dab5dbd49d_sk
│   ├── signcerts
│   │   └── orderer0.my118.com-cert.pem
│   └── tlscacerts
│       └── tlsca.my118.com-cert.pem
└── tls
    ├── ca.crt
    ├── server.crt
    └── server.key
```

比对后发现 ：

**msp目录下**

cacerts  tlscacerts admincerts这三个目录就是父目录下msp目录的内容

keystore目录是orderer0的私钥；

signcerts/orderer0.my118.com-cert.pem 是orderer CA签署的orderer0的证书。

tls 目录下

ca.crt是 orderer的tls CA；

server.key是orderer0的tls 私钥；

server.crt是orderer0的tlsCA签署的orderer0的 tls 证书



---

下面再看看peer节点，选择进入ccic,

```
# cd crypto-config/peerOrganizations/ccic.com

# tree
.
├── ca
│   ├── 7c77020ac73d6089e94a7b86e0fc3e786284f88069b5fa074a69b2751cc68131_sk
│   └── ca.ccic.com-cert.pem
├── msp
│   ├── admincerts
│   │   └── Admin@ccic.com-cert.pem
│   ├── cacerts
│   │   └── ca.ccic.com-cert.pem
│   ├── config.yaml
│   └── tlscacerts
│       └── tlsca.ccic.com-cert.pem
├── peers
│   ├── peer0.ccic.com
│   │   ├── msp
│   │   │   ├── admincerts
│   │   │   │   └── Admin@ccic.com-cert.pem
│   │   │   ├── cacerts
│   │   │   │   └── ca.ccic.com-cert.pem
│   │   │   ├── config.yaml
│   │   │   ├── keystore
│   │   │   │   └── 0d5ba722de757d9e8f818d9522139f0d1a185c70b720235f5e9ffea8d7ee7f96_sk
│   │   │   ├── signcerts
│   │   │   │   └── peer0.ccic.com-cert.pem
│   │   │   └── tlscacerts
│   │   │       └── tlsca.ccic.com-cert.pem
│   │   └── tls
│   │       ├── ca.crt
│   │       ├── server.crt
│   │       └── server.key
│   └── peer1.ccic.com
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@ccic.com-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.ccic.com-cert.pem
│       │   ├── config.yaml
│       │   ├── keystore
│       │   │   └── f47185eb28389df59137475196b000518af58849a9bd914b863ee21a7721d594_sk
│       │   ├── signcerts
│       │   │   └── peer1.ccic.com-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.ccic.com-cert.pem
│       └── tls
│           ├── ca.crt
│           ├── server.crt
│           └── server.key
├── tlsca
│   ├── ad5a10d6fbee212b2388804e995fa412f4e79c4d6fe210c43a0833db55f8374a_sk
│   └── tlsca.ccic.com-cert.pem
└── users
    ├── Admin@ccic.com
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@ccic.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.ccic.com-cert.pem
    │   │   ├── keystore
    │   │   │   └── 3d7688039db187b9f332e6fa77b5a6ead75ff2d4324de2023331c5f39d425ff9_sk
    │   │   ├── signcerts
    │   │   │   └── Admin@ccic.com-cert.pem
    │   │   └── tlscacerts
    │   │       └── tlsca.ccic.com-cert.pem
    │   └── tls
    │       ├── ca.crt
    │       ├── client.crt
    │       └── client.key
    ├── User1@ccic.com
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── User1@ccic.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.ccic.com-cert.pem
    │   │   ├── keystore
    │   │   │   └── 5757fb7fce606a34111442ca18afe657c353699c643e54f68127e86f4f894bcb_sk
    │   │   ├── signcerts
    │   │   │   └── User1@ccic.com-cert.pem
    │   │   └── tlscacerts
    │   │       └── tlsca.ccic.com-cert.pem
    │   └── tls
    │       ├── ca.crt
    │       ├── client.crt
    │       └── client.key
    └── User2@ccic.com
        ├── msp
        │   ├── admincerts
        │   │   └── User2@ccic.com-cert.pem
        │   ├── cacerts
        │   │   └── ca.ccic.com-cert.pem
        │   ├── keystore
        │   │   └── bd5079dde34fdc0d679cc2baf409a0f6398587ecbbeb5e856a80b15019ada0eb_sk
        │   ├── signcerts
        │   │   └── User2@ccic.com-cert.pem
        │   └── tlscacerts
        │       └── tlsca.ccic.com-cert.pem
        └── tls
            ├── ca.crt
            ├── client.crt
            └── client.key


```

ca、tlsca目录和orderer一样，都是自签名的CA证书

users/Admin User1 User2目录里面内容都跟orderer目录下一致

msp 目录比orderer下的msp多了一个config.yaml，内容如下：

```
NodeOUs:
  Enable: true
  ClientOUIdentifier:
    Certificate: cacerts/ca.ccic.com-cert.pem
    OrganizationalUnitIdentifier: client
  PeerOUIdentifier:
    Certificate: cacerts/ca.ccic.com-cert.pem
    OrganizationalUnitIdentifier: peer
```



----

正式生产环境联盟创建的步骤：

1. 每个组织生成自己的CA证书、tlsCA证书、admin证书，发给中心技术人员；
2. 中心技术人员把每个组织的证书名称放到单独的msp目录；
3. 中心技术人员创建 configtx.yaml，保证每个组织的msp目录设置正确；
4. 中心技术人员生成网络创世块及通道创世块，稍后发送给每个组织
5. 中心技术人员创建中心节点的网络环境：	

   - 创建orderer节点	

   - 创建kafka节点	

   - 创建zookeeper节点
6. 中心技术人员把orderer节点的域名、ip、端口号等配置信息及创世块发给每个组织
7. 每个组织创建自己的peer节点，并把orderer节点的域名映射加入peer节点中
8. 组织peer节点启动后，使用网路创世块及通道创世块来加入网络和通道



----

联盟成员组织生成自己证书的过程

1. 已有现成 CA 的，可以是已有的 CA；没有的，需要生成新的 CA，包括：私钥、自签名证书（公钥）；私钥要单独安全的保存；
2. 生成自有tlsCA，包括：私钥、自签名证书（公钥）；私钥要单独安全的保存；
3. 生成管理员私钥，然后使用 CA 签署成管理员证书；私钥要单独安全的保存；
4. 把以上三个已经签名的证书作为组织的msp目录：cacerts、 tlscacerts 、admincerts目录，后缀名为.pem；这个msp就可以发给联盟中心技术人员。
5. 为所有将要新建的 peer 节点新建证书；为peer节点生成私钥，用ca证书签署后放入peer节点的msp/signcerts目录；
6. 为每个peer节点生成tls私钥，用tlsca签署后放入peer节点的tls目录，server.crt是签署后的证书 ,server.key是私钥， ca.crt是组织的ca公钥。
7. 每个peer节点新建config.yaml文件，在里面制定 ClientOUIdentifier PeerOUIdentifier
8. 为所有要使用区块链的用户席间证书；每个用户生成自己的私钥，用组织ca签署后放入用户的msp/signcerts目录
9. 为每个用户生成tls私钥，用tlsca签署后放入用户的tls目录，client.crt是签署后的证书 ,client.key是私钥， ca.crt是组织的ca公钥。

----

联盟成员生成证书的方法：

### 1. 生成椭圆加密根证书步骤

- 生成私钥

```bash
openssl ecparam -name prime256v1 -genkey -out ca-key_.pem
```

- 私钥转换成pk8格式

```bash
openssl pkcs8 -inform PEM -outform PEM -topk8 -nocrypt -in ca-key_.pem -out ca-key.pem
```

- 创建生成证书请求

```bash
openssl req -config openssl.cnf -new -key ca-key.pem -out cert-req.csr
```

- 生成自签名证书

```objecbashtivec
openssl x509 -req   -extfile openssl.cnf -extensions usr_cert  -in cert-req.csr -out ca-cert.pem -signkey ca-key.pem  -CAcreateserial -days 3650
```

### 2. 查看

- 查看cert-req.csr

```bash
openssl req -in cert-req.csr -text
```

- 查看ca-cert.pem

```bash
openssl x509 -in ca-cert.pem -text -noout
```

### 3. 加载ca-cert.pem证书,开启ca服务.

- 导入ca-cert.pem证书和ca-key.pem秘钥

```bash
mv ca-cert.pem ca-key.pem $GOPATH/src/github.com/hyperledger/fabric-ca/docker/server/fabric-ca-server
```

- 开启ca服务

```bash
cd $GOPATH/src/github.com/hyperledger/fabric-ca/docker/server/
docker-compose up
```

- 客户端申请证书

```shell
fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
```

openssl.conf

```
[ req ]
default_md              = sha256
default_keyfile         = ca-key.pem
distinguished_name      = req_distinguished_name
prompt = no
x509_extensions = v3_ca
req_extensions  = v3_req
x509_extensions = usr_cert
[ req_distinguished_name ]
C=US
ST=North Carolina
L=Raleigh
O=Example
CN=Example.com
[ usr_cert ]
basicConstraints=critical,CA:true,pathlen:1
keyUsage=critical, keyCertSign,cRLSign
subjectKeyIdentifier=hash
[ v3_req ]
basicConstraints=critical,CA:true,pathlen:1
keyUsage=critical, keyCertSign,cRLSign
subjectKeyIdentifier=hash
```

