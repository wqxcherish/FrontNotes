# 一、初始化项目文件夹
在任意目录下，新建一个文件夹作为你的项目文件夹，命名随意。随后使用命令行工具，切换到该文件夹，键入npm init进行初始化（遇到的问题一直回车就好了），初始化完成之后可以看到生成了一个package.json文件。
随后在该项目文件夹下新建两个文件夹：/dist和/src，其中/src用于放置开发的源码，/dist用于放置“编译”后的代码。
随后在/src目录下新建index.html、index.css和index.js文件

# 二、安装webpack工具
通过命令行使用webpack 4需要安装两个模块：webpack和webpack-cli，都安装为开发环境依赖。

npm install -D webpack webpack-cli

安装完成之后可以看到你的package.json文件发生了变化，在devDependencies属性下多了两个包的属性。
# 三、配置最基本的webpack

## 1.安装最基本的插件：
  npm install -D html-webpack-plugin clean-webpack-plugin webpack-dev-server css-loader webpack-merge style-loader

## 2.在项目文件夹下新建文件webpack.base.conf.js，表示最基本的配置文件，内容如下：
```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
      entry: './src/index.js',
      output: {
          filename: 'bundle.[hash].js',
          path: path.join(__dirname, '/dist')
      },
      module: {
          rules: [
              {
                  test: /\.css$/,
                  use: ['style-loader', 'css-loader']
              }
          ]
      },
      plugins: [
          new HtmlWebpackPlugin({
              template: './src/index.html'
          }),
          new CleanWebpackPlugin(['dist'])
      ]
  };
```

其中，/src/index.html是你的网站入口HTML文件，/src/index.js是你的入口js文件。


## 3.在项目文件夹下新建webpack.dev.conf.js文件，表示开发环境下的配置。内容如下：
```
  const merge = require('webpack-merge');
  const baseConfig = require('./webpack.base.conf.js');

  module.exports = merge(baseConfig, {
      mode: 'development',
      devtool: 'inline-source-map',
      devServer: {
          contentBase: './dist',
          port: 3000
      }
  });
```



## 4.在项目文件夹下新建webpack.prod.conf.js文件，表示生产环境的配置，内容如下：
```
  const merge = require('webpack-merge');
  const baseConfig = require('./webpack.base.conf.js');

  console.log(__dirname);
  module.exports = merge(baseConfig, {
      mode: 'production'
  });
```


#　四、配置npm scripts
配置了三个配置文件以满足两个不同环境下的代码构建，使用语义化较好的npm scripts来构建代码有利于简化工作。
添加新的scripts内容到package.json文件的scripts属性，记得用双引号引起来，其属性如下：
```
// package.json
{
    "scripts": {
        "start": "webpack-dev-server --open --config webpack.dev.conf.js",
        "build": "webpack --config webpack.prod.conf.js"
    }
}
```
配置完之后，可以尝试修改/src/index.html、/src/index.js或/src/index.css，运行npm scripts命令查看效果。
比如按照以下内容创建文件：
```
index.html
<html>
    <head>
        <meta charset="utf-8"/>
        <title>React & Webpack</title>
    </head>
    <body>
        <div id="root">
            <h1>Hello React & Webpack!</h1>
        </div>
    </body>
</html>

index.css
body {
    background-color: blue;
}

#root {
    color: white;
    background-color: black;
}

index.js
import './index.css';

console.log('Success!');
```
随后使用命令npm run start，即可看到效果。修改css或者js文件，保存之后可以看到浏览器自动刷新并且展示出了你刚刚所做的更改。
做到这里，一个基本的开发环境已经搭建出来了，下一步就是针对React特定的环境，配置不同的webpack来进行构建。
使用React开发，主要是ES6（虽然最近所有高级浏览器都已经支持ES6，但是还是要为低级IE做准备）和React的JSX语法需要进行转换。下面针对这两种语法进行配置。

# 五、配置Babel
Babel是一个优秀的JavaScript编译器（这句话源自Babel官网），通过Babel的一些插件，可以将JSX语法、ES6语法转换为ES5的语法，使得低级浏览器也可以运行我们写的代码。
（1）安装Babel预设
通过以下命令安装Babel预设、babel-loader、babel-polyfill和babel-preset-react：
npm install -D babel-preset-env babel-loader babel-polyfill babel-preset-react
复制代码
（2）配置.babelrc
在项目文件夹的根目录下新建一个.babelrc的文件（Windows下无法直接创建，可以通过将文件命名为.babelrc.达到创建的目的），在文件内输入以下内容：
{
    "presets": ["env", "react"]
}
复制代码
（3）配置webpack.base.conf.js
在module.rules中插入一个新对象，内容如下：
{
    test: /\.js$/,
    use: 'babel-loader',
    exclude: /node_modules/
}
复制代码
（4）安装react和react-dom模块
npm install --save react react-dom
复制代码
（5）开始开发
在/src中新建一个App.js文件，内容如下：
import React from 'react';
```
class App extends React.Component {
    render() {
        return <div>
            <h1>Hello React & Webpack!</h1>
            <ul>
                {
                    ['a', 'b', 'c'].map(name => <li>{`I'm ${name}!`}</li> )
                }
            </ul>
        </div>
    }
}

export default App;
```

清空index.js之后在其中写入如下内容：
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './index.css';

ReactDOM.render(<App/>, document.getElementById('root'));
使用npm run start命令打开页面可以看到使用React写出来的效果了。
打开浏览器查看编译后的代码，找到App组件中的map函数这一段，可以发现ES6的语法已经被转换到了ES5的语法：
['a', 'b', 'c'].map(function (name) {
    return _react2.default.createElement(
        'li',
        null,
        'I\'m ' + name + '!'
    );
})
箭头函数被写成了function匿名函数。