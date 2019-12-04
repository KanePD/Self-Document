[TOC]

# 红帽学习笔记[RHCSA]第二周

## 环境

1. rhel-server-7.0-x86_64-dvd.iso 镜像
2. VMware-workstation

## 第七课[网络配置相关]

### 在Vmware中添加网卡

`编辑` -> `编辑虚拟网络` -> `添加网络`->`随便选择一个如VMnet2`-> `选择仅主机模式` -> **勾掉**`使用本地DHCP服务将ip分给虚拟机` -> `子网Ip默认就行`

注:

​	1. win10 用户需要管理源权限,否则都是灰的添加不了,点击重启就好

​	2. Wmnet1,WMnet8是WMware默认的网卡

```bash
# 打开 windows CMD
# 会看到如下的网卡信息
C:\Users\kanewang>ipconfig
Ethernet adapter VMware Network Adapter VMnet2:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::bcdf:fdfb:c5b2:884b%34
   IPv4 Address. . . . . . . . . . . : 192.168.94.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

### 将网卡添加到虚拟机上

`右键要添加的虚拟机` - > `设置` -> `网络设配器` -> `自定义(U):特定虚拟网络` - > `选择刚才添加的一块VMnet2` -> `确定`

### 关于网卡命名规则

```bash
# 开头
en+一位字母 -> 以太网网卡
	或
wl+一位字母 -> 无线网网卡
# 一位字母
o 表示 板载
s 表示 热插拔
p 表示 pci插槽
# 用命令查看一下网卡信息 ip addr
# 第一块是环回测试地址 127.0.0.1
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:fb:7d:19 brd ff:ff:ff:ff:ff:ff
```

### 配置网络

```bash
# 查看当前配置 
[root@localhost Desktop]# nmcli connection show
NAME  UUID  TYPE  DEVICE 
# 增加一个配置
[root@localhost Desktop]# nmcli connection add ifname eno16777736 con-name fisrt type ethernet autoconnect yes ip4 192.168.11.2/24 gw4 192.168.94.1 
Connection 'fisrt' (9ca186d9-8fb9-48fa-9973-bb8bdb7d18f0) successfully added.
# 命令解析
        # add  添加
        # ifname  网卡名字  可以tab不全出来
        # con-nam 连接的名字 随意起
        # type    类型ethernet 以太网
        # autoconnect 自动连接
        # ip4 ip地址 与网卡的子网要在一个网段(为了测试ip设置了错误的)
        # gw4 可以没有网关,这里写的是VMnet2 这块网卡的ip
# 启动这个连接
[root@localhost Desktop]# nmcli connection up fisrt 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)
# 再次ip addr查看

[root@localhost Desktop]# ip addr
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:fb:7d:19 brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.2/32 brd 192.168.11.2 scope global eno16777736
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fefb:7d19/64 scope link 
       valid_lft forever preferred_lft forever
# 在windows中 cmd ping刚才的ip 192.168.11.2 
C:\Users\kanewang>ping 192.168.11.2
Pinging 192.168.11.2 with 32 bytes of data:
Request timed out.
# 修改一个连接
	# 将ip改为与网卡一个子网的
[root@localhost Desktop]# nmcli connection modify fisrt ipv4.addresses 192.168.94.200/24
# 重启网络
[root@localhost Desktop]# systemctl restart network
# 再次查看 ip addr
[root@localhost Desktop]# ip addr
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:fb:7d:19 brd ff:ff:ff:ff:ff:ff
    inet 192.168.94.200/24 brd 192.168.52.200 scope global eno16777736
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fefb:7d19/64 scope link 
       valid_lft forever preferred_lft forever
# 再次使用windows cmd ping这个IP
C:\Users\kanewang>ping 192.168.94.200
Pinging 192.168.94.200 with 32 bytes of data:
Reply from 192.168.94.200: bytes=32 time<1ms TTL=64
Reply from 192.168.94.200: bytes=32 time<1ms TTL=64
```

注:上面的操作,可以直接编辑 /etc/sysconfig/network-scripts/ifcfg-xxx文件来达到效果.我们查看一下,我刚才生生的文件.标*的是必要行

```bash
[root@localhost Desktop]# cat /etc/sysconfig/network-scripts/ifcfg-fisrt 
TYPE=Ethernet *
BOOTPROTO=none *
IPADDR0=192.168.94.200*
PREFIX0=24*
DEFROUTE=yes*
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=fisrt
UUID=9ca186d9-8fb9-48fa-9973-bb8bdb7d18f0
DEVICE=eno16777736
ONBOOT=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
```

### 网络配置命令总结

```
# 查看 connection
nmcli connection show
# 增加
nmcli connection add #后面参数上面详细介绍了
# 修改
nmcli connection modify
# 删除
nmcli connection delete
# 开启
nmcli connection up 
```

### 更改hostname

```bash
# 临时更改
[root@localhost Desktop]# hostname test
[root@localhost Desktop]# 
	# 发现并没有改变,重新开一个terminal就能看见变化了
