---
title: web应用下js命名空间的实现
date: 2018-07-09 17:38:01
categories:
- javascript
tags:
- js
---

前言：
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;javascript作为一门弱类型语言没有创建和管理模块功能和命名空间机制，但是我们在开发大型项目过程中会遇到全局变量被污染的情况，因此我们必须通过js代码去模拟命名空间机制。

简单的命名空间
```
var school = school || {};    // 创建school命名空间，命名空间如果存在，不覆盖原有的对象
school.students = {};    // student命名空间已经定义
(function(students) {
    //这里业务的代码 
	function Subject(){
		//业务逻辑
	};
	function Grade(){
	    //业务逻辑 
	};
    // 将公共API导到上面定义的命名空间中
    students.Subject = Subject;
    students.Grade = Grade;
    // 这里也不需要返回值
})(school.students);
```

作为一种替代方案，定义全局命名空间，通过模块直接设置那个对象，大部分插件都用的上面的思路，同时也存在这弊端。  
首先我们在开发的过程中每一个加载文件都要反复去写这样的代码，虽然空间不覆盖，但是一旦索引一样仍然会覆盖原来的值，改命名空间缺乏最基本的判断，因此我们需要封装更加健壮的命名空间代码

### 创建模块
```
Module.createNamespace = function (name, version) {
    if (!name) throw new Error('name required');
    if (name.charAt(0) == '.' || name.charAt(name.length-1) == '.' || name.indexOf('..') != -1) throw new Error('illegal name');
    
    var parts = name.split('.');
    
    var container = Module.globalNamespace;//初始化对象，默认值为window对象
    for (var i=0; i<parts.length; i++) {
        var part = parts[i];
		//过滤掉相同的模块代码
        if (!container[part]) container[part] = {};
        container = container[part];
    }
    
    var namespace = container;
    if (namespace.NAME) throw new Error('module "'+name+'" is already defined');
    namespace.NAME = name;
    if (version) namespace.VERSION = version;
    //将模块信息存储到Module.modules对象中
    Module.modules[name] = namespace;
    return namespace;
};
```

### 初始化模块
```
var Module;

if (!!Module && (typeof Module != 'object' || Module.NAME)) throw new Error("NameSpace 'Module' already Exists!");

Module = {};

Module.NAME = 'Module';
Module.VERSION = 0.1;
//默认允许通过的模块名
Module.EXPORT = ['require', 
                 'importSymbols'];

Module.EXPORT_OK = ['createNamespace', 
                    'isDefined',
                    'modules',
                    'globalNamespace'];
                    
Module.globalNamespace = this;

Module.modules = {'Module': Module}; 
```
### 模块检测，版本检测
```
// check name is defined or not 
Module.isDefined = function (name) {
    return name in Module.modules;
};
// check version 
Module.require = function (name, version) {
    if (!(name in Module.modules)) throw new Error('Module '+name+' is not defined');
    if (!version) return;
    
    var n = Module.modules[name];
    if (!n.VERSION || n.VERSION < version) throw new Error('version '+version+' or greater is required');
};
```
### 导入业务代码
```
Module.importSymbols = function (from) {
    if (typeof from == 'string') from = Module.modules[from];
    var to = Module.globalNamespace; //dafault
    var symbols = [];
    var firstsymbol = 1;
    
    if (arguments.length>1 && typeof arguments[1] == 'object' && arguments[1] != null) {
        to = arguments[1];
        firstsymbol = 2;
    }
    
    for (var a=firstsymbol; a<arguments.length; a++) {
        symbols.push(arguments[a]);
    }
    
    if (symbols.length == 0) {
        //default export list
        if (from.EXPORT) {
            for (var i=0; i<from.EXPORT.length; i++) {
                var s = from.EXPORT[i];
                to[s] = from[s];
            }
            return;
        } else if (!from.EXPORT_OK) {
            // EXPORT array && EXPORT_OK array both undefined
            for (var s in from) {
                to[s] = from[s];
                return;
            }
        }
    }
    
    if (symbols.length > 0) {
        var allowed;
        if (from.EXPORT || from.EXPORT_OK) {
            allowed = {};
            if (from.EXPORT) {
                for (var i=0; i<from.EXPORT.length; i++) {
                    allowed[from.EXPORT[i]] = true;
                }
            }
            if (from.EXPORT_OK) {
                for (var i=0; i<from.EXPORT_OK.length; i++) {
                    allowed[from.EXPORT_OK[i]] = true;
                }
            }
        }

    }
    //import the symbols
    for (var i=0; i<symbols.length; i++) {
        var s = symbols[i];
        if (!(s in from)) throw new Error('symbol '+s+' is not defined');
        if (!!allowed && !(s in allowed)) throw new Error(s+' is not public, cannot be imported');
        to[s] = from[s];
    }
}
```

