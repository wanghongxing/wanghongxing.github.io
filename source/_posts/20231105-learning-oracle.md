20231105-learning-oracle学习oracle基础



登录oracle

本地连接

```bash
sqlplus system/passwd
select * from tabs;
```

远程连接

```bash
sqlplus system/zaq1XSW2@192.168.56.10
sqlplus system/zaq1XSW2@192.168.56.10/whx
```

创建表空间

```sql
create tablespace waterboss
datafile '/home/vagrant/tb/waterboss.dbf'
size 100m
autoextend on
next 10m;
```

创建用户

```sql
create user wateruser
identified by zaq1XSW2
default tablespace waterboss;

grant dba to wateruser;
```

建个表

```sql
SQL> create table test(
id number(11) primary key,
name varchar2(200),
age number(3),
birthday date,
lastvisit timestamp  
);
SQL> desc test;
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 ID					   NOT NULL NUMBER(11)
 NAME						    VARCHAR2(200)
 AGE						    NUMBER(3)
 BIRTHDAY					    DATE
 LASTVISIT					    TIMESTAMP(6) 

SQL> alter table test add whx varchar2(100);

Table altered.

SQL> desc test;
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 ID					   NOT NULL NUMBER(11)
 NAME						    VARCHAR2(200)
 AGE						    NUMBER(3)
 BIRTHDAY					    DATE
 LASTVISIT					    TIMESTAMP(6)
 WHX						    VARCHAR2(100)
 
SQL> insert into test(id,name,age) values(1,'whx',12);
1 row created.

```

导出

```
exp wateruser/passwd owner=wateruser file=water.dmp
```

导入

```
imp wateruser/passwd file=water.dmp fromuser=wateruser

```



 

