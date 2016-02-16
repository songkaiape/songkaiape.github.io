title:  Markdown
date: 2016-2-16 9:48:11
tags: markdown
categories: 技术相关

#Markdown简介 
Markdown是由Jonh Gruber创建的一种轻量级的标记语言,目标是实现「易读易写」。它让人们能够以纯文本格式来编写文档，然后转换成html 文档。因为易读易写的特性，在写博客的人群中大受欢迎。


#Markdown 编辑器
推荐以下几种Markdown编辑器：


>1. Mac: Mou、Byword
>2. Windows: Sublime Text+Markdown 插件 、MarkdownPad
>3. Linux: Retext
>4. 在线编辑器：[简书](http://jianshu.com),[stackedit](https://stackedit.io/editor#)


#Markdown常用语法

##标题
Markdown标题支持类 Setext 和类 atx 形式。
类Setext使用底线的格式，利用*=*(最高阶标题)和*-*(第二阶标题)，例如：
```
This is an H1
=======
This is an H2
------------
```
任意数量的*=*和*-*都有效果
类Atx的形式则是在行首插入1到6个*#*，对应标题1到6阶，例如：
```
#这是 H1
##这是 H2
##### 这是H5
```
实际效果如下
#这是 H1
##这是 H2
##### 这是H5


##区块引用
Markdown 标记区块引用是使用类似email的方式，在每行头部加上*>*，当然你也可以偷懒只在区块的第一行添加。区块引用也可以嵌套
```
>This is a paragraph
>hello world
>>hello world    嵌套使用
>
>hello world 
```

##列表
Markdown 支持有序列表和无序列表。
无序列表使用星号、加号或是减号作为列表标记：
```
*  red
*  blue
*  yellow
```
有序列表使用数字接着一个英文句点：
```
1.  Bird
2.  Fish
3.  Dragon
```

##代码区块
建立代码区块只需要简单的缩进4个空格或者一个制表符就可以。

	这是一个普通段落。
		这是一个代码区块

也可以使用`来标志如下所示：

	```python
	def say(gt):
		print("hello,world")
	```

##分割线
使用3个以上的星号,减号，或者底线就可以实现：

* * *
- - -
	***
	---
如上所示但是分割线行内不能出现除空格符以上三种字符以外其他字符

##超链接

Markdown支持两种形式的链接语法：**行内式**和**参考式**。
两种都使用[方括号]来标记链接文字。
**行内式**只需要在方括号后面紧跟着圆括号并且插入链接网址即可，如果希望加入链接的title文字，只需要网址后面用双引号把title文字包含起来。

	This is [a link](http://google.com/"title") inline link.
也可以使用相对路径：
	
	This is [about](/about/) page
**参考式**链接是在链接文字接上另一个方括号，而在第二个方括号里面要填入用以辨识的链接的标记:

	This is [my website][Inspiration]

然后你可以在任意位置把这个标记的链接内容定义出来

	[Inspiration]: http://songkaiape.github.io

##强调

Markdown使用*和_表示强调
语法：

	单星号 = *斜体*
	单下划线 = _斜体_
	双星号 = **加粗**
	双下划线 = __加粗__

##图片
图片使用方法和链接类似，只需要在中括号前面加**叹号**。
>*markdown 语法不能设置图片大小，如果必须设置则应该使用HTML标记<img>*

	行内式：！[文本]（/img/background.jpg "title"）
	参考式：！[文本][pic]
	[pic]：/img/1.jpg "title"
	HTML：<img src="/img/1.jpg" alt="文本" title="标题文本" width="200"/>


#结语

Markdown语法因为编辑器的不同可能会有些细微差别，使用时请注意多多查询。


#参考
http://equation85.github.io/blog/markdown-examples/
http://wowubuntu.com/markdown/
http://snails.github.io/2012/05/08/Learn-to-Markdown/



