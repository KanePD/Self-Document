# NPM 上传自己的包

项目中经常 npm install,npm init啥的，那么如何上传自己的包到npm上呢。

## 注册账号

[NPM官网](https://www.npmjs.com/)

## 本地弄一个包

- 创建npm_test 文件夹    *注：不能够有一些特殊字符命名文件夹，比如空格*
- cmd cd到目录下，键入 **npm init**
- 键入 **npm login**，输入用户名、密码、邮箱
- 键入 **npm publish**

- 报错：

```CMD
npm ERR! publish Failed PUT 403
npm ERR! code E403
npm ERR! You do not have permission to publish "npm_test". Are you logged in as the correct user? : npm_test

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\kanewang\AppData\Roaming\npm-cache\_logs\2018-12-12T05_30_31_284Z-debug.log
```

- 经过查看愿意你是 npm_test在 库中已经存在了，我没有权限发布，
- 我们直接去package.json中更改

```json
{
  "name": "kane_test",
  "version": "1.0.0",
  "description": "kane test",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "kane",
  "license": "ISC"
}

```

- 再次键入**npm publish**

```CMD
#输出结果
+ kane_test@1.0.0
```

- 换个文件夹，尝试安装这个包，键入 **npm install kane_test**
- 结果：

```cmd
+ kane_test@1.0.0
added 1 package in 2.764s
```

- 当然这个包没有什么用