### demo实例
```
Module.createNamespace('Hongru.me', '0.1.1');
var demo = {};
demo.EXPORT = ['define'];
demo.define = function () {
    this.NAME = '__me';
}
Module.importSymbols(demo, Hongru);//把me名字空间下的标记导入到Hongru名字空间下
```

### 代码完整封装
```
var Module;
//check Module --> make sure 'Module' is not existed
if (!!Module && (typeof Module != 'object' || Module.NAME)) throw new Error("NameSpace 'Module' already Exists!");

Module = {};

Module.NAME = 'Module';
Module.VERSION = 0.1;

Module.EXPORT = ['require', 
                 'importSymbols'];

Module.EXPORT_OK = ['createNamespace', 
                    'isDefined',
                    'modules',
                    'globalNamespace'];
                    
Module.globalNamespace = this;

Module.modules = {'Module': Module}; 

// create namespace --> return a top namespace
Module.createNamespace = function (name, version) {
    if (!name) throw new Error('name required');
    if (name.charAt(0) == '.' || name.charAt(name.length-1) == '.' || name.indexOf('..') != -1) throw new Error('illegal name');
    
    var parts = name.split('.');
    
    var container = Module.globalNamespace;
    for (var i=0; i<parts.length; i++) {
        var part = parts[i];
        if (!container[part]) container[part] = {};
        container = container[part];
    }
    
    var namespace = container;
    if (namespace.NAME) throw new Error('module "'+name+'" is already defined');
    namespace.NAME = name;
    if (version) namespace.VERSION = version;
    
    Module.modules[name] = namespace;
    return namespace;
};
// check name is defined or not 
Module.isDefined = function (name) {
    return name in Module.modules;
};
// check version 
Module.require = function (name, version) {
    if (!(name in Module.modules)) throw new Error('Module '+name+' is not defined');
    if (!version) return;
    
    var n = Module.modules[name];
    if (!n.VERSION || n.VERSION < version) throw new Error('version '+version+' or greater is required');
};
// import module
Module.importSymbols = function (from) {
    if (typeof from == 'string') from = Module.modules[from];
    var to = Module.globalNamespace; //dafault
    var symbols = [];
    var firstsymbol = 1;
    
    if (arguments.length>1 && typeof arguments[1] == 'object' && arguments[1] != null) {
        to = arguments[1];
        firstsymbol = 2;
    }
    
    for (var a=firstsymbol; a<arguments.length; a++) {
        symbols.push(arguments[a]);
    }
    
    if (symbols.length == 0) {
        //default export list
        if (from.EXPORT) {
            for (var i=0; i<from.EXPORT.length; i++) {
                var s = from.EXPORT[i];
                to[s] = from[s];
            }
            return;
        } else if (!from.EXPORT_OK) {
            // EXPORT array && EXPORT_OK array both undefined
            for (var s in from) {
                to[s] = from[s];
                return;
            }
        }
    }
    
    if (symbols.length > 0) {
        var allowed;
        if (from.EXPORT || from.EXPORT_OK) {
            allowed = {};
            if (from.EXPORT) {
                for (var i=0; i<from.EXPORT.length; i++) {
                    allowed[from.EXPORT[i]] = true;
                }
            }
            if (from.EXPORT_OK) {
                for (var i=0; i<from.EXPORT_OK.length; i++) {
                    allowed[from.EXPORT_OK[i]] = true;
                }
            }
        }
    }
    //import the symbols
    for (var i=0; i<symbols.length; i++) {
        var s = symbols[i];
        if (!(s in from)) throw new Error('symbol '+s+' is not defined');
        if (!!allowed && !(s in allowed)) throw new Error(s+' is not public, cannot be imported');
        to[s] = from[s];
    }
}
```
