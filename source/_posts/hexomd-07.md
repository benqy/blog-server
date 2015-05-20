title: "自己动手开发更好用的markdown编辑器-07(扩展语法)"
date: 2015-05-19 18:23:13
tags:
- nw.js
- Hexo MD
---

[上一篇](http://benq.im/2015/05/12/hexomd-06/)我们实现了`自动更新`的功能.

在前面的6篇中,我们基本没做什么创造,都只是像玩乐高那样把零件拼接成我们想要的东西.
今天这篇将对`marked`进行简单扩展, 增加我们的markdown编辑器支持的语法,实现`目录`,`emojis`表情两种新语法.
以及改造`codemirror`,实现我们自定义语法的编辑器高亮显示(这个本来是要放到下一篇,但是刚刚做完后发现内容很短,所以就又合并到这篇里来了).

对于不想看如何实现的朋友,直接下载[v0.6.0.2](http://pan.baidu.com/s/1gdtkr6r),然后点击右上角的更新按钮更新到最新版即可.

## 准备工作
首先打开[marked](https://github.com/chjj/marked),Fork一份到自己仓库. 对marked的改造都将基于我们的这个fork版本.
![](http://7ximoo.com1.z0.glb.clouddn.com/rghke2f5nee45sk8gb5h2y2vuz.png)

## 目录语法
功能描述: 自动提取所有H标签,形成目录树,在解析markdown文本时,如果遇到`[TOC]`标签则自动将其替换为目录.

将我们fork的版本clone到本地,打开[lib/marked.js](https://github.com/benqy/marked/blob/master/lib/marked.js).所有代码都在这个文件里.

修改`inline.gfm`,增加目录语法匹配正则
```
/**
 * GFM Inline Grammar
 */

inline.gfm = merge({}, inline.normal, {
  escape: replace(inline.escape)('])', '~|])')(),
  url: /^(https?:\/\/[^\s<]+[^<.,:;"')\]\s])/,
  del: /^~~(?=\S)([\s\S]*?\S)~~/,
  toc: /\s*\[TOC\]/,
  text: replace(inline.text)
    (']|', '~]|')
    ('|', '|https?://|')
    ()
});
```

修改`Renderer`,增加`toc`和`tocItem`两个方法,用于生成目录标签:
```
Renderer.prototype.toc = function (items) {
  var html = '<div id="toc" class="toc"><ul class="toc-tree">';
  html += items;
  html += '</ul></div>';
  return html;
}

Renderer.prototype.tocItem = function (id, level, text) {
  return '<li class="toc-item toc-level-' + level + '"><a class="toc-link" href="#' + id + '"><span class="toc-number"></span> <span class="toc-text">' + text + '</span></a></li>';
};
```

修改`Renderer`的`heading`方法,为其赋予id作为点击目录项的锚点
```
Renderer.prototype.heading = function(text, level, raw) {
  var escapedText = text.toLowerCase();
  return '<h' + level + '><a name="' +
                escapedText +
                 '" class="anchor" href="#' +
                 escapedText +
                 '"><span class="header-link"></span></a>' +
                  text + '</h' + level + '>';
};
```

修改 `Parser.prototype.parse`,在解析时预生成好目录标签备用:
```
/**
 * Parse Loop
 */

Parser.prototype.parse = function (src) {
  var me = this, tocItems = '';
  //预生成好目录标签
  src.forEach(function (token) {
    if (token.type == 'heading') {
      id = token.text.toLowerCase();
      tocItems += me.renderer.tocItem(id, token.depth, token.text);
    }
  });
  this.inline = new InlineLexer(src.links, this.options, this.renderer);
  this.inline.tocHTML = me.renderer.toc(tocItems);

  this.tokens = src.reverse();
  
  var out = '';
  while (this.next()) {
    out += this.tok();
  }

  return out;
};
```

最后是修改`InlineLexer`,在匹配到`[TOC]`时将其替换为完整的目录标签
```
/**
 * Lexing/Compiling
 */

InlineLexer.prototype.output = function(src) {
  var out = ''
    , link
    , text
    , href
    , cap;
  while (src) {
    //toc语法
    if (cap = this.rules.toc.exec(src)) {
      src = src.substring(cap[0].length);
      out += this.tocHTML;
      continue;
    }
    ...
}
```

这样目录语法就完成了,没几行代码,效果如图(预览的样式比较丑,这系列的某一篇会专门优化预览样式):
![](http://7ximoo.com1.z0.glb.clouddn.com/5m56sa81v6wm6cee4ioyd2rgxe.png)


## emojis表情语法

### 准备表情素材
我将要实现的`emoji`表情库基于[http://www.emoji-cheat-sheet.com/](http://www.emoji-cheat-sheet.com/)这个项目,大家可以通过这个页面查看所有表情的命名.
我将这里所有表情上传一份到我的七牛空间里,这样访问会快一些.

### 实现功能

`emojis`语法的实现跟目录类似.

修改`inline.gfm`,增加emojis语法匹配正则
```
/**
 * GFM Inline Grammar
 */

inline.gfm = merge({}, inline.normal, {
  escape: replace(inline.escape)('])', '~|])')(),
  url: /^(https?:\/\/[^\s<]+[^<.,:;"')\]\s])/,
  del: /^~~(?=\S)([\s\S]*?\S)~~/,
  emoji: /^:([A-Za-z0-9_\-\+]+?):/,
  toc: /\s*\[TOC\]/,
  text: replace(inline.text)
    (']|', ':~]|')
    ('|', '|https?://|')
    ()
});
```
为`Renderer`增加`emoji`方法:
```
Renderer.prototype.emoji = function (emoji) {
  //图片使用自己的七牛空间
  return '<img src="'
  + 'http://7xj6bw.com1.z0.glb.clouddn.com/'
  + encodeURIComponent(emoji)
  + '.png"'
  + ' alt=":'
  + escape(emoji)
  + ':"'
  + ' title=":'
  + escape(emoji)
  + ':"'
  + ' class="emoji" align="absmiddle" height="20" width="20">';
};
```
最后,在`InlineLexer`里:
```
/**
 * Lexing/Compiling
 */

InlineLexer.prototype.output = function(src) {
  var out = ''
    , link
    , text
    , href
    , cap;
  while (src) {
    ...    
    // emoji (gfm)
    if (cap = this.rules.emoji.exec(src)) {
      src = src.substring(cap[0].length);
      out += this.renderer.emoji(cap[1]);
      continue;
    }
    ...
}
```
完成!这个功能比目录功能更加简单

![](http://7ximoo.com1.z0.glb.clouddn.com/8hds06f22xz5bfofwsv8j6k7ta.png)

## 编辑器语法高亮

这里就不再去fork [codemirror](https://github.com/codemirror/CodeMirror)这个项目了,有兴趣的可以去fork,修改完后提交给官方.
我们直接简单粗暴的修改[lib/codemirror/mode/markdown/markdown.js](https://github.com/benqy/hexomd/blob/master/app/lib/codemirror/mode/markdown/markdown.js).

增加toc和emoji的正则:
```javascript
  ...
  ,    toc = 'toc'
  ,    emoji = 'emoji'
  ..
  ..
  var hrRE = /^([*\-=_])(?:\s*\1){2,}\s*$/
  ,   ulRE = /^[*\-+]\s+/
  ,   olRE = /^[0-9]+\.\s+/
  ,   taskListRE = /^\[(x| )\](?=\s)/ // Must follow ulRE or olRE
  ,   atxHeaderRE = /^#+/
  ,   tocRE = /\[TOC\]/
  ,   emojiRE = /^:([A-Za-z0-9_\-\+]+?):/
  ..
```

在`blockNormal`方法里为匹配到的标签返回独立的class:
```
    ...
    } else if(match = stream.match(tocRE)){
      return toc;
    } else if(match = stream.match(emojiRE)){
      return emoji;
    }
    ...
```
这样就搞定了,编辑器会为匹配到的代码加上相应的class
![](http://7ximoo.com1.z0.glb.clouddn.com/r4af6mn4fhl4n4yq7oumqb3ce4.png)
![](http://7ximoo.com1.z0.glb.clouddn.com/fhlt1ecew21627h8764oe2ji0y.png)
有了,class,就可以在样式修改自定义语法的高亮显示了,比如我现在用的样式文件[mdn-like](https://github.com/benqy/hexomd/blob/master/app/lib/codemirror/theme/mdn-like.css)
打开这个样式文件,加上样式:
```
.cm-toc{
  background:#ccc;
}
.cm-emoji{
  color:#F7A21B;
}
```
现在这些语法在编辑器里有独特的高亮效果了:
![](http://7ximoo.com1.z0.glb.clouddn.com/jp3qgvvzgubzww1x3r29umfyo8.png)

## 总结

通过两个自定义语法的实现,我们可以总结出自定义语法的一般步骤:

1. 增加语法关键词的匹配正则.
2. 在`Renderer`里增加相应的标签生成方法.
3. 在`InlineLexer`里处理匹配到的语法.

接下来的计划:

1. 导出pdf,html文件.
2. 美化预览样式.

## 附件
[v0.6.0.2](http://pan.baidu.com/s/1gdtkr6r)
[项目地址](https://github.com/benqy/hexomd)