# 永久更改 
hostnamectl set-hostname master
# 更改 /etc/hosts 将域名与ip配置关系
# 更改 /etc/resolv.conf配置dns
```

### 关于SSH的一些配置

配置仅主机模式的网卡后,物理机就可以通过ssh远程访问虚拟机了.我们来做一些配置

1. 不让root用户远程登陆

```bash
# 更改文件/etc/ssh/sshd_config 为no 并打开配置
[root@localhost Desktop]# vi /etc/ssh/sshd_config
PermitRootLogin no
```

2. 客户端提示当前保存的信息与一致的不符

```makdown
删除该用户家目录下的 .ssh/known_hosts文件,重新保存
```

### 远程复制文件 SCP

```bash
# 其实是往自己机器上copy
scp /test/ root@192.168.94.200:/root/test
```

### 关于init（在7中已经不用了）

运行级别

```markdown
# 0 - 停机（千万不能把initdefault 设置为0 ）
# 1 - 单用户模式
# 2 - 多用户，没有 NFS
# 3 - 完全多用户模式(标准的运行级)
# 4 - 没有用到
# 5 - X11 （xwindow)
# 6 - 重新启动 （千万不要把initdefault 设置为6 ） 
```

```bash
init 5 #进入桌面模式
```

## 第八课

### nice值

- 什么是nice值

给进程设置的优先级就是nice。nice的范围是`-20~20`。nice值越小占用的系统资源就越多，就是这个进程不nice。

- 如何查看nice值

```bash
# 使用top命令查看时 会有一列时NI 就是nice值
top - 14:01:36 up 9 min,  2 users,  load average: 0.08, 0.23, 0.18
Tasks: 494 total,   2 running, 492 sleeping,   0 stopped,   0 zombie
%Cpu(s): 16.7 us, 16.7 sy,  0.0 ni, 66.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
  3742 root      20   0  619884  18616  12072 R  60.4  1.0   0:00.85 gnome-terminal-                      
  3778 root      20   0  123924   1940   1152 R  60.4  0.1   0:00.43 top                                  
     1 root      20   0   53692   7660   2524 S   0.0  0.4   0:03.93 systemd                              
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.07 kthreadd                             
     3 root      20   0       0      0      0 S   0.0  0.0   0:00.34 ksoftirqd/0                          
     5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H                         
     7 root      rt   0       0      0      0 S   0.0  0.0   0:00.66 migration/0     
```

- 设置nice值

```bash
# 执行4个 dd命令 一致在吃资源
PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
  3985 root      20   0  107920    624    532 R  95.6  0.0   0:14.21 dd if=/dev/zero of=/dev/null         
  3977 root      20   0  107920    624    532 R  93.7  0.0   0:15.64 dd if=/dev/zero of=/dev/null         
  3981 root      20   0  107920    620    532 R  86.4  0.0   0:14.18 dd if=/dev/zero of=/dev/null         
  3973 root      20   0  107920    624    532 R  84.4  0.0   0:16.26 dd if=/dev/zero of=/dev/null 
# renice 其那两个nice值为-20
renice -19 3977
renice -20 3985
renice +19 3981
renice +19 3973
#再次查看top
   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                              
  3977 root       1 -19  107920    624    532 R 100.0  0.0  10:14.42 dd if=/dev/zero of=/dev/null         
  3985 root       0 -20  107920    624    532 R 100.0  0.0  10:38.96 dd if=/dev/zero of=/dev/null         
  3981 root      39  19  107920    620    532 R  96.5  0.0  10:01.17 dd if=/dev/zero of=/dev/null         
  3973 root      39  19  107920    624    532 R  88.6  0.0   9:55.61 dd if=/dev/zero of=/dev/null   
# 修改后-20的进程一直比19nice值得占用的多，此时如果再开一个进程会更明显，但是虚拟机卡住了，做不了任何操作
```

### 调整时间

- timedatectl

```bash
# 查看
[root@master /]# timedatectl
      Local time: Fri 2019-08-30 14:21:58 CST
  Universal time: Fri 2019-08-30 06:21:58 UTC
        RTC time: Fri 2019-08-30 14:21:58
        Timezone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
# 设置时间
[root@master /]# timedatectl set-time "2019-08-30 14:24:20"
[root@master /]# date
Fri Aug 30 14:24:23 CST 2019
# 查看所有时区
[root@master /]# timedatectl list-timezones 
# 设置时区
[root@master /]# timedatectl set-timezone Asia/Hong_Kong
# 设置ntp同步为yes
timedatectl set-ntp 1
```

- 自动对时

使用的时服务 chronyd。redhat7自带的。

```bash
# 编辑配置文件，注释掉默认的4行，增加阿里云的同步时间配置
[root@master ~]# vi /etc/chrony.conf 
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server ntp6.aliyun.com iburst
[root@localhost ~]# vi /etc/chrony.conf 
[root@localhost ~]# vi /etc/chrony.conf 
[root@localhost ~]# systemctl restart chronyd
[root@localhost ~]# systemctl enable chronyd


```

### 安装软件包

- 什么时RPM：RPM package manager 包管理

- 包的全名：

```markdown
httpd-tools-2.4.6-17.el7.x86_64.rpm
# httpd-tools是名字
# 2.4.6是版本
# 17是发行班号（发行的次数）
# .el7是radhat7平台
# .x86_64处理器cpu的架构
# .noarch是32位64位都可以
```

- 考试提供的iso镜像中有rpm的包。挂载后直接使用

```bash
[root@localhost /]# mount /dev/sr0 /mnt
mount: /dev/sr0 is write-protected, mounting read-only
[root@localhost /]# cd /mnt/Packages/
# packages下面全都是rpm包 
	# -i 安装 -v详细信息 -h 显示进度条
