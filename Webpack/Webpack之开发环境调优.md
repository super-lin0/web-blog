---
title: Webpack之开发环境调优
date: 2019-10-03 14:57:26
updateed: 2019-10-03 14:57:26
tags: [javascript, Webpack]
comments: true
---

<center>
  本文主要介绍了一些对日常开发有一定帮助的 Webpack 插件以及调试方法
<center>
</br>
</center>
  标签：#javascript, #Webpack
</center>

<!-- more -->

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003144918.png)

## 前言

Webpack 作为打包工具的重要使命之一就是提升效率。本文我们将介绍一些对日常开发有一定帮助的 Webpack 插件及调试方法。本文内容包括：

- Webpack 周边插件介绍

- 模块热替换及其原理

## Webpack 开发效率插件

Webpack 拥有非常强大的生态系统，社区中相关的工具也是数不胜数。这里我们介绍几个使用比较广泛的 Webpack 插件，可以从不同的方面对 Webpack 的能力进行增强。

### webpack-dashboard

Webpack 每一次构建结束后都会在控制台输出一些打包相关的信息日志，但是这些日志是以列表形式呈现的，有时会显得不够直观。Webpack-dashboard 则是以更好的方式来展示这些信息的。先来看一下最终效果。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003105248.png)

**安装**

```
yarn add webpack-dashboard --dev

```

**webpack 配置**

```
// webpack.config.js
const DashboardPlugin = require("webpack-dashboard/plugin");

module.exports = {
  context: path.join(__dirname, "./src"),
  entry: {
    app: "./index.js",
    util: "./util.js"
  },
  output: {
    filename: "[name].js",
    publicPath: "./dist/"
  },
  plugins: [
    new htmlPlugin({
      title: path.basename(__dirname)
    }),
    new DashboardPlugin()
  ],
};

```

**修改启动方式**

```
  //package.json
  "scripts": {
    "build": "webpack",
    "start": "webpack-dashboard -- webpack-dev-server --open-page \"./index.html\""
  },
```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/09-best-pratice-for-dev/webpack-dashboard" >webpack-dashboard</a>

### webpack-merge

顾名思义，我们猜测这个插件是用来合并 Webpack 配置的。先来看看用法

```
// webpack.common.config.js
const path = require("path");
const HtmlPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "[name].js",
    publicPath: "/dist/"
  },
  mode: "development",
  devServer: {
    port: 3000,
    publicPath: "/dist/"
  },
  plugins: [new HtmlPlugin({ title: path.basename(__dirname) })],
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: "file-loader"
      },
      {
        test: /.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  }
};

```

```
// webpack.prod.config.js

const merge = require("webpack-merge");
const commonCfg = require("./webpack.common.config");
const ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = merge.smart(commonCfg, {
  mode: "production",
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: "css-loader"
        })
      }
    ]
  }
});

```

是具体使用过程当中，我们会发现它在合并 module.rules 的过程中会以 test 属性作为标识符，当发现有相同项出现的时候会以后面的规则覆盖前面的规则。

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/09-best-pratice-for-dev/webpack-merge" >webpack-merge</a>

### speed-measure-webpack-plugin

可以分析出 Webpack 整个打包过程中在各个 loader 和 pulgin 上耗费的时间，这将有助于找出构建过程中的性能瓶颈。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003114051.png)

**使用方法**

```
// webpack.config.js

const path = require("path");
const HtmlPlugin = require("html-webpack-plugin");
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

const smp = new SpeedMeasurePlugin();

module.exports = smp.wrap({
  entry: "./src/index.js",
  output: {
    filename: "[name].js",
    publicPath: "/dist/"
  },
  mode: "development",
  devServer: {
    port: 3000,
    publicPath: "/dist/"
  },
  plugins: [new HtmlPlugin({ title: path.basename(__dirname) })]
});

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/09-best-pratice-for-dev/speed-measure-webpack-plugin" >speed-measure-webpack-plugin</a>

### size-plugin

可以帮我们监控资源体积的变化，尽早发现问题

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003115149.png)

**使用方法**

```
const path = require("path");
const HtmlPlugin = require("html-webpack-plugin");
const SizePlugin = require("size-plugin");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "[name].js",
    publicPath: "/dist/"
  },
  mode: "development",
  devServer: {
    port: 3000,
    publicPath: "/dist/"
  },
  plugins: [
    new HtmlPlugin({ title: path.basename(__dirname) }),
    new SizePlugin()
  ]
};

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/09-best-pratice-for-dev/size-plugin" >size-plugin</a>

