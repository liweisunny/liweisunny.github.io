---
title: Python黑魔法—上下文管理器
date: 2017-12-20 16:51:59
tags: python基础
toc: true
categories: Python
---

在正常的管理各种系统资源(文件、锁定和连接)，在涉及到异常时通常是个棘手的问题。异常很可能导致控制流跳过负责释放关键资源的语句。例如打开一个文件进行操作时，如果意外情况发生（磁盘已满、特殊的终端信号让其终止等），就会抛出异常，这样可能最后的文件关闭操作就不会执行。如果这样的问题频繁出现，则可能耗尽系统资源。

是的，这样的问题并不是不可避免。在没有接触到上下文管理器之前，我们可以用“try/finally”语句来解决这样的问题。或许在有些人看来，“try / finally”语句显得有些繁琐。上下文管理器就是被设计用来简化“try / finally”语句的，这样可以让程序的代码可读性更强并且错误更少。

<!--more-->

# With基本语法

	with open('output', 'w') as f:
    	f.write('Hello world')

上面的代码往output文件写入了Hello world字符串，with语句会在执行完代码块后自动关闭文件。这里无论写文件的操作成功与否，是否有异常抛出，with语句都会保证文件被关闭。

如果不用with，我们可能要用下面的代码实现类似的功能

	try:
	    f = open("output", "w")
	    f.write("Hello world")
	finally:
	    f.close()

可以看到使用了with的代码比上面的代码简洁许多。

那么问题来了，上面的with代码背后发生了些什么？

我们来看下它的执行流程:
<font color=red size=3>

1. 首先执行open('output', 'w')，返回一个文件对象；
2. 调用这个文件对象的\_\_enter\_\_方法，并将\_\_enter\_\_方法的返回值赋值给变量f；
3. 执行with语句体，即with语句包裹起来的代码块；
4. 不管执行过程中是否发生了异常，执行文件对象的\_\_exit\_\_方法，在\_\_exit\_\_方法中关闭文件。

这里的关键在于open返回的文件对象实现了\_\_enter\_\_和\_\_exit\_\_方法。一个实现了\_\_enter\_\_和\_\_exit\_\_方法的对象就称之为上下文管理器。

需要说明的是：With 语句仅能工作于支持上下文管理协议(context management protocol)的对象。也就是说上述两个方法的对象才能和 with 一起工作。</font>

Python内置了一些支持该协议的对象，如下所列是一个简短列表：

1. file
2. decimal.Context
3. thread.LockType
4. threading.Lock
5. threading.RLock
6. threading.Condition
7. threading.Semaphore
8. threading.BoundedSemaphore

# 上下文管理器

上下文管理器定义执行 with 语句时要建立的运行时上下文，负责执行 with 语句块上下文中的进入与退出操作。\_\_enter\_\_方法在语句体执行之前进入运行时上下文，\_\_exit\_\_在语句体执行完后从运行时上下文退出。

在实际应用中，\_\_enter\_\_一般用于资源分配，如打开文件、连接数据库、获取线程锁；\_\_exit\_\_一般用于资源释放，如关闭文件、关闭数据库连接、释放线程锁。

# 自定义上下文管理器
既然上下文管理器就是实现了\_\_enter\_\_和\_\_exit\_\_方法的对象，我们能不能定义自己的上下文管理器呢？答案是肯定的。

我们先来看下\_\_enter__和\_\_exit\_\_方法的定义：

\_\_enter\_\_: 进入上下文管理器的运行时上下文，在语句体执行前调用。如果有as子句，with语句将该方法的返回值赋值给 as 子句中的 target。

