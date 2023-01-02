---
title: ipfs-cluster 测试
date: 2020-11-18 11:04:34
tags:
---




```bash
echo "a">test.txt
echo "b">test2.txt
echo "c">test3.txt
```

master:

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 add test3.txt
 added QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 test3.txt
 
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive

# ipfs-cluster-ctl pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 |  | PIN | Repl. Factor: -1 | Allocations: [everywhere] | Recursive | Metadata: no | Exp: ∞

#ipfs pin ls
QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
 
```

**注：QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn 是一个空的文件目录，每个初始化的节点都有。**

在另外两台服务器上都执行 pin ls，

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
# ipfs pin ls
QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
```

说明使用cluster add的时候在所有的节点都已经同步保存了。



whx:

```bash
# ipfs add test.txt
added Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 test.txt

# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive

# ipfs pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive

```

说明普通的add命令只在本机pin

whx：

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin add Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
pinned Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursively
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
```

在另外两台服务器上都执行 pin ls，

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
```

说明使用 ipfs add 以后需要用 cluster 的 pin add 来让 cluster 保存了。

whx:

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm  Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
unpinned Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
```

执行删除操作后，pin ls看不到该文件了。

master:

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin add Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
pinned Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursively

# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
```

在master上执行pin add 后该文件又出现了。（难到这个在存储里还有，重新pin add 后又出现了）





worker:

```bash
# ipfs add test2.txt
added QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM test2.txt
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
# ipfs pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM recursive
QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive

```

whx:

```bash
# ipfs cat QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
b
# ipfs pin ls
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
```

说明可以看不同节点增加的文件，但是并没有pin。

whx:

```bash
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin add QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
pinned QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM recursively
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM recursive
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive

```

 在任意一台服务器上

```

# ipfs --api /ip4/127.0.0.1/tcp/9095 cat QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
b
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
unpinned QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
# ipfs --api /ip4/127.0.0.1/tcp/9095 cat QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
b
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin ls
Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9 recursive
QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7 recursive
# ipfs cat QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
b
```

这时候删除了都能看，然后在一台清除gc

```bash
# ipfs repo gc
removed QmWLX2v5rnFeAbFyP4VPkDKLVVk91LncD6S648Ce4GgGgz
removed QmevuAKVMG3Pr6nTp2oy9MzuPhZuZMxRfFApbfzk5zafr6
removed QmV7XXZAUrBaE76Susmd4gRLzziK5QH1EigHqgma8adKf9
removed QmWgtt1onrFVgqwAuYBWQ2HNkxJtBgnDzqWRVZQzpWeDUj
removed QmWNGqp2zsw6nLr6aySK3UAQkTkoYozMNszwcz1bWmuckX
removed QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
removed QmT9PmBnECcLRCu4hYmbijCca6k7ocdagGqUFpH9gWf5Cn
removed QmaoaDnyHfZYEXjCuX59bgzA2YbgXMUKuSARdS3GHgbxoN
# ipfs cat QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
b

```

一台上清除gc后依然能看，下面在所有服务器上清除

```bash
# ipfs repo gc
removed QmWLX2v5rnFeAbFyP4VPkDKLVVk91LncD6S648Ce4GgGgz
removed QmevuAKVMG3Pr6nTp2oy9MzuPhZuZMxRfFApbfzk5zafr6
removed QmV7XXZAUrBaE76Susmd4gRLzziK5QH1EigHqgma8adKf9
removed QmWgtt1onrFVgqwAuYBWQ2HNkxJtBgnDzqWRVZQzpWeDUj
removed QmWNGqp2zsw6nLr6aySK3UAQkTkoYozMNszwcz1bWmuckX
removed QmPphhXAJkNKktecNqsbRf3zXXdsSA4fM9DS4LMaWFmbti
removed QmR9pC5uCF3UExca8RSrCVL8eKv7nHMpATzbEQkAHpXmVM
removed QmT9PmBnECcLRCu4hYmbijCca6k7ocdagGqUFpH9gWf5Cn
removed QmaoaDnyHfZYEXjCuX59bgzA2YbgXMUKuSARdS3GHgbxoN

```

清除完以后就看不到文件了，说明只要gc还没有清除，文件就都还能看。



