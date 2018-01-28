---
title: python面向对象初识-新式类VS经典类
date: 2018-01-10 18:15:58
tags: python面向对象
toc: true
categories: Python
---
在python2.x中，从object继承得来的类称为新式类（如class A(object)）不从object继承得来的类称为经典类（如class A()）；python3中所有的类都为新式类。

那么新式类和经典类有啥区别呢？ 主要区别体现在五个方面，接下来我们详细介绍一下。
<!--more-->

<font size=4>**\_\_class\_\_属性**</font>

新式类对象有\_\_class\_\_属性， 可以直接通过\_\_class\_\_属性获取自身类型:type;经典类没有这个属性。


经典类：

	>>> class A:
	...     pass
	...
	>>> A.__class__
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	AttributeError: class A has no attribute '__class__'
	>>> type(A)
	<type 'classobj'>
	>>> a=A()
	>>> a.__class__
	<class __main__.A at 0x00000000006277C8>
	>>> type(a)
	<type 'instance'>
	>>> a.__class__ is type(a)
	False

新式类：

	>>> class B(object):
	...     pass
	...
	>>> B.__class__
	<type 'type'>
	>>> type(B)
	<type 'type'>
	>>> B.__class__ is type(B)
	True
	>>> b=B()
	>>> b.__class__
	<class '__main__.B'>
	>>> type(b)
	<class '__main__.B'>
	>>> b.__class__ is type(b)
	True

能看出，新式类及其实例均有\_\_class\_\_属性，并且通过内建函数type()获取到的值与\_\_class\_\_属性获取到的值一致；经典类没有\_\_class\_\_属性，但其实例有，但是通过内建函数type()获取到的值与\_\_class\_\_属性获取到的值不一致。


<font size=4>**多继承属性搜索顺序**</font>

经典类多继承时属性搜索顺序: 先深入继承树左侧，再返回，开始找右侧（即深度优先搜索）;

新式类多继承属性搜索顺序: 先水平搜索，然后再向上移动（即宽度优先）。

例子：

经典类(深度优先) attr查找顺序：D、B、A、C

	class A:
    	attr=1
	class B(A):
    	pass
	class C(A):
    	attr = 2
	class D(B,C):
    	pass
	d=D()
	print(d.attr)

结果：

	1

新式类(宽度优先) attr查找顺序：D、B、C、A

	class A(object):
    	attr=1
	class B(A):
    	pass
	class C(A):
    	attr = 2
	class D(B,C):
    	pass
	d=D()
	print(d.attr)

结果：

	2

<font size=4>**\_\_slots\_\_属性**</font>

新式类增加了\_\_slots\_\_属性，具体请参考：[http://waisunny.com/2018/01/15/python%E4%B8%AD-slots-%E8%AF%A6%E8%A7%A3/](http://waisunny.com/2018/01/15/python%E4%B8%AD-slots-%E8%AF%A6%E8%A7%A3/ "python中 __slots__属性详解")

<font size=4>**\_\_getattribute\_\_方法**</font>

新式类增加了\_\_getattribute\_\_方法，具体请参考：[http://waisunny.com/2018/01/12/python%E4%B8%AD-get-getattr-getattribute-%E8%AF%A6%E8%A7%A3/](http://waisunny.com/2018/01/12/python%E4%B8%AD-get-getattr-getattribute-%E8%AF%A6%E8%A7%A3/ "python中__get__、__getattr__、__getattribute__详解")

<font size=4>**\_\_new\_\_方法**</font>

新式类增加了\_\_new\_\_方法，具体请参考：[http://waisunny.com/2018/01/18/python%E4%B8%AD-new-VS-init/](http://waisunny.com/2018/01/18/python%E4%B8%AD-new-VS-init/ "python中 __new__和__init__的区别")

