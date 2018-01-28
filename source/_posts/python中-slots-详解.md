---
title: python中 __slots__属性详解
date: 2018-01-15 08:26:28
tags: python面向对象
toc: true
categories: Python
---
# 引言
新式类增加了\_\_slots\_\_内置属性, 它的作用是把实例属性锁定到\_\_slots\_\_规定的范围内。

\_\_slots\_\_是一个元组，包括了当前能访问到的属性。当定义了slots后，slots中定义的变量变成了类的描述符，相当于java，c++中的成员变量声明，类的实例只能拥有slots中定义的变量。

看下面的例子：
<!--more-->

例一：

	class A(object):
    	attr=1
	a=A()
	print(a.__dict__)
	a.x=10
	print(a.x)
	print(a.__dict__)

结果：

	{}
	10
	{'x': 10}

例二：

	class A(object):
    	attr=1
    	__slots__=('x','y')
	print(a.x)  # AttributeError: x
	a=A()
	#a.__dict__ # AttributeError: 'A' object has no attribute '__dict__'
	a.x=10
	#a.z=20     # AttributeError: 'A' object has no attribute 'z'
	print(a.x)
结果：

	10
通过上面的例子我们能看出，没有定义\_\_slots\_\_的类的实例包含\_\_dict\_\_属性，它是一个字典，默认为空，可以通过该属性动态添加属性，方法等。

但当定义了\_\_slots\_\_属性后，\_\_dict\_\_属性就没了，因此不能动态添加属性了，只能使用\_\_slots\_\_元组中规定的属性，前提是要先赋值。

Python是一门动态语言，可以在运行过程中，修改实例的属性和增删方法，使用\_\_slots\_\_属性后失去了这个特性；那么\_\_slots\_\_属性有什么用呢？继续往下面看：

# Slots的实现

我们首先来看看用纯Python是如何实现\_\_slots\\__（为了将以下实现的slots与原slots区分开来，代码中用单下划线的\_slots\_来代替)


	class Member(object):
	    # 定义描述器实现slots属性的查找
	    def __init__(self, i):
	        self.i = i
	    def __get__(self, obj, type=None):
	        return obj._slotvalues[self.i]
	    def __set__(self, obj, value):
	        obj._slotvalues[self.i] = value
	        
	class Type(type):
	    # 使用元类实现slots
	    def __new__(self, name, bases,  namespace):
	        slots = namespace.get('_slots_')
	        if slots:
	            for i, slot in enumerate(slots):
	                namespace[slot] = Member(i)
	            original_init = namespace.get('__init__')
	            def __init__(self, *args, **kwargs):
	                # 创建_slotvalues列表和调用原来的__init__
	                self._slotvalues = [None] * len(slots)
	                if original_init(self, *args, **kwargs):
	                    original_init(self, *args, **kwargs)
	            namespace['__init__'] = __init__
	        return type.__new__(self, name, bases, namespace)
	    
	# Python2与Python3使用元类的区别    
	try:
	    class Object(object): __metaclass__ = Type
	except:
	    class Object(metaclass=Type): pass
	
	class A(Object):
	    _slots_ = 'x', 'y'
	
	a = A()
	a.x = 10
	print(a.x)


在**CPython**中，当一个A类定义了\_\_slots\_\_ = ('x', 'y')，A.x就是一个有\_\_get\_\_和\_\_set\_\_方法的member_descriptor，并且在每个实例中可以通过直接访问内存（direct memory access）获得。（具体实现是用偏移地址来记录描述器，通过公式可以直接计算出其在内存中的实际地址 ，访问\_\_dict\_\_也是用相同的方法，也就是说访问A.\_\_dict\_\_和A.x描述器的速度是相近的）。

在上面的例子中，我们用纯Python实现了一个等价的slots。当一个元类看到\_slots\_定义了x和y，它会创建两个的类变量，x = Member(0)和y = Member(1)。然后，装饰\_\_init\_\_方法让新的实例创建一个_slotvalues列表。

**例子中的实现和CPython不同的是**：

例子中\_slotvalues是一个存储在类对象外部的列表，而在**CPython**中它与实例对象存储在一起，可以通过直接访问内存获得。相应地，member decriptor也不是存在外部列表中，而同样可以通过直接访问内存获得。

