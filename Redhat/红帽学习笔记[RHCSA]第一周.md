[TOC]

# 红帽学习笔记[RHCSA]第一周

## 环境

1. rhel-server-7.0-x86_64-dvd.iso 镜像
2. VMware-workstation

## 第一课

### 关于Shell

- Shell是什么

Shell是系统的用户界面，提供了用户与内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核中执行。

- bash shell是大多数Linux的缺省shell
- 交互方式使用shell会显示一个字符

```bash
# root用户是 #
[root@c2c9702c7e20 /]#
# 普通用户是 $
[test@c2c9702c7e20 /]$ 
```

- 使用shell 需要一个终端：虚拟控制台

```bash
# 如果是图形化界面的
[CTRL+ALT]+F1 是图形界面
[CTRL+ALT]F2-F6是5个红帽的虚拟控制台
#如果是关闭图形化界面
[CTRL+ALT] F1-F5是5个红帽的虚拟控制台
```

### 命令的基础知识

- 组成

```bash
#命令由三部分组成
命令：需要运行
选项：用于调整命令的行为
参数：通常是命令的目标
```

### 在终端中敲命令的快捷键

```
	Ctrl+a  ： 跳到命令行头部
​	Ctrl+e  ：跳到命令结尾
​	Ctrl+u  ：删除到光标前的内容
​	Ctrl+k  ：删除光标之后的内容
​	Ctrl+l ： 删除面板上的内容
​	Ctrl+r ： 查找 history中内容
```

### 本次课程涉及的命令

#### `ls `查看当前目录下的文件以及文件夹的属性。

```bash
ls
#查看当前文件夹下的n
[root@localhost /]# ls test
11.txt  22.txt  33  44
# ----------------------------
ls -d test/
# -d的意义会查看后面这个test目录，如果不加，ls会查看test里面的内容
[root@localhost /]# ls -d test
test
# ----------------------------
ls -l 等价于 ll
# -l的意义是长格式查看，会显示权限、所属者、大小等信息
[root@localhost /]# ls -l test
total 0
-rw-r--r--. 1 root root 0 Aug 25 11:21 11.txt
-rw-r--r--. 1 root root 0 Aug 25 11:22 22.txt
drwxr-xr-x. 2 root root 6 Aug 25 11:22 33
drwxr-xr-x. 2 root root 6 Aug 25 11:22 44
# ----------------------------
ls -a 
# -a会显示隐藏文件
[root@localhost /]# ls -a test
.  ..  11.txt  22.txt  33  44
# ----------------------------
ls -R 
# 递归显示目录
[root@localhost /]# ls -R test
test:
11.txt  22.txt  33  44

test/33:

test/44:
```

#### `mkdir`创建文件夹

```bash
mkdir -p /test/11/22/33
# -p参数会根据需要创建父级文件夹。以上，如果不-p的话 由于11文件夹不存在创建会失败
```

#### `passwd`修改密码

```bash
[root@localhost /]# passwd
Changing password for user root.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

#### `date`查看日期

```bash
# 查看当天
[kane@localhost /]$ date
Mon Aug 26 15:01:53 CST 2019
# 查看未来45天
[kane@localhost /]$ date -d "+45days"
Thu Oct 10 15:01:59 CST 2019
```

#### `file`扫描内容的开头，显示文件类型。

```bash
[root@localhost /]# file test
test: directory
[root@localhost /]# file test/11.txt 
test/11.txt: ASCII text, with very long lines
```

#### `head`用来显示文件的开头
默认前10行。可以通过-n 设定显示行数。也可以直接-number
```bash
[root@localhost test]# head 22.txt 
1
2
3
4
5
6
7
8
9
10
[root@localhost test]# head -1 22.txt 
1
[root@localhost test]# head -n 2 22.txt 
1
2
```

#### `tail`显示文件末尾

默认显示前10行。其他的与head类似，除了`-f`

```bash
[root@localhost test]# tail -f 22.txt 
6
7
8
9
10
11
12
13
14
#除了显示尾10行之外，还会监测22.txt文件的变化并输出到屏幕上。
```

#### `wc`计算文件见行数、字数、字符数

```bash
[root@localhost test]# wc 22.txt 
15 14 34 22.txt
#行 字 字符 文件   
#-l 单独行数 -w单独字数 -c单独字符数
```

#### `history`与`!`历史记录，快速使用历史命令

```bash
[root@localhost test]# history
  225  head 22.txt 
  226  head -1 22.txt 
  227  head -n 2 22.txt 
  228  head --help
  229  tail -f 22.txt 
  230  wc 22.txt 
  231  wc -l
  232  wc -l 22.txt 
  233  wc -w 22.txt 
  234  wc -c 22.txt 
  235  history
