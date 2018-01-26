---
title: python中\_\_get\_\_、\_\_getattr\_\_、\_\_getattribute\_\_详解
date: 2018-01-12 08:25:32
tags: python面向对象
toc: true
categories: Python
---
\_\_get\_\_,\_\_getattr\_\_和\_\_getattribute\_\_（只存在于新式类）都是访问属性的方法，但有一定的区别。 

	object.__getattr__(self, name) 
通过实例访问属性时，若属性不存在则会调用\_\_getattr\_\_方法，默认触发AttributeError异常；若属性存在，但手动引发AttributeError异常，也会调用\_\_getattr\_\_方法，好比一个异常处理函数。

	object.__getattribute__(self, name) 
通过实例访问属性时，\_\_getattribute\_\_方法是无条件调用的，无论被访问的属性是否存在，它都会被触发，并返回属性本身；

	object.__get__(self, instance, owner) 
如果class定义了它，则这个class就可以称为descriptor。owner是所有者的类，instance是访问descriptor的实例，如果不是通过实例访问，而是通过类访问的话，instance则为None。需要注意的是：descriptor的实例自己访问自己时不会触发\_\_get\_\_，而会触发\_\_call\_\_，只有descriptor作为其它类的属性才有意义。

前两个都是通过实例访问才会触发，最后一个通过类或实例访问都可以触发，下面我们看一个例子，加深一下理解：

<!--more-->

	class C(object):                                                      
    	a = 'abc'                                                         
    	def __getattribute__(self, *args, **kwargs):                      
        	print("__getattribute__() is called")                         
        	return object.__getattribute__(self, *args, **kwargs)         
    	def __getattr__(self, name):                                      
        	print("__getattr__() is called ")                             
        	return name + " from getattr"                                 
                                                                      
    	def __get__(self, instance, owner):                               
        	print("__get__() is called", instance, owner)                 
                                                                      
	class C2(object):                                                     
    	d = C()                                                           
	if __name__ == '__main__':                                            
    	c = C()
		c2 = C2()                                                       
		print(c.a)                                                      
		print(c.zzzzzzzz)                                               
		c2.d                                                            
		print(c2.d.a)    


运行结果：

	__getattribute__() is called
	abc
	__getattribute__() is called
	__getattr__() is called 
	zzzzzzzz from getattr
	('__get__() is called', <__main__.C2 object at 0x0000000002A76400>, <class '__main__.C2'>)
	('__get__() is called', <__main__.C2 object at 0x0000000002A76400>, <class '__main__.C2'>)
	__getattribute__() is called
	abc   

简单说明一下：

if里面前两行是获取类实例，不会有任何输出；

第三行通过实例c访问存在的属性a会调用 \_\_getattribute\_\_ 方法，看到前两行输出结果；

第四行通过实例访问不存在的属性zzzzzzzz会先调用\_\_getattribute\_\_ 方法然后调用\_\_getattr\_\_方法，看到了3~5行的输出结果；

第五行通过C2的实例访问其属性d，d又是C类的一个实例，所以会触发\_\_get\_\_方法，看到第六行输出结果。

第六行通过C2的属性d（C的实例）调用C存在的属性a会先触发方法\_\_get\_\_然后调用\_\_getattribute\_\_ 方法，看到7~9行输出。


<font size=5>**补充**</font>

大多时候我们并不太需要关注\_\_getattribute\_\_ 和\_\_getattr\_\_的一些细节，一般情况下使用我们自定义的类的时候，我们对类的结构都了解，不会刻意偏离，造成一些属性访问的错误等。

值得一提的是适当的重写会使我们的程序变的优雅，但也有一些需要注意的地方，看下面的例子：

链式生成url:

	class UrlGenerator(object):                                        
    	def __init__(self, root_url):                                  
        	self.url = root_url                                        
                                                                   
    	def __getattr__(self, item):                                   
        	if item == 'get' or item == 'post':                        
            	print (self.url)                                       
        	return UrlGenerator('{}/{}'.format(self.url, item))        
                                                                   
	url_gen = UrlGenerator('http://www.aa.com')                        
	url_gen.users.show.get

结果：

	http://www.aa.com/users/show                                             


<font size=4>**注意点：**</font>

**1.自定义getattribute的时候防止无限递归**

因为getattribute在访问属性的时候一直会被调用，自定义的getattribute方法里面同时需要返回相应的属性，通过self.__dict__取值会继续向下调用getattribute，造成循环调用：

	class A(object):                                                                                                     
    	def __getattribute__(self, item):                                                                                
        	print(item)                                                                                                  
        	return self.__dict__[item]    #RuntimeError: maximum recursion depth exceeded while calling a Python object  
	a=A()                                                                                                                
	a.aaa                                                                                                                
 
上面的代码运行会输出 aaa ，但是会报超出最大递归深度的异常，这是需要重点注意的。                                                                                                                     
                                                                                                                     
	class A(object):                                                                                                     
    	def __getattribute__(self, item):                                                                                
        	try:                                                                                                         
            	print(item)                                                                                              
            	super(A,self).__getattribute__(item)# AttributeError: 'A' object has no attribute 'aaa'                  
        	except KeyError as e:                                                                                        
            	return 'default'                                                                                         
	a=A()                                                                                                                
	a.aaa                                                                                                                

上面的代码运行也会输出 aaa ,然后 aaa这个属性找不到，然后抛出AttributeError，调用 \_\_getattr\_\_方法。这也是我们想要的结果。


**2.同时覆盖掉getattribute和getattr的时候，在getattribute中需要模仿原本的行为抛出AttributeError或者手动调用getattr**


	class A(object):                                           
    	def  __init__(self,name):                              
        	self.name=name                                     
    	def __getattribute__(self, item):                      
        	try:                                               
            	return super(A,self).__getattribute__(item)    
        	except KeyError:                                   
            	return 'default'                               
        	except AttributeError as e:                        
            	print(e)                                       
    	def __getattr__(self, item):                           
        	return 'default'                                   
                                                           
	a=A('_learner')                                            
	print(a.name)                                              
	print(a.age)                                               

运行结果：

	_learner
	'A' object has no attribute 'age'
	None

上面例子里面的getattr方法根本不会被调用，因为原本的AttributeError被我们自行处理并未抛出，也没有手动调用getattr，所以访问age的结果是None而不是default.


<font size='5'>**总结**</font>

每次通过实例访问属性，都会经过\_\_getattribute\_\_函数。而当属性不存在时，仍然需要访问\_\_getattribute\_\_，不过接着要访问\_\_getattr\_\_，\_getattr\_\_好像是一个异常处理函数。 

每次访问descriptor（即实现了\_\_get\_\_的类），都会先经过\_\_get\_\_函数。 

当使用类访问不存在的变量时，不会经过\_\_getattr\_\_函数。而descriptor不存在此问题，只是把instance标识为none而已。

在重写 \_\_getattribute\_\_、\_getattr\_\_方法时，要注意堆栈溢出以及吃掉AttributeError异常的问题。