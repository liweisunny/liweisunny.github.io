---
title: python之异常处理
date: 2017-11-26 17:19:07
tags: python基础
categories: Python
toc: true
---
在谈如何处理python中的异常前，我们先来明白两个概念-->**错误和异常**。

**错误**：从软件方面来说, 错误是语法或是逻辑上的。语法错误指示软件的结构上有错误, 导致不能被解释器解释或编译器无法编译。 这些错误必须在程序执行前纠正。当程序的语法正确后, 剩下的就是逻辑错误了. 逻辑错误可能是由于不完整或是不合法的输入所致; 在其他情况下, 还可能是逻辑无法生成, 计算, 或是输出结果需要的过程无法执行。这些错误通常分别被称为域错误和范围错误。当 Python 检测到一个错误时, 解释器就会指出当前流已经无法继续执行下去. 这时候就出现了异常。

**异常**：对异常的最好描述是: 它是因为程序出现了错误而在正常控制流以外采取的行为. 这个行为又分为两个阶段: 首先是引起异常发生的错误, 然后是检测(和采取可能的措施)阶段即处理异常阶段。
<!--more-->
# python中的异常

所有的标准/内建异常都是从根异常派生的.目前,有 3 个直接从 BaseException 派生的异常子
类:SystemExit,KeyboardInterrupt 和 Exception。 

SystemExit：python解释器请求退出。

KeyboardInterrupt：用户中断执行(通常是输入^C)。

Exception 常规语法错误；所有的内建异常都是其子类。

**列举几个最常用的内建异常：**

**NameError**:尝试访问一个未声明的变量

**ZeroDivisionError**: 除数为零

**SyntaxError**: Python 语法错误（SyntaxError 异常是唯一不是在运行时发生的异常. 它代表 Python 代码中有一个不正确的结构, 在它改正之前程序无法执行）

**IndexError**:请求的索引超出序列范围

**KeyError**:请求一个不存在的字典关键字

**IOError**: 输入/输出错误

**AttributeError**: 尝试访问未知的对象属性

# 检测和处理异常

使用try语句来检测异常。try 语句有两种主要形式: try-except 和 try-finally。这两个语句是互斥的, 也就是说你只能使用其中的一种. 一个 try 语句可以对应一个或多个 except 子句, 但只能对应一个finally 子句, 或是一个 try-except-finally 复合语句。

另外值得说明的是，python中的try语句支持else语句，

接下来看几个例子：

1.try-except语句

	try:
    	x=int(input('input x:'))
    	y=int(input('input y:'))
    	print('x/y=%.2f'%(x/y))
	except ZeroDivisionError:
    	print('除数不能为0')

2.带有多个except的try语句  

	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)  
	except ZeroDivisionError: #捕捉除0异常  
    	print("ZeroDivision")  
	except TypeError as e: #捕捉类型异常并将异常对象输出  
    	print(e) 
	except ValueError as e: #捕捉值异常并将异常对象输出  
    	print(e) 
    

3.处理多个异常的except语句（异常必须放在一个元组里）

	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)   
	except (TypeError,ValueError,ZeroDivisionError) as e: #捕捉多个异常，并将异常对象输出  
    	print(e) 


4.try-except-else（在try范围中没有异常被检测到时，执行 else 子句）

	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)  
	except ZeroDivisionError as e: #捕捉除0异常  
    	print(e)  
	else: #当try块中的语句没有发生异常完全执行成功时执行
    	print('it work well')  

5.finally 语句（不管是否出现异常，最后都会执行finally的语句块内容，用于清理工作。所以，你可以在 finally 语句中关闭文件，这样就确保了文件能正常关闭）


	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)  
	except (TypeError,ValueError,ZeroDivisionError) as e: #捕捉多个异常  
    	print(e)  
	else: #没有异常时执行  
    	print('it work well')  
	finally: #不管是否有异常都会执行  
    	print("Cleaning up")  

6.捕获所有异常

	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)  
	except BaseException as e: #捕捉所有异常  
    	print(e)  

	try:  
    	x = int(input('input x:'))  
    	y = int(input('input y:'))  
    	print('x/y = ',x/y)  
	except: #捕捉所有异常 这种方法不推荐 
    	pass

为什么用BaseException而不是Exception已经在文章开头说过。

**说明：**

1. 在python2中将异常对象输出支持使用exception,e这种语法，而在python3中不在支持，只能将‘,’换成 as 关键字。
2. 当try语句中发生异常后，try语句中剩余的代码将不会在执行，当存在多个处理器（except字句）时，解释器将这一串解释器中查找匹配的异常，如果找到对应的处理器，执行流将会跳转到这里。
3. 在使用finally语句时，如果 finally 中的代码引发了另一个异常或由于 return,break,continue 语
法而终止,原来的异常将丢失而且无法重新引发。
4. 异常抛出之后，如果没有被接收，那么程序会抛给它的上一层，比如函数调用的地方，要是还是没有接收，那继续抛出，如果程序最后都没有处理这个异常，那它就丢给操作系统了 -- 你的程序崩溃了，这点和C++一样的。

# 主动触发异常

raise主动触发异常，我们可以使用raise语句自己触发异常

raise语法格式如下：

	raise [Exception [, args [, traceback]]]

语句中Exception是异常的类型（例如，NameError）参数是一个异常参数值。该参数是可选的，如果不提供，异常的参数是"None"。最后一个参数是可选的（在实践中很少使用），如果存在，是跟踪异常对象。

	try:
	    raise TypeError('类型错误')
	except TypeError as e:
	    print(e)
	
	结果：
	类型错误

# 断言

断言是一句必须等价于布尔真的判断，通过assert实现；断言语句如果成功不采取任何措施否则除法AssertionError(断言错误)的异常；用法如下：

	assert expression[,aeguments]

AssertionError异常和其他的异常一样可以用try-except语句块捕捉,但是如果没有捕捉,它将
终止程序运行而且提供一个如下的 traceback：

	>>> assert 1==0
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	AssertionError

使用try-except捕获AssertionError异常：

	>>> try:
	...     assert 1==0,'1!=0'
	... except AssertionError as args:
	...     print('%s:%s'%(args.__class__.__name__,args))
	...
	AssertionError:1!=0

# 自定义异常

自定义异常的前提是继承异常类，可以是基类异常（BaseException），也可以其他内建异常。

	>>> class myException(Exception):
	...     def __init__(self,msg):
	...         self.msg=msg
	...     def __str__(self):
	...         return self.msg
	...
	>>> try:
	...     raise myException('类型错误')
	... except myException as e:
	...     print(e)
	...
	类型错误

