[TOC]

# 防火墙

## 基本介绍

使用`firewalld`对防火墙进行管理。`firewalld`是`Red Hat Enterprise Linux 7`用于管理主机级别防火墙的默认方法。

注：`firewalld.service`  与 `iptables.service`、`ip6tables.service`、`ebtables.service`服务彼此冲突，所以为了防止意外启动其中一个，可以把他们`mask`掉

`systemctl mask iptables`

`systemctl mask ip6tables`

`systemctl mask ebtables`

## `firewalld` 区域`zone`

`firewalld`将传入的流量划分成不同的`区域zone`，不同的zone可以配置不同的规则。

- 哪个`zone`的规则在起作用？

```markdown
`按照如下的规则匹配，优先级依次降低`
1. 如果传入包的原地址与区域中某个原规则设置相匹配，该包将通过该`zone`路由
2. 如果包的传入的接口与`zone`的过滤器设置匹配，该包将通过该`zone`路由
3. 如果都没匹配的话会使用默认的`zone`，默认安装时的默认`zone`是`public`可以更改
```

- `firewalld`区域的默认配置

```markdown
# trusted
允许所有流量传入
# home
除非与传出流量相关，或与ssh\mdns\ipp-client\samba-client\dhcpv6-client预定义服务匹配否则拒绝传入流量
# internal
开始时与`home`配置相同
# work
除非与传出流量相关，或与ssh\ipp-client\dhcpv6-client预定义服务匹配否则拒绝传入流量
# public
除非与传出流量相关，或与ssh\dhcpv6-client预定义服务匹配否则拒绝传入流量
```

## 管理`firewalld`

```markdown
1. 使用命令行工具`firewalld-cmd`。需要安装推行界面
2. 使用图形工具`firewall-config`
3. 直接更改`/etc/firewalld`配置文件。不建议。
```

- `firewall-cmd`命令介绍

| 命令                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| --get-default-zone       | 查询当前默认区域                                             |
| --set-default-zone=      | 设置默认区域                                                 |
| --get-zones              | 列出所有可用区域                                             |
| --get-services           | 列出所有预定义服务                                           |
| --get-active-zones       | 列出当前正在使用的所有区域                                   |
| --add-source=<CIDR>      | 将来自IP地址或网络、子网掩码的所有流量指定到区域             |
| --remove-source          | 删除                                                         |
| --add-interface          | 将来自<INTERFACE>的流量路由到指区域，interface就是虚机的链接的网卡 |
| --change-interface       | 修改                                                         |
| --list-all --zone=<zone> | 列出所有配置默认`zone`可指定                                 |
| --list-all-zones         | 列出所有的zone的配置                                         |
| --add-service            | 允许某个service的流量                                        |
| --remove-service         | 删除                                                         |
| --add-port               | 增加端口。格式[222/tcp]                                      |
| --remove-port            | 删除                                                         |
| --reload                 | 重新加载防护墙配置                                           |
| --permanent              | 永久修改防护墙配置                                           |

- 注意

修改防火墙配置一定要有两步`临时修改`与`永久修改`

```bash
1. 永久修改一次临时修改一次
firewall-cmd --set-default-zone 
firewall-cmd --set-default-zone trusted --permanent
2. 永久修改一次，reload一下
firewall-cmd --set-fefault-zone trusted --permanent
```

- 做的实验当中，无论启动了什么服务，都需要在防火墙中添加，还有一些端口也需要添加

## 关于富规则

### 定义

富规则为管理员提供了一种表达性的语言，来定义防火墙自身带有之外的规则。例如：`允许单IP地址` 。富规则可以表达`允许、拒绝`规则，也可以`配置记录`、`端口转发`、`伪装`、`速率限制`等。

### `firewalld`操作富规则的命令

```markdown
1. --add-rich-rule
2. --remove-rich-rule
3. --query-rich-rul
4. --list-rich-rules
```

### 语法

可查看 `man 5 firewalld.richlanguage`

