[TOC]

# DNS 域名系统

## 定义

域名系统是域名和IP地址相互映射的一个分布式数据库，能够是用户更方便的访问互联网。不用去记住能够被机器直接读取的IP。

## 域名分类

域是分层管理的

```markdown
# 根域: [.]
# 顶级域:
按性质:	[.org\.net\.com\.edu\.gov]
按国家:	[.cn\.tw\.hk]
# 普通域
比如: [.baidu]
```

## 解析流程

`本地DNS缓存` -> `本地hosts文件` -> `指定的DNS服务器`

如果指定的DNS服务器没有找到对应的域名，会返回到客户端，客户端会向上一级DNS服务器继续发送请求。直至查询到或者顶级服务器也没查询到。

## DNS分类

```markdown
1. 主DNS服务器：存储原始资料
2. 从DNS服务器：自动更新注DNS服务器的数据
3. 缓存服务器：转发来自客户端的请求，但是会缓存查询回来的结果 
4. 转发器：不向根域发送请求，而是直接发给其他的服务器，并不缓存结果
```

## 资源记录

### 格式

`域名	生存期	类别		类型		值	`

```markdown
1. 域名：指定这条记录适用于哪个域
2. 生存期：指定该条记录的稳定程度单位秒
3. 类别：互联网信息都是 IN
4. 类型：每个资源记录类型
5. 值：对应的值
```

### 资源记录类型

```markdown
1. SOA Star of authority 起始授权 必须是第一行 只能有一个
2. A IPV4 address
3. AAAA IPV6 address
4. MX Mail exchange
5. NS Name Server
6. CNAME Canonical name 可以将域名转换，别名
7. PTR 方向解析
...还有很多
```

## 用`bind`搭建一台`DNS`服务器

我们在`RHEL1`上搭建DNS服务器

### 安装`bind`

```bash
yum install bind bind-utils
```
### 创建自己的`zone`文件

```bash
# 可以使用如下命令查看 包 bind 影响的文件夹以及文件夹
[root@rhel1 Desktop]# rpm -ql bind
# 在 /var/named 文件下创建 node.com.zone
[root@rhel1 named]# cat node.com.zone 
$TTL 1D # $TTL指令表示一个资源记录在其他DNS服务器
		# $ORIGIN指令表示该zone文件用来描述的域(domain)名称
@	IN	SOA @ admin.qq.com.	(
						20191128;serial版本号
						1D;refresh 刷新时间,每隔多久去查看版本号
						1H;retry 重新刷新时间
						1W;expire 过期时间
						3H;否定答案缓存时间)  # 配置SOA。它定义了一个域的全局特性，必须是出现在zone文件中的第一个资源记录，而且一个zone文件中必须只有一个SOA资源记录。
@	IN NS 	server		# 配置NS
server	IN A	192.168.143.10	#配置server地址
control1	A	192.168.143.10	#配置控制接口的域名
control2 	A	192.168.143.11	#配置控制接口的域名
control3	A	192.168.143.12	#配置控制接口的域名
net121	A	192.168.140.10		#配置12服务器连接接口的域名
net122	A	192.168.140.11		#配置12服务器连接接口的域名
net231	A	192.168.245.10		#配置23服务器连接接口的域名
net232  A	192.168.245.11		#配置23服务器连接接口的域名

```

### 在主配置文件中，增加自己的zone

```bash
# 编辑配置文件 /etc/named.rfc1912.zones 添加如下内容
[root@rhel1 named]# tail -4 /etc/named.rfc1912.zones 
zone "node.com." IN {
	type master;
	file "node.com.zone" 
};
### 这个位置正常些文件名字就可以，如果named启动不了改成绝对路径/etc/named/node.com.zone
# 也可以在 /etc/named.conf 文件中增加
```

### 检测是否配置成功

```bash
#  检测是否配置成功
[root@rhel1 named]# named-checkzone node.com.  node.com.zone 
zone node.com/IN: loaded serial 20191128
OK
# 启动named
[root@rhel1 named]# systemctl start named
[root@rhel1 named]# systemctl enable named
```

### 测试配置的结果

