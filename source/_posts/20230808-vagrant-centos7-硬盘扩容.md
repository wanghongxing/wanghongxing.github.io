vagrant centos7 硬盘扩容



vagrantfile 中增加

```
config.disksize.size = "200GB"
```

vagrant reload

安装工具 

```bash
vagrant ssh

yum install cloud-utils-growpart gdisk -y 

# growpart /dev/sda 1
CHANGED: partition=1 start=2048 old: size=83884032 end=83886080 new: size=419428319 end=419430367

# xfs_growfs /
meta-data=/dev/sda1              isize=512    agcount=4, agsize=2621376 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=10485504, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=5119, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 10485504 to 52428539
# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  8.5M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1       200G  3.6G  197G   2% /
tmpfs           379M     0  379M   0% /run/user/1000
```

