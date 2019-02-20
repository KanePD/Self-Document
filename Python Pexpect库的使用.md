# Python Pexpect库的使用

## 简介

最近需要远程操作一个服务器并执行该服务器上的一个python脚本，查到可以使用Pexpect这个库。记录一下。

*什么是Pexpect？Pexpect能够产生子应用程序，并控制他们，并能够通过期望模式对子应用的输出做出反应。Pexpect允许你的脚本产生子应用、控制他们像一个人类在输入命令一样。*

*Pexpect使用在自动交互的应用，例如SSH、SFTP、PASSWD、TELNET。它可以被应用在使用自动设置脚本为不同的服务器自动地重复的安装软件包。也可以被应用在自动的软件测试。*

*Pexpect的主要特点是需要Python的基本库pty，这个库只有在类Unix系统上才有* 

##  Pexpect关于SSH的使用

注：测试，我们直接用虚拟机本机ssh本机

- 环境

```txt
1. win10 物理机
2. Vmware Centos 虚拟机
3. Xshell
4. 虚拟机python安装pexpect:pip install pexpect
```

- 在虚拟机创建一个 python文件

```python
#-*- coding:UTF-8 -*-
import pexpect
# 定义ssh连接
def ssh(user,host,password,command):
    #创建子应用，命令是 ssh -l root 127.0.0.1 python /home/python/test.py
    child = pexpect.spawn('ssh -l %s %s %s'%(user,host,command))
    # 期待开启的子程序的显示，子程序的不同显示会匹配到不同key然后我们定义不同的操作
    # 0 ： 连接超时
    # 1 ：ssh有时候提示你是否确认连接
    # 2 ：提示输入密码
    # 3 ：匹配到#号，表示命令已经执行完毕。没用到
    i = child.expect([pexpect.TIMEOUT, 'Are you sure you want to continue connecting','password:',r"([^-]>|#)"])
    # 如果登录超时，renturn none
    if i == 0: # Timeout
        print "Timeout"
        return None
    # 提示是否确认连接
    if i == 1: 
        child.sendline ('yes')  # 我们输入yes
        child.expect ('password: ')# 输入yes后 子程序应该提示输入密码，我们再次期待password
        i = child.expect([pexpect.TIMEOUT, 'password: '])
        #超时
        if i == 0: # Timeout
                return None
    # 不考虑其他情况，走到此处时，要么timeout 已经return ，要么等待输入密码
    #输入密码
    child.sendline(password)
    # 返回子程序
    return child
if __name__ =='__main__':
    try:
        # 配置数据
        host='127.0.0.1'
        user="root"
        password = '********'
        command = 'python /home/python/test.py'
        #command="ls -l"
        child = ssh(user,host,password,command)
        #这句是将子程序的命令行拉到末端
        test = child.expect(pexpect.EOF)
        #child中before就是我们要的数据，有时候还会在 after中
        print child.before
        print child.after
    except Exception,e:
        print str(e)
# 最终的显示结果是 test.py中打印的hahaha结果，
[root@localhost python]# python test_pexpect.py 
 
hahaha


<class 'pexpect.exceptions.EOF'>
```

- 我们尝试一下开两个虚拟机的情况

```python
上面的代码只需要更改ip user password即可
# ip 192.168.233.133
# user root
# 在另一台虚拟机的相同位置创建/home/pyhton/test.py 内容如下
if __name__=="__main__":
    print "another virual machine hahaha"
# 打印结果
[root@localhost python]# python test3.py
 
another virual machine hahaha

<class 'pexpect.exceptions.EOF'>
```

## Pexpect 关于 SFTP的使用

与ssh相同，就是使用python在当前机器上输入sftp ip 然后期望结果，输入密码，并发送get下载文件即可。

**注：使用的时候发现一点注意：在每次执行sendline之前 都需要重新期望一下当前的sftp>，或者在每次输入sendline之后重新期望一下sftp>。也就是期望到这行，否则输入的命令都没有反应，我理解是远程连接的服务器有输出时候当前的位置可能不在sftp>这里所以在sendline的任何东西都是无意义的。如果这个解释不对望高人指点一下，**

```python
# --*-- coding:utf-8 --*--
import pexpect
import os
import time
def sftp(ip , password , command):
                # 创建子应用
                child = pexpect.spawn("sftp %s"%(ip))
                i = child.expect([pexpect.TIMEOUT,'password:'])
                # 超时
                if i == 0 :
                        print "Timeout"
                        return None
                # 准备输入密码
                if i == 1 :
                    	# 输入密码
                        child.sendline(password)
                        j = child.expect([pexpect.TIMEOUT,'sftp>'])
                		# 超时
    					if j == 0:
                                print "Timeout"
                                return None
                        # 匹配到进入sftp命令模式
                        if j==1:
                                print 'Before sftp get command'
                                print child.before
                                print "-----------------"
                                #发送命令
                                child.sendline(command)
                                child.expect(['sftp>'])
                                print "After sftp get command"
                                print child.before
                                print "-----------------"
                                child.sendline("bye")
                                #child.expect(['sftp>'])
                                print "After sftp bye"
                                print child.before
                                print "-----------------"
                                print child.after
                                return child
if __name__=='__main__':
                ip = "192.168.233.133"
                command = "get /home/python/test.txt"
                password = "********"
                child = sftp(ip , password , command)
                print child.before
                print child.after
                if os.path.exists("./test.txt"):
                                print "Make sure transfer successfully"
                else :
                                print "Can not find the transfer file"

# ----------------------------结果-----------------------------------------------
'''
Before sftp get command
 
Connected to 192.168.233.133.

-----------------
After sftp get command
 get /home/python/test.txt
Fetching /home/python/test.txt to test.txt
/home/python/test.txt                         100%   73    25.2KB/s   00:00    

-----------------
After sftp bye
 get /home/python/test.txt
Fetching /home/python/test.txt to test.txt
/home/python/test.txt                         100%   73    25.2KB/s   00:00    

-----------------
sftp>
 get /home/python/test.txt
Fetching /home/python/test.txt to test.txt
/home/python/test.txt                         100%   73    25.2KB/s   00:00    

sftp>
Make sure transfer successfully
'''
```

