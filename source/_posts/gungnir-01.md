title: "前端调试工具:Gungnir"
date: 2015-04-17 09:58:32
tags:
- 前端
- Gungnir
---

搞前端的,特别是负责广告脚本开发的,经常接到需求:"xxx页面广告出不来/xx页面有脚本错误,赶紧帮忙看下".这时候就得把页面内容下载到本地,打开fiddler,然后配置好代理,再用编辑器打开调试.

天天遇到这种事情的话,就会觉得fiddler的这些操作还不够简单.而且fiddler也不能把代理设置按项目分类,都是堆在一起,不能有效的分类存储代理设置以备下次使用.

而且作为码农,日常工作中觉得麻烦的重复性操作,能简化得一定要想尽办法简化.

诸如此类的问题,加上刚好在学习angularjs和node-webkit,于是就花时间做了一个类似fiddler的小工具,不求功能比fiddler强,只求更方便自己使用,并且在有需要时要做成跨平台也比较容易.

关于这个工具的名字:刚开始做的那段时间刚好在看北欧神话,于是就拿奥丁的永恒之枪当名字,寓意找bug百发百中.

### 界面介绍
![](http://oneaboveall.qiniudn.com/v8ua2xeq6gktktyemg0fzsz9qk.png)

顶部是不同的功能面板开关,目前只有NETWORK面板,HOST和SETTING还没做好,暂时屏蔽.

右上是检查更新.

左边是项目资源管理以及工具条,工具条按钮功能分别为:`打开项目`,`运行服务器`,`新增代理`,`打开控制台`,`刷新项目`,`保存(ctrl+s)`.右键点击选中的条目会出现快捷菜单.底部蓝色横条会显示一些操作提示信息.

右侧大面板是文本编辑器

### 项目资源管理界面
项目按文件夹来划分,同一时间只能打开一个项目.可以通过将文件夹拖动到软件窗口,或者通过工具条的打开项目按钮来选择要打开的项目目录.

项目的根目录会默认生成一个zproject.json文件,用来保存代理配置等一些项目设置,这样你每次打开该文件夹都会还原上次的工作状态.

项目项的右键菜单包含了一些常用的功能.其中`预览页面`会有两种情况,如果该项或该项的父目录被设置了代理,则会通过代理的地址打开,否则使用本地地址打开文件.


### 文本编辑器功能
Gunnir集成了前端开发常用的语言的文本编辑器功能(使用开源的编辑器codemirror),目前我的开发都已经使用自己做的这个工具作为IDE了,因为这样很多功能都可以按自己的需要去改造.
目前支持的语言列表:`html`,`js`,`css`,`sass`,`php`,`coffeescript`,`markdown`,`aspx`

其中对js的支持功能会多一些,包括语法提示,jshint等

![](http://oneaboveall.qiniudn.com/v20z3wq47abjf6o1qp5nstwgbv.png)
光标移动到当前标识符并按ctrl+i会显示注释


![](http://oneaboveall.qiniudn.com/ii8izuv36wr60skp5zqjg67gmn.png)
自动列出成员列表


一些常用的快捷键功能说明:
`CTRL+W` 关闭当前文件
`CTRL+TAB` 切换到下个文件
`CTRL+K` 注释选中代码
`CTRL+N` 取消注释
`CTRL+L` 跳转到指定行
`CTRL+F` 搜索
`CTRL+G` 跳转到下个搜索结果
`CTRL+M` 格式化代码
`ALT+.` 跳转到定义

### 代理功能

为了尽可能的简化操作,代理的创建方式可以有多种,选择哪种取决于你的应用场景.

#### 自动下载线上文件
如果是临时需要调试线上的某个页面,这是最常用的方法.比如假设现在要调试http://www.17173.com

点击工具条上的新增代理设置按钮,填写要调试的页面地址,点击保存.
![](http://oneaboveall.qiniudn.com/i1lsiowztk9shhsuwp7zwjmy7r.png)

程序会自动把页面内容下载到本地,并按按路径存放
![](http://oneaboveall.qiniudn.com/ydhguzf0qj7l24p6amja0guwn4.png)

我们修改index.html的页面title为"Gungnir测试",点击工具条上的`启动服务器`按钮(开启软件时默认是运行的).然后刷新页面,就可以看到页面已经被代理到本地文件了(如果使用chrome浏览器并且未代理成功,请检查chrome的代理设置是不是被插件托管了,如果是,则先切换到`使用系统代理设置`)

![](http://oneaboveall.qiniudn.com/05gl8v8j1ct053r4zx0l0hiksx.png)

如果要删除代理,右键点击被代理的文件`index.html`,然后选择`代理设置`->`移除代理`即可

#### 使用本地已有文件
最经常见的情况是要将线上的文件代理到本地已有的未压缩版本.

先在项目里刚才自动生成的www.17173.com目录里新增一个index.js文件
然后随便配置一个不存在的域名
![](http://oneaboveall.qiniudn.com/v9o0od19dxjkb6gxwei6nw523a.png)
右键点击index.js,选择`预览页面`
![](http://oneaboveall.qiniudn.com/tvl5qyzdwvpr2lui9hh666ed34.png)


#### 代理整个目录
在`www.17173.com`目录下新增两个文件`a.js`,`b.js`
选中文件夹,随便设置一个代理
![](http://oneaboveall.qiniudn.com/525m8z7yntf649d4rr5otm70vy.png)
右键`www.17173.com`文件夹->`预览页面`
![](http://oneaboveall.qiniudn.com/7rirljd6kowdnk6lqzp7yn6zux.png)


#### 执行文件内容后返回结果
这个功能在前后端配合开发里会比较有用.后端接口已经定义好,但是尚未实现,就可以用这种方法做一些mock.

设置代理的时候,可以勾选`返回代码执行结果`
![](http://oneaboveall.qiniudn.com/dbrccllon5vzpgu0675lzr9c0u.png)

代理程序会把脚本里的代码当作一个函数执行后把结果作为内容返回.该函数包含一个参数`query`,表示url里的参数

函数签名如下:
```
//query为url参数,例如:
// www.1.com?a=1&b=2
// 则query = {a:1,b:2}
function(query){
	//文件里的内容会被当作脚本放在此函数里运行
}
```

拿`http://www.17173.com`的+1接口来举例.为了防止刷票,这个接口同一个ip每天只能点击一次

接口地址
![](http://oneaboveall.qiniudn.com/05gfefrv2gm34v5zsg54gufwuq.png)

再次点击,就会返回已投过票
![](http://oneaboveall.qiniudn.com/zax1i60q773hx46c7g9ibusj1y.png)

现在要移除这个限制,我们自己写代码来返回需要的mock数据.

先代理到本地,勾选`返回代码执行结果`
![](http://oneaboveall.qiniudn.com/p8zn0ipf55ovfqjkjd40vyzj8a.png)

然后编辑`index.js`,模拟实现`+1`的功能.
```
//存放点击数
var support = window.sessionStorage.getItem('support') || 1189;
//+1
support++;
window.sessionStorage.setItem('support',support);

var result = {
  flag:1,
  ajaxId:0,
  support:support,
  oppose:0
};
return query.jsonp + '('+JSON.stringify(result)+')';
```

每次刷新接口:http://hits.17173.com/support/support_opb.php?jsonp=fn&action=1&channel=90103&web_id=1419588860&kind=1 ,点击数都会+1了.
![](http://oneaboveall.qiniudn.com/2ks00l5hf2z0vrsbq960s02j4i.png)

### NETWORK面板

NETWORK面板可以监控电脑上所有http请求的细节
![](http://oneaboveall.qiniudn.com/nk1tttre5aa1c6xiz30kjatgfo.png)

点击加号可以给该请求设置代理
![](http://oneaboveall.qiniudn.com/lcwui1np0uw0c415c95esibzf4.png)

### 项目地址
https://github.com/benqy/Gungnir