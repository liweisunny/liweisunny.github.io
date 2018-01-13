---
title: Python 开发者都会遇到的错误：UnboundLocalError
date: 2017-12-17 11:03:42
tags: python基础
toc: true
categories: python
---
每个 Python 开发者都至少会经历一次 UnboundLocalError 的错误，初次遇到这种错误会觉得莫名其妙，用这张图来描述当时的心情最为贴切：
![](https://i.imgur.com/nJmYi4j.jpg)
<!--more-->
比如下面的代码在 foo 函数中给 x 自增 1：

	x = 10
	def foo():
    	x += 1
    	print(x)
	foo()

调用 foo() 的时候，就会看到这个错误：

	UnboundLocalError: local variable 'x' referenced before assignment

堆栈日志告诉我们：局部变量 x 赋值前在其它地方被引用了，换句话说就是 x 在当前作用域内还没有定义就拿来使用了。

明明 x 在函数 foo 的外面定义了，为什么却告知我们说 x 没赋值就被引用了呢？

因为这是几乎每个人都会遇到的错误，所以 Python 官方把这个问题收入到了它的 FAQ 中，它是这样说的：

This is because when you make an assignment to a variable in a scope, that variable becomes local to that scope and shadows any similarly named variable in the outer scope.

<font color=red>**翻译过来就是，当你在局部作用域中给变量赋值时，那么这个变量就会变成一个局部变量，不管它在外部有没有初始化。如果外部作用域有相同名字的变量，那么对局部空间来说这些相同名字的全局变量都将被隐藏为不可见的。**</font>

**如果在局部作用域中没有给变量 x 重新赋值，而是直接引用它，那么 Python 会沿着 LEGB（local->-enclosing->global->built-in） 的规则顺序查找变量。**

在这里，x += 1 等价于 x = x + 1，首先会执行 x + 1 操作，然后再赋值给 x。而执行加法操作之前它会先查找变量 x，根据官方的这个解释所述， Python 认为 x 对函数 foo 来说一个局部变量，因为它有赋值操作，既然 x 是局部变量，在执行加法操作时，又引用了 x，所以抛出了 **UnboundLocalError** 异常。

那么，该异常如何解决？办法非常简单，Python 提供了一个关键字 globlal，用来显示地标识 x 为全局变量，x 设置为全局变量后，执行加操作的时候就会去全局命名空间查找 x 。

	x = 10
	def foo():
    	global x
    	x += 1
    	print(x)
	foo()
执行结果：

	11

容易混淆的可变类型的例子：

	lst = [1, 2, 3]
	def foo():
    	lst.append(5)   # 正常执行，lst 没有重新赋值，首先在局部作用域查找，没找到再往全局作用域查找 lst
		#lst += [5]     # UnboundLocalError 错误，因为重新赋值了，而在局部作用域没定义就引用了
	foo()
	print(lst)

这个问题解决之后，再来看另外一个类似的问题：

	def external():
    	x = 10
    	def internal():
        	x += 1
        	print(x)
    	internal()
	external()

这是一个 Python 闭包，执行 external 函数的时候，看代码你也知道会报同样的 UnboundLocalError 错误，那么用 global 来 修复可行吗？试试：

	def external():
    	x = 10
    	def internal():
        	global x
        	x += 1
        	print(x)
    	internal()
	external()

新的错误出现了:

	NameError: name 'x' is not defined
全部变量 x 没有定义，仔细想想也是啊，你看 x 是定义在 external 函数中的一个局部变量，现在你要把 internal 函数中的 x 声明为全局变量，Python 在全局作用域空间 找不到 x ，所以出现了 NameError，那么这个问题又该如何解决呢？如果你是使用 Python3，恭喜你，Python3 中新增了一个关键字 nonlocal，用于表示非局部变量。

	def external():
    	x = 10
    	def internal():
        	nonlocal x
        	x += 1
        	print(x)
    	internal()
	external()
执行结果：
	
	11


补充：

	import dis
	x = 10
	def foo():
    	x += 1
    	print(x)
	dis.dis(foo)

用 dis.dis(foo) 可以看到 Python 内部字节码指令的执行过程，从字节码中可以知道，第三行代码 x= x+1 操作的指令是 LOAD_FAST ，LOAD_FAST  0 (x) 表示 Python 解释器 从局部作用域加载 x，而 x 由找不到，因此出现了UnboundLocalError。

	4         0 LOAD_FAST                0 (x)
              2 LOAD_CONST               1 (1)
              4 INPLACE_ADD
              6 STORE_FAST               0 (x)

	5         8 LOAD_GLOBAL              0 (print)
             10 LOAD_FAST                0 (x)
             12 CALL_FUNCTION            1
             14 POP_TOP
             16 LOAD_CONST               0 (None)
             18 RETURN_VALUE