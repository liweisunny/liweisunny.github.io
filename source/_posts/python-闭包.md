---
title: python 闭包
date: 2017-12-18 11:03:42
tags: python基础
toc: true
categories: Python
---

首先，什么是闭包？，它的定义如下：

如果在一个内部函数里，对在外部作用域 (但不是全局作用域)的变量进行引用，那么内部函数就被认为是闭包。

定义在外部函数内的但由内部函数引用或者使用的变量被称为自由变量。

分析一下满足闭包的必备条件：

1. 需要函数嵌套, 就是一个函数里面再写一个函数.
2. 外部函数中有一些局部变量, 并且, 这些局部变量在内部函数中有使用。
3. 外部函数必须返回内嵌函数
<!--more-->

看一个简单的例子：

	def counter(start_at=0): 
		count = [start_at] 
		def incr():
			count[0] += 1
			return count[0]
		return incr

上面的例子使用闭包实现了一个计数器，调用结果如下：

	>>> cou=counter(5)
	>>> cou()
	6
	>>> cou()
	7

# \_\_closure\_\_属性

在Python中，函数对象有一个__closure__属性，我们可以通过这个属性看看闭包的一些细节。

	def counter(start_at=0):
    	count = [start_at]
    	def incr():
        	count[0]+=1
        	return count
    	return incr

	func=counter(5)
	print(dir(func))
	print(func.__closure__)
	print(type(func.__closure__))
	print(type(func.__closure__[0]))
	print(func.__closure__[0].cell_contents)

执行结果：

	['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
	(<cell at 0x00000000029C85E8: list object at 0x000000000291CE88>,)
	<type 'tuple'>
	<type 'cell'>
	[5]

通过__closure__属性看到，它对应了一个tuple，tuple的内部包含了cell类型的对象。

对于这个例子，可以得到cell的值（内容）为”[5]”，也就是变量”count”的值。

从这里可以看到闭包的原理，当内嵌函数引用了包含它的函数（enclosing function）中的变量后，这些变量会被保存在enclosing function的__closure__属性中，成为enclosing function本身的一部分；也就是说，这些变量的生命周期会和enclosing function一样。

# 闭包和函数
闭包只是在表现形式上跟函数类似，但实际上不是函数。

	def counter(start_at=0):
    	count = [start_at]
    	def incr():
        	count[0]+=1
        	return count
    	return incr

	func1=counter(5)
	print ("function name is:", func1.__name__)
	print ("id of mGreeting is:", id(func1))

	func2=counter(5)
	print ("function name is:", func2.__name__)
	print ("id of mGreeting is:", id(func2))


执行结果：

	('function name is:', 'incr')
	('id of mGreeting is:', 44399080L)
	('function name is:', 'incr')
	('id of mGreeting is:', 44399192L)

从代码的结果中可以看到，闭包在运行时可以有多个实例。

# 总结

本文介绍了如何通过Python创建一个闭包，以及Python创建的闭包是如何工作的。