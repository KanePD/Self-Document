 

# 【免费】自己科学上网，你懂的

## 获取免费得国外得服务器

- 这里一定注意，必须是国外的服务器才可以。经过一番寻找，找到了一个可以免费用一段时间的服务器。

- 网址<https://www.skysilk.com/>

- 名人不说暗话，https://www.skysilk.com/ref/Ee8HhS9APo这个是我的邀请链接，官网得意思是通过我的邀请会给被邀请人100$。但是其实只要注册了就会有，现在得时间是2019年5月23号，以后的政策可能会更改。
- 注册一个账号，推荐使用gmail。我尝试过163邮箱跟qq邮箱都失败了。如果选择第三方登陆github得话如果注册github得邮箱是163得邮箱也会失败。最终我使用了自己的gmail邮箱注册得。
- 登陆成功后，点击右上角的用户名- MY ACCOUNT-Billing，查看一下账户余额是否是100$

## 在服务器上部署**Shadowsocks**

本部分内容参照网址 <https://zoomyale.com/2016/vultr_and_ss> 其中的 【**3 部署 Shadowsocks**】

1. 获得sh脚本，上面网址说明是 [秋水逸冰](<https://teddysun.com/>)提供的。

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh

chmod +x shadowsocks-all.sh

./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

2. 进入到Shadowsocks 的部署过程

```bash
Which Shadowsocks server you'd select:
1) Shadowsocks-Python
2) ShadowsocksR
3) Shadowsocks-Go
4) Shadowsocks-libev
Please enter a number (Default Shadowsocks-Python):4

You choose = Shadowsocks-libev
```

```bash
Please enter password for Shadowsocks-libev
(Default password: teddysun.com):test

password = test
```

```bash
Please enter a port for Shadowsocks-libev [1-65535]
(Default port: 16011):8888

port = 8888
```

```bash
# 此处注意一下，默认情况是 1) aes-256-gcm 但是我找到ios的app 不提供这种加密方式，查看了大概3个链接vpn的ios的app 都有 【7) aes-256-cfb】所以建议选择这个，对于Android机子来说随意，都支持。
Please select stream cipher for Shadowsocks-libev:
1) aes-256-gcm
2) aes-192-gcm
3) aes-128-gcm
4) aes-256-ctr
5) aes-192-ctr
6) aes-128-ctr
7) aes-256-cfb
8) aes-192-cfb
9) aes-128-cfb
10) camellia-128-cfb
11) camellia-192-cfb
12) camellia-256-cfb
13) xchacha20-ietf-poly1305
14) chacha20-ietf-poly1305
15) chacha20-ietf
16) chacha20
17) salsa20
18) rc4-md5
Which cipher you'd select(Default: aes-256-gcm):7

cipher = aes-256-cfb
```

```bash
Do you want install simple-obfs for Shadowsocks-libev? [y/n]
(default: n):y

You choose = y
```

```bash
Which obfs you'd select(Default: http):2

obfs = tls
Press any key to start...or Press Ctrl+C to cancel
```

按任意键后，进入部署，大概耗费了15分钟左右。

3. 检测、查看配置信息

```bash
# 检测状态
[root@myvpn ~]# /etc/init.d/shadowsocks-libev status
Shadowsocks-libev (pid 72678) is running...
# 查看监听状态
[root@myvpn ~]# netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:10956           0.0.0.0:*               LISTEN      72679/obfs-server               
udp        0      0 0.0.0.0:10956           0.0.0.0:*                           72678/ss-server   
# 查看配置
[root@myvpn ~]# cat /etc/shadowsocks-libev/config.json
{
    "server":"**********",
    "server_port":**********,
    "password":"**********",
    "timeout":300,
    "user":"nobody",
    "method":"aes-256-cfb",
    "fast_open":false,
    "nameserver":"8.8.8.8",
    "mode":"tcp_and_udp",
    "plugin":"obfs-server",
    "plugin_opts":"obfs=tls"
}
```

## 开启BBR 进行加速

```bash
# 获取安装脚本，并执行
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh 
chmod +x bbr.sh 
./bbr.

---------- System Information ----------
 OS      : CentOS 7.3.1611
 Arch    : x86_64 (64 Bit)
 Kernel  : 4.15.18-13-pve
----------------------------------------
 Auto install latest kernel for TCP BBR

 URL: https://teddysun.com/489.html
----------------------------------------

Press any key to start...or Press Ctrl+C to cancel
```



## 配置Android机的客户端

直接访问<https://github.com/shadowsocks/shadowsocks-android/releases> 下载shadowsocks*.apk，下载后直接安装。安装成功后，点击【+号-手动设置】输入 服务器ip、端口号、密码、更改加密方式为【AES-256-CFB】主要是与上面部署的时候一致。然后就能科学上网了。

这里我使用的是华为的手机，发现下载twitter、facebook、ins都不能 正常使用，一直显示网络错误，但是用浏览器的时候姐可以正常查看。最后发现google play里面提供了 lite版的是可以正常使用的。

## 配置IOS机的客户端

1. 一个国外的appleid账号，这块应该是将appleid的区域设置成us即可，需要进一步填写信息，瞎填就可以
2. 搜索app【Shadowlink】，如果不是国外的appleid 搜索不到这个app
3. 参照Android机 添加一个连接即可

## 接下来就可以科学上网了。