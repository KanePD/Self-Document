# Nginx 部署与反向代理配置

最近我们的angular项目部署，我们采用的的是Nginx，下面对Nginx做一个简单的介绍。

## 为什么选择Nginx

- 轻：相比于Apache，同样的web服务器占用的资源少
- 多线程模式：Nginx拥有多个worker进程，处理请求时是异步非阻塞的
- 社区活跃
- 可以做反向代理
- 支持7层负载均衡。[什么是七层负载均衡](https://kb.cnblogs.com/page/188170/)
- 配置简单，易上手。**这才是我们选择的主要原因。**

## 上一个官方下载下来的文档

链接：https://pan.baidu.com/s/1bkbGk8bcZcqe6sVG843WHA 
提取码：eg59 

## Window下 的 Nginx

虽然一般的服务器都不使用windows系统，我们还是先来一段window的

- 访问 ngix[下载页](http://nginx.org/en/download.html)下载windows 版本的 ngix压缩包
- 解压到相应的目录下。
- 打开CMD， cd到解压ngix的目录下，键入 **start nginx.exe**
- 打开浏览器，输入localhost/127.0.0.1
- 上图：

![ngix_1](C:\Users\kanewang\Desktop\ngix_1.png)

- 常用命令：

```CMD
nginx -s stop                  #停止nginx
nginx -s reload                #重新加载nginx配置
nginx -s reopen				   #重新启动
nginx -s quit                  #退出nginx
```

## Linux(CentOS) 下的 Nginx

下面是我们真正使用的Linux 下 搭建Nginx，演示时我使用的WM Ware创建的虚拟机。使用putty进行远程连接。注：如果使用服务器操作的话，粘贴可就麻烦了，所以还是用远程连接吧，能直接copy paste命令

### 菜鸟教程的方法

直接上链接。[Nginx安装](http://www.runoob.com/linux/nginx-install-setup.html)

### 官方文档上的方法

- cd 到 yum的资源目录下

```linux
 cd  /etc/yum.repos.d/
```

- 创建一个文件：nginx.repo，

```linux
vi nginx.repo
# 内容
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
# 文档上的url是http://nginx.org/packages/mainline/OS/OSRELEASE/$basearch/
# 替换成你的 os 与 版本
# 保存退出
```

- 键入如下一系列命令

```linux
yum -y install nginx    # 安装
systemctl enable nginx  #开机自启
systemctl start nginx   #启动nginx
firewall-cmd --permanent --zone=public --add-port=80/tcp #永久开启80端口
firewall-cmd --reload   #重新加载防火墙
```

- 下面我们回到物理机，测试一下虚拟机上的Nginx 服务是否安装成功。在物理机打开浏览器，键入：**虚拟机IP:80**,上图：

![nginx_2](C:\Users\kanewang\Desktop\nginx_2.png)

- 常用命令与Windows相同。
- 个人建议使用官方上的配置。

## 说明Angular 项目的打包，并部署到虚拟机的Nginx

- 在本地找了一个angular项目目录下 ng-build,会生一个dist文件夹
- 键入如下命令：**nginx -t**

```linux
nginx -t #查看配置文件路径
#结果
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```

```conf
#查看上述路径的文件
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

- 注意上面最后一句话包含conf.d文件夹下的所有.conf。我们再那个文件夹下找到了default.conf ,编辑default.conf

```conf
server {
    listen       80;
    server_name  localhost;
    location / {
        #root   /usr/share/nginx/html;
        root   /usr/share/nginx/html/dist/demo;
        #更改成我们上传的目录一定要写到有index.html那一级
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}


```

- 重新载入Nginx配置

```linux
nginx -s reload
```

- 上对比图

![nginx_3](C:\Users\kanewang\Desktop\nginx_3.png)

## 配置一个简单的反向代理

前端需要调用后端的Rest API，我们需要将一部分请求配置反向代理。

- 直接上conf配置

```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html/dist/demo;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
        # 匹配到/proxy/这个url的时候代理到220.181.112.244 百度这个服务器
    location ^~ /proxy/ {
        proxy_set_header Host 220.181.112.244;
        proxy_set_header X-Real-IP  220.181.112.244;
        proxy_pass http://220.181.112.244/proxy/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

- 在重新载入配置前，我们先尝试一下，上图：

注：本地项目，没有配置路由，所有会没有proxy这个东西，会报404错误，我们可以通过查看当前404是哪个服务器包的错，来判断是否发生反向代理

从图中可以看出，此时没有进行反向代理，在虚拟机的服务器上提示404

![nginx_4](C:\Users\kanewang\Desktop\nginx_4.png)

**注：**这里说明一点，就是即便发生了法相贷，但是network中的显示还是我的虚拟机的ip，所以不能当做是否发生反向代理的标注

- 重新载入Nginx配置

```
nginx -s reload
```

- 刷新刚才的页面，上图：可以发现，已经代理到百度的错误页面上去了。

![nginx_5](C:\Users\kanewang\Desktop\nginx_5.gif)

- 一个简单的反向代理就配置好了。