# 使用编号为234的历史命令
[root@localhost test]# !234
wc -c 22.txt 
34 22.txt
# 使用首字母是wc的最后一个命令
[root@localhost test]# !wc
wc -c 22.txt 
34 22.txt
# 清除历史记录
history -c   # 重启后历史记录还在
# 删除某个编号的历史记录
history -d 234
```

## 第二课

### 常用的目录结构与用途

```bash
/  根目录
/boot 存储的是系统起动时的信息和内核等
/dev  存储的是设备文件
/etc 存储的是系统的配置文件
/root 存储的是root用户的家目录
/home 存储的是普通用户的家目录
/mnt  存储的是自定义的挂在光盘或U盘
/var    存储的系统里经常变化的信息，比如日志
/tmp  存储的是临时文件
/bin    出处的是系统里的基本命令。普通用户就可以执行
/sbin   存储的是管理类的命令，root用户执行
/usr    存储的是应用程序文件，占用磁盘最大的
/proc    伪文件系统，几乎里面的东西全是内存中的数据，基本不占用磁盘空间。
```

### 本次课程涉及到的命令

#### `cd`切换目录

```bash
# cd 什么都不加会切换到当前用户的家目录
[root@localhost /]# cd 
[root@localhost ~]# 
# cd ~ 同样的操作
[root@localhost /]# cd ~
[root@localhost ~]# 
# cd -切换到上一次的目录
[root@localhost ~]# cd -
/
[root@localhost /]# 
# cd . 当前目录
[root@localhost /]# cd .
[root@localhost /]# 
# cd ..切换到上一级目录
[root@localhost test]# cd ..
[root@localhost /]# 
```

#### `pwd`显示当前所在目录

```bash
[root@localhost test]# pwd
/test
[root@localhost test]# 
```

#### `$LANG`临时切换语言环境

```bash
[root@localhost test]# LANG=zh_CN.UTF8
[root@localhost test]# aa
bash: aa: 未找到命令...
[root@localhost test]# LANG=zh_EN.UTF8
[root@localhost test]# aa
bash: aa: command not found...
```

#### `touch`更新文件时间戳。
将文件的时间戳更新为当前的日期与时间，如果文件不出存在会创建文件
```bash
[root@localhost test]# touch 33.txt
[root@localhost test]# 
[root@localhost test]# ls
11.txt  22.txt  33  33.txt  44
# 可以同时touch多个文件
[root@localhost test]# touch 11.txt 22.txt 33.txt 44.txt
[root@localhost test]# ls
11.txt  22.txt  33.txt  44.txt  55
# 关于同时创建多个文件的写法
# 		创建100个文件 test1.mul  到 test100.mul
[root@localhost test]# touch test{1..100}.mul
# 		创建26个文件 testa.mul 到 testz.mul 
[root@localhost test]# touch test{a..z}.mul
# 		创建两个文件 test_i.mul  test_y.mul
[root@localhost test]# touch test{i,y}.nul
# ------------------------------------------------
# {}可以并列写多个，可以嵌套写，{..}表示一连串{,}表示并列。
```

#### `cp`复制文件或者目录

```bash
[root@localhost test]# cp 11.txt /test1
[root@localhost test]# ls test1
ls: cannot access test1: No such file or directory
[root@localhost test]# ls /test1
11.txt
```

#### `mv`移动文件，或重命名文件

```bash
# test文件夹下
[root@localhost test]# ls
11.txt  22.txt  33  33.txt  44
# 将22.txt移动到 /test1下
[root@localhost test]# mv 22.txt /test1
# 将33.txt重命名为44.txt
[root@localhost test]# mv 33.txt 44.txt
#查看test下结果
[root@localhost test]# ls
11.txt  33  44  44.txt
#查看test1下结果
[root@localhost test]# ls /test1
11.txt  22.txt
```

#### `rm`删除文件或文件夹

```bash
# 查看test内容
[root@localhost test]# ls
11.txt  33  44  44.txt
# 删除11.txt 提示是否确认删除
[root@localhost test]# rm 11.txt 
rm: remove regular file '11.txt'? y
# -f 强制删除44.txt，不提示是否删除
[root@localhost test]# rm -f 44.txt 
# 删除33文件夹失败
[root@localhost test]# rm -f 33
rm: cannot remove '33': Is a directory
# -r 删除文件夹
[root@localhost test]# rm -rf 33
[root@localhost test]# ls
44
```

#### `rmdir`删除空的文件夹

```bash
# 递归显示55文件夹下有文件 44是空的文件夹
[root@localhost test]# ls -R 
.:
44  55

