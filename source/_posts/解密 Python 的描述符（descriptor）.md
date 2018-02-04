---
title: 解密 Python 的描述符（descriptor）
date: 2018-01-27 09:40:19
tags: python面向对象
toc: true
categories: Python
---
# 什么是描述符？

严格来讲：描述符实际上可以是任何一个（新式）类，这种类至少实现了三个特殊方法\_\_get\_\_()、\_\_set\_\_()、\_\_delete\_\_()中的一个，这三个特殊方法冲当描述符协议。（摘自核心编程）

\_\_get\_\_()、\_\_set\_\_()及\_\_delete\_\_()的原型如下所示：

　　① \_\_get\_\_(self, instance, owner) ==> Value

　　② \_\_set\_\_(self, instance, value) ==>None

　　③ \_\_delete\_\_(self, instance)  ==> None

<!--more-->

# 描述的有啥用？

一句话：描述符是用作属性代理的。**前提这个属性必须是类属性**(后面会讲解为啥不能是实例属性)；通过描述符我们可以将属性复杂的逻辑判断封装到一个单独的类中，以便于重用。

案例：

想象一下你正在编写管理电影信息的代码。你最后写好的Movie类可能看上去是这样的：

	class Movie(object):
	    def __init__(self, title, rating, runtime, budget, gross):
	        self.title = title
	        self.rating = rating
	        self.runtime = runtime
	        self.budget = budget
	        self.gross = gross
	 
	    def profit(self):
	        return self.gross - self.budget

你开始在项目的其他地方使用这个类，但是之后你意识到：如果不小心给电影打了负分怎么办？你觉得这是错误的行为，希望Movie类可以阻止这个错误。 你首先想到的办法是将Movie类修改为这样：

	class Movie(object):
	    def __init__(self, title, rating, runtime, budget, gross):
	        self.title = title
	        self.rating = rating
	        self.runtime = runtime
	        self.gross = gross
	        if budget < 0:
	            raise ValueError("Negative value not allowed: %s" % budget)
	        self.budget = budget
	 
	    def profit(self):
	        return self.gross - self.budget

但这行不通。因为其他部分的代码都是直接通过Movie.budget来赋值的——这个新修改的类只会在\_\_init\_\_方法中捕获错误的数据，但对于已经存在的类实例就无能为力了。

## property
幸运的是，Python的property解决了这个问题：

	class Movie(object):
	    def __init__(self, title, rating, runtime, budget, gross):
	        self._budget = None
	        self.title = title
	        self.rating = rating
	        self.runtime = runtime
	        self.gross = gross
	        self.budget = budget
	
	    @property
	    def budget(self):
	        return self._budget
	
	    @budget.setter
	    def budget(self, value):
	        if value < 0:
	            raise ValueError("Negative value not allowed: %s" % value)
	        self._budget = value
	
	    def profit(self):
	        return self.gross - self.budget

	m = Movie('Casablanca', 97, 102, 964000, 1300000)
	print(m.budget)
	try:
	    m.budget = -100 
	except ValueError:
	    print("Woops. Not allowed")
运行结果：
	
	964000
	Woops. Not allowed

我们用@property装饰器指定了一个getter方法，用@budget.setter装饰器指定了一个setter方法。当我们这么做时，每当有人试着访问budget属性，Python就会自动调用相应的getter/setter方法。比方说，当遇到m.budget = value这样的代码时就会自动调用budget.setter。

花点时间来欣赏一下Python这么做是多么的优雅：如果没有property，我们将不得不把所有的实例属性隐藏起来，提供大量显式的类似get_budget和set_budget方法。像这样编写类的话，使用起来就会不断的去调用这些getter/setter方法，这看起来就像臃肿的Java代码一样。更糟的是，如果我们不采用这种编码风格，直接对实例属性进行访问。那么稍后就没法以清晰的方式增加对非负数的条件检查——我们不得不重新创建set_budget方法，然后搜索整个工程中的源代码，将m.budget = value这样的代码替换为m.set_budget(value)。太蛋疼了！！

因此，property让我们将自定义的代码同变量的访问/设定联系在了一起，同时为你的类保持一个简单的访问属性的接口。干得漂亮！

但property也有不足的地方，对property来说，最大的缺点就是它们不能重复使用，假设你想为rating，runtime和gross这些字段也添加非负检查，那么你就需要向像给budget增加方法一样，分别为每一个属性定义它们的赋值和取值方法。

## 描述符登场（最终的大杀器）

