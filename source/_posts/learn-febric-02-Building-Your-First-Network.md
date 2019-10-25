---
title: learn febric 02 Building Your First Network
date: 2018-12-01 19:56:03
tags:
  - fabric
  - blockchain
---
参照官网：
https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html

 We provide a fully annotated script - byfn.sh - that leverages these Docker images to quickly bootstrap a Hyperledger Fabric network comprised of 4 peers representing two different organizations, and an orderer node. It will also launch a container to run a scripted execution that will join peers to a channel, deploy and instantiate chaincode and drive execution of transactions against the deployed chaincode.

就是说这个脚本会启动四个 peer，代表两个组织、一个 orderer 节点。它会启动一个容器，容器会执行一个脚本：把 peer 连接到 channel， 部署并实例化链码，然后针对部署的链码发起交易的执行。

If you choose not to supply a channel name, then the script will use a default name of mychannel. The CLI timeout parameter (specified with the -t flag) is an optional value; if you choose not to set it, then the CLI will give up on query requests made after the default setting of 10 seconds.

不提供 -c channel 名字，缺省创建mychannel

- Generate Network Artifacts 创建第一个网络

执行
```
./byfn.sh generate
```
瞬间执行了好多东西，不知道干啥了

This first step generates all of the certificates and keys for our various network entities, the genesis block used to bootstrap the ordering service, and a collection of configuration transactions required to configure a Channel.
应该是创建了：所有网络实体的证书、密钥，启动ordering 服务的创世块，以及一系列配置Channel所需要的交易配置信息？？

- Bring Up the Network
把这个网络跑起来
Next, you can bring the network up with one of the following commands:
```
./byfn.sh up
```
这个命令会编译Golang链码镜像，然后运行相应的容器。

The logs will continue from there. This will launch all of the containers, and then drive a complete end-to-end application scenario. 



注意：
只能同时启动一个，除非你把当前的删掉并重新创建网络
Do not run both of these commands. Only one language can be tried unless you bring down and recreate the network between.

The above command will compile Golang chaincode images and spin up the corresponding containers. Go is the default chaincode language, however there is also support for Node.js and Java chaincode. If you’d like to run through this tutorial with node chaincode, pass the following command instead:
Go是缺省链码开发语言，其实还支持nodejs java, 如果愿意可以加上 -l node 执行nodejs的链码镜像
```
# we use the -l flag to specify the chaincode language
# forgoing the -l flag will default to Golang
./byfn.sh up -l java

./byfn.sh up -l node
```


跑完后有这么多docker 
```
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
0ce3814812fd        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   6 minutes ago       Up 7 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
3d53d2a05872        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   7 minutes ago       Up 7 minutes                                                           dev-peer0.org1.example.com-mycc-1.0
72a9c0727267        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   8 minutes ago       Up 8 minutes                                                           dev-peer0.org2.example.com-mycc-1.0
b045f65f36f4        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              9 minutes ago       Up 9 minutes                                                           cli
84f8c3649564        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
ddd5b3942165        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
81b3285302bf        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
86ebd0fc4588        hyperledger/fabric-orderer:latest                                                                      "orderer"                9 minutes ago       Up 9 minutes        0.0.0.0:7050->7050/tcp                             orderer.example.com
8b48b3ab2880        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
```


- 停止（或者删除）网络 Bring Down the Network
Finally, let’s bring it all down so we can explore the network setup one step at a time. The following will kill your containers, remove the crypto material and four artifacts, and delete the chaincode images from your Docker Registry:
```
./byfn.sh down
```
执行了，啥都删了，那些容器都没有了

If you’d like to learn more about the underlying tooling and bootstrap mechanics, continue reading. In these next sections we’ll walk through the various steps and requirements to build a fully-functional Hyperledger Fabric network.

继续阅读，可以学习更多的底层工具、启动引导技术。下面我们走一遍各种方法、需求来创建一个全功能的超级账本网络

The manual steps outlined below assume that the FABRIC_LOGGING_SPEC in the cli container is set to DEBUG. You can set this by modifying the docker-compose-cli.yaml file in the first-network directory. e.g.
后面要把cli 容器设置成 Debug 模式
```
cli:
  container_name: cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - FABRIC_LOGGING_SPEC=DEBUG
    #- FABRIC_LOGGING_SPEC=INFO
```
我没有找到 FABRIC_LOGGING_SPEC ，看到只有
```
      #- CORE_LOGGING_LEVEL=DEBUG
      - CORE_LOGGING_LEVEL=INFO
```
但我仍然添加了 FABRIC_LOGGING_SPEC=DEBUG

