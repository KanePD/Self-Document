# Docker 搭建 Tomcat + Mysql

## 准备

1. 虚拟机
2. 虚拟机安装Docker

## 在纯净的Centos镜像上搭建

### Centos镜像准备

- 虚拟机上拉取 Centos 镜像： docker pull centos
- 创建一个容器运行Centos镜像：docker run -it -d --name mycentos centos /bin/bash

**注：**这里遇到了一个错误【**WARNING: IPv4 forwarding is disabled. Networking will not work.**】

```remark
更改虚拟机文件：vim  /usr/lib/sysctl.d/00-system.conf
添加如下内容
net.ipv4.ip_forward=1
重启网络：systemctl restart network
```

**注：**这里又衍生一个问题，docker中systemctl无法正常使用。在官网找到如下解决办法

链接：https://forums.docker.com/t/systemctl-status-is-not-working-in-my-docker-container/9075/4

```bash
run 镜像的时候用如下语句
docker run --privileged  -v /sys/fs/cgroup:/sys/fs/cgroup -it -d --name usr_sbin_init_centos centos /usr/sbin/init
#注意几点 
#1. 必须有--privileged
#2. 必须有-v /sys/fs/cgroup:/sys/fs/cgroup
#3. 将bin/bash替换成 /usr/sbin/init
```

最后终于能够正常运行起来一个Centos镜像了。

### 安装JAVA 环境

- 准备JDK tar包上传到 虚拟机中
- 使用docker cp 将tar包放入docker容器中

```bash
docker cp  jdk-11.0.2_linux-x64_bin.tar.gz 41dbc0fbdf3c:/
#与linux cp指定用法相同，需要加上container的标识：id或者name
```

- 解压tar包

```bash
tar -xf jdk-11.0.2_linux-x64_bin.tar.gz  -C /usr/local/java/jdk
```

- 编辑profile文件 export java环境变量

```config
# /etc/profile
export JAVA_HOME=/usr/local/java/jdk/jdk1.8.0_91
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

- 运行 source /etc/profile，使环境变量生效
- 检测是否成功

```bash
java --version 
#结果
java 11.0.2 2019-01-15 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.2+9-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.2+9-LTS, mixed mode)
```

### 安装Tomcat

- 准备好tomcat tar 包上传到虚拟机，并cp到docker容器中
- 解压到

```
tar -xf apache-tomcat-8.5.38.tar.gz  -C /usr/local/tomcat
```

- 设置开机启动，通过使用rc.local文件实现

```bash
#rc.local 增加如下代码
export JAVA_HOME=/usr/local/java/jdk/jdk-11.0.2
/usr/local/tomcat/apache-tomcat-8.5.38/bin/startup.sh
```

- 开启tomcat 

```bash
#到/usr/local/tomcat/apache-tomcat-8.5.38/bin/目录下 运行
./startup.sh
```

- 检测

```bash
curl localhost:8080
#返回html源码内容
```

### 安装mysql

- 获取 mysql 的yum源

```bash
 wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

- 安装上面的yum 源

```bash
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

- yum 安装 mysql

```bash
yum -y install mysql-community-server
```

- 更改mysql 配置:/etc/my/cnf

```bash
validate_password=OFF # 关闭密码验证
character-set-server=utf8 
collation-server=utf8_general_ci
log-error=/var/log/mysqld.log 
pid-file=/var/run/mysqld/mysqld.pid
[client]
default-character-set=utf8
```

- 获取mysql 初始密码

```bash
grep "password" /var/log/mysqld.log
#结果：[Note] A temporary password is generated for root@localhost: k:nT<dT,t4sF
#使用这个密码登录mysql
```

- 进入到mysql，进行操作

```bash
# 进入
mysql -u root -p 
#更改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '111111';
# 更改 使mysql可以远端访问
update user set host = '%' where user = 'root';
```

- 测试，可以使用物理机，使用navicat 对docker中的mysql进行访问

### 打包容器

- 放到docker hub上

```bash
# 将容器提交成  镜像
docker commit -a 'kane' -m 'test' container_id images_name:images_tag
# 推到dockerhub
docker push kane0725/tomcat
```

- 到处本地tar包

```bash
# 导出打成本地 tar 包
docker export -o test.tar a404c6c174a2
# 将 tar 包导入成镜像
docker import test.tar test_images
```

## 使用Dockerfile

注：只搭建一个tomcat的镜像

### 准备工作

1. 创建一个专门的文件夹，放入jdk 与tomcat的 tar包
2. 在这个目录下创建Dockerfile文件
3. centos 基础镜像

### 文件内容

```bash
FROM centos
MAINTAINER tomcat mysql
ADD jdk-11.0.2 /usr/local/java
ENV JAVA_HOME /usr/local/java/
ADD apache-tomcat-8.5.38 /usr/local/tomcat8
EXPOSE 8080
```

### 使用docker build输出结果

```bash
[root@localhost dockerfile]# docker build -t tomcats:centos .
Sending build context to Docker daemon 505.8 MB
Step 1/7 : FROM centos
 ---> 1e1148e4cc2c
