---
title: python黑魔法—装饰器
date: 2017-12-26 09:37:26
tags: python基础
toc: true
categories: Python
---
先来一个形象的比喻：

内裤可以用来遮羞，但是到了冬天它没法为我们防风御寒，聪明的人们发明了长裤，有了长裤后宝宝再也不冷了，装饰器就像我们这里说的长裤，在不影响内裤作用的前提下，给我们的身子提供了保暖的功效。

再回到我们的主题

<!--more-->

**什么是装饰器？**

装饰器本质上是一个Python函数，它可以让其它函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值是一个函数对象。

**装饰器有哪些使用场景？**

它经常用于有切面需求的场景，比如：**插入日志、性能测试、事务处理、缓存、权限校验**等场景。
装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。

#装饰器函数

我们用一个小例子一步步引出装饰器：

	def fun():
		print('my name is fun')

上面我们定义了一个fun函数，函数运行时告诉用户一些信息。

现在我们有一个新的需求，需要给增加日志记录的功能，怎么办？

	import logging
	logging.basicConfig(filename='log1.log',level=logging.DEBUG)

	def fun():
    	print('my name is fun')
    	logging.debug('fun is running')

	fun()

我们借助了python内置的logging模块实现了日志记录的功能，运行上述代码能成功记录日志，新增的需求开发完了？真的开发完了吗？如果我们有fun1(),fun2()...等其它函数也需要记录日志呢，难道每一个函数都要加上一句logging.debug()语句吗？这样的话会有很多重复代码；我们可以定义一个专门记录日志的函数：

	import logging
	logging.basicConfig(filename='log1.log',level=logging.DEBUG)

	def use_log(fun):
    	logging.debug('%s is running'%fun.__name__)
    	fun()

	def fun():
    	print('my name is fun')

	use_log(fun)

逻辑上不难理解，而且解决了重复代码的问题 ，但是它有一个极大的弊端；它改变了 用户的请求方式，以前我们只需要调用fun()函数即可，现在我们必须调用	use_log(fun)才行，这是 一个很糟糕的设计，想一想，我们的程序可能已经发布到了生产环境，也行正在有很多用户在使用，这么一改，用户按照以前的方式调用，全挂掉了，这种设计还不如第一种呢....,那没有什么好办法了吗？在不改变原有调用方式的前提下，增加日志记录功能。当然有,就是我们的装饰器：

简单的装饰器：

	import logging
	logging.basicConfig(filename='log1.log',level=logging.DEBUG)

	def use_log(fun):
    	def wrapper():
        	logging.debug('%s is running'%fun.__name__)
        	fun()
    	return wrapper

	def fun():
    	print('my name is fun')


	fun=use_log(fun)
	fun()

上述调用方式看起来有点麻烦，幸运的是python中给我们提供了语法糖，上述代码直接修改成：

	@use_log  等价于 fun=use_log(fun)
	def fun():
    	print('my name is fun')
	fun()

use_log函数就是一个装饰器函数，内部函数 wrapper 是用来执行具体操作逻辑的函数。

不难看出usel_log其实也是一个闭包函数，没错，装饰器就是基于（闭包，高阶函数）实现的。

这样我们在不改变原有函数调用方式的前提下，增加了日志功能，并提高了代码的复用性。

上面的装饰器过于简单，如果我们要执行的函数带有参数怎么办？

## **给功能函数加参数**

	import logging
	logging.basicConfig(filename='log1.log',level=logging.DEBUG)
	def use_log(fun):
    	def wrapper(*args,**kwargs):
        	logging.debug('%s is running'%fun.__name__)
        	fun(*args,**kwargs)
    	return wrapper
	@use_log
	def fun(name):
    	print('my name is fun,params is %s'%name)
	fun('_learner')


我们将装饰器的内部函数定义成了 def wrapper(*args,**kwargs): 的形式，这样就可以接受任何参数了。

现在我们又有了一个新的需求，函数可以自己控制是否记录日志？这时就需要用到装饰器参数了

## **给装饰器函数加参数**


	import logging
	logging.basicConfig(filename='log1.log',level=logging.DEBUG)
	def use_Log(is_log=1):
    	def decorator(fun):
        	def wrapper(*args,**kwargs):
            	if is_log:
                	logging.debug('%s is running'%fun.__name__)
            	fun(*args,**kwargs)
        	return wrapper
    	return decorator
	@use_Log(0) 等价于 @decorator 等价于 fun=decorator(fun)
	def fun(name):
    	print('my name is fun,params is %s'%name)
	fun('_learner')

给装饰器函数加参数的方式：多定义一层外部函数，用户接受参数，该外部函数的返回值就是装饰器函数。

# **类装饰器**

再来看看类装饰器，相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器还可以依靠类内部的\_\_call\_\_方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

## 装饰器无参数，被装饰对象有参数

	class Decorator:
    	def __init__(self,func):# func是被装饰的对象
        	self.func=func
    	def __call__(self,*args, **kwargs):#*args, **kwargs用于承载被装饰对象的参数
            print('class decoratoe is running')
            self.func(*args,**kwargs)
            print('class decorator is runned')
	@Decorator
	def fun(name):
    	print('my name is %s'%name)
	fun('_learner')

运行结果：

	class decoratoe is running
	my name is _learner
	class decorator is runned

## 装饰器有参数，被装饰对象也有参数

	class Decorator:
    	def __init__(self,is_print): # is_print属于装饰器参数
        	self.is_print=is_print
    	def __call__(self,func): # func是被装饰的对象
        	def _cal(*args, **kwargs):#*args, **kwargs用于承载被装饰对象的参数
            	if self.is_print:
                	print('class decoratoe is running')
                	func(*args,**kwargs)
                	print('class decorator is runned')
            	else:
                	func(*args, **kwargs)
        	return _cal
	@Decorator(0)
	def fun(name):
    	print('my name is %s'%name)
	fun('_learner')

运行结果：

	my name is _learner

# **functools.wraps**

使用装饰器极大地复用了代码，但是他有一个缺点就是原函数的元信息不见了，比如函数的docstring、__name__、参数列表，先看例子：

	def logged(fun):
    	def with_logging(*args,**kwargs):
        	'''with_logging'''
        	print('was called: %s'%fun.__name__)
        	return fun(*args,**kwargs)
    	return with_logging
	@logged
	def f(x):
    	''' does some math'''
    	return x*x
	f(5)
	print(f.__name__)
	print(f.__doc__)

运行结果：

	was called: f
	with_logging
	with_logging

能看出函数f的docstring、__name__属性值全部变成了内部函数with_logging的了；这个问题是比较严重的，好在我们有functools.wraps，wraps本身也是一个装饰器，它能把原函数的元信息拷贝到装饰器函数中，这使得装饰器函数也有和原函数一样的元信息了。

	from  functools import wraps
	def logged(fun):
    	@wraps(fun)
    	def with_logging(*args,**kwargs):
        	'''with_logging'''
        	print(with_logging.__doc__)
        	print(with_logging.__name__)
        	return fun(*args,**kwargs)
    	return with_logging
	@logged
	def f(x):
    	'''does some math'''
    	return x*x
	f(5)
	print(f.__name__)
	print(f.__doc__)

运行结果：

	does some math
	f
	f
	does some math


# **装饰器执行顺序**
	
	@a
	@b
	@c
	def fun():
		pass

等价于

	fun=a(b(c(fun)))


# **内置装饰器**

@staticmathod、@classmethod、@property

# 总结

装饰器是python中一个相当重要的内容，它是在闭包合高阶函数的基础上实现的；可以用函数装饰器、类装饰器等；它们分别都可以装饰函数、类等。