这就是描述符所解决的问题。描述符是property的升级版，允许你为重复的property逻辑编写单独的类来处理。下面的示例展示了描述符是如何工作的：
	
	from weakref import WeakKeyDictionary
	
	class NonNegative(object):
	    """A descriptor that forbids negative values"""
	
	    def __init__(self, default):
	        self.default = default
	        self.data = WeakKeyDictionary()
	
	    def __get__(self, instance, owner):
	        # we get here when someone calls x.d, and d is a NonNegative instance
	        # instance = x
	        # owner = type(x)
	        return self.data.get(instance, self.default)
	
	    def __set__(self, instance, value):
	        # we get here when someone calls x.d = val, and d is a NonNegative instance
	        # instance = x
	        # value = val
	        if value < 0:
	            raise ValueError("Negative value not allowed: %s" % value)
	        self.data[instance] = value
	
	class Movie(object):
	    # always put descriptors at the class-level
	    rating = NonNegative(0)
	    runtime = NonNegative(0)
	    budget = NonNegative(0)
	    gross = NonNegative(0)
	
	    def __init__(self, title, rating, runtime, budget, gross):
	        self.title = title
	        self.rating = rating
	        self.runtime = runtime
	        self.budget = budget
	        self.gross = gross
	
	    def profit(self):
	        return self.gross - self.budget
	
	m = Movie('Casablanca', 97, 102, 964000, 1300000)
	print(m.budget)
	m.rating = 100
	print(m.rating) 
	try:
	    m.rating = -1
	except ValueError:
	    print("Woops, negative value")

运行结果:

	964000
	100
	Woops, negative value

NonNegative是一个描述符对象，因为它定义了\_\_get\_\_，\_\_set\_\_或\_\_delete\_\_方法。

Movie类现在看起来非常清晰。我们在类的层面上创建了4个描述符，把它们当做普通的类属性。显然，描述符在这里为我们做非负检查。

### 访问描述符

当解释器遇到print(m.buget)时，它就会把budget当作一个带有\_\_get\_\_ 方法的描述符，调用Movie.budget.\_\_get\_\_方法并将方法的返回值打印出来，而不是直接传递m.budget来打印。这和你访问一个property相似，Python自动调用一个方法，同时返回结果。

\_\_get\_\_接收2个参数：一个是点号左边的实例对象（在这里，就是m.budget中的m），另一个是这个实例的类型（Movie）。在一些Python文档中，Movie被称作描述符的所有者（owner）。如果我们需要访问Movie.budget，Python将会调用Movie.budget.\_\_get\_\_(None, Movie)。可以看到，第一个参数要么是所有者的实例，要么是None。这些输入参数可能看起来很怪，但是这里它们告诉了你描述符属于哪个对象的一部分。当我们看到NonNegative类的实现时这一切就合情合理了。

### 对描述符赋值

当解释器看到m.rating = 100时，Python识别出rating是一个带有\_\_set\_\_方法的描述符，于是就调用Movie.rating.\_\_set\_\_(m, 100)。和\_\_get__一样，\_\_set\_\_的第一个参数是点号左边的类实例（m.rating = 100中的m）。第二个参数是所赋的值（100）。

### 删除描述符

为了说明的完整，这里提一下删除。如果你调用del m.budget，Python就会调用Movie.budget.\_\_delete\_\_(m)。

### NonNegative类是如何工作的？

带着前面的困惑，我们终于要揭示NonNegative类是如何工作的了。每个NonNegative的实例都维护着一个字典，其中保存着所有者实例和对应数据的映射关系。当我们调用m.budget时，\_\_get\_\_方法会查找与m相关联的数据，并返回这个结果（如果这个值不存在，则会返回一个默认值）。\_\_set\_\_采用的方式相同，但是这里会包含额外的非负检查。我们使用WeakKeyDictionary来取代普通的字典以防止内存泄露——我们可不想仅仅因为它在描述符的字典中就让一个无用的实例一直存活着。

使用描述符会有一点别扭。因为它们作用于类的层次上，每一个类实例都共享同一个描述符。这就意味着对不同的实例对象而言，描述符不得不手动地管理不同的状态，同时需要显式的将类实例作为第一个参数准确传递给\_\_get\_\_、\_\_set\_\_以及\_\_delete\_\_方法。

我希望这个例子解释清楚了描述符可以用来做什么——它们提供了一种方法将property的逻辑隔离到单独的类中来处理。如果你发现自己正在不同的property之间重复着相同的逻辑，那么本文也许会成为一个线索供你思考为何用描述符重构代码是值得一试的。

