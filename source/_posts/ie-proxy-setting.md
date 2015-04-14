title: "使用程序修改系统(IE)代理设置"
date: 2015-04-13 17:41:01
categories: 工具
tags:
- 代理
- Gungnir
---

这是本人在做的一个前端开发调试工具([HttpMock](https://github.com/benqy/Gungnir)),功能是web服务器+http日记+http代理(类似fiddler),其中的代理功能,需要在web服务启动时,自动去设置各浏览器的代理设置.

我原先是通过写注册表的方式去实现,实现方法很简单,写一个注册表文件,启动的时候自动运行一下这个注册表文件就可以.  

修改代理的注册表文件的内容,**proxy.reg**:   
```    
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings]  
"MigrateProxy"=dword:00000001  
"ProxyEnable"=dword:00000001  
"ProxyServer"="http=127.0.0.1:17173"
```

类似的,取消代理的注册表文件,**disproxy.reg**:
```
	[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings]
	"MigrateProxy"=dword:00000001
	"ProxyEnable"=dword:00000000
	"ProxyServer"=""
```
这个方式非常的简单,但是存在一个严重的问题,就是修改完注册表之后,如果不重启ie,代理设置不会马上生效(这样做出来的软件,实在太山寨,简直没法用啊,什么烂软件). 

后来,发现fiddler是用**wininet**这个工具来设置ie代理.因此,google了一番之后,找到了wininet.dll这个东西,然后用.NET写了个简单的命令行工具.功能很简单,直接上代码:  

**ProxyManager.cs**
```
	using System;
	using System.Collections.Generic;
	using System.Runtime.InteropServices;
	using Microsoft.Win32;
	
	namespace proxysetting
	{
	    public class ProxyManager
	    {
	        [DllImport("wininet.dll", SetLastError = true)]
	        private static extern bool InternetSetOption(IntPtr hInternet, int dwOption, IntPtr lPBuffer, int lpdwBufferLength);
	        private const int INTERNET_OPTION_REFRESH = 0x000025;
	        private const int INTERNET_OPTION_SETTINGS_CHANGED = 0x000027;
	        private const string regeditKey = "Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings";
	        private List<string> proxyLibs = new List<string>();
	
	        private void Reflush()
	        {
	            InternetSetOption(IntPtr.Zero, INTERNET_OPTION_SETTINGS_CHANGED, IntPtr.Zero, 0);
	            InternetSetOption(IntPtr.Zero, INTERNET_OPTION_REFRESH, IntPtr.Zero, 0);
	        }
	
	        public void Add(string server)
	        {
	            this.proxyLibs.Add(server);
	        }
	
	        public void Run()
	        {
	            RegistryKey key = Registry.CurrentUser.OpenSubKey(regeditKey, true);
	            key.SetValue("ProxyServer", String.Join(";", this.proxyLibs.ToArray()));
	            key.SetValue("ProxyEnable", 1);
	            key.Close();
	            this.Reflush();
	        }
	
	        public void Stop()
	        {
	            RegistryKey key = Registry.CurrentUser.OpenSubKey(regeditKey, true);
	            key.SetValue("ProxyEnable", 0);
	            key.Close();
	            this.Reflush();
	        }
	    }
	}
```
ProxyManager类简单的封装wininet.dll的代理设置接口,和使用注册表文件的区别在于**this.Reflush()**这个方法,强制刷新代理设置,这样就不用重启IE了.  

然后是在Main函数里调用:  

**Program.cs**
```
	namespace proxysetting
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            
	            var pm = new ProxyManager();
	            if (args.Length < 1) return;
	            if (args[0] == "stop")
	            {
	                pm.Stop();
	            }
	            else
	            {
	                for (var i = 0; i < args.Length; i++)
	                {
	                    pm.Add(args[i]);
	                }
	                pm.Run();
	            }
	        }
	    }
	}
```
通过命令行参数来设置代理,可以用空格来设置多个协议的代理,如下:

设置http和https代理,代理ip为127.0.0.1,端口号17173  
**proxysetting http=127.0.0.1:17173 https=127.0.0.1:17173**  
nodejs中调用:

```
exports.setProxy = function () {
var exec = require("child_process").exec;
exec(__dirname + 'proxysetting http=127.0.0.1:17173 https=127.0.0.1:17173');
};
``` 


取消代理   
**proxysetting stop**  
nodejs中调用:

```	
  exports.disProxy = function () {
  var exec = require("child_process").exec;
  exec(__dirname + 'proxysetting stop');
  };
``` 

打包好的exe文件:[proxysetting](http://pan.baidu.com/s/1pJjuNG3)