## 加密生成器 Crypto Generator
We will use the cryptogen tool to generate the cryptographic material (x509 certs and signing keys) for our various network entities. These certificates are representative of identities, and they allow for sign/verify authentication to take place as our entities communicate and transact.
我们用它生成各种网络实体需要的加密素材（就是证书、密钥）。这些证书是身份的代表：他们用来在实体间通讯和交易的时候做加密/认证。

### 加密是怎么工作的 How does it work?
Cryptogen consumes a file - crypto-config.yaml - that contains the network topology and allows us to generate a set of certificates and keys for both the Organizations and the components that belong to those Organizations. Each Organization is provisioned a unique root certificate (ca-cert) that binds specific components (peers and orderers) to that Org. By assigning each Organization a unique CA certificate, we are mimicking a typical network where a participating Member would use its own Certificate Authority. Transactions and communications within Hyperledger Fabric are signed by an entity’s private key (keystore), and then verified by means of a public key (signcerts).
cryptogen 使用 crypto-config.yml 做配置文件，配置文件中包含了网络拓扑，让我们能为两个组织和他们的组织成员生成一系列证书和密钥。
每个组织有一个自己的唯一根证书，根证书绑定组织成员（peers orderers）。
由于每个组织一个唯一的CA证书，我们就模拟了一个典型的网络，参与的成员必须使用他们自己组织的CA （证书授权）。
超级账本上的交易和通讯采用实体的私钥来签名，并用他自己的公钥校验。

orderer 名字从配置文件里根据 Specs/Hostname 及 Domain 来确定
The naming convention for a network entity is as follows - “{ {.Hostname} }.{ {.Domain} }”. So using our ordering node as a reference point, we are left with an ordering node named - orderer.example.com that is tied to an MSP ID of Orderer. This file contains extensive documentation on the definitions and syntax. You can also refer to the Membership Service Providers (MSP) documentation for a deeper dive on MSP.

After we run the cryptogen tool, the generated certificates and keys will be saved to a folder titled crypto-config.

##  交易配置生成器 Configuration Transaction Generator

configtxgen 用来创建四个配置
The configtxgen tool is used to create four configuration artifacts:

- orderer 【genesis block】 创世块
- channel 【configuration transaction channel】配置交易（还是交易配置）
- and two 【anchor peer transactions】 - one for each Peer Org. 这个看不懂

没看懂后面两个
The orderer block is the Genesis Block for the ordering service, and the channel configuration transaction file is broadcast to the orderer at Channel creation time. The anchor peer transactions, as the name might suggest, specify each Org’s Anchor Peer on this channel.

### 交易配置生成器是怎么工作的 How does it work?

这玩意使用 configtx.yaml 作为配置文件，定义了网络中的成员定义。
包括三个成员：一个 Orderer 组织，两个 Peer 组织。
每个管理并维护连个 peer 节点。
这个文件还定义了一个 "联盟" SampleConsortium, 包括两个 Peer 组织
Configtxgen consumes a file - configtx.yaml - that contains the definitions for the sample network. There are three members - one Orderer Org (OrdererOrg) and two Peer Orgs (Org1 & Org2) each managing and maintaining two peer nodes. This file also specifies a consortium - SampleConsortium - consisting of our two Peer Orgs. Pay specific attention to the “Profiles” section at the top of this file. You will notice that we have two unique headers. One for the orderer genesis block - TwoOrgsOrdererGenesis - and one for our channel - TwoOrgsChannel.

These headers are important, as we will pass them in as arguments when we create our artifacts.

*Note*
sample联盟是在系统级定义的，然后再 channel 级别引用。 Channels 是联盟的一个概念，所有联盟必须在广义网络范围内定义。
这里还有连个值得注意的规范，第一就是为每个组织定义了 anchor peers；第二个就是制定了每个组织的 MSP directory，就是为每个组织 *在创世块中* 保存根证书。这样，每个网络实体跟 ordering service 通讯都可以用它的数字签名校验了。