默认情况下，\_\_new\_\_方法会为每个实例创建一个字典\_\_dict\_\_来存储实例的属性。但如果定义了\_\_slots\_\_，\_\_new\_\_方法就不会再创建这个字典。

由于不存在\_\_dict\_\_来存储新的属性，所以使用一个不在\_\_slots\_\_中的属性时，程序会报错。

## 更快的属性访问速度

默认情况下，访问一个实例的属性是通过访问该实例的\_\_dict\_\_来实现的。如访问a.x就相当于访问a.\_\_dict\_\_['x']。为了便于理解，我粗略地将它拆分为四步：

1. a.x 
2. a.\_\_dict\_\_ 
3. a.\_\_dict\_\_['x'] 
4. 结果

从\_\_slots\_\_的实现可以得知，定义了\_\_slots\_\_的类会为每个属性创建一个描述器。访问属性时就直接调用这个描述器。在这里我将它拆分为三步：

1. b.x 
2. member decriptor 
3. 结果

我在上文提到，访问\_\_dict\_\_和描述器的速度是相近的，而通过\_\_dict\_\_访问属性多了a.\_\_dict\_\_['x']字典访值一步（一个哈希函数的消耗）。由此可以推断出，使用了\_\_slots\_\_的类的属性访问速度比没有使用的要快。下面用一个例子验证：

	from timeit import repeat
	
	class A(object): pass
	
	class B(object): __slots__ = ('x')
	
	def get_set_del_fn(obj):
	    def get_set_del():
	        obj.x = 1
	        obj.x
	        del obj.x
	    return get_set_del
	
	a = A()
	b = B()
	ta = min(repeat(get_set_del_fn(a)))
	tb = min(repeat(get_set_del_fn(b)))
	print("%.2f%%" % ((ta/tb - 1)*100))

在本人电脑上测试速度有10%-20%的提升。

## 减少内存消耗

**Python内置的字典本质是一个哈希表，它是一种用空间换时间的数据结构。为了解决冲突的问题，当字典使用量超过2/3时，Python会根据情况进行2-4倍的扩容。由此可预见，取消\_\_dict\_\_的使用可以大幅减少实例的空间消耗。**

下面使用memory_profiler模块，memory_profiler模块是在逐行的基础上，测量代码的内存使用率。尽管如此，它可能使得你的代码运行的更慢。使用装饰器@profile来标记哪个函数被跟踪。
	
	from  memory_profiler import profile
	
	class A(object):  # 没有定义__slots__属性
	    def __init__(self, x):
	        self.x = x
	
	@profile
	def main():
	    f = [A(523825) for i in range(100000)]
	
	if __name__ == '__main__':
	    main()

运行结果：

	Line #    Mem usage    Increment   Line Contents
	================================================
	 7        17.2 MiB     17.2 MiB    @profile
	 8                                 def main():
	 9        35.3 MiB     17.0 MiB       f = [A(523825) for i in range(100000)]

第2列表示该行执行后Python解释器的内存使用情况，第3列表示该行代码执行前后的内存变化。

在没有定义\_\_slots\_\_属性的情况下，该代码共使用了(17.2+17.0)MiB内存。

从结果可以看出，内存使用是以MiB为单位衡量的,表示的mebibyte(1MiB = 1.05MB)。

	from memory_profiler import profile

	class A(object):  # 定义了__slots__属性
	    __slots__ = ('x')
	    def __init__(self, x):
	        self.x = x
	
	@profile
	def main():
	    f = [A(523825) for i in range(100000)]
	
	if __name__ == '__main__':
	    main()

运行结果：

	Line #    Mem usage    Increment   Line Contents
	================================================
	   7      16.9 MiB     16.9 MiB   @profile
	   8                              def main():
	   9      22.9 MiB     6.0 MiB       f = [A(523825) for i in range(100000)]

定义\_\_slots\_\_属性的情况下，该代码共使用了(16.9+6.0)MiB内存。

从上述结果可看到使用\_\_slots\_\_能极大地减少内存空间的消耗，这也是最常见到的用法。

# 补充

1.只有非字符串的迭代器可以赋值给\_\_slots\_\_

	>>> class A(object): __slots__ = ('a', 'b', 'c')
	>>> class B(object): __slots__ = 'abcd'
	>>> B.__slots__
	'abc'

若直接将字符串赋值给它，就只有一个属性。

2.关于_\_slots\_\_的继承问题

在一般情况下，使用_\_slots\_\_的类需要直接继承object，如class Foo(object): \_\_slots\_\_ = ()