[root@localhost Packages]#  rpm -ivh tree-1.6.0-10.el7.x86_64.rpm 
warning: tree-1.6.0-10.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:tree-1.6.0-10.el7                ################################# [100%]
# 卸载
[root@localhost Packages]# rpm -evh tree
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:tree-1.6.0-10.el7                ################################# [100%]
# 查看包
[root@localhost Packages]# rpm -q tree
tree-1.6.0-10.el7.x86_64
#查看所有包，太多了不直接显示
[root@localhost Packages]# rpm -qa|grep "tree"
tree-1.6.0-10.el7.x86_64
# 查看包的详细
[root@localhost Packages]# rpm -qi tree
Name        : tree
Version     : 1.6.0
Release     : 10.el7
Architecture: x86_64
Install Date: Fri 30 Aug 2019 04:16:26 PM CST
Group       : Applications/File
Size        : 89505
License     : GPLv2+
Signature   : RSA/SHA256, Thu 03 Apr 2014 05:33:48 AM CST, Key ID 199e2f91fd431d51
Source RPM  : tree-1.6.0-10.el7.src.rpm
Build Date  : Tue 28 Jan 2014 01:29:58 AM CST
Build Host  : x86-020.build.eng.bos.redhat.com
Relocations : (not relocatable)
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
Vendor      : Red Hat, Inc.
URL         : http://mama.indstate.edu/users/ice/tree/
Summary     : File system tree viewer
Description :
The tree utility recursively displays the contents of directories in a
tree-like format.  Tree is basically a UNIX port of the DOS tree
utility.
# 查看包装在哪里
[root@localhost Packages]# rpm -ql tree
/usr/bin/tree
/usr/share/doc/tree-1.6.0
/usr/share/doc/tree-1.6.0/LICENSE
/usr/share/doc/tree-1.6.0/README
/usr/share/man/man1/tree.1.gz
# 查看文件从哪个包来的
[root@localhost Packages]# rpm -qf /usr/share/doc/tree-1.6.0
tree-1.6.0-10.el7.x86_64
```

- 关于rpm 包的依赖问题

  - 树形依赖: `a->b->c`直接按照提示从下层一个个装就可以，按照c>b>a的顺序

  - 环形依赖：`a->b->c->a`将三个包名写在一起，直接一次装

  - 模块依赖：用httpd举例

```bash
# 需要一些模块
[root@localhost Packages]# rpm -ivh httpd-2.4.6-17.el7.x86_64.rpm 
warning: httpd-2.4.6-17.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
error: Failed dependencies:
	/etc/mime.types is needed by httpd-2.4.6-17.el7.x86_64
	httpd-tools = 2.4.6-17.el7 is needed by httpd-2.4.6-17.el7.x86_64
	libapr-1.so.0()(64bit) is needed by httpd-2.4.6-17.el7.x86_64
	libaprutil-1.so.0()(64bit) is needed by httpd-2.4.6-17.el7.x86_64
# 访问 http://www.rpmfind.net/ 去查找这个模块在哪个包中，安装那个包
```

### 使用yum 安装软件包

- 什么是yum ： yellow dog update manage黄狗更新管理器

- 使用rpm安装yum

```bash
[root@master Packages]# rpm -ivh yum-3.4.3-118.el7.noarch.rpm 
warning: yum-3.4.3-118.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                          ################################# [100%]
	package yum-3.4.3-118.el7.noarch is already installed
```

- 配置yum源

```bash
###注：考试给一个http的连接给啥写啥###
# 将iso光盘挂在到/mnt目录上
sudo mount -o /dev/sr0 /mnt
# 查看目录挂载
[kane@master mnt]$ df -H
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   19G  3.3G   16G  18% /
devtmpfs               949M     0  949M   0% /dev
tmpfs                  958M  152k  958M   1% /dev/shm
tmpfs                  958M  9.4M  949M   1% /run
tmpfs                  958M     0  958M   0% /sys/fs/cgroup
/dev/sda1              521M  125M  397M  24% /boot
/dev/sr0               3.8G  3.8G     0 100% /mnt
# 我们将iso中的package配置成yum源
sudo yum-config-manager --add-repo=file:///mnt
# 到 yum源目录下
[root@master request-key.d]# cd /etc/yum.repos.d/
# 追加内容gpgcheck=0。让其不检查
[root@master yum.repos.d]# echo "gpgcheck=0" >> ./mnt.repo
# 刷新缓存
[root@master yum.repos.d]# yum makecache

```

- 安装与安装回滚

```bash
# 安装httpd
[root@master yum.repos.d]# yum install httpd
# 查看安装历史
[root@master yum.repos.d]# yum history
Loaded plugins: langpacks, product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
ID     | Login user               | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     5 | kane <kane>              | 2019-10-22 13:45 | Install        |    5      
history list
# 回滚某次安装 使用ID
[root@master yum.repos.d]# yum history undo 5
# 再次查看，会多一个抹去的操作
[root@master yum.repos.d]# yum history
Loaded plugins: langpacks, product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
ID     | Login user               | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     6 | kane <kane>              | 2019-10-22 13:47 | Erase          |    5   
     5 | kane <kane>              | 2019-10-22 13:45 | Install        |    5    