./44:

./55:
11.txt
# 删除空的44
[root@localhost test]# rmdir 44
# 删除非空55失败
[root@localhost test]# rmdir 55
rmdir: failed to remove '55': Directory not empty
```

#### `ESC+.`使用上一个命令的最后一个参数

这是一个快捷键，使用是一条命令的最后一个参数

```bash
[root@localhost test]# ls 55
11.txt
# 下面这个 55 是通过ESC+.按出来的
[root@localhost test]# ls 55
```

#### `clear`清空当前屏幕，但是不是真正的删除

```bash
# CTRL + L相同的作用
```

#### `man` 和 `firefox file:///usr/share/doc/`

查看帮助

```bash
# 查看 ls 命令的帮助
[root@localhost test]# man ls
# 用 火狐打开帮助文档
firefox file:///usr/share/doc/
```

## 第三课

### 关于Linux的输入输出

- 输入输出

```bash
0  stdin  		标准输入    仅读取 
1  stdout 		标准输出    仅写入
2  stderr 		标准错误    仅写入
3  filename  	其他文件    读取和/或写入
```

- 输出重定向

```bash
# > file 或 1 > file 标准输出重定向到file并覆盖文件
[root@localhost test]# ls > stdout
[root@localhost test]# cat stdout 
11.txt
22.txt
stdout
# >> file 或 1 >> file 标准输出重定向到file追加到文件结尾
[root@localhost test]# ls >> stdout
[root@localhost test]# cat stdout 
11.txt
22.txt
stdout
11.txt
22.txt
stdout
# 2 > file 将标准错误重定向到file >>用法一致
# 将标准输出或者标准错误丢弃
[root@localhost test]# ls > /dev/null
# >file 2>&1 或 &>file 将标准错误重定向到标准输出
```

### 本次课程涉及的命令

#### `find`查找某个名字的文件或者文件夹

```bash
[root@localhost test]# find / -name test
/root/test
/var/lib/AccountsService/users/test
/var/db/sudo/test
/var/spool/mail/test
/usr/bin/test
/usr/lib/alsa/init/test
/usr/lib64/python2.7/test
/usr/lib64/python2.7/site-packages/OpenSSL/test
/usr/lib64/python2.7/unittest/test
/usr/share/espeak-data/voices/test
/home/test
/test
# / 是查找的目录  test是查找的精确字符
# 使用 通配符 * 表示匹配任意
```

#### `grep`在某个文件中查找字符

```bash
[root@localhost test]# grep boy 11.txt 
I am boy
# 带行数  -n
[root@localhost test]# grep bo 11.txt  -n
1:I am boy
# -v不匹配
[root@localhost test]# grep bo 11.txt  -nv
2:I am girl
3:hahahaha
# -B查看前面的行数-A查看后面的行，会带出查找行以及前面或后面的行
[root@localhost test]# grep bo 11.txt  -n -A1
1:I am boy
2-I am girl
```

#### `|`管道前一个标准输出作为后面的标准输入

```bash
[root@localhost test]#  cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator
```

#### `tee`

将标准输入复制到标注输出中，并且还会向标准输出输出内容。用于保存中间步骤的标注输出

```bash
[root@localhost test]#  cat /etc/passwd |tee passwd.txt| grep root
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost test]# ls
11.txt  22.txt  passwd.txt  stderr  stdout
```

#### `vi`或`vim`

- 四种模式

  1. 命令模式
  2. 编辑模式
  3. 末行模式
  4. 可视化模式

