---
title: python中的作用域
date: 2017-12-16 11:03:42
tags: python基础
toc: true
categories: python
---
1、‘块级作用域’

想想此时运行下面的程序会有输出吗？执行会成功吗？
 
	if 1 == 1:
    	name = "_learner"
	print(name)
	for i in range(10):
    	age = i
	print(age)
执行结果：

	_learner
	9
<!--more-->
代码执行成功，没有问题；但在Java/C#中，执行上面的代码会提示name，age没有定义，而在Python中可以执行成功，这是因为在**Python中是没有块级作用域的**，代码块里的变量，外部可以调用，所以可运行成功；　　

 
2、局部作用域

回顾之前学过的知识，我们学函数的时候，函数是个单独的作用域，Python中没有块级作用域，但是有局部作用域；看看下面的代码

	def  func():
    	name = "lzl"
	print(name)
运行这段代码，想想会不会有输出？

抛出异常：

	NameError: name 'name' is not defined
运行报错，我相信这个大家都能理解，name变量只在func()函数内部中生效，所以在全局中是没法调用的；对上面代码做个简单调整，再看看结果如何?

	def  func():
    	name = "lzl"
	func()          #执行函数
	print(name)
对之前的代码添加了一句代码，在变量name打印之前，执行了一下函数，此时打印会不会有变化？

还是抛出异常：
	NameError: name 'name' is not defined

执行依然报错，还是回到刚才那句话：即使执行了一下函数，name的作用域也只是在函数内部，外部依然无法进行调用；把前两个知识点记住，接下来要开始放大招了


3、作用域链(嵌套作用域)

对函数做下调整，看看下面的代码执行结果如何？

	name = "lzl"
	def f1():
    	name = "Eric"
    	def f2():
        	name = "Snor"
        	print(name)
    	f2()
	f1()
学过函数，肯定知道最后f1()执行完会输出Snor；我们先记住一个概念，Python中有作用域链，变量会由内到外找，先去自己作用域去找，自己没有再去上级去找，直到找不到报错为止。

 
4、终极版作用域

	name = "lzl"
	def f1():
    	print(name)
 
	def f2():
    	name = "eric"
    	f1()
	f2()
想想最后f2()执行结果是打印“lzl”呢，还是打印“eric”？记住自己的答案，现在先不把答案贴出来，先看看下面这段代码：

	name = "lzl"
	def f1():
    	print(name)
 
	def f2():
    	name = "eric"
    	return f1
	ret = f2()
	ret()
 
执行结果：

	lzl
执行结果为“lzl”，分析下上面的代码，f2()执行结果为函数f1的内存地址，即ret=f1；执行ret()等同于执行f1()，执行f1()时与f2()没有任何关系，name=“lzl”与f1()在一个作用域链，函数内部没有变量是会向外找，所以此时变量name值为“lzl”；理解了这个，那么刚才没给出答案的那个终极代码的
执行结果：

	lzl
是的，输出的是“lzl”，记住<font color=red>在函数未执行之前，作用域已经形成了，作用域链也生成了。</font>


5、新浪面试题

	>>> li = [lambda :x for x in range(10)]

判断下li的类型？li里面的元素是什么类型？

	>>> type(li)
	<class 'list'>
	>>> type(li[0])
	<class 'function'>
可以看到li为列表类型，list里面的元素为函数，那么打印list里面第一个元素的返回值，此时返回值为多少？

	>>> res = li[0]()
	>>> res
	9
li第一个函数的返回值为9还不是0，记住：<font color=red>函数在没有执行前，内部代码不执行</font>；所以，列表li中每个函数元素的执行结果都是 for x in range(10) 返回的最后一个x的值，即：9。

	>>> for res in li:
	...     res()
	...
	9
	9
	9
	9
	9
	9
	9
	9
	9
	9
