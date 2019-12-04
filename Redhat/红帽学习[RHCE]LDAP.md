[TOC]

# `OpenLDAP` 服务端与客户端配置

`OpenLDAP`是`轻型目录访问协议`，利用它可以进行统一的认证服务。如果需求需要配置多个服务器，并且多账号的话，可以考虑用`OpenLDAP`进行统一配置，来完成一组计算机之间的认证。  

## 关于`LDIF`

### 一个LDIF基本结构`一个条目`

注：解释的不完整。

```yml
dn: ou=People,dc=example,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit
```

1. `dn`就是数据库中的唯一主键，在`LDAP`中唯一标识一个条目

2. `ou`就是`organizationalUnit`该条目需要有`organizationalUnit`而`ou`这一行就是设置具体值得
3. `objectClass`表示属性，类似于代码中的类的属性

### 属性

```bash
 cn：common name，指一个对象的名字。如果指人，需要使用其全名。
 o：organizationName，指一个组织的名字。
 ou：organizationalUnitName，指一个组织单元的名字。
 uid: user id ，唯一的用户id
 userPassword：用户密码
 uidNumber: 用户ID
 gidNumber: 用户组ID
 homeDirectory: 用户目录
```

### `Object`的类型

```bash
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
```

## 服务端

### 安装

```bash
[root@rhel2 ~]# yum install openldap openldap-servers migrationtools
```

### 生成证书

```bash
# 分别生成一个公钥与私钥
openssl req -new -X509 -nodes -out /etc/openldap/certs/cert.pem -keyout /etc/openldap/certs/private.pem -days 365


# 更改两个文件的权限 为ldap
cd /etc/openldap/certs
chown ldap:ldap *.pem
# 查看
ll
total 8
-rw-r--r-- 1 ldap ldap 1273 Nov 14 20:10 cert.pem
-rw-r--r-- 1 ldap ldap 1704 Nov 14 20:10 private.pem
# 将私钥改成只有ldap用户可读写
chmod 600 private.pem 
ll
total 8
-rw-r--r-- 1 ldap ldap 1273 Nov 14 20:10 cert.pem
-rw------- 1 ldap ldap 1704 Nov 14 20:10 private.pem
```

注：生成证书时 `Common Name (eg, your name or your server's hostname) []:`这个东西一定要写当前服务器的`hostname`，否则证书没法验证

### 生成默认数据

```bash
# 将 示例DB_CONFIG.example复制出来
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/DB_CONFIG
# 到目录下
cd /var/lib/ldap/
# 更改权限
chown ldap:ldap *
# 执行一次 slaptest
```

### 修改基本的配置

```bash
# 在任意位置增加如下文件，内容如下
# 关于ldif的格式
# 1. 每一个dn都需要有空行
# 2. 每一行末尾都`不能`有空格
# 3. 每一个: 后面都要跟着一个空格
[root@rhel2 openldap]# vim changes.ldif
# 修改 主dn 为  dc=example,dc=com 可以替换成自己的
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=example,dc=com
# 修改管理员密码
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}zcesbHdBiGMQbVBCa7W2ZxjWxXGVHeG2

# 指定公钥位置
dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFIle: /etc/openldap/certs/cert.pem

# 指定私钥位置
dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFIle: /etc/openldap/certs/private.pem

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=example,dc=com" read by * none
# 以上修改的其实就是在修改 /etc/openldap/slapd.d 下的文件，但是文件汇总有提示不让手动更改，通过ldapmodify 更改
# 执行修改
[root@rhel2 openldap]# ldapmodify -Y EXTERNAL -H ldapi:/// -f changes.ldif 
###########################结果#########################
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "cn=config"

modifying entry "cn=config"

modifying entry "olcDatabase={1}monitor,cn=config"
##########################结果#########################
# 测试密码
[root@rhel2 openldap]# ldapwhoami -x  -W -D "cn=config"
Enter LDAP Password: 
dn:cn=config
```

### 导入基础数据

```bash
# 增加文件 base.ldif
# 关于ldif的格式
# 1. 每一个dn都需要有空行
# 2. 每一行末尾都`不能`有空格
# 3. 每一个: 后面都要跟着一个空格
[root@rhel2 openldap]# vim base.ldif 
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

dn: ou=People,dc=example,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=example,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit


# 实际上市增加了两个ou People 与 Group 及 用户 与 用户组
# 执行新增
[root@rhel2 openldap]# ldapadd -Y EXTERNAL -H ldapi:/// -f base.ldif 
#######################结果###########################
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "ou=People,dc=example,dc=com"

adding new entry "ou=Group,dc=example,dc=com"
########################结果##########################

# 查询数据
[root@rhel2 openldap]# ldapsearch -x ou=Group -b dc=example,dc=com
########################结果##########################
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: ou=Group
# requesting: ALL
#

# Group, example.com
dn: ou=Group,dc=example,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
########################结果##########################
```

### 关于`ldif`的格式

1. 每一个`dn`都需要有空行