- 切换模式

  	1. 刚进入vi时，是`命令模式`
   	2. `insert` `I` `i` `A` `a` `O` `o`都可以进入`编辑模式`

  ```bash
  I :	再本行行首进行插入  
  i:	光标前面插入      
  a:	光标后面插入
  A: 	本行得行尾进行插入   
  o:	新建一行插入     
  O:	本行得上一行进行插入
  # -------------------
  按`ESC`退出编辑模式回到命令模式
  ```

  3. `shift+;` 也是是`:`可以进入`末行模式`
  4. `Ctrl+V`或 `Ctrl+v`进入可视化模式

- 命令模式命令汇总

```
x :			光标在哪里删除哪里
u : 		撤销操作
Ctrl+r : 	重做撤销
dw :		删除一个单词
dd :		删除一行
dG : 		删除光标行到文件末
dgg :		删除光标行到文件首
yy :		复制一行
p :			粘贴
5yy :		复制多行
yw :		复制一个单词
G :			到文件末尾
gg :		到文件首部
```

- 末行模式常用汇总

``` bash
q  	#离开
w  	#保存
x 	#保存离开 wq
q! 	#强制离开
r /test/11.txt	# 将11.txt文件的内容，追加到当前编辑文件的尾部
```

- Linux 自带的`vimtutor`教程

## 第四课

### 关于Linux 的用户

1. 用户分类：

```bash
# UID 是用户ID
​	UID 0分配给超级用户(root)
​	UID 1-200 是一系列的 系统用户 静态分配给红帽的系统进程
​	UID 201-999  是一系列的 系统用户，共文件系统中没有自己的文件的系统进程使用。通常在安装需要他们的软件时，从可用池中动态分配他们。程序一这些五特权的系同用户身份运行。一边限制他们仅访问正常运行的所需资源
​	UID 1000+ 普通用户
```

2. 用户组

```bash
# 主要组或基本组
如果没有指定用户组，创建用户的时候系统会默认同时创建一个和这个用户名同名的组，这个组就是基本组。在创建文件时，文件的所属组就是用户的基本组。
# 附属组
除了基本组之外，用户所在的其他组，都是附加组。用户是可以从附加组中被删除的。
# ---------------------------
用户不论为与基本组中还是附加组中，就会拥有该组的权限。一个用户可以属于多个附加组。但是一个用户只能有一个基本组。 
```

3. 关于用户与组的信息存放位置

```bash
/etc/passwd  用户信息
/etc/shadow 用户密码相关
/etc/group  用户组信息
/etc/gshadow  存放组密码信息
```

4. 关于3中文件的内容

```bash
# /etc/passwd
用户名：密码：UID：GID：描述信息：家目录：shell环境
# /etc/shadow
用户名：加密后的密码：最后一次修改密码的时间：密码最小生存周期：密码最大生存周期：密码到期前一天开始警告：密码到期之后还可以使用的天数：账号过期时间：保留项
```

### Linux 红帽系破解root密码

1. 重启 reboot
2. 进入启动项后，选择非默认项，按键 `e`进入编辑模式

```bash
# 我的常用默认项
Red Hat Enterprise Linux Server,with Linux 3.10.0-123.e17.x86_64
# 我的非默认项
Red Hat Enterprise Linux Server,with Linux 0-rescue-
```

3. 找到 `linux16`关键字所在行，在行末键入`console=tty0 rd.break`
4. 按`Ctrl+x`，等待进入 switch_root模式
5. 键入`mount -0 remount,rw /sysroot`
6. 键入`chroot /sysroot`
7. 键入`echo "new passwd"|passwd --stdin root`
8. 键入`touch /.autorelabel`
9. 键入两次`exit`，等待自动重启。

### 本次课程涉及的命令

#### `id`查看用户信息

```bash
# 查看当前用户
[root@localhost Desktop]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
# 查看其他用户
[root@localhost Desktop]# id user1
uid=1003(user1) gid=1003(user1) groups=1003(user1),10(wheel)
```

#### `passwd`修改用户密码

