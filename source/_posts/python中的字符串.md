---
title: python中的字符串
date: 2017-11-18 17:39:38
tags: python基础
toc: true
---
# 字符串介绍
基本字符串操作，索引、分片、乘法、判断成员资格、求长度、取最大最小值等，但请记住字符串是不可变的，so 通过分片赋值都是不合法的。

	>>> x='Dany'
	>>> x[0]
	D
	>>> x[1,3]
	an
	>>> len(x)
	4
	>>> x*3
	DanyDanyDany
	>>> x[1:3]='co'#非法操作，会出错的
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	TypeError: 'str' object does not support item assignment

# 字符串格式化
## 方式一 ‘%’
使用%，在 **%** 左侧放置一个字符串（需要被格式化的字符串），右侧放置希望被格式化成的值，可以使用一个值，如一个字符串或者一个数字；（也可以使用多个值的元组或者下章将会讨论到的字典，前提是需要格式化多个值），一般情况下使用元组

	>>> format="hello %s . My Name is %s"
	>>> words=('Word','Dany')
	>>> format%words
	hello Word . My Name is Dany

说明：格式化操作符的右操作数可以是任意类型，如果右操作数是元组的话，则其中的每一个元素都会被单独格式化，每个值都需要一个对应的转换说明符，并且元组作为转换表达式的一部分存在时，必须使用圆括号括起来。

转换说明符：它们标记了需要插入转换值的位置。s表示值会被格式化为字符串--如果不是字符串，则会用str函数将其转换为字符串。

**如果使用列表或者其他序列代替元组，那么序列会被解释为一个值，只有元组和字典可以格式化一个以上的值。**

	>>> str="hello %s . My Name is %s"
	>>> words=['Word','Dany']
	>>> str%words
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	TypeError: not enough arguments for format string

格式化整数，**%d**

	>>> str="My age is %d"
	>>> vales=23
	>>> str%vales
	My age is 23

格式化实数（浮点数），**%.2f ** 一个句点'.'在加上希望保留的小数位。
	>>> str="Hello,My Height is %.2f m"
	>>> vales=(1.765)
	>>> str%vales
	Hello,My Height is 1.76 m

	>>> str="Pi with three decimal:%.3f"
	>>> str math import pi
	>>> str%pi
	Pi with three decimal:3.141

## 方式二 ‘模板字符串’
模板字符串：string模块提供的另外一种格式化字符的方式。

substitute这个模板方法会用传递进来的关键字参数foo替换字符串中的$foo,并返回替换后的值。

	>>> from string import Template #导入模板
	>>> s=Template('$x. glorious $x!')#包装字符串
	>>> s.substitute(x='slurm')#调用模板方法格式换字符串
	slurm. glorious slurm!

如果替换字段是单词的一部分，那么参数名就必须用括号括起来，从而准确指明结尾：

	>>> s=Template("It's ${x}tastic!")
	>>> s.substitute(x='slurm')
	It's slurmtastic!

可以使用$$插入美元符号

	>>> s=Templatte("Make $$ selling $x!")
	>>> s.substitute(x='slurm')
	Make $ selling slurm!
	

除了关键字参数外，还可以使用字典变量提供值/名称对

	>>> s=Template("A $thing must never $action.")
	>>> d={}
	>>> d['thing']='gentleman'
	>>> d['action']='show his socks'
	>>> s.substitute(d)
	A gentleman must never show his socks.

说明：safe_substitute相比substitute不会因为缺少值或者不正确使用$字符而出错

## 方式三 ‘format函数’

	>>> s_info="{0}'s age is {1}".format('Dany',23)
	>>> s_info
	Dany's age is 23

	>>> s_info="{name}'s age is {age}".format(name='Dany',age=23)
	>>> s_info
	Dany's age is 23

说明：推荐使用第三种，简明直观。

# 字符串拼接
## 方式一 ‘+’
	website = 'http:\\' + 'python' + '.org'

## 方式二 ‘join方法’

	listStr = ['http:\\', 'python', '.org'] 
	website = ''.join(listStr)

## 方式三 ‘格式化(替换)’

	website = '%s%s%s' % ('http:\\', 'python', '.org')

## 三种方式比较

方法1，使用简单直接，但是这种方法效率低，耗内存，之所以说python 中使用 + 进行字符串连接的操作效率低，是因为python中字符串是不可变的类型，使用 + 连接两个字符串时会生成一个新的字符串，生成**新的字符串就需要重新申请内存**，当连续相加的字符串很多时(a+b+c+d+e+f+...) ，效率低下就是必然的了

方法2，使用略复杂，但对多个字符进行连接时效率高，只会有一次内存的申请。而且如果是对list的字符进行连接的时候，这种方法必须是首选

方法3：字符串格式化，这种方法非常常用，本人也推荐使用该方法
### 实验对比 ‘+’ VS ‘join方法’

