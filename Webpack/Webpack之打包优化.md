---
title: Webpack之打包优化
date: 2019-10-02 17:15:15
updateed: 2019-10-02 17:15:15
tags: [javascript, Webpack]
comments: true
---

<center>
  本文主要介绍了一些优化 Webpack 配置的方法
<center>
</br>
</center>
  标签：#javascript, #Webpack
</center>

<!-- more -->

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191003100729.png)

## 前言

本文主要介绍一些优化 Webpack 配置的方法，目的是让打包的速度更快，输出的资源更小。本文将会包含以下内容：

- 多线程打包与 happyPack

- 缩小打包的作用域

- 动态链接库思想与 DLLPlugin

- 死代码检测与 tree shaking

## HappyPack

HappyPack 是一个通过多线程来提升 Webpack 打包速度的工具。

### 工作原理

在打包过程当中最耗时的工作就是使用 loader 将各种资源进行转译处理，最常见的包括使用 babel-loader 转译 ES6+ 语法，和 ts-loader 转译 TypeScript。转译的工作流程如下：

- 1、从配置中获取打包入口

- 2、匹配 loader 规则，并对入口模块进行转译

- 3、对转译后的模块进行依赖查找（如 a.js 中加载了 b.js 和 c.js）

- 4、对新找到的模块重复进行第 2、3 步骤，直到没有新的依赖模块。

由于运行在 Node.js 上的 Webpack 是单线程模型的，所以 Webpack 需要一步步地获取更深层级的资源，然后逐个进行串行转译，所以在模块多、依赖复杂的情况下，Webpack 构建速度将会变得很慢。HappyPack 恰好能解决此问题。

**核心特性**

可以开启多个线程，并行地对不同模块进行转译，这样就可以充分利用本地的计算资源来提升打包速度。

**适合场景**

HappyPack 适用于那些转译任务比较重的工程，例如 babel-loader、ts-loader，对于其他的如 sass-loader、less-loader 本身消耗时间并不太多的工程效果一般。

### 单个 loader 的优化

在实际使用过程中，要用 HappyPack 提供的 loader 来替换原有 loader，并将原有的那个 loader 通过 HappyPack 插件传进去。

**使用 HappyPack 之前**

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
        options: {
          presets: ["react"]
        }
      }
    ]
  },
}
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191002111803.png)

**使用 HappyPack**

```

const babelQuery = {
  presets: [
    [
      "env",
      {
        modules: false,
        targets: {
          browsers: ["> 1%", "last 3 versions"]
        }
      }
    ],
    "react"
  ]
};

module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "happypack/loader"
      }
    ]
  },
  plugins: [
    new htmlPlugin({
      title: path.basename(__dirname)
    }),
    new HappyPack({
      loaders: [
        {
          loader: "babel-loader",
          options: babelQuery
        }
      ]
    })
  ],
};

```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20191002112031.png)

由上面的结果可以大致发现，用 HappyPack 确实能够提升打包速度。

### 多个 loader 的优化

在使用 HappyLoader 优化多个 loader 时，需要为每一个 loader 配置一个 id，否则 HappyPack 无法知道 rules 与 plugins 如何一一对应。

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "happypack/loader?id=js"
      },
      {
        test: /\.ts$/,
        exclude: /node_modules/,
        loader: "happypack/loader?id=ts"
      }
    ]
  },
  plugins: [
    new htmlPlugin({
      title: path.basename(__dirname)
    }),
    new HappyPack({
      id: "js",
      loaders: [
        {
          loader: "babel-loader",
          options: {}
        }
      ]
    }),
    new HappyPack({
      id: "ts",
      loaders: [
        {
          loader: "ts-loader",
          options: {}
        }
      ]
    })
  ],
};

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/08-bundle-optimization/happypack" >happyPack</a>

## 缩小打包作用域

从宏观角度来讲，提升性能的方法有两种：

- 增加资源

就是使用更多 CPU 和内存，用更多的计算能力来缩短执行任务的时间。例如上文提到的 HappyPack

- 缩小范围

