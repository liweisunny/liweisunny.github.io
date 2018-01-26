---
title: python面向对象初识-新式类VS经典类
date: 2018-01-10 18:15:58
tags: python面向对象
toc: true
categories: Python
---
在python2.x中，从object继承得来的类称为新式类（如class A(object)）不从object继承得来的类称为经典类（如class A()）；python3中所有的类都为新式类。

那么新式类和经典类有啥区别呢？ 接下来我们详细介绍一下。
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

新式类增加了\_\_slots\_\_属性，具体请参考：

<font size=4>**\_\_getattribute\_\_方法**</font>

新式类增加了\_\_getattribute\_\_方法，具体请参考：

<font size=4>**\_\_new\_\_方法**</font>

新式类内置有\_\_new\_\_方法而经典类没有\_\_new\_\_方法而只有\_\_init\_\_方法

在python中创建类的一个实例时，如果该类具有\_\_new\_\_方法，会先调用\_\_new\_\_方法，\_\_new\_\_方法接受当前正在实例化的类作为第一个参数（这个参数的类型是type，这个类型在c和python的交互编程中具有重要的角色，感兴趣的可以搜下相关的资料），其返回值是本次创建产生的实例，也就是我们熟知的__init__方法中的第一个参数self。那么就会有一个问题，这个实例怎么得到？

注意到有\_\_new\_\_方法的都是object类的后代，因此如果我们自己想要改写\_\_new\_\_方法（注意不改写时在创建实例的时候使用的是父类的\_\_new\_\_方法，如果父类没有则继续上溯）可以通过调用object的\_\_new\_\_方法类得到这个实例（这实际上也和python中的默认机制基本一致），如：

	class display(object):
		def __init__(self, *args, **kwargs):
    		print("init")
		def __new__(cls, *args, **kwargs):
    		print("new")
    		print(type(cls))
    		return object.__new__(cls, *args, **kwargs)  
	a=display()

结果：

	new
	<class 'type'>
	init


因此我们可以得到如下结论：

**在实例创建过程中\_\_new\_\_方法先于\_\_init\_\_方法被调用，它的第一个参数类型为type,f返回值是当前实例对象。**

**如果不需要其它特殊的处理，可以使用object的\_\_new\_\_方法来得到创建的实例（也即self)。**

**于是我们可以发现，实际上可以使用其它类的\_\_new\_\_方法类得到这个实例，只要那个类或其父类或祖先有\_\_new\_\_方法。**

	class another(object):
		def __new__(cls,*args,**kwargs):
    		print("newano")
    		return object.__new__(cls, *args, **kwargs)  
	class display(object):
		def __init__(self, *args, **kwargs):
    		print("init")
		def __new__(cls, *args, **kwargs):
    		print("newdis")
    		print(type(cls))
    		return another.__new__(cls, *args, **kwargs)  
	a=display()

结果：

	newdis
	<class 'type'>
	newano
	init

我们发现\_\_new\_\_和\_\_init\_\_就像这么一个关系，\_\_new\_\_提供生产的原料self(但并不保证这个原料来源正宗，像上面那样它用的是另一个不相关的类的__new__方法类得到这个实例)，而\_\_init\_\_就用\_\_new\_\_给的原料来完善这个对象（尽管它不知道这些原料是不是正宗的）。