Step 2/7 : MAINTAINER tomcat mysql
 ---> Using cache
 ---> 889454b28f55
Step 3/7 : ADD jdk-11.0.2 /usr/local/java
 ---> Using cache
 ---> 8cad86ae7723
Step 4/7 : ENV JAVA_HOME /usr/local/java/
 ---> Running in 15d89d66adb4
 ---> 767983acfaca
Removing intermediate container 15d89d66adb4
Step 5/7 : ADD apache-tomcat-8.5.38 /usr/local/tomcat8
 ---> 4219d7d611ec
Removing intermediate container 3c2438ecf955
Step 6/7 : EXPOSE 8080
 ---> Running in 56c4e0c3b326
 ---> 7c5bd484168a
Removing intermediate container 56c4e0c3b326
Step 7/7 : RUN /usr/local/tomcat8/bin/startup.sh
 ---> Running in 7a73d0317db3

Tomcat started.
 ---> b53a6d54bf64
Removing intermediate container 7a73d0317db3
Successfully built b53a6d54bf64
```

### docker build的问题

```bash
一定要带上命令后面的  .  否则会报错的
"docker build" requires exactly 1 argument(s).
```

### 运行一个容器

```bash
# 进入容器
docker run -it --name tomcats --restart always -p 1234:8080 tomcats /bin/bash
#运行tomcat startup.sh
/usr/local/tomcat8/bin/startup.sh
#结果
Using CATALINA_BASE:   /usr/local/tomcat8
Using CATALINA_HOME:   /usr/local/tomcat8
Using CATALINA_TMPDIR: /usr/local/tomcat8/temp
Using JRE_HOME:        /usr/local/java/
Using CLASSPATH:       /usr/local/tomcat8/bin/bootstrap.jar:/usr/local/tomcat8/bin/tomcat-juli.jar
Tomcat started.
```

## 使用docker compose

### 安装 docker compose 

官方：https://docs.docker.com/compose/install/

我选择的方式是pip安装

```bash
# 安装
pip install docker-compose
# 检测
docker-compose --version
# -----------------------
docker-compose version 1.23.2, build 1110ad0
```

### 编写docker-compose.yml

```yml
# 这个yml文件 搭建一个mysql 一个 tomcat的容器
version: "3"   
services:
  mysql:
    container_name: mysql
    image: mysql:5.7                          
    restart: always
    volumes:
      - ./mysql/data/:/var/lib/mysql/                             
      - ./mysql/conf/:/etc/mysql/mysql.conf.d/
    ports:
      - "6033:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=********
  tomcat:
    container_name: tomcat
    restart: always
    image: tomcat
    ports:
      - 8080:8080
      - 8009:8009
    links:
      - mysql:m1                                       #连接数据库镜像
```

### 运行命令

注：必须在yml文件的目录下下执行

```bash
docker-compose up -d
# 结果----------查看docker container-------
[root@localhost docker-compose]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS                                             NAMES
1a8a0165a3a8        tomcat              "catalina.sh run"        7 seconds ago       Up 6 seconds                     0.0.0.0:8009->8009/tcp, 0.0.0.0:8080->8080/tcp    tomcat
ddf081e87d67        mysql:5.7           "docker-entrypoint..."   7 seconds ago       Up 7 seconds                     33060/tcp, 0.0.0.0:6033->3306/tcp                 mysql
```





