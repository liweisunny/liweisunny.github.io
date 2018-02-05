---
title: python-类(继承、派生、组合、包装(授权))
date: 2018-01-28 10:40:19
tags: python面向对象
toc: true
categories: Python
---
在学习python面向对象编程这一章节时，我们会接触到类（class）,与之俱来的就是很多名词：继承、派生、组合、包装授权等等，看介绍感觉看起来它们的意思差不多，都是用来提高代码重用的；整的我们一头雾水，下面我按照个人的理解把它们一一讲解一番。希望对各位迷茫的python学习者有所帮助。
<!--more-->

# 先说继承与派生
我们之所以吧继承和派生放在一起，其实是因为它俩其实是就是一回事，只是定义的角度不同，导致叫法不一样，先不告诉你为什么，看下面。

## 什么是继承？
继承是一种创建类的方法，在python中，一个类可以继承来自一个或多个父类。原始类称为基类或超类。

	class ParentClass1: #定义父类
	    pass
	
	class ParentClass2: #定义父类
	    pass
	
	class SubClass1(ParentClass1): #单继承，基类是ParentClass1，派生类是SubClass1
	    pass
	
	class SubClass2(ParentClass1,ParentClass2): #多继承用逗号分隔开，基类是ParentClass1,ParentClass2，派生类是SubClass2
	    pass
	
查看继承： 使用类的**\_\_bases\_\_** 属性，它返回一个元组。

	>>> SubClass2.__bases__ 
	(<class '__main__.ParentClass1'>, <class '__main__.ParentClass2'>)

### 什么时候用继承
假如已经有几个类，而类与类之间有共同的变量属性和函数属性，那就可以把这几个变量属性和函数属性提取出来作为基类的属性。而特殊的变量属性和函数属性，则在本类中定义，这样只需要继承这个基类，就可以访问基类的变量属性和函数属性。可以提高代码的可扩展性。

### 继承的好处
提高代码的复用。

### 继承的弊端
可能特殊的本类又有其他特殊的地方，又会定义一个类，其下也可能再定义类，这样就会造成继承的那条线越来越长，使用继承的话,任何一点小的变化也需要重新定义一个类,很容易引起类的爆炸式增长,产生一大堆有着细微不同的子类. 所以有个**“多用组合少用继承”**的原则。

## 什么是派生？
那么说了半天什么是派生呢？

**派生就是子类在继承父类的基础上衍生出新的属性。子类中独有的，父类中没有的；或子类定义与父类重名的东西。子类也叫派生类。**

**继承是从子类的角度来讲的，派生是从父类的角度来讲的**；

**子类继承了父类，父类派生了子类 。**

所以派生和继承就是一回事，只是定义的角度不同而已。

# 再说组合

代码复用的重要的方式除了继承，还有组合。

**组合：在一个类中以另外一个类的对象作为数据属性，称为类的组合。**

	class Teacher:
	    def __init__(self, name, gender, course):
	        self.name = name
	        self.gender = gender
	        self.course = course #使用course对象作为数据属性

	class Course:
	    def __init__(self, name, price, period):
	        self.name = name
	        self.price = price
	        self.period = period
	
	course_obj = Course('Python', 15800, '5months')  #新建课程对象
	t_c = Teacher('egon', 'male', course_obj)  #新建老师实例，组合课程对象
	print(t_c.course.name)

运行结果：

	Python

组合与继承都是有效地利用已有类的资源的重要方式，但是二者的概念和使用场景皆不同。

## 继承的方式

通过继承建立了派生类与基类之间的关系，它是一种'是'的关系，比如白马是马，人是动物。

当类之间有很多相同的功能，提取这些共同的功能做成基类，用继承比较好，比如教授是老师:

	class Teacher:
	    def __init__(self,name,gender):
	        self.name=name
	        self.gender=gender
	    def teach(self):
	        print('teaching')
	
	
	class Professor(Teacher):
	    pass
	
	p1=Professor('egon','male')
	p1.teach()

运行结果：

	teaching

## 组合的方式

