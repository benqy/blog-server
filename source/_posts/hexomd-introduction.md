title: "用nw.js开发markdown编辑器-功能介绍"
date: 2015-10-29 11:48:55
tags:
---

做这个markdown编辑器是因为自己平常用markdown写文档写得比较多,网上找的都不太好用,而且都不支持扩展开发,无法实现自己所需的一些定制化功能.


从今年四月份开始,陆陆续续的利用一些零碎时间做到现在,所需的基本功能都实现了,因此现在写一篇总结把功能介绍一遍.

这个编辑器的主要特色是`自动上传图片`,`文档分享`以及`导出pdf`

![](http://7ximoo.com1.z0.glb.clouddn.com/ye03atv1o84u09mbkby4s8jrio.png)

### 功能列表

* 基本markdown语法
* 自动更新.
* 实时预览窗口
* 编辑器,预览,代码段样式选择和自定义
* 自动上传图片
* emoji表情.
* 导出html,pdf文件.
* 目录语法
* 一键文档分享

### 基本markdown语法
> Markdown 是一种轻量级标记语言，创始人为约翰·格鲁伯（John Gruber）。它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档”。    —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)


本编辑器的markdown语法基于github的`GitHub Flavored Markdown`扩展出更多语法,因此基本的语法直接看Github的[帮助文档](https://help.github.com/articles/github-flavored-markdown/)

### 自动更新
目前提供最新版的下载地址是 [v1.0.0.0](http://pan.baidu.com/s/1eQEw1Wm)
第一次使用请直接下载,以后只要点击右上角的![](http://7ximoo.com1.z0.glb.clouddn.com/3dc6nzg54kne9wrbto19lonp3z.png)按钮就会检查更新,如果有新版本,则会提示是否要更新.

### 实时预览窗口
`系统设置`里提供两种方式的预览窗口选择,如图

![](http://7ximoo.com1.z0.glb.clouddn.com/uanc9uxdlutmx65nmelfhxpy9o.png). 

双屏幕的用户可以选择在新窗口打开预览. 编写文档时,预览窗口会实时的更新并滚动到当前编辑位置.
预览窗口可使用![](http://7ximoo.com1.z0.glb.clouddn.com/icf4n6qvrttkewasrrv22sq4u0.png)按钮切换开关.
### 样式选择与自定义
![](http://7ximoo.com1.z0.glb.clouddn.com/0bx09pxux1q69vcw0b5yhuqgax.png)

如图,可以选择编辑器样式和预览窗口样式以及预览窗口里的代码段样式.
软件预设了一些样式供选择,用户也可以直接编辑样式文件自定义.这3个样式存放的目录分别在软件app目录下的:

` 编辑器样式`: app\lib\codemirror\theme
` 预览样式目录`: \app\css\previewtheme
`代码段样式目录`: app\node_modules\highlight.js\styles

可以直接修改里面已有的样式,也可以直接新增文件,下拉菜单会自动读取所有样式文件供选择.

### 云存储配置
由于自动上传图片和一键文档分享需要用到云存储(目前用的是`七牛`),因此这里先讲下系统设置里的云存储设置.

首先得注册一个[七牛帐号](http://www.qiniu.com/).

进入后台,新建二个共享空间,一个用于存储图片,另一个存储共享文档(其实也可以用同一个,看个人习惯)
![](http://7ximoo.com1.z0.glb.clouddn.com/z1gu22y4uqnf8iae1m7fygvptx.png)

选择新建的空间,点击`空间设置`>`域名设置`,查看自动分配的域名
![](http://7ximoo.com1.z0.glb.clouddn.com/pby78cgpeaa1c41w4n2d2v5vlh.png)

回到后台首页,点击`账号设置`,可以查看`accessKey(AK)`和`SecretKey(SK)`

![](http://7ximoo.com1.z0.glb.clouddn.com/q2eqjgnuof5ka3x6pygd2h23s1.png)

在系统设置里配置好这几个字段
![](http://7ximoo.com1.z0.glb.clouddn.com/yiww9xyyyg944k6cvugncavhrg.png)
我把我空间的密钥遮住了,大家请填上自己的空间密钥

### 自动上传图片
将图片复制到剪贴板后(如qq截图,系统截屏等),直接在编辑器里粘贴图片,会自动将图片上传到配置好的七牛空间里.并在编辑器里填入markdown格式的图片引用,如图
![](http://7ximoo.com1.z0.glb.clouddn.com/g7dq1k4r8zx5h5j3qbfg4tpt4k.png)

图片名称是随机生成的(目前这样的话,用久了图片很乱,暂时想不到什么好办法可以不牺牲易用性,又方便分类管理图片).
我博客里所有图片都是这样的,写起博客来特方便.

### 一键文档分享
如果文档里有此格式的标签`[SHARE:文件名]`. 
则点击 ![](http://7ximoo.com1.z0.glb.clouddn.com/zov4s5ezjf25uvkenxe0v8dnjc.png)按钮时,会自动将文档解析成html,并上传到配置好的文档空间,然后在浏览器打开.文件名为标签里指定的文件名.

![](http://7ximoo.com1.z0.glb.clouddn.com/t81uqybwykmg3es8d6i9p8kw3i.png)

![](http://7ximoo.com1.z0.glb.clouddn.com/f3gxhwkyejj7nulkimp0z2kv5c.png)

### emoji表情功能.
目前支持[此表格](http://www.emoji-cheat-sheet.com/)里的所有emoji表情.
只要在写文档时,以这种格式 `:表情代号:`,就会被解析为对应emoji表情.如下面这些表情.
:+1: :shit: :-1: :point_right::ok_hand:
表情代号在上面的表格里查询

### 导出html,pdf文件功能.
点击![](http://7ximoo.com1.z0.glb.clouddn.com/6b1fob7kipjsjkcfgup3juqvyr.png)可以导出解析好的文档到html或者pdf文件.导出哪种类型,取决于你输入的后缀名(如果为pdf,则导出时需要等待几秒)
![](http://7ximoo.com1.z0.glb.clouddn.com/innvyknr1yno26znmsyri4yig4.png)

![](http://7ximoo.com1.z0.glb.clouddn.com/zr9vm4m9c852kg9zm0959daemi.png)

### 目录语法
文档里如果带有TOC标签
```
[TOC]
```
则会自动将h1~h6标签按嵌套结构解析为目录树,并替换显示在TOC标签位置

### 备注
[开发过程随笔](http://www.cnblogs.com/honghongming/)