针对任务本身，比如去掉冗余的流程，尽量不要作重复性的工作等。

### exclude 和 include

exclude 和 include 是用来排除或包含指定目录下的模块，可接收正则表达式或者字符串（文件绝对路径），以及由他们组成的数组。

```
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js"
  },
  mode: "development",
  plugins: [],
  module: {
    rules: [
      {
        test: /\.css$/, // 匹配所有以.css结尾的文件
        use: ["style-loader", "css-loader"], // 接收一个数组，该数组包含该规则所使用的loader，按照从后往前的顺序将资源交给loader处理
        exclude: /node_modules/, // 所有node_modules下的模块不应用这个规则
        include: /src/  // 只有src目录下的模块才会应用这个规则
      }
    ]
  },
}
```

**exclude**

所有被正则表达式所匹配到的模块都排除在该规则之外

**include**

该规则只针对被正则表达式匹配到的模块生效

**Note**

exclude 和 include 同时生效时，exclude 的优先级更高

### noParse

有些库我们是希望 Webpack 完全不要去进行解析的，即不希望应用任何 loader 规则，库的内部也不会有对其他模块的依赖，那么这时可以使用 noParse 进行忽略。

```
module.exports = {
  module: {
    // 忽略所有文件名中包含lodash的模块，这些模块仍然会被打包进资源文件，但Webpack不会对其进行任何解析
    noParse: /lodash/,
  }
}
```

```
module.exports = {
  module: {
    noParse: function(fullPath) {
      // fullPath为绝对路径，如 /Users/me/app/webpack-no-parse/lib/lodash.js
      return /lib/.test(fullPath);  // 忽略所有lib目录下的资源解析
    }
  }
}
```

### IgnorePlugin

exclude 和 include 是确定 loader 的规则范围，noparse 是不去解析但仍会打包到 bundle 中。而 IgnorePlugin 则可以完全排除一些模块，被排除的模块即便被引用了也不会被打包进资源文件中。

```
plugins: [
  new webpack.IgnorePlugin({
    resourceMapExp: /^\.\/locale$/, //  匹配资源文件
    contextRegExp: /moment$/  // 匹配检索目录
  })
]
```

一些由库产生的额外资源我们用不到但又无法去掉，因为引用的语句处于库文件内部。因此可以使用 IgnorePlugin 来排除一些库相关文件。例如，Moment.js 是一个日期时间处理相关的库，为了做本地化它会加载很多语言包，对于我们来说，一般不会用到其他地区的语言包，但他们会占用很多体积，这时就可以像上面这样用 IgnorePlugin 来去掉。

### Cache

有些 loader 会有一个 cache 配置项，用来在编译代码后同时保存一份缓存，在执行下一次编译前会先检查源文件是否有变化，如果没有就直接采用缓存。

## 动态链接库与 DLLPlugin

**动态链接库**

- 早期 windows 系统由于受限于当时计算机内存空间较小的问题而出现的一种内存优化方法

- 当一段相同的子程序被多个程序调用时，为了减少内存消耗，可以将这段子程序存储为一个可执行文件，当多个程序调用时只在内存中生成和使用一个实例

**DLLPlugin**

DLLPlugin 借鉴动态链接库的思路，对于第三方库或者一些不常变化的模块，可以将它们预先编译和打包，然后在项目实际构建过程中直接取用即可。

**DLLPlugin 和 Code Spliting 的区别**

两者类似都可以用来提取公共模块，但本质上有区别

- Code Spliting 设置一些特定的规则并在打包过程中根据这些规则提取模块

- DLLPlugin 则是将 vendor 完全拆出来，有自己的一整套 Webpack 配置并独立打包，在实际工程构建时就不用再对它进行任何处理，直接取用即可。因此，理论上来讲，DLLPlugin 打包速度更快，但相应配置增加，资源管理复杂度增加。

### vendor 配置

首先需要为动态链接库单独创建一个 webpack 配置文件，比如文件名为 webpack.vendor.config.js