```bash
# dig命令检测 dns 解析，
### 没有dig命令 请安装 `bind-utils`
[root@rhel1 named]# dig -t A control1.node.com
; <<>> DiG 9.9.4-RedHat-9.9.4-14.el7 <<>> -t A control1.node.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61583
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;control1.node.com.		IN	A

;; ANSWER SECTION:
control1.node.com.	86400	IN	A	192.168.143.10

;; AUTHORITY SECTION:
node.com.		86400	IN	NS	server.node.com.

;; ADDITIONAL SECTION:
server.node.com.	86400	IN	A	192.168.143.10

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Nov 28 13:31:41 CST 2019
;; MSG SIZE  rcvd: 99
# 直接ping也可以
[root@rhel1 named]# ping control1.node.com
PING control1.node.com (192.168.143.10) 56(84) bytes of data.
64 bytes from 192.168.143.10: icmp_seq=1 ttl=64 time=0.027 ms
# nslookup 测试
[root@rhel1 named]# nslookup
> control1.node.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	control1.node.com
Address: 192.168.143.10
```
### 防火墙放行

```bash
firewall-cmd --add-service=dns --permanent
firewall-cmd --add-service=dns
```

### 在`RHEL2`上配置并测试

```bash
# 增加DNS配置
[root@rhel2 html]# tail -1 /etc/resolv.conf 
nameserver 192.168.143.10
# 重启网络
[root@rhel2 html]# systemctl restart network
# 确认配置还在
[root@rhel2 html]# tail -1 /etc/resolv.conf 
nameserver 192.168.143.10
### 有时候重启后配置就不在了。是因为/etc/sysconfig/network-script下的链接配置里已经配置了DNS，可以在链接配置了增加
[root@rhel2 html]# ping control1.node.com
PING control1.node.com (192.168.143.10) 56(84) bytes of data.
```

## 用 unbound搭建一个缓存服务器

### 安装

```bash
[root@rhel2 html]# yum install -y unbound
[root@rhel2 html]# systemctl start unbound
[root@rhel2 html]# systemctl enable unbound
```

### 更改配置

```bash
[root@rhel2 html]# vim /etc/unbound/unbound.conf
# 下面的配置在文件中都能找到，只是需要打开注释
server:
 		 # whitespace is not necessary, but looks cleaner.
         interface: 0.0.0.0
         acces-control: 0.0.0.0/0 allow
         domain-insecure: "node.com"
         # verbosity number, 0 is least verbose. 1 is default.
         verbosity: 1
forward-zone:
         name: "."
         forward-host: 192.168.143.10
# 检测配置
[root@rhel2 html]# unbound-checkconf 
unbound-checkconf: warning: . forward-host: "192.168.143.10" is an IP4 address, and when looked up as a host name during use may not resolve.
unbound-checkconf: no errors in /etc/unbound/unbound.conf
# 重启
[root@rhel2 html]# systemctl restart unbound
```

### 防火墙放行

```bash
[root@rhel2 html]# firewall-cmd --add-service=dns --permanent 
[root@rhel2 html]# firewall-cmd --add-service=dns 
```

### 在`RHEL3`上将DNS服务器配置成缓存服务器

```bash
[root@rhel3 ~]# tail -1 /etc/resolv.conf 
nameserver 192.168.143.11
[root@rhel3 ~]# systemctl restart network

[root@rhel3 ~]# ping control1.node.com
PING control1.node.com (192.168.143.10) 56(84) bytes of data.
```

注： `Unbound`与 `bind`都能够搭建主DNS服务器

# 邮件服务器

## 电子邮件架构

Linux服务器也会发送电子邮件，一般是出于自动用途，或者向管理员报告错误。