### 秘诀和陷阱

#### 把描述符放在类的层次上（class level）

为了让描述符能够正常工作，它们必须定义在类的层次上。如果你不这么做，那么Python无法自动为你调用\_\_get\_\_和\_\_set\_\_方法。

	class Broken(object):
	    y = NonNegative(5)
	    def __init__(self):
	        self.x = NonNegative(0)  # NOT a good descriptor
	
	b = Broken()
	print("X is %s , Y is %s" % (b.x, b.y))

运行结果：

	X is <__main__.NonNegative object at 0x0000025D941AF358> , Y is 5

可以看到，访问类层次上的描述符y可以自动调用__get__。但是访问实例层次上的描述符x只会返回描述符本身，真是魔法一般的存在啊。

#### 确保实例的数据只属于实例本身 

你可能会像这样编写NonNegative描述符：
	
	class BrokenNonNegative(object):
	    def __init__(self, default):
	        self.value = default
	 
	    def __get__(self, instance, owner):
	        return self.value
	 
	    def __set__(self, instance, value):
	        if value < 0:
	            raise ValueError("Negative value not allowed: %s" % value)
	        self.value = value
	 
	class Foo(object):
	    bar = BrokenNonNegative(5) 
	 
	f = Foo()
	try:
	    f.bar = -1
	except ValueError:
	    print "Caught the invalid assignment"
运行结果：
	 
	Caught the invalid assignment

这么做看起来似乎能正常工作。但这里的问题就在于所有Foo的实例都共享相同的bar，这会产生一些令人痛苦的结果：

	class Foo(object):
	    bar = BrokenNonNegative(5) 
	 
	f = Foo()
	g = Foo()
	 
	print ("f.bar is %s\ng.bar is %s" % (f.bar, g.bar))
	print "Setting f.bar to 10"
	f.bar = 10
	print ("f.bar is %s\ng.bar is %s" % (f.bar, g.bar))
运行结果：

	f.bar is 5
	g.bar is 5

	Setting f.bar to 10

	f.bar is 10
	g.bar is 10

这就是为什么我们要在NonNegative中使用数据字典的原因。\_\_get\_\_和\_\_set\_\_的第一个参数告诉我们需要关心哪一个实例。NonNegative使用这个参数作为字典的key，为每一个Foo实例单独保存一份数据。这就是描述符最令人感到别扭的地方（坦白的说，我不理解为什么Python不让你在实例的层次上定义描述符，并且总是需要将实际的处理分发给__get__和__set__。这么做行不通一定是有原因的）。

#### 注意不可哈希的描述符所有者

NonNegative类使用了一个字典来单独保存专属于实例的数据。这个一般来说是没问题的，除非你用到了不可哈希（unhashable）的对象：

	class MoProblems(list):  #you can't use lists as dictionary keys
	    x = NonNegative(5)
	 
	m = MoProblems()
	print (m.x)  # TypeError: unhashable type: 'MoProblems'

因为MoProblems的实例（list的子类）是不可哈希的，因此它们不能为MoProblems.x用做数据字典的key。有一些方法可以规避这个问题，但是都不完美。最好的方法可能就是给你的描述符加标签:

	class Descriptor(object):
	    def __init__(self, label):
	        self.label = label
	
	    def __get__(self, instance, owner):
	        print('__get__', instance, owner)
	        return instance.__dict__.get(self.label)
	
	    def __set__(self, instance, value):
	        print('__set__')
	        instance.__dict__[self.label] = value
	
	class Foo(list):
	    x = Descriptor('x')
	    y = Descriptor('y')
	
	f = Foo()
	f.x = 5
	print(f.x)
运行结果：

	__set__
	__get__ [] <class '__main__.Foo'>
	5

这种方法依赖于Python的方法解析顺序（即，MRO）。我们给Foo中的每个描述符加上一个标签名，名称和我们赋值给描述符的变量名相同，比如x = Descriptor('x')。之后，描述符将特定于实例的数据保存在f.\_\_dict\_\_['x']中。这个字典条目通常是当我们请求f.x时Python给出的返回值。然而，由于Foo.x 是一个描述符，Python不能正常的使用f.\_\_dict\_\_['x']，但是描述符可以安全的在这里存储数据。只是要记住，不要在别的地方也给这个描述符添加标签:

	class Foo(object):
	    x = Descriptor('y')
	
	
	f = Foo()
	f.x = 5
	print(f.x)
	print(f.__dict__)

	f.y = 4  # oh no!
	print(f.__dict__)
	print(f.x)

