---
title: python—函数式编程(匿名函数、高阶函数、偏函数)
date: 2017-12-15 11:03:42
tags: python基础
toc: true
categories: Python
---
函数式编程是使用一系列函数去解决问题，按照一般编程思维，面对问题时我们的思考方式是“怎么干”，而函数函数式编程的思考方式是我要“干什么”。 

Python 不是也不大可能会成为一种函数式编程语言，但是它支持许多有价值的函数式编程语言构建。
<!--more-->

#lambda表达式(匿名函数)

语法：

	lambda [arg1[,arg2,....argN]]: expression

定义一个计算两个数乘积的函数,并返回结果值：

普通函数定义方式：

	def funcTake(a,b):
    	return a*b
	>>> funcTake(2,3)
	6

匿名函数定义方式：

	>>> funcTake=lambda a,b:a*b #定义一个匿名函数并将函数赋值给变量funcTake
	>>> funcTake(2,4)
	8

匿名函数和普通函数一样，可以不带参数，也支持默认参数，可变参数，关键字参数等。

	>>> func=lambda:True  #无参数
	>>> func
	<function <lambda> at 0x000001CC2BAEA510>
	>>> func()
	True
	>>> func=lambda a,b=5:a*b #默认参数
	>>> func(2)
	10
	>>> func=lambda a,*args,**kw:[ a* i for i in args if i in kw.values()] #可变参数和关键字参数
	>>> func(3,2,3,num=2,num2=5)
	[6]

#高阶函数

1. 函数名可以作为函数的参数输入的函数是高阶函数。

2. 函数名可以作为返回值的函数也是高阶函数。

满足上述任何一点均是高阶函数。

	def funcA(a,b):
		return a+b
	fun=funcA #将函数赋值给变量，其实就是给函数funcA()增加了一个引用。
	>>> fun(2,3)
	5

	def funcB(num1,num2,func):#将函数作为参数传入
    	return func(num1,num2)
	>>> funcB(2,3,funcA)
	5

下面介绍四个内置的高阶函数。

## filter()
语法：

	filter(func,seq)

调用一个布尔函数 func 来迭代遍历每个 seq 中的元素； p返回一个使 func 返回值为 ture 的元素构成的filter对象。python2中返回的是一个过滤后的序列。

用代码的形式阐述filter函数的实现机制：

	def filter(bool_func,seq):
		filtered_seq=[]
		for item in seq:
			if bool_func(item):
				filtered_seq.append(item)
		return filtered_seq

下面我们用filter()函数过滤掉1~10以内的所有偶数：

	>>> def odd(num):
	...     return num%2
	...
	>>> result=filter(odd,range(1,11))
	>>> result
	<filter object at 0x000001CC2BB1A390>
	>>> list(result)
	[1, 3, 5, 7, 9]

上面的例子可以用lambda表达式一句话搞定：

	>>> result=filter(lambda num :num%2,range(1,11))
	>>> result
	<filter object at 0x0000011EE27AF9E8>
	>>> list(result)
	[1, 3, 5, 7, 9]

## map()

语法：

	map(func, seq1[,seq2...])

将函数 func 作用于给定序列（s)的每个元素，并用一个列表来提供返回值；python2中如果 func 为 None， func 表现为一个身份函数，返回一个含有每个序列中元素集合的 n 个元组的列表。

<font color=red>python2中返回一个新的列表，**注意是列表不是序列** ，而python3中返回的是一个map对象。</font>

用代码的形式阐述map函数的实现机制：

	def map(func,seq):
		mapped_seq=[]
		for item in seq:
			mapped_seq.append(func(item))
		return mapped_seq

看个简单的例子：

	>>> result=map((lambda x: x+2), [0, 1, 2, 3, 4, 5])
	>>> result
	<map object at 0x0000011EE27DA438>
	>>> list(result)
	[2, 3, 4, 5, 6, 7]

把列表所有数字转为字符串：

	>>> result=map(str,[1,2,3,4,5,6])
	>>> list(result)
	['1', '2', '3', '4', '5', '6']

map()函数作用于多个序列：

	>>> result=map(lambda x,y:x+y,[1,3,5],[2,4,6])
	>>> list(result)
	[3, 7, 11]

	>>> result=map(lambda x,y:(x+y,x-y),[1,3,5],[2,4,6])
	>>> list(result)
	[(3, -1), (7, -1), (11, -1)]

另外，python2中map()函数支持None参数

	>>> result=map(None,[1,3,5],[2,4,6])  #结果等价于zip()函数
	>>> result
	[(1, 2), (3, 4), (5, 6)]
python3中不在支持None参数：

	>>> result=map(None,[1,3,5],[2,4,6])
	>>> list(result)
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	TypeError: 'NoneType' object is not callable

## reduce()

语法:

	reduce(func, seq[, init])

