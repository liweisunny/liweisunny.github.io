---
title: python—模块剖析
date: 2017-12-28 11:12:17
tags: python基础
toc: true
categories: python
---
在计算机程序的开发过程中，随着程序代码越写越多，在一个文件里代码就会越来越长，越来越不容易维护。为了编写可维护的代码，我们把很多函数分组，分别放到不同的文件里，这样，每个文件包含的代码就相对较少，很多编程语言都采用这种组织代码的方式。在Python中，一个.py文件就称之为一个模块（Module）。

使用模块有什么好处？

最大的好处是大大提高了代码的可维护性。其次，编写代码不必从零开始。当一个模块编写完毕，就可以被其他地方引用。我们在编写程序的时候，也经常引用其他模块，包括Python内置的模块和来自第三方的模块。
<!--more-->

# 导入模块

## 模块导入原理

模块的导入需要一个叫做"路径搜索"的过程。 即在文件系统"预定义区域"中查找 mymodule.py
文件(如果你导入 mymodule 的话)。 这些预定义区域只不过是你的 Python 搜索路径的集合。

路径搜索和搜索路径是两个不同的概念, 前者是指查找某个文件的操作, 后者是去查找一组目录。

解释器启动之后可以通过Python的sys.path属性获得当前搜索路径集合：

	>>> import sys
	>>> sys.path
	['C:\\Python36\\DLLs', 'C:\\Python36\\lib', 'C:\\Python36', 'C:\\Python36\\lib\\site-packages', 'C:\\Python36\\lib\\site-packages\\pip-9.0.1-py3.6.egg', 'C:\\Python36\\lib\\site-packages\\pymysql-0.7.11-py3.6.egg', 'C:\\Python36\\lib\\site-packages\\requests-2.18.4-py3.6.egg', 'C:\\Python36\\lib\\site-packages\\certifi-2017.7.27.1-py3.6.egg']

如果你知道你需要导入的模块是什么,而它的路径不在搜索路径里, 那么只需要向sys.path 添加即可：

	sys.path.append('/home/wesc/py/lib')

由于append()方法是把元素放在集合尾部，**解释器是按照搜索路径顺序查找匹配的第一个模块名称**，如果你有特殊需要, 那么应该使用列表的 insert() 方法操作。

补充：sys.path集合中值得顺序是：当前目录--->环境变量PYTHONPATH中配置的目录--->python的安装设置相关的默认路径

## 模块导入方式

### import 语句

	import mymodule

这种方式导入模块，就在当前的名称空间(namespace)建立了一个到该模块的引用。这种引用必须使用全称，也就是说，当使用在被导入模块中定义的函数时，必须包含模块的名字。所以不能只使用funcname，而应该使用 mymodule.funcname

当前的名称空间(namespace)，意思就是说如果在一个模块的顶层导入, 那么它的作用域就是全局的; 如果在函数中导入, 那么它的作用域是局部的。

如果mymodule模块是被第一次导入, 它将被加载并执行。

### from-import语句

	from module import name1[ name2[,....nameN]

from-import语句导入模块的指定属性。name1、name2... 直接被导入到当前名称空间里去，这意味着你不需要使用属性/句点属性标识来访问模块的标识符，可直接使用name1、name2。

值得提一下的是：

	from module import *

上面这句话是把module模块中的’所有‘名称导入到当前名称空间中，但是：

1. 如果module中定义了\_\_all\_\_变量，那只能导入\_\_all\_\_中的名称。

		# module.py
		__all__ = ['a', 'b']
		a = 10
		b = 20
		c=50
	只会导入 a、b两个名称。

	<font color=red>\_\_all\_\_只针对 from module import * 语句起作用。</font>

2. 没有定义\_\_all\_\_变量时，会导入所有非'_'开头的名称，在python中以'_'开头认为是私有的。

		# module.py
		a = 10
		_b = 20
		c=50
只会导入 a、c两个名称。

另外，我们认为 "from module import *" 不是良好的编程风格, 因为它"污染"当前名称
空间, 而且很可能覆盖当前名称空间中现有的名字。

我们只在两种场合下建议使用这样的方法, 一个场合是：目标模块中的属性非常多, 反复键入
模块名很不方便, 例如 Tkinter (Python/Tk) 和 NumPy (Numeric Python) 模块, 可能还有
socket 模块。另一个场合是在交互解释器下, 因为这样可以减少输入次数。

## 扩展的import语句(as)

意思是给自己导入的模块或模块中的名称起别名。

	import Tkinter as tk
	from cqi import FieldStorage as fiesto

## 导入模块执行机制

第一次载入时执行模块(执行是指模块的顶层代码将被直接执行)，一个模块只被执行一次，无论被导入多少次，这样可以阻止多重导入时代码被多次执行。这个机制是通过sys.modules控制的。

导入模块时，是创建一个名为模块名称的对象，该对象引用模块的名字空间，这样就可以通过这个对象访问模块中的函数及变量。当多次导入同一个模块时，后面的import语句只是简单的创建一个到模块名字空间的引用而已。

sys.modules是一个字典，保存着所有被导入模块的模块名到模块对象的映射。这个字典用来决定是否需要使用import语句来导入一个模块的最新拷贝。

## 模块导入规范

我们推荐所有的模块在 Python 模块的开头部分导入。 而且最好按照这样的顺序:

1. Python 标准库模块
2. Python 第三方模块
3. 应用程序自定义模块

然后使用一个空行分割这三类模块的导入语句。 这将确保模块使用固定的习惯导入, 有助于减
少每个模块需要的 import 语句数目。


# 模块内建函数

## \_\_import\_\_函数

	__import__(module_name[, globals[, locals[, fromlist]]])
	

__import__() 函数作为实际上导入模块的函数, 这意味着 import 语句调用 __import__() 函数完成它的工作。提供这个函数是为了让有特殊需要的用户覆盖它, 实现自定义的导入算法。
	
	#这两句话相等
	mymodule = __import__ (’module_name’)
	import module_name

## global()和locals()

globals() 和 locals() 内建函数分别返回调用者全局和局部名称空间的字典。 在一个函数内
部, 局部名称空间代表在函数执行时候定义的所有名字, locals() 函数返回的就是包含这些名字
的字典。 globals() 会返回函数可访问的全局名字。

在全局名称空间下, globals() 和 locals() 返回相同的字典, 因为这时的局部名称空间就是
全局空间。

	def foo():
		print '\ncalling foo()...'
		aString = 'bar'
		anInt = 42
		print "foo()'s globals:", globals().keys()
		print "foo()'s locals:", locals().keys()
	print "__main__'s globals:", globals().keys()
	print "__main__'s locals:", locals().keys() foo()

执行这个脚本, 我们得到如下的输出:
	
	>>> $ namespaces.py
	__main__'s globals: ['__doc__', 'foo', '__name__', '__builtins__']
	__main__'s locals: ['__doc__', 'foo', '__name__', '__builtins__']


## reload()

reload() 内建函数可以重新导入一个已经导入的模块。 它的语法如下:

	reload(module)

上述是python2中的写法，在python3中写法如下：

	import importlib
	importlib.reload(module)
使用 reload() 的时候有一些标准。 首先模块必须是全部
导入(不是使用 from-import), 而且它必须被成功导入。另外 reload() 函数的参数必须是模块自
身而不是包含模块名的字符串。 也就是说必须类似 reload(sys) 而不是 reload('sys')。

# 总结
说了那么多，主要记住,模块导入原理，根据sys.path搜索的；模块执行机制，根据sys.modules控制的；以及模块的导入方式就可以了，接下来会有博文介绍python中包的概念。