2. 每一行末尾都***`不能`***有空格

3. 每一个`:` 后面都要跟着一个空格

### 批量创建用户

```bash
# 创建一个本地目录存放用户
[root@rhel2 openldap]# mkdir /home/guests
# 使用for 创建20个用户
[root@rhel2 guests]# for i in $(seq 1 20); do useradd -d /home/guests/ldapuser$i ldapuser$i; done
# 利用for设置20个用户的密码
[root@rhel2 guests]# for i in $(seq 1 20); do echo ldapuser$i | passwd --stdin ldapuser$i; done
```

### 批量导入用户到`LDAP`

1. 更改`migrationtools` 的 `baseDN`

```bash
[root@rhel2 openldap] cd /usr/share/migrationtools
[root@rhel2 migrationtools]# vi migrate_common.ph
# 更改下面两行为自己的 base dn
# Default DNS domain
$DEFAULT_MAIL_DOMAIN = "example.com";

# Default base 
$DEFAULT_BASE = "dc=example,dc=com";
```

2. 导入用户

```bash
# 查询所有用户并输出到一个文件
[root@rhel2 migrationtools]# grep ":10[0-9][0-9]" /etc/passwd > > /etc/openldap/passwd

# 如果用户中有部分不想加入的用户直接删除即可

# 将 /etc/openldap/passwd 文件 转成 ldif文件
[root@rhel2 migrationtools]# ./migrate_passwd.pl /etc/openldap/passwd /etc/openldap/passwd.ldif

# 下面是用户ldapuser1 的数据样式
[root@rhel2 migrationtools]# cat /etc/openldap/passwd.ldif 
dn: uid=ldapuser1,ou=People,dc=example,dc=com
uid: ldapuser1
cn: ldapuser1
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$1$V1vroanf$NTVsrHIwOTQUpY0Q2CXHM/
shadowLastChange: 18215
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/guests/ldapuser1

```

3. 导入用户组

```bash
# 查询所有用户组并输出到一个文件
[root@rhel2 migrationtools]# grep ":10[0-9][0-9]" /etc/group > > /etc/openldap/group

# 如果用户中有部分不想加入的用户直接删除即可

# 将 /etc/openldap/group 文件 转成 ldif文件
[root@rhel2 migrationtools]# ./migrate_group.pl /etc/openldap/group /etc/openldap/group.ldif

# 下面是用户组ldapuser1 的数据样式
[root@rhel2 migrationtools]# cat /etc/openldap/group.ldif 
dn: cn=ldapuser1,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapuser1
userPassword: {crypt}x
gidNumber: 1001
```

4. 查询刚才添加的用户与组