history list
# 此处看不到每个Id都是做什么的，我们可以查看yum安装log
[root@master yum.repos.d]# cat /var/log/yum.log
Oct 22 13:45:09 Installed: apr-1.4.8-3.el7.x86_64
Oct 22 13:45:09 Installed: apr-util-1.5.2-6.el7.x86_64
Oct 22 13:45:09 Installed: httpd-tools-2.4.6-17.el7.x86_64
Oct 22 13:45:09 Installed: mailcap-2.1.41-2.el7.noarch
Oct 22 13:45:11 Installed: httpd-2.4.6-17.el7.x86_64
Oct 22 13:47:04 Erased: httpd-2.4.6-17.el7.x86_64
Oct 22 13:47:04 Erased: httpd-tools-2.4.6-17.el7.x86_64
Oct 22 13:47:04 Erased: mailcap-2.1.41-2.el7.noarch
Oct 22 13:47:04 Erased: apr-util-1.5.2-6.el7.x86_64
Oct 22 13:47:05 Erased: apr-1.4.8-3.el7.x86_64
```

- 软件包组

```bash
[root@master yum.repos.d]# yum groups list
Loaded plugins: langpacks, product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
There is no installed groups file.
Maybe run: yum groups mark convert (see man yum)
Available environment groups:
   Minimal Install
   Infrastructure Server
   File and Print Server
   Basic Web Server
   Virtualization Host
   Server with GUI
Available Groups:
   Compatibility Libraries
   Console Internet Tools
   Development Tools
   Graphical Administration Tools
   Legacy UNIX Compatibility
   Scientific Support
   Security Tools
   Smart Card Support
   System Administration Tools
   System Management
Done
```

## 第九课

### 文件归档

- tar是什么

通过`tar`命令可以将大型文件汇集成一个文件（归档），注意没有压缩功能。

- 压缩方式

`gzip` 

通过gzip过滤文档，使用最广泛

`bzip2`

通常比gzip压缩小，但是不如gzip广泛

 `xz`

比较新，压缩率比较高

- tar命令参数介绍

```bash
-A 追加文件至归档
-c 创建一个新的归档
-v 列出处理文件的过程
-f 要操作的归档的名字
-t 列出归档内容
-x 从归档文件中解压出内容
-a 使用文档后缀名来决定压缩程序
-j bzip2 压缩方式
-J xz压缩方式
-z gzip压缩方式
```

- tar命令操作

```bash
# 创建tar
[root@master kane]# tar cf 1.tar ./tar_test/
# 查看tar内容
[root@master kane]# tar -tf 1.tar 
./tar_test/
./tar_test/11.q
./tar_test/22.q
./tar_test/44.q
./tar_test/33.q
# 解压出来
[root@master test]# ls
1.tar
[root@master test]# tar -xvf 1.tar 
./tar_test/
./tar_test/11.q
./tar_test/22.q
./tar_test/44.q
./tar_test/33.q
[root@master test]# ls
1.tar  tar_test
# 创建一个使用gzip压缩的文件
[root@master kane]# tar -zcf 2.tar ./tar_test/
# 比较一下 1.tar 与 2.tar 。发现2.tar确实小了很多
[root@master test]# ll
total 16
-rw-r--r--. 1 root root 10240 Oct 22 14:57 1.tar
-rw-r--r--. 1 root root   178 Oct 22 15:01 2.tar
drwxr-xr-x. 2 root root    50 Oct 22 14:57 tar_test
```

### 硬盘

- 分类

```
/dev/sd      表示串口设备
/dev/hd      表示并口设备（现在很难找并口设备）
/dev/vd    虚拟出来的
```

- 硬盘设备的命名规则

```bash
# 按设备
第一块  /dev/sda
第二款  /dev/sdb
# 按分区
第一个分区 /dev/sda1
第二个分区 /dev/sda2
```

- 硬盘结构

```bash
# 物理结构
机械硬盘与固态硬盘(寿命不如机械硬盘)
# 逻辑结构
1. 磁道：以盘片为中心，半径大小不同的同心圆组成的环形区域。
2. 扇区：磁道上的一段圆弧
3. 柱面：虚的，类似于筒壁
```

### 分区创建、使用

- 为什么要分区

```markdown
1. 便于管理
2. 不同类型的数据分开放
3. 分隔操作系统
4. 初始化硬盘
```

- 分区的类型

```markdown
1. 主分区：
   安装操作系统，启动和引导操作系统。最多只能有`4`个主分区。MBR(Master Boot Recorder) 主要开机扇区，放置硬盘的信息。一共512B，前面446B字节是程序，`64B`表示分区每个分区是16B。所以一共只能有4个主分区。最后两个字节是保留位。
2. 扩展分区：
   为了解除只有4个主分区的限制就有了扩展分区。但是扩展分区占用主分区的编号，最多只能有一个扩展分区。可以在扩展分区上创建逻辑分区供我们来使用
3. 逻辑分区：
   在扩展分区上创建逻辑分区来使用。
4. 注意：
   分区不是无限的，对于串口设备，最多15个。`1~4`主分区或扩展分区`5-15`逻辑分区。即使主分区只用了一个分区号，逻辑分区也不能使用前面四个分区号。
```

- 创建分区的步骤

```markdown
1. 进入到分区操作
2. 通知内核读取分区表
3. 创建文件系统即格式化
4. 挂载使用
```

- 创建`主分区` 操作

```markdown
1. 操作在虚拟机上操作，需要添加一块硬盘给虚拟机。
2. 查看硬盘情况
[root@localhost Desktop]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0  500M  0 part /boot
└─sda2          8:2    0 19.5G  0 part 
  ├─rhel-root 253:0    0 17.5G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0   20G  0 disk 