Notice that our SampleConsortium is defined in the system-level profile and then referenced by our channel-level profile. Channels exist within the purview of a consortium, and all consortia must be defined in the scope of the network at large.
This file also contains two additional specifications that are worth noting. Firstly, we specify the anchor peers for each Peer Org (peer0.org1.example.com & peer0.org2.example.com). Secondly, we point to the location of the MSP directory for each member, in turn allowing us to store the root certificates for each Org in the orderer genesis block. This is a critical concept. Now any network entity communicating with the ordering service can have its digital signature verified.

## 运行工具 Run the tools

既可以手动执行来生成证书和各种配置。当然也可以是这使用 byfn.sh 完成这些任务
You can manually generate the certificates/keys and the various configuration artifacts using the configtxgen and cryptogen commands. Alternately, you could try to adapt the byfn.sh script to accomplish your objectives.

## 手动创建 Manually generate the artifacts

其实可以参考 byfn.sh 脚本中的 generateCerts 来生成证书，方便期间还是在这里做个说明
```
../bin/cryptogen generate --config=./crypto-config.yaml
```
结果就是
```
org1.example.com
org2.example.com
```
证书就生成到 crypto-config 目录，在 first-network/crypto-config 
生成的目录结构基本和配置文件的目录结构差不多。


下面，我们告诉 configtxgen 到哪里去找 configtx.yaml
```
export FABRIC_CFG_PATH=$PWD
```
然后，执行  configtxgen 来创建 orderer 创世块:
```
configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
```
输出
```
2018-12-01 22:29:19.647 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-01 22:29:19.683 CST [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2018-12-01 22:29:19.685 CST [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block

#看看结果
ll channel-artifacts
-rw-r--r--   1 whx  staff  12770 12  1 22:29 genesis.block
```

## 创建 一个 Channel Configuration Transaction

下面创建 channel transaction artifact. 
确保替换 $CHANNEL_NAME 或者设置 CHANNEL_NAME 为环境变量:

# The channel.tx artifact contains the definitions for our sample channel
```
export CHANNEL_NAME=mychannel  
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

```

输出
```
2018-12-01 22:34:48.777 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-01 22:34:48.808 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-12-01 22:34:48.812 CST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

#看下结果
ll channel-artifacts
-rw-r--r--   1 whx  staff    346 12  1 22:34 channel.tx
-rw-r--r--   1 whx  staff  12770 12  1 22:29 genesis.block
```

下面，我们定义 网络中组织1的 anchor peer 
```
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

2018-12-01 22:39:15.789 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-01 22:39:15.831 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-12-01 22:39:15.831 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

```

紧接着就是组织2

```
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP


2018-12-01 22:39:24.500 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-12-01 22:39:24.535 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2018-12-01 22:39:24.535 CST [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

看看生成的内容
```
ll channel-artifacts
-rw-r--r--   1 whx  staff    284 12  1 22:39 Org1MSPanchors.tx
-rw-r--r--   1 whx  staff    284 12  1 22:39 Org2MSPanchors.tx
-rw-r--r--   1 whx  staff    346 12  1 22:34 channel.tx
-rw-r--r--   1 whx  staff  12770 12  1 22:29 genesis.block
```

## 启动网络 Start the network

We will leverage a script to spin up our network. The docker-compose file references the images that we have previously downloaded, and bootstraps the orderer with our previously generated genesis.block.

为了剖析每一步的功能，我们一步步通过命令行手动执行。
受限我们启动网络
```
docker-compose -f docker-compose-cli.yaml up -d

Creating network "net_byfn" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver
Creating orderer.example.com    ... done
Creating peer1.org1.example.com ... done
Creating peer0.org2.example.com ... done
Creating peer0.org1.example.com ... done
Creating peer1.org2.example.com ... done
Creating cli                    ... done
```
起了 orderer 和 四个 peers
如果想看看网络实时日志，就不要加 -d 参数。If you let the logs stream, 就打开第二个 terminal 执行 CLI calls .

## 环境变量 Environment variables
下面的 CLI 命令是针对 peer0.org1.example.com 的, 我们需要设定如下四个环境变量用来执行我们的命令。
这些变量会带入 CLI 容器, 因为我们操作的时候不用管就行。
但是如果想要对不同的 peers 执行命令，就需要修改这四个环境变量
```
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

