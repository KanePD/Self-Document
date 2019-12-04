[TOC]

# iSCSI

## 定义

`SCSI` 是 `Small Computer System Interface`。

`iSCSI`就是Internet小型计算机系统接口，基于`TCP/IP`协议。通过网络仿真`SCSI`高性能本地存储总栈，从而为远程块存储设备提供数据传输和管理。

## 组件术语

### 启动器

一个`iSCSI`客户端，通常是软件提供，必须为启动器授予一个唯一名称(IQN)

### 目标

一个`iSCSI`存储资源，针对来自`iSCSI`服务器的连接而配置。必须为目标授予一个唯一名称(IQN)。一个服务器可以同时提供多个目标。

### ACL

访问权限控制条目，一个使用节点`IQN`来验证启动器的范根权限的访问限制。

### 发现

查询目标服务器以流出配置的目标

### IQN

iSCSI限定名称，一个全球唯一名称，用于以强制命名格式来识别启动器和目标。

`iqn.YYYY-MM.com.reversed.domain[:optional_string]`

```markdown
`iqn` 表示此名称将使用域作为其标识符
`YYYY-MM`拥有域名的第一个月
`com.reversed.domain`组织的逆向域名
```

### 登录

项目表进行身份验证以开始说那个客户端块设备

### LUN

逻辑单元号。带有编号的块设备连接到目标并且通过目标来使用。可以有一个或多个LUN连接到单个目标，但是通常一个目标仅提供一个LUN

### 节点

任何iSCSI启动器或目标由其`IQN`来标识

### 门户

目标或启动器上用于建立连接的IP地址和端口

### TPG

目标门户组，某个特定的`iSCSI`目标将要侦听的接口IP地址和TCP端口的集合

## 搭建一个iSCSI服务

### 提供一个iSCSI目标

#### 安装`targetcli`

```bash
[root@rhel1 ~]# yum install -y targetcli
```

#### 创建新的分区

在原有的基础上创建了`/dev/sdb7`的分区类型是`Linux LVM (8e)`

```bash
[root@rhel1 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xed07c356

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20973567    10485760    5  Extended
/dev/sdb5            4096      413695      204800   83  Linux
/dev/sdb6          415744      825343      204800   83  Linux

Command (m for help): n
Partition type:
   p   primary (0 primary, 1 extended, 3 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 7
First sector (827392-20973567, default 827392): 
Using default value 827392
Last sector, +sectors or +size{K,M,G} (827392-20973567, default 20973567): +500M
Partition 7 of type Linux and of size 500 MiB is set

Command (m for help): t
Partition number (1,5-7, default 7): 
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xed07c356

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20973567    10485760    5  Extended
/dev/sdb5            4096      413695      204800   83  Linux
/dev/sdb6          415744      825343      204800   83  Linux
/dev/sdb7          827392     1851391      512000   8e  Linux LVM

Command (m for help): w
[root@rhel1 ~]# partprobe
```

#### 创建逻辑卷

```bash
[root@rhel1 ~]# pvcreate /dev/sdb7
  Physical volume "/dev/sdb7" successfully created
[root@rhel1 ~]# vgcreate iSCSI /dev/sdb7
  Volume group "iSCSI" successfully created
[root@rhel1 ~]# 
[root@rhel1 ~]# lvcreate -n iSCSI1 -L 100M iSCSI 
  Logical volume "iSCSI1" created
```

#### 运行`targetcli`

进入后是交互模式

```bash
[root@rhel1 ~]# targetcli
/> ls
o- / .............................................................. [...]
  o- backstores ................................................... [...]
  | o- block ....................................... [Storage Objects: 0]
  | o- fileio ...................................... [Storage Objects: 0]
  | o- pscsi ....................................... [Storage Objects: 0]
  | o- ramdisk ..................................... [Storage Objects: 0]
  o- iscsi ................................................. [Targets: 0]
  o- loopback .............................................. [Targets: 0]
/> 
```

#### 添加`块BLOCK`

```bash
/> backstores/block create server_disk1 /dev/iSCSI/iSCSI1 
Created block storage object server_disk1 using /dev/iSCSI/iSCSI1.
```

#### 常见一个目标并配置`IQN`

```bash
/> iscsi/ create iqn.2019-11.com.node:control1
Created target iqn.2019-11.com.node:control1.
Created TPG 1.

```

#### 为目标添加一个启动器并配置`IQN`

````bash
/> iscsi/iqn.2019-11.com.node:control1/tpg1/acls create iqn.2019-11.com.node:control2
Created Node ACL for iqn.2019-11.com.node:control2
# 一会客户端会通过iqn.2019-11.com.node:control2这个IQN连接过来
````

#### 为目标分配一个`LUN`逻辑单元

```bash
/> iscsi/iqn.2019-11.com.node:control1/tpg1/luns create /backstores/block/server_disk1 
Created LUN 0.
Created LUN 0->0 mapping in node ACL iqn.2019-11.com.node:control2
# 也就是说连接到这个目标就可以使用这个块设备了。可以配置多个
```

