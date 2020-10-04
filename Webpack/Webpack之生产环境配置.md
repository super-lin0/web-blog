---
title: Webpack之生产环境配置
date: 2019-10-01 19:09:15
updateed: 2019-10-01 19:09:15
tags: [javascript, Webpack]
comments: true
---

<center>
  本文总结了 Webpack 在生产环境配置中一些需要注意的地方
<center>
</br>
</center>
  标签：#javascript, #Webpack
</center>

<!-- more -->

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003160517.png)

## 前言

在生产环境（线上环境），资源打包将会遇到很多新的问题。本文将会包含以下内容

- 环境变量的使用

- source map 机制与策略

- 资源压缩

- 优化 hash 与缓存

- 动态 HTML

## 环境配置的封装

### 使用相同的配置文件

Webpack 不管在什么环境下打包都使用 webpack.config.js,只是在构建开始前将当前所属环境作为一个变量传进去，然后在 webpack.config.js 中通过各种判断条件来决定使用哪个配置条件。

```
// package.json
{
  "scripts": {
    "build": "ENV=production webpack",
    "start": "ENV=production webpack-dev-server --open-page \"dist/index.html\""
  }
}

// webpack.config.js
const ENV = process.env.ENV;
const isProd = ENV === "production";

module.exports = {
  mode: ENV,
  output: {
    filename: isProd ? "bundle@[chunkhash].js" : "bundle.js"
  }
}
```

### 为不同环境创建各自的配置文件

例如，生产环境配置文件名叫 webpack.production.config.js，开发环境配置文件名可叫做 webpack.development.config.js。

```
// package.json
{
  "scripts": {
    "build": "webpack --config=webpack.production.config.js",
    "start": "webpack-dev-server --config=webpack.production.config.js"
  }
}
```

**Note**

开发环境和生产环境配置重复的部分可以提取出来，创建文件 webpack.common.config.js

## 开启 production 模式

webpack 4 中直接加了一个 mode 配置项，让开发者可以通过它来直接切换打包模式。

```
// webpack.config.js
module.exports = {
  mode: "production",
}
```

## 环境变量

```
// wepack.config.js
module.exports = {
  mode: "production",
  plugins: [
    new webpack.DefinePlugin({
      ENV: JSON.stringify("production"),
      IS_PROD: true,
      ENV_ID: 130912098,
    }),
  ],
};

// index.js
document.write("ENV::" + ENV);
```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/define-plugin" >define plugin</a>

## source map

source map 指的是将编译、打包、压缩后的代码映射回源代码的过程。Webpack 打包压缩后的代码可读性较差，因此基本不能调试，source map + 调试工具(dev tools)正好可以解决这个问题。

### 原理

Webpack 对于工程源代码的每一步处理都有可能会改变代码的位置、结构，甚至是所处文件，因此每一步都需要生成对应的 source map。若我们启用了 devtool 配置项，source map 就会跟随源代码一步步被传递，直到生成最后的 map 文件。这个文件默认就是打包后的文件名加上.map，如 bundle.js.map。

**Note**

- 生成 mapping 文件的同时，bundle 文件中会追加上一句注释来标识 map 文件的位置。当我们打开浏览器开发者工具时，map 文件会同时被加载，这时浏览器会使用它来对打包后的 bundle 文件进行解析，分析出源代码的目录结构和内容。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191001165658.png)

### 配置

```
{
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.less/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              sourceMap: true
            }
          },
          {
            loader: "less-loader",
            options: {
              sourceMap: true
            }
          }
        ]
      }
    ]
  },
}
```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/source-map" >Source map</a>

### 安全

有了 source map 也就意味着任何人通过浏览器的开发者工具都可以看到工程源码，对于安全性来说是极大的隐患。Webpack 提供了 hidden-source-map 及 nosources-source-map 两种策略来提升 source map 的安全性。

**hidden-source-map**

