[TOC]
# 使用`Gitlab  Runner`实现

## 再要部署的服务器上安装 `gitlab runner`

### 下载可执行文件

```bash
# 按照架构自行选择 本文选择的是  Linux x86-64
# Linux x86-64
wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Linux x86
wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-386

# Linux arm
wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm
```

### 设置可执行权限权限

```bash
chmod +x /usr/local/bin/gitlab-runner
```

### 创建用户

```bash
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

### 运行服务

```bash
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
gitlab-runner start
```

## 注册 `Runner`

### 到`gitlab`上找到需要用的URL与token

- 路径是：`Project-> Settings -> CI/CD -> Runners -> Expand` 

- 截图

![](C:\Users\kanewang\Desktop\setting_cicd.png)

![runners_expand](C:\Users\kanewang\Desktop\runners_expand.png)

![url-token](C:\Users\kanewang\Desktop\url-token.png)

### 在浏览器中下载`gitlab`的ssl证书

1. 点击浏览器链接的左边`锁头`可以下载证书。本文下载的格式是`cer`

2. 将下载好的证书上传到要部署的服务器上。

###  注册`runner`

```bash
gitlab-runner register --tls-ca-file=/home/gitlab-runner/test.cer
# 根据提示 依次输入如下内容
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://example.com/
Please enter the gitlab-ci token for this runner:
1111111111111111111111
Please enter the gitlab-ci description for this runner:
[centos.localdomain]: test
Please enter the gitlab-ci tags for this runner (comma separated):
test
Registering runner... succeeded                     runner=nZsc7EsF
Please enter the executor: docker-ssh+machine, parallels, shell, ssh, virtualbox, docker+machine, kubernetes, custom, docker, docker-ssh:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
#——————————————————————————注——————————————————————————
本文的executor选择的是shell
# 注册成功后再次run一下gitlab-runner
[root@centos target]# gitlab-runner start
# 注册成功后生成 /etc/gitlab-runner/config.toml
[root@centos target]# cat /etc/gitlab-runner/config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "test"
  url = "https://example.com/"
  token = "11111111111111111111111111111"
  tls-ca-file = "/home/gitlab-runner/11111.cer"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

## 在项目中配置`.gitlab-ci.yml`文件

### 本文的`.gitlab-ci.yml`

注： 只做了本次部署的配置，具体其他配置可以查看官网https://docs.gitlab.com/ee/ci/yaml/README.html

```yml
stages:
  - build
before_script:
  - export MVN_HOME  # export Envionment Variable
  - export JAVA_HOME
  - java -version
  - sh /home/gitlab-runner/kill-bgp.sh
# 定义 job
build-bgp-web:
  stage: build      # stage
  tags:
    - first         # runner tag you configured
  only:        
    - test          # branch support regex
  script:           #command
    - mvn clean
    - mvn package
    - cd ./target
    - nohup java -Xms3g -Xmx3g -jar  BGP-new-0.0.1-SNAPSHOT.jar --server.port=9999 > bgp.log 2>&1 &
```

### 验证 `.gitlab-ci.yml`的正确性

可以在`gitlab`上使用`CI Lint`验证上面`yml`文件的正确行。`CI Lint`在 `CI/CD`里面。下面是我的验证结果。

```yml
Status: syntax is correct
Parameter 	Value
Build Job - build-bgp-web 	

export MVN_HOME
export JAVA_HOME
java -version
sh /home/gitlab-runner/kill-bgp.sh
mvn clean
mvn package
cd ./target
nohup java -Xms3g -Xmx3g -jar  BGP-new-0.0.1-SNAPSHOT.jar --server.port=9999 > bgp.log 2>&1 &


Tag list: first
Only policy: refs, test
Except policy:
Environment:
When: on_success 
```

## 注意事项

1. `.gitlab-ci.yml`文化中指定的 runner tag一定要存在[否则找不到runner会一直pending]
2. 出现报错`fatal: unable to access  'https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab.x.com/root/cmop.git/': Peer's Certificate issuer is not recognized. `

```bash
# 关闭ssl校验
[root@gitlab-runner ~]# su - gitlab-runner
[gitlab-runner@gitlab-runner ~]$ git config --global http."sslVerify" false
# 查看
[gitlab-runner@gitlab-runner ~]$ cat /home/gitlab-runner/.gitconfig 
[http]
    sslVerify = false 
```



## 查看一下最使用情况

Pipelines

<img src="C:\Users\kanewang\Desktop\pipelines.png" alt="pipelines" style="zoom:50%;" />

Jobs

![jobs](C:\Users\kanewang\Desktop\jobs.png)

失败会收到邮件

