# fabric chaincode 开发环境配置文档

 

 

 

 

 

 

**版本信息**

| **版本** | **修改内容** | **修改日期** | **备注** |
| -------- | ------------ | ------------ | -------- |
| 1.0      | 基本框架     | 2019-10-28   | 王红星   |

 

 

 

 

 

# 一、目标

方便chaincode开发人员快速搭建开发环境，敏捷开发、测试chaincode。

# 二、初始来源

下载fabric-sample工程，把chaincode-docker-dev 改为docker-dev。

修改docker-dev目录下的docker-compose-simple.yaml文件，为docker-compose.yaml，把chaincode容器中的

```shell
CORE_PEER_ADDRESS=peer:7051
```

改为

```shell
CORE_PEER_ADDRESS=peer:7052
```

把peer容器的ports中添加7052。

 然后在chaincode目录新建tradetrain目录，新建tradetrain.go文件，把我们现成的chaincode复制进去。

删除除了chaincode 和 docker-dev目录以外的其它文件。

修改项目目录为fabric-chaincode，把本目录作为新的git仓库提交。

这个项目框架基本就完成了。

# 三、增加chaincode执行脚本

创建run_cc.sh,内容如下：

```shell
# !/bin/bash

: ${CC_VERSION:="1.0"}
CC_NAME="tradetrain"
CHANNEL_NAME="myc"
cd ${CC_NAME}
go build 
set -x

CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=${CC_NAME}:${CC_VERSION} ./${CC_NAME}

```

然后修改docker-compose.yaml

在chaincode容器中添加volumes：

```
- ./run_cc.sh:/bin/run_cc.sh
```

# 四、增加cli中安装chaincode的脚本

创建install_cc.sh,这个脚本用来安装、实例化chaincode并执行查询和调用操作，内容如下：

```shell
#!/bin/bash
CC_VERSION=$1
: ${CC_VERSION:="1.0"}
CC_NAME="tradetrain"
CHANNEL_NAME="myc"
SRC_PATH="/opt/gopath/src"
CC_PATH="chaincodedev/chaincode/tradetrain"

set -x
peer chaincode install -p ${CC_PATH} -n ${CC_NAME} -v ${CC_VERSION}
sleep 5
peer chaincode instantiate -n ${CC_NAME} -v ${CC_VERSION} -c '{"Args":[ "init" , "a", "100" , "b" , "200" ]}' -C ${CHANNEL_NAME}
```

# 五、增加打包chaincode的脚本

创建package_cc.sh,这个脚本用来安装、实例化chaincode并执行查询和调用操作，内容如下：

```shell
#!/bin/bash

CC_VERSION=$1
: ${CC_VERSION:="1.0"}
SRC_PATH="/opt/gopath/src"
CC_PATH="chaincodedev/chaincode/tradetrain"
CC_NAME="tradetrain"
CHANNEL_NAME="myc"

peer chaincode package -n ${CC_NAME}  -v ${CC_VERSION} -p ${CC_PATH} ${SRC_PATH}/${CC_PATH}/${CC_NAME}-${CC_VERSION}.out
```

 

# 六、最终成型的ChainCode开发环境

├── README.md //说明文档

├── chaincode

│   └── tradetrain

│       ├── tradetrain //我们自己chaincode的二进制文件

│       └── tradetrain.go  //我们自己chaincode的源代码

└── docker-devmode

​    ├── chaincode  

​    ├── docker-compose.yaml  //开发环境的容器配置文件

​    ├── install_cc.sh  //安装chaincode的脚本

​    ├── msp    //证书目录

​    ├── myc.tx //通道配置

​    ├── orderer.block  

​    ├── run_cc.sh  //执行chaincode的脚本

​    ├── package_cc.sh  //打包chaincode的脚本

​    └── script.sh  //cli每次启动后初始化网络和channel的脚本

# 七、调试开发chaincode的步骤

每次要调试chaincode的时候，需要按照如下步骤进行

## （一）启动开发模式的区块链容器

打开第一个终端窗口，进入docker-dev目录，执行：

```shell
docker-compose up
```

## （二）安装并实例化chaincode

打开第二个终端窗口，进入docker-dev目录，执行如下代码进入chaincode容器

```shell
docker-compose exec cli bash
```

 然后执行: 

```shell
./install_cc.sh
```

或者手动执行：

```shell
peer chaincode install -p chaincodedev/chaincode/tradetrain -n tradetrain -v 1.0

peer chaincode instantiate -n tradetrain -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -C myc
```

如果当前容器没有执行过清除命令（docker-compose down）的话，下次调试/测试不需要执行安装实例化。

注意：接下来我们需要选择到底是测试chaincode还是调试；如果测试，请执行第三步《运行chaincode》；如果要调试，请执行第四部《调试chaincode》。

## （三）运行chaincode

打开第三个终端窗口，进入docker-dev目录，执行如下代码进入chaincode容器

```shell
docker-compose exec chaincode bash
```

 然后在chaincode容器中执行：

```shell
/bin/run_cc.sh
```

或者手动执行：

```shell
cd tradetrain
go build 
CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=tradetrain:0 ./tradetrain
```

## （四）调试chaincode

首先确保你的电脑安装了最新的go环境，并确保你的操作系统的环境变量有正确的GOROOT GOPATH。

使用vscode打开项目，确保项目目录下创建了.vscode目录，然后在.vscode 目录下创建launch.json

```yaml
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch file",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "program": "${file}",
      "env": {
        "CORE_PEER_ADDRESS": "localhost:7052" ,
        "CORE_CHAINCODE_ID_NAME": "tradetrain:1.0",
      }
    }
  ]
}   
```

然后在vscode中打开chaincode文件（tradetrain.go），打开左方的调试选项卡，然后点击调试工具栏中的“开始调试”按钮就可以调试chaincode。

修改完chaincode后，需要停止并重新启动调试。

## （五）调用chaincode

打开第二个终端窗口，可以执行如下命令来查询、调用chaincode来测试：

```shell
peer chaincode invoke -n tradetrain -c '{"Args":["save", "a", "20"]}' -C myc

peer chaincode query -n tradetrain -c '{"Args":["load","a"]}' -C myc
```

## （六）打包chaincode

打开第二个终端窗口，进入docker-dev目录，执行如下代码进入chaincode容器

```shell
docker-compose exec chaincode bash
```

然后在chaincode容器中执行：

```shell
/bin/pacage_cc.sh
```

或者手动执行：

```shell
peer chaincode package -n tradetrain -p chaincodedev/chaincode/tradetrain /opt/gopath/src/chaincodedev/chaincode/tradetrain/tradetrain-1.0.out -v 1.0
```

 

##  （七）发布chaincode

把上一步打包好的chaincode 文件发送给各组织的技术运维人员并协调安装升级。