## 创建、加入 Channel Create & Join Channel
回顾前面使用 configtxgen 创建 channel configuration transaction 。
通过使用 configtx.xml 中相同或者不同的 profiles ，我们可以重复创建更多的 channel configuration transactions。
然后就可以重复上面介绍的方法在网络中创建其它的channel。
我们将使用 docker exec 命令进入 CLI 容器
```
docker exec -it cli bash

root@6248103d0d59:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

下一步，我们想要把我们前面创建的 channel configuration transaction artifact 发送给 orderer ，作为创建 channel 的请求。
我们用 -c 来设置 channel name
-f 设置  channel configuration transaction 这里是 channel.tx
你可以用你自己的名字
 with the -f flag. In this case it is channel.tx, however you can mount your own configuration transaction with a different name. Once again we will set the CHANNEL_NAME environment variable within our CLI container so that we don’t have to explicitly pass this argument. Channel names must be all lower case, less than 250 characters long and match the regular expression [a-z][a-z0-9.-]*.
```
export CHANNEL_NAME=mychannel

# the channel.tx file is mounted in the channel-artifacts directory within your CLI container
# as a result, we pass the full path for the file
# we also pass the path for the orderer ca-cert in order to verify the TLS handshake
# be sure to export or replace the $CHANNEL_NAME variable appropriately

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-12-02 00:44:23.818 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-12-02 00:44:23.887 UTC [cli/common] readBlock -> INFO 002 Received block: 0

ll
drwxr-xr-x  7 root root   224 Dec  1 14:39 channel-artifacts/
drwxr-xr-x  4 root root   128 Dec  1 14:24 crypto/
-rw-r--r--  1 root root 15778 Dec  2 00:44 mychannel.block
drwxr-xr-x 10 root root   320 Nov 30 06:27 scripts/

```
这时候当前目录多了一个文件 mychannel.block
这就是一个创世块，用来连接 channel 的。

接下来我们把 peer0.org1.example.com 接入到 channel.
```
# By default, this joins ``peer0.org1.example.com`` only
# the <channel-ID.block> was returned by the previous command
# if you have not modified the channel name, you will join with mychannel.block
# if you have created a different channel name, then pass in the appropriately named block

 peer channel join -b mychannel.block

 2018-12-02 00:50:39.672 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-12-02 00:50:39.748 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel

```


修改上面的环境变量，可以把其它几个peer 连接到 channel 上。

我们暂时只把 peer0.org2.example.com 接入 channel ，这样我们就可以更新channel 的  anchor peer definitions .
命令如下:

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
#其实就是在一行中把环境变量直接设置好，带入peer 命令中，分开就是

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp 

export CORE_PEER_ADDRESS=peer0.org2.example.com:7051 

export CORE_PEER_LOCALMSPID="Org2MSP" 

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 

peer channel join -b mychannel.block

# 结果
2018-12-02 01:00:42.483 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-12-02 01:00:42.549 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel

```
第二个就不用再创建channel了，因为第一个已经创建了，直接使用第一次返回的 mychannel.block 来加入就行了

## 更新 anchor peers

anchor peers 是啥意思？没太搞明白 
channel The following commands are channel updates and they will propagate to the definition of the channel. In essence, we adding additional configuration information on top of the channel’s genesis block. Note that we are not modifying the genesis block, but simply adding deltas into the chain that will define the anchor peers.

更新 channel definition ，定义 Org1 的 anchor peer 为 peer0.org1.example.com:
```
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-12-02 02:32:20.264 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-12-02 02:32:20.298 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
```
然后更新 Org2 的 anchor peer for 为 peer0.org2.example.com. 这次得带着环境变量
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2018-12-02 02:33:43.892 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-12-02 02:33:43.925 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
```


## 安装并实例化 Chaincode  Install & Instantiate Chaincode

应用程序通过 Chaincode 跟账本的区块链交互，因此我们必须在每个 peer 上安装 Chaincode，Chaincode 会执行并且为我们的交易背书，然后在 channel上实例化 chaincode
Applications interact with the blockchain ledger through chaincode. As such we need to install the chaincode on every peer that will execute and endorse our transactions, and then instantiate the chaincode on the channel.

我们首先在四个 peer 节点中的一个安装  Go, Node.js or Java chaincode 例子。
这些命令把特殊的源代码添加到 peer 的文件系统中。
*注意*

一个Chinacode 名字只能安装一个版本。peer 文件系统中以 chinacode 名字和版本的方式保存源代码; 
它是语言无关的。实例化的 Chaincode 容器会反应那个语言安装在 peer 中 。

Golang
```
# this installs the Go chaincode. For go chaincode -p takes the relative path from $GOPATH/src
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
2018-12-02 02:46:15.195 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-12-02 02:46:15.196 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-12-02 02:46:15.527 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
```

也需要在 org2 中安装,如果不安装，后面 invoke 提示失败
```
# 环境变量
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

