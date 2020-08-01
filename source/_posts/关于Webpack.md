---
title: 关于Webpack
date: 2020-07-21 21:00:54
tags: [前端, 工具]
---

<br/>
<br/>
谁能想到两年前做的东西……两年后要重温一遍呢。并且随着合作过的程序员越来越多，有意识到蛮多同事（甚至是专职写前端的程序员）往往根本没有机会去设置webpack，对这个打包工具的概念一知半解，就干脆把一些简单的概念总结一下（当然官网上写的非常好，个人很推荐感兴趣的同学直接阅读官网），并且提供一些小练习来加深理解。

## 1. 什么是Webpack
简单来说，webpack是一个<u>静态模块打包工具</u>。它的主要概念就是将多个JavaScript，css，image等不同的任何类型的文件捆绑在一起。它会分析这些文件之间的依赖关系，比如什么文件import（导入）了什么文件，并且在内部生成一个dependency graph（依赖图），从而生成一个或者多个bundle（捆绑）文件。
但webpack又不仅仅只是一个捆绑工具，它同时也可以帮忙处理优化文件，如manifest；也可以使用loader去进行一些转义工作，使得不同类型的文件可以被捆绑；还可以使用plugin去进行environment variable injection等操作。但所有这些处理的最终目的都是为了将不同的文件处理成适合的浏览器可以接受的格式并且打包。

<br/>
<br/>

## 2. Webpack如何工作

我们知道webpack会将多个文件进行分析并且打包，那么它是如何做到的呢？首先，webpack需要一个entry point（入口起点），没有这个起点，webpack就无法构建内部依赖图。这可以通过配置 `entry` 选项来指定：
```javascript
    module.exports = {
        entry: './src/app.js'
    };
```
当然，我们也可以指定多个起点来告诉webpack我们需要多个<b>独立的</b>依赖图，这种情况往往应用于多页面应用，每次从服务器获取新的HTML页面的时候，页面就会重新加载资源，不同的页面可能需要不同的打包文件：
```javascript
    module.exports = {
        entry: {
            main: './src/main/app.js',
            payment: './src/paymeng/app.js'
        }
    };
```

再说明完了webpack如何<b>找到入口开始分析</b>后，我们通过可以配置 `output` （输出）选项来指定webpack我们希望它生成的文件名:
```javascript
    module.exports = {
        output: {
            filename: 'bundle.js',
            path: __dirname + '/dist'
        }
    };
```
根据以上的配置，webpack将会把文件打包并命名成 `bundle.js` 然后写入 `./dist` 目录。

> 即使可以存在多个 `entry` 起点，但只能指定一个 `output` 配置。

在了解了webpack如何<b>开始</b>以及<b>结束</b>了之后，我们可以开始更多的了解一下它的中间流程是怎么样的了。首先让我们理解一下两个概念：
- <b>Loader</b>：Loader用于对模块的代码进行转换。
- <b>Plugin</b>：Plugin用于解决所有loader无法解决的问题。
<br/>

打个比方，在使用React的时候，我们常常遇到一些需要对某些组件进行CSS的调整，这种情况下，我们该如何链接该JS文件与CSS文件呢？我们可以使用“css-loader”来解决：
```javascript
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' }
    ]
  }
};
```
这个情况下，webpack就会知道，<b>使用 `css-loader` 来加载所有 `.css` 文件</b>。

> loader的使用有三种方式：通过配置`webpack.config.js`，通过在`import`语句中显式指定`loader`又或者在`shell`命令中指定。但通常我们会选择使用配置。

另外，Webpack的loader都是per-file based的。即所有的 `.css` 文件都会被 `css-loader` 转换。另外我们还可以指定<b>多个loader</b>:
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```
以上为官方的例子，我们可以理解为，<b>以下至上</b>，所有的 `.css` 文件都会首先被 `sass-loader` 转换，接着被 `css-loader` 转换，最后被 `style-loader` 转换。

与 `loader` 相反， `plugin` 是对<b>连接生成的文件</b>进行处理。即它会全局式地对所有已经被 `loader` 处理完的文件再次进行优化。

<br/>
<br/>

## 3. 练习：从零开始使用webpack去设置React应用

练习的时候可以参考这个代码来看是否哪里设置错误：https://github.com/shimingwu/webpack-demo-tutorial

首先，创建一个本地文件夹 `webpack-demo`，使用npm进行管理，生成 `package.json` 来进行项目包的依赖管理。
```bash
npm init
```
生成的文件如下：
```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "shiming",
  "license": "ISC"
}
```

安装webpack。当然通常我们也需要安装 `webpack-dev-server` 和 `webpack-cli` 用来跑本地的项目。另外安装在 `devDependecies` 的主要原因是我们并不需要把我们的项目发布，所以并没有生产环境下面需要的依赖项。即，如果放在 `Dependencies` 下面，当我们发布我们的项目的时候，依赖项会被一起打包加载。
```
npm install --save-dev webpack webpack-dev-server webpack-cli
```
安装完成后，我们应该看到 `package.json` 被更新了：
```json
"devDependecies": {
    "webpack": "^4.44.1",
    "webpack-dev-server": "^3.11.0",
    "webpack-cli": "^3.11.0"
}
```
并且我们会看到 `node_modules` 文件夹。
{% asset_img node_modules_generated.PNG %}

安装React相关的依赖，并创建一个简单的React App。
```bash
npm install --save react react-dom
```
```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "shiming",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.44.1",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "^3.11.0"
  },
  "dependencies": {
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  }
}
```

创建 `src` 文件夹，创建 `index.html`：
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=1024, initial-scale=1.0, shrink-to-fit=no, maximum-scale=1.0, user-scalable=no">
    <meta name="theme-color" content="#000000">
    <title>Webpack Demo</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

创建 `index.js`，该文件将作为webpack的entry：
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

创建 `App.js`, 这就是我们的React App文件:

```javascript
import React from 'react';