```bash
# 查看当前邮箱列表，可以看到有一封信
[root@rhel1 Desktop]# mail 
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/mail/root": 3 messages 1 new 2 unread
    1 user@localhost.local  Fri Aug 16 19:06 830/62447 "[abrt] full crash report"
# 我们自己给自己发一封
[root@rhel1 Desktop]# mail root
Subject: test
This is test mail!!!
EOT   # 按CTRL+D发送
# 查看
[root@rhel1 mail]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/mail/root": 2 messages 1 new
    1 user@localhost.local  Fri Aug 16 19:06 848/63026 "[abrt] full crash report"
>N  2 root                  Thu Nov 28 18:10  18/570   "test"
& 2
Message  2:
From root@rhel1.node.com  Thu Nov 28 18:10:16 2019
Return-Path: <root@rhel1.node.com>
X-Original-To: root
Delivered-To: root@rhel1.node.com
Date: Thu, 28 Nov 2019 18:10:05 +0800
To: root@rhel1.node.com
Subject: test
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: root@rhel1.node.com (root)
Status: R

This is test mail!!!
```

## 邮件协议

#### 简单邮件传输协议(SMTP)

用来发送或中转发出的电子邮件，占用`25/tcp`端口

#### 第三版邮局协议(POP3)

用于将服务器上把邮件存储到本地主机，占用`110/tcp`端口

#### 第四版互联网信息访问协议(IMAP4)·

用于在本地追加上访问邮件，占用`143/tcp`端口

## 电子邮件系统

单独使用`Postfix`服务程序不能完成收发邮件的操作。我们需要下面的软件：

1. `Postfix`提供的邮件发送服务`SMTP`
2. `Dovercot`提供的邮件收取服务，即`POP3`
3. `OutlookExpress`客户端收发邮件工具

## 搭建邮件系统

### 创建空的`Postfix`服务器

在`RHEL1`上创建空`Postfix`，在`Redhat`系统中已经默认安装了这个服务程序。

- 查看配置文件

```bash
# 我们只需要配置下面的配置项即可
myhostname			邮局系统的主机名
mydomain			邮局系统的域名
myorigin			从本机寄出邮件的域名名称
inet_interfaces		监听的网卡接口
mydestination		可接受邮件的主机名或域名
mynetworkds			设置可转发哪些主机的邮件
relay_domains		设置可转发那些域名得邮件
```

- 配置`etc/postfix/main.cf`为空的`postfix`

```bash
# 不建议直接编辑，使用`postconf`配置
### 配置本机邮件寄出的域名，会将从本机发出去的发件人域重写成配置项
[root@rhel1 ~]# postconf -e "myorigin=node.com"
### 设置成仅侦听用于发送电子邮件的回环接口
[root@rhel1 ~]# postconf -e "inet_interfaces=loopback-only"
### 设置指向的邮件服务器
### 此处的control2是上面 dns配置的服务器2的域名
[root@rhel1 ~]# postconf -e "relayhost=[control2.node.com]"
### 关闭本地电子邮件发送
[root@rhel1 ~]# postconf -e "local_transport=error:local delivery disabled"
### 本地发送不会接受收件人为本地电子邮件账户的邮件
[root@rhel1 ~]# postconf -e "mydestination="
### 源自127.0.0.0/8和[::1]/128网络的邮件能够由本地空客户端发送到主机
[root@rhel1 ~]# postconf -e "mynetworks=127.0.0.0/8 [::1]/128"
[root@rhel1 ~]# systemctl restart postfix

```

### 配置接收端

在`RHEL2`上配置

```bash
[root@rhel2 ~]# postconf -e "inet_interfaces = all"
[root@rhel2 ~]# postconf -e "mydestination = node.com"
[root@rhel2 ~]# systemctl restart postfix.service 
[root@rhel2 ~]# firewall-cmd --add-service=smtp --permanent 
success
[root@rhel2 ~]# firewall-cmd --add-service=smtp
```

### 在`RHEL1`上发送`RHEL2`查看

```bash
[root@rhel1 ~]# mail root
Subject: From control1.node.com
I am control1
EOT
[root@rhel2 ~]# mail 
>N  5 root                  Thu Nov 28 06:26  21/740   "From control1.node.co"
& 5
Message  5:
From root@node.com  Thu Nov 28 06:26:41 2019
Return-Path: <root@node.com>
X-Original-To: root@node.com
Delivered-To: root@node.com
Date: Thu, 28 Nov 2019 22:26:41 +0800
To: root@node.com
Subject: From control1.node.com
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: root@node.com (root)
Status: R

I am control1
```

## 配置客户端下载邮件

