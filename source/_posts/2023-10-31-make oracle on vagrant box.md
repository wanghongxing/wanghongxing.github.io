make oracle on vagrant box

https://geraldonit.com/2018/06/11/creating-an-oracle-database-vagrant-box/

根据 

生成两个vm，使用oraclelinux/7 ， 分别是192.168.56.10 192.168.56.11



```properties

# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Box metadata location and box name
BOX_URL = "https://oracle.github.io/vagrant-projects/boxes"
BOX_NAME = "oraclelinux/7"

# UI object for printing information
ui = Vagrant::UI::Prefixed.new(Vagrant::UI::Colored.new, "vagrant")

# Define constants
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use vagrant-env plugin if available
  if Vagrant.has_plugin?("vagrant-env")
    config.env.load(".env.local", ".env") # enable the plugin
  end

  # VM name
  VM_NAME = default_s('VM_NAME', 'oracle11g-vagrant-whx')

  # Memory for the VM (in MB, 2048 MB = 2 GB)
  VM_MEMORY = default_i('VM_MEMORY', 4096)

  # VM time zone
  # If not specified, will be set to match host time zone (if possible)
  VM_SYSTEM_TIMEZONE = default_s('VM_SYSTEM_TIMEZONE', host_tz)

  # Listener port
  VM_LISTENER_PORT = default_i('VM_LISTENER_PORT', 1521)
  VM_LISTENER_HOST_PORT = default_i('VM_LISTENER_HOST_PORT', 1521)

  # Oracle Database password for SYS and SYSTEM accounts
  # If left blank, the password will be generated automatically
  VM_ORACLE_PWD = default_s('VM_ORACLE_PWD', '')
end

# Convenience methods
def default_s(key, default)
  ENV[key] && ! ENV[key].empty? ? ENV[key] : default
end

def default_i(key, default)
  default_s(key, default).to_i
end

def host_tz
  # get host time zone for setting VM time zone
  # if host time zone isn't an integer hour offset from GMT, fall back to UTC
  offset_sec = Time.now.gmt_offset
  if (offset_sec % (60 * 60)) == 0
    offset_hr = ((offset_sec / 60) / 60)
    timezone_suffix = offset_hr >= 0 ? "-#{offset_hr.to_s}" : "+#{(-offset_hr).to_s}"
    'Etc/GMT' + timezone_suffix
  else
    'UTC'
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = BOX_NAME
  config.vm.box_url = "#{BOX_URL}/#{BOX_NAME}.json"
  config.vm.define VM_NAME

  # Provider-specific configuration
  config.vm.provider "virtualbox" do |v|
    v.memory = VM_MEMORY
    v.name = VM_NAME
  end
  config.vm.provider :libvirt do |v|
    v.memory = VM_MEMORY
  end

  config.vm.network :private_network, ip: "192.168.56.10"

  # add proxy configuration from host env - optional
  if Vagrant.has_plugin?("vagrant-proxyconf")
    ui.info "Getting Proxy Configuration from Host..."
    has_proxy = false
    ["http_proxy", "HTTP_PROXY"].each do |proxy_var|
      if proxy = ENV[proxy_var]
        ui.info "HTTP proxy: " + proxy
        config.proxy.http = proxy
        has_proxy = true
        break
      end
    end

    ["https_proxy", "HTTPS_PROXY"].each do |proxy_var|
      if proxy = ENV[proxy_var]
        ui.info "HTTPS proxy: " + proxy
        config.proxy.https = proxy
        has_proxy = true
        break
      end
    end

    if has_proxy
      # Only consider no_proxy if we have proxies defined.
      no_proxy = ""
      ["no_proxy", "NO_PROXY"].each do |proxy_var|
        if ENV[proxy_var]
          no_proxy = ENV[proxy_var]
          ui.info "No proxy: " + no_proxy
          no_proxy += ","
          break
        end
      end
      config.proxy.no_proxy = no_proxy + "localhost,127.0.0.1"
    end
  else
    ["http_proxy", "HTTP_PROXY", "https_proxy", "HTTPS_PROXY"].each do |proxy_var|
      if ENV[proxy_var]
        ui.warn 'To enable proxies in your VM, install the vagrant-proxyconf plugin'
        break
      end
    end
  end

  # VM hostname
  config.vm.hostname = VM_NAME

  # Oracle port forwarding
  config.vm.network "forwarded_port", guest: VM_LISTENER_PORT, host: VM_LISTENER_HOST_PORT

  # Provision everything on the first run
  config.vm.provision "shell", path: "scripts/install.sh", env:
    {
       "SYSTEM_TIMEZONE"     => VM_SYSTEM_TIMEZONE,
       "LISTENER_PORT"       => VM_LISTENER_PORT,
       "ORACLE_PWD"          => VM_ORACLE_PWD
    }

end

```

安装依赖

