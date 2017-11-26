---
title: python 环境部署-Windows
date: 2017-11-15 16:53:48
tags: python环境部署
toc: true
---
# 第一步
①下载python解释器：	
  		
进入python官网：[http://www.python.org](http://www.python.org) 点击Downlods，选择Windows，然后看到如下界面：

![](https://i.imgur.com/L62Jf03.jpg)

圈出来的一个是python3版本一个是python2版本，建议都安装，日后可根据需求使用，**建议使用python3**，虽然说现在市场上大部分是使用python2,但是python2最终会被python3替换掉，一些新项目都是使用python3开发的。另外没必要过于纠结两个版本，他们在一些语法上使用是有些许不同，但整体相似。详情参考:[https://www.zhihu.com/question/19698598](https://www.zhihu.com/question/19698598 "python2和python3的区别")

下载好后直接安装即可，**记录住安装目录**。	
# 第二步
②配置环境变量：	

步骤(Win10)：右键单击电脑属性--->高级系统设置--->环境变量--->编辑系统变量Path--->将刚才的安装路径添加进去，如下图：

![](https://i.imgur.com/Akxo4lX.jpg)
我配置了python3和python2的环境变量，并把python3的放在了第一位，系统会采用环境变量靠前的python版本；环境变量设置的路径就是你python的安装路径，以python2为例，如下图：

![](https://i.imgur.com/9DFsk0G.jpg)

# 检验是否安装成功
打开cmd命令框，快捷键是windows+r ,输入cmd-->回车--->在弹出的命令框中输入 **python**,看到如下图内容即表示配置成功：

![](https://i.imgur.com/jbijBPp.jpg)

# 完善
为了方便后续的python版本切换，建议将python安装路径下的python.exe程序重命名为对应版本的名称，即python2.7版本的命名为：python2.exe，python3.6版本的命名为python3.exe；这样我们就可以在命令框中直接输入对应的名称就可以使用各版本的python，不用切换环境变量的先后顺序了。如下图：

![](https://i.imgur.com/Jvns0hw.jpg)

# 补充
这是最基本的python环境部署，后续在实际开发过程中我们还要安装很多类库、插件等；推荐安装pip,使用pip安装类库等十分方便。

如何安装pip,详见：[http://waisunny.com/2017/11/17/setuptools%E5%92%8Cpip%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/](http://waisunny.com/2017/11/17/setuptools%E5%92%8Cpip%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/ "pip安装")