### 更改`RHEL2`上的 `Postfix`配置

```bash
[root@rhel2 ~]# postconf -e "inet_interfaces=all"
[root@rhel2 ~]# postconf -e "myhostname=control2.node.com"
[root@rhel2 ~]# postconf -e "mydomain=node.com"
[root@rhel2 ~]# postconf -e "myorigin=node.com"
[root@rhel2 ~]# postconf -e "mydestination=node.com"
[root@rhel2 ~]# systemctl restart postfix
```

### 创建一个账号

````bash
[root@rhel2 ~]# useradd test_mail_user
[root@rhel2 ~]# echo "test_mail_user" |passwd --stdin test_mail_user
Changing password for user test_mail_user.
passwd: all authentication tokens updated successfully.
````

### 配置`POP3`服务器

```bash
# 安装 dovecot
[root@rhel2 ~]# yum install -y dovecot
# 修改配置文件
[root@rhel2 ~]# grep "^[^#]" /etc/dovecot/dovecot.conf 
protocols = imap pop3 lmtp
disable_plaintext_auth = no
login_trusted_networks = 0.0.0.0/0
# 配置邮件格式与存储位置
[root@rhel2 ~]# grep "^mail_location" /etc/dovecot/conf.d/10-mail.conf 
mail_location = mbox:~/mail:INBOX=/var/mail/%u
# 创建邮件的存储目录
[root@rhel2 ~]# su - test_mail_user
[test_mail_user@rhel2 ~]$ mkdir -p mail/.imap/INBOX
[test_mail_user@rhel2 ~]$ exit
logout
# 重启dovecot服务
[root@rhel2 ~]# systemctl restart dovecot
[root@rhel2 ~]# systemctl enable dovecot
ln -s '/usr/lib/systemd/system/dovecot.service' '/etc/systemd/system/multi-user.target.wants/dovecot.service'
# 防火墙放行
[root@rhel2 ~]# firewall-cmd --add-service=smtp --permanent 
success
[root@rhel2 ~]# firewall-cmd --add-port=110/tcp --permanent 
success
[root@rhel2 ~]# firewall-cmd --reload
success
```

### 在物理机上打开`Foxmail`进行配置

```markdown
1. 输入 用户名密码：test_mail_user@node.com  test_mail_user
2. 输入服务器IP两个都是`RHEL2`: 192.168.143.11
```

- 在物理机发送一封邮件给`root@node.com` cc `test_mail_user@node.com`

```bash
[root@rhel2 ~]# mail 
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 8 messages 4 new 7 unread
    1 kane@localhost.local  Mon Oct 28 19:18  17/723   "*** SECURITY informat"
 U  2 kane@localhost.local  Mon Oct 28 19:19  17/722   "*** SECURITY informat"
 U  3 root                  Thu Nov 28 06:24  22/742   "from rhel1 teset"
 U  4 root                  Thu Nov 28 06:25  22/721   "11"
>N  5 root                  Thu Nov 28 06:26  21/740   "From control1.node.co"
 N  6 root                  Thu Nov 28 06:34  21/711   "ttt"
 N  7 root                  Thu Nov 28 06:34  22/711   "12"
 N  8 test_mail_user@node.  Thu Nov 28 19:46  49/1965  "Test Mail From Window"
& 8
Message  8:
From test_mail_user@node.com  Thu Nov 28 19:46:47 2019
Return-Path: <test_mail_user@node.com>
X-Original-To: root@node.com
Delivered-To: root@node.com
Date: Fri, 29 Nov 2019 11:46:47 +0800
From: "test_mail_user@node.com" <test_mail_user@node.com>
To: root <root@node.com>
Cc: test_mail_user <test_mail_user@node.com>
Subject: Test Mail From Windows
X-Priority: 3
X-Has-Attach: no
X-Mailer: Foxmail 7.2.14.409[cn]
Content-Type: multipart/alternative;
	boundary="----=_001_NextPart204586814442_=----"
Status: R

Content-Type: text/plain;
	charset="us-ascii"

Hi root
    
    This is a test mail from Foxmail.



test_mail_user@node.com
```

- 在物理机上同样也可以收到这封邮件。