---

测试  `replication_factor_min` 和 `replication_factor_max`

~/.ipfs-cluster/service.json 文件中 `replication_factor_min` 和 `replication_factor_max` 均为默认值 -1

集群中每个节点都存了一份文件副本。这不是我们想要的，需要继续实验如何控制副本数。



修改 ~/.ipfs-cluster/service.json 文件中的两个配置项： `replication_factor_min` 和 `replication_factor_max`。

- replication_factor_min 代表存储该文件的集群最小节点数，可以理解为副本数下线，-1 代表全部。本次实验中设置为 1.
- replication_factor_max 代表存储该文件的集群最大阶段属，可以理解为副本数上限，-1 代表全部。本次实验中设置为 1.





```
jq ".cluster.replication_factor_min = 2" ${IPFS_CLUSTER_PATH}/service.json |jq ".cluster.replication_factor_max = 3" > ${IPFS_CLUSTER_PATH}/tmp.service.json
mv -f ${IPFS_CLUSTER_PATH}/tmp.service.json ${IPFS_CLUSTER_PATH}/service.json
grep replication_factor ${IPFS_CLUSTER_PATH}/service.json 
systemctl restart ipfs-cluster
systemctl status ipfs-cluster


```

随机生成100M的大文件

```
dd if=/dev/urandom of=test1.file count=100000 bs=1024
dd if=/dev/urandom of=test2.file count=100000 bs=1024
dd if=/dev/urandom of=test3.file count=100000 bs=1024
```

添加文件并查看ipfs存储

```
# ipfs --api /ip4/127.0.0.1/tcp/9095 add test1.file
added QmaVdYFJP88SzYtDayfiU4xbzCTmStmB6Ug4ihtJV7Nzgg test1.file
#  du -sh /data/*
101M	/data/ipfs
44K	/data/ipfs-cluster
```

所有三台服务器上存储内容一致.

清理所有存储

```
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm QmVFxmvDaBBGBFUMPLSRL9mb4N6Xm8xEYnCuy5m85gPAaP
unpinned QmVFxmvDaBBGBFUMPLSRL9mb4N6Xm8xEYnCuy5m85gPAaP
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm QmavpLffUr1iY2QfhsbMoHrm9tb8rrjoPGP8erwFW2M3tX
unpinned QmavpLffUr1iY2QfhsbMoHrm9tb8rrjoPGP8erwFW2M3tX
# ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7
unpinned QmetGxZTgo8tYAKQH1KLsY13MxqeVHbxYVmvzBzJAKU6Z7
#  ipfs --api /ip4/127.0.0.1/tcp/9095 pin rm Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
unpinned Qmbvkmk9LFsGneteXk3G7YLqtLVME566ho6ibaQZZVHaC9
# ipfs repo gc
# du -sh /data/*
352K	/data/ipfs
48K	/data/ipfs-cluster

```

重新设定复制因子



```
jq ".cluster.replication_factor_min = 1" ${IPFS_CLUSTER_PATH}/service.json |jq ".cluster.replication_factor_max = 2" > ${IPFS_CLUSTER_PATH}/tmp.service.json
mv -f ${IPFS_CLUSTER_PATH}/tmp.service.json ${IPFS_CLUSTER_PATH}/service.json
grep replication_factor ${IPFS_CLUSTER_PATH}/service.json 
systemctl restart ipfs-cluster
systemctl status ipfs-cluster


```

添加文件并查看ipfs存储

```bash

whx: # ipfs --api /ip4/127.0.0.1/tcp/9095 add test1.file
added QmaVdYFJP88SzYtDayfiU4xbzCTmStmB6Ug4ihtJV7Nzgg test1.file

whx: #  du -sh /data/*
100M	/data/ipfs
68K	/data/ipfs-cluster

master: # du -hs /data/*
101M	/data/ipfs
68K	/data/ipfs-cluster

worker: # du -hs /data/*
972K	/data/ipfs
68K	/data/ipfs-cluster
```

添加第二个文件：

