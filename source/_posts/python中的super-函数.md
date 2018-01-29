---
title: python中的super()函数
date: 2018-01-23 14:50:41
tags: python面向对象
toc: true
categories: Python
---
super() 函数用于调用下一个父类(超类)并返回该父类的实例，主要用途是来查找父类的属性，比如 ，
super(MyClass,self).\_\_init\_\_()。它只能用在新式类中。

语法如下：
	
	super(type[, obj])
super()“返回此 type 的父类”。如果你希望父类被绑定,你可以传入 obj 参数(obj必须是 type 类型的)。否则父类不会被绑定。obj 参数也可以是一个类型，但它应当是 type 的一个子类。通常，当给出 obj 时：

1. 如果 obj 是一个实例，**isinstance(obj,type)**就必须返回 True
2. 如果 obj 是一个类或类型，**issubclass(obj,type)**就必须返回 True
<!--more-->

# super函数的作用

使用 super()的漂亮之处在于，你不需要明确给出任何基类名字... “跑腿事儿”，它帮你干了！

使用 super()的重点，是你不需要明确提供父类。这意味着如果你改变了类继承关系，你只需要改一
行代码（class 语句本身）而不必在大量代码中去查找所有被修改的那个类的名字。

在多重继承的情况下会涉及到查找顺序（**MRO**）、重复调用等种种问题。使用super()来避免这些问题。

**MRO** 就是类的方法解析顺序表, 其实也就是继承父类方法时的顺序表,下面会介绍到。

下面我们通过对比使用 super函数和基类类名调用父类方法来介绍super函数的作用：

<!--more-->

## 单继承

先看一个单继承的例子：

	class BaseCls(object):
	    def __init__(self,name):
	        print('Create BaseCls,name is %s'%name)
	        self.name=name
	
	class ChildA(BaseCls):
	    def __init__(self,name):
	        print('Create ChildA')
	        BaseCls.__init__(self,name)
	
	class ChildB(BaseCls):
	    def __init__(self,name):
	        print('Create ChildB')
	        super(ChildB,self).__init__(name)
	
	a=ChildA('_learner1')
	b=ChildB('_learner2')

运行结果：

	Create ChildA
	Create BaseCls,name is _learner1
	Create ChildB
	Create BaseCls,name is _learner2

**区别是使用super()继承时不用显式引用基类。避免类名发生变化时我们需要调整继承的代码。**

## 多继承

### 继承顺序问题

在多重继承时会涉及继承顺序，<font size=4 color=red>**super()相当于返回继承顺序的下一个类，而不是父类**</font>，类似于这样的功能：

	def super(class_name, self):  
	    mro = self.__class__.mro()  
	    return mro[mro.index(class_name) + 1]  

