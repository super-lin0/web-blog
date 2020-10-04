---
title: Webpack之代码分片
date: 2019-09-30 15:24:15
updateed: 2019-09-30 15:24:15
tags: [javascript, Webpack]
comments: true
---

<center>
  本文总结了利用 Webpack 来实现代码分片的几种方式
<center>
</br>
</center>
  标签：#javascript, #Webpack
</center>

<!-- more -->

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003152823.png)

### 前言

实现高性能应用其中重要的一点就是尽可能地让用户每次只加载必要的资源，优先级不太高的的资源则采用延迟加载等技术渐进式地获取。代码分片（Code Spliting）是 Webpack 作为打包工具所特有的一项技术，通过这项技术，我们可以把代码按照特定的形式进行拆分，使用户不必一次全部加载，而是按需加载。

Webpack 提供了三种方式来进行代码分片

- 入口配置：entry 入口使用多个入口文件；

- 抽取公有代码：使用 SplitChunks 抽取公有代码；

- 动态加载：动态加载一些代码。

### 通过入口划分代码

- 在 Webpack 中每个入口都将生成一个对应的资源文件，通过入口的配置我们可以进行一些简单有效的代码拆分

- 对于 Web 应用来说通常会有一些库和工具是不常变动的，可以把他们放在一个单独的入口中，由该入口产生的资源不会经常更新，可以有效利用客户端缓存，让用户不必在每次请求页面时都重新加载。

- 这种拆分方法主要适合于那些将接口绑定在全局对象上的库。

```
  entry: {
    app: "./app.js",
    lib: ["lib-a", "lib-b", "lib-c"]
  }
```

### SplitChunks(抽取公有代码)

- SplitChunks 是由 webpack 4 内置的 SplitChunksPlugin 插件提供的能力，可直接在 optimization 选项中配置

- 可以将多个 Chunk 中公共的部分提取出来

```
  // SplitChunks默认配置
  module.exports = {
    //...
    optimization: {
      splitChunks: {
        chunks: 'async',  // 表示从哪些chunks里面抽取代码，除了三个可选字符串值 initial（只对入口chunk生效）、async(只提取异步chunk)、all(两种模式同时开启) 之外，还可以通过函数来过滤所需的 chunks；
        minSize: 30000, // 表示抽取出来的文件在压缩前的最小大小，默认为 30000；
        maxSize: 0, // 表示抽取出来的文件在压缩前的最大大小，默认为 0，表示不限制最大大小；
        minChunks: 1, // 表示被引用次数，默认为1；
        maxAsyncRequests: 5,  // 最大的按需(异步)加载次数，默认为 5；
        maxInitialRequests: 3,  // 最大的初始化加载次数，默认为 3；
        automaticNameDelimiter: '~',  // 抽取出来的文件的自动生成名字的分割符，默认为 ~；
        name: true, // 抽取出来文件的名字，默认为 true，表示自动生成文件名；
        cacheGroups: {  // 分离chunks时的规则
          vendors: {  //  用于提取node_modules中符合条件的模块
            test: /[\\/]node_modules[\\/]/,
            priority: -10
          },
          default: {  // 作用于被多次引用的模块
            minChunks: 2,
            priority: -20,
            reuseExistingChunk: true
          }
        }
      }
    }
  };
```

**SplitChunks 默认情况下的提取条件**

- 提取后的 chunk 可被共享或者来自 node_modules 目录

- 提取后的 JavaScript Chunk 体积大于 30KB，CSS Chunk 体积大于 50KB。

- 在按需加载过程中，并行请求的资源最大值小于等于 5

- 在首次加载时，并行请求的自愿书最大值小于等于 3

**默认的异步提取**

- 实际上 SplitChunks 不需要配置也能生效，但仅仅针对异步资源

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/06-code-spliting/split-chunks" >SplitChunksPlugin</a>

### 资源异步加载（按需加载）

当模块数量过多、资源体积过大时，可以把一些暂时使用不到的模块延迟加载。这样使页面初次渲染的时候用户下载的资源尽可能小，后续的模块等到恰当的时机再去出发加载。

**import()**

- 同 ES6 中的 import 语法不同，通过 import 函数加载的模块及其依赖会被异步地进行加载，并返回一个 Promise 对象。

- ES6 中要求 import 必须出现在代码的顶层作用域，而 Webpack 的 import 函数则可以在任何我们希望的时候调用。

```
if(condition) {
  import("./a.js").then(a => {
    console.log(a)
  })
}
```

- 通过 JavaScript 在页面的 head 标签里插入一个 script 标签/dist/0.js

![](https://raw.githubusercontent.com/super-lin0/pic/master/20190930110448.png)

**异步 chunk 的配置**

通过 output.chunkFilename 和特有注释来指定异步 chunk 的文件名

```
  output: {
    filename: "[name].js",
    publicPath: "/dist/",
    chunkFilename: "[name].js" // 指定异步chunk的文件名
  },

  // 注释
  import(/* webpackChunkName: "bar..." */ "./bar.js").then(({ add }) => {
    console.log(add(2, 3));
  });
```

最终结果：

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/%E5%9B%BE%E7%89%87.png)

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/06-code-spliting/async-chunk" >资源异步加载</a>

（完）

### 总结

本文我们了解了 Webpack 代码分片的几种方式：合理的规划入口，使用 SplitChunks，以及资源异步加载。借助这些方法我们可以有效地缩小资源体积，同时更好的利用缓存，给用户更友好的体验。

### 参考文献

1、<a href="https://www.webpackjs.com/plugins/split-chunks-plugin/#defaults" >SplitChunksPlugin</a>

1、<a href="https://imweb.io/topic/5b66dd601402769b60847149" >webpack 4 Code Splitting 的 splitChunks 配置探索</a>

2、《Webpack 实战入门、进阶与调优）》- 居玉皓

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
