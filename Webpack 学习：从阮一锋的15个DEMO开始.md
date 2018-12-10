# WebPack 学习：从阮神的15个DEMO开始

## WebPack 是什么

官方就一句话，**打包所有的资源**。

## 从阮神的 15 DEOM入手 Webpack

### Github 地址

[阮神GIT](https://github.com/ruanyf/webpack-demos) 

### 按照 ReadME 操作

- npm webpack-dev-server ，为了能够运行起来demo的代码
- cd 到任何一个demo下，执行npm run dev 即可运行demo

npm run dev 是在 demo下的package.json文件中 配置的 script，实际上是在执行 **webpack-dev-server --open**

### 对DEMO拆解

- 准备工作：
- + 请保证电脑上有npm
  + 创建一个文件夹 webpack-tutorial
  + cmd cd到上面的文件夹，键入 npm init 根据提示步骤创建package.json文件，可以一直enter
  + npm install webpack webpack-cli webpack-dev-server 安装需要的包

- demo1 ：单个入口

  - 创建 webpack.config.js文件，代码如下

  ```javascript
  module.exports = {
    entry: './main.js',//入口文件是当前目录下的 main.js文件
    output: {
      filename: 'bundle.js'//打包后的文件名称是 bundle.js
      //这里webpack会自动创建dist文件夹，将bundle.js放到里面。
      //这里的./会指定到 dist下
      //比如 filename:'./js/bundle.js' 会在dist下找js文件夹放入bundle.js文件
     	//可以通过path指定不同的目录
    }
  };
  ```

  - 创建 main.js文件 ，代码如下：

  ```javascript
  // main.js
  document.write('<h1>Hello World</h1>');//往页面上写hello world
  ```

  - CMD中键入 webpack进行打包，键入进行开启

  ```cmd
  //webpack 输出 
      Asset       Size  Chunks             Chunk Names
  bundle.js  968 bytes       0  [emitted]  main
  Entrypoint main = bundle.js
  ```

  ```npm
  webpack-dev-server --open 
   //打开浏览器后发现是一个ftp的页面
   //我们下面将自动创建index.html文件
  ```

  - CMD 键入

  ```npm
   npm install html-webpack-plugin
   //安装自动生成html插件
  ```

  - 重新编辑 webpack.config.js

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  module.exports = {
    entry: './main.js',//入口文件是当前目录下的 main.js文件
    output: {
      filename: 'bundle.js'//打包后的文件名称是 bundle.js
      //这里webpack会自动创建dist文件夹，将bundle.js放到里面。
      //这里的./会指定到 dist下
      //比如 filename:'./js/bundle.js' 会在dist下找js文件夹放入bundle.js文件
     	//可以通过path指定不同的目录
    },
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
    ]
  };
  ```

  - 重新CMD 键入 webpack 打包 ，键入  webpack-dev-server --open 开启server

  ```cmd
  //webpack输出 
      Asset       Size  Chunks             Chunk Names
   bundle.js  968 bytes       0  [emitted]  main
  index.html  187 bytes          [emitted]
  Entrypoint main = bundle.js
  ```

  会发现 dist 文件夹下自动生成了 index.html并且引入 打包好的 bundle.js文件，浏览器正常显示一个h1的hello world

- demo2 ：两个入口

  - 更改 webpack.config.js 文件如下：

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  module.exports = {
    entry: {
      bundle1: './main1.js',//入口1 main1.js
      bundle2: './main2.js'//入口2 main2.js
    },
    output: {
      filename: '[name].js'//name是entry的键名，最后会生成bundle1.js bundle2.js
    },
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
    ]
  };
  ```

  - 分别创建 main1.js 与main2.js

  ```javascript
  // main1.js
  document.write('<h1>Hello World</h1>');
  // main2.js
  document.write('<h2>Hello Webpack</h2>');
  ```

  - CMD 键入 webpack 打包 ，键入  webpack-dev-server --open 开启server

  ```cmd
  //webpack 输出
       Asset       Size  Chunks             Chunk Names
  bundle1.js  968 bytes       0  [emitted]  bundle1
  bundle2.js  971 bytes       1  [emitted]  bundle2
  index.html  245 bytes          [emitted]
  Entrypoint bundle1 = bundle1.js
  Entrypoint bundle2 = bundle2.js
  [0] ./main1.js 39 bytes {0} [built]
  [1] ./main2.js 41 bytes {1} [built]
  ```

  会在dist下 创建 bundle1.js/bundle2.js/index.html，浏览器会有h1的helloworld h2的hellowebpack

