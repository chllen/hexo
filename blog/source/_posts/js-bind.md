---
title: javascript中bind()方法的使用与实现
date: 2018-07-08 23:55:40
categories:
- javascript
tags:
- js
---
前言：  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bind方法，字面理解绑定意思，是在ECMAScript 5中新增的方法，但在ES3也可以模拟bind()。bind所做的就是自动封装函数在函数自己的闭包中，这样我们可以捆绑上下文（this关键字）和一系列参数到原来的函数。

### bind()基本用法
```
var p = new Point(1, 2);
p.toString(); // '1,2'
	
var YAxisPoint = Point.bind(null, 1);
	
var axisPoint = new YAxisPoint(5);
axisPoint.toString();
axisPoint instanceof Point; // true
axisPoint instanceof YAxisPoint; // true
new Point(17, 42) instanceof YAxisPoint; // true
YAxisPoint.isPrototypeOf(new Point(17, 42));//false
```
上面例子中Point和YAxisPoint共享原型，因此使用instanceof运算符判断时为true。

instanceof和isPrototypeOf的区别：  
instanceof运算符：左操作数是待检测其类的对象，有操作数是定义类的构造函数。如果o继承自c.prototype,则表达式o instanceof c值为true，这里可以不是直接继承，如果o所继承自另一个对象，后一个对象继承自c.prototype,这个表达式运算结果也是true；
isPrototypeOf方法：检测对象原型链上是否存在特定的原型对象，<span style="color:red">可以检测是否存在构造函数为中介的方法</span>

### bind实现
[参考博客](https://segmentfault.com/a/1190000002662251)
可以通过如下代码实现这种绑定
```
function bind(){
  if(f.bind) {
	return f.bind();}
  else {
	return function (){
	  return f.apply(this,arguments);
	}
  }
}
```
考虑到函数柯里化的情况，我们可以构建一个更加健壮的bind()；
```
if(!Function.prototype.bind){
  Function.prototype.bind = function(context){
    var args = Array.prototype.slice.call(arguments, 1);
	self = this;
	bound = function(){
	  var innerArgs = Array.prototype.slice.call(arguments);
	  var finalArgs = args.concat(innerArgs);
		//this代表window对象，表示self为全局方法
		return self.apply(this, finalArgs);
	  };
	bound.prototype = self.prototype;
	return bound;		
  };
}
```
### arguments的使用
arguments对象，可以获取function的参数值，为什么我们要使用Array.prototype.slice.call(arguments),而不直接使用arguments.slice()呢？  
我们需要认识到，arguments是一个类数组对象，我们给出类数组的对象的代码：  
```
var a={};//从一个常规空对象开始
//添加一些属性，称为“类数组”
var i = 0;
while(i<10){
  a[i] = i*i;
  i++;
}
//类数组不能一定要添加length属性
a.length = i;
```
上面的类数组对象使用a.slice(),会出现typeError的错误。

[Array.prototype.slice为什么能将类数组对象转为真正的数组？](https://www.cnblogs.com/henryli/p/3700945.html)