2018-12-02 04:32:29.371 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-12-02 04:32:29.371 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-12-02 04:32:29.767 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >

```

Nodejs
```
# this installs the Node.js chaincode
# make note of the -l flag to indicate "node" chaincode
# for node chaincode -p takes the absolute path to the node.js chaincode
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/

2018-12-02 02:46:39.551 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-12-02 02:46:39.551 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-12-02 02:46:39.571 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:500 message:"error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exists)" > payload:"\n b\300\260V'\311\254\013\376\213L\261\235\207\267ia=\340\020\242kh\266\327D\034\354\230\362\014p\022y\032o\010\364\003\022jerror installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exists)\"\006\022\004lscc"
```
因为已经安装了 go 语言版本，所以安装nodejs版本的提示出错

Java
```
# make note of the -l flag to indicate "java" chaincode
# for java chaincode -p takes the absolute path to the java chaincode
peer chaincode install -n mycc -v 1.0 -l java -p /opt/gopath/src/github.com/chaincode/chaincode_example02/java/

2018-12-02 02:49:28.758 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-12-02 02:49:28.758 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-12-02 02:49:28.809 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:500 message:"error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exists)" > payload:"\n \026'\027\260\334\016\371\006x`\237\354P\374\337\017$}\320\261\353\335\234}t\317\241\324Q\263Q\300\022y\032o\010\364\003\022jerror installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exists)\"\006\022\004lscc"
```
同样，安装java版本的也出错了

接着，在网络中实例化 chaincode，这将会在网络中初始化 chaincode，设置 chaincode的背书策略，然后为目标 peer 启动一个 chaincode 容器 。
注意 -P 参数，这是我们的策略，它指定这个交易背书的 chaincode 验证策略级别
下面的命令中可以看到我们的策略是 -P "AND ('Org1MSP.peer','Org2MSP.peer')". 
这意味着这个背书需要 Org1 和 Org2 同时确认。如果我们用了“OR”，就只需要其中一个确认就可以。

### Golang
```
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well

peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"


2018-12-02 02:57:35.533 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-12-02 02:57:35.533 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc

```

##Node.js

 注意
实例化Nodejs chaincode 需要点时间，这个命令不是挂起了，而是他要安装 fabric-shim 层，这个image需要编译
```
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well
# notice that we must pass the -l flag after the chaincode name to identify the language

peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

## Java

注意
这个也需要点时间，因为它需要编译 chaincode 并且下载带有 java 环境的 docker 容器
```
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l java -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```
如果你想其它的 peers 跟账本交互，就需要把他们加入到 channel 中，然后在相应的 peer 文件系统中安装同样名字、版本、语言的 chaincode。每个 peer 在于 chaincode 交互前将会启动一个 chaincode 容器。
再一次，我们要知道的事实： nodejs 镜像编译起来较慢
只要 chaincode 在 channel 中实例化了，我们就可以忘记 l flag。我们只需要传入 channel id 和 chaincode 名字

## 查询 Query
我们来查询一下 a 的值，以确认 chaincode 已经正常实例化、状态数据库也 populated(密布、分步、同步)
Let’s query for the value of a to make sure the chaincode was properly instantiated and the state DB was populated. The syntax for query is as follows:

```
# be sure to set the -C and -n flags appropriately
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

100

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","b"]}'

200
```