mro()用来获得类的继承顺序。 例如：

	class Base(object):
	    def __init__(self):
	        print('Base Create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        super(ChildA, self).__init__()
	        print ('leave A')
	class ChildB(Base):
	    def __init__(self):
	        print('enter B ' )
	        super(ChildB, self).__init__()
	        print ('leave B')
	class ChildC(ChildA, ChildB):
	    pass
	c = ChildC()
	print (c.__class__.__mro__ )

运行结果：

	enter A 
	enter B 
	Base create
	leave B
	leave A
	(<class '__main__.childC'>, <class '__main__.ChildA'>, <class '__main__.ChildB'>, <class '__main__.Base'>, <class 'object'>)

supder和父类没有关联，因此执行顺序是 A —> B—>—>Base

执行过程相当于：

1. 初始化childC()时，先会去调用childA的构造方法\_\_init\_\_,输出 enter A;
2. 然后执行super(childA, self).__init__()， super(ChildA, self)返回当前类的继承顺序中ChildA后的一个类childB;
3. 因此会进入到childB类执行器构造方法输出 enter B;
4. 然后执行 super(ChildB, self).__init__()，返回继承顺序的下一个类，即Base;
5. 执行Base类的\_\_init\_\_构造方法，输出 Base Create；
6. 然后继续执行，输出 leave B,最后就是 leave A。

在上面的例子中，如果把继承方式由super换成Base._init_(self)，会看到什么样的输出结果呢？

	class Base(object):
	    def __init__(self):
	        print('Base create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        Base.__init__(self)
	        print ('leave A')
	class ChildB(Base):
	    def __init__(self):
	        print('enter B ' )
	        Base.__init__(self)
	        print ('leave B')
	class childC(ChildA, ChildB):
	    pass
	c = childC()
	print (c.__class__.__mro__ )

运行结果：

	enter A 
	Base create
	leave A
	(<class '__main__.childC'>, <class '__main__.ChildA'>, <class '__main__.ChildB'>, <class '__main__.Base'>, <class 'object'>)

能看出，继承ChildA后就会直接跳到Base类里，而略过了ChildB。

### 重复调用问题

如果ChildA基础Base, ChildB继承childA和Base，如果ChildB需要调用Base的__init__()方法时，就会导致\_\_init\_\_()被执行两次：
	
	class Base(object):
	    def __init__(self):
	        print('Base create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        Base.__init__(self)
	        print('leave A')
	class ChildB(ChildA, Base):
	    def __init__(self):
	        print('enter B')
	        ChildA.__init__(self)
	        Base.__init__(self)
	b = ChildB()
	print (b.__class__.mro())
运行结果：

	enter B
	enter A 
	Base create
	leave A
	Base create
	[<class '__main__.ChildB'>, <class '__main__.ChildA'>, <class '__main__.Base'>, <class 'object'>]


使用super()函数可避免重复调用：

	class Base(object):
	    def __init__(self):
	        print('Base create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        super(ChildA, self).__init__()
	        print('leave A')
	class ChildB(ChildA, Base):
	    def __init__(self):
	        print('enter B')
	        super(ChildB,self).__init__()
	b = ChildB()
	print (b.__class__.mro())  
运行结果：

	enter B
	enter A 
	Base create
	leave A
	[<class '__main__.ChildB'>, <class '__main__.ChildA'>, <class '__main__.Base'>, <class 'object'>]

# super()函数使用注意点

从super()方法可以看出，super()的第一个参数可以是继承链中任意一个类的名字。

如果是本身就会依次继承下一个类；

如果是继承链里之前的类便会无限递归下去；

如果是继承链里之后的类便会忽略继承链汇总本身和传入类之间的类；

看例子:

将ChildA()中的super改为：super(ChildC, self).__init__()，程序就会无限递归下去:

	class Base(object):
	    def __init__(self):
	        print('Base Create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        super(ChildC, self).__init__()
	        print ('leave A')
	class ChildB(Base):
	    def __init__(self):
	        print('enter B ' )
	        super(ChildB, self).__init__()
	        print ('leave B')
	class ChildC(ChildA, ChildB):
	    pass
	c = ChildC()

运行结果：

	enter A 
	enter A 
	enter A 
	enter A 
	enter A Traceback (most recent call last):
	RecursionError: maximum recursion depth exceeded while calling a Python object

将ChildA()中的super改为：super(ChildB, self).__init__()，程序就会跳过ChildB:

	class Base(object):
	    def __init__(self):
	        print('Base Create')
	class ChildA(Base):
	    def __init__(self):
	        print('enter A ')
	        super(ChildB, self).__init__()
	        print ('leave A')
	class ChildB(Base):
	    def __init__(self):
	        print('enter B ' )
	        super(ChildB, self).__init__()
	        print ('leave B')
	class ChildC(ChildA, ChildB):
	    pass
	c = ChildC()

运行结果：
	
	enter A 
	Base Create
	leave A

另外，super函数只能应用在新式类中，用在经典类中会报错的：

	class A:
	    def __init__(self):
	        print('Create A')
	class B(A):
	    def __init__(self):
	        print('Create B')
	        super(B,self).__init__()
	
	b=B()

运行结果:

	Create B
	Traceback (most recent call last):
         super(B,self).__init__()
	TypeError: must be type, not classobj

# 总结

使用super()函数能规避掉多继承中U存在的各种问题，但要注意super函数的一个参数，一般都是当前类，一旦写错，会造成很严重的后果，另外super函数只能应用在新式类中，在涉及到调用父类方法时，推荐使用该函数。