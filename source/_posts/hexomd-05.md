title: "自己动手开发更好用的markdown编辑器-05(粘贴上传图片)"
date: 2015-04-28 15:13:55
tags:
- nw.js
- Hexo MD
---

[上一篇](http://benq.im/2015/04/25/hexomd-04/)我们实现了`实时预览`功能.

今天这篇要利用免费的七牛云存储服务来实现`粘贴自动上传图片`的功能.
不想看过程的朋友可以直接下载[打包好的程序](http://pan.baidu.com/s/1eQtSpwQ)使用,使用之前记得[配置七牛帐号](#配置七牛帐号).

文章的内容包含以下三点:

1. [七牛云存储](http://www.qiniu.com/).
2. [clipboard-apis](http://www.w3.org/TR/clipboard-apis/)
3. ajax文件上传


## 七牛云存储

### 系统设置
首先在系统设置里增加`七牛空间设置`部分,这里就简单的贴上代码,因为系统设置模块之前几篇了都讲过了.


[system/model.js](https://github.com/benqy/hexomd/blob/master/app/modules/system/model.js)
```
...
  //默认设置
  var defaultSystemData = {
    //最后一次打开的文件
    lastFile: null,
    //编辑器样式
    theme:'ambiance',
    //预览窗口样式
    preViewTheme:'github',
    //预览代码块样式
    preViewHighLightTheme:'default',
    
    /*七牛空间设置*/
    accessKey:'',
    secretKey:'',
    //空间名称
    bucketName:'test',
    //空间访问地址
    bucketHost:'7xit3a.com1.z0.glb.clouddn.com',
    //过期时间,从设置之后多少小时过期.
    deadline:1000
  };
```

[system.html]()
```
...
    <div class="form-group">
      <label>AccessKey</label>
      <input class="form-control" name="accessKey" ng-model="systemSetting.accessKey"/>
    </div>
    <div class="form-group">
      <label>SecretKey</label>
      <input class="form-control" name="secretKey" ng-model="systemSetting.secretKey"  type="text"/>
    </div>
    <div class="form-group">
      <label>空间名</label>
      <input class="form-control" name="bucketName" ng-model="systemSetting.bucketName" type="text"/>
    </div>
    <div class="form-group">
      <label>空间域名</label>
      <input class="form-control" name="bucketHost" ng-model="systemSetting.bucketHost" type="text"/>
    </div>
    <div class="form-group">
      <label>过期时间</label>
      <input name="deadline" class="form-control" ng-model="systemSetting.deadline" type="number"/>
    </div>
...
```
增加了`accesskey`,`secretkey`,`空间名`,`过期时间`四个用于上传图片的字段. 其中`accesskey`, `secretkey`用于验证权限,`空间名`用于指定上传图片的存储空间,`过期时间`指定授权的过期时间. 这四个字段共同组成上传授权的`Token`,生成的方法如下:

安装七牛nodejs版sdk
```
npm install qiniu --save
```
![](http://7ximoo.com1.z0.glb.clouddn.com/a2dsa5gcztr19hqvlorc9apuv7.png)

[system/model.js](https://github.com/benqy/hexomd/blob/master/app/modules/system/model.js)
```
...
  //生成七牛存储空间token
  system.qiniuKeygen = function(systemSetting){
    var qiniu = require('../app/node_modules/qiniu');
    qiniu.conf.ACCESS_KEY = systemSetting.accessKey;
    qiniu.conf.SECRET_KEY = systemSetting.secretKey;
    var putPolicy = new qiniu.rs.PutPolicy(systemSetting.bucketName);
    putPolicy.expires = Math.round(new Date().getTime() / 1000) + systemSetting.deadline * 3600;
    systemSetting.qiniutoken = putPolicy.token();
    return systemSetting;
  };
...
```

### 配置七牛帐号
首先得注册一个[七牛帐号](http://www.qiniu.com/).

进入后台,新建一个空间,我这里取的空间名为`blog`,用于我博客的图片存储.
![](http://7ximoo.com1.z0.glb.clouddn.com/z1gu22y4uqnf8iae1m7fygvptx.png)

选择新建的空间,点击`空间设置`>`域名设置`,查看自动分配的域名
![](http://7ximoo.com1.z0.glb.clouddn.com/pby78cgpeaa1c41w4n2d2v5vlh.png)

回到后台首页,点击`账号设置`,可以查看`accessKey(AK)`和`SecretKey(SK)`

![](http://7ximoo.com1.z0.glb.clouddn.com/q2eqjgnuof5ka3x6pygd2h23s1.png)

在刚做好的后台里配置好这几个字段
![](http://7ximoo.com1.z0.glb.clouddn.com/3iu3kwp04uyaxdnsbp8wo9xsns.png)
我把我空间的密钥遮住了,大家请填上自己的空间密钥

## 图片上传
图片的存储空间准备好了,现在来实现上传的功能.

初始化editor的时候传入七牛的token
[studio/directive.js](https://github.com/benqy/hexomd/blob/master/app/modules/studio/directives.js)
```
...
studio.directive('hmdEditor', function () {
    return function ($scope, elem) {
      var systemData = hmd.system.get();
      hmd.editor.init({
        el:elem[0],
        theme:systemData.theme,
        //七牛云存储授权
        qiniuToken:hmd.system.qiniuKeygen(systemData).qiniutoken,
        //空间的域名
        bucketHost:systemData.bucketHost
      },systemData.lastFile);
      //保存最后一次打开的文件
      hmd.editor.on('setFiled',function(filepath){
        hmd.system.setLastFile(filepath);
      });
...
```

[studio/editor](https://github.com/benqy/hexomd/blob/master/app/modules/studio/editor.js)里实现图片上传功能

```
    ...
    initQiniu:function(options){
      this.qiniuToken = options.qiniuToken;
      this.bucketHost = options.bucketHost;
      //监听粘贴事件
      $('.studio-wrap')[0].onpaste = this.uploadImage.bind(this);
    },
    uploadImage:function(ev){
      var clipboardData, items, item;
      if(!this.qiniuToken){
        this.fire('error',{msg:'未设置七牛密钥,无法上传图片'});
      }
      //如果粘贴的是图片
      else if (ev && (clipboardData = ev.clipboardData) && (items = clipboardData.items) &&
          (item = items[0]) && item.kind == 'file' && item.type.match(/^image\//i)) {
        //取图片数据
        var blob = item.getAsFile();
        //生成随机的图片名
        var fileName = this.guid() + '.' +  blob.type.split('/')[1];
        //上传
        this._qiniuUpload(blob, this.qiniuToken, fileName, function (blkRet) {
          //生成markdown格式的图片地址
          var img = '![](http://'+this.bucketHost+'/' + blkRet.key + ')';
          //在光标处插入图片
          this.cm.doc.replaceSelection(img);
        }.bind(this));
        return false;
      }
    },
    //上传图片,参数为:图片2进制内容,七牛token,文件名,回调函数
    _qiniuUpload:function (f, token, key,fn) {
      var xhr = new XMLHttpRequest();
      //创建表单
      xhr.open('POST', 'http://up.qiniu.com', true);
      var formData, startDate;
      formData = new FormData();
      if (key !== null && key !== undefined) formData.append('key', key);
      formData.append('token', token);
      formData.append('file', f);
      var taking;
	  
      xhr.onreadystatechange = function (response) {
        //上传成功则执行回调
        if (xhr.readyState == 4 && xhr.status == 200 && xhr.responseText) {
          var blkRet = JSON.parse(xhr.responseText);
          fn(blkRet);
        } else if (xhr.status != 200 && xhr.responseText) {
          if(xhr.status == 631){
            hmd.editor.fire('error',{msg:'七牛空间不存在.'});
          }
          else{
            hmd.editor.fire('error',{msg:'七牛设置错误.'});
          }
        }
      };
      startDate = new Date().getTime();
      //提交数据
      xhr.send(formData);
    }
    ...
```
至此这个功能就完成了,随便截张图然后在编辑器里粘贴,编辑器就会自动生成图片引用地址:
![](http://7ximoo.com1.z0.glb.clouddn.com/qvxnvawlbv37h2ebs5oxw62r6y.png)

这个功能较为简单,因此今天篇幅较小.


## 总结
粘贴上传图片的功能让我不用频繁的停下来上传图片,可以大大的提高用markdown写文章的效率.
目前功能还较为简单,不能指定图片名,不能分目录,大家可以根据自己的需求修改代码,完善功能.


接下来的计划:

1. 自动更新.
2. 云同步.
3. 插件机制
3. 表情插件.

## 附件
[本篇程序打包](http://pan.baidu.com/s/1eQtSpwQ),七牛云存储未设置好,请自行根据[教程](#配置七牛帐号)设置.
[项目地址](https://github.com/benqy/hexomd)