```bash
whx: # ipfs --api /ip4/127.0.0.1/tcp/9095 add test2.file
added QmNzEZywMY6M79wZ6TWuaUU5LFfGDgiqxbrF6mxMdJnjjY test2.file
whx: # du -hs /data/*
200M	/data/ipfs
68K	/data/ipfs-cluster

worker : # du -hs /data/*
101M	/data/ipfs
68K	/data/ipfs-cluster

master: # du -hs /data/*
101M	/data/ipfs
68K	/data/ipfs-cluster
```

结论：每个文件保存了两份

在master上新增文件：

```bash
master: # ipfs --api /ip4/127.0.0.1/tcp/9095 add test3.file

whx: # du -hs /data/*
200M	/data/ipfs
68K	/data/ipfs-cluster

worker: # du -hs /data/*
200M	/data/ipfs
68K	/data/ipfs-cluster

master: # du -hs /data/*
200M	/data/ipfs
68K	/data/ipfs-cluster

```

这时候三台服务器持平

在worker上操作：

```
dd if=/dev/urandom of=test4.file count=100000 bs=1024
dd if=/dev/urandom of=test5.file count=100000 bs=1024
dd if=/dev/urandom of=test6.file count=100000 bs=1024
ipfs --api /ip4/127.0.0.1/tcp/9095 add test4.file
ipfs --api /ip4/127.0.0.1/tcp/9095 add test5.file
ipfs --api /ip4/127.0.0.1/tcp/9095 add test6.file


```

完成后所有服务器存储容量一致：

```
# du -hs /data/*
399M	/data/ipfs
72K	/data/ipfs-cluster
```

继续增加



```bash
dd if=/dev/urandom of=test7.file count=100000 bs=1024
dd if=/dev/urandom of=test8.file count=100000 bs=1024
ipfs --api /ip4/127.0.0.1/tcp/9095 add test7.file
ipfs --api /ip4/127.0.0.1/tcp/9095 add test8.file

whx: # du -hs /data/*
598M	/data/ipfs
76K	/data/ipfs-cluster
worker: # du -hs /data/*
500M	/data/ipfs
76K	/data/ipfs-cluster
master: # du -hs /data/*
499M	/data/ipfs
76K	/data/ipfs-cluster

```

说明cluster尽量在节点之间保持平衡

---

使用ipfs-cluster-ctl添加

```bash
# dd if=/dev/urandom of=test9.file count=100000 bs=1024
# ipfs-cluster-ctl add  --rmin 1 --rmax 1 test9.file
added QmdTZ9vhg9ocqgPE7frdYGAZHFkF8cYM51dLWzxx9y4nSW test9.file
whx: # du -hs /data/*
598M	/data/ipfs
76K	/data/ipfs-cluster
worker: # du -hs /data/*
500M	/data/ipfs
76K	/data/ipfs-cluster
master: # du -hs /data/*
599M	/data/ipfs
76K	/data/ipfs-cluster
```

单独增加一个后，增加到master

继续

```bash
# dd if=/dev/urandom of=test10.file count=100000 bs=1024
# ipfs-cluster-ctl add  --rmin 1 --rmax 1 test10.file
added QmeNha17eq2MfJugthsu9KWdcb18aGKjHbP6LV5rJzXjYo test10.file
whx: # du -hs /data/*
598M	/data/ipfs
76K	/data/ipfs-cluster
worker: # du -hs /data/*
600M	/data/ipfs
76K	/data/ipfs-cluster
master: # du -hs /data/*
599M	/data/ipfs
76K	/data/ipfs-cluster
```

这时候所有服务器存储容量保持平衡。

----

测试别的命令：

