---
title: python 类属性和实例属性
date: 2018-01-20 11:46:54
tags: python面向对象
toc: true
categories: Python
---

在介绍类属性和实例属性区别之前我们先来了解一下属性是什么？什么是类属性？什么是实例属性？类属性和实例属性的是如何查找访问的？

**（本篇博文实例代码均是在python3下的执行结果，python2下略有不同。）**
<!--more-->
# 属性

什么是属性呢？属性就是属于另一个对象的数据或者函数元素,可以通过我们熟悉的句点属性标识法来访问。一些 Python 类型比如复数有数据属性（实部和虚部），而另外一些，像列表和字典，拥有方法（函数属性）。

上面的定义听起来太抽象了，下面还是整点实在的吧！

## 类属性

	class A:
		attr=1

上面就定义的 attr就是一个类属性，类属性是和类直接绑定的，可以通过类直接访问，也可以通过类的实例访问。

	>>> A.attr
	1

## 实例属性

	>>> a=A()
	>>> a.x=10
	10
上述代码定义了一个实例a然后动态的给实例a添加了一个属性x。

## 实例属性和类属性的访问方式

在介绍如何访问属性之前我们先介绍一个特殊的属性\_\_dict\_\_，每一个类及其每一个实例均有自己的\_\_dict\_\_属性，这个属性就是用来存储类及其实列的属性，方法等的。还有一个内建函数dir()也是用来查看对象包含的属性，方法等的，下面让我们来看一下：

	>>> class A:
	...     attr=1
	...     def showInfo():
	...         print('my name is showInfo')
	...
	>>> A.__dict__
	mappingproxy({'__module__': '__main__', 'attr': 1, 'showInfo': <function A.showInfo at 0x0000020F36E5A510>, '__dict__': <attribute '__dict__' of 'A' objects>, '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None})
	>>> dir(A)
	['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'attr', 'showInfo']
	>>> a=A()
	>>> a.__dict__
	{}
	>>> a.x=10
	>>> a.__dict__
	{'x': 10}
	>>> dir(a)
	['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'attr', 'showInfo', 'x']
	>>> a2=A()
	>>> a2.__dict__
	{}
	>>> a2.y=20
	>>> a2.__dict__
	{'y': 20}
	>>> dir(a2)
	['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'attr', 'showInfo', 'y']
	>>>
dir()返回的是对象的属性的一个名字列表，它包含了一些内置的属性和方法名，而\_\_dict\_\_返回的是一个字典(可以把 **mappingproxy**理解成是字典)，它的键(keys)是属性名，键值(values)是相应的属性对象的数据值。另外，类的实例的\_\_dict\_\_只包含实例自己的属性和方法，默认是一个空字典。

当访问一个类属性的时候，Python 解释器将会搜索字典以得到需要的属性。如果在\_\_dict\_\_中没有找到，将会在基类的字典中进行搜索，采用“深度优先搜索”顺序。基类集的搜索是按顺序的，从左到右，按其在类定义时，定义父类参数时的顺序。对类的修改会仅影响到此类的字典；基类的\_\_dict\_\_属性不会被改动的。

当访问实例属性的时候，和访问类属性差不多，先在实例的字典中查找，如果\_\_dict\_\_中找不到再去类的字典找那个查找，然后就是类的基类里面去找。

如果要访问的属性最终没有找到就会抛出 **AttributeError** 异常。

## 通过实例访问类属性的注意点

	class Student:
	    name='_learner'
	
	s1=Student()
	s2=Student()
	print('Student:{n}，s1:{n1}，s2:{n2}'.format(n=Student.name,n1=s1.name,n2=s2.name))
	print(s1.__dict__)
	s1.name='Wai'
	print(s1.__dict__)

	print('Student:{n}，s1:{n1}，s2:{n2}'.format(n=Student.name,n1=s1.name,n2=s2.name))

	Student.name='Sunny'
	print('Student:{n}，s1:{n1}，s2:{n2}'.format(n=Student.name,n1=s1.name,n2=s2.name))
	
	del s1.name
	print('Student:{n}，s1:{n1}，s2:{n2}'.format(n=Student.name,n1=s1.name,n2=s2.name))
	print(s1.__dict__)

运行结果：

	Student:_learner，s1:_learner，s2:_learner
	{}
	{'name': 'Wai'}

	Student:_learner，s1:Wai，s2:_learner

	Student:Sunny，s1:Wai，s2:Sunny

	Student:Sunny，s1:Sunny，s2:Sunny
	{}
从上面的例子可以看出，在编写程序的时候，千万不要对实例属性和类属性使用相同的名字，因为相同名称的实例属性将屏蔽掉类属性，但是当你删除实例属性后，再使用相同的名称，访问到的将是类属性。

核心提示：使用类属性来修改自身（不是实例属性）正如上面所看到的那样，使用实例属性来试着修改类属性是很危险的。原因在于实例拥有它们自已的属性集\_\_dict\_\_。

# 总结
python中类及实例的属性访问是通过字典\_\_dict\_\_实现的，另外，切记不要定义同名的类属性和实例属性，这样使用实例访问是会覆盖掉类属性，当修改类属性值时使用类修改而不是用实例修改。






	