## 模块热替换（HMR）

Webpack 可以让代码在网页不刷新的前提下得到最新的改动，我们甚至不需要重新发起请求就能够看到更新后的效果，这就是模块热替换功能（Hot Module Replacement, HMR）。适合大型应用，HMR 可以在保留页面当前状态的前提下呈现出最新的改动，可以节省开发者大量的时间成本。

### 开启 HMR

- 项目必须基于 webpack-dev-server 或 webpack-dev-middle 进行开发的

**手动调用 HMR API**

```
// webpack.config.js
const path = require("path");
const htmlPlugin = require("html-webpack-plugin");
const webpack = require("webpack");

module.exports = {
  plugins: [
    //  将会为每个模块绑定一个module.hot对象，这个对象包含了 HMR 的 API
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    hot: true,
    port: 3000
  }
};

```

```
// index.js

import {add} from "util.js";
add(2, 3);

if(module.hot) {
  module.hot.accept();
}

```

**借助第三方现成工具**

```
const path = require("path");
const htmlPlugin = require("html-webpack-plugin");
const webpack = require("webpack");

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
        options: {
          presets: ["react"],
          plugins: ["react-hot-loader/babel"]
        }
      }
    ]
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    hot: true,
  }
};

```

```
// index.js
import React, { Component } from "react";
import ReactDOM from "react-dom";
import { hot } from "react-hot-loader";

class App extends Component {
  render() {
    return <div>Hello World</div>;
  }
}

const AppComponent = hot(module)(App);

const render = () =>
  ReactDOM.render(<AppComponent />, document.getElementById("app"));

render();

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/09-best-pratice-for-dev/hmr-react" >hmr</a>

### HMR 原理

**定义**

- 客户端： 浏览器

- 服务器端：webpack-dev-server(WDS)

**核心**

HMR 的核心就是客户端从服务器端拉取更新后的资源（准确地说，HMR 拉取的不是整个资源文件，而是 chunk diff,即 chunk 需要更新的部分）

**步骤**

- 浏览器什么时候去拉取这些更新

WDS 与浏览器之间维护了一个 websocket，当本地资源发生变化时 WDS 会向浏览器推送更新事件，并带上这次构建的 hash，让客户端与上一次资源进行对比。通过 hash 的比对可以防止冗余更新的出现。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003142942.png)

- 去拉取什么

有了恰当的拉取资源的时机，下一步就是要知道拉取什么。这部分信息并没有包含在刚才的 websocket 中，因为我们刚才只是想知道这次构建的结果是不是和上一次一样。

现在客户端已经知道了新的构建结果和当前的有了差别，就会向 WDS 发起一个请求来获取更改文件的列表，即哪些模块有了改动。返回的结果告诉客户端，需要更新的模块为 app，这样客户端就可以再借助这些信息继续向 WDS 获取该 chunk 的增量更新了。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003143511.png)

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003143557.png)

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003143738.png)

- 获取到增量更新之后的处理

  Webpack 提供了相关的 API 或使用 react-hot-loader 和 vue-loader 这样的第三方 API 来处理客户端获取到增量更新之后的事。

## 小结

本文介绍了一些 Webpack 周边插件，以及如何使用 HMR 以及 HMR 原理。

## 参考文献

1、《Webpack 实战入门、进阶与调优）》- 居玉皓

2、<a href="https://www.npmjs.com/package/webpack-dashboard" >webpack-dashboard</a>

3、<a href="https://www.webpackjs.com/api/hot-module-replacement/" >模块热替换</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
