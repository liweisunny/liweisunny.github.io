---
title: python函数参数总结
date: 2017-12-13 11:03:42
tags: python基础
toc: true
categories: python
---
定义函数的时候，我们把参数的名字和位置确定下来，函数的接口定义就完成了。对于函数的调用者来说，只需要知道如何传递正确的参数，以及函数将返回什么样的值就够了，函数内部的复杂逻辑被封装起来，调用者无需了解。

Python的函数定义非常简单，但灵活度却非常大。除了正常定义的必选参数外，还可以使用默认参数、可变参数和关键字参数，使得函数定义出来的接口，不但能处理复杂的参数，还可以简化调用者的代码。
<!--more-->
#位置参数
我们定义一个计算x*y的函数：

	def power(x,y):
    	return x*y

对于power(x)函数，参数x,y就是位置参数。

调用power函数时，必须传入两个参数x,y：

	>>> power(5,4)
	20
	>>> power(15,2)
	30

#默认参数

我们定义一个显示学生信息的函数:

	def showSudentInfo(name,age=20):
    	print('{}:{}'.format(name,age))

调用函数时必须传入name参数而age参数是可选的：

	>>> showSudentInfo('_learner')
	_learner:20
	>>> showSudentInfo('jack',18)
	jack:18

说明：大部分学生的年龄都是20岁，所以我们设置age参数的默认值是20，这样就简化了函数的调用，另外，设置默认参数需要注意几点：

1. 必选参数在前，默认参数在后，否则Python的解释器会报错（思考一下为什么默认参数不能放在必选参数前面）；

2. 如何设置默认参数。
	1. 当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数。
	2. 使用默认参数有什么好处？最大的好处是能降低调用函数的难度。

<font color=red>但是，默认参数使用不当会掉坑里，最大的坑，看下面这个列子：</font>

	def func(li=[]):
    	li.append('end')
    	return li
调用函数：

	>>> func([1,2,3])
	[1, 2, 3, 'end']
	>>> func()
	['end']
	>>> func()
    ['end', 'end']
	>>> func()
	['end', 'end','end']

由结果可知，前两次调用一切正常，当连续使用默认参数的形式调用的时候结果不对了，按我们理解的后两次func()调用 输出的结果应该也是['end']，为啥不是呢？

原因是这样的：python函数在定义的时候，默认参数L的值就被计算出来了，即[]，[]所占用的内存地址也不会变了，除非重新赋值，因为[]是一个可变对象，每次给他li.append('end')添加值，只是改变了其值，其所指向的对象地址没有变。所以多次调用func()操作的是同一个li,所以就出现了上述情况。

修改一下这个例子,输出id值：

	def func(li=[]):
    	li.append('end')
        print(id(li))
    	return li
调用函数：

	>>> func([1,2,3])
	1497991838664
	[1, 2, 3, 'end']
	>>> func()
	1497991837896
	['end']
	>>> func()
	1497991837896
    ['end', 'end']
	>>> func()
	1497991837896
	['end', 'end','end']

能看出，后面三次调用操作的是同一个对象li。

<font color=red>定义默认参数要牢记一点：默认参数必须指向不变对象！</font>

上述列子的改进方案：

	def func(li=None):
    	if not li:
        	li=[]
    	li.append('end')
    	return li

调用函数：

	>>> func([1,2,3])
	[1, 2, 3, 'end']
	>>> func()
	['end']
	>>> func()
	['end', 'end']
	>>> func()
	['end', 'end','end']

#可变参数

顾名思义，可变参数就是传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个。

定义可变参数在参数前面加了一个*号。在函数内部，参数numbers接收到的是一个tuple。

	def func(*numbers):
    	sum=0
    	for num in numbers:
        	sum=sum+num
    	return sum

调用函数：

	>>> func(1,2,3)
	6
	>>> tup=(1,3,5,8)
	>>> func(*tup)
	17
	>>> li=[1,3,5,8]
	>>> func(*li)
	17

#关键字参数

可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。请看示例：

	def person(name, age, **kw):
    	print('name:', name, 'age:', age, 'other:', kw)

函数person除了必选参数name和age外，还接受关键字参数kw。在调用该函数时，可以只传入必选参数：

	>>> person('Michael', 30)
	name: Michael age: 30 other: {}

