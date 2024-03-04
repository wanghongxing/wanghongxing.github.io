---
title: 2023-0928-docker-compose-oracle
---

docker-compose 安装oracle





使用 https://hub.docker.com/r/oracleinanutshell/oracle-xe-11g 镜像

创建docker-compose.yml

```yaml
version: '3'

services:
  oracle-db:
    image: oracleinanutshell/oracle-xe-11g:latest
    ports:
      - 1521:1521
      - 5500:5500
    environment:
      - ORACLE_ALLOW_REMOTE:true

```

拉镜像

 docker-compose pull

启动后

docker-compose up -d



复制/u01 到 ./data

docker cp dck_oracle-db_1:/u01 ./data



然后修改docker-compose.yml

```yaml
version: '3'

services:
  oracle-db:
    image: oracleinanutshell/oracle-xe-11g:latest
    ports:
      - 1521:1521
      - 5500:5500
    environment:
      - ORACLE_ALLOW_REMOTE:true
    volumes:
      - ./data:/u01
```

这时候数据就保存在本地硬盘了



docker-compose exec oracle-db bash

sql-plus

user：system

pwd: oracle

By default, the password verification is disable(password never expired)
Connect database with following setting:

```
hostname: localhost
port: 49161
sid: xe
username: system
password: oracle
```

Password for SYS & SYSTEM

```
oracle
```





最新版



https://container-registry.oracle.com/

这里是直接下载各种版本的地方 https://www.oracle.com/cn/database/technologies/oracle-database-software-downloads.html#db_free

为了安装docker版本， 必须在 https://container-registry.oracle.com/ords/f?p=113:1:102342682778790:::1:P1_BUSINESS_AREA:3&cs=3T4nkcPYyj6ewF0qib0dU82LAmedE6CKr9q8rmOAgxLycqz9EVlXaoK41xRcvBalvRR4e6uVOdKMXmQIvBkPMdw 同意oracle的terms后才能pull

```
docker login container-registry.oracle.com/

user: wanghongxing@gmail.com

pwd: *Y4r6?C@~c7jX!h
```



# Tags

Download Mirror

| Tag           | OS/Architecture | Pull Command                                                 | Last Updated  |
| ------------- | --------------- | ------------------------------------------------------------ | ------------- |
| 19.3.0.0      | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:19.3.0.0 | 5 weeks ago   |
| latest        | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:latest | 5 weeks ago   |
| 21.3.0.0      | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:21.3.0.0 | 5 weeks ago   |
| 19.19.0.0     | linux/arm64     | docker pull container-registry.oracle.com/database/enterprise:19.19.0.0 | 2 months ago  |
| 12.2.0.1      | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:12.2.0.1 | 6.1 years ago |
| 12.2.0.1-slim | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:12.2.0.1-slim | 6.1 years ago |
| 12.1.0.2      | linux/amd64     | docker pull container-registry.oracle.com/database/enterprise:12.1.0.2 | 6.2 years ago |