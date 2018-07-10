---
title: js中值引用和地址引用
date: 2018-07-10 11:30:39
categories:
- javascript
tags:
- js
---

前言：
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天写这篇文章是因为上篇文章中Module.globalNamespace对象利用值引用的方法简化了程序，同时也增加了程序阅读的难度，对于初学者而言值引用和地址引用理解起来还是有点困难的。

### 概念
>引用类型指的是对象。可以拥有属性和方法，并且我们可以修改其属性和方法。引用对象存放的方式是：在栈中存放对象变量标示名称和该对象在堆中的存放地址，在堆中存放数据。

通过概念，我们一头雾水︿(￣︶￣)︿，通过代码分析：
```
var a = [1,2,3];
b = a;
b[2] = 1;
console.log(a);//1,2,1
console.log(b);//1,2,1
```
上面例子可以看出，a的值传给了b，b修改数组索引值，a的值也发生了变化，这是为什么了，通过概念我们可以看出数组作为一种引用类型对象，将值存在堆中，b的修改并没有修改到栈中的对象名，而是堆中的值，而a和b指向同一个堆,因此在b索引的值改变，a的值也会改变。
```
var a = [1,2,3];
b = a;
b = 1;
console.log(a);//1,2,3
console.log(b);//1
```
这个例子，我们可以看出，虽然a的值传给了b，但是修改b后，a没有任何变化，因为b没有修改索引，而是直接改变了栈的指向，因此b的变化不会影响到a。

```
var a={'modules':{'test':1},'me':2};
function importSymbols(from){
	var to = a;
	to = from;
	to['test'] = 2;
	console.log(a);//{'modules':{'test':2},'me':2}
}
importSymbols(a.modules);
```
我们初看到这样的代码真的很难理解，我将代码做下面的改造：
```
var a = {'modules':{'test':1},'me':2};
to = a.modules;
to['test'] = 2;
console.log(a);//{'modules':{'test':2},'me':2}
```
这样大家就能理解的更清楚了，本来to指向的地址是a但是后来又改成了a.modules，修改to的索引值，a.modules堆中的值也发生了变化，a.modules是对象a的一部分。

最后我们看一下，两个相同对象之间的比较：
```
var a= {'modules':{'test':1},'me':2};
var b= {'modules':{'test':1},'me':2};
alert(a == b);//返回false
a = b;
alert(a == b);//返回true
```
a和b的值存在不同的栈中，因此他们不可能相等。