实验一：

	from time import time
	def method1():
    	t = time()
    	for i in xrange(100000):
       		 s ='python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.	org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'py	thon.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org	'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'pytho	n.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'	python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.org'+'python.o	rg'
    print (time() - t)
	def method2():
    	t = time()
		for i in xrange(100000):
        	s = ''.join	(['python.org','python.org','python.org','python.org','python.org','python.org','python.org','pytho	n.org','python.org','python.org','python.org','python.org','python.org','python.org','python.org','	python.org','python.org','python.org','python.org','python.org','python.org','python.org','python.o	rg','python.org','python.org','python.org','python.org','python.org','python.org','python.org','pyt	hon.org','python.org','python.org','python.org','python.org','python.org','python.org','python.org'	,'python.org','python.org','python.org','python.org','python.org','python.org','python.org','python	.org'])
    print (time() -t)
	method1()
	method2()
结果：

	1.38100004196
	0.137000083923

实验二：

	from time import time
	def method3():
    	t = time()
    	for i in xrange(100000):
        	s = 'python.org'+'python.org'+'python.org'+'python.org'
    	print (time() - t)
	def method4():
    	t = time()
    	for i in xrange(100000):
        	s = ''.join(['python.org','python.org','python.org','python.org'])
    	print (time() -t)
	method3()
	method4()
结果：

	0.0250000953674
	0.0599999427795

上面两个实验出现了完全不同的结果，分析这两个实验唯一不同的是：字符串连接个数。

**结论：加号连接效率低是在连续进行多个字符串连接的时候出现的，如果连接的个数较少，加号连接效率反而比join连接效率高，但是使用‘+’会申请新的内存，当连接次数较少时，推荐使用格式化方法。**

# 字符串方法

## join方法
非常重要的一个字符串方法，它是split方法的逆方法，用来连接序列中的元素，返回字符串。

	>>> strs=['1','2','3','4']
	>>> '+'.join(strs)
	1+2+3+4

	>>> strs='liwei','lijie'
	>>> '--love-->'.join(strs)
	liwei--love-->lijie

	>>> strs=[1,2,3,4,5]
	>>> '+'.join(strs)#出错，因为join需要连接的序列元素必须都是字符串
	Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	TypeError: sequence item 0: expected str instance, int found


## lower方法
返回字符串的小写字母版

	>>> strs='Dany is A Good Boy'
	>>> strs.lower()
	dany is a good boy

## upper方法
返回字符串的大写字母版

	>>> strs='dany is A Good Boy'
	>>> strs.upper()
	DANY IS A GOOd BOY

## replace方法
方法返回某字符串的所有匹配项均被替换后得到的字符串

	>>> strs='Dany Is is nice boy'
	>>> strs.replace('is','very')
	Dany Is very nice boy

## split方法
将字符串分割成序列，返回的数据类型是列表

	>>> strs='1+2+3+4'
	>>> strs.split('+')
	['1', '2', '3', '4']

	>>> strs='1,2,3.4/5'
	>>> strs.split(',')
	['1','2','3.4/5']

	>>> strs='1 2   3,4'
	>>> print(strs.split())#不指定分隔符会默认把所有的空格作为分隔符（空格，制表，换行）
	['1','2','3,4']

## strip方法
去除两侧的空格，不包含内部的

	>>> strs='    123 3323  34      '
	>>> strs.strip()
	123 3323  34

	>>> strs='!*12321!*221!*1231!*'
	>>> print(strs.strip('!*'))#去除指定的字符，但只是去除两侧的
	12321!*221!*1231

## startswith方法
判断是否以某一个字符串开头，返回布尔值

	>>> strs='Dany is a good boy'
	>>> strs.startswith('Da') #注意这里区分大小写
	True

## find方法
查找某一段字符串在一段字符串中出现的第一个位置并返回索引值

	>>> strs='Dany is a good boy，Dany'
	>>> strs.find('Dany')
	0

## count方法
返回某一段字符串出现的次数

	>>> strs='Dany is a good boy，Dany'
	>>> strs.count('Dany')
	2

## center方法 
使字符串居中

	>>> strs='DanyInfo'
	>>> strs.center(30,'*')#参数1：字符串总长度，参数2：填充的字符
	***********DanyInfo***********

## format方法
字符串格式化

	>>> s_info="{0}'s age is {1}".format('Dany',23)
	>>> s_info
	Dany's age is 23

	>>> s_info="{name}'s age is {age}".format(name='Dany',age=23)
	>>> s_info
	Dany's age is 23

# 总结
字符串就介绍这么多，基本上覆盖到我们平常用到的所有字符串相关大知识点，另外在强调一点，字符串是不可变的，不可变类型和可变类型的详细介绍可参考我的下一篇博文。

