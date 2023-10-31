学习部署 zhuge



vagrant 部署两个centos7

realtime-1 192.168.33.10





修改允许root登录

/etc/ssh/sshd_config

PasswordAuthentication yes

sudo systemctl restart sshd



### 基础

#### 修改环境变量：

```
export WORKSPACE=/data/realtime-dj/SingleVersion
```



#### 修改 global.yml

修改global.yml中GLOBAL.sw变量，即软件仓库的位置为softwares目录即可。

```
 vim global.yml
# 软件仓库地址
sw: /data/realtime-dj/SingleVersion/softwares
  # smc 配置包目录
pkg: /data/single-standard-online
```

#### 安装Ansible

安装pip ，Ansible的安装需要用到pip，该版本的pip在python版本 >= 2.7的环境下皆可离线安装

```bash
yum install -y epel-release
yum install -y rsync python-pip  unzip
```



```
cd $WORKSPACE/softwares/python3
tar -xvf ./pip-18.0.tar.gz
cd pip-18.0
python ./setup.py install


```

安装Ansible:

```
cd $WORKSPACE/softwares/ansible
tar -xvf ./ansible-2.5.0-pip.tar.gz
cd ansible-2.5.0-pip
sh ./install_ansible.sh
```



#### 打通ssh

控制机要自身与自身打通SSH

对于单机版，只需要打通控制机自身即可：

```
cd $WORKSPACE/
ansible-playbook -i ./hosts ./open_ssh.yml -e '@global.yml' --connection=local

```

 

#### 修改 hostname 和 hosts

```
ansible-playbook -i ./hosts ./hosts.yml -e '@global.yml'
```

---

这个在没有网络的时候做



清理旧有的yum源

如果机器没有外网.需要将/etc/yum.repos.d/下的文件转移走，防止yum不断重试无效的yum 源：

```
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak/
##然后无论有网无网，都要执行下清理操作
yum clean all

```

```

```

**这个我后面恢复了。**

---



执行部署

根据tag来一步一步执行部署操作

执行部署时，如果碰到了带有…ignoring字样的错误，那么可以忽略，否则，请及时解决

```
 cd  $WORKSPACE
 
```

#### 基础环境安装 env

env tag会为本次安装创建基础环境**   ((内网环境建议拆分安装))

```
 ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags env
 
```



ansible 部署解析，上面命令具体执行了以下操作，

1./data/tmp_install_dir 创建此临时目录。  

2.selinux 关闭selinux

3.修改时区

4.防火墙关闭

 

### basic_software 

basic_software tag会安装所需的基础软件 (内网环境建议拆分安装)

```

ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags basic_software


```

ansible 部署解析，上面命令具体执行了以下操作，  及对应tags

1.构建本地离线yum源。 

2.安装dnsmasq    

3.安装ntp

4.安装JDK      

 5.安装python3及相关模块 

6.安装Supervisor   

7.安装redis_cli    

8.安装docker



其中安装本地yum源的时候因为版本稍微旧了点，导致安装失败，手动回复yum 源，然后安装了基本软件

```
yum  install -y mariadb telnet lsof unzip zip sysstat autoconf automake make gcc patch gcc-c++ vim dnsmasq ntp zlib-devel bzip2-devel openssl-devel ncurses-devel MySQL-python device-mapper-persistent-data lvm2 libseccomp libtool-ltdl rsync lrzsz java-1.8.0-openjdk

```

 

 



### mysql 安装

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags mysql


```

#####  **容器时间修改：**

```bash
进入mysql容器
# docker exec -it mysql57 bash
修改时间
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
重启mysql容器生效
# docker restart mysql57

```

mysql 安装在 /data/mysql目录，其中启动脚本为/data/mysql/init/start_container.sh

##### **安装前端库**

初始化root和web账户,初始化sdkv表结构

```
进入容器
# docker exec -it mysql57 bash
# mysql  -uroot -pzanalytics -e "drop database sdkv;CREATE DATABASE IF NOT  EXISTS sdkv DEFAULT CHARSET utf8 COLLATE utf8_general_ci;GRANT ALL ON *.* TO 'web'@'%';"
创建hive库
# mysql -uroot -pzanalytics -e 'CREATE DATABASE IF NOT EXISTS hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;GRANT ALL ON `hive`.* TO "hive"@"%"  IDENTIFIED BY "zanalytics";'
```

##### **导入打包生成的sdkv.sql初始化建表语句**，文件位置：

```
# mv 2021.../tools/data/tools/ /data/
mysql -hrealtime-1 -uweb -pzanalytics -Dsdkv -e "source /data/tools/sql/sdkv.sql"  
#需要指定文件全路径否则报错
# 执行后如果有语法报错可忽略，这是导入导出的mysql版本不一致导致的，只要确认导入的表达到301张即 可
```

#####  **修改相应配置**

涉及授权，版本修改，事件量限制修改

```
mysql -hrealtime-1 -uweb -pzanalytics -Dsdkv

