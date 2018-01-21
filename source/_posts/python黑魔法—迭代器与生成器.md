---
title: python黑魔法—迭代器与生成器
date: 2017-11-28 17:44:06
tags: python基础
toc: true
categories: Python
---
在了解Python的数据结构时，容器(container)、可迭代对象(iterable)、迭代器(iterator)、生成器(generator)、列表/集合/字典推导式(list,set,dict comprehension)众多概念参杂在一起，难免让初学者一头雾水，我将用一篇文章试图将这些概念以及它们之间的关系捋清楚。
![](https://i.imgur.com/RLHjcHO.png)

<!--more-->

# 容器(container)

容器是一种把多个元素组织在一起的数据结构，容器中的元素可以逐个地迭代获取，可以用in, not in关键字判断元素是否包含在容器中。通常这类数据结构把所有的元素存储在内存中（也有一些特例，并不是所有的元素都放在内存，比如迭代器和生成器对象）在Python中，常见的容器对象有：

list, deque, ….

set, frozensets, ….

dict, defaultdict, OrderedDict, Counter, ….

tuple, namedtuple, …

str

容器比较容易理解，因为你就可以把它看作是一个盒子、一栋房子、一个柜子，里面可以塞任何东西。从技术角度来说，当它可以用来询问某个元素是否包含在其中时，那么这个对象就可以认为是一个容器，比如 序列，set，dic都是容器对象：

	>>> assert 1 in [1,2,3] #list
	>>> assert 4 not in [1,2,3]
	>>> assert 1 in {1,2,3} #set
	>>> assert 1 in (1,2,3) #tuple
	>>> assert 'a' in 'a,b,c' #str
	>>> assert 'key1' in {'key1':2,'key2':5} # dic

尽管绝大多数容器都提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是 可迭代对象 赋予了容器这种能力，当然并不是所有的容器都是可迭代的，比如（[Bloom filter](http://blog.csdn.net/wh_springer/article/details/52193110)）。

# 可迭代对象（Iterable）
刚才说过，很多容器都是可迭代对象，此外还有更多的对象同样也是可迭代对象，比如处于打开状态的files，sockets等等。但凡是可以返回一个 迭代器 的对象都可称之为可迭代对象。听起来可能有点困惑，没关系，可迭代对象与迭代器有一个非常重要的区别。先看一个例子：

	>>> x=[1,2,3]
	>>> y=iter(x)
	>>> next(y)
	1
	>>> type(x)
	<class 'list'>
	>>> type(y)
	<class 'list_iterator'>

x 是一个可迭代对象，可迭代对象和容器一样是一种通俗的叫法，并不是指某种具体的数据类型，list是可迭代对象，dict是可迭代对象，set也是可迭代对象。 y 是一个独立的迭代器，迭代器内部持有一个状态，该状态用于记录当前迭代所在的位置，以方便下次迭代的时候获取正确的元素。迭代器有一种具体的迭代器类型，比如 list\_iterator ， set\_iterator 。**可迭代对象实现了\_\_iter\_\_方法，该方法返回一个迭代器对象。**

判断一个对象是否是可迭代对象：

	>>> from collections import Iterable 
	>>> isinstance('abc', Iterable)     
	True
	>>> isinstance(1, Iterable)     
	False
	>>> isinstance([], Iterable)
	True

可迭代对象一般都用for循环遍历元素，也就是能用for循环的对象都可称为可迭代对象：

	>>> for i in [1,2,3]:
	...     pass
	...

执行上述代码时实际的调用过程如下：
![](https://i.imgur.com/HbRZYL4.png)

第一步：调用iter()方法将可迭代对象转换成迭代器

第二步：调用next()方法获取元素值。

第三步：处理StopIteration异常。（下面会讲到）

So,我们可以将上述代码理解成：

	list_iter=iter([1, 2, 3])
	while True:
    	try:
        	next(list_iter)
    	except StopIteration:
        	break


#迭代器（Iterator）

那么什么迭代器呢？它是一个带状态的对象，他能在你调用next()方法的时候返回容器中的下一个值，任何实现了\_\_iter\_\_和\_\_next\_\_()（python2中实现next()）方法的对象都是迭代器，\_\_iter\_\_返回迭代器自身，\_\_next\_\_返回容器中的下一个值，如果容器中没有更多元素了，则抛出**StopIteration**异常，至于它们到底是如何实现的这并不重要。

所以，迭代器就是实现了工厂模式的对象，它在你每次询问要下一个值的时候给你返回。有很多关于迭代器的例子，比如itertools函数返回的都是迭代器对象。

生成一个无线序列：

	>>> from itertools import count
	>>> counter = count(start=13)
	>>> next(counter)
	13
	>>> next(counter)
	14

以斐波那契数列()为例，学习为何创建以及如何创建一个迭代器：

著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：

1, 1, 2, 3, 5, 8, 13, 21, 34, ...

版本一：

	def fab(max): 
    	n, a, b = 0, 0, 1 
    	while n < max: 
        	print(b) 
        	a, b = b, a + b 
        	n = n + 1
	fab(5)

结果：

	1, 1, 2, 3, 5

直接在函数fab(max)中用print打印会导致函数的可复用性变差，因为fab返回None。其他函数无法获得fab函数返回的数列。

版本二：

	def fab(max):
    	L = [1,1]
    	n=2
    	while n < max:
        	L.append(L[-1]+L[-2])
        	n = n + 1
    	return L
	print(fab(5))
	
结果：

	[1, 1, 2, 3, 5]

版本二满足了可复用性的需求，但是占用了内存空间，最好不要。

对比python2中 

	for i in range(1000): 
		pass
和
	
	for i in xrange(1000): 
		pass，

前一个返回1000个元素的列表，而后一个在每次迭代中返回一个元素，因此可以使用迭代器来解决占空间的问题。

版本三：自定义迭代器实现斐波那契数列

	class Fab:
    	def __init__(self,max):
        	self.max=max
        	self.n,self.a,self.b=0,0,1
    	def __iter__(self):
        	return self
    	def __next__(self):
        	if self.n<self.max:
            	r=self.b
            	self.a,self.b=self.b,self.a+self.b
            	self.n+=1
            	return r
        	raise StopIteration

	fab=Fab(5)
	for i in fab:
    	print(i)

结果：

	1, 1, 2, 3, 5


使用内建函数调用上面的迭代器：
	
	fab=Fab(5)
	l=list(fab)
	print(type(l))
	for i in fab:
    	print(i)

上面代码的输出结果只有：

	<class 'list'>

为什么 for循环没有输出结果呢？原因是因为在对迭代器调用list、tuple等内建函数时会调用next()方法将当前迭代器对象的值全部算出来，迭代器已经计算完了，所以后面的for循环没有值输出。

Fab既是一个可迭代对象（因为它实现了 \_\_iter\_\_ 方法），又是一个迭代器（因为实现了 \_\_next\_\_ 方法）。实例变量 self .a 和 self.b 用于维护迭代器内部的状态。每次调用 next() 方法的时候做两件事：

1. 为下一次调用 next() 方法修改状态

2. 为当前这次调用生成返回结果

迭代器就像一个懒加载的工厂，等到有人需要的时候才给它生成值返回，没调用的时候就处于休眠状态等待下一次调用。

#生成器（generator）

生成器算得上是Python语言中最吸引人的特性之一，**生成器其实是一种特殊的迭代器**，不过这种迭代器更加优雅。上述版本3的代码远没有版本一的代码简洁，生成器（yield）既可以保持版本一的简洁性，又可以保持版本三的效果。它不需要再像上面的类一样写 \_\_iter\_\_() 和 \_\_next\_\_() 方法了，只需要一个 yiled 关键字。 生成器有如下特征使它一定是迭代器（反之不成立），因此任何生成器也是以一种懒加载的模式生成值。用生成器来实现斐波那契数列的例子是：

	def fab(max):
    	n,a,b=0,0,1
    	while n<max:
        	yield b
        	n+=1
        	a,b=b,a+b

	f=fab(5)
	for i in f:
    	print(i)

结果：

	1,1,2,3,5


fab 就是一个普通的python函数，它特别的地方在于函数体中没有 return 关键字，函数的返回值是一个生成器对象。当执行 f=fab(5) 返回的是一个生成器对象，此时函数体中的代码并不会执行，只有显示或隐示地调用next的时候才会真正执行里面的代码。

yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，可以称作生成器函数；Python 解释器会将其视为一个 generator，在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

也可以手动调用 fab(5) 的 next() 方法（因为 fab(5) 是一个 generator 对象，该对象具有 next() 方法），这样我们就可以更清楚地看到 fab 的执行流程：

	>>> f = fab(3)
	>>> f.__next__()
	1
	>>> f.__next__()
	1
	>>> f.__next__()
	2
	>>> f.__next__()
 
	Traceback (most recent call last):
	  File "<pyshell#62>", line 1, in <module>
        f.next()
	StopIteration

## 生成器中的return
在一个生成器中，如果没有return，则默认执行到函数完毕；如果遇到return,如果在执行过程中 return，则直接抛出 StopIteration 终止迭代。

	def fab():
    	print('yield 1')
    	yield 1
    	return
    	print('yield 2')
    	yield 2

	for i in fab():
    	print(i)

结果：

	yield 1
	1

for循环帮我们处理了StopIteration异常，要是自己调用两次next()方法就会抛出异常的。

## 生成器表达式

生成器表达式与列表解析表达式很相似，区别就是将‘[]’换成‘()’。

定义一个生成器表达式：

	>>> result = (x for x in range(5))
	>>> result
	<generator object <genexpr> at 0x0000022F66D2F4C0>
	>>> type(result)
	<class 'generator'>
	>>> next(result)
	0
	>>> for i in result:
	...     print(i)
	...
	1
	2
	3
	4

定义一个列表解析表达式：
	
	>>> result = [ x for x in range(5)]
	>>> type(result)
	<type 'list'>
	>>> result
	[0, 1, 2, 3, 4]

## 生成器的send()方法

send()方法和next()方法基本一样（send(None)等价于next()），都能启动生成器并获取yield 的返回值，它比next()方法多一个功能，可以给生成器传递值；需要注意的是，第一次启动生成器时，请使用next()语句或是send(None)，不能使用send发送一个非None的值，否则会出错的，因为没有 yield语句来接收这个值。

	def gen():
    	print('1111')
    	name=yield 1
    	print(name)
    	yield 2

	g=gen()
	#g.send(1) #TypeError: can't send non-None value to a just-started generator
	print(g.send(None))
	print(g.send('_learner'))

结果：

	1111
	1
	_learner
	2
	
# 总结

凡是可作用于for循环的对象都是可迭代对象(Iterable)类型。

凡是可作用于next()函数的对象都是迭代器(Iterator)类型，生成器是一个特殊的迭代器，它们表示一个惰性计算的序列。

序列、字典等是Iterable但不是Iterator，不过可以通过iter()函数获得一个Iterator对象。

Python的for循环本质上就是通过不断调用next()函数实现的。