```
yum install binutils  compat-libstdc++-33  compat-libstdc++-33.i686  gcc   gcc-c++  glibc  glibc.i686  glibc-devel  glibc-devel.i686  ksh  libgcc   libgcc.i686  libstdc++ libstdc++.i686 libstdc++-devel  libstdc++-devel.i686  libaio  libaio.i686  libaio-devel  libaio-devel.i686  libXext libXext.i686 libXtst libXtst.i686 libX11 libX11.i686  libXau  libXau.i686 libxcb  libxcb.i686   libXi  libXi.i686  make  sysstat  unixODBC unixODBC-devel  zlib-devel  elfutils-libelf-devel -y

yum install -y xorg-x11-xauth xorg-x11-font-utils xorg-x11-fonts-* xorg-x11-fonts-Type1


```

 修改/etc/ssh/sshd_config

```
X11forwarding yes
X11UseLocalhost no
X11DisplayOffset 10
```



退出终端

生成进入x终端的脚本

```bash
#!/bin/sh
PORT=$(vagrant ssh-config | grep Port | grep -o '[0-9]\+')

IDFILE=$(vagrant ssh-config |grep IdentityFile|awk  '{print $2}')
ssh -Y  \
    -o UserKnownHostsFile=/dev/null \
    -o StrictHostKeyChecking=no \
    -o "XAuthLocation=/opt/X11/bin/xauth" \
    -i $IDFILE \
    vagrant@localhost \
    -p $PORT \
    "$@"




```





XQuartz中启动terminal，执行命令：

```
./whx.sh
echo $DISPLAY

cd /data/database
./runInstaller
```

root用户执行：

```
/home/vagrant/app/oraInventory/orainstRoot.sh
/home/vagrant/app/vagrant/product/11.2.0/dbhome_1/root.sh
```



### master设置：

设置sid： whx

设置密码：zaq1XSW2



```
export ORACLE_SID=whx
export ORACLE_BASE=/home/vagrant/app/vagrant
export ORACLE_HOME=/home/vagrant/app/vagrant/product/11.2.0/dbhome_1

PATH=$PATH:$ORACLE_HOME/bin
```



### slaver设置

设置sid： orcl

设置密码：zaq1XSW2

```
export ORACLE_SID=orcl
export ORACLE_BASE=/home/vagrant/app/vagrant
export ORACLE_HOME=/home/vagrant/app/vagrant/product/11.2.0/dbhome_1

PATH=$PATH:$ORACLE_HOME/bin 
```

启动oracle

```
dbstart  $ORACLE_HOME
```

### 下面开始试着主从复制

https://www.cnblogs.com/hooly/p/8178570.html



进入主从，分别执行

```
sqlplus / as sysdba
alter system set aq_tm_processes=2 scope=both;   ---启用对队列消息的时间监视
alter system set global_names=true scope=both;   ---设置全局名称为true
alter system set undo_retention=3600 scope=both;   --设置回滚段时间,默认是900
alter system set streams_pool_size=25M scope=spfile; --sga设置为自动调整情况下不需设置该参数

```



**设置主数据库（whx ）为归档mode （以as sysdba身份，可在sqlplus中执行）**

（以as sysdba身份，可在sqlplus中执行）

 查是否归档,如是归档,请忽略第3点 

```
SQL> archive log list;

Database log mode	       No Archive Mode
Automatic archival	       Disabled
Archive destination	       USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     6
Current log sequence	       8
```

归档设置

```
shutdown immediate;
startup mount;
alter database archivelog;
alter system set LOG_ARCHIVE_DEST_1='LOCATION=/home/vagrant/app/archive';  ---设置归档目录 （执行此句后，在Windows系统文件夹中看看arc文件夹有没有创建成功，如果没有，则手动创建，在执行此语句）
alter database open;
alter system switch logfile; --相应目录检查是否生成arc文件 （如果提示数据库没开启，则先执行alter database open;）

```



**主/从数据新建stream管理用户(在主从数据库都执行以下操作)** 

```
sqlplus / as sysdba;
create tablespace tbs_stream datafile '/home/vagrant/app/vagrant/oradata/whx/tbs_stream01.dbf' size 2000m autoextend on maxsize unlimited segment space management auto; ---创建主环境的Stream专用表空间
execute dbms_logmnr_d.set_tablespace('tbs_stream'); --将logminer的数据字典从system表空间转移到新建的表空间，防止撑满system表空间
create user strmadmin identified by strmadmin default tablespace tbs_stream temporary tablespace temp;  --创建用户
grant dba to strmadmin;  --直接给dba权限.

```



```
sqlplus / as sysdba;
create tablespace tbs_stream datafile '/home/vagrant/app/vagrant/oradata/orcl/tbs_stream01.dbf' size 2000m autoextend on maxsize unlimited segment space management auto; ---创建主环境的Stream专用表空间
execute dbms_logmnr_d.set_tablespace('tbs_stream'); --将logminer的数据字典从system表空间转移到新建的表空间，防止撑满system表空间
create user strmadmin identified by strmadmin default tablespace tbs_stream temporary tablespace temp;  --创建用户
grant dba to strmadmin;  --直接给dba权限.


```



主数据库新建连接从数据库的link