\_\_exit\_\_(exc_type, exc_val, exc_tb)：退出与上下文管理器相关的运行时上下文，返回一个布尔值表示是否对发生的异常进行处理。如果with语句体中没有异常发生，则\_\_exit\_\_的3个参数都为None，即调用\_\_exit\_\_(None, None, None)，并且\_\_exit\_\_的返回值直接被忽略。如果有发生异常，则使用 sys.exc_info 得到的异常信息为参数调用\_\_exit\_\_方法。出现异常时，如果\_\_exit\_\_方法返回 False，则会重新抛出异常，让with之外的语句逻辑来处理异常；如果返回 True，则忽略异常，不再对异常进行处理。

理解了\_\_enter\_\_和\_\_exit\_\_方法后，我们来自己定义一个简单的用于文件操作的上下文管理器：


	class OpenFile:
	    def __init__(self, path, mark):
	        self.path = path
	        self.mark = mark
	
	    def __enter__(self):
	        print('__enter__ action starting')
	        self.f = open(self.path, self.mark)
	        return self.f
	
	    def __exit__(self, exc_type, exc_val, exc_tb):
	        print('__exit__ action starting')
	        self.f.close()
	        if exc_type is None:
	            print("[in __exit__] Exited without exception")
	        else:
	            print("[in __exit__] Exited with exception: %s" % exc_val)
	            return False

	if __name__ == "__main__":
		with OpenFile('test.txt', 'w') as f:
		    print('starting write.....')
		    f.write('123456')

运行上面的代码，会得到如下的输出:

	__enter__ action starting
	starting write.....
	__exit__ action starting
	[in __exit__] Exited without exception

我们修改with语句体人为的抛出一个异常：

	with OpenFile('test.txt', 'w') as f:
	    print('starting write.....')
	    f.write('Testing context managers')
	    raise(Exception("something wrong"))

会得到如下输出：

	__enter__ action starting
	starting write.....
	__exit__ action starting
	Traceback (most recent call last):
	  File "D:/Python/Project/InterviewQuestions/8.上下文管理器.py", line 63, in <module>
	    raise(Exception("something wrong"))
	Exception: something wrong
	[in __exit__] Exited with exception: something wrong


如我们所期待，with语句体中抛出异常，\_\_exit\_\_方法中exc_type不为None，\_\_exit\_\_方法返回False，异常被重新抛出。

以上，我们通过实现\_\_enter\_\_和\_\_exit\_\_方法来实现了一个自定义的上下文管理器。

# 上下文管理工具（contextlib模块）

**contextlib**模块提供更易用的上下文管理器。

## @contextmanager

使用contextlib的contextmanager函数作为装饰器，来创建一个上下文管理器。让我们将上面的代码修改成使用contextmanager的:
	
	from contextlib import contextmanager
	
	@contextmanager
	def file_open(path, mark):
	    try:
	        f_obj = open(path, mark)
	        yield f_obj
	    except :
	        print("We had an error!")
	    finally:
	        print("Closing file.....")
	        f_obj.close()
	
	if __name__ == "__main__":
	    with file_open("test.txt", 'w') as f:
	        f.write("Testing context managers")
在这里，我们从contextlib模块中引入contextmanager，然后装饰我们所定义的file_open函数。这就允许我们使用Python的with语句来调用file_open函数。在函数中，我们打开文件，然后通过yield，将其传递出去，最终主调函数可以使用它。


运行结果：

	starting write.....
	Closing file.....

修改代码：

	if __name__ == "__main__":
	    with file_open("test.txt", 'w') as f:
	        print('starting write.....')
	        f.write("Testing context managers")
	        raise(Exception("something wrong"))
运行结果：

	starting write.....
	We had an error!
	Closing file.....

一旦with语句结束，控制就会返回给file_open函数，它继续执行yield语句后面的代码。这个最终会执行finally语句--关闭文件。如果我们在打开文件时或者写入数据时遇到了错误，它就会被捕获，最终finally语句依然会关闭文件句柄。

同时能看出，使用@contextmanager让我们通过编写generator可以简化上下文管理的构造。

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

# 总结

上下文管理协议就介绍这么多，知其原理，才能将其合理的应用到程序中。