- 基本规则

```bash
General rule structure

    rule
    [source]
    [destination]
    service|port|protocol|icmp-block|masquerade|forward-port
    [log]
    [audit]
    [accept|reject|drop]
```

- 说明

```
1. rule:
	rule [family="ipv4|ipv6"]
2. [source] 可选,链接的来源ip
	source address="ip/mask"
3. [Destination]可选，目的
	destination address="ip/mask"
4. service|port|protocol|icmp-block|masquerade|forward-port
	必选服务、端口、协议、伪装、端口转发
5. [log] 可选，
	记录进来的链接[prefix=前缀[level=级别limit value=rate/duration频次每duration最多rate次
6. [audit] 
7. [accept|reject|drop]
	接受、拒绝、丢弃
```

## 常用的示例

- 监听服务`httpd`，并输出log

```bash
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.143.11/24 service name="http" log level=notice prefix="Test log" limit value="1/s" accept'
firewall-cmd --reload
# 在服务器192.168.143.11上 curl 192.168.143.10
[root@rhel2 ~]# curl 192.168.143.10
# 检测192.168.143.10 /var/log/messages
[root@rhel1 Desktop]# grep "Test log" /var/log/messages
Nov 25 19:22:20 rhel1 kernel: Test logIN=eno16777736 OUT= MAC=00:0c:29:fb:7d:19:00:0c:29:10:6a:95:08:00 SRC=192.168.143.11 DST=192.168.143.10 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=28126 DF PROTO=TCP SPT=35439 DPT=80 WINDOW=14600 RES=0x00 SYN URGP=0 
```

- 禁止刚才的`ip`访问10的http服务

```bash
# 删除刚才的富规则，添加如下规则、
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.143.11/24 service name="http" reject'
firewall-cmd --reload
# 再通过11服务器访问
[root@rhel2 ~]# curl 192.168.143.10
curl: (7) Failed connect to 192.168.143.10:80; Connection refused
```

- `IP`伪装，将内部IP伪装成某个IP发送出去

```bash
# 将局域网中的 IP `192.168.143.10[RHEL1]` 伪装成`192.168.245.11[RHEL2]`发送出去
firewall-cmd --permanent --add-rich-rule="rule family=ipv4 source address=192.168.143.12/24 masquerade"
firewall-cmd --reload
# 我们在rhel1上 ssh到 192.168.245.10[RHEL3]这个ip上
# 我们在RHEL3上对 ssh服务做了log
[root@rhel3 ~]# grep "SSH" /var/log/messages
Nov 25 19:05:05 rhel3 kernel: SSH LOG:IN=eno50332216 OUT= MAC=00:0c:29:07:88:32:00:0c:29:10:6a:a9:08:00 SRC=192.168.245.11 DST=192.168.245.10 LEN=60 TOS=0x00 PREC=0x00 TTL=63 ID=63405 DF PROTO=TCP SPT=46855 DPT=22 WINDOW=14600 RES=0x00 SYN URGP=0
# 可以看到SRC是 192.168.245.11 
```

- 端口转发，将外部进来的链接转发到别处

```bash
# 在RHEL2上提供httpd服务
## 别忘了防火墙放行
[root@rhel2 html]# curl localhost
rhel2
# 在RHEL3 上同样提供httpd服务
## 别忘了防火墙放行
[root@rhel3 html]# curl localhost
rhel3
# 在RHEL2上 开启端口转发
[root@rhel2 ~]# firewall-cmd --add-forward-port=port=80:proto=tcp:toport=80:toaddr=192.168.245.10
success
# 在RHEL1上访问 
[root@rhel1 Desktop]# curl 192.168.140.11
rhel3
```

# 网络合作

通过网络合作，以一种逻辑方式将两块网卡绑在一起对外服务，实现故障转移与更高的吞吐量。这个逻辑方式就是`网络组`

## 链路聚合

#### 在`RHEL1`上再准备两块网卡

