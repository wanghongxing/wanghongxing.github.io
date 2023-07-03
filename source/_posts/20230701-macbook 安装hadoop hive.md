macbook 安装hadoop hive

```
brew install hardoop

brew install hive
```

## 配置hadoop 

hadoop安装的是3.3.4

echo "127.0.0.1 wanghongxing" >> /etc/hosts



目录是 /usr/local/Cellar/hadoop/3.3.4/

进入 /usr/local/Cellar/hadoop/3.3.4/目录，

```
cd  /usr/local/Cellar/hadoop/3.3.4/
mkdir tmp
mkdir -p dfs/name
mkdir hadoop
```

进入libexec目录，修改etc下的配置文件

- core-site.xml
- hdfs-site.xml
- mapred-site.xml
- yarn-site.xml

### **修改 core-site.xml 文件**

设置 Hadoop 的临时目录和文件系统，localhost:9000 表示本地主机。如果使用远程主机，要用相应的 IP 地址来代替，填写远程主机的域名，则需要到 /etc/hosts 文件中做 DNS 映射。在 core-site.xml 文件里作如下配置：

```xml
<configuration>
	<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>

  <!--用来指定hadoop运行时产生文件的存放目录  自己创建-->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/Cellar/hadoop/3.3.4/tmp</value>
  </property>
</configuration>
```

### **修改 hdfs-site.xml 文件**

hdfs-site.xml 的配置修改如下，注意 name 和 data 的路径都要替换成本地的路径：

```xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
	<!--不是root用户也可以写文件到hdfs-->
	<property>
		<name>dfs.permissions</name>
		<value>false</value>    <!--关闭防火墙-->
	</property>
  <!--把路径换成本地的name坐在位置-->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/usr/local/Cellar/hadoop/3.3.4/dfs/name</value>
  </property>
  <!--在本地新建一个存放hadoop数据的文件夹，然后将路径在这里配置一下-->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/Cellar/hadoop/3.3.4/hadoop</value>
  </property>
</configuration>
```

### **修改 mapred-site.xml 文件**

由于根目录下 etc/hadoop 中没有 mapred-site.xml 文件，所以需要创建该文件。但是目录中提供了 mapred-site.xml.template 模版文件。我们将其重命名为 mapred-site.xml，然后将 yarn 设置成数据处理框架：

```xml
<configuration>
  <property>
    <!--指定mapreduce运行在yarn上-->
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

### **修改 yarn-site.xml 文件**

配置数据的处理框架 yarn：

```xml
<configuration>
  <!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>localhost:8088</value>
  </property>
</configuration>
```



### 名称节点设置

```
$ hdfs namenode -format

```

### 验证Hadoop  

​	 

```
$ sbin/start-all.sh
```

 

### 在浏览器访问Hadoop

​	访问Hadoop的默认端口号为9870(老版本的是 50070 ). 使用以下网址，以获取浏览器Hadoop服务。

```
http://localhost:9870/
```

 



### 验证集群的所有应用程序

​	访问集群中的所有应用程序的默认端口号为8088。使用以下URL访问该服务。

```
http://localhost:8088/
```

###  重启

关闭

```

sbin/stop-all.sh

rm -rf tmp/dfs

sbin/start-all.sh

```



至此，hadoop看着么有问题。

-----





## 配置hive

### 在 ~/.bash_profile文件新增

```
#Setting PATH for Hive

export HIVE_HOME=/usr/local/Cellar/hive/3.1.3/libexec

export PATH=$PATH:HIVE_HOME/bin
```



### mysql

```
create database hivestore;

 CREATE USER  'hardoop'@'%'  IDENTIFIED BY 'WhxHeart12#';

GRANT ALL PRIVILEGES ON  *.* TO 'hardoop'@'%' WITH GRANT OPTION;

flush privileges;
```

### **修改Hive配置文件



```
cd $HIVE_HOME/conf
cp hive-default.xml.template hive-site.xml
vim hive-site.xml
```



```xml
<configuration>
<property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hadoop</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hivestore</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>
    
    <property>
      <name>hive.exec.local.scratchdir</name>
      <value>/usr/local/Cellar/hive/3.1.3/libexec/iotmp</value>
    </property>
    <property>
      <name>hive.querylog.location</name>
      <value>/usr/local/Cellar/hive/3.1.3/libexec/iotmp</value>
  	</property>
    <property>
      <name>hive.downloaded.resources.dir</name>
      <value>/usr/local/Cellar/hive/3.1.2/libexec/iotmp</value>
    </property>
  
