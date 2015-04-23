title: "自己动手制作更好用的markdown编辑器-01"
date: 2015-04-21 06:11:21
tags:
- nw.js
- Hexo MD
---

前段时间用hexo重新搭了个人博客，顺便写了个简单的博客搭建[教程](http://hmj.name/2015/04/13/github-hexo-blog/).

用[markdown](http://zh.wikipedia.org/zh/Markdown)写起博客流畅很多，但是用了几个markdown编辑器，都没有一个适合自己使用的。于是就想自己动手做一个，当然不是完全从0开始做，语法高亮和markdown解析都用的是开源的项目.
<!--more-->
从这篇开始，我会把整个开发过程记录成系列随笔,因此开发进度较为缓慢.

博客写得少,像这样写长一点的随笔就有点混乱,看不懂的请用力喷,我会努力改进.

## 简介

先介绍下开发过程中用到的一些比较重要的开源项目：
1. [nw.js](https://github.com/nwjs/nw.js),原名node-webkit，用webkit和node来做基于web技术的跨平台客户端软件.
2. [CodeMirror](codemirror.net),基于web技术实现的文本编辑器，实现了大部分的IDE功能以及几乎全部你会用到的语言的支持.目前我日常开发都是用这个IDE,甚至在做hexomd这个项目时用的IDE也是CodeMirror做的.
3. [angularjs](angularjs.org),google的mvvm开发框架,这个相信不用我多做介绍.我用的不熟,觉得好用就拿来即用,没有深入的了解过.

关于这些开源项目的使用，我在这系列文章里不会详细解释，如果有疑问，可以去看官网的入门教程和wiki，当然也欢迎讨论.

## 项目结构
![](http://oneaboveall.qiniudn.com/yrbc5bjmablkrccu0fqsc6su7h.png)

图片里的是我目前的项目结构，大概讲解一下一些目录和文件的用途。

1. `icudtl.dat`,`nw.exe`,`nw.pak`
这3个是nw.js在windows运行所必须的文件.

2. `package.json`
nw.js的配置文件
```
{
  "name": "HexoMD",
  "description": "Markdown for hexo",
  "main": "app/index.html",//程序入口页面
  "author": "hmjlr123@gmail.com",
  "license": "MIT",
  "directories": {
    "test": "no"
  },
  "devDependencies": {},
  //窗口配置
  "window": {
    "title": "HexoMD",
    "icon": "app/img/logo.png", //logo
    "toolbar": true, //是否显示地址栏工具条(调试的时候启用)
    "frame": false, //是否显示程序边框
    "width": 1000, //默认宽度
    "height": 700, //默认高度
    "position": "center", //启动时在屏幕中的位置
    "min_width": 600, //最小宽度
    "min_height": 400 //最小高度
  }
}
```

3. `app目录`
程序的所有源代码的根目录.

4. `app/lib`
存放angular,jquery,codemirror等开源库/框架的源代码

5. `app/helpers`
存放一些node的工具函数

6. `app/modules`
程序代码在这个目录,按功能模块分成不同的子目录.
`modules/app.js`是整个程序的入口点

7. `app/package.json`
node模块配置,注意与上层的package.json意义不同

8. `app/index.html`
程序的主界面窗口

## 程序主界面

[index.html](https://github.com/benqy/hexomd/blob/master/app/index.html)
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Hexo Markdown</title>
  <link href="css/bootstrap.css" rel="stylesheet">
  <link href="lib/codemirror/lib/codemirror.css" rel="stylesheet" />
  <link href="css/index.css" rel="stylesheet">
</head>
<body>
  <!--导航栏-->
  <nav class="navbar navbar-inverse navbar-fixed-top">
   	省略...
  </nav>
  <!--模块视图区域-->
  <article class="container app" ui-view ng-animate="'view'"></article>
  <!--工具栏-->
  <footer class="tool"></footer>
  <!--end codemirror-->
  <script src="lib/jquery-2.1.3.js"></script>
  <script src="lib/angular.js"></script>
  <script src="lib/angular-ui-router.js"></script>
  <!--程序入口函数-->
  <script src="modules/app.js"></script>
  <script>
    //初始化angular,hmd为自定义的根模块名
    angular.bootstrap($('body'), ['hmd']);
  </script>
</body>
</html>
```
只贴出部分代码.以后的所有代码也类似,都只会把重要的贴出来,并给出完整的链接.

![](http://oneaboveall.qiniudn.com/8l5ewvbdsbsp2wuxs4qejhflay.png)
界面采用比较简洁的三栏布局,分别为`导航栏`、`内容区`、`状态栏/工具条`.
最顶部的地址栏只有在开发的时候为了方便调试才开启,发布时会关闭掉.

## 拖动窗口
为了美观,我们在配置里去掉了系统自带的边框.因此要实现自定义的拖动窗口功能还需要增加一些设置.
所谓的设置,其实只要加上对应的样式即可,功能都由nw.js实现了.

```
.navbar{
  -webkit-app-region: drag;
}
```
带有此样式的元素可以作为窗口的拖拽区域,并且双击时最大化/还原窗口.

```
.navbar .navbar-collapse a {
    -webkit-app-region: no-drag;
}
```
被标志为可`drag`的容器里的链接将不可点击,因此要特别为链接加上`no-drag`

另外为了让程序看起来更像客户端一点,我默认禁用掉了文本选择,防止一些被作为按钮的a标签的文本被选中
```
html {
    height: 100%;
    -webkit-user-select: none;
}
```

## app.js
[app.js](https://github.com/benqy/hexomd/blob/master/app/modules/app.js)作为程序的入口点,定义了整个项目代码的结构,需要特别拿出来说明一下.

```
angular.module('hmd', ['ui.router','hmd.studio'])
```
定义angular模块,`modules`所有的业务模块都会放到单独的子目录里,如这里注册的`hmd.studio`


```
	//模块根目录
	var baseModuleDir = './app/modules/';
  //引入模块,模块内js文件会被自动加载到页面中
  hmd.regModule = function (name, reqModule) {
    hmd[name] = angular.module('hmd.' + name, reqModule || []);
    hmd[name].moduleName = name;
    //模块存储数据的目录
    hmd[name].dataPath = hmd.storeDir + '\\' + hmd[name].moduleName;
    fs.readdirSync(baseModuleDir + name)
    .forEach(function (file) {
      if (~file.indexOf('.js')) {
        document.write('<script src="modules/' + name + '/' + file + '"></script>');
      }
    });
  };
```
`regModule`方法实现最简单的模块载入,自动加载模块内的所有脚本到页面中,并为每个模块赋予一个单独的数据存储目录`dataPath`


```
hmd.storeDir =  require('nw.gui').App.dataPath;
```

程序的数据存储目录

## 导航栏按钮
导航栏右边有4个按钮,分别为:`检查更新`、`最小化`、`最大化`、`关闭`
```
...
<!--导航栏功能按钮-->
<div class="btn-group window-tool">
  <a class="btn rectbtn" href="javascript://" title="点击检查更新">
  	<i class="glyphicon  mdfi_action_system_update_tv"></i></a>
  <a class="btn rectbtn" href="javascript://"><i class="glyphicon glyphicon-minus"></i></a>
  <a class="btn rectbtn" href="javascript://"><i class="glyphicon glyphicon-fullscreen"></i></a>
  <a class="btn rectbtn" href="javascript://"><i class="glyphicon glyphicon-remove"></i></a>
</div>
...
```
检查更新等以后再实现.现在先实现后面3个功能
因为这3个功能是全局的,因此在modules根目录新建`directives.js`用于实现全局的[Directive](https://docs.angularjs.org/guide/directive).
```javascript
(function () {
  var gui = require('nw.gui'), win = gui.Window.get(),winMaximize = false;
  angular.module('hmd.directives', [])
  //最小化窗口
  .directive('hmdMinisize', [function () {
    return function (scope, elem) {
      $(elem[0]).on('click', function () {
        win.minimize();
      });
    };
  }])
  //最大化与还原窗口
  .directive('hmdMaxToggle', [function () {
    return function (scope, elem) {
      //窗口最大化和还原时会触发对应的事件,在事件里去控制按钮样式.
      //TODO:这里的实现应该可以优化得更优雅一点,以后再说
      win.on('maximize', function () {
        winMaximize = true;
        $(elem[0]).find('i').removeClass('glyphicon-fullscreen').addClass('glyphicon-resize-small');
      });
      win.on('unmaximize', function () {
        winMaximize = false;
        $(elem[0]).find('i').removeClass('glyphicon-resize-small').addClass('glyphicon-fullscreen');
      });
			//切换窗口状态
      $(elem[0]).on('click', function () {
        if (winMaximize) {
          win.unmaximize();
        }
        else {
          win.maximize();
        }
      });
    };
  }])
  //关闭应用程序
  .directive('hmdClose', [function () {
    return function (scope, elem) {
      $(elem[0]).on('click', function () {
        require('nw.gui').Window.get().close();
      });
    };
  }]);
})();
```
定义了全局directive模块`angular.module('hmd.directives', [])`,实现了3个`Directive`.

接下来将directive应用到按钮上
```
...
<a class="btn rectbtn" href="javascript://" hmd-minisize><i class="glyphicon glyphicon-minus"></i></a>
<a class="btn rectbtn" href="javascript://" hmd-max-toggle><i class="glyphicon glyphicon-fullscreen"></i></a>
<a class="btn rectbtn" href="javascript://" hmd-close><i class="glyphicon glyphicon-remove"></i></a>
...
```

将脚本引用`<script src="modules/directives.js"></script>`添加到`index.html`的app.js之后

app.js里的angular模块注册里增加hmd.directives模块`angular.module('hmd', ['ui.router','hmd.directives','hmd.studio'])`

刷新程序,三个按钮已经生效.

## 实现简单的markdown编辑器
先在页面添加相应的codemirror脚本引用
```
	...
  <footer class="tool"></footer>
  <!--codemirror-->
  <script src="lib/codemirror/lib/codemirror.js"></script>
  <script src="lib/codemirror/addon/mode/overlay.js"></script>
  <script src="lib/codemirror/addon/edit/continuelist.js"></script>
  <script src="lib/codemirror/mode/markdown/markdown.js"></script>
  <script src="lib/codemirror/mode/gfm/gfm.js"></script>
  <!--end codemirror-->
  <script src="lib/jquery-2.1.3.js"></script>
  ...
```

然后在`modules`目录下新增`studio`子目录,所有编辑器功能都在这个模块里实现.
在`app.js`里增加加载`studio`模块的代码
```
hmd.regModule('studio');
```

每个子模块一般都会包含`route.js`,`controllers.js`,`directive.js`这三个基本的angular功能.以及`views`子目录,用于存放模块用到的html视图
![](http://oneaboveall.qiniudn.com/yagnklzjdtb7q38x7j8yjio340.png)
`studio`模块多了一个`editor.js`,我们将编辑器的一些基本功能封装在这个脚本里

### 定义路由
[route.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/route.js)
```
  hmd.studio.config(function ($stateProvider, $urlRouterProvider) {
    $stateProvider
      .state('studio', {
        url: "/studio",
        templateUrl: "modules/studio/views/studio.html",
        controller: 'studio'
      });
  });
```
修改`app.js`,将默认路由指定到`/studio`模块
```
  hmd.config(function ($stateProvider, $urlRouterProvider) {
    $urlRouterProvider.otherwise("/studio");
  });
```

### 实现controller
[controllers.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/controllers.js)
```
  var studio = hmd.studio;
  studio
    .controller('studio', function ($scope, $state, $stateParams) {
      console.log('stuido controller');
    });
```

### 添加视图模版
[views/studio.html](https://github.com/benqy/hexomd/blob/master/app/modules/studio/views/studio.html)
```
<div class="content studio-wrap">
   <textarea name="" cols="30" rows="10"></textarea>
</div>
```

重新打开应用,可以看到模块跳到了studio路由,并且执行了对应的控制器
![](http://oneaboveall.qiniudn.com/1qwbu4vbhgvkuwtwk75jiygbym.png)

### 实现editor
[editor.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/editor.js)

```
  var util = require('./helpers/util');
  var defaultConfig = {
    theme: 'ambiance',
    mode: 'gfm',
    lineNumbers: false,
    extraKeys: {"Enter": "newlineAndIndentContinueMarkdownList"},
    dragDrop: false,
    autofocus: true,
    lineWrapping: true,    
    foldGutter: true,
    styleActiveLine: true
  };

  hmd.editor = {
    init: function (options,filepath) {
      var el = options.el,txt,me = this;
      options = $.extend({}, defaultConfig, options);
      //编辑器样式文件动态加载,用于以后增加样式选择功能
      if(options.theme != 'default'){
        $('head').append('<link href="lib/codemirror/theme/'+options.theme+'.css" rel="stylesheet" />');
      }
      this.cm = this.cm || CodeMirror.fromTextArea(el, options);
      //指定要打开的文件,如果未指定,则保存时会弹出文件选择对话框
      filepath && this.setFile(filepath);
      //编辑器内容修改时触发change事件
      this.cm.on('change', function (em, changeObj) {
        me.hasChange = true;
        me.fire('change', {
          em: em,
          changeObj: changeObj
        });
      });
      //绑定按键
      this.cm.addKeyMap({
        "Ctrl-S": function () {
          me.save();
        }
      });
    },
    //设置当前文件
    setFile:function(filepath){
      var txt = util.readFileSync(filepath);
      this.filepath = filepath;
      this.cm.setValue(txt);
    },
    //弹出保存文件对话框
    saveAs:function(){
      hmd.msg('保存新文件');
    },
    //保存文件
    save: function () {
      var txt = this.cm.getValue();
      if(this.filepath){
        util.writeFileSync(this.filepath, txt);
        this.hasChange = false;
        var fileNameArr = this.filepath.split('\\');
        hmd.msg('文件:' + fileNameArr[fileNameArr.length - 1] + '保存成功!');
      	this.fire('save');
      }
      else{
        this.saveAs();
      }
    },
    events: {},
    fire: function (eventName, obj) {
      var me = this;
      this.events[eventName] && this.events[eventName].forEach(function (fn) {
        fn.call(me,obj);
      });
    },
    on: function (eventName, fn) {
      this.events[eventName] = this.events[eventName] || [];
      this.events[eventName].push(fn);
    }
  };
```

我们将编辑器的实现封装在`hmd.editor`这个对象上.

编辑器的模式设置为[GFM](https://help.github.com/articles/github-flavored-markdown/).

### 实现directive
[directives.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/directives.js)
```
var studio = hmd.studio;
  
studio.directive('hmdEditor', function () {
  return function ($scope, elem) {
    //第二个参数为测试用的本地md文件,因为选择文件的功能还没实现.你可以改成你电脑上的文件.
    hmd.editor.init({el:elem[0]},'E:\\Temp\\test\\test.md');
  };
});
```
定义了`'hmd-editor`,用于绑定hmd.editor的调用.
在视图模版里调用`hmd-deitor`
```
<div class="content studio-wrap">
   <textarea name="" cols="30" rows="10" hmd-editor></textarea>
</div>
```
刷新应用,可以看到textarea已经变成markdown编辑器,按`ctrl+s`保存会有简单的提示.

### 最终效果
![](http://oneaboveall.qiniudn.com/le40oe7t44ih3tenq51wxt64rt.png)

## 总结
到目前为止,只是搭建了开发环境,实现了基础的编辑器功能,还完全不能真正的使用.
接下来几篇暂定计划是:
* 打开文件,保存新文件,系统设置等基本功能.
* 自动更新功能.
* 实时预览窗口.
* 自动上传图片.
* 表情功能.
* 集成hexo命令.

## 附件
[本篇结果打包](http://pan.baidu.com/s/1dDvVsh7)
[github项目地址](https://github.com/benqy/hexomd)