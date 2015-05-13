title: "使用hexo在github上搭建个人博客"
date: 2015-04-13 18:35:31
categories: 其他
tags:
- github
- hexo
---


此教程适合我这种虽然在用github,却不懂git命令的文盲.

### 建立博客源码仓库
首先在github上创建一个空的仓库用来存放博客程序.
![](http://oneaboveall.qiniudn.com/cfw3ouietza4t3gvs6zhl0a32q.png)

安装github windows客户端https://windows.github.com/. 注意github客户端是在线安装,如果安装不成功,就使用代理试试.
安装完成github客户端后,打开客户端并登录,把刚才创建的项目clone到本地.

![](http://oneaboveall.qiniudn.com/gabnv1oin9e01xtzndsg8rys98.png)

![](http://oneaboveall.qiniudn.com/siea9xj50icnmbsdq8u72m7dp9.png)

### 安装hexo

```
npm install -g hexo
```
hexo安装完成后,打开命令行,进入刚才的github仓库目录的上一级,初始化hexo项目.
```
hexo init <目录名>
```
![](http://oneaboveall.qiniudn.com/7fnwja7g53y83s8yye5jy12d42.png)

进入仓库目录,安装依赖模块
```
npm install
```
大局域网安装起来可能会有点慢,耐心等待..

安装完成后,运行hexo服务端
```
hexo server
```
打开http://127.0.0.1:4000.
![](http://oneaboveall.qiniudn.com/65033gqkxlz69mxb0z8yoa28ua.png)
安装成功

常用命令:
```
hexo clean //清理
hexo new filename //创建新文章
hexo generate //生成静态站点(位于public目录)
hexo deploy //发布,后面会讲
```
更多hexo命令,可以查看官方文档http://hexo.io/docs/


### 安装hexo主题
hexo主题有点少. 目前我在用jacman这个主题,这个比较适合国人用,集成了多说评论.

在仓库目录里运行命令
```
git clone https://github.com/wuchong/jacman.git themes/jacman
```
将主题下载到themes/jacman目录.
打开仓库根目录下的配置文件`_config.yml` ,修改theme为 `theme: jacman`


重启服务器(hexo server)即可看到新样式
jacman主题的详细介绍 http://wuchong.me/jacman/2014/11/20/how-to-use-jacman/

### 提交仓库
删除`theme`目录下的`landscape`目录,这个主题我们不用了.
删除`theme/jacman`目录下.git目录和.gitignore文件.
切换到github for windows客户端,提交仓库并同步到线上
![](http://oneaboveall.qiniudn.com/ry1v1bsfgeewm8fnozxo4w7ory.png)

### 建立gh-pages分支

用网页打开仓库地址https://github.com/benqy/hello-benqy
点击Settings
![](http://oneaboveall.qiniudn.com/majdcf5zu3hrrtjj3u9lk3ifz8.png)
然后
![](http://oneaboveall.qiniudn.com/wnj9c3c5g10xu7w7iq90pynjt6.png)
再然后
![](http://oneaboveall.qiniudn.com/xvo1g5nw18c4042228nbpkmmib.png)
最后
![](http://oneaboveall.qiniudn.com/znh4em8b089kvlvak7dw6v5597.png)
这样gh-pages分支就创建完成了.可以打开http://benqy.github.io/hello-benqy 看看效果

切换到该分支

![](http://oneaboveall.qiniudn.com/wwtkxifls0d7k6uaebrod3m2yy.png)

复制分支的clone URL https://github.com/benqy/hello-benqy.git

继续打开根目录的`_config.yml`,将deploy改为

```
deploy:
  type: git
  repository: https://github.com/benqy/hello-benqy.git
  branch: gh-pages
```
在根目录运行命令
```
npm install hexo-deployer-git --save
```
接下来运行生成静态站点并发布的命令
```
hexo deploy --generate
```

过程中会需要输入github帐号密码
发布成功:
![](http://oneaboveall.qiniudn.com/jnb4pxrpbfenr4i326igl7ukjf.png)
打开博客地址:http://benqy.github.io/hello-benqy 会发现页面乱了,因为还没配置博客路径
依然是打开配置文件`config.yml`,根据注释修改URL配置
```
url: http://yoursite.com
root: /
```
改为
```
url: http://benqy.github.io/hello-benqy
root: /hello-benqy
```
重新发布
```
hexo clean
hexo deploy --generate
```
再次打开博客,一切都正常了

### 发布文章
至此,博客的搭建完成了.
以后要发文章,只要在博客目录运行
```
hexo new 文件名
```
就会在`source/_posts`下生成对应的`.md`文件.
运行本地服务器
```
hexo server
```
通过markdown格式编写文章,并打开本地地址127.0.0.1:4000查看实时效果

文章写完后
```
hexo deploy --generate
```
提交即可
记得主仓库也用github for windows提交到github上
### 配置cname
...

### 进阶:图片自动上传
这是我做的[markdown编辑器](http://benq.im/2015/04/28/hexomd-05/)
利用七牛免费的存储和方便的接口来让我们的markdown编辑器在粘贴图片时自动上传到七牛云存储,并返回图片地址.
我博客里的所有图片都是这样上传的,写文章时完全不用停下来传图.