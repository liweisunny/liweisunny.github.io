---
title: python中 __new__和__init__的区别
date: 2018-01-18 09:14:27
tags: python面向对象
toc: true
categories: Python
---
新式类内置有\_\_new\_\_方法而经典类没有\_\_new\_\_方法，只有\_\_init\_\_方法。

在python中创建类的一个实例时，如果该类具有\_\_new\_\_方法，会先调用\_\_new\_\_方法，\_\_new\_\_方法接受当前正在实例化的类作为第一个参数（这个参数的类型是type，这个类型在c和python的交互编程中具有重要的角色，感兴趣的可以搜下相关的资料），其返回值是本次创建产生的实例，也就是我们熟知的\_\_init\_\_方法中的第一个参数self。那么就会有一个问题，这个实例怎么得到？
<!--more-->

有\_\_new\_\_方法的都是object类(新式类)的后代，因此如果我们自己想要改写\_\_new\_\_方法（注意不改写时在创建实例的时候使用的是父类的\_\_new\_\_方法，如果父类没有则继续上溯）可以通过调用object的\_\_new\_\_方法类得到这个实例（这实际上也和python中的默认机制基本一致），如：

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

**在实例创建过程中\_\_new\_\_方法先于\_\_init\_\_方法被调用，它的第一个参数类型为type,返回值是当前实例对象。**

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

<font size=4>**总结**</font>

我们发现\_\_new\_\_和\_\_init\_\_就像这么一个关系，\_\_new\_\_提供生产的原料self(但并不保证这个原料来源正宗，像上面那样它用的是另一个不相关的类的__new__方法类得到这个实例)，而\_\_init\_\_就用\_\_new\_\_给的原料来完善这个对象（尽管它不知道这些原料是不是正宗的）。