它通过取出序列的头两个元素，将他们传入二元函数来获得一个单一的值来实现。然后又用这
个值和序列的下一个元素来获得又一个值，然后继续直到整个序列的内容都遍历完毕以及最后的值
会被计算出来为止；如果初始值 init 给定，第一个比较会是 init 和第一个序列元素而不是序列的头两个元素。

<font color=red>在Python 3里,reduce()函数已经被从全局名字空间里移除了，它现在被放置在fucntools模块里 用的话要先引入。</font>

用代码的形式阐述reduce函数的实现机制：

	def reduce(bin_func,seq,init=None):
		Iseq=list(seq)
		if init is None:
			res=lseq.pop(0) #获取第一个元素并移除。
		else:
			res=init
		for item in Iseq:
			res=bin_func(res,item)
		return res


简单的例子，计算1~100的和：

	>>> from functools import reduce
	>>> result=reduce(lambda x,y:x+y,range(1,101))
	>>> result
	5050
	>>> result=reduce(lambda x,y:x+y,range(1,101),1)
	>>> result
	5051

## sorted()

sorted()也是一个高阶函数。用sorted()排序的关键在于实现一个映射函数。

python3中用法如下：

语法:

	sorted(iterable[, key][, reverse]) 

sorted的第一个参数是一个迭代器，第二个参数是用来排序的key，第三个参数的排序顺序：正序还是倒序。

第一个参数是迭代器：

	>>> sorted([36, 5, -12, 9, -21])
	[-21, -12, 5, 9, 36]

第二个参数是用来排序的key：

	>>> sorted([36, 5, -12, 9, -21], key=abs)
	[5, 9, -12, -21, 36]

key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序。

第三个参数决定正向还是反向排序，要进行反向排序，可以传入第三个参数reverse=True：

	>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
	['Zoo', 'Credit', 'bob', 'about'] #忽略大小写反序排列


python2中的用法如下：

语法：

	sorted(iterable[,cmp][, key][, reverse]) #多了一个cmp参数。

看一个例子:

	def reversed_cmp(x, y):
    	if x > y:
        	return -1
    	if x < y:
        	return 1
    	return 0
	>>> sorted([2,5,6,2,1,3],reversed_cmp) #倒序排
	[6,5,3,2,2,1]

其他参数与python3中的用法一致。


# 偏函数(currying)

Python的functools模块提供了很多有用的功能，其中一个就是偏函数（Partial function）。要注意，这里的偏函数和数学意义上的偏函数不一样。

在介绍函数参数的时候，我们讲到，通过设定参数的默认值，可以降低函数调用的难度。而偏函数也可以做到这一点。举例如下：

int()函数可以把字符串转换为整数，当仅传入字符串时，int()函数默认按十进制转换：

	>>> int('12345')
	12345
但int()函数还提供额外的base参数，默认值为10。如果传入base参数，就可以做N进制的转换：

	>>> int('12345', base=8)
	5349
	>>> int('12345', 16)
	74565
假设要转换大量的二进制字符串，每次都传入int(x, base=2)非常麻烦，于是，我们想到，可以定义一个int2()的函数，默认把base=2传进去：

	def int2(x, base=2):
    	return int(x, base)
这样，我们转换二进制就非常方便了：

	>>> int2('1000000')
	64
	>>> int2('1010101')
	85
functools.partial就是帮助我们创建一个偏函数的，不需要我们自己定义int2()，可以直接使用下面的代码创建一个新的函数int2：

	>>> import functools
	>>> int2 = functools.partial(int, base=2)
	>>> int2('1000000')
	64
	>>> int2('1010101')
	85

所以，简单总结functools.partial的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

注意到上面的新的int2函数，仅仅是把base参数重新设定默认值为2，但也可以在函数调用时传入其他值：

	>>> int2('1000000', base=10)
	1000000
最后，创建偏函数时，实际上可以接收函数对象、*args和**kw这3个参数，当传入：

	>>> int2 = functools.partial(int, base=2)
实际上固定了int()函数的关键字参数base，也就是：

	>>> int2('10010')
	18
相当于：

	>>> kw = { 'base': 2 }
	>>> int('10010', **kw)
	18
当传入：

	>>> max2 = functools.partial(max, 10)
实际上会把10作为*args的一部分自动加到左边，也就是：

	>>> max2(5, 6, 7)
	10
相当于：

	>>> args = (10, 5, 6, 7)
	>>> max(*args)
	10

小结

当函数的参数个数太多，需要简化时，使用functools.partial可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。


# 总结

最后我们可以看到，函数式编程有如下好处：

1. 代码更简单了。
2. 数据集，操作，返回值都放到了一起。
3. 你在读代码的时候，没有了循环体，于是就可以少了些临时变量，以及变量倒来倒去逻辑。
4. 你的代码变成了在描述你要干什么，而不是怎么去干。