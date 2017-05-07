---
layout: post
title: "Android draw text clock"
date: 2014-08-27 19:47:16 +0800
comments: true
categories: android view
---
之前在看“伪物语”的时候，看到墙上的某个时钟看起来特别精致，突然间就想做出来试试。可是就算看完了动画，我对如何画那个钟还是没有一点想法。  
<!-- more -->

![][text_clock_dst]  
主要的难题就是黑白交界处文字的绘制:单个文字有部分是黑色的，有部分是白色的。  
按照之前看到过的Titanic的思路，可以用BitmapShader来做。但是如果使用BitmapShader的话，又需要创建Bitmap，或者读取一个Bitmap。最最重要的是，如果是用BitmapShader来做的话，可定制性太差了。如果我想**让字体黑色跟白色部分的跟着时针的转动一起变化**呢？可以做到这样子吗？  
关于这个问题，想了几天，在某天晚上突然想到了某种三层布局：1）背景色 2）中间可变化的半圆区域 3）透明的字体  
一开始觉得终于找到解决方案了，可是后来越想越有问题。想了一天，感觉在这样子下去不行。**打鸡血!**先尝试再说。  
#### 1. 在12个顶点绘制字
一开始有想过使用 `drawTextOnPath()` 的方法，但是绘制之后发现字体也会绕着Path旋转，另外字体间隔也不好控制，最终放弃了。还是老老实实地计算坐标了。
```java
protected void onSizeChanged() {
		...
		float sRadius = mRect.height() * 0.4f;
		float offset = mRect.height() * 0.1f;
		mPointFs.clear();
		for (int angle = 30; angle < 390; angle += 30) {
			float x = sRadius * (1 + (float) Math.sin(Math.toRadians(angle)));
			float y = sRadius * (1 - (float) Math.cos(Math.toRadians(angle)));
			mPointFs.add(new PointF(offset + x, offset + y));
		}	
}
```
计算出坐标之后就不难做了，直接在canvas里面按照给定的字符串绘制字体就好了。但是有个问题要考虑，就是字体的宽度和高度。  
绘制字体的时候是以左上角为起点的，如果没有进行计算字体的宽高进行绘制的话，字体的位置就无法居中在计算的点。  
字体的宽高只能通过 `Paint.getTextBound()` 来获取，因为 `Paint.measureText()` 只能用来计算宽度。拿到宽高后，直接改坐标绘制：
```java
canvas.drawText(num, pos + i + offset, 
		pos + i + 1 + offset, x, 
		y + mTextRect.height() * 0.5f, mTextPaint);
```

#### 2. 绘制“拾壹”“拾贰”的竖直布局
之前使用纯数字的，所以没怎么注意。在写成文字之后，仔细看了下，发现11跟12对应的文字需要竖直排列。似乎竖直布局还是蛮麻烦的。  
一开始还是想用path的方式来做，但是后来发现不行，文字会横着过来。而我需要的效果是一个字一个字地叠下来。  
那么加个换行符号？还是不行，`Canvas.drawText()` 的时候是会把换行符给忽略的。后来在网上找了下，英文必须无果--因为他们没有这种需求。有的只有中国跟日本的那些竖排文字。

#### 3. 绘制半黑半白的字体

##### 注意：
- Canvas的 `drawTextOnPath()` 方法不支持实时预览。
- `drawTextOnPath()` 会将文字自动旋转，相当于绘制文字的时候已经旋转了画布
- Paint的两个绘制文字的方法 `measureText()` 和 `getTextBound()` 在底层做的事情是一样的。唯一的区别是前者返回来的是 `float` 而后者返回的是 `int`， 而且是宽高都有。 


##### 遇到的问题
我的git又遇到问题了，一直提示unpack error，看下出错记录，发现是Author的名字出错了“=BrainKu <<kuxinwei@163.com>"。这是什么啊= =  
最后是用rebase来解决问题了
```
git rebase -i -p HEAD (before the commit you want to change)
git commit --amend --author "BrainKu <kuxinwei@163.com>"
git rebase --continue
```

参考资料  
[git change commit author]

[text_clock_dst]: /images/androi-text-clock/text_clock_dst.jpg
[text_clock_00]: /images/androi-text-clock/00.jpg
[git_change_commit_author]: http://stackoverflow.com/questions/750172/change-the-author-of-a-commit-in-git


