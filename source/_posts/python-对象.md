---
title: python 对象
date: 2017-11-22 09:44:18
tags: python基础
categories: Python
toc: true
---
在python中一切皆为对象，Python 使用对象模型来存储数据。构造任何类型的值都是一个对象。

所有的 Python 对像都拥有三个特性：**身份、类型、值**。

**身份**：每一个对象都有一个唯一的身份标识自己，任何对象的身份可以使用内建函数 id()来得到。
这个值可以被认为是该对象的内存地址。您极少会用到这个值，也不用太关心它究竟是什么。

**类型**：对象的类型决定了该对象可以保存什么类型的值，可以进行什么样的操作，以及遵循什么样的规则。您可以用内建函数 type()查看 Python 对象的类型。因为在 Python 中类型也是对象。

**值**：对象表示的数据项

Python 有一系列的基本（内建）数据类型，必要时也可以创建自定义类型来满足你的应用程序的需求。绝大多数应用程序通常使用**标准类型**，对特定的数据存储则通过创建和实例化类
来实现
<!--more-->

# python 标准类型

1. 数字（分为几个子类型，其中有三个是整型）
	1. Integer 整型
	2. Boolean布尔型
	3. Long integer长整型
	4. Float浮点型
	5. Complex number复数型

2. String 字符串 

3. List 列表

4. Tuple 元组

5. Dictionary 字典

## 标准类型操作符

### 值比较
	>>> 2 == 2
	True
	>>> 2.46 <= 8.33
	True
	>>> 5+4j >= 2-3j
	True
	>>> 'abc' == 'xyz'
	False
	>>> 'abc' > 'xyz'
	False                            
	>>> 'abc' < 'xyz'
	True
	>>> [3, 'abc'] == ['abc', 3]
	False
	>>> [3, 'abc'] == [3, 'abc']
	True
	不同于很多其它语言，多个比较操作可以在同一行上进行，求值顺序为从左到右。
	>>> 3 < 4 < 7 # same as ( 3 < 4 ) and ( 4 < 7 )
	True
	>>> 4 > 3 == 3 # same as ( 4 > 3 ) and ( 3 == 3 )
	True
	>>> 4 < 3 < 5 != 2 < 7
	False
我们会注意到比较操作是针对对象的值进行的，也就是说比较的是对象的数值而不是对象本身。

### 身份比较
python 提供了 is 和 is no操作符来测试两个变量是否指向同一个对象。

	>>> a=3
	>>> b=3
	>>> a is b # 等价于 id(a)==id(b)
	True
	>>> l=[1,2,3]
	>>> ll=[1,2,3]
	>>> l is ll
	False #可变类型的赋值操作是新创建一个对象，详见：[Python的可变与不可变数据类型](http://waisunny.com/2017/11/20/Python%E7%9A%84%E5%8F%AF%E5%8F%98%E4%B8%8E%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/)

## 标准类型内建函数

Python 提供了一些内建函数用于这些基本对象类型：cmp(), repr(), str(), type(), 和等同于 repr()函数的单反引号(``) 运算符。

1. cmp(obj1, obj2)： 比较 obj1 和 obj2, 根据比较结果返回整数 i:
	1. i < 0 if obj1 < obj2
	2. i > 0 if obj1 > obj2
	3. i == 0 if obj1 == obj2
2. repr(obj) 或 `obj` 返回一个对象的字符串表示

3. str(obj) 返回对象适合可读性好的字符串表示

4. type(obj) 得到一个对象的类型，并返回相应的 type 对象

说明：cmp()函数在python3中已被舍弃，详情参考：[http://blog.csdn.net/sushengmiyan/article/details/11332589](http://blog.csdn.net/sushengmiyan/article/details/11332589 "cmp函数在python3中的替换版本")
## 标准类型工厂函数

工厂函数:看上去有点象函数， 实质上他们是类。当你**调用它们时，实际上是生成了该类型的一个实
例**， 就象工厂生产货物一样。

在以前的python版本中称为内建函数，现在是工厂函数：

int(), long(), float(), complex()，str()

unicode(), basestring(),list(), tuple(),type()

python后来版本增加的工厂函数：

dict(),bool(),set(), frozenset(), object(),file()

classmethod(),staticmethod(), super(), property()

	>>> list()
	[]
	>>> dict
	<class 'dict'>
	>>> list()
	[]
	>>> dict()
	{}
	>>> float()
	0.0
	>>> int()
	0
	>>> bool()
	False

## 标准类型分类

### 存储模型

1. 标量/原子类型： 数值（所有数值类型），字符串

2. 容器类型：列表、元祖、字典

### 更新模型
1. 可变类性：列表、字典
2. 不可变类型：数字、字符串、元祖

### 访问模型
1. 直接访问：数字
2. 顺序访问：字符串、列表、元祖
3. 映射访问：字典

# python其他内建类型

1. 类型(type)
2. Null 对象 (None)
3. 文件
4. 集合/固定集合（set）
5. 函数/方法
6. 模块
7. 类

# python内部类型
1. 代码（编译过的python源代码片段）
2. 帧(python的执行帧栈)
3. 跟踪记录(Traceback)
4. 切片
5. 省略
6. range


# 补充
解释：**内建** ：是由python的解释器本身支持的，不需要任何import就可以使用的，而且也找不到.py的源码。