update company set  version_stop_time = '2099-01-01 00:00:00' ;
alter table company alter column version_stop_time set default '2099-01-01 00:00:00';
alter table company alter column version set default 11;

alter table company_app alter column event_sum set default 3000;
```

### 安装redis

redis tag会在本地编译一个Redis服务，并通过supervisor来管理

```
# ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags redis
redis 密码已设置，默认：Sun_unix*1101**
```

通过supervisorctl来验证安装：

插曲：这时候发现supervisor不好用，重新安装supervisor

`ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags supervisor`

随后ok



```
# supervisorctl
redis_26480                      RUNNING   pid 27340, uptime 0:00:15

通过客户端验证安装：
redis# redis-cli -h realtime-1 -p 26480 -a Sun_unix*1101
realtime-1:26480> set a 1
OK
realtime-1:26480> get a
"1"
```



### ssdb

ssdb tag会通过gcc-编译安装两个ssdb实例，并通过supervisor来管理

```bash
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags ssdb
```

通过supervisorctl来验证安装：

```bash
# supervisorctl
redis_26480                      RUNNING   pid 27340, uptime 0:05:45
ssdb_8892                        RUNNING   pid 3574, uptime 0:01:46
ssdb_8897                        RUNNING   pid 12192, uptime 0:00:29
```

### Kafka

kafka除了安装启动kafka程序之外，还会初始化所必须的topic，比如sdklua_online 

开始ansible安装新版本kafka

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags kafka
```

验证安装：

```
# supervisorctl
kafka                            RUNNING   pid 1826, uptime 0:00:35
redis_26480                      RUNNING   pid 15194, uptime 0:06:56
ssdb_8892                        RUNNING   pid 23863, uptime 0:02:52
ssdb_8897                        RUNNING   pid 32511, uptime 0:01:38
zookeeper                        RUNNING   pid 1245, uptime 0:00:43

# source /etc/profile
# ./bin/kafka-topics.sh  --zookeeper realtime-1:2182 --list
pay_statisv2
pay_zg_total
sdklua_online

```



### hadoop

  hadoop tag会做很多事情，安装配置启动namenode、datanode、  resourcemanager

```bash
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags hadoop

```

  常见报错：缺少依赖包：  hadoop-lzo

```
1.处理方式：

# yum install -y java-1.8.0-openjdk

(**只需要安装openjdk解决依赖，然后mv /usr/bin/java /usr/bin/java-bak; source /etc/profile)

2.然后安装

yum install -y hadoop-lzo
```



验证方式

```
# hdfs dfs -mkdir /test
# hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - root hadoop          0 2023-08-08 11:43 /test
```

### hive

hive tag会在前端库中初始化hive安装所必须的元信息，然后安装启动hive metastore服务：
然后注释掉deploy_single.yml中执行create_hive_table.sh的相关部分

```bash

ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags hive

```

注：若hive-metastore服务启动报错mysql-connect连接问题，请查看mysql-connector-java-  5.1.38.jar权限，然后重新增加新的权限

chmod 755 /usr/lib/hive/lib/mysql-connector-java-5.1.38.jar



### cdh_impala

cdh_impala tag会根据hosts文件中的配置，在对应机器上安装并启动impala服务

```bash
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags cdh_impala

```



验证安装：