运行结果：

	__set__
	__get__ <__main__.Foo object at 0x000001F2C89C0CF8> <class '__main__.Foo'>
	5
	{'y': 5}

	__get__ <__main__.Foo object at 0x000001F2C89C0CF8> <class '__main__.Foo'>
	4
	{'y': 4}
因为 f.x 描述符实际上就是 f.\_\_dict\_\_['y']；f.y=4 相当于给实例f声明一个y属性，相当于f.\_\_dict\_\_['y']=4，这样就改变了f.x的值。

### 给描述符动态添加回调函数

	from weakref import WeakKeyDictionary
	class CallbackProperty(object):
	    """A property that will alert observers when upon updates"""
	
	    def __init__(self, default=None):
	        self.data = WeakKeyDictionary()
	        self.default = default
	        self.callbacks = WeakKeyDictionary()
	
	    def __get__(self, instance, owner):
	        if instance is None:
	            return self
	        return self.data.get(instance, self.default)
	
	    def __set__(self, instance, value):
	        for callback in self.callbacks.get(instance, []):
	            # alert callback function of new value
	            callback(value)
	        self.data[instance] = value
	
	    def add_callback(self, instance, callback):
	        """Add a new function to call everytime the descriptor within instance updates"""
	        if instance not in self.callbacks:
	            self.callbacks[instance] = []
	        self.callbacks[instance].append(callback)
	
	
	class BankAccount(object):
	    balance = CallbackProperty(0)
	
	
	def low_balance_warning(value):
	    if value < 100:
	        print("the value: %d  must be lgt 100"%value)
	
	
	ba1 = BankAccount()
	ba2 = BankAccount()
	BankAccount.balance.add_callback(ba1, low_balance_warning)
	
	ba1.balance = 5000
	ba2.balance = 6000
	print("ba1 is %s and ba2 is %s" % (ba1.balance,ba2.balance))
	ba1.balance = 99
	ba2.balance = 88
	print("ba1 is %s and ba2 is %s" % (ba1.balance,ba2.balance))

运行结果：

	ba1 is 5000 and ba2 is 6000
	the value: 99  must be lgt 100
	ba1 is 99 and ba2 is 88

这是一个很有吸引力的模式——我们可以动态为某一个或多个实例添加自定义回调函数，执行一些特殊的处理逻辑，而且完全无需修改这个类的代码。

但是我们是如何做到的呢？当我们试图访问它们时，描述符总是会调用__get__。就好像add_callback方法是无法触及的一样！其实关键在于利用了一种特殊的情况，即，当从类的层次访问时，__get__方法的第一个参数是None。

# 补充—属性查询优先级

1. \_\_getattribute\_\_()方法无条件调用
2. 数据描述符
3. 实例属性
4. 非数据描述符
5. \_\_getattr\_\_()方法

一个类，如果只定义了 \_\_get\_\_() 方法，而没有定义 \_\_set\_\_()或 \_\_delete\_\_() 方法则这个类是非数据描述符，反之则为数据描述符。
	class Desc(object):
	    def __init__(self, name):
	        self.name = name
	        print("__init__(): name = ", self.name)
	
	    def __get__(self, instance, owner):
	        print("__get__() ...")
	        return self.name
	
	    def __set__(self, instance, value):
	        self.value = value
	
	
	class DescTest(object):
	    _x = Desc('x')
	    def __init__(self, x):
	        self._x = x
	t=DescTest(10)
	print(t._x)
	print(t.__dict__)
运行结果：

	__init__(): name =  x
	__get__() ...
	x
	{}
因为Desc类是一个数据描述符，优先级高于实例属性。当我们去掉 \_\_set\_\_()方法时结果就不一样了：
	class Desc(object):
	    def __init__(self, name):
	        self.name = name
	        print("__init__(): name = ", self.name)
	
	    def __get__(self, instance, owner):
	        print("__get__() ...")
	        return self.name
	
	
	class DescTest(object):
	    _x = Desc('x')
	    def __init__(self, x):
	        self._x = x
	t=DescTest(10)
	print(t._x)
	print(t.__dict__)
运行结果：

	__init__(): name =  x
	10
	{'_x': 10}
实例属性的优先级高于非数据描述符。

# 结语

希望你现在对描述符是什么和它们的适用场景有了一个认识，前进吧骚年。

参考文章：[解密 Python 的描述符（descriptor）](http://python.jobbole.com/81899/)