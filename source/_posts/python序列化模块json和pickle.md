---
title: python序列化模块json和pickle
date: 2018-01-29 16:06:58
tags: python模块
toc: true
categories: Python
---
什么是序列化？

我们把对象(变量)从内存中变成可存储或传输的过程称之为序列化，在Python中叫pickling，在其他语言中也被称之为serialization，marshalling，flattening等等，都是一个意思。

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。
<!--more-->

# json

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。JSON不仅是标准格式，并且比XML更快，而且可以直接在Web页面中读取，非常方便。
JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：
![](https://i.imgur.com/4a221yG.png)

Json模块提供了四个功能：dumps、dump、loads、load 。

	import json
	dic = {'name': '_learner', 'age': 23, 'sex': 'male'}
	print(type(dic))  # <class 'dict'>

	# -----------------------------序列化
	j = json.dumps(dic)
	print(type(j))  # <class 'str'>
	
	f = open('file.txt', 'w')
	f.write(j)  # 等价于json.dump(dic,f)
	f.close()
	
	# -----------------------------反序列化
	f = open('file.txt')
	data = json.loads(f.read())  # 等价于data=json.load(f)
	print(data) # {'name': '_learner', 'age': 23, 'sex': 'male'}

## 使用json注意事项

	import json
	#dic={'1':111} #json.loads方法只接受字符串参数
	#print(json.loads(dic)) # TypeError: the JSON object must be str, bytes or bytearray, not 'dict'
	
	#dic="{'1':111}"   # json 不认单引号
	#print(json.loads(dic)) # json.decoder.JSONDecodeError: Expecting property name enclosed in double quotes
	
	#dic=str({"1":111})# 报错,因为生成的数据还是单引号:{'one': 1}
	#print(json.loads(dic)) # json.decoder.JSONDecodeError: Expecting property name enclosed in double quotes
	
	dic='{"1":111}'
	print(json.loads(dic))# {'1': 111}


json不认单引号；另外，无论数据是怎样创建的，只要满足json格式，就可以json.loads出来,不一定非要dumps的数据才能loads。

# pickle

pickle是python特有的序列化方式，只能在python间使用。

pickle模块也提供了四个功能：dumps、dump、loads、load。

	import pickle
	dic = {'name': '_learner', 'age': 23, 'sex': 'male'}
	print(type(dic))  # <class 'dict'>
	
	##-------------------------序列化
	j = pickle.dumps(dic)
	print(type(j))  # <class 'bytes'>
	f = open('file_pickle', 'wb')  # 注意w是写入str,wb是写入bytes,j是'bytes'类型的
	f.write(j)  # -------------------等价于pickle.dump(dic,f)
	f.close()
	
	# -------------------------反序列化
	f = open('file_pickle', 'rb')
	data = pickle.loads(f.read())  # 等价于data=pickle.load(f)
	print(data['age']) # 23


# pickle和json区别 

json是可以在不同语言之间交换数据的，而pickle只在python之间使用。

json只能序列化最基本的数据类型，而pickle可以序列化所有的数据类型，包括类，函数都可以序列化。

# 补充

在这里介绍一下shelve模块，它也是一种序列化数据的方式。

shelve模块比pickle模块简单，只有一个open函数，返回类似字典的对象，可读可写;key必须为字符串，而值可以是python所支持的数据类型。

	import shelve
	f = shelve.open(r'shelve.txt')
	f['stu_info']={'name':'_learner','age':'23'}
	f.close()
	f = shelve.open(r'shelve.txt','r')
	print(f.get('stu_info')['age'])
	f.close()