![](C:\Users\kanewang\Desktop\failed.png)

# 使用 Jenkins 实现

## 下载/使用`jenkins`

```markdown
1. 访问 ： https://jenkins.io/download/。本文采用的使用是`war`包安装

2. 下载： `wget http://ftp-chi.osuosl.org/pub/jenkins/war-stable/2.190.2/jenkins.war`

3. 运行 ：`nohup java -Dhudson.util.ProcessTree.disable=true -jar jenkins.war --httpPort=8888 > jenkins.log 2>&1 &`
注： `-Dhudson.util.ProcessTree.disable=true`参数很重要，为了不让jenkins杀掉job创建的进程。如果不加的话，即便是`nohup`执行的命令也会在job执行之后杀掉。
```

## 访问 `ip:8888`进行初始化设置

注：如果之前安装过`jenkins`，会自动升级，并保留之前的数据

### 使用初始密码登录

```bash
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
# 使用下面的密码进入，实际是一个md5的串
1111111111111111111111111
```

### 选择`plugins`

本文直接选择了 `suggestions plugins`，部分插件安装失败可以直接跳过。

### 配置密码

重新设置一个密码

```markdown
# 忘记秘密：
到`/root/.jenkins/users` `admin`用户下找到config.xml，修改下面的内容
<passwordHash>#jbcrypt:$2a$10$MiIVR0rr/UhQBqT.bBq0QehTiQVqgNpUGyWW2nJObaVAM/2xSQdSq</passwordHash>
这个密码是`123456`
```

## 在`jenkins`中配置`Gitlab`实现自动部署

### 安装`gitlab`插件

```markdown
1. 依次访问：`Manage Jenkins`->`Manage Plugins`
2. 在：`Available`中搜索`Gitlab`，安装`Gitlab`插件
3. 等待安装。
```

### 配置`gitlab`连接[可不做]

```markdown
1. 依次访问：`Manage Jenkins`->`Configure System`
2. 找到 Gitlab 标签页
3. 依次填入`Connection name`,`Gitlab host URL`
4. 添加一个`Credntials`,选择`Gitlab API token`
5. 填入在gitlab上设置的`Personal Access Tokens`
6. 点开`Advanced`,勾上`Ignore SSL Certificate Errors`
注：如果不勾上步骤6的话，gitlab是https的将会不成功。
7. 点击`Test Connnection`
8. 保存
```

### 创建一个`Freestyle project`的Job`test`

此时可以先不做任何配置，直接保存。

### 配置`test`job `gitlab`仓库

```markdown
1. 在job`test`页面点击`Configure`
2. 找到`Source Code Management`选择`Git`
3. 配置`Repository URL`并添加一个`Credentials`
注：此处想使用`Personal Access Tokens`添加不上，最后使用的账号密码
4. 保存
```

### 配置`test` job ` Triggers `

```markdown
1. 在job`test`页面点击`Configure`
2. 找到`Build Triggers`选择`Build when a change is pushed to GitLab`其他默认就行
3. 保存
```

### 配置`webhook`

```markdown
1. 到`Gitlab`项目页面->`Settings`->`Integrations`
2. 输入上一步配置后面的`url`
3. 返回错误`Url is blocked: Requests to the local network are not allowed`
4. 上面的解决办法：
管理员账号登录gitlab,在Admin area中，左侧Settings -> Network -> Outbound requests，勾选Allow requests to the local network from hooks and services
但是没有gitlab管理员权限，我们将采取别的办法
```

### 重新配置`test`job ` Triggers `改为`Poll SCM`

```markdown
1. 配置每分钟刷新一次
*/1 * * * *
```

### 配置`bulid`执行命令

```bash
export MVN_HOME # export Envionment Variable
export JAVA_HOME
java -version
sh /home/gitlab-runner/kill-bgp.sh
cd /root/.jenkins/workspace/test
mvn clean
mvn package
cd ./target
nohup java -Xms3g -Xmx3g -jar  BGP-new-0.0.1-SNAPSHOT.jar --server.port=9999 > bgp.log 2>&1 &
# 这些也可以写到一个shell脚本中，jenkins调用脚本
```

## Push一次代码查看一下

![jenkins](C:\Users\kanewang\Desktop\jenkins.png)



# 对比

- `gitlab-ci`

1. 上手简单
2. 与`gitlab`完美兼容
3. 没有web页面，但是`gitlab`有提供
4. 需要自己配置编译环境

- `jenkins`

1. 上手简单
2. 需要配置`webhook`,或者像本文一样轮询
3. 有自己的web页面
4. 有丰富的插件，功能强大
5. 编译环境例如`jdk` `mvn`可以在设置中配置，不需要构建