```bash
# 修改当前用户密码
[root@localhost Desktop]# passwd
Changing password for user root.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
# root用户修改普通用户的密码
[root@localhost Desktop]# passwd user1
Changing password for user user1.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

#### `useradd`添加用户

```bash
# 创建用户
[root@localhost Desktop]# useradd testuser
[root@localhost Desktop]# id testuser
uid=1008(testuser) gid=1008(testuser) groups=1008(testuser)
# -u 指定UID
# -G 指定附属组
# -g 指定主组（默认会创建一个与user同名的组）
# -s 指定一个shell
# 不让用户登陆系统
[root@localhost Desktop]# useradd test_no_login -s /sbin/nologin
[root@localhost Desktop]# id test_no_login
uid=1009(test_no_login) gid=1009(test_no_login) groups=1009(test_no_login)
[root@localhost Desktop]# su test_no_login
This account is currently not available.
# 发现不能够登陆shell
```

#### `usrmod`修改用户一些信息。

```bash
# -u 修改UID
# -L 锁定用户
# -s 指定一个shell
# -U 解锁用户
# -g 修改主要组
# -G 修改附属组
```

#### `usrdel`删除用户

```bash
# 删除用户
[root@localhost Desktop]# userdel user2
# -r 删除家目录
[root@localhost Desktop]# userdel -r user2
```

#### `su` 切换用户

```bash
# root用户切换到任何用户不需要输入密码
# su - test 是用这个用户直接登陆。不加-是用它的shell环境
[root@localhost Desktop]# su kane
[kane@localhost Desktop]$ 
```

#### `sudo`以root身份运行命令。注：需要再附属组上增加（wheel）

```bash
[kane@localhost /]$ rm -rf test
rm: cannot remove ‘test’: Permission denied
[kane@localhost /]$ sudo rm -rf test
[sudo] password for kane:
[kane@localhost /]$ 
```

#### `groupadd`添加一个组

```bash
#  -g可以直接给一个组ID
```

#### `groupmod`修改组的信息

```bash
# 修改组id
groupmod -g gid
# 修改组的名字
groupmod -n new_name old_name
```

#### `groupdel`删除组

#### `chage`负责管理用户密码时效问题

```bash
# -m 两次改变密码之间相距的最小天数即"最小天数"
# -M 两次改变密码之间相距的最大天数即"最大天数"
# -W 过期警告天数即"警告天数"
# -E 过期日期 日期格式：YYYY-MM-DD
chage -E 2019-10-01 test
```

## 第五课

### 用户权限

1. 查看文件的权限

```bash
[kane@localhost /]$ ll
total 36
----------.   1 root root 1751 Aug 22 20:58 ~
lrwxrwxrwx.   1 root root    7 Aug 16 04:39 bin -> usr/bin
dr-xr-xr-x.   3 root root 4096 Aug 15 20:55 boot
drwxr-xr-x.  20 root root 3260 Aug 26 10:07 dev
drwxr-xr-x. 132 root root 8192 Aug 26 15:00 etc
```

2. 关于查看为你教案权限结果的解释

```bash
# 1位 ：d是目录 -是 文件  c代表字符设备  b代表块设备  l代表得是连接设备
# -------------------------------------
# 2-10位：三组权限，每三个一组一次为：[用户u][组g][其他o]
# r可读 w可写 x可执行- 是没有这个权限
# 数字表示 r:4 w:2 x:1 -:0 [修改权限使用]
# -------------------------------------
# 11位	代表有多少的连接
# 12位  	用户
# 13位	所属组
# 14位	文件大小，默认是字节单位
# 15位	最后修改的时间
# 16位	文件名字
```

### 特殊权限

1. suid 表示以文件的拥有者身份运行
2. sgid 表示以文件的所属者身份运行
3. sticky 表示目录下面创建的文件将继承目录的组所属的关系，而不是用户的。只有对目录有w全县的用户可以删除拥有的的文件，其他拥有着都不可以删除。
4. 特殊权限的表示

```bash
分别占用 u g o 三组权限的得第三位 
suid   s 正常   S说明没有x权限
guid   s 正常   S说明没有x权限
sticky t 正常   T说明没有x权限
# --------------演示---------------
[root@localhost /]# chmod u+s test
[root@localhost /]# ls -dl test
drwsrwxrwx. 2 root root 6 Aug 26 16:10 test
# 去除执行权限s变成S
[root@localhost /]# chmod u-x test
[root@localhost /]# ls -dl test
drwSrwxrwx. 2 root root 6 Aug 26 16:10 test
# 其他两个类似
# 数字法 suid =4      sgid  2   sticky 1 
# 用数字法设置sgid
[root@localhost /]# chmod 2777 test
[root@localhost /]# ls -dl test
drwsrwsrwx. 2 2777 root 6 Aug 26 16:10 test
```

### 创建文件文件夹默认权限umask

1. 查看umask

```bash
# root用户 0022
[root@localhost /]# umask
0022
# 普通用户0002
[root@localhost /]# su kane
[kane@localhost /]$ umask
0002
```

2. 默认权限

```bash
# 目录默认权限 777
777-umask
# 文件默认权限 666
666-umask
# 以root用户为例
[root@localhost test]# touch 11.txt
[root@localhost test]# mkdir 11
[root@localhost test]# ll
total 0
drwxr-sr-x. 2 root root 6 Aug 26 16:33 11
-rw-r--r--. 1 root root 0 Aug 26 16:32 11.txt
# 11.txt权限是 664
# 11 文件夹权限是644
```

3. 修改 umask

```bash
# 临时修改
umask = 777
# 永久修改，不建议
修改  .bash_profile   /etc/profile
```

### 本次课程涉及到的命令

#### `chmod`修改文件权限

```bash
# 数字法修改，7是4(r)+2(w)+1(x) 三个7是设置3组
[root@localhost /]# chmod 777 test
# 符号法修改，可以直接等，可加，可减
[root@localhost /]# chmod u=rw test
[root@localhost /]# chmod u+x test
```

#### `chown`修改用户与组

```bash
chown user1 123  		#将123的用户改成user1
chown  : user2 123 		#将123的组改成user1
chown user3:user3 123	#同时设置用户与组
chown --reference /tmp/ /test/ 复制文件得权限
```

## 第六课

### 进程

**进程**：已经启动的可执行程序的运行中的实例。每个进程都有自己的地址空间，并占用了一定的系统资源。

### 如何产生一个进程

1. 执行程序或命令
2. 计划任务

### 在终端中对进程管理

1. 运行一个前台进程

```bash
[root@master Desktop]# firefox