</configuration>
```

### **下载mysql连接器**

在https://dev.mysql.com/downloads/connector/j/

mysql-connector，下载选platform independent的操作系统。解压以后，把jar文件复制到/usr/local/Cellar/hive/3.1.3/libexec/lib目录下面。



在/usr/local/Cellar/hive/3.1.2/libexec/（即$HIVE_HOME）文件夹内新建iotmp文件夹



### **初始化库**

在/usr/local/Cellar/hive/3.1.2/libexec/bin目录下



```
schematool -initSchema -dbType mysql
```

查看初始化信息



```
schematool -dbType mysql -info
```



### 启动hive

```
wanghongxing:~ whx$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/Cellar/hive/3.1.3/libexec/lib/log4j-slf4j-impl-2.17.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/Cellar/hadoop/3.3.4/libexec/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Hive Session ID = 8a401692-9f09-41f6-babb-0ac77fd8eb16

Logging initialized using configuration in jar:file:/usr/local/Cellar/hive/3.1.3/libexec/lib/hive-common-3.1.3.jar!/hive-log4j2.properties Async: true
Hive Session ID = 380b0568-e219-453f-abf8-12eb4d7cf331
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive> show  databases ;
OK
default
Time taken: 0.798 seconds, Fetched: 1 row(s)
hive>

```



---

折腾了半天，重新看hadoop，启动总是出问题，因为自己是arm芯片的macbook，决定用docker的方式折腾。

从 https://github.com/big-data-europe/docker-hadoop clone 了他的代码库

看他头打包镜像的脚本，执行make build就可以打包。

```
DOCKER_NETWORK = docker-hadoop_default
ENV_FILE = hadoop.env
current_branch := $(shell git rev-parse --abbrev-ref HEAD)
build:
	docker build -t bde2020/hadoop-base:$(current_branch) ./base
	docker build -t bde2020/hadoop-namenode:$(current_branch) ./namenode
	docker build -t bde2020/hadoop-datanode:$(current_branch) ./datanode
	docker build -t bde2020/hadoop-resourcemanager:$(current_branch) ./resourcemanager
	docker build -t bde2020/hadoop-nodemanager:$(current_branch) ./nodemanager
	docker build -t bde2020/hadoop-historyserver:$(current_branch) ./historyserver
	docker build -t bde2020/hadoop-submit:$(current_branch) ./submit

wordcount:
	docker build -t hadoop-wordcount ./submit
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} bde2020/hadoop-base:$(current_branch) hdfs dfs -mkdir -p /input/
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} bde2020/hadoop-base:$(current_branch) hdfs dfs -copyFromLocal -f /opt/hadoop-3.2.3/README.txt /input/
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} hadoop-wordcount
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} bde2020/hadoop-base:$(current_branch) hdfs dfs -cat /output/*
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} bde2020/hadoop-base:$(current_branch) hdfs dfs -rm -r /output
	docker run --network ${DOCKER_NETWORK} --env-file ${ENV_FILE} bde2020/hadoop-base:$(current_branch) hdfs dfs -rm -r /input

```

 

另外Dockerfile是从debian 来做，安装arm64版jdk比较麻烦，因为改成openjdk11,同时把jdk相应的安装去掉了，顺便把hardoop升级到3.2.3

```
FROM openjdk:11

MAINTAINER Ivan Ermilov <ivan.s.ermilov@gmail.com>
MAINTAINER Giannis Mouchakis <gmouchakis@iit.demokritos.gr>

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      net-tools \
      curl \
      netcat \
      gnupg \
      libsnappy-dev \
    && rm -rf /var/lib/apt/lists/*
      
# ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/

RUN curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS

RUN gpg --import KEYS

ENV HADOOP_VERSION 3.2.3
ENV HADOOP_URL https://www.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz

RUN set -x \
    && curl -fSL "$HADOOP_URL" -o /tmp/hadoop.tar.gz \
    && curl -fSL "$HADOOP_URL.asc" -o /tmp/hadoop.tar.gz.asc \
    && gpg --verify /tmp/hadoop.tar.gz.asc \
    && tar -xvf /tmp/hadoop.tar.gz -C /opt/ \
    && rm /tmp/hadoop.tar.gz*

RUN ln -s /opt/hadoop-$HADOOP_VERSION/etc/hadoop /etc/hadoop

RUN mkdir /opt/hadoop-$HADOOP_VERSION/logs

RUN mkdir /hadoop-data

ENV HADOOP_HOME=/opt/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=/etc/hadoop
ENV MULTIHOMED_NETWORK=1
ENV USER=root
ENV PATH $HADOOP_HOME/bin/:$PATH

ADD entrypoint.sh /entrypoint.sh

RUN chmod a+x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

```

这个弄好后hadoop没有问题，但是加上 hive后总是有问题。



拿出intel芯片的macbook，这样搞。