```bash
# 会发现用户与用户组都查询出来了
[root@rhel2 migrationtools]# ldapsearch -x cn=ldapuser1 -b dc=example,dc=com
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: cn=ldapuser1
# requesting: ALL
#

# ldapuser1, People, example.com
dn: uid=ldapuser1,ou=People,dc=example,dc=com
uid: ldapuser1
cn: ldapuser1
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQxJFYxdnJvYW5mJE5UVnNySEl3T1RRVXBZMFEyQ1hITS8=
shadowLastChange: 18215
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/guests/ldapuser1

# ldapuser1, Group, example.com
dn: cn=ldapuser1,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapuser1
userPassword:: e2NyeXB0fXg=
gidNumber: 1001

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

## 客户端

### 安装

```bash
[root@rhel1 ~]# yum install openldap-clients nss-pam-ldapd authconfig-gtk
```

### 配置`ldap`[有问题]

```bash
# 图形化配置工具
[root@rhel1 ~]# authconfig-gtk
# 如果使用xshell 时输入上面的命令，需要下载不免费的 Xmanager
# 所以想使用  authconfig-gtk 必须保证客户端安装了图形化界面。
```

1. 将服务端的公钥`cert.pem`复制到 客户端。

```bash
scp cert.pem ip:/root/
```

2. 激活图形化配置后按如下进行选择

```bash
User Account Database	 	 ------- LDAP
LDAP Search Base DN  	 	 ------- dc=example,dc=com
LDAP Server           		 ------- ldap://ip/
Use TLS to encrypt connnects ------- check
# 此处必须check上否则不让通过
Authentication Method		 ------- LDAP password
# 点击 Download CA Certificate 输入本地目录
file:///root/cert.pem
```

3. 查看用户

```bash
[root@rhel1 ~]# getent passwd ldapuser1
```

4. 问题

此处进行多次尝试还是有问题，一直连接不到`LDAP`的Server。最终可以用`nslcd`的log中获得如下错误

```bash
Nov 18 16:24:30 rhel1.node.com nslcd[35848]: [8b4567] <passwd="ldapuser1"> ldap_start_tls_s() failed (uri=ldap://192.168.245.11/): Connect error: TLS error -8172:Peer's certificate issuer has been marked as not trusted by the user.
```

记录下我尝试的方法：

```markdown
1. 在文件 /etc/openldap/ldap.conf 增加 `TLS_REQCERT never` 
2. 在文件 /etc/openldap/ldap.conf 增加 `TLS_REQCERT allow` 
3. 在服务的使用公钥、私钥获得自签名证书，复制到客户端使用
4. 在服务端 /etc/sysconfig/slap 文件中 增加`ldaps:///`，监控端口`636`已经在使用了，但是客户端配置ldaps的时候还是不能够正确访问
```

**要是有人尝试成功了麻烦告知一下**

### 不使用SSL进行客户端配置

图形化界面`authconfig-gtk` 要求必须使用SSL证书，或者使用`ldaps`进行连接

`authconfig-tui`对此没有要求

```bash
[root@rhel1 ~]# authconfig-tui
# 进入一个简易的画面，
# 1. 使用tab 进行切换选择项
# 2. 使用space 进行选择
# 3. 使用F12 直接到下一步
```

```markdown
1. 第一个页面左边选中 `Use LDAP`，右边选中 `Use LDAP AUthertication`
2.  第二个页面，不选中 `Use TLS`
	Server: `ldap://192.168.245.11/`
	Base DN: `dc=example,dc=com`
注： 在这里也尝试了选中 `Use TLS` 还是会报证书那个问题
3. 等待一段时间
4. 验证结果
[root@rhel1 ~]# getent passwd ldapuser1
ldapuser1:x:1001:1001:ldapuser1:/home/guests/ldapuser1:/bin/bash
[root@rhel1 ~]# getent passwd ldapuser20
ldapuser20:x:1020:1020:ldapuser20:/home/guests/ldapuser20:/bin/bash
5. 用户已经同步到改客户端上了。并且认证通过
```

## `NFS`共享用户家目录

### 不配置`家目录共享`用户无法正常使用

```bash
# 认证通过后我们使用用户会报错 
[root@rhel1 ~]# su - ldapuser1
su: warning: cannot change directory to /home/guests/ldapuser1: No such file or directory
mkdir: cannot create directory '/home/guests': Permission denied
-bash-4.2$ 
# 只能完成登录，但是做不了任何操作
```

### 配置服务端的`NFS`

```bash
[root@rhel2 ~]# yum install nfs-utils
[root@rhel2 ~]# systemctl enable nfs-server
[root@rhel2 ~]# systemctl start nfs-server
[root@rhel2 ~]# vim /etc/exports
[root@rhel2 ~]# cat /etc/exports
/home/guests	192.168.245.0/24(rw,sync)
# 配置 共享/home/guests 目录 共享读写到网段 192.168.245.0
[root@rhel2 ~]# exportfs -rv
exporting 192.168.245.0/24:/home/guests
[root@rhel2 ~]# exportfs -v
/home/guests  	192.168.245.0/24(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
# 添加防火墙开放
[root@rhel2 ~]# firewall-cmd --add-service=nfs --permanent
[root@rhel2 ~]# firewall-cmd --reload
```

### 配置客户端的`autofs`

```bash
[root@rhel1 ~]# yum install autofs nfs-utils
# 添加自定义配置的auto.guests
[root@rhel1 ~]# vim /etc/auto.guests 
[root@rhel1 ~]# cat /etc/auto.guests 
*	-rw,nfs4 rhel2.node.com:/home/guests/&
# 编写主文件
[root@rhel1 ~]# vim /etc/auto.master
/home/guests	/etc/auto.guests
# 行末加入
[root@rhel1 ~]# systemctl enable autofs
[root@rhel1 ~]# systemctl start autofs
# 再次切换用户
[root@rhel1 ~]# su - ldapuser1
Last login: Tue Nov 19 10:07:36 CST 2019 on pts/2
[ldapuser1@rhel1 ~]$ pwd
/home/guests/ldapuser1
[ldapuser1@rhel1 ~]$ touch test.txt
[ldapuser1@rhel1 ~]$ ll
total 4
-rw-rw-r--. 1 ldapuser1 ldapuser1 5 Nov 19 10:22 test.txt
```

### 查看客户端挂载情况

```bash
[ldapuser1@rhel1 ~]$ df -Th
df: ‘/run/user/0/gvfs’: Permission denied
Filesystem                            Type      Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root                 xfs        18G  3.0G   15G  17% /
devtmpfs                              devtmpfs  905M     0  905M   0% /dev
tmpfs                                 tmpfs     914M  140K  914M   1% /dev/shm
tmpfs                                 tmpfs     914M   21M  894M   3% /run
tmpfs                                 tmpfs     914M     0  914M   0% /sys/fs/cgroup
/dev/sdb5                             ext4      190M  1.6M  175M   1% /third
/dev/sda1                             xfs       497M  119M  379M  24% /boot
/dev/sr0                              iso9660   3.5G  3.5G     0 100% /mnt
rhel2.node.com:/home/guests/ldapuser1 nfs4       18G  1.3G   17G   8% /home/guests/ldapuser1
[ldapuser1@rhel1 ~]$ 
```