- demo3 ：Babel-loader 的使用

**什么loaders**：就是webpack使用loaders来预处理文件，允许打包除了js文件外任何静态资源。

**什么是Babel-loader**：是javascript编译器，将现行的javascript代码变成浏览器可以兼容的代码。

+ - 例子中是 react 所有我们先安装下相关的包

  ```npm
  npm install react react-dom//js 用到
  npm install babel-loader babel-core babel-preset-es2015 babel-preset-react
  //这里说明一下 babel-loader单独安装一下7.1.5的版本，因为最新的8版本 打包时会报错
  ```

  - 创建main.jsx文件

  ```javascript
  // main.jsx
  //我们需要增加一部分代码
  //增加一个创建wrapper div的过程
  //这边我的index.html是自动创建的，打包后在更改 会报错
  //不知道阮神是怎么弄的，等我研究明白了再回来说明一下
  //如下方法可成功显示
  var div = document.createElement('div');  
  div.id = 'wrapper';  
  document.body.appendChild(div);
  //结束
  const React = require('react');
  const ReactDOM = require('react-dom');
  ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.querySelector('#wrapper')//将h1这个标签放到 id是wrapper的div中
  );
  ```

  - 修改webpack.config.js文件

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  module.exports = {
    entry: './main.jsx',//不说明了
    output: {
      filename: 'bundle.js'
    },
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,//匹配.jsx文件
          exclude: /node_modules/,//不包含这个文件夹 npm包
          use: {
            loader: 'babel-loader',//使用babel-loader这个
            options: {
              presets: ['es2015', 'react']//使用 es2015 跟react
              //这里说明一下，这里的顺序是从右到左加载的，即，先用react再用es2015
            }
          }
        }
      ]
    }
  };
  ```

  - CMD 键入 webpack 打包 ，键入  webpack-dev-server --open 开启server

  ```cmd
  //webpack 输出
       Asset       Size  Chunks                    Chunk Names
   bundle.js    248 KiB       0  [emitted]  [big]  main
  index.html  187 bytes          [emitted]
  ```

- demo4 ：css-loader使用

- - cmd 键入 

  ```npm
  npm install css-loader style-loader
  ```

- - 创建 app.css文件

  ```css
  body {
    background-color: blue;
  }
  ```

  - 更改 webpack.config.js，增加一个css-loader

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  module.exports = {
    entry: './main.js',
    output: {
      filename: 'bundle.js'
    },
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [ 'style-loader', 'css-loader' ]//同样的 从右到左加载
        },
      ]
    }
  };
  ```

  - 更改main.js文件

  ```javascript
  require('./app.css');
  ```

  - CMD 键入 webpack 打包 ，键入  webpack-dev-server --open 开启server

  ```CMD
  //webpack 输出
     Asset       Size  Chunks             Chunk Names
   bundle.js    147 KiB       0  [emitted]  main
  index.html  187 bytes          [emitted]
  ```

- demo5 ：image loader

不做赘述，需要 npm url-loader。与上一个demo类似

- demo6 ： CSS Module

