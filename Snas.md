# 流网络分析系统-SNAS

SNAS，Streaming Network Analytics System (project SNAS) ，是一个收集、跟踪、存取 千万条实时路由对象的系统。

官网：https://www.snas.io/

## 安装

- docker pull openbmp/aio
- create mysql volume
- docker run

```bash
## 注意需要加上 --privileged=true 否则maraidb创建mysql会失败
docker run -d -e KAFKA_FQDN=127.0.0.1 --privileged=true --name=openbmp_aio  -v /sys/fs/cgroup:/sys/fs/cgroup -e MEM=3 -e MYSQL_ROOT_PASSWORD=OpenBMP\
     -v /var/openbmp/mysql:/data/mysql \
     -p 3306:3306 -p 2181:2181 -p 9092:9092 -p 5000:5000 -p 8001:8001 \
     openbmp/aio /usr/sbin/init
```

- 安装UI 界面

```bash
 docker run -d --name=openbmp_ui  --privileged=true \
     -e OPENBMP_API_HOST=localhost \
     -p 8000:8000 \
     openbmp/ui
```


