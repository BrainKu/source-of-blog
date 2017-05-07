---
layout: post
title:  "搭好了，我的github博客"
date:   2014-08-23 17:14:00
tags: 博客
---

折腾了一个下午之后终于把本地的Jekyll环境搭好并且运行起来了，可喜可贺。  
说到我为什么要建博客，大概是因为一直没找到一个*省钱*又省心的环境来写些记录吧。最近又开始在想，虽然我一直都有在记录思维导图，但是思维导图也是有些碎片化了吧。没有好好整理自己的想法的话，会像是在重造轮子。  
<!-- more-->
举个栗子，比如原先我看了某个博客，理解了某段内容，如果没有按照自己的想法记录下来，下次再看到似乎也是一个再消化的过程。
在举个栗子，阅读了别人的自定义控件之后，我尝试按照自己的想法写自己的控件。这个写控件的过程就是一个遇到问题，再查找资料，最终解决问题的过程。如果不记录这个过程，或者说*因为思维导图的关系*无法详细描述。下次再遇到这样子的问题，你可以记住那些"原本不重要的"细节吗？  
所以呢，搭个博客来写写感想还是蛮不错的吧~域名的事再另外考虑，写博客是个积累的过程嘛。现在只是开始，不急。

-----
在搭博客的过程中参考了多位大牛的资料，贴在下面以防下次出问题方便查找~  
[BeeYuu][BeeYuuGit]        [使用Github Pages建独立博客 ][BeeYuu]  
[Run Jekyll on Windows][Jekyll_in_windows]  
[Jekyll 官方文档][jekyll]  
[Jekyll本地调试][jekyll_debug]  

-----
> 21:18

继续写下我遇到的事情。
我本来想在PHPStorm里面写下HTML跟CSS之类的，但是似乎没办法的样子诶。  
PHPStorm是支持SASS跟SCSS，但是Jekyll的CSS文件里面有部分加了自己的语法- - 
~~我现在就觉得Jekyll的sass目录结构有病，导入的先后顺序应该不是那个样子的吧？~~
好像如果在页面的开始标注上--- ----的话Jekyll会按照自己的语法规则来处理scss。删掉的话会怎样？  
尝试了一下，发现真的无法处理正常的结构目录。

> 00:25

发现了一个OctoPress的东西，这东西把Jekyll封装了起来，而且还加了很多主题~今晚就折腾这个东西过去了。  
感觉还是git比较花时间啊= =几个命令都不熟来着。

[BeeYuu]: 	     http://beiyuu.com/github-pages
[jekyll]:        http://jekyllrb.com/docs/home/
[BeeYuuGit]:   	 https://github.com/beiyuu
[jekyll_debug]:  http://chxt6896.github.io/blog/2012/02/13/blog-jekyll-native.html
[Jekyll_in_windows]: http://jekyll-windows.juthilo.com/