#### 配置门户

```bash
/> iscsi/iqn.2019-11.com.node:control1/tpg1/portals create 192.168.143.10Using default IP port 3260
Created network portal 192.168.143.10:3260.
# 配置当前服务器IP 端口号默认3260可以更换
```

#### 保存配置并退出

```bash
/> saveconfig
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json

```

#### 防火墙放行，启动服务

```bash
[root@rhel1 ~]# firewall-cmd --add-port=3260/tcp --permanent 
success
[root@rhel1 ~]# firewall-cmd --add-port=3260/tcp 
success
[root@rhel1 ~]# systemctl start target
[root@rhel1 ~]# systemctl enable target
ln -s '/usr/lib/systemd/system/target.service' '/etc/systemd/system/multi-user.target.wants/target.service'
```

#### 查看状态

````bash
[root@rhel1 ~]# netstat -tunpl | grep 3260
tcp        0      0 192.168.143.10:3260     0.0.0.0:*               LISTEN      -       
````

### 配置iSCSI客户端

使用`RHEL2`充当客户端

#### 配置

```bash
# 安装
[root@rhel2 ~]# yum install iscsi*
# 启动
[root@rhel2 ~]# systemctl restart iscsi
[root@rhel2 ~]# systemctl enable iscsi
# 配置
### 此处添加的是服务器那边配置的ACL的IQN
[root@rhel2 ~]# cat /etc/iscsi/initiatorname.iscsi 
InitiatorName=iqn.2019-11.com.node:control2
```

#### 使用

```bash
# 查看服务端的target
[root@rhel2 ~]# iscsiadm -m discovery -t st -p 192.168.143.10
192.168.143.10:3260,1 iqn.2019-11.com.node:control1
# 操作前查看一下当前disk只有/dev/sda
[root@rhel2 ~]# fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c979f

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      616447      307200   83  Linux
/dev/sda2          616448     4810751     2097152   82  Linux swap / Solaris
/dev/sda3         4810752    41943039    18566144   83  Linux

# 登录到这个target上
[root@rhel2 ~]# iscsiadm -m node -T iqn.2019-11.com.node:control1 -l
Logging in to [iface: default, target: iqn.2019-11.com.node:control1, portal: 192.168.143.10,3260] (multiple)
Login to [iface: default, target: iqn.2019-11.com.node:control1, portal: 192.168.143.10,3260] successful.
# 再次查看disk 多了/dev/sdb
[root@rhel2 ~]# fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c979f

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      616447      307200   83  Linux
/dev/sda2          616448     4810751     2097152   82  Linux swap / Solaris
/dev/sda3         4810752    41943039    18566144   83  Linux

Disk /dev/sdb: 104 MB, 104857600 bytes, 204800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes

```

#### 分区、配置挂载

- 分区

```bash
[root@rhel2 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x0eb5d0ea.

Command (m for help): p

Disk /dev/sdb: 104 MB, 104857600 bytes, 204800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes
Disk label type: dos
Disk identifier: 0x0eb5d0ea

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (8192-204799, default 8192): 
Using default value 8192
Last sector, +sectors or +size{K,M,G} (8192-204799, default 204799): 50^H
Value out of range.
Last sector, +sectors or +size{K,M,G} (8192-204799, default 204799): +50M
Partition 1 of type Linux and of size 52 MiB is set

Command (m for help): p

Disk /dev/sdb: 104 MB, 104857600 bytes, 204800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 4194304 bytes
Disk label type: dos
Disk identifier: 0x0eb5d0ea

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            8192      114687       53248   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@rhel2 ~]# partprobe
[root@rhel2 ~]# 
```

- 格式化

```bash
[root@rhel2 ~]# mkfs.ext4 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=4096 blocks
13328 inodes, 53248 blocks
2662 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
1904 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

- 配置自动挂载

```bash
# 查看uuid
[root@rhel2 ~]# blkid /dev/sdb1
/dev/sdb1: UUID="eff8388b-2e93-4501-bf74-60bd870b5192" TYPE="ext4" 
# 编辑/etc/fstab文件
####################一定要加上`,_netdev`##########################
[root@rhel2 ~]# tail -1 /etc/fstab 
UUID=eff8388b-2e93-4501-bf74-60bd870b5192 /allow ext4 defaults,_netdev 0 0 
[root@rhel2 ~]# mkdir /allow
[root@rhel2 ~]# mount -a 
[root@rhel2 ~]# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sda3      xfs        18G  1.4G   17G   8% /
devtmpfs       devtmpfs  909M     0  909M   0% /dev
tmpfs          tmpfs     914M     0  914M   0% /dev/shm
tmpfs          tmpfs     914M  8.6M  905M   1% /run
tmpfs          tmpfs     914M     0  914M   0% /sys/fs/cgroup
/dev/sda1      xfs       297M   85M  213M  29% /boot
/dev/sr0       iso9660   3.5G  3.5G     0 100% /mnt
/dev/sdb1      ext4       47M  1.1M   42M   3% /allow
# 可以重启试一下 是否自动挂载
```



