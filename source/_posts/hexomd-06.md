title: "自己动手开发更好用的markdown编辑器-06(自动更新)"
date: 2015-05-12 15:10:09
tags:
- nw.js
- Hexo MD
---

[上一篇](http://benq.im/2015/04/28/hexomd-05/)我们实现了`粘贴上传图片`功能.

今天将实现`自动更新`的功能,有了这个功能以后我再发博客就不需要每次都把最新的程序重新打包上传了.

对于不想看如何实现的朋友,直接下载[打包好的程序](http://pan.baidu.com/s/1gdtkr6r)就行,以后更新可以点击软件右上角的第一个按钮即可(手动).
![](http://7ximoo.com1.z0.glb.clouddn.com/bkkjy41nrprmge49xbte3t29y6.png)

## 自动更新方案
在做上一个软件[Gungnir](https://github.com/benqy/Gungnir)的时候,为了可以显示更新进度,自动更新的方案是列出所有需要更新的文件,然后自动下载每个文件并覆盖,但是在需要更新一些node模块(文件一般都很多)时就相当麻烦了,有一个文件传输失败就会导致更新出错.实现起来相当麻烦,而且也并不能带来什么优势.

所以做这个软件的自动更新的时候,我用了更为简单粗暴的方案:将需要更新的文件打包成zip文件,直接下载并解压覆盖即可.

## 实现
自动更新作为较单独的功能模块,我把全部代码放在[modules/updater.js](https://github.com/benqy/hexomd/blob/master/app/modules/updater.js),这里就不把全部代码贴出来了,需要的自己点链接看,里面有注释. 我只讲一些实现细节.

### 安装依赖模块

首先是安装两个新增了的node模块依赖`when`和`bufferhelper`,第一个是`promise`模块,第二个看名字就知道,无须解释.
```
npm install when --save
```
```
npm install bufferhelper --save
```

将`7z.exe`放到软件根目录备用
![](http://7ximoo.com1.z0.glb.clouddn.com/4a8nq20f7v1mjqjko7y8qb48ff.png)

### 自定义[package.json](https://raw.githubusercontent.com/benqy/hexomd/master/package.json)

增加了updater配置节点,配置最新的版本号`version`和对应的补丁文件地址`package`,由于我这个软件功能很少,代码并不多,因此我现在每次更新都是包含之前所有补丁的文件打包,加起来也才1m多,这样实现比较简单,只要下载最新的包即可.
```
...
  "updater":{
    "version":"0.6.0.1",
    "package":"http://7xit5q.com1.z0.glb.clouddn.com/update.zip"
  }
```

### updater.js
`updater.js`里实现了`hmd.updater`模块,包含4个方法

```
  //配置文件,用于判断是否有新版本
  var packageFile = 'https://raw.githubusercontent.com/benqy/hexomd/master/package.json',
  //当前程序的运行目录
  execPath = require('path').dirname(process.execPath),
  //补丁文件存放目录
  updatePath = execPath + '\\update',
  fs = require('fs'),
  util = require('./helpers/util'),
  when = require('./node_modules/when');
  var checkUpdateTimer;
  hmd.updater = {
    //下载指定url的内容并返回promise对象
    get: function (url) {
      ...
    },
    //检查更新
    checkUpdate: function () {
    	...
    },
    //下载补丁包
    update:function(packageUrl){
      ...
    },
    //安装补丁包
    install: function () {
     ...
    }
  };
```

更新的流程为 : `checkUpdate`检查是否有更新 > `update`下载补丁包 > `install`安装补丁包

`get`方法里要注意的是下载下来的内容要判断是否经过`GZIP`压缩,如果是,则要用node自带的`zlib`解压.
```
      ...
      req = protocolModule.get(urlOpt, function (res) {
        //是否经过gzip压缩
        var isGzip = !!res.headers['content-encoding'] && !!~res.headers['content-encoding'].indexOf('gzip');
       ...
          if (isGzip) {
            require('zlib').unzip(buffer, function (err, buffer) {
              gzipDeferred.resolve(buffer);
            });
          }
          else {
            gzipDeferred.resolve(buffer);
          }
      ...
      return deferred.promise;
```

`checkUpdate`方法里先下载线上的`package.json`文件与本地进行比较,如果版本号不一致,则提示用户更新.如果用户选择更新,则下载package.json到本地,然后调用`update`方法下载补丁

```
    checkUpdate: function () {
      hmd.msg('===正在检查更新===');
      //超时检查
      checkUpdateTimer =setTimeout(function(){
        hmd.msg('===更新失败,可能github被墙了===', hmd.MSG_LEVEL.error);
      }, 10000);
      var locPackage = require('nw.gui').App.manifest;
      //获取版本信息和更新文件列表
      hmd.updater.get(packageFile)
      .then(function (packageData) {
        clearTimeout(checkUpdateTimer);
        packageData.text = packageData.buffer.toString();
        if (!packageData.text) return;
        var remotePackage = JSON.parse(packageData.text);
        if (remotePackage.updater.version != locPackage.updater.version){
          if (confirm('是否更新到最新版本:' + remotePackage.updater.version)) {
            //如果update目录不存在则创建
            if (!fs.existsSync(updatePath)) {
              util.mkdir(updatePath, true);
            }
            //保存最新的配置文件
            fs.writeFileSync(updatePath + '\\package.json', packageData.buffer);
            //下载补丁包
          	hmd.updater.update(remotePackage.updater.package);
          }
        }
        else {
          hmd.msg('当前版本:' + remotePackage.updater.version + ',已经是最新版');
        }
      });
    }
```

`update`方法下载补丁包到`update`目录,然后调用`install`安装补丁
```
    update:function(packageUrl){
      hmd.msg('===正在下载更新文件===', hmd.MSG_LEVEL.warnings);
      hmd.updater.get(packageUrl + '?' + new Date() * 1)
      .then(function (data) {
        fs.writeFileSync(updatePath + '\\update.zip',data.buffer);
        hmd.updater.install();
      });
    },
```

`install`将补丁包通过`7z.exe`解压覆盖到程序目录,然后提示用户重启软件.
```
    install: function () {
      //移动配置文件
      require("child_process").exec('xcopy "' + updatePath + '\\package.json" "' + execPath + '\\package.json" /s /e /y');
      //解压缩补丁文件
      var unzip = execPath + '\\7z.exe x '+ updatePath +'\\update.zip -y';
      require("child_process").exec(unzip,function(){
      hmd.msg('===更新完成,重启后生效===');
      });
    }
```
这里我直接用7z.exe,反正也不大,也可以使用一些开源的node压缩模块.

### 绑定更新按钮

更新模块完成了,现在将功能绑定到按钮上.
先在[modules/directives](https://github.com/benqy/hexomd/blob/master/app/modules/directives.js)增加新的directive`hmdUpdate`
```
  angular.module('hmd.directives', [])
  .directive('hmdUpdate', [function () {
    return function (scope, elem) {
      $(elem[0]).on('click', function () {
        hmd.updater.checkUpdate();
      });
    };
  }])
```

然后将[index.html](https://github.com/benqy/hexomd/blob/master/app/index.html)上的更新按钮与directive绑定
```
<a class="btn rectbtn" href="javascript://" title="点击检查更新" hmd-update>...</a>
```

别忘了引用`updater.js`

```
  <script src="modules/app.js"></script>
  <script src="modules/updater.js"></script>
  <script src="modules/directives.js"></script>
```

## 总结
基本的自动更新的功能比图片上传更为简单,但是今天做的这个功能还有很多细节问题,比如:

1. 无法自动删除新版本不需要的文件
2. 以后如果程序大了,更新补丁每次都全量打包会导致更新很慢
3. 更新后不会自动重启软件.
4. 更好的方案是自动根据git的提交信息生成更新列表,并且根据版本号管理.

![](http://7ximoo.com1.z0.glb.clouddn.com/fv6a4ol9mtjractcur4ba7drev.png)

接下来的计划:

1. 云同步.
2. 插件机制
3. 表情插件.

## 附件
[本篇程序打包](http://pan.baidu.com/s/1gdtkr6r).
[项目地址](https://github.com/benqy/hexomd)