Css Module：给CSS加入了局部作用域和模块依赖。详情还是请看[阮神博客](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)。还有[官网](https://css-modules.github.io/webpack-demo/)。

本次demo 只是介绍了局部作用于与全局作用域。

- - 更改main.jsx文件。同样我们增加一部分代码。

  ```javascript
  //增加start
  var div = document.createElement('div');  
  div.id = 'example';  
  document.body.appendChild(div);
  
  var h1 = document.createElement('h1');  
  h1.className="h1";
  var t1=document.createTextNode("Hello World");
  h1.appendChild(t1);
  document.body.appendChild(h1);
  
  var h2 = document.createElement('h2');
  h2.className="h2";  
  var t2=document.createTextNode("Hello Webpack");
  h2.appendChild(t2);
  document.body.appendChild(h2);
  //end
  var React = require('react');
  var ReactDOM = require('react-dom');
  var style = require('./app.css');
  
  ReactDOM.render(
    <div>
      <h1 className={style.h1}>Hello World</h1>
      <h2 className="h2">Hello Webpack</h2>
    </div>,
    document.getElementById('example')
  );
  ```

  - 更改webpack.config.js

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  module.exports = {
    entry: './main.jsx',//每次看好打包的入口文件呦
    output: {
      filename: 'bundle.js'
    },
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {//使用module
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - 更改app.css

  ```css
  /* local scope */
  .h1 {
    color:red;
  }
  
  /* global scope */
  :global(.h2) {
    color: blue;
  }
  ```

  - CMD 键入 webpack 打包 ，键入  webpack-dev-server --open 开启server

  ```cmd
      Asset       Size  Chunks             Chunk Names
   bundle.js   6.89 KiB       0  [emitted]  main
  index.html  187 bytes          [emitted]
  ```

  - 结果是 有一个红色字体的h1 一个黑色字体的h1 两个蓝色字体的h2， 因为h2 的class是global的 h1的local的

- demo7：使用 uglifyjs 插件

**什么事uglifyjs 插件**：将输出的文件bundle.js变到最小。丑化js代码。

**这里需要注意webpack 两种模式的 产品模式下 uglifyjs插件是默认开启的我们需要在development模式下搞**

- - 更改main.js使用阮神demo中的代码：

  ```javascript
  var longVariableName = 'Hello';
  longVariableName += ' World';
  document.write('<h1>' + longVariableName + '</h1>');
  ```

- - 在webpack.config.js文件中加入如下配置：

  ```javascript
  mode: 'development',//开发者模式
  ```

  - 我们先打包一次main.js打包成的bundle.js的大小为3.89kb，然后使用插件。这个时候bundle.js可读
  - npm install uglifyjs-webpack-plugin，更改webpack.config.js，加一个插件其他不更改

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  
  module.exports = {
    entry: './main.js',//每次看好打包的入口文件呦
    output: {
      filename: 'bundle.js'
    },
    //加入html自动生成
    mode: 'development',//开发者模式
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		new UglifyJsPlugin(),//开发者模式下使用插件
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - CMD 键入 webpack 打包 ，查看bundle.js大小1.23KB

- demo8 ：html-webpack-plugin与open-browser-webpack-plugin两个插件。

html-webpack-plugin：自动生成html插件。这个插件我们已经使用啦，不介绍了。

open-browser-webpack-plugin：自动打开浏览器插件。

- - npm install open-browser-webpack-plugin，更改webpack.config.js如下 

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  var OpenBrowserPlugin = require('open-browser-webpack-plugin');
  module.exports = {
    entry: './main.js',//每次看好打包的入口文件呦
    output: {
      filename: 'bundle.js'
    },
  	mode: 'development',
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		new UglifyJsPlugin(),
  		new OpenBrowserPlugin({
        url: 'http://localhost:8080'//开启服务后自动打开这个网址
      })
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - CMD 键入 webpack 打包 ，键入  webpack-dev-server  开启server，不需要--open了 否则会打开两个页面

- demo9 ：Environment flags

在开发时，有些东西要放出来，产品环境时需要屏蔽掉。我们可以定一个变量去看当前的模式。

- - 更改main.js

  ```javascript
  document.write('<h1>Hello World</h1>');
  
  if (__DEV__) {//__DEV__ 在webpack.config.js中定义
    document.write(new Date());
  }
  ```

  - webpack.config.js

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  var OpenBrowserPlugin = require('open-browser-webpack-plugin');
  
  var devFlagPlugin = new webpack.DefinePlugin({
    __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
  });
  
  
  module.exports = {
    entry: './main.js',//每次看好打包的入口文件呦
    output: {
      filename: 'bundle.js'
    },
  	mode: 'development',
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		new UglifyJsPlugin(),
  		new OpenBrowserPlugin({
        url: 'http://localhost:8080'
      }),
  		devFlagPlugin,
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - webpack 打包，并输入cross-env DEBUG=true webpack-dev-server，来控制debug

  阮神是想通过命令将当前是否是debug传入到程序中，我们更改一下代码。官网上介绍可以使用process.env.NODE_ENV来访问当前的mode。

  - main.js

  ```javascript
  document.write('<h1>Hello World</h1>');
  
  if (process.env.NODE_ENV=="development") {
    document.write(new Date());
  }
  ```

  - 直接看页面。注：更改了config文件需要重新打包其他的不需要。

- demo10 ：code splitting

code splitting：打包时，分文件打包

- - main.js

  ```javascript
  // main.js
  require.ensure(['./a'], function (require) {
    var content = require('./a');
    document.open();
    document.write('<h1>' + content + '</h1>');
    document.close();
  });
  ```

  - 创建a.js

  ```javascript
  // a.js
  module.exports = 'Hello World';
  ```

- - 其他不变，直接webpack打包，多生成了0.bundle.js文件

  ```cmd
  //wepack 输出
       Asset       Size  Chunks             Chunk Names
  0.bundle.js  307 bytes       0  [emitted]
    bundle.js   2.45 KiB    main  [emitted]  main
   index.html  187 bytes          [emitted]
  ```

- demo 11：Code splitting with bundle-loader

- - 这个与demo10 一样，npm install bundle-loader后直接更改 main.js

  ```javascript
  // main.js
  
  // Now a.js is requested, it will be bundled into another file
  var load = require('bundle-loader!./a.js');
  
  // To wait until a.js is available (and get the exports)
  //  you need to async wait for it.
  load(function(file) {
    document.open();
    document.write('<h1>' + file + '</h1>');
    document.close();
  });
  ```

  - 直接webpack打包：效果是相同的

  ```javascript
    Asset       Size  Chunks             Chunk Names
  0.bundle.js  307 bytes       0  [emitted]
    bundle.js   2.45 KiB    main  [emitted]  main
   index.html  187 bytes          [emitted]
  ```

- demo12:Common chunk

是自动将相同代码打包成一个common.js

- - 这里需要说明，阮神的做法在webpack4中已经**废弃**了，现在可以直接使用，更改webpack.config.js

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  var OpenBrowserPlugin = require('open-browser-webpack-plugin');
  var webpack = require('webpack');
  var devFlagPlugin = new webpack.DefinePlugin({
    __DEV__: JSON.stringify(JSON.parse(process.env.NODE_ENV || 'false'))
  });
  
  
  module.exports = {
    entry: {
      bundle1: './main1.jsx',
      bundle2: './main2.jsx'
    },
    output: {
      filename: '[name].js'
    },
  	mode: 'development',
  	//增加了这个配置即可
  	optimization: {
  		splitChunks: {      // old CommonsChunkPlugin
  			chunks: "all"
  		},
  	},
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		//new UglifyJsPlugin(),//我们想看打包后的文件，注释掉这个
  		new OpenBrowserPlugin({
        url: 'http://localhost:8080'
      }),
  		devFlagPlugin,
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - 创建main1.jsx main2.jsx

  ```javascript
  // main1.jsx
  var React = require('react');
  var ReactDOM = require('react-dom');
  
  ReactDOM.render(
    <h1>Hello World</h1>,
    document.getElementById('a')
  );
  
  // main2.jsx
  var React = require('react');
  var ReactDOM = require('react-dom');
  
  ReactDOM.render(
    <h2>Hello Webpack</h2>,
    document.getElementById('b')
  );
  ```

  - 直接webpack打包，会多一个vendors~bundle1~bundle2.js文件，

  ```
                       Asset       Size                   Chunks             Chunk Names
                  bundle1.js   6.72 KiB                  bundle1  [emitted]  bundle1
                  bundle2.js   6.72 KiB                  bundle2  [emitted]  bundle2
                  index.html  318 bytes                           [emitted]
  vendors~bundle1~bundle2.js    832 KiB  vendors~bundle1~bundle2  [emitted]  vendors~bundle1~bundle2
  ```

- demo13 ：vendor chunk

作用：将某一部分类库，打包到vendor js中。需要先 npm install jquery

- - main.js

  ```javascript
  var $ = require('jquery');
  $('h1').text('Hello World');
  ```

  - webpack .config.js

  ```javascript
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  var OpenBrowserPlugin = require('open-browser-webpack-plugin');
  var webpack = require('webpack');
  var devFlagPlugin = new webpack.DefinePlugin({
    __DEV__: JSON.stringify(JSON.parse(process.env.NODE_ENV || 'false'))
  });
  
  
  module.exports = {
    entry: {
      app: './main.js',
      vendor: ['jquery'],
    },//每次看好打包的入口文件呦
    output: {
      filename: '[name].js'
    },
  	mode: 'development',
  	optimization: {
  		splitChunks: {      // old CommonsChunkPlugin
  			chunks: "all"
  		},
  	},
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		//new UglifyJsPlugin(),
  		new OpenBrowserPlugin({
        url: 'http://localhost:8080'
      }),
  		devFlagPlugin,
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - webpack 打包

  ```cmd
  # webpack 输出
     Asset       Size  Chunks             Chunk Names
        0.js  552 bytes       0  [emitted]
      app.js   9.39 KiB     app  [emitted]  app
  index.html  240 bytes          [emitted]
   vendor.js    305 KiB  vendor  [emitted]  vendor
  ```

  - 说明下，阮神还是用的CommonsChunkPlugin，但是webpack4已经废弃了，其实本例的最后完成方法就类似于两个入口文件了，只不过其中一个的意思是所有的jquery包，打包到vendor.js中
  - 这个例子最后阮神还介绍了一种不需要一直require包的方法
  - webpack.config.js

  ```
  var HtmlwebpackPlugin = require('html-webpack-plugin');
  var UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  var OpenBrowserPlugin = require('open-browser-webpack-plugin');
  var webpack = require('webpack');
  var devFlagPlugin = new webpack.DefinePlugin({
    __DEV__: JSON.stringify(JSON.parse(process.env.NODE_ENV || 'false'))
  });
  
  
  module.exports = {
    entry: {
      app: './main.js',
      vendor: ['jquery'],
    },//每次看好打包的入口文件呦
    output: {
      filename: '[name].js'
    },
  	mode: 'development',
  	optimization: {
  		splitChunks: {      
  			name (module) {
          // generate a chunk name...
          return; //...
        }
  		},
  	},
    //加入html自动生成
    plugins:[
      new HtmlwebpackPlugin({
        title: 'Webpack-tutorial',
        filename: 'index.html'
      }), 
  		//new UglifyJsPlugin(),
  		new OpenBrowserPlugin({
        url: 'http://localhost:8080'
      }),
  		devFlagPlugin,
  		//增加定义$ jquery 随时可以使用
  	new webpack.ProvidePlugin({
        $: 'jquery',
        jQuery: 'jquery'
      })
    ],
  	module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['es2015', 'react']
            }
          }
        },
  			{
          test: /\.css$/,
          use: [
            {
              loader: 'style-loader'
            },
            {
               loader: 'css-loader',
               options: {
                 modules: true
               }
            }
          ]
        }
      ]
    }
  };
  ```

  - 删除掉main.js中的 ，会发现 var $ = require('jquery');正常使用

- demo14：Exposing global variables

作用：定义一些全局变量，并在代码中使用它。

- - 创建文件 data.js

  ```javascript
  var data = 'Hello World';//定义全局变量
  ```

  - 更改 main.jsx 文件

  ```javascript
  // main.jsx
  var data = require('data');
  var React = require('react');
  var ReactDOM = require('react-dom');
  
  ReactDOM.render(
    <h1>{data}</h1>,
    document.body
  );
  ```

  - webpack.config.js 文件

  ```javascript
  //增加如下配置
  	externals: {
      // require('data') is external and available
      //  on the global var data
      'data': 'data'
    }
  ```

  - webpack打包

  ```cmd
  # webpack 输出
   Asset     Size  Chunks             Chunk Names
  app.js  837 KiB     app  [emitted]  app
  Entrypoint app = app.js
  [./main.js] 189 bytes {app} [built]
  [./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {app} [built]
  [data] external "data" 42 bytes {app} [built]
  # 可以看到external data 这个变量可以require后使用了
  ```

  - 不知到为什么，我使用html-webpack-plugin创建index.html后 不好使，而我在dist同级目录下创建一个index.html放入如下代码：打包开启server后，就能够正常显示data的数据了，最后也没弄清楚。

  ```html
  <html>
    <body>
      <script src="data.js"></script>
      <script src="app.js"></script>
    </body>
  </html>
  ```

- demo 15：React router

关于这段react 留着以后再研究。

## 总结

至此阮神的15DEMO演示完毕，本人也是先从官网看的文档，看的很迷糊，一些概念性的东西，完全没头没脑的看，通过一些简单的配置，可以对webpack有一个更好地理解。