```bash
# impala-shell
Starting Impala Shell without Kerberos authentication
Connected to realtime-1:21000
Server version: impalad version 2.10.0-cdh5.13.1 RELEASE (build 1e4b23c4eb52dac95c5be6316f49685c41783c51)
***********************************************************************************
Welcome to the Impala shell.
(Impala Shell v2.10.0-cdh5.13.1 (1e4b23c) built on Thu Nov  9 08:29:47 PST 2017)

You can change the Impala daemon that you're connected to by using the CONNECT
command.To see how Impala will plan to run your query without actually executing
it, use the EXPLAIN command. You can change the level of detail in the EXPLAIN
output by setting the EXPLAIN_LEVEL query option.
***********************************************************************************

[realtime-1:21000] > create table impala_test (id int, name string); 

Query: create table impala_test (id int, name string)
Fetched 0 row(s) in 0.57s

[realtime-1:21000] > insert into impala_test (id, name) values (1, "Sam");

Query: insert into impala_test (id, name) values (1, "Sam")
Query submitted at: 2023-08-08 11:52:19 (Coordinator: http://realtime-1:25000)
Query progress can be monitored at: http://realtime-1:25000/query_plan?query_id=8f4b738f6a7ac6cb:3d1795eb00000000
Modified 1 row(s) in 4.07s

[realtime-1:21000] > select * from impala_test;

Query: select * from impala_test
Query submitted at: 2023-08-08 11:55:26 (Coordinator: http://realtime-1:25000)
Query progress can be monitored at: http://realtime-1:25000/query_plan?query_id=344014aadfe49592:3c570d0d00000000
+----+------+
| id | name |
+----+------+
| 1  | Sam  |
+----+------+
Fetched 1 row(s) in 0.30s



```



### cdh_kudu

```bash
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags cdh_kudu

```

kudu安装成功与否，可在安装跑批后一并验证，看相关kudu表是否能创建成功。

### spark

单机版中，  spark只是作为一个依赖库存在，不需要启动，只需要安装并且声明SPARK_HOME

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags spark
```

### etl_batch

etl_batch tag会安装跑批程序，创建所需要的python venv，设置相关crontab，并在仓库中创建所需 要的表：

注：Impala仓库修改为zanalytics之后，需要修改etl-batch相关的yml文件，以及软件目录中的配置文 件，将之前的旧的仓库名修改为现在的**zanalytics**，然后执行ansible部署操作。

```bash
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags etl_batch
```

验证建表

```bash
impala-shell
> use zanalytics; 
> show tables;
Query: show tables
+---------------------------+
| name                      |
+---------------------------+
| apple_models              |
| etl_log                   |
| f_user_detail_1           |
| f_user_detail_sum_1       |
| f_user_join_1             |
| filter_data_1_1_1         |
| funnel_data_1_1_1         |
| funnel_data_step_1_1_1    |
| group_data_1_1_1          |
| hdfs_user_label_summary   |
| mccmnc                    |
| network                   |
| order_data_1_1_1          |
| order_data_1_1_3          |
| platform                  |
| retention_data_1_1_1      |
| retention_data_1_1_3      |
| retention_data_step_1_1_1 |
| retention_data_step_1_1_3 |
| t_user_active_1           |
| t_user_detail_1           |
| t_user_detail_sum_1       |
| t_user_duration_1         |
| t_user_join_1             |
| user_label_demo           |
+---------------------------+
Fetched 25 row(s) in 0.01s
```

/data/etl-batch/etl_impala2/init_flush_tables.sql 是数据库初始化脚本

### etl_single

etl_single tag包括了三个步骤：

准备安装：包括向HDFS上传IP库、上传安装包、执行前端库初始化脚本、创建所需要的python venv

安装etl-single：安装并启动etl-single程序

安装etl-single扩展：安装其他的扩展应用，比如服务检查脚本、  HDFS备份工具等等。 注意新增合并操作

ansible与配置管理做了兼容，需要将新生成的etlApp目录提前放入指定位置
> 例如：cp -rp /data/20210317-single-xx/etlApp/ /tmp/
>
> 

```bash

 ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags etl_single

```

检查安装

```
# tail -f /etlLog/etl-single/etl.out
# tail -f /etlLog/etl-single/etl.log
```



### nginx

nginx tag将会导入nginx镜像并启动名为zgnginx容器，同时安装Lua网关程序

! 注意新增合并操作

```
ansible与配置管理做了兼容，需要将新生成的nginx相关目录提前放入指定位置 ,例如
cp -rp /data/20210317-single-玉美/single-nginx /tmp/
cp -rp /data/20210317-single-玉美/statics /tmp/
```



```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags nginx
```

 检查安装：

```
# curl -v http://realtime-1/web_event/web.gif

```

nginx 的启动命令

```
cat /tmp/nginx-cluster-docker.sh

