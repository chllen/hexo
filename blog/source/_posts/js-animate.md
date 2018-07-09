---
title: 原生js仿jq实现链式调用，延时加载
date: 2018-07-03 15:24:22
categories:
- javascript
tags:
- js
- jquery
---
### 前言  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们在开发项目的过程中，经常会遇到类似于jq的animate()，queue(),end(),或者是angular.js的promise等延时加载和链式调用的方法，理解其原理有利于我们编程能力的提高  

### animate用法  
[具体参数详解](http://www.runoob.com/jquery/eff-animate.html);  

### animate动画实例  
```
	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="utf-8"> 
	<title>animate动画效果</title>
	<script src="https://cdn.bootcss.com/jquery/1.10.2/jquery.min.js">
	</script>
	<script> 
	$(document).ready(function(){
		var div=$("div");
		$("#start").click(function(){
			div.animate({height:300},"slow");
			div.animate({width:300},"slow");
			div.queue(function () {
				div.css("background-color","red");  
				div.dequeue();
			});
			div.animate({height:100},"slow");
			div.animate({width:100},"slow");
		});
		$("#stop").click(function(){
			div.clearQueue();
		});
	});
	</script> 
	</head>
	<body>
	
	<p>
	<button id="start">开始动画</button>
	<button id="stop">停止动画</button>
	</p>
	<div style="background:#98bf21;height:100px;width:100px;position:relative"></div>
```

使用原生js实现动画实例的效果，首先我们需要下面两个封装代码
#### js队列封装
```
 var Queue = function() {
        this.list = [];
    };
    Queue.prototype = {
        constructor: Queue,
        queue: function(fn) {
            this.list.push(fn)
            return this;
        },
        wait: function(ms) {
            this.list.push(ms)
            return this;
        },
        dequeue: function() {
            var self = this,
                list = self.list;
            self.isdequeue = true;
            var el = list.shift() || function() {};
            if (typeof el == "number") {
                setTimeout(function() {
                    self.dequeue();
                }, el);
            } else if (typeof el == "function") {
                el.call(this)
                if (list.length) {
                    self.dequeue();
                } else {
                    self.isdequeue = false;
                }
            }


        }
    };
```

#### 封装动画框架：
```
 function startMov(obj, json, fn) {//fn回调函数
        clearInterval(obj.timer);//执行动画之前清除动画
        var flag = true;//是否动画都完成了
        obj.timer = setInterval(function () {
            for (var attr in json) {
                var icur = 0;
                if (attr == 'opacity') {
                    icur = Math.round(parseFloat(getStyle(obj, attr)) * 100);//转换成整数,并且四舍五入下
                    //计算机在计算小数的时候往往是不准确的！
                } else {
                    icur = parseInt(getStyle(obj, attr));
                }
                var speed = 0;
                speed = (json[attr] - icur) / 8;//缓冲运动效果
                speed = speed > 0 ? Math.ceil(speed) : Math.floor(speed);
                if (icur != json[attr]) {
                    flag = false;
                }
                if (attr == 'opacity') {
                    obj.style.filter = 'alpha(opacity:' + (icur + speed) + ')';
                    obj.style.opacity = (icur + speed) / 100;
                } else {
                    obj.style[attr] = icur + speed + 'px';
                }
                if (flag) {
                    clearInterval(obj.timer);
                    if (fn) {
                        fn();
                    }
                }
            }
        }, 30);
    }
    function getStyle(obj, attr)
    {
        if (obj.currentStyle) {
            return obj.currentStyle[attr];//IE获取样式属性
        } else {
            return getComputedStyle(obj, false)[attr];
        }
    }
```

#### 链式调用

通过对上面原生js的改造，实现和动画实例一样的效果：
```
<!DOCTYPE html>
	<html>
	<head>
	<meta charset="utf-8"> 
	<title>animate动画效果</title>
	<script> 
	window.onload=function(){
		var selecter=document.getElementById('selecter');
		document.getElementById('start').onclick=function(){
			selecter.animate({height:300,width:100}).animate({width:300}).end(1000);
			selecter.queue(function (){
				selecter.style.backgroundColor = "red";
				selecter.dequeue();
			});
			selecter.animate({height:100}).animate({width:100})
		};
		document.getElementById('stop').onclick=function(){
			selecter.clearQueue();
		};
	};
	</script> 
	</head>
	<body>
	
	<p>
	<button id="start">开始动画</button>
	<button id="stop">停止动画</button>
	</p>
	<div id="selecter" style="background:#98bf21;height:100px;width:100px;position:relative"></div>
	</body>
	<script>
	var Queue = function(obj) {
			this.obj = obj || this;
			this.obj.task = [];
			this.obj.params = [];
			this.instance = null;
    };
    Queue.prototype = {
        constructor: Queue,
        queue: function(fn) {
			if(typeof fn == "function"){
				this.obj.task.push(fn)
			}else{
				this.obj.params.push(fn);
			}
            return this;
        },
        wait: function(ms) {
			this.obj.params.push('');
            this.obj.task.push(ms);
            return this;
        },
        dequeue: function() {
			var self = this;
				obj = self.obj,
                task = obj.task;
				params = obj.params;
			//isdequeue属性判断正在运行队列，true为正在运行
            obj.isdequeue = true;
            var el = task.shift() || function (){};
			var args = params.shift() || [];
			Array.prototype.unshift.call(args,this,obj);
            if (typeof el == "number") {
                setTimeout(function() {
                    self.dequeue();
                }, el);
            } else if (typeof el == "function") {
                el.apply(this,args);
            }
        }
    };
	//单例模式
	Queue.getInstance = function(name){
	   if(!this.instance){
			this.instance = new Queue(name);
		}
		return this.instance;
	};
	//原生js中，animate是Element对象的属性
	Element.prototype.animate = function (){
		var q = Queue.getInstance(this);
		q.queue(arguments);
		q.queue(startMov);
		//避免重复运行
		if(!q.obj.isdequeue){
			q.dequeue();
		}
		//方便链式调用
		return this;
	}
	
	Element.prototype.queue = function (fn){
		var q = Queue.getInstance(this);
		//没有参数传入，数据为空
		q.queue(null);
		q.queue(fn);
		return this;
	}
	
	Element.prototype.dequeue = function (){
		var q = Queue.getInstance(this);
		q.dequeue();
		//保证动画停止后还可以继续运行
		obj = q.obj;
		if(!obj.task.length){
			obj.isdequeue = false;
		}
	}
	
	Element.prototype.clearQueue = function (){
		new Queue(this);
		return false;
	}
	
	Element.prototype.end = function (ms){
		//延时加载
		var q = Queue.getInstance(this);
		q.wait(ms);
		return this;
	}
	
	function startMov(q,obj, json, fn) {//fn回调函数
        clearInterval(obj.timer);//执行动画之前清除动画
        obj.timer = setInterval(function () {
			var isComplate = [];
            for (var attr in json) {
				//可枚举，自有属性
				if(!json.hasOwnProperty(attr)){
					continue;
				}
                var icur = 0;
                if (attr == 'opacity') {
                    icur = Math.round(parseFloat(getStyle(obj, attr)) * 100);//转换成整数,并且四舍五入下
                    //计算机在计算小数的时候往往是不准确的！
                } else {
                    icur = parseInt(getStyle(obj, attr));
                }
                var speed = 0;
                speed = (json[attr] - icur) / 8;//缓冲运动效果
                speed = speed > 0 ? Math.ceil(speed) : Math.floor(speed);
				//判断动画是否完成
				flag = icur != json[attr] ? false : true;
				isComplate.push(flag);
                if (attr == 'opacity') {
                    obj.style.filter = 'alpha(opacity:' + (icur + speed) + ')';
                    obj.style.opacity = (icur + speed) / 100;
                } else {
                    obj.style[attr] = icur + speed + 'px';
                }
            }
			var ret = isComplate.every(function (val){
				return val === true;
			});
			if (ret) {
                clearInterval(obj.timer);
                if (fn) {
                    fn();
                }
				//动画框架植入队列，保障动画的链式调用
				obj = q.obj;
				q.dequeue();
				if(!obj.task.length){
					obj.isdequeue = false;
				}
            }
        }, 30);
    }
    function getStyle(obj, attr)
    {
        if (obj.currentStyle) {
            return obj.currentStyle[attr];//IE获取样式属性
        } else {
            return getComputedStyle(obj, false)[attr];
        }
    }
	</script>
```
注意事项：
	1. 代码实现了对animate的全局调用，支持链式调用，与jq的区别是动画效果仅支持淡入、淡出、以及缓冲运动
	2. 代码通过js的push()和shift()方法,运用队列先入先出的调用方式，完成动画效果的链式调用
	3. 在书写代码过程中为防止多次创建新实例，采用单例的设计模式，有效避免了因为多次调用初始化对列
	4. 参数必须要和函数一起存在对列里，无参数，传入值为空
	5. 通过element.end(timer)可以实现对列的延时加载