(process:3731): GLib-CRITICAL **: g_slice_set_config: assertion `sys_page_size == 0' failed
# 会占用当前得终端
```

2. 运行一个后台进程

```bash
[root@master Desktop]# firefox &
[1] 3796
[root@master Desktop]# 
(process:3796): GLib-CRITICAL **: g_slice_set_config: assertion `sys_page_size == 0' failed

[root@master Desktop]# 
```

3. jobs查看当前终端下得进程

```
[root@master Desktop]# jobs
[1]+  Running            
```

4. 前台后台切换

```bash
# 切换到前端 终端又被占用
root@master Desktop]# fg 1
firefox
# 切换到后端
root@master Desktop]# bg 1
```

### Systemd 控制服务启动，守护进程

1. 服务单元的状态

```markdown
loaded   服务单元的配置文件已被处理
active （running）    运行中
active(exited)	某些一次性运行的服务已经陈工被执行并退出
active(waiting)服务已经运行，但正在等待某个时间
inactive 	没运行
disabled 开机不运行
enabled 开机运行
static 	不能够被设置为开机启动	
```

2. systemd命令

```markdown
# 查看服务状态
systemctl status sshd
# 开启
systemctl start sshd
# 结束
systemctl stop sshd
# 重启
systemctl restart sshd
# 是否开机启动
systemctl is-enabled sshd
# 是否活动
systemctl is-active sshd
# 列出所有的服务单元
systemctl list units 
# 列出所有服务单元文件
systemctl  list-unit-files 
```

### 本次课程涉及到的命令

#### `ps -aux`查看进程

注：该命令不加`-`也是好用的。

```bash
# a显示所有前台进程，x显示所有后台进程 u显示用户
[root@centos ~]# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  51896  3256 ?        Ss   Jul02   1:59 /usr/lib/systemd/systemd --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    Jul02   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Jul02   1:54 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Jul02   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    Jul02   0:23 [migration/0]
root         8  0.0  0.0      0     0 ?        S    Jul02   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        S    Jul02   6:56 [rcu_sched]
root        10  0.0  0.0      0     0 ?        S    Jul02   1:31 [watchdog/0]
```

- 最上一行信息介绍

```markdown
USER 	哪个用户产生进程
PID 	进程号
%CPU	占用cpu资源的百分比
%MEM 	占用物理内存的百分比
VSZ   	虚拟内存的占用
RSS  	占用物理内存的大小单位（kb）
TTY		只以哪一个控制台（终端）运行程序
STAT	状态
        R 表示正在运行
        +表示后台进程
        S表示睡眠状态的进程
        s包含子进程（父进程停止，子进程也会停止）
        T停止状态的进程（显示T的可以被唤醒）