用组合的方式建立了类与组合的类之间的关系，它是一种‘有’的关系,比如教授有生日，教授教python课程。

	class BirthDate:
	    def __init__(self,year,month,day):
	        self.year=year
	        self.month=month
	        self.day=day
	    def __str__(self):
	        return '生日：{Y}-{M}-{D}'.format(Y=self.year, M=self.month, D=self.day)
	    __repr__=__str__
	
	class Couse:
	    def __init__(self,name,price,period):
	        self.name=name
	        self.price=price
	        self.period=period
	
	    def __str__(self):
	        return '课程名：{name}，价格：{price}，时长：{period}'.format(name=self.name, price=self.price, period=self.period)
	    __repr__ = __str__
	
	
	class Teacher:
	    def __init__(self,name,gender):
	        self.name=name
	        self.gender=gender
	    def teach(self):
	        print('teaching')

	class Professor(Teacher):
	    def __init__(self,name,gender,birth,course):
	        Teacher.__init__(self,name,gender)
	        self.birth=birth
	        self.course=course
	
	p1=Professor('_learner','male',
	             BirthDate('1994','09','09'),
	             Couse('python','28000','4 months'))
	
	print(p1.birth)
	print(p1.course)

运行结果：
	
	生日：1994-09-09
	课程名：python，价格：28000，时长：4 months

# 最后谈包装(授权)

包装：意思是对一个**已存在**的对象进行包装，不管它是数据类型，还是一段代码，可以是对一个已存在的对象，增加新的，删除不要的，或者修改其它已存在的功能。

包装包括定义一个类，它的实例拥有标准类型的核心行为。换句话说，它现在不仅能唱能跳，还能够像原类型一样步行，说话。

下图举例说明了在类中包装的类型看起像个什么样子。在图的中心为标准类型的核心行为，但它也通过新的或最新的功能，甚至可能通过访问实际数据的不同方法得到提高。

![](https://i.imgur.com/nN64ARe.png)

你还可以包装类，但这不会有太多的用途，因为已经有用于操作对象的机制，并且在上面已描述过，对标准类型有对其进行包装的方式。你如何操作一个已存的类，模拟你需要的行为，删除你不喜欢的，并且可能让类表现出与原类不同的行为呢？我们前面已讨论过，就是采用派生。

所以说包装和派生之间没有明确的界限，包装主要用于对已存在的类型进行操作。

**授权是包装的一个特性，可用于简化处理有关 dictating 功能，采用已存在的功能以达到最大限度的代码重用。**

**实现授权的关键点就是覆盖\_\_getattr\_\_()方法，在代码中包含一个对 getattr()内建函数的调用。**

**特别地，调用 getattr()以得到默认对象属性（数据属性或者方法）并返回它以便访问或调用。**

**特殊方法\_\_getattr\_\_()的工作方式是，当搜索一个属性时，任何局部对象首先被找到（定制的对象）。如果搜索失败了，则\_\_getattr\_\_()会被调用，然后调用 getattr()得到一个对象的默认行为。**

换言之，当引用一个属性时，Python 解释器将试着在局部名称空间中查找那个名字，比如一个自定义的方法或局部实例属性。如果没有在局部字典中找到，则搜索类名称空间。最后，如果两类搜索都失败了，搜索则对原对象开始授权请求，此时，\_\_getattr\_\_()会被调用。意思就是当访问的属性找不到的时候回调用\_\_getattr\_\_()。

包装标准类型：

	class WrapMe:
	    def __init__(self,obj):
	        self.__data=obj
	    def __repr__(self):
	        return repr(self.__data)
	    def __getattr__(self, item):
	        return getattr(self.__data,item)
	
	
	cpl=WrapMe(3.5+4.2j)
	print(cpl)
	print(cpl.real )
	print(cpl.imag )
	print(cpl.conjugate())
	print('='*20)

	lst=WrapMe([1,2,5,2])
	lst.append('bar')
	print(lst)
	print(lst.index(2)) #1
	print(lst.count(2)) #2
	lst.pop()
	print(lst)

运行结果：

	(3.5+4.2j)
	3.5
	4.2
	(3.5-4.2j)
	====================
	[1, 2, 5, 2, 'bar']
	1
	2
	[1, 2, 5, 2]

包装特殊对象：

	class CapOpen:
	    def __init__(self,fn,m='r',buf=-1):
	        self.file=open(fn,m,buf)
	
	    def __iter__(self):
	        return self.file
	
	    def write(self,line):
	        self.file.write(line.upper())
	
	    def __getattr__(self, item):
	        return getattr(self.file,item)
	f=CapOpen('test.txt','r+')
	f.write('_learner')
	for lin in f:
	    print(lin)
	f.close()

运行结果：

	_LEARNER

# 结语
相信你看完这篇文章，应该能搞明白什么是继承、派生、组合、包装授权了。