```
create public database link orcl connect to strmadmin IDENTIFIED BY strmadmin
using '(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.11)(PORT = 1521))
)
(CONNECT_DATA =
(SID= orcl)
)
)';
```



从数据库新建连接主数据库的link

```
create public database link whx connect to strmadmin IDENTIFIED BY strmadmin
using '(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.10)(PORT = 1521))
)
(CONNECT_DATA =
(SID = whx)
)
)';
```

select sysdate from dual@whx



**主数据库流队列创建 （可在plsql中的SQL中执行，登录名应为strmadmin）**

```
connect strmadmin/strmadmin  
begin
dbms_streams_adm.set_up_queue(queue_table => 'whx_queue_table',queue_name => 'whx_queue');
end;
/
```

提示：PL/SQL procedure successfully completed.



**从数据库流队列创建**

```
connect strmadmin/strmadmin  --以strmadmin身份，登录从数据库。
begin
dbms_streams_adm.set_up_queue(queue_table => 'orcl_queue_table',queue_name => 'orcl_queue');
end;
/
```

提示：PL/SQL procedure successfully completed.

**主数据库创建捕获进程** 

```
connect strmadmin/strmadmin

begin
dbms_streams_adm.add_schema_rules(
schema_name => 'strmadmin',
streams_type => 'capture',
streams_name => 'capture_whx',
queue_name => 'strmadmin.whx_queue',
include_dml => true,
include_ddl => true,
include_tagged_lcr => false,
source_database => null,
inclusion_rule => true);
end;
/

```

提示：PL/SQL procedure successfully completed.

**从数据库实例化strmadmin用户  （这两个路径须一致）**

```
exp strmadmin/strmadmin@whx file='/home/vagrant/crm.dmp' object_consistent=y rows=y
exp strmadmin/strmadmin@orcl file='/home/vagrant/crm.dmp' object_consistent=y rows=y

```

这个执行失败

```
EXP-00056: ORACLE error 12154 encountered
ORA-12154: TNS:could not resolve the connect identifier specified
EXP-00000: Export terminated unsuccessfully
```

修改 /home/vagrant/app/vagrant/product/11.2.0/dbhome_1/network/admin/tnsnames.ora增加 WHX部分内容

```

ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = oracle11g-vagrant-slave)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

WHX =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.10)(PORT = 1521))
    (CONNECT_DATA =
      (SID = whx)
    )
  )

```

然后导出正常。



在从数据库新建strmadmin 

```
imp strmadmin/strmadmin@orcl file='/home/vagrant/crm.dmp' ignore=y commit=y streams_instantiation=y full=y


Import: Release 11.2.0.1.0 - Production on Tue Oct 31 23:08:12 2023

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.


Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

Export file created by EXPORT:V11.02.00 via conventional path
import done in US7ASCII character set and AL16UTF16 NCHAR character set
import server uses AL32UTF8 character set (possible charset conversion)
. importing STRMADMIN's objects into STRMADMIN
. . importing table        "AQ$_WHX_QUEUE_TABLE_C"          0 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_G"          0 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_H"
Note: table contains ROWID column, values may be obsolete          0 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_I"
Note: table contains ROWID column, values may be obsolete          0 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_L"          0 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_S"          1 rows imported
. . importing table        "AQ$_WHX_QUEUE_TABLE_T"          0 rows imported
. . importing table              "WHX_QUEUE_TABLE"          0 rows imported
Import terminated successfully without warnings.

```

**主数据库创建传播进程** 

```
connect strmadmin/strmadmin
begin
dbms_streams_adm.add_schema_propagation_rules(
schema_name => 'strmadmin',
streams_name => 'whx_to_orcl',
source_queue_name => 'strmadmin.whx_queue',
destination_queue_name => 'strmadmin.orcl_queue@orcl',
include_dml => true,
include_ddl => true,
include_tagged_lcr => false,
source_database => 'whx',
inclusion_rule => true);
end;
/
```



注意：此段语句执行可能会报错，如果报错，不用管，继续执行后面的。

修改propagation休眠时间为0，表示实时传播LCR，latency以秒为单位 

```
begin
dbms_aqadm.alter_propagation_schedule(
queue_name => 'whx_queue',
destination => 'orcl',
latency => 0);
end;
/
```



**从数据创建应用进程**

```
connect strmadmin/strmadmin
begin
dbms_streams_adm.add_schema_rules(
schema_name => 'strmadmin',
streams_type => 'apply',
streams_name => 'apply_orcl',
queue_name => 'strmadmin.orcl_queue',
include_dml => true,
include_ddl => true,
include_tagged_lcr => false,
source_database => 'whx',
inclusion_rule => true);
end;
/
```



**启动Stream**

从数据库启动应用进程 

```
connect strmadmin/strmadmin
begin
dbms_apply_adm.start_apply(
apply_name => 'apply_orcl');
end;
/
```

主数据库启动捕获进程 

```
begin
dbms_capture_adm.start_capture(
capture_name => 'capture_whx');
end;
/
```



**现在就可以进行测试了，在whx用户中作何一个测试表新增数据,删除数据,增加表,修改表结构,进行同步测试**

