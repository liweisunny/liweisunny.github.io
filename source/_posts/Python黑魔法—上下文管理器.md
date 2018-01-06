---
title: Python黑魔法—上下文管理器
date: 2017-12-20 16:51:59
tags: python基础
toc: true
categories: Python
---
# 什么是上下文管理器
上下文管理器顾名思义是管理上下文的,也就是负责冲锋和垫后,而让主人专心完成自己的事情。我们在编写程序的时候，通常会将一系列操作放到一个语句块中，当某一条件为真时执行该语句快。有时候，我们需要再执行一个语句块时保持某种状态，并且在离开语句块后结束这种状态。例如对文件的操作，我们在打开一个文件进行读写操作时需要保持文件处于打开状态，而等操作完成之后要将文件关闭。所以，上下文管理器的任务是：代码块执行前准备，代码块执行后收拾。上下文管理器是在Python2.5加入的功能，它能够让你的代码可读性更强并且错误更少。

需求的产生
在正常的管理各种系统资源(文件、锁定和连接)，在涉及到异常时通常是个棘手的问题。异常很可能导致控制流跳过负责释放关键资源的语句。例如打开一个文件进行操作时，如果意外情况发生（磁盘已满、特殊的终端信号让其终止等），就会抛出异常，这样可能最后的文件关闭操作就不会执行。如果这样的问题频繁出现，则可能耗尽系统资源。

是的，这样的问题并不是不可避免。在没有接触到上下文管理器之前，我们可以用“try/finally”语句来解决这样的问题。或许在有些人看来，“try/finally”语句显得有些繁琐。上下文管理器就是被设计用来简化“try/finally”语句的，这样可以让程序更加简洁。
<!--more-->

# With语句
With语句用于执行上下文操作，它也是复合语句的一种，其基本语法如下所示：
	
	with context_expr [as var]:
    	with_suite
With 语句仅能工作于支持上下文管理协议(context management protocol)的对象。也就是说只有内建了”上下文管理”的对象才能和 with 一起工作。Python内置了一些支持该协议的对象，如下所列是一个简短列表：

1. file
2. decimal.Context
3. thread.LockType
4. threading.Lock
5. threading.RLock
6. threading.Condition
7. threading.Semaphore
8. threading.BoundedSemaphore

由以上列表可以看出，file 是已经内置了对上下文管理协议的支持。所以我们可以用下边的方法来操作文件：

	with open('/etc/passwd', 'r') as f:
    	for eachLine in f:
        	# ...do stuff with eachLine or f...

上边的代码试图打开一个文件,如果一切正常,把文件对象赋值给 f。然后用迭代器遍历文件中的每一行,当完成时,关闭文件。无论是在这一段代码的开始,中间,还是结束时发生异常,会执行清理的代码,此外文件仍会被自动的关闭。

# 自定义上下文管理器
要实现上下文管理器，必须实现两个方法：一个负责进入语句块的准备操作，另一个负责离开语句块的善后操作。Python类包含两个特殊的方法，分别名为：**\_\_enter\_\_ 和 \_\_exit\_\_**。

\_\_enter\_\_: 该方法进入运行时上下文环境，并返回自身或另一个与运行时上下文相关的对象。返回值会赋给 as 从句后面的变量，as 从句是可选的。

\_\_exit\_\_: 该方法退出当前运行时上下文并返回一个布尔值，该布尔值标明了“如果 with_suit 的退出是由异常引发的，该异常是否须要被忽略”。如果 \_\_exit\_\_() 的返回值等于 False，那么这个异常将被重新引发一次；如果 \_\_exit\_\_() 的返回值等于 True，那么这个异常就被无视掉，继续执行后面的代码。

With 语句的实际执行流程是这样的：

1. 执行 context_exp 以获取上下文管理器
2. 加载上下文管理器的 \_\_exit\_\_() 方法以备稍后调用
3. 调用上下文管理器的 \_\_enter\_\_() 方法
4. 如果有 as var 从句，则将 \_\_enter\_\_() 方法的返回值赋给 var
5. 执行子代码块 with_suit
6. 调用上下文管理器的 \_\_exit\_\_() 方法，如果 with_suit 的退出是由异常引发的，那么该异常的 type、value 和 traceback 会作为参数传给 \_\_exit\_\_()，否则传三个 None。
7. 如果 with_suit 的退出由异常引发，并且 \_\_exit\_\_() 的返回值等于 False，那么这个异常将被重新引发一次；如果 \_\_exit\_\_() 的返回值等于 True，那么这个异常就被无视掉，继续执行后面的代码

下面我们自己来实现一个支持上下文管理协议的类：

	class Query(object):

    	def __init__(self, name):
        	self.name = name

    	def __enter__(self):
        	print('Begin')
        	return self

    	def __exit__(self, exc_type, exc_value, traceback):
        	if exc_type:
            	print('Error')
        	else:
            	print('End')

    	def query(self):
        	print('Query info about %s...' % self.name)

这样我们就可以把自己写的资源对象用于with语句：

	with Query('_learner') as q:
    	q.query()

执行结果：

	Begin
	Query info about _learner...
	End

# 上下文管理工具（contextlib模块）

**contextlib**模块提供更易用的上下文管理器。

## @contextmanager

编写__enter__和__exit__仍然很繁琐，因此Python的标准库contextlib提供了更简单的写法，上面的代码可以改写如下：

	from contextlib import contextmanager

	class Query(object):

    	def __init__(self, name):
        	self.name = name

    	def query(self):
        	print('Query info about %s...' % self.name)

	@contextmanager
	def create_query(name):
    	print('Begin')
    	q = Query(name)
    	yield q
    	print('End')

**@contextmanager**这个decorator接受一个generator，用yield语句把with ... as var把变量输出出去，然后，with语句就可以正常地工作了：

	with create_query('_learner') as q:
    	q.query()

执行结果：

	Begin
	Query info about _learner...
	End

很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现。例如：

	@contextmanager
	def tag(name):
    	print("<%s>" % name)
    	yield
    	print("</%s>" % name)

	with tag("h1"):
    	print("hello")
    	print("world")
上述代码执行结果为：

	<h1>
	hello
	world
	</h1>
代码的执行顺序是：

1. with语句首先执行yield之前的语句，因此打印出<h1\>；
2. yield调用会执行with语句内部的所有语句，因此打印出hello和world；
3. 最后执行yield之后的语句，打印出</h1\>。

因此，@contextmanager让我们通过编写generator来简化上下文管理。

## @closing

如果一个对象没有实现上下文，我们就不能把它用于with语句。这个时候，可以用closing()来把该对象变为上下文对象。例如，用with语句使用urlopen()：

	from contextlib import closing
	from urllib.request import urlopen
	with closing(urlopen('https://www.python.org')) as page:
    	for line in page:
        	print(line)

**closing**也是一个经过**@contextmanager**装饰的generator，这个generator编写起来其实非常简单：

	@contextmanager
	def closing(thing):
    	try:
        	yield thing
    	finally:
        	thing.close()

重写上面的例子：

	from contextlib import  contextmanager
	from urllib.request import urlopen
	@contextmanager
	def closing(thing):
    	try:
        	yield thing
    	finally:
        	thing.close()
	with closing(urlopen('https://www.python.org')) as page:
    	for line in page:
        	print(line)

它的作用就是把任意对象变为上下文对象，并支持with语句。