也可以传入任意个数的关键字参数：

	>>> person('Bob', 35, city='Beijing')
	name: Bob age: 35 other: {'city': 'Beijing'}
	>>> person('Adam', 45, gender='M', job='Engineer')
	name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
关键字参数有什么用？它可以扩展函数的功能。比如，在person函数里，我们保证能接收到name和age这两个参数，但是，如果调用者愿意提供更多的参数，我们也能收到。试想你正在做一个用户注册的功能，除了用户名和年龄是必填项外，其他都是可选项，利用关键字参数来定义这个函数就能满足注册的需求。

和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：

	>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
	>>> person('Jack', 24, **extra)
	name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}

<font color=red>\*\*extra表示把extra这个dict的所有key-value用关键字参数传入到函数的\*\*kw参数，kw将获得一个dict，注意kw获得的dict是extra的一份拷贝，对kw的改动不会影响到函数外的extra。</font>

#命名关键字参数

如果要限制关键字参数的名字，就可以用命名关键字参数，例如，只接收city和job作为关键字参数。这种方式定义的函数如下：

	def person(name, age, *, city, job):
    	print(name, age, city, job)
	
和关键字参数**kw不同，命名关键字参数需要一个特殊分隔符*，*后面的参数被视为命名关键字参数。

调用方式如下：

	>>> person('Jack', 24, city='Beijing', job='Engineer')
	Jack 24 Beijing Engineer

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了：

	def person(name, age, *args, city, job):
    	print(name, age, args, city, job)

调用方式如下：

	>>> person('_learner', 23, '酷爱Python',city='北京',job='程序员')
	_learner 23 ('酷爱Python',) 北京 程序员

命名关键字参数必须传入参数名，这和位置参数不同。如果没有传入参数名，调用将报错：

	>>> person('Jack', 24, 'Beijing', 'Engineer')
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	TypeError: person() takes 2 positional arguments but 4 were given

由于调用时缺少参数名city和job，Python解释器把这4个参数均视为位置参数，但person()函数仅接受2个位置参数。

命名关键字参数可以有缺省值，从而简化调用：

	def person(name, age, *, city='Beijing', job):
    	print(name, age, city, job)
由于命名关键字参数city具有默认值，调用时，可不传入city参数：

	>>> person('Jack', 24, job='Engineer')
	Jack 24 Beijing Engineer

# 参数组合

在Python中定义函数，可以用必选(位置)参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选(位置)参数、默认参数、可变参数、命名关键字参数和关键字参数。

比如定义一个函数，包含上述若干种参数：

	def f1(a, b, c=0, *args, **kw):
    	print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

	def f2(a, b, c=0, *, d, **kw):
    	print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
在函数调用的时候，Python解释器自动按照参数位置和参数名把对应的参数传进去。

	>>> f1(1, 2)
	a = 1 b = 2 c = 0 args = () kw = {}
	>>> f1(1, 2, c=3)
	a = 1 b = 2 c = 3 args = () kw = {}
	>>> f1(1, 2, 3, 'a', 'b')
	a = 1 b = 2 c = 3 args = ('a', 'b') kw = {}
	>>> f1(1, 2, 3, 'a', 'b', x=99)
	a = 1 b = 2 c = 3 args = ('a', 'b') kw = {'x': 99}
	>>> f2(1, 2, d=99, ext=None)
	a = 1 b = 2 c = 0 d = 99 kw = {'ext': None}
最神奇的是通过一个tuple和dict，你也可以调用上述函数：

	>>> args = (1, 2, 3, 4)
	>>> kw = {'d': 99, 'x': '#'}
	>>> f1(*args, **kw)
	a = 1 b = 2 c = 3 args = (4,) kw = {'d': 99, 'x': '#'}
	>>> args = (1, 2, 3)
	>>> kw = {'d': 88, 'x': '#'}
	>>> f2(*args, **kw)
	a = 1 b = 2 c = 3 d = 88 kw = {'x': '#'}
<font color=red>所以，对于任意函数，都可以通过类似func(*args, **kw)的形式调用它，无论它的参数是如何定义的。
虽然可以组合多达5种参数，但不要同时使用太多的组合，否则函数接口的可理解性很差。</font>