START  	进程的启动时间
TIME  	进程占了CPU多长时间
COMMAND 这个进程哪个命令来的
```

#### `ps -le`查看进程

```bash
# l显示详细信息 e显示左右进程 f标准格式
[root@centos ~]# ps -efl
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root         1     0  0  80   0 - 12974 ep_pol Jul02 ?        00:01:59 /usr/lib/systemd/systemd --system --deserialize 21
1 S root         2     0  0  80   0 -     0 kthrea Jul02 ?        00:00:00 [kthreadd]
1 S root         3     2  0  80   0 -     0 smpboo Jul02 ?        00:01:54 [ksoftirqd/0]
1 S root         5     2  0  60 -20 -     0 worker Jul02 ?        00:00:00 [kworker/0:0H]
1 S root         7     2  0 -40   - -     0 smpboo Jul02 ?        00:00:23 [migration/0]

```

#### `top` 动态查看进程

```bash
top - 17:05:45 up 56 days,  6:46,  1 user,  load average: 0.11, 0.05, 0.05
Tasks: 126 total,   1 running, 125 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.2 sy,  0.0 ni, 99.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  5946700 total,   783956 free,  3881272 used,  1281472 buff/cache
KiB Swap:  4194300 total,  4097288 free,    97012 used.  1686820 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                         
15056 root      20   0  990364  31204   7096 S   0.7  0.5  34:30.57 dockerd-current                                                                                                                                                 
12612 root      20   0       0      0      0 S   0.3  0.0   0:00.06 kworker/1:1   
```

- top拥有交互式命令

```markdown
	? 或H 弹出帮助信息
​	P 以CPU的使用率排序
​	M 以内存的使用率排
​	N 以PID的顺序排
​	q退出top
```

- top 最上层显示的意义

```markdown
# 第一行
当前时间 开机时间 几个用户 资源使用率：1分钟，5分钟，15分钟
# 第二行
总共任务数  运行的  睡眠的  停止的  僵尸进程
`注：什么是僵尸进程：父进程停止了 子进程没停止叫做僵尸进程。`
# 第三行
CPU使用率  us用户占用sy 系统占用的 ni改变过优先级的进程占用的 id空闲的  wa 等待进程  hi 硬件中断占用的  si 软中断占用的  st 有虚拟机是，虚拟占用的
# 第四行
物理内存：总数 使用中 空闲的  缓存区
# 第五行
虚拟内存：总数 使用中 空闲的  缓存区
# 第六行
PID USER PR 优先级 NI nice值   VIRT 申请的虚拟内存 RES常驻内存实际的 SHR 共享的  S 状态
```

#### `free`查看内存使用情况

```bash
# -h 是以人类能看的懂得样式输出
[root@centos ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           5.7G        3.7G        760M         33M        1.2G        1.6G
Swap:          4.0G         94M        3.9G
```

#### `pstree` 树状结构显示进程

```bash
# -p参数是显示子进程
[root@centos ~]# pstree -p
systemd(1)─┬─NetworkManager(701)─┬─{NetworkManager}(726)
           │                     └─{NetworkManager}(729)
           ├─abrt-watch-log(7869)
           ├─abrtd(7850)
           ├─atd(7480)
           ├─auditd(24144)───{auditd}(24153)
           ├─crond(31725)
           ├─dbus-daemon(660)
```

#### `kill` 杀掉进程

```bash
# -l  查看信号 -1 重启（让进程立即关闭，然后重新读取配置文件）-2 强制结束 （终止，力度轻）-9 强制结束	（强制关闭，用来立即结束程序）
# -15 正常终止 （正常关闭）-18 恢复后台 （可以让暂停的进程恢复执行，本信号不能被阻断）-19 前台与运行的放在后台（该信号可以暂停前台进程，相当于CTRL+Z。本新号不能被阻断）
[root@centos ~]# kill -15 pid
```

#### `killall` `pkill`用进程名字杀进程

