---
title: JavaScript this 的用法
date: 2019-08-20 16:29:15
updateed: 2019-08-20 16:29:15
tags: [javascript]
---

<center>
  本文总结了 JavaScript 中 this 的一些概念以及常见问题及解答
<center>
</br>
</center>
  标签：#javascript
</center>

<!-- more -->

### 前言

除了声明时定义的形式参数，每个函数还接收两个附加的参数：`this`、`argmuments`。参数`this`在面向对象中非常重要，他的值取决于调用的模式。在`JavaScript`中，一共有四种调用模式：

- 方法调用模式
- 函数式调用模式
- 构造器调用模式
- apply 调用模式

下面就是我的学习笔记，对于 Javascript 初学者应该是很有用的。

### 方法调用模式

- 当一个函数被保存为一个对象的属性时，我们称它为一个方法。当一个方法被调用时，this 被绑定到该对象。

```
const object = {
  name: 'zhangsan',
  sayName: function() {
    name = 'lisi'
    console.log(this.name) // zhangsan
    this.name += " ss"
  }
}

object.sayName()
object.sayName()  // zhangsan ss

```

方法可以使用 this 访问自己所属的对象，所以它能从对象中取值或者对对象进行修改。

- 注意：
  this 到对象的绑定发生在调用的时候

### 函数调用模式

- 匿名函数的执行环境具有全局性，因此其 this 对象通常指向 window （当一个函数并非一个对象的属性时，它就是被当作一个函数来被调用的）。

```
var x = 1;
function test() {
   console.log(this.x);
}
test();  // 1
```

```
const object = {
  name: 'zhangsan'
}

var name = 'The Window'

object.say_name = function() {
  var name = 'The inner'
  const that = this // 方法调用模式，此时 this 指向 object
  const sayName = function() {
    console.log('this.name', this.name) // The window(浏览器环境下)
    console.log('that.name', that.name) // zhangsan
  }

  sayName() // 以函数的调用的模式
}

// 以方法的形式调用 say_name
object.say_name()

```

通过上面两个例子可以看出来，以函数的形式调用函数的时候，函数内部的 this 通常指向全局。

### 构造器调用模式

- 当一个函数用作构造函数时（使用 new 关键字），它的 this 被绑定到正在构造的新对象。
- 如果在一个函数前面带上 new 来调用，那么背地里将会创建一个连接到该函数的 prototype 成员的新对象，同时，this 会被绑定到那个新对象上。

```
// 创建一个名为 Person 的构造器函数。它构造一个带有 name 属性的相关对象。
var Person = function(string) {
  this.name = string
}

// 给 Person 的所有对象实例都提供一个 getName 的公共方法
Person.prototype.getName = function() {
  return this.name
}

// 构造一个 Person 实例
var zhangsan = new Person('zhangsan')

console.log(zhangsan.getName()) // zhangsan
```

- 约定
  函数命名需要首字母大写

### Apply 调用模式

- apply() 方法调用一个具有给定 this 值的函数，以及作为一个数组（或类似数组对象）提供的参数。

apply 方法让我们构建一个参数数组传递给调用函数。它也允许我们选择 this 的值。apply 接受两个参数，第一个是要绑定给 this 的值，第二个就是一个参数数组

```
const add = (x, y) => x + y
const arr = [3, 4]
console.log(add.apply(null, arr)) // 7

const lisi = Person.prototype.getName.apply({ name: 'lisi' })
console.log('lisi:', lisi) // lisi: lisi

```

apply() 的参数为空时，默认调用全局对象。因此，这时的运行结果为 0，证明 this 指的是全局对象。

### 例题讲解

在面试笔试中，经常会碰到下面这道题。

```
var name = 'The Window'

var object = {
  name: 'My Object',

  getNameFunc: function() {
    var that = this // 方法调用模式，此时this被绑定到object对象上
    console.log('this.name:::', this.name)  // My Object
    return function() {
      console.log(name) //  The Window （这里不会产生闭包）匿名函数的执行通常具有全局性，因此this对象通常指向window
      console.log(this.name) // The Window
      return that.name // My Object 闭包，在该函数被调用的时候可以访问到外层变量that
    }
  }
}

console.log(object.getNameFunc()())
```

注释已经写的很清楚了，在此不再赘述。

关于 this 的学习到此就差不多了，查阅了很多的文献和参考资料，然后总结出了自己的一些想法，希望能够帮助到大家，另外也是对自己学习的一种记录和总结，文中有不当之处，欢迎指正讨论，一起来交流。
（完）

### 参考文献：

1、<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this" >MDN-this</a>

2、《JavaScript 高级程序设计（第三版）》- Nicholas C.Zakas

3、《JavaScript 语言精粹（修订版）》- Douglas Crockford

4、<a href="http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html">Javascript 的 this 用法-阮一峰</a>

### 推荐阅读

1、<a href="http://www.ruanyifeng.com/blog/2018/06/javascript-this.html">JavaScript 的 this 原理</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
