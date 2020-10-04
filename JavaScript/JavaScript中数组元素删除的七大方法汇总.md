---
title: JavaScript中数组元素删除的七大方法汇总
date: 2016-10-08 19:16:15
updateed: 2016-10-08 19:16:15
tags: [javascript]
---

<center>
  本文总结了ES5中常见的数组删除方法
<center>
</br>
</center>
  标签：#javascript
</center>

<!-- more -->

在 JavaScript 中，除了 Object 之外，Array 类型恐怕就是最常用的类型了。与其他语言的数组有着很大的区别，JavaScript 中的 Array 非常灵活。今天我就来总结了一下 JavaScript 中 Array 删除的方法。大致的分类可以分为如下几类：
1、length
2、delete
3、栈方法
4、队列方法
5、操作方法
6、迭代方法
7、原型方法

下面我对上面说的方法做一一的举例和解释。

一、length

    JavaScript中Array的length属性非常有特点一一它不是只读的。因此，通过设置这个属性可以从数组的末尾移除项或添加新项，请看下面例子：

```
var colors = ["red", "blue", "grey"];	//创建一个包含3个字符串的数组
colors.length = 2;
console.log(colors[2]);  //undefined
```

二、delete 关键字

```
var arr = [1, 2, 3, 4];
delete arr[0];

console.log(arr);   //[undefined, 2, 3, 4]
```

可以看出来，delete 删除之后数组长度不变，只是被删除元素被置为 undefined 了。

三、栈方法

```
var colors = ["red", "blue", "grey"];
var item = colors.pop();
console.log(item);      //"grey"
console.log(colors.length);    //2
```

可以看出，在调用 Pop 方法时，数组返回最后一项，即"grey"，数组的元素也仅剩两项。

四、队列方法

队列数据结构的访问规则是 FIFO（先进先出），队列在列表的末端添加项，从列表的前端移除项，使用 shift 方法，它能够移除数组中的第一个项并返回该项，并且数组的长度减 1。

```
var colors = ["red", "blue", "grey"];
var item = colors.shift();
console.log(item);      //"red"
console.log(colors.length);    //2
```

五、操作方法
splice()恐怕要算最强大的数组方法了，他的用法有很多种，在此只介绍删除数组元素的方法。在删除数组元素的时候，它可以删除任意数量的项，只需要指定 2 个参数：要删除的第一项的位置和要删除的项数，例如 splice(0, 2)会删除数组中的前两项。

```
var colors = ["red", "blue", "grey"];
var item = colors.splice(0, 1);
console.log(item);      //"red"
console.log(colors);    //["blue", "grey"]
```

六、迭代方法

所谓的迭代方法就是用循环迭代数组元素发现符合要删除的项则删除，用的最多的地方可能是数组中的元素为对象的时候，根据对象的属性例如 ID 等等来删除数组元素。下面介绍两种方法：

第一种用最常见的 ForEach 循环来对比元素找到之后将其删除：

```
var colors = ["red", "blue", "grey"];

colors.forEach(function(item, index, arr) {
    if(item == "red") {
        arr.splice(index, 1);
    }
});
```

第二种我们用循环中的 filter 方法：

```
var colors = ["red", "blue", "grey"];

colors = colors.filter(function(item) {
    return item != "red"
});

console.log(colors);	//["blue", "grey"]
```

代码很简单，找出元素不是"red"的项数返回给 colors（其实是得到了一个新的数组），从而达到删除的作用。

七、原型方法

通过在 Array 的原型上添加方法来达到删除的目的：

```
Array.prototype.remove = function(dx) {

    if(isNaN(dx) || dx > this.length){
        return false;
    }

    for(var i = 0,n = 0;i < this.length; i++) {
        if(this[i] != this[dx]) {
            this[n++] = this[i];
        }
    }
    this.length -= 1;
};

var colors = ["red", "blue", "grey"];
colors.remove(1);

console.log(colors);    //["red", "grey"]
```

在此把删除方法添加给了 Array 的原型对象，则在此环境中的所有 Array 对象都可以使用该方法。尽管可以这么做，但是我们不推荐在产品化的程序中来修改原生对象的原型。道理很简单，如果因某个实现中缺少某个方法，就在原生对象的原型中添加这个方法，那么当在另一个支持该方法的实现中运行代码时，就可能导致命名冲突。而且这样做可能会意外的导致重写原生方法。

在此，我汇总了 JavaScript 的 Array 中常用的删除元素的方法，欢迎大家来补充。

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