docker run -d --name nginx_lua \
  -p 80:80 \
  -p 443:443 \
  -p 9991:9991 \
  -p 9992:9992 \
  -p 9993:9993 \
  -v /data/nginx/js:/var/www/html \
  -v /data/nginx/conf:/usr/local/nginx_new/conf \
  -v /data/nginx/logs:/usr/local/nginx_new/logs \
  -v /data/nginx/certs:/usr/local/nginx_new/certs \
  -v /data/nginx/lua:/usr/local/nginx_new/lua/sdk-request/lua \
  -v /data/nginx/statics:/usr/local/nginx_new/statics \
  --add-host realtime-1:172.25.130.2 \
  zanalytics/nginx156:lua_zg3 /usr/bin/supervisord
```



### tomcat-api

tomcat-api tag将会安装分析平台API服务，并初始化相关路由信息

 注意新增合并操作

ansible与配置管理做了兼容，需要将新生成的api相关目录提前放入指定位置并改名 ,例如

 cp -rp /data/zhuge/20210317-single-玉美/api/tomcat   /tmp/tomcat-api

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags tomcat-api
```

验证安装

```
# supervisorctl
kafka                            RUNNING   pid 1826, uptime 2:09:18
redis_26480                      RUNNING   pid 15194, uptime 2:15:39
ssdb_8892                        RUNNING   pid 23863, uptime 2:11:35
ssdb_8897                        RUNNING   pid 32511, uptime 2:10:21
tomcat-api                       RUNNING   pid 2565, uptime 0:33:02
zookeeper                        RUNNING   pid 1245, uptime 2:09:26
```



### tomcat-web

tomcat-web tag将会安装分析平台网站的后端

 注意新增合并操作

ansible与配置管理做了兼容，需要将新生成的api相关目录提前放入指定位置并改名 ,例如

```
cp -rp /data/20210317-single-玉美/web/tomcat /tmp/tomcat-web

cd /tmp/tomcat-web/bin

chmod +x *.sh
```

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags tomcat-web
```

此时，系统已经可以通过浏览器访问，并可以创建账号和测试应用。

默认账号/密码：13552145052/123456

验证安装

```
# supervisorctl
kafka                            RUNNING   pid 1826, uptime 2:15:13
redis_26480                      RUNNING   pid 15194, uptime 2:21:34
ssdb_8892                        RUNNING   pid 23863, uptime 2:17:30
ssdb_8897                        RUNNING   pid 32511, uptime 2:16:16
tomcat-api                       RUNNING   pid 2565, uptime 0:38:57
tomcat-web                       RUNNING   pid 5420, uptime 0:01:08
zookeeper                        RUNNING   pid 1245, uptime 2:15:21
```



### 实时调试 -debug

注意新增合并操作

ansible与配置管理做了兼容，需要将新生成的debug相关目录提前放入指定位置并改名 ,例如 cp -rp /data/20210317-single-玉美/zanalytics_online_debug/ /tmp/

```
ansible-playbook -i ./hosts ./deploy_single.yml -e '@global.yml' --tags online-debug
```

注意：  nginx配置文件中确认是否为16688端口

```
# debug
upstream zgsvr_debug {
server realtime-1:16688;
}
```

 验证

```
# curl realtime-1:16688/debug
Welcome to SockJS!
```

### crontal定时任务部署

定时文件位置：  $realtime-dj/crontab.txt,导入crontab，先把已有的配置清空，然后导入：

```
ansible-playbook -i ./hosts ./manage_crontabs.yml -e '@global.yml'
```

然后发现如下脚本没有复制过去

```
*/5 * * * * sh /server/scripts/cdh-maintain.sh >> /var/log/cdh-maintain.log
15 4 * * * sh /server/scripts/mysql_bak.sh
30 23 * * * /bin/sh /server/scripts/cut_nginx_log.sh  & > /tmp/nginx-cut.log
*/5 * * * * /bin/bash /server/scripts/check-nginx-log.sh
30 9,13,18 * * * /usr/bin/sh /server/scripts/DJ_new_check_all.sh
0 20 * * * sh /server/scripts/deleteMergeDir.sh > /tmp/deleteMergeFile.log 2>&1
```

发现这些文件在 realtime-dj/SingleVersion/softwares/scripts目录，就复制过去

```bash
 mkdir /server
 cp -r SingleVersion/softwares/scripts /server/
```

