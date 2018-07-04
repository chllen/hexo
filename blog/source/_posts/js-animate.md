---
title: jquery  animate分析，以及队列在动画效果中的应用
date: 2018-07-03 15:24:22
tags:
---
###jquery animate基础知识

####animate定义
animate() 方法执行 CSS 属性集的自定义动画。
该方法通过 CSS 样式将元素从一个状态改变为另一个状态。CSS属性值是逐渐改变的，这样就可以创建动画效果。  
只有数字值可创建动画（比如 "height:'30px'"）。字符串值无法创建动画（比如 "background-color:red"）。  
提示：请使用 "+=" 或 "-=" 来创建相对动画,（例如："left:+=20"或者"left:-=30",这样运动轨迹相反）  

####animate用法
<$(selector).animate({styles},speed,easing,callback);  
或者 
<$(selector).animate({styles},{options});
[具体参数详解]();

###animate动画实例

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

如何实现animate的链式调用呢？
首先我们介绍一下，js的队列，代码如图：
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

封装动画框架：
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
			selecter.effect({height:300});
			selecter.effect({width:300});
			//selecter.queue(function () {
				//div.css("background-color","red");  
				//div.dequeue();
			//});
			//selecter.animate({height:100});
			//selecter.animate({width:100},"slow");
		};
		document.getElementById('stop').click=function(){
			selector.clearQueue();
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
            this.obj.task.push(ms)
            return this;
        },
        dequeue: function() {
            var self = this.obj,
                task = self.task;
				params = self.params;
            self.isdequeue = true;
            var el = task.shift() || function() {};
			var args = params.shift() || [];
			Array.prototype.unshift.call(args,self);
            if (typeof el == "number") {
                setTimeout(function() {
                    self.dequeue();
                }, el);
            } else if (typeof el == "function") {
                el.apply(this,args);
                if (task.length) {
                    self.dequeue();
                } else {
                    //self.isdequeue = false;
                }
            }
        }
    };
	Object.prototype.effect = function (){
		var q = new Queue(this);
		q.queue(arguments);
		q.queue(startMov);
		if(!q.obj.isdequeue){
			q.dequeue();
		}
	}
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
					q.dequeue();
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
