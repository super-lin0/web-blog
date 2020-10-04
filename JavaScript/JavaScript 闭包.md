---
title: JavaScript 闭包
date: 2019-08-17 16:09:15
updateed: 2019-08-17 16:09:15
tags: [javascript]
---

<center>
  本文总结了 JavaScript 中闭包的一些概念以及常见问题及解答
<center>
</br>
</center>
  标签：#javascript
</center>

<!-- more -->

### 前言

有不少开发人员总是搞不清闭包和匿名函数这两个概念，因此经常混用。闭包（closure）是 Javascript 语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。

下面就是我的学习笔记，对于 Javascript 初学者应该是很有用的。

### 概念

- MDN
  JavaScript 中的函数会形成闭包。 闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量。允许将函数与其所操作的某些数据（环境）关联起来。
- 《JavaScript 高级程序设计（第 3 版）》
  闭包是指有权访问另一个函数作用域中的变量的函数
- 阮一峰的个人博客
  闭包就是能够读取其他函数内部变量的函数；定义在一个函数内部的函数；闭包就是将函数内部和函数外部连接起来的一座桥梁。
- 个人理解
  函数在被调用的时候还保持着该函数在声明时所拥有的作用域链

## 实例理解

```
function sayHi() {
  var name = 'My Object'
  return function() {
    console.log('name', name) // My Object 产生闭包，在调用的时候持有该函数的作用域链,因此可以访问到外层函数的 name 属性
  }
}
sayHi()()
```

在这个例子中，内部函数（一个匿名函数）中的代码，访问了外部函数的变量`name`,即使这个函数被返回了，而且是在其他地方被调用，但他仍然可以访问这个变量，这是因为内部函数的作用域链中包含`sayHi`的作用域。

- Notes
  当某个函数第一次被调用时，会创建一个执行环境及相应的作用域链，并把作用域链赋值给一个特殊的内部属性([[Scope]])。然后，使用`this`、`arguments`和其他命名参数的值来初始化函数的活动对象。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位。。。直到作为作用域链终点的全局执行环境。

因此，在函数的执行过程中，为了读取和写入变量的值，就需要在作用域链中查找变量。来看下面的例子：

```
var sayName = function() {
  var name = 'My Object'

  return function() {
    return function() {
      return function() {
        console.log('name', name) // My Object 产生闭包，在调用的时候持有该函数的作用域链
      }
    }
  }
}
sayName()()()()
```

以上代码先定义了`sayName()`函数，然后又在全局作用域中调用了它。当最后调用`sayName`时，会顺着最内层的匿名函数的作用域链开始查找一个名为`name`的属性，直到作用域链的终点，也就是说全局作用域，如果在全局作用域中也没有找到，则返回`undefined`

### 闭包的用途

- 读取函数内部的变量

- 让这些变量的值始终保持在内存中

```
// 创建函数
var sayName = function() {
  name = 'My Object'
  return function() {
    console.log('name', name) //1、 My Object 产生闭包，在调用的时候持有该函数的作用域链,因此可以访问到外层函数的 name 属性
  }
}
// 调用函数
sayName()()
console.log(name) // 2、My Object

// 解除对匿名函数的引用（以便释放内存）
sayName = null
console.log(name) // undefined
```

首先，创建的函数被保存在`sayName`变量中。而通过将`sayName`设置为`null`解除该函数的引用，就等于通知垃圾回收例程将其清除。随着匿名函数的作用域被销毁，其他作用域（除了全局作用域）也都可以安全的销毁了

### 注意点

- 由于闭包会携带包含它的函数作用域，因此会比其他函数占用更多的内存。所以不能滥用闭包。建议大家只在绝对必要的时候再考虑使用闭包。并且，在退出函数之前，将不使用的局部变量全部删除。

- 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。

### 例题讲解

在面试中，经常会碰到下面这道题。

```
var createFunctions = function(nodes) {
  var results = []
  for (var i = 0; i < 10; i++) {
    results[i] = function() {
      console.log(i)    // 被调用时候再去打印输出，这个时候i值已经变为10
    }
  }
  return results
}

var results = createFunctions() // 第一步

results.forEach(f => f()) // 第二步 10 10 10 10 10 ...
```

`createFunctions`的本意是想传递给每个数组元素一个为一值（i）。但是未能达到目的，最终所有输出 10。
关于这个问题，很多人的回答差强人意。我们来看看一些经典的回答。

“因为函数绑定了变量 i 本身，而不是函数在构造时的变量 i 的值” - Douglas Crockford

“因为每个函数的作用域链中都保存着 createFunctions 函数的活动对象，所以他们引用的都是同一个变量 i ,当 createFunctions() 函数返回后，变量 i 的值是 10，此时每个函数都引用着保存变量 i 的同一个变量对象，所以每个函数内部 i 的值都是 10” - Nicholas C.Zakas

这上面两个大神的讲解不知道大家是否能看得明白，反正我看完以后还是云里雾里的。下面谈谈我自己的看法。

上面每个`results`中的函数，是在被调用的时候再去执行该匿名函数，而在函数调用的时候，也就是上述代码注释中的"第二步"，这个时候循环已经完成，i 被赋值为 10，此时再去执行相应的匿名函数，该数组全部打印 10。简单来说，就是`results`中的每个函数在被调用的时候，i 已经被赋值为 10。

- 解决办法

关于上面这道题的解决办法无需多说，大家都懂的。直接看代码

```
var createFunctions = function(nodes) {
  var results = []
  var helper = function(i) {
    return function() {
      console.log(i)
    }
  }
  for (var i = 0; i < 10; i++) {
    results[i] = helper(i) // 在循环内部调用时绑定i值，而不是在声明的时候绑定i值
  }
  return results
}

var results = createFunctions() // 第一步

results.forEach(f => f()) // 第二步 0 1 2 3 ...
```

关于闭包的学习到此就差不多了，查阅了很多的文献和参考资料，然后总结出了自己的一些想法，希望能够帮助到大家，另外也是对自己学习的一种记录和总结，文中有不当之处，欢迎指正讨论，一起来交流。

### 参考文献：

1、<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures" >MDN-闭包</a>

2、《JavaScript 高级程序设计（第三版）》- Nicholas C.Zakas

3、《JavaScript 语言精粹（修订版）》- Douglas Crockford

4、<a href="http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html">学习 Javascript 闭包（Closure）-阮一峰</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
