---
title: JavaScript之数组五大迭代方法总结
date: 2016-10-08 19:16:15
updateed: 2016-10-08 19:16:15
tags: [javascript]
---

<center>
  本文总结了ES5中常见的数组迭代方法
<center>
</br>
</center>
  标签：#javascript
</center>

<!-- more -->

如果去问一个不太了解 JavaScript 数组的开发人员，JavaScript 的数组有多少种迭代方法，你可能得到的答案为，for/while/do-while...等等，这个是循环中的方法，和我们数组的迭代还是有一些区别的。虽然在数组中我们也可以用这些方法去迭代。但是，为了装逼为了飞，我们就来写一点带有脚本味道的代码吧！

    1、every(): 对数组中的每一项运行给定的函数，如果该函数对每一项都返回true，则结果返回true。
    2、filter(): 对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
    3、forEach(): 对数组中的每一项运行给定函数，这个方法没有返回值。
    4、map(): 对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
    5、some(): 对数组中的每一项运行给定函数，如果该函数任意一项返回true，则返回true。

    以上的方法都不会修改数组中包含的值。
    在这些方法中，最相似的是every()和some()。他们都用于查询数组中的项是否满足某个条件。对every来说，传入的函数必须对数组中的每一项都返回true，这个方法才返回true；否则，他就返回false。在这里我们总结为有假则假，都真才真。而some()方法则是只要传入的函数对数组中的任何一项返回true，就返回true。请看下面例子:

```
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

var everyResult = numbers.every(function(item, index, array) {
    return (item > 2);
});

var someResult = numbers.some(function(item) {
    return (item > 2);
});

console.log(everyResult);      //false
console.log(someResult);        //true
```

    需要提醒的是，在some()方法的函数中我并没有传入三个参数，而是不在函数体内用不到后两个参数，所以我省略了。以上的迭代调用了every()和some()，传入的函数只要给定项大于2就会返回true。对于every()，它返回false，因为数组中只有部分满足条件。对于some()，结果返回true，因为数组中至少有一项是大于2的。

    下面再来看看filter()函数，他利用指定的函数确定是否在返回的数组中包含某一项，就是说根据指定的函数来筛选符合条件的项组成新的数组并返回。例如，要返回一个所有值都大于2的数组，可以使用下面的代码：

```
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];
var filterResult = numbers.filter(function(item) {
    return (item > 2);
});

console.log(filterResult);	\\[3, 4, 5, 4, 3]
```

    这里，通过调用filter()方法创建并返回了包含3、4、5、4、3的数组，因为传入的函数对它们每一项都返回true。这个方法对查询符合某些条件的所有数组项非常有用。

    map()方法也返回一个数组，而这个数组的每一项都是在原始数组中的对应项上运行传入函数的结果。例如，可以给数组中的每一项乘以2，然后返回这些乘积组成的数组，如下所示：

```
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];
var mapResult = numbers.map(function(item) {
    return (item * 2);
});

console.log(mapResult);     //[2, 4, 6, 8, 10, 8, 6, 4, 2]
```

    以上代码返回的数组中包含给每个数乘以2之后的结果。这个方法适合创建包含的项与另一个数组一一对应的数组。

    最后一个方法就是forEach()。它只是对数组中的每一项运行传入的函数。这个方法没有返回值，本质上与用for循环迭代数组一样，来看一个例子：

```
var numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];
numbers.forEach(function(item, index, arr) {
    //这里执行一些操作
});
```

    这些数组方法通过执行不同的操作，可以大大方便处理数组的任务。还是那句话，在自己的脚本里面写一些有脚本味道的代码，能够提高代码的质量和可读性，甚至减少代码量。支持这些迭代方法的浏览器有IE9+、FireFox 2+、Safari 3+、Opera 9.5+和chrome。

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