sr0            11:0    1 1024M  0 rom 
3. 使用fdisk 进入到 sda硬盘的分区
[root@master Desktop]# fdisk /dev/sda
4. 查看分区（此时还没有分区）使用命令 p
命令(输入 m 获取帮助)：p

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0000220c

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    41943039    20458496   8e  Linux LVM
5. 使用fdisk进入到sdb开始分区
[root@localhost Desktop]# fdisk /dev/sdb
6. 创建一个主分区
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-41943039, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): +1000M
Partition 1 of type Linux and of size 1000 MiB is set

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xb43d3ed1

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2050047     1024000   83  Linux
7. 注：关于扇区大小，精确计算扇区位置比较复杂直接使用 +{K，M，G}比较方便
8. 保存 w 自动退出
9. 通知内核
[root@localhost Desktop]# partprobe
10. 创建文件系统即格式化
[root@localhost Desktop]# mkfs.ext4 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
64000 inodes, 256000 blocks
12800 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=262144000
8 block groups
32768 blocks per group, 32768 fragments per group
8000 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
11. 挂在到目录上
[root@localhost Desktop]# mount /dev/sdb1 /first
12. 查看分区
[root@localhost Desktop]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0  500M  0 part /boot
└─sda2          8:2    0 19.5G  0 part 
  ├─rhel-root 253:0    0 17.5G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0   20G  0 disk 
└─sdb1          8:17   0 1000M  0 part /first
sr0            11:0    1 1024M  0 rom 
```

- 创建 `扩展分区` `逻辑分区`

```markdown
1. 直接进入sdb
[root@localhost Desktop]# fdisk /dev/sdb
2. 创建扩展分区，创建了10个G
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): e
Partition number (2-4, default 2): 
First sector (2050048-41943039, default 2050048): 
Using default value 2050048
Last sector, +sectors or +size{K,M,G} (2050048-41943039, default 41943039): +10G
Partition 2 of type Extended and of size 10 GiB is set

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xb43d3ed1

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2050047     1024000   83  Linux
/dev/sdb2         2050048    23021567    10485760    5  Extended
3. 创建逻辑分区
Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (2052096-23021567, default 2052096): 
Using default value 2052096
Last sector, +sectors or +size{K,M,G} (2052096-23021567, default 23021567): +1G
Partition 5 of type Linux and of size 1 GiB is set

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xb43d3ed1

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2050047     1024000   83  Linux
/dev/sdb2         2050048    23021567    10485760    5  Extended
/dev/sdb5         2052096     4149247     1048576   83  Linux
4. 保存-通知内核-格式化-挂载
[root@localhost Desktop]# partprobe
[root@localhost Desktop]# mkfs.ext4 /dev/sdb5
[root@localhost Desktop]# mount /dev/sdb5 /second
5. 查看分区
[root@localhost Desktop]# 
[root@localhost Desktop]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0  500M  0 part /boot
└─sda2          8:2    0 19.5G  0 part 
  ├─rhel-root 253:0    0 17.5G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0   20G  0 disk 
├─sdb1          8:17   0 1000M  0 part /first
├─sdb2          8:18   0    1K  0 part 
└─sdb5          8:21   0    1G  0 part /second
sr0            11:0    1 1024M  0 rom  
6. 查看文件系统
[root@localhost Desktop]# df -H
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   19G  3.2G   16G  17% /
devtmpfs               949M     0  949M   0% /dev
tmpfs                  958M  144k  958M   1% /dev/shm
tmpfs                  958M  9.3M  949M   1% /run
tmpfs                  958M     0  958M   0% /sys/fs/cgroup
/dev/sda1              521M  125M  397M  24% /boot
/dev/sdb1              1.1G  2.6M  944M   1% /first
/dev/sdb5              1.1G  2.7M  951M   1% /second
```

### 分区自动挂载

- 查看文件`/etc/fstab`

```bash
[root@localhost Desktop]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Aug 15 20:39:14 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root   /                       xfs     defaults        1 1
UUID=a0d45f55-dcf8-4d8e-b32d-4d1eb66a07bc /boot                   xfs     defaults        1 2
/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
```

- 自动挂载的三种方式

```markdown
1. 使用UUID
2. 使用label
3. 使用设备名（不建议，因为设备名有可能会更换）
```

- UUID

```markdown
1. 查看UUID
[root@localhost Desktop]# blkid /dev/sdb5
/dev/sdb5: UUID="1eb4fbaa-5744-48e1-809d-90181843eddb" TYPE="ext4" 
/* 在没创建文件系统的时候是没有UUID的*/
2. 编辑/etc/fstab文件
UUID=1eb4fbaa-5744-48e1-809d-90181843eddb /third xfs defaults 0 0 
设备或分区或镜像名 挂载点 文件系统类型 挂载选项 是否备份 是否开机检测
3. 重启虚拟机
4. 查看目录情况，可以看到已经自动挂载上了
[root@localhost Desktop]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   18G  3.0G   15G  17% /
devtmpfs               905M     0  905M   0% /dev
tmpfs                  914M   84K  914M   1% /dev/shm
tmpfs                  914M  8.9M  905M   1% /run
tmpfs                  914M     0  914M   0% /sys/fs/cgroup
/dev/sdb5              190M  1.6M  175M   1% /third
/dev/sda1              497M  119M  379M  24% /boot
```

- 标签

```markdown
1. 重新创建一个逻辑分区db6
1. 查看分区标签（默认为空）
[root@localhost Desktop]# e2label /dev/sdb6

