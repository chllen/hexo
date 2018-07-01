---
title: 使用Github+hexo搭建个人博客教程
categories:
- javascript
tags:
- js
- node.js
- github  
---

1. 什么是hexo
    >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[hexo(中文网站)](https://hexo.io/zh-cn/)是一个快速, 简洁且高效的博客框架. 让上百个页面在几秒内瞬间完成渲染. Hexo支持Github Flavored Markdown的所有功能, 甚至可以整合Octopress的大多数插件. 并自己也拥有强大的插件系统.

2. 使用Github搭建博客的优点：
    1. 不需要域名和服务器       
    2. 提供免费代码仓库服务，方便版本控制       
    3. 推送上线一次完成    
    
让我们先来预览一篇博客,这是我自己用Github搭建的：[https://chllen.github.io](https://chllen.github.io)  
### 以下操作基于Windows环境

#### 首先博客搭建的环境﻿:  
>[node.js](https://nodejs.org/en/) 因为整个博客框架是基于node.js的，所以必须安装node.js环境，安装过程中一路Next即可。  
[Git客户端](https://git-scm.com/downloads/) Git用来将hexo的相关文件部署到Github上去，安装过程一路Next。
#### 安装hexo框架  
先新建一个blog的空文件夹,进入blog文件右键进入git命令行，如图所示：
	
![](update/01.png)  

打开Git的命令窗口，在命令窗口输入安装命令后，然后回车：  

>npm install -g hexo

初始化hexo命令：  

>hexo init

安装相关的依赖包，输入下面的命令，回车：  

>npm install
 
接着依次执行以下命令:  
 
>hexo g //生成博客静态文件  
hexo s //启动本地服务 

命令执行完后浏览器访问[http://localhost:4000](http://localhost:4000) 或者 [127.0.0.1:4000](127.0.0.1:4000),就会看到hexo的初始界面，本地hexo搭建完成  

### 将本地博客部署到Github   
#### 创建Github号  
首先我们需要到Github官网创建一个账号，创建链接：[Github](https://github.com/)

#### 创建仓库  
创建完账号后我们新建一个Repository  
![](update/02.png)  
	
![](update/03.png)
#### 部署文件到Github
接下来就是部署文件到Github了。  
用文本编辑器打开hexo文件夹下面的_config.yml文件，该文件的最下面找到关键字deploy,然后修改成下面这样，用我自己的做案例  
```
deploy:
	type: git
	//repository: https://github.com/chllen/chllen.github.io.git  //https存在bug
	repo: git@github.com:chllen/chllen.github.io.git //推荐使用SSH
	branch: master
```
ps:每个冒号后面都有一个空格，修改的时候别忘了。  

#### 修改完保存
但是这样还不能连接到 github ，我们还需要配置SSH，找到路径C:\Users\Administrator\.ssh，如果已经存在SSH Keys ，直接删除.ssh 文件夹下的所有的文件，如下图。	  
![](update/04.png)  

然后继续在blog文件夹下面输入下面的指令:  
>ssh-keygen -t rsa -C "邮箱地址"
	 
然后再回车三次，等命令执行完，再输入以下指令：
	
>eval `ssh-agent -s`   
ssh-add
	
然后输入指令拷贝Key，将id_rsa.pub的密钥复制到github里面
![](update/05.png)

![](update/06.png)

![](update/07.png)  

接下来测试ssh是否配好了，输入下面的指令，会提示你输入yes/no你输入yes就行，这样ssh就配好了，接下来我们就可以将项目部署到Github上面了。  
	
>ssh -T git@github.com
	  
![](update/08.png)  

生成静态文件，将文件上传至github
>hexo g  
hexo d //上传到github

但是输入hexo d可能会报ERROR Deployer not fount： git错误，这是因为没有安装hexo-deployer-git这个模块，导致Git不能识别该命令，输入下面指令安装该模块即可。  
>npm install hexo-deployer-git --save

博客已部署到github里面  
终于有了属于自己的博客：[https://chllen.github.io](https://chllen.github.io)

###改变hexo主题
>初始的landscape主题，对它实在是不感冒，hexo可以自定义主题，同时也提供了一些好看的[hexo主题](https://github.com/hexojs/hexo/wiki/Themes),我们选择nexT主题来美化我们的博客！ 
 
![](update/09.png)  

在自己的博客目录下，进入主题文件夹themes,输入git命令：  
>git clone https://github.com/iissnan/hexo-theme-next next

![](update/10.png)  
如图，hexo主题已经下载完成，我们还需要做如下配置才能使用nexT主题：  
1.进入博客根目录，配置_config.yml，找到theme选项，把主题切换为next .如下图,将原来的landscape删掉，改为next,然后保存即可。
！[](update/11.png) 
 
2.配置_config.yml，站点基本信息：
![](update/13.png)  
注意将language信息改为: zh-Hans,可以在\themes\next\languages\zh-Hans.yml查看配置的语言字典信息。  

3.打开你的博客地址/themes/next找到里面的配置文件_config.yml，选择自己想要的风格：
![](update/12.png)  

4. 最后在git里，依次输入：
>hexo clean  
hexo g  
hexo d  

博客主题已经修改完成，迫不及待想浏览博客了吧！  

### 发布文章，文章分类设置文章标签
<前面我们已经完成了hexo博客的搭建，现在我们需要发布文章，这又该怎么做呢？首先，我们要知道一般网站发布文章需要借助富文本编辑器，将文章的内容保存到后台，hexo博客不存在后台数据库，他采用markdown文件编辑，然后程序生成静态html文件，因此我们需要先下载个[markdown编辑器](http://markdownpad.com/)，我们开始编辑文章吧。

#### 创建.md文件
>hexo new hexo-1  

执行hexo创建一个hexo-1.md文件，生成的markdown文件保存在\source\_posts\目录下；然后通过markdown编辑器进行编辑，在网上熟悉mardown简单语法也是很重要的。

#### 创建文章分类
首页菜单栏默认只有首页和归档，我们想添加分类和标签，需要做下面的设置：
1.进入\themes\next\_config.yml,配置主题菜单信息。  
![](update/14.png)  
菜单内容：categories: /categories/ || th 分别代表：分类：分类文件访问路径||图标  

2.生成tags和categories文件
>hexo page new categories  

这样source\categories\index.md文件生成完成，修改文件头部信息，如图：
！[](update/15.png)   
标签内容生成方式同上。

3.设置文章的分类和标签
进入前面生成的hexo-1文件,配置文章头部信息，如图：
![](update/16.png)  
成功创建javascript分类和js，node.js，github 标签  

4.编辑好文章内容后，最后在git里，依次输入：  
>hexo g  
hexo d  

我们所期待的分类和贴上标签的文章创建完成，继续向前。  

### 如何上传文章本地图片
我们在编辑文章的时候，需要上传一些本地图片，但是我们只能操作md编辑器，因此我们还需要一些配置才能将图片显示在博客。

1 把根目录文件_config.yml 里的post_asset_folder:这个选项设置为true  

2 在你的hexo目录下执行这样一句话npm install hexo-asset-image --save，这是下载安装一个可以显示本地图片的插件 

3.运行hexo new "xxxx"来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹，本地图片复制到该文件夹内，最后通过编辑文章时，利用格式：\!\[你想输入的替代文字](xxxx/图片名.jpg)

这样发布文章已经没有坑了，希望对大家有所帮主!










	


	