```bash
# ipfs add usage-ipfs.txt
added Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx usage-ipfs.txt
# ipfs-cluster-ctl pin add --name whx --replication 1 Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx
Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx | whx:
    > whx                  : PINNED | 2021-05-13T06:19:34.061276362Z
    > 12D3KooWDWcqJD1y6JtSByN1N8zRphLkc7UbFvhd1BeKpG9MxdPe : REMOTE | 2021-05-13T06:19:34.065641318Z
    > 12D3KooWNTfChtCFYH8kHkrMQttJcFTKRXrFbg6yMFN6uGt3fpWV : REMOTE | 2021-05-13T06:19:34.065641318Z
 
 # ipfs-cluster-ctl pin ls Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx
Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx | whx | PIN | Repl. Factor: 1--1 | Allocations: [12D3KooWAdivuLNnoka4H3hNRCjnHxeBNVhUBwQ1YR1HBmUnjLTT] | Recursive | Metadata: no | Exp: ∞

# ipfs-cluster-ctl status Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx
Qmefd5f3y8z7SDyr9SrTeGZvWYueAJaijmmfCBYxFEdwhx | whx:
    > whx                  : PINNED | 2021-05-13T06:27:04.864319281Z
    > 12D3KooWDWcqJD1y6JtSByN1N8zRphLkc7UbFvhd1BeKpG9MxdPe : REMOTE | 2021-05-13T06:27:04.864091368Z
    > 12D3KooWNTfChtCFYH8kHkrMQttJcFTKRXrFbg6yMFN6uGt3fpWV : REMOTE | 2021-05-13T06:27:04.864091368Z
    
    
```



测试curl方式

```
# curl -X POST "http://127.0.0.1:4001/api/v0/pin/ls" |jq
{
  "Keys": {
    "QmNzEZywMY6M79wZ6TWuaUU5LFfGDgiqxbrF6mxMdJnjjY": {
      "Type": "recursive"
    }
 }
 
# curl -X POST "http://127.0.0.1:9095/api/v0/pin/ls" |jq
{
  "Keys": {
    "QmNzEZywMY6M79wZ6TWuaUU5LFfGDgiqxbrF6mxMdJnjjY": {
      "Type": "recursive"
    }
 }
 
curl -X GET  "http://127.0.0.1:9094/id"
curl -X GET  "http://127.0.0.1:9094/version"
curl -X GET  "http://127.0.0.1:9094/peers"
curl -X GET  "http://127.0.0.1:9094/pins"

curl -X POST  "http://127.0.0.1:9094/add"

curl -X GET  "http://127.0.0.1:9094/pins/QmQcCo8MqRjAwUMeZCcALb91aZc9NFCswPwakQbgQb5oH9"
```

add method param

```
replication
name
mode

```



```
ipfs --api /ip4/127.0.0.1/tcp/9095 cat



```



---



As a final tip, this table provides a quick summary of methods available.

| METHOD   | ENDPOINT                          | COMMENT                                                 |
| :------- | :-------------------------------- | :------------------------------------------------------ |
| `GET`    | `/id`                             | Cluster peer information                                |
| `GET`    | `/version`                        | Cluster version                                         |
| `GET`    | `/peers`                          | Cluster peers                                           |
| `DELETE` | `/peers/{peerID}`                 | Remove a peer                                           |
| `POST`   | `/add`                            | Add content to the cluster                              |
| `GET`    | `/allocations`                    | List of pins and their allocations (pinset)             |
| `GET`    | `/allocations/{cid}`              | Show a single pin and its allocations (from the pinset) |
| `GET`    | `/pins`                           | Local status of all tracked CIDs                        |
| `POST`   | `/pins/sync`                      | Sync local status from IPFS                             |
| `GET`    | `/pins/{cid}`                     | Local status of single CID                              |
| `POST`   | `/pins/{cid}`                     | Pin a CID                                               |
| `POST`   | `/pins/{ipfs\|ipns\|ipld}/<path>` | Pin using an IPFS path                                  |
| `DELETE` | `/pins/{cid}`                     | Unpin a CID                                             |
| `DELETE` | `/pins/{ipfs\|ipns\|ipld}/<path>` | Unpin using an IPFS path                                |
| `POST`   | `/pins/{cid}/sync`                | Sync a CID                                              |
| `POST`   | `/pins/{cid}/recover`             | Recover a CID                                           |
| `POST`   | `/pins/recover`                   | Recover all pins in the receiving Cluster peer          |
| `GET`    | `/monitor/metrics`                | Get a list of metric types known to the peer            |
| `GET`    | `/monitor/metrics/{metric}`       | Get a list of current metrics seen by this peer         |
| `GET`    | `/health/alerts`                  | Display a list of alerts (metric expiration events)     |
| `GET`    | `/health/graph`                   | Get connection graph                                    |
| `GET`    | `/health/alerts`                  | Get connection graph                                    |
| `POST`   | `/ipfs/gc`                        | Perform GC in the IPFS nodes                            |