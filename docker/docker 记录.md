# docker 记录

- 在docker中启动centos容器报错：WARNING: IPv4 forwarding is disabled. Networking will not work. 

 vim  /usr/lib/sysctl.d/00-system.conf 添加 ：net.ipv4.ip_forward=1

重启network ， 删除错误容器重新run

- docker exec -it mycentos bin/bash

- docker run -d -i -t --name mycentos centos /bin/bash

**必须加/bin/bash，否则启动后直接就关闭了即便有-d也不好使**

- docker中service无法启动：Failed to get D-Bus connection: Operation not permitted

亲测好使：创建container的时候使用：**docker run --privileged  -v /sys/fs/cgroup:/sys/fs/cgroup -it -d --name usr_sbin_init_centos centos /usr/sbin/init**

安装service ：yum install initscripts -y

官网：https://forums.docker.com/t/systemctl-status-is-not-working-in-my-docker-container/9075/4



https://blog.csdn.net/honnyee/article/details/81263821

## centos mysql安装

安装成功后

获取密码 ：grep "password" /var/log/mysqld.log

到/etc/my.cfg文件增加：validate_password=OFF，关闭密码检验

进入mysql命令行：mysql -uroot -p 输入初始密码

更改root密码：ALTER USER 'root'@'localhost' IDENTIFIED BY 'cisco123';

远程访问允许：grant all privileges  on *.* to root@'%' identified by "cisco123";

## 安装java

准备tar包上传到服务器上，使用docker cp 命令复制到docker容器中。

docker cp  jdk-11.0.2_linux-x64_bin.tar.gz 41dbc0fbdf3c:/

解压到：tar -xf ‘tar包路径’  -C /usr/local/java/jdk

vi /etc/profile

```config
export JAVA_HOME=/opt/soft/jdk1.8.0_91
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

source /etc/profile

java --version

## 安装tomcat

tomcat自动启动：可以使用/etc/rc.d/rc.local文件 进行开机启动

## docker使用 tar命令

将tar包变成本地images:docker import test.tar test

docker images:查看是否多了个 test的images

docker run 的时候增加命令 https://blog.csdn.net/afr3828/article/details/79090984?utm_source=blogxgwz9

```bash
docker run --privileged -p 8888:8080 -p 3306:3306 -v /sys/fs/cgroup:/sys/fs/cgroup -it -d --restart always --name test test /usr/sbin/init

docker run --privileged -p 9999:8080 -p 3399:3306 -v /sys/fs/cgroup:/sys/fs/cgroup -v /root/test:/usr/local/tomcat/apache-tomcat-8.5.38/webapps -it -d --restart always  --name bpg_container test3 /usr/sbin/init
```

## dockerfile

## docker compose

pip install docker-compose

https://www.cnblogs.com/mushou/p/9537846.html

```yml
version: "3"
services:
  mysql:
    container_name: mysql
    image: mysql:5.7                            #从私有仓库拉镜像
    restart: always                       
    volumes:
      - ./mysql/data/:/var/lib/mysql/                             #映射mysql的数据目录到宿主机，保存数据
      - ./mysql/conf/:/etc/mysql/mysql.conf.d/ #把mysql的配置文件映射到容器的相应目录
    ports:
      - "6033:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=123456
tomcat:
    container_name: tomcat
    restart: always
    image: 192.168.1.30:5000/tomcat
    ports:
      - 8080:8080
      - 8009:8009
    volumes:
      - ./tomcat/conf/:/usr/local/tomcat/conf/  #映射 tomcat的配置文件到容器里
      - ./tomcat/webapps/web:/usr/local/tomcat/webapps/web          #映射一个web服务
      - ./tomcat/logs/:/usr/local/tomcat/logs/
    links:
      - mysql:m1                                                   #连接数据库镜像
```

注：

- volumn里必须是路径，不能指定文件

- tomcat指定外部conf的时候一直创建不成功，不知道为什么，提示

  ```bash
  tomcat    | Feb 20, 2019 2:23:29 AM org.apache.catalina.startup.Catalina load
  tomcat    | WARNING: Unable to load server configuration from [/usr/local/tomcat/conf/server.xml]
  tomcat    | Feb 20, 2019 2:23:29 AM org.apache.catalina.startup.Catalina start
  tomcat    | SEVERE: Cannot start server. Server instance is not configured.
  tomcat exited with code 1
  ```



