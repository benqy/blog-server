title: "自己动手制作更好用的markdown编辑器-02"
date: 2015-04-23 09:28:59
tags:
- nw.js
- Hexo MD
---

[上一篇](http://hmj.name/2015/04/21/hexomd-01/)我们搭建好了项目结构,简单的实现了第一个模块(studio)的基本功能,已经能够进行简单的markdown编辑.

在这篇里我们将实现以下功能:

1. 底部工具条UI,状态栏信息
2. 新建文件,打开文件,保存文件

## 工具条
由于工具条按钮绑定的都是studio模块下的功能,因此我把`index.html`上的工具条移动到了`studio`模块的视图模版`modules/studio/views/studio.html`里.

### 样式
```
<div class="content studio-wrap">
  <textarea name="" cols="30" rows="10" hmd-editor></textarea>
</div>
<footer class="tool">
  <!--状态栏消息-->
  <section class="msg" id="msg"></section>
  <section class="btn-group studio-btn-group">
    <a studio-newfile href="javascript://" class="btn btn-primary" style="border-radius:0;" title="新建文件"><i class="glyphicon glyphicon glyphicon-file"></i></a>
    <a studio-openfile href="javascript://" class="btn btn-primary" title="打开文件" ng-click="openTerminal()"><i class="glyphicon glyphicon-folder-open"></i></a>
    <a studio-save href="javascript://" class="btn btn-primary" title="保存更改" style="border-radius:0;"><i class="glyphicon glyphicon-floppy-disk"></i></a>
  </section>
</footer>
```
[CSS](https://github.com/benqy/hexomd/blob/master/app/css/index.css)写得比较难看,没什么好说的.样式里我大量使用了[calc](https://developer.mozilla.org/en-US/docs/Web/CSS/calc)这个功能,这在布局的时候非常的方便,比如:
```
body {
    height: calc(100% - 50px);
    overflow: hidden;
    color: #fff;
    background: #1E1E1E;
}
```
### 工具条截图
![](http://oneaboveall.qiniudn.com/3jb6jnygc9uan5wr7m4pekf8yq.png)
配色比较丑,一开始我是只在乎功能的,UI是我的弱项,我们还是先能用再好用最后才好看吧.
##  状态栏消息
状态栏消息这功能很简单,用来显示各种操作的信息.这个功能为全局可用,因此把功能写到`app.js`里
```
  //消息等级
  var msgTimer = null;
  var MSG_LEVEL = {
      info: 'info',
      warning: 'warning',
      debug: 'debug',
      error:'error'
  };
  //状态栏消息
  hmd.msg = function (txt, lv) {
    lv = lv || MSG_LEVEL.info;
    $('#msg')
    .removeClass(MSG_LEVEL.info)
    .removeClass(MSG_LEVEL.warning)
    .removeClass(MSG_LEVEL.debug)
    .removeClass(MSG_LEVEL.error)
    .addClass(lv).text(txt);
    clearTimeout(msgTimer);
    msgTimer = setTimeout(function () {
      $('#msg')
      .removeClass(MSG_LEVEL.info)
      .removeClass(MSG_LEVEL.warning)
      .removeClass(MSG_LEVEL.debug)
      .removeClass(MSG_LEVEL.error);
    }, 5000);
  };
```
eg: 文件保存成功时显示
```
   studio.directive('hmdEditor', function () {
    return function ($scope, elem) {
      hmd.editor.init({el:elem[0]},'E:\\Temp\\test\\test.md');
      hmd.editor.on('saved',function(filepath){
        var fileNameArr = filepath.split('\\');
        hmd.msg('文件:' + fileNameArr[fileNameArr.length - 1] + '保存成功!');
      });
    };
  });
```
有一点要注意的事,editor都是尽量采用事件的方式来对外提供接口,这样可以让editor与外部的耦合度降低.

##文件操作
工具条的bt-group里有三个按钮,分别绑定了三个studio下的directive:`studio-newfile`,`studio-openfile`,`studio-save`.
现在打开`modules/studio/directives.js`文件,开始实现这3个功能.

### 新建文件
这个功能很简单,只要把当前文件设为空,并且清空编辑器内容就算是新建文件了,保存的时候才会让用户选择保存路径.

修改`editor.js`的`setFile`方法
```
    //设置当前文件
    setFile:function(filepath){
      if(filepath && fs.existsSync(filepath)){
        var txt = util.readFileSync(filepath);
        this.filepath = filepath;
        this.cm.setValue(txt);
      }
      else{
        this.filepath = null;
        this.cm.setValue('');
      }
    }
```

实现`directive`,点击按钮时调用编辑器的`setFile`让`filepath`为空
```
  studio.directive('studioNewfile', function () {
    return function ($scope, elem) {
      $(elem[0]).on('click',function(){
        hmd.editor.setFile();
      });
    };
  });
```
这样新建文件按钮就完成了,其实这按钮的功能就是清空编辑器,真正的保存新文件功能在保存按钮功能里实现.

### 保存文件

```
<a studio-save ng-class="{'disabled':!editorChanged}" href="javascript://" class="btn btn-primary" title="保存更改(Ctrl+S)" style="border-radius:0;">省略..</a>
```
保存按钮只有在文本有改动时才可用,这样用户就能很直观的看到是否已保存(禁用按钮时,按ctrl+s依然可以保存,很多人都习惯一直按ctrl+s)
通过ng-class来实现这个功能,将class`disabled`绑定到editorChanged这个上下文变量上`ng-class="{'disabled':!editorChanged}"`.
```
  studio.directive('studioSave',function(){
    return function($scope,elem){
      var editor = hmd.editor;
      //标识是否有未保存的变更.
      $scope.editorChanged = false;
      editor.on('change', function (cm, change) {
        $scope.editorChanged = true;
        $scope.$digest();
      });
      editor.on('saved', function () {
        $scope.editorChanged = false;
        $scope.$digest();
      });
      
      $(elem[0]).on('click',function(){
        editor.save();
      });
    };
	});
```
这样$scope.editorChanged变化时,保存按钮也会跟着变化,我们不需要直接操作dom元素.


[editor.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/editor.js)里保存功能的实现

```
//保存文件
save : function () {
  var txt = this.cm.getValue();
  if(this.filepath){
    util.writeFileSync(this.filepath, txt);
    this.fire('saved',this.filepath);
  }
  else{
    this.saveAs();
  }
}
```

如果filepath不存在,那就调用saveAs方法来引导用户保存到新建的文件里.

```
    //另存为对话框
    saveAs:function(){
      var me = this;
      this.saveAsInput = $('<input style="display:none;" type="file"  accept=".md" nwsaveas/>');
      this.saveAsInput[0].addEventListener("change", function (evt) {
        if(this.value){
          me.filepath = this.value;
          me.save();
        }
      }, false);
      this.saveAsInput.trigger('click');

      hmd.msg('保存新文件');
    },
```
`nw.js`的[文件对话框](https://github.com/nwjs/nw.js/wiki/File-dialogs)比较特殊,可以通过代码触发单击事件来打开对话框,并没有一定要用户点击的限制,作为一个客户端开发框架,这样的改动是必要的.
因此我们上面的代码里直接创建了一个`input`标签,随即触发它的单击事件.其中`nwsaveas`是指定对话框的类型为另存为对话框.
用户输入或者选择好文件后,将filepath设置为用户指定的,并调用me.save()保存文件.
### 打开文件
将实现常用的三种打开文件方式:

1. 通过打开文件按钮
2. 拖动文件到编辑器
3. 双击md文件打开

**通过按钮打开**

通过按钮打开与另存为类似,直接上代码,不用多做解释.

```
    openFile:function(){
      var me = this;
      this.openFileInput = $('<input style="display:none;" type="file"  accept=".md"/>');
      this.openFileInput[0].addEventListener("change", function (evt) {
        if(this.value){
          me.setFile(this.value);
        }
      }, false);
      this.openFileInput.trigger('click');
    },
```
然后是实现按钮绑定的`directive`
```
<a studio-openfile href="javascript://" class="btn btn-primary" title="打开文件" ng-click="openTerminal()">省略..</a>
```
```
  studio.directive('studioOpenfile', function () {
    return function ($scope, elem) {
      $(elem[0]).on('click',function(){
        hmd.editor.openFile();
      });
    };
  });
```

**拖动打开**
到此三个按钮的功能都已实现,但是拖动打开文件是windows上程序的基本功能,因此我们也来实现它.

这个功能的实现放在编辑器的初始化代码后面,因为要编辑器初始化之后才能打开文件.
```
  studio.directive('hmdEditor', function () {
    return function ($scope, elem) {
      hmd.editor.init({el:elem[0]});
      hmd.editor.on('saved',function(filepath){
        var fileNameArr = filepath.split('\\');
        hmd.msg('文件:' + fileNameArr[fileNameArr.length - 1] + '保存成功!');
      });
      //监听拖动事件
      document.ondrop = function (e) {
        var path, $target = $(e.target), dir, system;
        e.preventDefault();
        if (!e.dataTransfer.files.length) return;
        //取到文件路径
        path = e.dataTransfer.files[0].path;
        //非目录,并且包含.md才会在编辑器里打开
        if (!fs.statSync(path).isDirectory() && ~path.indexOf('.md')) {
          hmd.editor.setFile(path);
        }
      };
    };
  });
```
添加了`document.ondrop`这一段代码,如果拖动的是一个`md`文件,则打开它

**双击md文件打开**

编写代码之前,我们选随便选中一个`md`文件,并设置默认用我们的程序打开.
![](http://oneaboveall.qiniudn.com/hx78bbaw2d3d64scor5lml4hd7.png)
然后关闭我们的程序,双击随便一个`md`文件试试,可以看到双击后我们的程序会启动,但是并不会打开双击的文件,接下来就写代码实现它.

我们把功能实现也放到editor的init之后.
```
  studio.directive('hmdEditor', function () {
    return function ($scope, elem) {
      hmd.editor.init({el:elem[0]});
      //省略部分代码...
      //双击md文件打开
      var gui = require('nw.gui'),
          filepath = gui.App.argv[0];
			~filepath.indexOf('.md') && hmd.editor.setFile(filepath);
      //如果程序已经打开,则会触发open事件
      gui.App.on('open', function(cmdline) {
      	window.focus();
        filepath = cmdline.split(' ')[2].replace(/\"/g,'');
        ~filepath.indexOf('.md') && hmd.editor.setFile(filepath);
      });
    };
  });
```
[相关知识](https://github.com/nwjs/nw.js/wiki/Handling-files-and-arguments)
关掉程序,再次双击md文件,就可以看到打开功能正常了.在软件已启动的状态双击文件也可以打开该文件.
##总结
今天实现了文件操作功能,这样我们的编辑器已经可用了.明天将实现`系统设置`的功能.

##附件
[本篇程序打包](http://pan.baidu.com/s/1sjJn3B3)
[项目地址](https://github.com/benqy/hexomd)