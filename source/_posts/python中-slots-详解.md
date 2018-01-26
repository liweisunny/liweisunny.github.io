---
title: python中 \_\_slots\_\_ 详解
date: 2018-01-15 08:26:28
tags: python面向对象
toc: true
categories: Python
---

新式类增加了\_\_slots\_\_内置属性, 它的作用是阻止在实例化类时为实例分配dict，可以把实例属性的种类锁定到\_\_slots\_\_规定的范围之中。

举个例子：

	class A(object):
    	attr=1
    	def print_info(self):
        	print('A')

	a=A()
	print(a.__dict__)#{}
	a.num=1
	print(a.__dict__)#{'num': 1}

结果：

	{}
	{'num':1}

可见：实例的\_\_dict\_\_只保持实例的变量，对于类的属性是不保存的，类的属性包括变量和函数。由于每次实例化一个类都要分配一个新的\_\_dict\_\_，存在空间的浪费，因此有了\_\_slots\_\_。

\_\_slots\_\_是一个元组，包括了当前能访问到的属性。当定义了\_\_slots\_\_后，\_\_slots\_\_中定义的变量变成了类的描述符，相当于java，c++中的成员变量声明。

类的实例只能拥有\_\_slots\_\_中定义的变量，不能再增加新的变量。注意：定义了slots后，就不再有dict。如下：

	class A(object):
    	__slots__=('x')
    	attr=1
    	def print_info(self):
        	print('A')

	a=A()
	print(a.__dict__) # AttributeError: 'A' object has no attribute '__dict__'
	a.num=1 # AttributeError: 'A' object has no attribute 'num'
	a.x=10
	print(a.x) # 10

如果类变量与slots中的变量同名，则该变量被设置为readonly！！！如下：

	class base(object):  
    __slots__=('y')  
    y=22 # y是类变量,y与__slots__中的变量同名  
    var=11  
    def __init__(self):  
        pass  
      
	b=base()  
	print （b.y）  
	print （base.y）  
	#b.y=66 #AttributeError: 'base' object attribute 'y' is read-only 
 
结果：

	22\_\_new\_\_
	22

