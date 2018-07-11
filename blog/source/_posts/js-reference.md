---
title: js中值引用和地址引用
date: 2018-07-10 11:30:39
categories:
- javascript
tags:
- js
---

前言：
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上篇文章中Module.globalNamespace对象利用值引用的方法简化了程序，同时也增加了开发者阅读程序的难度，对于初学者而言值引用和地址引用理解起来还是有点困难的。

### 概念
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;引用类型：对象可以拥有属性和方法，并且我们可以修改其属性和方法。引用对象存放的方式是：在栈中存放对象变量标示名称和该对象在堆中的存放地址，在堆中存放数据。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;堆：队列优先,先进先出；由操作系统自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;栈：先进后出；动态分配的空间 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收，分配方式倒是类似于链表。   
以上都属于计算机基础部分。

通过概念，我们一头雾水︿(￣︶￣)︿，我们直接上代码：
```
var a = [1,2,3];
b = a;
b[2] = 1;
console.log(a);//1,2,1
console.log(b);//1,2,1
```
a的值传给了b，b修改数组索引值，a的值也发生了变化，这是为什么了，通过概念我们可以看出数组作为一种引用类型对象，将值存在堆中，b的修改并没有修改到栈中的对象名，而是堆中的值，而a和b指向同一个堆,因此在b索引的值改变，a的值也会改变。
```
var a = [1,2,3];
b = a;
b = 1;
console.log(a);//1,2,3
console.log(b);//1
```
与第一个例子不同的是，a的值传给了b，但是修改b后，a没有任何变化，因为b的引用地址发生了变化，改变了栈的地址指向，因此b中值变化不会影响到a的值。

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
这样大家就能理解的更清楚了，本来to指向的地址是a对象,但是后来又改成了a.modules的引用地址，此时to和a.modules为同一个栈，to修改了索引test的值，a.modules也发生了改变，而a.modules对象是a的嵌套对象，因此a的值也发生了改变。

最后我们看一下，两个相同对象之间的比较：
```
var a= {'modules':{'test':1},'me':2};
var b= {'modules':{'test':1},'me':2};
alert(a == b);//返回false
a = b;
alert(a == b);//返回true
```
a和b的值存在不同的栈中，因此他们不相等。