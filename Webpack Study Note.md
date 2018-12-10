# Webpack Study Note

创建 文件夹

npm init

npm install webpack --save-dev

将 C:\Users\kanewang\AppData\Roaming\npm 放入环境变量

------

```cmd
$ npm i -g webpack webpack-dev-server
```

```
$ cd webpack-demos
$ npm install
```

```
$ cd demo01
$ npm run dev
```

------

不在包的目录下 进行 webpack 会提示 One CLI for webpack must be installed. These are recommended 

choices, delivered as separate packages:

让我们安装。 需要全局安装 npm install  webpack-cli -g

------

我们可以再root目录下 编写js文件进行webpack

------

为了看起来更整洁，我们再创建的文件夹下进行webpack打包

webpack 不进行配置的时候 默认回去找 src文件夹下的index.js文件   ，打包到 dist文件夹下main.js

也可以输入命令的时候 指定 文件与目标文件位置

也可以创建webpack_config_js文件进行配置

------

阮一锋 demo   git clone https://github.com/ruanyf/webpack-demos.git

+ demo1 单个入口

+ demo2 两个入口

+ demo3  react jsx文件  需要npm loader:Babel-loader【可将将.jsx/.es6文件转换成js文件】

+ demo4 css loader  use里面是从有道做使用的

  - css-loader:sass\less

  - style-loader:将css内嵌到html中
  - postcss-loader:对应插件autoprefixer浏览器兼容性

+ demo5 url loader,加载静态图片

+ demo6 css 加载这个demo 单独讲述了 webpack 使用global定义css 到达全局应用css样式的效果

+ demo7  UglifyJsPlugin插件，可以减少js代码大小，但是会影响webpack编译速度，所以可以使用isProduction变量，在生产环境的时候才使用这个插件

+ demo08 html-webpack-plugin插件 与 open-browser-webpack-plugin拆件的使用
  - html-webpack-plugin：为html引入资源文件、生成入口的html文件
  - open-browser-webpack-plugin：自动打开浏览器，使用 webpack-dev-server --open 如果没加--open可以在config中配置自动打开浏览器，如果都设置会打开两个tab

+ demo09  在webconfig 定义插件，检测当前的模式。在自己写的js文件中打印process.env是空的，在webpackconfig中process.env有很多值，其中DEBUG就是模式的切换

+ demo10  创建打包js的分离点，使用 require.ensure()进行断点的区分，也可以再webpack config文件中配置不同的entry也就是不同的入口文件，第二种方法可以自定义打包后文件位置、文件名啥的，而第一种方法湖自动生成

+ demo11 使用 bundle-loader. 实现按需加载  var load = require('bundle-loader!./a.js'); 将会跟 main.js打包成两个 bundle文件

+ demo12  关于  CommonsChunkPlugin的使用

  在webpack4中 将 插件CommonsChunkPlugin 废弃了，直接config配置就可以

  	optimization: {
  		splitChunks: {      // old CommonsChunkPlugin
  			chunks: "all"
  		},
  	},

- demo13 providePlugin 使用

自定义一个 index1.js 的module export 一个 变量

```javascript
var test = "Hello World";
module.exports = {test};
```

webpack.config.js

```javascript
new webpack.ProvidePlugin({
$:  path.resolve(__dirname, './src/js/index1.js'),
})
// remember that you need give the path to the $ or the webpack will not idetify the module that you define
```

index2.js

```javascript

var content1 = require('./index1').test;
var content = $.test;
console.log(require('./index1'));
console.log($.test);
document.write('<h1>' + content1 + '</h1>');
document.write('<h1>' + content + '</h1>');
// if you use the provideplugin you will not need to require or import every time
```

- demo 14 exposing the variable

- demo15  react 的route 的例子

------

angular框架中的webpack