function App() {
  return (
    <h1>Hello World</h1>
  );
}
export default App;
```

创建 `App.css`，用来修饰我们的H1 tag。
```css
.red{
    color:red
}
```

更新我们的 `App.js`，使其使用我们新创建的css文件：
```javascript
import React from 'react';
import './App.css'; //注意这里需要写完整名字，即写上.css，其他的js文件可以省略.js后缀，但是css文件不可以

function App() {
  return (
    <h1 className="red">Hello World</h1>
  );
}
export default App;
```

这时候我们已经写好了我们的sample app，更新 `package.json` 里面的 `scripts`， 写一条 `start` 命令用于跑 `webpack-dev-server`：
```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server"
  },
  "author": "shiming",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.44.1",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "^3.11.0"
  },
  "dependencies": {
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  }
}
```

启动项目:
```bash
npm start
```
我们会看到错误提示：
{% asset_img error01.PNG %}

这是因为React使用的是JSX，而webpack并不知道什么是JSX，于是我们要开始配置我们的webpack了。创建 `webpack.config.js`，这就是webpack的配置文件。我们已经了解过，我们需要一个入口以及一个输出：
```javascript
const path = require('path'); // node-js

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}
```
这样我们就可以让webpack帮忙把打包后的文件输出到 `/dist` 目录。

那么为了使webpack理解我们的 `js` 文件夹，我们需要依赖于某些 `loader` 。这里可以使用 `babel loader`，`babel` 则是一个通用的JavaScript编译器，Babel可以将某些版本的JavaScript代码转换为向后兼容的JavaScript语法。
```bash
npm install --save-dev @babel/core babel-loader @babel/preset-env @babel/preset-react
```

有了 `babel loader`，我们就可以开始配置webpack了。在 `module` 下面我们可以设置一个 `rule`，即：所有不在 `/node_modules` 目录下的 `.js` 文件都由 `babel-loader` 进行处理：
```javascript
const path = require('path'); // node-js
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            }
        ]
    }
}
```

但是这样一来又有一个问题，即 `babel-loader` 也不清楚我们希望使用什么样的转换。所以创建 `.babelrc` 文件进行设置。`.babelrc` 即 `babel` 的配置文件。
```json
{
    "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

现在我们再跑一次 `start` 命令，我们会看到新的错误提示：
{% asset_img error02.PNG %}

这是因为webpack不知道该如何处理 `css` 文件，这里我们可以使用 `css-loader` 使得webpack知道如何处理 `css` 文件，使用 `style-loader` 使得webpack知道如何将 `css` 载入html文件。首先下载这两个loader:
```bash
npm install --save-dev css-loader style-loader
```

这时候更新我们的 `webpack.config.js`：
```javascript
const path = require('path'); // node-js
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.css$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: "style-loader"
                    },
                    {   
                        loader: "css-loader"
                    }
                ]
            }
        ]
    }
}
```
这时候我们就可以看到我们的文件被成功编译了。但是，当我们跑localhost的时候，会发现并没有东西呈现出来。这时候我们就需要配置webpack去把我们刚刚生成的 `bundle.js` 放进 `<script>` tag以HTML的形式呈现，我们可以使用 `html-webpack-plugin` 来告诉webpack我们希望加载的HTML是 `index.html` ，并且我们希望将我们的react应用挂载在 `<body>` 里面:
```bash
npm install html-webpack-plugin --save-dev
```
```javascript
const path = require('path'); // node-js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: "babel-loader"
                }
            },
            {
                test: /\.css$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: "style-loader"
                    },
                    {   
                        loader: "css-loader"
                    }
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: __dirname + '/src/index.html',
            filename: 'index.html',
            inject: 'body'
        })
    ]
}
```

在跑一次 `npm start` 命令！恭喜！这时候一个简单的react应用就通过webpack生成了！
{% asset_img success.PNG %}

<br/>
<br/>

## 4. 参考资料
1. [Webpack文档](https://webpack.docschina.org/concepts) 