title: "javascript与as3交互"
date: 2015-04-13 18:08:50
categories: 前端
tags:
- javascript
- as3
---



由于公司的广告业务需要涉及到flash,因此稍微学习了一下,把一些重要的记录下来

这篇主要讲讲js和as3之间如何交互,如何实现简单的类似事件的机制.


### 模拟事件
在as3中,无论是调用js的方法,还是暴露接口给js,都是用ExternalInterface这个api.  
**暴露接口**:ExternalInterface.addCallback(函数名, 函数)  
**调用js方法**:ExternalInterface.call(方法名, 参数)

这里所说的模拟事件,就是在falsh素材的各个状态切换时,通知页面.如加载完毕时,触发onload事件,播放时触发onplay,停止时触发onstop等等.

采用的方案,是在object标签里,添加自定义的属性供as3调用,属性名是flash定死的:flashvars,值的格式类似url里的参数部分,为用&分隔的键值对,如下:

![01](https://raw.github.com/benqy/blogcdn/master/as3andjs/01.png)

在as3中,这样读取属性:

```
	flashvars= LoaderInfo( this.root.loaderInfo ).parameters;
```

flashvars是一个object变量,定义如下:

```
	private var flashvars:Object;
```

读取后,flashvars的值:

```
	Object {onload: "advnr70wyu5xncanvpwzzo0pory66", onplay: "advf4q8r99cfnryhn2tk2n3s48kf0", onstop: "adv7sgjk0529cd00q6ojuf537"}
```

对象的key为事件名,值为事件处理函数名,所有事件处理函数都在window.flashCallback命名空间下,并且名字是随机生成的不可重复的guid.

在as3里,我们判断是否支持调用js函数:

```
	ExternalInterface.available
```

以onload事件为例,在素材加载完毕后,如果支持调用js函数,则触发onload事件:

```
	//素材加载完毕,调用onload
	private function onload():void { 
		if (ExternalInterface.available) {
			textField.text = 'onload';
			if(flashvars.onload){
				ExternalInterface.call("window.flashCallback." + flashvars.onload);
			}
		} 
	} 
```

上面提到,函数名是随机生成的,类似jquery的$.ajax方法里的callback实现方式,以下是封装过的生成object标签的方法:

```
	//adv.flash.embed是封装过的生成object标签的方法
	asyncTest('flash标签自定义事件', function () {
	  expect(3);
	  document.body.appendChild(adv.flash.embed({
	    id: 'flashGetSWF',
	    source: 'test.swf',
	    params: {
	      onload: function () {
	        this.play();
	        ok(true);
	      },
	      onplay: function () {
	        ok(true);
	        this.stop();
	      },
	      onstop: function () {
	        ok(true);
	        start();
	      }
	    }
	  }));
	});
```

几个回调函数都是匿名函数,在生成标签之前,会给函数在window.flashCallback下分配一个随机名称.现在只是简单的实现单个事件处理函数的绑定,如果有需要支持多个,只要再做一些简单的封装即可.

**adv.flash.embed的实现要点**:

```
	//遍历自定义参数
	adv.util.forEach(params, function (value, key) {
	  var funcVar = value;
	  if (adv.util.isFunction(value)) {
	    //如果是函数,则生成一个随机的名称,并存储在flashCallback命名空间下
	    funcVar = adv.util.guid();
	    window.flashCallback[funcVar] = function() {
	      var swfObj = adv.flash.getSWF(id);
	      //this指向flash对象本身
	      value.apply(swfObj, arguments);
	    };
	  }
	  paramArr.push(key + '=' + funcVar);
	});
```

**执行结果**:  
![02](https://raw.github.com/benqy/blogcdn/master/as3andjs/02.png)


### 暴露接口给js

as3里,实现这个很简单,先上代码再说:

```
	public function test() { 
		flashvars= LoaderInfo( this.root.loaderInfo ).parameters;
		//此为调试用
		ExternalInterface.call("window.flashCallback.trace",flashvars);
		//加载素材
		textField.width=200;
		textField.height = 90;
		addChild(textField);
		//素材加载完毕,准备播放
		//暴露api给js
		if(ExternalInterface.available){
			ExternalInterface.addCallback("play", play);
			ExternalInterface.addCallback("stop", stop);
		}
		//触发onload
		onload();
	}
	
	//这个方法被调用时,开始播放
	private function play():Boolean {
		textField.text = 'play';
		if(flashvars.onplay){
			ExternalInterface.call("window.flashCallback." + flashvars.onplay);
		}
		return true;
	}
			
	//这个方法被调用时,停止播放
	private function stop():Boolean {
		textField.text = 'stop';
		if(flashvars.onstop){
			ExternalInterface.call("window.flashCallback." + flashvars.onstop,flashvars);
		}
		return true;
	} 
```

这里定义了两个方法,play和stop,在构造函数(test)里,判断ExternalInterface是否可用,如果可用,就暴露出这两个方法:

```
  ExternalInterface.addCallback("play", play);
  ExternalInterface.addCallback("stop", stop);
```

**在js里调用**:  

跨浏览器的获取flash对象引用方法

```
	...
	getSWF:function(name) {
	  if (navigator.appName.indexOf("Microsoft") != -1) {
	    return window[name];
	  } else {
	    return document[name];
	  }
	}
	...
```

暴露出的as3方法,会直接附加在object对象上,要注意的是,我们要等onload之后才去调用,保证falsh加载完毕:

```
	asyncTest('js调用flash方法', function () {
	  expect(1);
	  document.body.appendChild(adv.flash.embed({
	    id: 'flashcallSWF',
	    source: 'test.swf',
	    width: 300,
	    height: 190,
	    params: {
	      onload: function () {
	        var result = this.play();
	        ok(result);
	        start();
	      }
	    }
	  }));
	});
```

事件函数的this指向flash对象,所以这里不用再获取了


**执行结果**:  
![03](https://raw.github.com/benqy/blogcdn/master/as3andjs/03.png)


完整的as3代码:

```
	package { 
		import flash.display.LoaderInfo;
		import flash.display.Sprite;
		import flash.external.ExternalInterface;
		import flash.text.TextField;
		
		public class test extends Sprite {
			private var flashvars:Object;
			//用textField文本的改变来代表广告播放的状态转变
			private var textField:TextField = new TextField(); 
			public function test() { 
				flashvars= LoaderInfo( this.root.loaderInfo ).parameters;
				//加载素材
				textField.width=200;
				textField.height = 90;
				addChild(textField);
				//素材加载完毕,准备播放
				//暴露api给js
				if(ExternalInterface.available){
					ExternalInterface.addCallback("play", play);
					ExternalInterface.addCallback("stop", stop);
				}
				//触发onload
				onload();
			}
			
			//这个方法被调用时,开始播放
			private function play():Boolean {
				textField.text = 'play';
				if(flashvars.onplay){
					ExternalInterface.call("window.flashCallback." + flashvars.onplay);
				}
				return true;
			}
					
			//这个方法被调用时,停止播放
			private function stop():Boolean {
				textField.text = 'stop';
				if(flashvars.onstop){
					ExternalInterface.call("window.flashCallback." + flashvars.onstop,flashvars);
				}
				return true;
			} 
			
			private function onload():void { 
				if (ExternalInterface.available) {
					textField.text = 'onload';
					if(flashvars.onload){
						ExternalInterface.call("window.flashCallback." + flashvars.onload);
					}
				} 
			} 
		}
	}
```