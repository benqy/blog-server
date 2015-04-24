title: "自己动手制作更好用的markdown编辑器-03"
date: 2015-04-24 09:28:59
tags:
- nw.js
- Hexo MD
---

[上一篇](http://benq.im/2015/04/23/hexomd-02/)我们实现了文件的新建,保存,打开功能.

在这篇里我们将实现以下功能:

1. 系统模块,包含一些软件的设置和存储功能
2. 记录上次打开的文件
3. 编辑器样式选择

## 系统模块

跟之前的`studio`模块类似,我们在modules模块下增加`system`目录.
![](http://oneaboveall.qiniudn.com/gcgkul0h2yki6rkjp8kz5ftq8s.png)
比studio多了`model.js`文件,用来实现系统模块的一些功能.

在`app.js`里加载`system`模块
```
angular.module('hmd', ['ui.router','hmd.directives','hmd.studio','hmd.system'])
...
//引入模块
hmd.regModule('studio');
hmd.regModule('system');
...
```

**路由、导航栏**
`angular.js`用得不熟,导航栏的状态根据route来切换一直不知道怎么实现比较优雅.
我直接在`app.js`里增加了一个导航栏切换的方法,每个route的onEnter事件里自行调用这个方法.
```
  //TODO:更优雅的导航栏切换逻辑
  hmd.changeStatus =  function (state) {
    var $navList = $('#navlist');
    $navList.find('li').removeClass('active');
    $navList.find('.' + state).addClass('active');
  };
```

`system/route.js`
```
  hmd.system.config(function ($stateProvider, $urlRouterProvider) {
    $stateProvider
      .state('system', {
        url: "/system",
        templateUrl: "modules/system/views/system.html",
        controller: 'system',
        onEnter: function () {
          hmd.changeStatus('system');
        }
      });
  });
```


`studio/route.js`
```
...
onEnter: function () {
  hmd.changeStatus('studio');
}
...
```

然后在`index.html`里配置好导航
```
...
<ul class="nav navbar-nav" id="navlist" >
  <li class="studio"><a href="#/studio">编辑器</a></li>
  <li class="system"><a href="#/system">系统设置</a></li>
</ul>
...
```
导航栏最终效果:
![](http://oneaboveall.qiniudn.com/gxmx0vzj5mkuihhi7hfbjvf3cj.png)
## 记录上次打开的文件

每次打开文件都会被记住,下次重新启动程序时将默认打开最后一次打开的文件.

**system设置的读取和保存**

我们先在`system/model.js`实现保存和读取设置的功能.
```
  var util = require('./helpers/util'),
      fs = require('fs'),
      system = hmd.system,
      //存储设置的文件
      dataFile = system.dataPath + '\\system.json';

  //初始化存储目录
  if (!fs.existsSync(system.dataPath)) {
    fs.mkdirSync(system.dataPath);
  }
 	
  //默认设置
  var defaultSystemData = {
    //最后一次打开的文件
    lastFile: null
  };
  
  //读取设置
  system.get = function () {
    return $.extend(defaultSystemData,util.readJsonSync(dataFile));
  };

  //保存设置
  system.save = function (data) {
    data = data || defaultSystemData;
    util.writeFileSync(dataFile, JSON.stringify(data));
  };
  
    //设置最后打开的文件
  system.setLastFile = function (filepath) {
    var systemData  = system.get();
    systemData.lastFile = filepath;
    system.save(systemData);
  };
```

`system`实现了`get`和`save`方法,所有的设置都存储在一个简单的对象里,代码里并没有对这个对象做缓存,每次都是从文件里读取,因为这简单的文件还远达不到影响读取速度的情况.

然后我们修改`editor`的`setFile`方法,暴露`setFiled`事件给外部使用.
```
    //设置当前文件
    setFile:function(filepath){
      if(filepath && fs.existsSync(filepath)){
        var txt = util.readFileSync(filepath);
        this.filepath = filepath;
        this.cm.setValue(txt);
        this.fire('setFiled',this.filepath);
      }
      else{
        this.filepath = null;
        this.cm.setValue('');
        this.fire('setFiled');
      }
    }
```

最后修改`studio/directives.js`的`hmdEditor`,实现这个功能.
```
studio.directive('hmdEditor', function () {
    return function ($scope, elem) {
    	//读取最后一次打开的文件
      var systemData = hmd.system.get();
      hmd.editor.init({el:elem[0]},systemData.lastFile);
      //保存最后一次打开的文件
      hmd.editor.on('setFiled',function(filepath){
        hmd.system.setLastFile(filepath);
      });
    ...
```

## 编辑器样式选择

### 样式修改表单
样式文件在目录[app/lib/codemirror/theme](https://github.com/benqy/hexomd/tree/master/app/lib/codemirror/theme).
目录里每一个样式文件代表一种编辑器样式,还记得当初实现`editor`的`init`时,样式已经是通过配置传入的.
```
...
if(options.theme != 'default'){
	$('head').append('<link href="lib/codemirror/theme/'+options.theme+'.css" rel="stylesheet" />');
}
...
```
现在我们只要把theme参数存储到配置里,并提供给用户修改就可以.

在`system/model.js`里的默认配置增加一个`theme`字段.
```
...
  //默认设置
  var defaultSystemData = {
    //最后一次打开的文件
    lastFile: null,
    //当前样式
    theme:'ambiance'
  };
...
```
修改`system/views/system.html`模版,增加样式表单
```
<div class="content studio-wrap">
  <form class="system-form" name="systemForm">
    <div class="form-group">
      <label>编辑器样式</label>
      <select ng-model="systemSetting.theme" name="theme">
        <option value="ambiance">ambiance</option>
        <option value="mbo">mbo</option>
        <option value="neat">neat</option>
      </select>
    </div>    
    <button type="submit" class="btn btn-default" ng-click="save(systemSetting)">保存</button>
  </form>
</div>
```
这里的select控件我们先写了3个选项.现在先实现这个修改样式的功能,等完成这个功能后再把选项列表做成自动生成.

对应的`system/controllers.js`(开发了3天了,终于第一次用到controller了)
```
  system.controller('system', function ($scope) {
    $scope.systemSetting = system.get();
    $scope.save = function (systemSetting) {
      system.save(systemSetting);
    };
  });
```
`controller`里读取system的数据,并赋值给`$scope.systemSetting`,用于表单的数据绑定.由于`angular`实现了数据的双向绑定,因此用户编辑表单时,绑定的数据也会跟着更新.这样我们的`save`方法里只要将表单绑定的数据保存回system即可.

button按钮绑定save方法`ng-click="save(systemSetting)"`.

这里可以稍微感受到`angular`让我们节省了很多工作量.

### 自动生成select列表

修改`controller`
```
  var system = hmd.system,
  	fs = require('fs');
  system.controller('system', function ($scope) {
    //读取theme目录,生成样式列表
    var files = fs.readdirSync('./app/lib/codemirror/theme'),themes={};
    files.forEach(function (file) {
      if(~file.indexOf('.css')){
      	file = file.replace('.css','');
        themes[file] = file;
      } 
    });
    $scope.themes = themes;
    $scope.systemSetting = system.get();
    $scope.save = function (systemSetting) {
      system.save(systemSetting);
    };
  });
```
从`theme`目录里读取所有样式列表,生成键值对,然后赋值给`$scope.themes`

修改视图模版:
```
<select name="theme" ng-model="systemSetting.theme"  ng-options="k as v for (k, v) in themes">
</select>
```
`ng-options="k as v for (k, v) in themes"`是angular的[绑定语法](https://docs.angularjs.org/api/ng/directive/select)

这样我们就实现了样式列表的自动读取,用户如果想自定义样式,只要在`app/lib/codemirror/theme`目录新增一个样式文件,并写上自己的样式就可以在系统设置里选择自定义的样式了.


## 总结
今天实现了记忆最后一次打开的文件以及样式选择的功能,并且第一次使用了`angular`的`controller`,感受到了`angular`双向数据绑定的强大.

我们的程序又更好用一些了(但是随着界面变多,又更丑了,太为难了).

## 附件
[本篇程序打包](http://pan.baidu.com/s/1dD6eN1R)
[项目地址](https://github.com/benqy/hexomd)