---
title: python中的下划线
date: 2017-12-24 15:56:58
tags: python基础
toc: true
categories: Python
---
这篇文章讨论Python中下划线_的使用。跟Python中很多用法类似，下划线_的不同用法绝大部分（不全是）都是一种惯例约定。
<!--more-->

# 单个下划线（_）
主要有三种情况：

1. 解释器中
_符号是指交互解释器中最后一次执行语句的返回结果。这种用法最初出现在CPython解释器中，其他解释器后来也都跟进了。

		>>> _
		Traceback (most recent call last):
		File "", line 1, in 
		NameError: name '_' is not defined
		>>> 42
		>>> _
		42
		>>> 'alright!' if _ else ':('
		'alright!'
		>>> _
		'alright!'

2. 作为名称使用
这个跟上面有点类似。_用作被丢弃的名称。按照惯例，这样做可以让阅读你代码的人知道，这是个不会被使用的特定名称。举个例子，你可能无所谓一个循环计数的值：

		n = 42
		for _ in range(n):
			do_something()

3. i18n
_还可以被用作函数名。这种情况，单下划线经常被用作国际化和本地化字符串翻译查询的函数名。这种惯例好像起源于C语言。举个例子，在 [Django documentation for translation](https://docs.djangoproject.com/en/dev/topics/i18n/translation/) 中你可能会看到：

		from django.utils.translation import ugettext as _
		from django.http import HttpResponse
		def my_view(request):
    		output = _("Welcome to my site.")
    		return HttpResponse(output)

第二种和第三种用法会引起冲突，所以在任意代码块中，如果使用了_作i18n翻译查询函数，就应该避免再用作被丢弃的变量名。

# 单下划线前缀的名称（例如_shahriar）

单下划线开头被常用于模块中，在一个模块中以单下划线开头的变量和函数被默认当作内部函数,“私有成员”，如果使用 from a_module import * 导入时，这部分变量和函数不会被导入,除非模块/包的\_\_all\_\_列表明确包含了这些名称。不过值得注意的是，如果使用 import a_module 这样导入模块，仍然可以用 a\_module.\_some_var 这样的形式访问到这样的对象。


# 双下划线前缀的名称（例如__shahriar）

以双下划线做前缀的名称（特别是方法名）并不是一种惯例；它对解释器有特定含义。Python会改写这些名称，以免与子类中定义的名称产生冲突。Python documentation中提到，任何\_\_spam这种形式（至少以两个下划线做开头，绝大部分都还有一个下划线做结尾）的标识符，都会文本上被替换为\_classname\_\_spam，其中classname是当前类名，并带上一个下划线做前缀。

看下面这个例子：

	>>> class A(object):
	...     def _internal_use(self):
	...         pass
	...     def __method_name(self):
	...         pass
	... 
	>>> dir(A())
	['_A__method_name', ..., '_internal_use']

正如所料，_internal_use没有变化，但\_\_method\_name被改写成了\_ClassName\_\_method\_name。现在创建一个A的子类B（这可不是个好名字），就不会轻易的覆盖掉A中的\_\_method\_name了：

	>>> class B(A):
	...     def __method_name(self):
	...         pass
	... 
	>>> dir(B())
	['_A__method_name', '_B__method_name', ..., '_internal_use']

这种特定的行为差不多等价于Java中的final方法和C++中的正常方法（非虚方法）。

# 前后都带有双下划线的名称（例如 \_\_init\_\_）

这些是Python的特殊方法名，这仅仅是一种惯例，一种确保Python系统中的名称不会跟用户自定义的名称发生冲突的方式。通常你可以覆写这些方法，在Python调用它们时，产生你想得到的行为。例如，当写一个类的时候经常会覆写\_\_init\_\_方法。Python 官方推荐永远不要将这样的命名方式应用于自己的变量或函数，而是按照文档说明来使用。