```
const path = require("path");
const webpack = require("webpack");

const dllAssetPath = path.join(__dirname, "dist");
const dllLibraryName = "dllExample";

module.exports = {
  entry: ["react"], // 把哪些模块打包为vendor
  output: {
    path: dllAssetPath,
    filename: "vendor.js",
    library: dllLibraryName
  },
  plugins: [
    new webpack.DllPlugin({
      name: dllLibraryName, // 导出dll library的名字，需要与output.library值相对应
      path: path.join(dllAssetPath, "manifest.json") // 资源清单的绝对路径，业务代码打包时会使用这个清单进行模块索引
    })
  ]
};

```

### vendor 打包

在 package.json 中新增配置,利用 dll 命令进行打包

```
  "scripts": {
    "dll": "webpack --config webpack.vendor.config.js",
  },

```

### 链接到业务代码

使用 DLLReferencePlugin 插件将 vendor 链接到项目中。它起到一个索引和链接的作用。

```
// webpack.config.js
module.exports = {
  entry: {},
  output: {},
  plugins: [
    new Webpack.DllReferencePlugin({
      manifest: require(path.join(__dirname, "dll/manifest.json"))
    })
  ],
};

```

### 潜在的问题

当我们对 vendor 进行操作时，本来 vendor 中不应该受到影响的模块却改变了他们的 id。导致 vendor 变化需要用户重新下载左右模块资源

**解决办法**

在打包 vendor 时添加上 HashedModuleIdsPlugin

```
const path = require("path");
const webpack = require("webpack");

const dllAssetPath = path.join(__dirname, "dist");
const dllLibraryName = "dllExample";

module.exports = {
  entry: ["react"], // 把哪些模块打包为vendor
  output: {
    path: dllAssetPath,
    filename: "vendor.js",
    library: dllLibraryName
  },
  plugins: [
    new webpack.DllPlugin({
      name: dllLibraryName, // 导出dll library的名字，需要与output.library值相对应
      path: path.join(dllAssetPath, "manifest.json") // 资源清单的绝对路径，业务代码打包时会使用这个清单进行模块索引
    }),
    new webpack.HashedModuleIdsPlugin()
  ]
};

```

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/08-bundle-optimization/dll-plugin" >DLL Plugin</a>

## tree shaking

tree shaking 可以在打包过程中帮助我们检查工程中没有被引用过的模块，这部分代码将永远无法被执行到，因此也被称为“死代码”。Webpack 会对这部分代码进行标记，并在资源压缩时将它们从最终的 bundle 中去掉。

**Note**

tree shaking 本身只是为死代码添加标记，真正去除死代码是通过压缩工具来进行的。例如上一篇文章中我们提到过的 terser-webpack-plugin。在 webpack4 之后的版本中，将 mode 标记为 production 也可以达到相同的效果。

**完整示例**

<a href="https://github.com/super-lin0/webpack-study/tree/master/webpackinaction/08-bundle-optimization/tree-shaking" >Tree shaking</a>

## 总结

本文介绍了加快打包速度，减小资源体积的一些方法。对于一些对性能要求很高的项目来说这些方法可以起到一定的效果。
最后需要说明的一点是，每一种优化策略都有其使用场景，并不是任何一个点放在一切项目中都有效。当我们发现性能的问题时，还是要根据现有的情况分析出瓶颈在哪里，然后对症下药。

## 参考文献

1、《Webpack 实战入门、进阶与调优）》- 居玉皓

2、<a href="https://github.com/amireh/happypack" >HappyPack</a>

3、<a href="https://www.webpackjs.com/configuration/" >Webpack</a>

4、<a href="https://www.webpackjs.com/configuration/module/#module-noparse" >Webpack NoParse</a>

5、<a href="https://www.webpackjs.com/plugins/ignore-plugin/" >Webpack extract-text-webpack-plugin</a>

6、<a href="https://www.webpackjs.com/plugins/html-webpack-plugin/" >Webpack IgnorePlugin</a>

6、<a href="https://www.webpackjs.com/plugins/dll-plugin/" >DllPlugin</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