2. 设置一个标签
[root@localhost Desktop]# e2label /dev/sdb6 game
[root@localhost Desktop]# e2label /dev/sdb6
game
3. 编辑 /etc/fstab 添加以标签为开始的自动挂载
LABEL=game /forth ext4 defaults 0 0
4. 重启后查看文件系统
[root@localhost Desktop]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   18G  3.0G   15G  17% /
devtmpfs               905M     0  905M   0% /dev
tmpfs                  914M  152K  914M   1% /dev/shm
tmpfs                  914M  8.9M  905M   1% /run
tmpfs                  914M     0  914M   0% /sys/fs/cgroup
/dev/sdb6              190M  1.6M  175M   1% /forth
/dev/sdb5              190M  1.6M  175M   1% /third
/dev/sda1              497M  119M  379M  24% /boot
```

**注：上述编辑完 `/etc/fstab`后先`mount -a`测试一下是否正确，否则可能导致虚拟机启动不了。**

### Swap虚拟内存

- 虚拟内存是一种计算机内存管理的一种结束，可以把不常用的数据缓存到硬盘上。
- 两种方式

```markdown
1. 交换分区
2. 交换文件
```

- 查看虚拟内存

```bash
[root@localhost Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       809M       1.0G       9.6M       912K       268M
-/+ buffers/cache:       539M       1.3G
Swap:         2.0G         0B       2.0G
```

- 使用文件作为Swap

```markdown
1. 创建文件/swap/swapfile，使用dd用指定大小的块拷贝到问价中
[root@localhost Desktop]# dd if=/dev/zero of=/swap/swapfile bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 7.59467 s, 141 MB/s
2. 将文件转成swap
root@localhost Desktop]# mkswap /swap/swapfile
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=7df01b33-7874-4ae9-b275-1306427bcf82
3. 激活swap
[root@localhost Desktop]# swapon /swap/swapfile
swapon: /swap/swapfile: insecure permissions 0644, 0600 suggested.
4. 查看发现多了1个G
[root@localhost Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       1.7G        77M       9.7M       168K       1.0G
-/+ buffers/cache:       711M       1.1G
Swap:         3.0G         0B       3.0G
5. 配置开机生效
vi /etc/fstab
/swap/swapfile swap swap defaults 0 0
6. 关闭swap
[root@localhost Desktop]# swapoff /swap/swapfile
[root@localhost Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       1.7G        75M       9.8M       168K       1.0G
-/+ buffers/cache:       711M       1.1G
Swap:         2.0G         0B       2.0G
7. 重启一次看看开机是否挂载上
root@localhost Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       818M       1.0G       9.8M       916K       266M
-/+ buffers/cache:       551M       1.2G
Swap:         3.0G         0B       3.0G
```

- 使用分区作为swap

```markdown
1. 创建一个 1G swap 分区
[root@localhost Desktop]# fdisk /dev/sdb
Command (m for help): n
Partition type:
   p   primary (0 primary, 1 extended, 3 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 8
First sector (2926592-20973567, default 2926592): 
Using default value 2926592
Last sector, +sectors or +size{K,M,G} (2926592-20973567, default 20973567): +1G
Partition 8 of type Linux and of size 1 GiB is set
Command (m for help): t
Partition number (1,5-8, default 8): 8
Hex code (type L to list all codes): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'

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
/dev/sdb7          827392     2924543     1048576   82  Linux swap / Solaris
/dev/sdb8         2926592     5023743     1048576   82  Linux swap / Solaris
/* 创建后需要用t命令去修改分区类型为 82（ Linux swap / Solaris ） */
2. 保存退出后通知内核
3. 格式化成 交换分区 
[root@localhost Desktop]# mkswap /dev/sdb8
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=0cf98e08-cfe7-4818-8a39-2c51987c0677
4. 激活后查看又多了1G
[root@localhost Desktop]# swapon /dev/sdb8
[root@localhost Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       830M       996M       9.8M       1.7M       273M
-/+ buffers/cache:       555M       1.2G
Swap:         4.0G         0B       4.0G
```

### 软连接与硬链接

`硬链接`：基本不用，硬链接与cp类似，但是硬链接不占用内存。硬连接不更新文件时间。创建时不能跨分区

`软连接`：符号连接，类似windows快捷方式，占用i接待你，创建的时候没有其他限制

````bash
touch testfile
#创建硬链接
ln testfile testfile.hard
# 创建软连接
ln -s testfile testfile.hard


# 硬链接在原来的文件删除时还是好用的，软连接不好用
# 开机自启其实就是创建软连接
````

## 第十课

### 计划任务[At & Cron Jobs]

- `at`

```bash
# at 命令只能计划一次性任务但是比较方便。
#  先输入时间
[root@localhost Desktop]# at 10:02
#  输入要做的事情
at> echo 222 > test.log
# CTRL + D 退出 
at> <EOT>
job 2 at Fri Oct 25 10:02:00 2019
#  查看计划中的任务
[root@localhost Desktop]# atq
2	Fri Oct 25 10:02:00 2019 a root
#  时间过去后查看当前路径下 显示 10:02分创建了test.log
[root@localhost Desktop]# ll
total 4
-rw-r--r--. 1 root root 4 Oct 25 10:02 test.log
#  查看内容
[root@localhost Desktop]# cat test.log 
222
```

- `crond`周期性计划任务命令

```bash
crontab -e         # 进入编辑当前用户的计划任务
crontab -e -u kane # 为别的用户设置计划任务
crontab -l         # 查看自己计划任务
crontab -l -u kane # 查看别的用户的计划任务
```

- 创建计划任务

```bash
# 创建一个每隔1分钟echo一个内容并追加到一个文件中
[root@localhost Desktop]# crontab -e
*/1 * * * * /bin/date >> test2.log
# 查看一下
[root@localhost Desktop]# crontab -l
*/1 * * * * /bin/date >> /root/Desktop/test2.log
# 查看一下结果
Fri Oct 25 13:53:01 CST 2019
Fri Oct 25 13:54:01 CST 2019
Fri Oct 25 13:55:01 CST 2019
Fri Oct 25 13:56:02 CST 2019
Fri Oct 25 13:57:01 CST 2019
Fri Oct 25 13:58:01 CST 2019
Fri Oct 25 13:59:01 CST 2019
Fri Oct 25 14:00:01 CST 2019
Fri Oct 25 14:01:01 CST 2019
Fri Oct 25 14:02:01 CST 2019
Fri Oct 25 14:03:01 CST 2019
Fri Oct 25 14:04:02 CST 2019
Fri Oct 25 14:05:01 CST 2019
Fri Oct 25 14:06:01 CST 2019
Fri Oct 25 14:07:01 CST 2019
Fri Oct 25 14:08:01 CST 2019
Fri Oct 25 14:09:01 CST 2019
Fri Oct 25 14:10:01 CST 2019
Fri Oct 25 14:11:01 CST 2019
Fri Oct 25 14:12:01 CST 2019
Fri Oct 25 14:13:01 CST 2019
Fri Oct 25 14:14:01 CST 2019
Fri Oct 25 14:15:01 CST 2019
Fri Oct 25 14:16:01 CST 2019
Fri Oct 25 14:17:01 CST 2019
```

- 计划任务格式介绍

```bash
# 基本介绍
┌───────────── 分钟 (0 - 59)
│ ┌───────────── 小时 (0 - 23)
│ │ ┌───────────── 月中的天 (1 - 31)
│ │ │ ┌───────────── 月份 (1 - 12)
│ │ │ │ ┌───────────── 周中的天 (0 - 6)[0是周日]
│ │ │ │ │   ┌──────────── 命令[注意：bin下保存命令最好用绝对路径.eg:/bin/date]
| | | | |   |
* * * * *  /bin/date > date.log

# 特殊字符介绍
* 代表所有有效字符 
- 代表一个范围(eg:1-4 代表 1,2,3,4)
, 代表一些列的指定值(eg:1,2,3)
/ 代表间隔频率(eg:*/1 * * * * 就是每分钟)(eg:* */2 * * * 就是每两个小时)
```

- 计划任务黑白名单

```bash
# 黑名单
vi /etc/cron.deny 
# 白名单
vi /etc/cron.allow
# 经过测试白名单的优先级比黑名单高
```

### 逻辑卷管理

- 逻辑卷是对磁盘空间的划分和管理，创建过程是一个先整合在做划分的一个过程
- 概念

```markdown
1. 物理卷
指硬盘分区，也可以是整个硬盘或已创建的软RAID，是LVM的基本存储设备。
2. 卷组
是由一个或多个物理卷所组成的存储池，在卷组上能创建一个或多个逻辑卷。
3. 逻辑卷
类似于非LVM系统中的硬盘分区，它建立在卷组之上，是一个标准的块设备，在逻辑卷之上可以建立文件系统。
```

- 创建流程

```markdown
1. 创建物理卷	pvcreate
2. 创建卷组   	 vgcreate
3. 创建逻辑卷     lvcreate
4. 创建文件系统（格式化）
5. 挂载使用
```

- 创建

```bash
# 创建 物理卷
[root@localhost /]# pvcreate /dev/sdb5
  Wiping ext4 signature on /dev/sdb5.
  Physical volume "/dev/sdb5" successfully created
# 创建卷组
[root@localhost /]# vgcreate vg1 /dev/sdb5
  Volume group "vg1" successfully created
# 创建逻辑卷
[root@localhost /]# lvcreate -L 100M -n lv1 vg1
  Logical volume "lv1" created
# 查看物理卷
[root@localhost /]# pvs
  PV         VG   Fmt  Attr PSize   PFree 
  /dev/sda2  rhel lvm2 a--   19.51g     0 
  /dev/sdb5  vg1  lvm2 a--  196.00m 96.00m
# 查看卷组
[root@localhost /]# vgs
  VG   #PV #LV #SN Attr   VSize   VFree 
  rhel   1   2   0 wz--n-  19.51g     0 
  vg1    1   1   0 wz--n- 196.00m 96.00m
# 查看逻辑卷
[root@localhost /]# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  root rhel -wi-ao----  17.51g                                             
  swap rhel -wi-ao----   2.00g                                             
  lv1  vg1  -wi-a----- 100.00m 
# 挂载后就可以使用了
[root@localhost /]# mount /dev/vg1/lv1 /fifth
```

- 管理卷组

```bash
# 扩展卷组，将物理卷/dev/sdb6同样加到vg1这个卷组里
[root@localhost /]# vgextend vg1 /dev/sdb6
  Volume group "vg1" successfully extended
# 查看，又多了200M 来自/dev/sdb6
[root@localhost /]# vgs
  VG   #PV #LV #SN Attr   VSize   VFree  
  rhel   1   2   0 wz--n-  19.51g      0 
  vg1    2   1   0 wz--n- 392.00m 292.00m
# --------------------------
# lv的操作需要制定到路径 /dev/vg1/lv1
# 扩展逻辑卷，扩展大小
[root@localhost /]# lvextend -L +50M  /dev/vg1/lv1 
  Rounding size to boundary between physical extents: 52.00 MiB
  Extending logical volume lv1 to 152.00 MiB
  Logical volume lv1 successfully resized
# 查看逻辑卷
[root@localhost /]# lvs
  LV   VG   Attr       LSize   Pool Origin Data%  Move Log Cpy%Sync Convert
  root rhel -wi-ao----  17.51g                                             
  swap rhel -wi-ao----   2.00g                                             
  lv1  vg1  -wi-a----- 152.00m  
# 减少到 40M
[root@localhost /]# lvreduce -L 40M /dev/vg1/lv1
  WARNING: Reducing active logical volume to 40.00 MiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce lv1? [y/n]: y
  Reducing logical volume lv1 to 40.00 MiB
  Logical volume lv1 successfully resized
[root@localhost /]# lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- 17.51g                                             
  swap rhel -wi-ao----  2.00g                                             
  lv1  vg1  -wi-a----- 40.00m
# 减少vg
[root@localhost /]# vgreduce vg1 /dev/sdb6
  Removed "/dev/sdb6" from volume group "vg1"
[root@localhost /]# vgs
  VG   #PV #LV #SN Attr   VSize   VFree  
  rhel   1   2   0 wz--n-  19.51g      0 
  vg1    1   1   0 wz--n- 196.00m 156.00m
```

- 删除卷

```markdown
1. 先卸载挂载
[root@localhost /]# umount /dev/vg1/lv1
2. 删除LV
[root@localhost /]# lvremove /dev/vg1/lv1
Do you really want to remove active logical volume lv1? [y/n]: y
  Logical volume "lv1" successfully removed
3. 删除VG
[root@localhost /]# vgremove vg1
  Volume group "vg1" successfully removed
4. 删除PV
[root@localhost /]# pvremove /dev/sdb6
  Labels on physical volume "/dev/sdb6" successfully wiped
```

- 按需挂载`autofs`

```bash
# yum安装autofs
[root@localhost /]# yum install autofs
# 编辑 文件 /etc/auto.master
/misc	/etc/auto.misc
/test /etc/test.autofs --timeout 10 # 增加行,10秒后自动卸载
挂载目录    按照配置文件
# 编辑挂载配置文件
[root@localhost etc]# vi /etc/test.autofs 
test -fstype=ext4,rw :/dev/sdb6
# 重启autofs
[root@localhost etc]# systemctl restart autofs
# 查看当前挂载情况
[root@localhost etc]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   18G  3.0G   15G  17% /
devtmpfs               905M     0  905M   0% /dev
tmpfs                  914M  140K  914M   1% /dev/shm
tmpfs                  914M  8.9M  905M   1% /run
tmpfs                  914M     0  914M   0% /sys/fs/cgroup
/dev/sda1              497M  119M  379M  24% /boot
/dev/sdb5              190M  1.6M  175M   1% /third
/dev/sr0               3.5G  3.5G     0 100% /mnt
# 访问目录 /test/test
[root@localhost etc]# cd /test/test
# 再次查看自动挂载上了
[root@localhost test]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   18G  3.0G   15G  17% /
devtmpfs               905M     0  905M   0% /dev
tmpfs                  914M  140K  914M   1% /dev/shm
tmpfs                  914M  8.9M  905M   1% /run
tmpfs                  914M     0  914M   0% /sys/fs/cgroup
/dev/sda1              497M  119M  379M  24% /boot
/dev/sdb5              190M  1.6M  175M   1% /third
/dev/sr0               3.5G  3.5G     0 100% /mnt
/dev/sdb6              190M  1.6M  175M   1% /test/test

```

## 十一课

### 文件特殊权限`facl`

- 查看权限

```bash
[root@localhost test]# getfacl lost+found/
# file: lost+found/
# owner: root
# group: root
user::rwx           # 代表用户拥有读、写、执行权限
group::---			# 代表组没有任何权限
other::---			# 代表其他没有任何权限
```

- 编辑权限

```bash
# 查看kane用户 对文件夹权限
[kane@localhost test]$ cd lost+found/
bash: cd: lost+found/: Permission denied
# 给文件夹lost+found/ kane用户加上rwx的权限
[root@localhost test]# setfacl -m u:kane:rwx lost+found/
# 先查看一下特殊权限
[root@localhost test]# getfacl lost+found/
# file: lost+found/
# owner: root
# group: root
user::rwx
user:kane:rwx
group::---
mask::r--
other::---
# 用kane用户再次访问
bash: cd: lost+found/: Permission denied
[kane@localhost test]$ cd lost+found/
[kane@localhost lost+found]$ 
# 修改组就是
[root@localhost test]# setfacl -m g:kane:rwx lost+found/
# 修改其他
[root@localhost test]# setfacl -m o:rwx lost+found/

```

### Selinux

加强的安全Linux体系

```bash
[root@localhost test]# setenforce 0 # 临时修改
[root@localhost test]# getenforce
Permissive
# 每个参数的意义
enforcing[1] 严格模式。selinux全部开启，只有匹配的才会允许，并记录到日志当中
permissive[0] 非严格模式。selinux部分开启，所有允许访问，不配的时候，会记录到日志
disabled  禁用
# 永久修改。需要重启
[root@localhost test]# vi /etc/selinux/config 
SELINUX=disabled
```