## Invoke
从 a 转移 10 到 b，交易会产生一个新的 block ，然后更新状态数据库
```
# be sure to set the -C and -n flags appropriately

peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

# 提示出错了
Error: could not assemble transaction: ProposalResponsePayloads do not match - proposal response: version:1 response:<status:200 > payload:"\n \343\340\037+\317\374\235L\312NCb\0066!j\234\334\263\255Q?ivn\271G\374\006`\247A\022Y\nE\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\003\022-\n\004mycc\022%\n\007\n\001a\022\002\010\003\n\007\n\001b\022\002\010\003\032\007\n\001a\032\00290\032\010\n\001b\032\003210\032\003\010\310\001\"\013\022\004mycc\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\252\006-----BEGIN CERTIFICATE-----\nMIICKDCCAc6gAwIBAgIQBJEnZPvyJFY+mpVRXW5JdTAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0xODEyMDExNDE5MDBaFw0yODExMjgxNDE5MDBa\nMGoxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMQ0wCwYDVQQLEwRwZWVyMR8wHQYDVQQDExZwZWVyMC5vcmcx\nLmV4YW1wbGUuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEDvZGZr/QfQz4\nzcevok8uwRC9uKrOBsvPzu6XNKyskxD3r+S0GGw4F/5plNJYH2fwWwdE/Lp630g2\ntu0BSFW6IaNNMEswDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwKwYDVR0j\nBCQwIoAgIidiMO1ijvL6iOrMvOLhe4ymOnTA6HaIplyPBHJTYvwwCgYIKoZIzj0E\nAwIDSAAwRQIhAI8yVRt8eOG2wzKWfNDc7kAIFVFApmDODCa3E20UWL97AiBZ3ddq\ndc64M/vUnn0EvLmHTfTA+ioOLLeeVHu6Vyz+SA==\n-----END CERTIFICATE-----\n" signature:"0E\002!\000\322\372\375I\276\365\257\333\007\240\345(\367\t\264/\304%\344h\347P\021{\334\357N\037\311\363\226z\002 &\346\336]\207J\217\321\024&!\031\337{\023\203P\031\243\256uj\333'\374\366*D\222\341\231\320" >

```
然后检查估计是需要在 Org2 上安装 Chaincode,就安装了一下

重新执行
```
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

2018-12-02 04:35:50.056 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```
然后再次查询
```
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
90
```
这就执行成功了个
然后本着测试的精神继续玩，
```
# 再转 a=>b 2 
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","2"]}'

2018-12-02 04:38:16.449 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200


peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
88

# b=>a 3
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","b","a","3"]}'
2018-12-02 04:40:04.542 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200


peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
91

```
完美

##  这到底都干了些什么 What’s happening behind the scenes?

注意：

These steps describe the scenario in which script.sh is run by ‘./byfn.sh up’. Clean your network with ./byfn.sh down and ensure this command is active. Then use the same docker-compose prompt to launch your network again

- A script - script.sh - is baked inside the CLI container. The script drives the createChannel command against the supplied channel name and uses the channel.tx file for channel configuration.

- The output of createChannel is a genesis block - <your_channel_name>.block - which gets stored on the peers’ file systems and contains the channel configuration specified from channel.tx.

- The joinChannel command is exercised for all four peers, which takes as input the previously generated genesis block. This command instructs the peers to join <your_channel_name> and create a chain starting with <your_channel_name>.block.

- Now we have a channel consisting of four peers, and two organizations. This is our TwoOrgsChannel profile.

- peer0.org1.example.com and peer1.org1.example.com belong to Org1; peer0.org2.example.com and peer1.org2.example.com belong to Org2

- These relationships are defined through the crypto-config.yaml and the MSP path is specified in our docker compose.

- The anchor peers for Org1MSP (peer0.org1.example.com) and Org2MSP (peer0.org2.example.com) are then updated. We do this by passing the Org1MSPanchors.tx and Org2MSPanchors.tx artifacts to the ordering service along with the name of our channel.

- A chaincode - chaincode_example02 - is installed on peer0.org1.example.com and peer0.org2.example.com
- The chaincode is then “instantiated” on peer0.org2.example.com. Instantiation adds the chaincode to the channel, starts the container for the target peer, and initializes the key value pairs associated with the chaincode. The initial values for this example are [“a”,”100” “b”,”200”]. This “instantiation” results in a container by the name of dev-peer0.org2.example.com-mycc-1.0 starting.

- The instantiation also passes in an argument for the endorsement policy. The policy is defined as -P "AND ('Org1MSP.peer','Org2MSP.peer')", meaning that any transaction must be endorsed by a peer tied to Org1 and Org2.

- A query against the value of “a” is issued to peer0.org1.example.com. The chaincode was previously installed on peer0.org1.example.com, so this will start a container for Org1 peer0 by the name of dev-peer0.org1.example.com-mycc-1.0. The result of the query is also returned. No write operations have occurred, so a query against “a” will still return a value of “100”.

- An invoke is sent to peer0.org1.example.com to move “10” from “a” to “b”
The chaincode is then installed on peer1.org2.example.com

- A query is sent to peer1.org2.example.com for the value of “a”. This starts a third chaincode container by the name of dev-peer1.org2.example.com-mycc-1.0. A value of 90 is returned, correctly reflecting the previous transaction during which the value for key “a” was modified by 10.#