hidden-source-map 意味着 Webpack 任然会产出完整的 map 文件，只不过不会在 bundle 文件中添加对于 map 文件的引用。这样一来，当打开浏览器开发者工具时，我们看不到 map 文件，浏览器自然也无法对 bundle 进行解析。如果想要追溯源码，需要利用一些第三方服务，例如 Sentry。

**nosource-source-map**

打包部署之后，我们可以在浏览器开发者工具的 Sources 选项卡中看到源码的目录结构，但是文件的具体内容会被隐藏起来。对于错误来说，仍可以在 Console 控制台中查看源代码的错误栈以及日志的准确行数

**Ngnix 配置**

正常打包出 source map，然后通过服务器的 Ngnix 配置（或其他类似工具）将 .map 文件只对固定白名单（比如公司内网）开放。

## 资源压缩

在将资源发布到线上环境前，我们通常需要对代码进行压缩，即将多余的空格、换行及执行不到的代码，缩短变量名，在执行结果不变的前提下将代码替换为更短的形式。

### 压缩 JavaScript

Webpack4 集成 terser

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/teser-plugin" >Terser plugin demo</a>

### 压缩 CSS

压缩 CSS 文件的前提是使用 extract-text-webpack-plugin 或 mini-css-extract-plugin 将样式提取出来，接着使用 optimize-css-assets-webpack-plugin 来进行压缩，这个插件本质上使用的是压缩器 cssnano。

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/minification-css" >CSS 压缩</a>

## 缓存

缓存是指重复利用浏览器已经获取过的资源。合理地使用缓存是提升客户端性能的一个关键因素。具体的缓存策略由服务器来决定，浏览器会在资源过期前一直使用本地缓存进行响应。

### 资源 hash

在每次打包的过程中对资源的内容计算一次 hash，并作为版本号存放在文件名中，如 bundle@c58f651eb71ab3bbcd01.js。

- 用 chunkhash 来作为文件版本号，它会为每一个 chunk 单独计算一个 hash。

```
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle@[chunkhash].js"
  }
}

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/cache" >chunkhash</a>

### 输出动态 HTML

资源名改变之后意味着 HTML 中的引用路径的改变。每次更改后都要手动的去维护是很麻烦的，因此，最理想的情况是在打包结束后自动把最新的资源名同步过去。使用 html-webpack-plugin 可以帮我们做到这一点。

```
// webpack.config.js
const htmlPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [new htmlPlugin({ title: path.basename(__dirname) })],
};
```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/07-cfg-in-prod/cache" >动态输出 HTML</a>

## bundle 体积监控和分析

VS Code 插件：Import Cost

工具：webpack-bundle-analyzer

自动化资源监控工具：bundlesize

## 小结

本文了解了在生产环境下需要进行的一些特殊的配置。

开发环境中我们关注的可能是开发速度，而在生产环境中我们关注的则是输出的资源体积以及如何优化客户端缓存来缩短页面渲染时间。我们介绍了设置生产环境变量、压缩代码、监控资源体积等方法。缓存的控制住要依赖于从 chunk 内容生成 hash 作为版本号，并添加到资源文件名中，使资源更新后可以立即被客户端获取到。

source map 对于追溯线上问题十分重要，但也存在安全隐患。通过一些特殊的 source map 配置以及第三方服务，我们可以兼顾两者。

Webpack 4 提供了"mode: 'production'"的配置项，通过它可以节省很多生产环境下的特定代码，让配置文件更加的简洁

## 参考文献

1、《Webpack 实战入门、进阶与调优）》- 居玉皓

2、<a href="https://www.webpackjs.com/concepts/mode/" >Webpack 模式</a>

3、<a href="https://www.webpackjs.com/plugins/define-plugin/" >Webpack DefinePlugin</a>

4、<a href="https://www.webpackjs.com/configuration/devtool/" >Webpack devtool</a>

5、<a href="https://www.webpackjs.com/plugins/extract-text-webpack-plugin/" >Webpack extract-text-webpack-plugin</a>

6、<a href="https://www.webpackjs.com/plugins/html-webpack-plugin/" >Webpack html-webpack-plugin</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
