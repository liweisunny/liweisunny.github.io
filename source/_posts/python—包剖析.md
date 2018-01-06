---
title: python—包剖析
date: 2017-12-29 17:11:03
tags: python基础
toc: true
categories: python
---
包是一个有层次的文件目录结构, 它定义了一个由模块和子包组成的 Python 应用程序执行
环境。Python 1.5 加入了包, 用来帮助解决如下问题:

1. 为平坦的名称空间加入有层次的组织结构
2. 允许程序员把有联系的模块组合到一起
3. 允许分发者使用目录结构而不是一大堆混乱的文件
4. 帮助解决有冲突的模块名称

与类和模块相同, 包也使用句点属性标识来访问他们的元素。 使用标准的 import 和
from-import 语句导入包中的模块。
<!--more-->

# 包的目录结构

	Phone/
		__init__.py
		common_util.py
		Voicedta/
			__init__.py
			Pots.py
		Fax/
			__init__.py
			G3.py
		Mobile/
			__init__.py
			Analog.py
			Digital.py
		Pager/
			__init__.py
			Numeric.py
Phone 是最顶层的包, Voicedta 等是它的子包。 

从上面的目录结构我们发现每一个包中都包含一个\_\_init\_\_.py文件。

这个文件有什么作用呢？

1. \_\_init\_\_.py的第一个作用就是包的标识，如果没有该文件，该目录就不会认为是包。
2. \_\_init\_\_.py的另外一个作用就是定义包中的\_\_all\_\_变量，该变量包含执行 from module import * 这样的语句时应该导入的模块的名字. 它由一个模块名字符串列表组成。

# 导入包

## import语句

	import Phone.Mobile.Analog #导入子包中的模块

	Phone.Mobile.Analog.dial() #调用子包模块中的函数dial()

## from-import语句

	from Phone import Mobile
	Mobile.Analog.dial('555-1212')

	from Phone.Mobile import Analog
	Analog.dial('555-1212')

    from package.module import *