```bash
[root@rhel1 Desktop]# nmcli device show 
GENERAL.DEVICE:                         eno67109440
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:0C:29:FB:7D:37
GENERAL.MTU:                            1500
GENERAL.STATE:                          30 (disconnected)
GENERAL.CONNECTION:                     --
GENERAL.CON-PATH:                       --
WIRED-PROPERTIES.CARRIER:               on

GENERAL.DEVICE:                         eno83886664
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:0C:29:FB:7D:41
GENERAL.MTU:                            1500
GENERAL.STATE:                          30 (disconnected)
GENERAL.CONNECTION:                     --
GENERAL.CON-PATH:                       --
WIRED-PROPERTIES.CARRIER:               on

```

#### 配置网络组`team0`

- 网络组使用的运行程序

```bash
# 创建组接口时通过`config`指定
## 语法
config '{"runner":{"name":"METHOD"}}'
## 类型
broadcast:广播模式，只有冗余机制，有点浪费资源
roundrobin:轮询调度，把请求轮流费赔给内部服务器
activebackup:主备，只有一个内部服务器保持工作,故障转移
loadbalance:负载均衡，监控流量尝试在选择传输端口的时候达到完美均衡
lacp:利用LACP协议进行聚合
```

```bash
# 创建一个主备模式的 网络组
nmcli connection add type team con-name team0 ifname team0 config '{"runner":{"name":"activebackup"}}'
# 分配IP地址
# ip随便配一个
nmcli connection modify team0 ipv4.addresses "192.168.200.100/24" ipv4.method manual
```

- 将两个新的准好的新的两个`接口`添加到`team0`上

```bash
nmcli connect add type team-slave con-name team0-port1 ifname eno67109440 master team0 
nmcli connect add type team-slave con-name team0-port2 ifname eno83886664 master team0
# 查看当前网络组的状态，可以看到当前工作的是 eno67109440
[root@rhel1 Desktop]# teamdctl team0 state
setup:
  runner: activebackup
ports:
  eno67109440
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
  eno83886664
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
runner:
  active port: eno67109440
```
- 验证主备

```bash
# 通过team0 ping本地网关
ping -I team0 0.0.0.0
[root@rhel1 Desktop]# nmcli device disconnect eno67109440 
[root@rhel1 Desktop]# teamdctl team0 state
setup:
  runner: activebackup
ports:
  eno83886664
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
runner:
  active port: eno83886664
# 而另一边的ping命令没有断
```

### 网络组的文件

```bash
[root@rhel1 .ssh]# cat /etc/sysconfig/network-scripts/ifcfg-team0
DEVICE=team0
TEAM_CONFIG="{\"runner\":{\"name\":\"activebackup\"}}"
DEVICETYPE=Team
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=team0
UUID=5d30ce30-7c39-44ea-b170-04dd21d2a3fd
ONBOOT=yes
IPADDR0=192.168.200.100
PREFIX0=24
GATEWAY0=192.168.200.1
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
```

### 网络组命令

```bash
teamnl
teamdctl
```

## 桥接

网桥是一个链路层设备，可基于MAC地址在网络之间转发流量。

```bash
# 再次添加一个网卡
nmcli device show
GENERAL.DEVICE:                         eno100663888
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:0C:29:FB:7D:4B
GENERAL.MTU:                            1500
GENERAL.STATE:                          30 (disconnected)
GENERAL.CONNECTION:                     --
GENERAL.CON-PATH:                       --
WIRED-PROPERTIES.CARRIER:               on
# 增加一个网桥
## 在创建的时候就指定好 ipv4 后来modify不起作用
nmcli connection add type bridge con-name br0 ifname br0 ip4 192.168.150.100/24 gw4 192.168.150.1
# 添加接口
nmcli connection add type bridge-slave con-name br0-port0 ifname eno100663888 master br0 
# 检查是否能ping通本地网关
[root@rhel1 Desktop]# ping -I br0 0.0.0.0
```

