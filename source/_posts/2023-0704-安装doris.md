安装doris

yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64

cd /data/application

```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/doris/1.1/1.1.5-rc02/apache-doris-fe-1.1.5-bin.tar.gz
tar zxf apache-doris-fe-1.1.5-bin.tar.gz

wget https://mirrors.tuna.tsinghua.edu.cn/apache/doris/1.1/1.1.5-rc02/apache-doris-be-1.1.5-bin-x86_64.tar.gz
tar zxf apache-doris-be-1.1.5-bin-x86_64.tar.gz




```



fe

```
cd apache-doris-fe-1.1.5-bin


echo "priority_networks=10.0.0.0/24" >>vi conf/fe.conf
./bin/start_fe.sh --daemon
```



看看是不是启动成功

```
curl http://127.0.0.1:8030/api/bootstrap
{"msg":"success",
"code":0,
"data":{"replayedJournalId":0,"queryPort":0,"rpcPort":0,"version":""},
"count":0}


rpm -ivh https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-client.x86_64

mysql -uroot -P9030 -h127.0.0.1

show frontends\G


```



```
cd /data/application/apache-doris-be-1.1.5-bin-x86_64
echo "priority_networks=10.0.0.0/24" >>vi conf/be.conf
sysctl -w vm.max_map_count=2000000
bin/start_be.sh --daemon


```

吧be加入fe

```

mysql -uroot -P9030 -h127.0.0.1

ALTER SYSTEM ADD BACKEND "10.0.0.11:9050";
```

设置密码

```
SET PASSWORD FOR 'root' = PASSWORD('7Kf8o_Wqid8HVJ6h');
```





```
10000,2017-10-01,北京,20,0,2017-10-01 06:00:00,20,10,10
10000,2017-10-01,北京,20,0,2017-10-01 07:00:00,15,2,2
10001,2017-10-01,北京,30,1,2017-10-01 17:05:45,2,22,22
10002,2017-10-02,上海,20,1,2017-10-02 12:59:12,200,5,5
10003,2017-10-02,广州,32,0,2017-10-02 11:20:00,30,11,11
10004,2017-10-01,深圳,35,0,2017-10-01 10:00:15,100,3,3
10004,2017-10-03,深圳,35,0,2017-10-03 10:20:22,11,6,6
```