在继承自己创建的类时，我根据子类父类是否定义了\_\_slots\_\_，将它细分为六种情况:

① 父类有，子类没有：

子类的实例还是会自动创建\_\_dict\_\_来存储属性，不过父类\_\_slots\_\_已有的属性不受影响。
	
	>>> class Father(object): __slots__ = ('x')
	>>> class Son(Base): pass
	>>> son = Son()
	>>> son.x, son.y = 1, 1
	>>> son.__dict__
	>>> {'y': 1}
	
② 父类没有，子类有：

虽然子类取消了\_\_dict\_\_，但继承父类后它会继续生成。同上面一样，\_\_slots\_\_已有的属性不受影响。

	>>> class Father(object): pass
	>>> class Son(Father): __slots__ = ('x')
	>>> son = Son()
	>>> son.x, son.y = 1, 1
	>>> son.__dict__
	>>> {'y': 1}
	
③ 父类有，子类有：

只有子类的\_\_slots\_\_有效，访问父类有子类没有的属性依然会报错。

	>>> class Father(object): __slots__ = ('x', 'y')
	>>> class Son(Father): __slots__ = ('x', 'z')
	>>> son = Son()
	>>> son.x, son.y, son.z = 1, 1, 1
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	AttributeError: 'Son' object has no attribute 'y'

④ 多个拥有非空_\_slots\_\_的父类：

由于\_\_slots\_\_的实现不是简单的列表或字典，多个父类的非空\_\_slots\_\_不能直接合并，所以使用时会报错（即使多个父类的非空\_\_slots\_\_是相同的）。

	>>> class Father(object): __slots__ = ('x')
	>>> class Mother(object): __slots__ = ('x')
	>>> class Son(Father, Mother): pass
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	TypeError: Error when calling the metaclass bases
	multiple bases have instance lay-out conflict

⑤ 多个空_\_slots\_\_的父类：

这是关于_\_slots\_\_使用多继承唯一办法。

⑥ 某些父类有，某些父类没有：

跟第一种情况类似。

为了正确使用\_\_slots\_\_，最好直接继承object。如有需要用到其他父类，则父类和子类都要定义\_\_slots\_\_，还要记得子类的\_\_slots\_\_会覆盖父类的\_\_slots\_\_;除非所有父类的\_\_slots\_\_都为空，否则不要使用多继承。

3.添加\_\_dict\_\_获取动态特性

在特殊情况下，可以在\_\_slots\_\_里添加\_\_dict\_\_来获取与普通实例同样的动态特性。

	>>> class A(object): __slots__ = ()
	>>> class B(A): __slots__ = ('__dict__', 'x')
	>>> b = B()
	>>> b.x, b.y = 1, 1
	>>> b.__dict__
	{'y': 1}

4.添加\_\_weakref\_\_获取弱引用功能

\_\_slots\_\_的实现不仅取消了\_\_dict\_\_的生成，也取消了\_\_weakref\_\_的生成。同样的，在\_\_slots\_\_将其添加可以重新获取弱引用这一功能。

5.如果类变量与\_\_slots\_\_的变量同名，则该变量被设置为readonly！！！

	class base(object):  
	    __slots__=('y')  
	    y=22 # y是类变量,y与__slots__中的变量同名  
	    var=11  
	    def __init__(self):  
	        pass  
	      
	b=base()  
	print (b.y)  
	print (base.y)  
	#b.y=66 #AttributeError: 'base' object attribute 'y' is read-only  
运行结果：

	22
	22
6.namedtuple

利用内置的namedtuple不可变的特性，结合\_\_slots\_\_，能创建出一个轻量不可变的实例。(约等于一个元组的大小)

	>>> from collections import namedtuple
	>>> class MyNt(namedtupele('MyNt', 'bar baz')): __slots__ = ()
	>>> nt = MyNt('r', 'z')
	>>> nt.bar
	'r'
	>>> nt.baz
	'z'

# 总结

当一个类需要创建大量实例时，可以使用\_\_slots\_\_来减少内存消耗。如果对访问属性的速度有要求，也可以酌情使用。另外可以利用\_\_slots\_\_的特性来限制实例的属性。而用在普通类身上时，使用\_\_slots\_\_后会丧失动态添加属性和弱引用的功能，进而引起其他错误，所以在一般情况下不要使用它。