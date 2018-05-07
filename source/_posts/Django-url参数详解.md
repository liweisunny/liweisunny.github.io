---
title: Django url参数详解
date: 2018-02-10 18:01:33
tags: Django基础
toc: true
categories: Django
---
Django项目中的url.py文件中用于配置请求路由的url()函数可以接受4个参数：

	urlpatterns = [
		url(regex, view, kwargs=None, name=None),
	]


两个是必须的：regex和view参数；

两个是可选的：kwargs和name参数；

下面来看一下这些参数是干啥的。

<!--more-->

# regex参数
“regex”参数url模式的匹配。

Django会把请求的url从第一条正则表达式开始匹配，直到找到一个可以匹配上的正则表达式为止。

需要注意的是，正则表达式并不查询get参数、post参数、域名，例如，当请求指向https://www.example.com/myapp/?page=3， URLconf还是会查找**myapp/**。

另外，如果您需要正则表达式的帮助，请参阅re模块的文档，然而，实际上，您不需要是正则表达式的专家，因为您只需要知道如何捕获简单的模式。

实际上，复杂的正则表达式的查找性能会很差，所以你可能不应该依靠正则表达式的全部功能。

最后，一个性能说明：这些正则表达式是第一次加载URLconf模块时被编译。它们超级快（只要查找不是太复杂，如上所述）。

# view参数
当正则表达式被匹配，view函数就会被调用。而且HttpRequest对象会作为第一个参数，其它被正则表达式捕获的值会作为其它参数传入到函数中。

# kwargs参数
其它的参数(即要传入到view函数中的参数)以字典的形式传入。

# name参数
给你的url起个别名能让你在其它地方明确引用它。这个强大的功能可以让你在修改URL模式的时候只需修改单个文件。

创建一个django项目后，添加一个模板页：

![](https://i.imgur.com/iq5xDcA.png)

url配置如下：

![](https://i.imgur.com/SCmF2Cw.png)

我们计算加法的时候用的是 **'add/4/5/'**这个url ，后来需求发生变化，url改成 **'/4_add_5/'**，但在html页中，视图函数代码中有很多地方都写死了**'add/4/5/'**。如果这样写死url，会使得在改了url（正则）后，模板（template)，视图(views.py，用于跳转)，模型(models.py，可以用用于获取对象对应的地址）用了此url的，都得进行相应的更改，修改的代价很大，一不小心，有的地方没改过来，就不能用了。那么有没有更优雅的方式来解决这个问题呢？当然答案是肯定的:

	<a href="{% url 'add2' 4 5 %}">link</a>

上面的代码渲染成最终的url是：

	<a href="/add/4/5/">link</a>

使用：

	{% url 'add2' 4 5 `%}
替换写死的url。 其中 'add2' 就是url配置中name参数的值，后面 跟的两个参数是固定死的参数值。

当 urls.py 进行更改，前提是不改 name（这个参数设定好后不要轻易改），获取的url也会动态地跟着变，比如改成：

	url(r'^new_add/(\d+)/(\d+)/$', calc_views.add2, name='add2')

注意看重点add变成了new_add，但是后面的name='add2'没改，这时 

	{% url 'add2' 4 5 %}
就会渲染对应的网址成 **'/new_add/4/5/'**用在 views.py 或 models.py 等地方的 **reverse**函数，同样会根据 name 对应的url获取到新的网址。

# 生成url的函数reverse

	from django.core.urlresolvers import reverse  # django 1.4.x - django 1.10.x  
	from django.urls import reverse  # django 1.10.x 新的，更加规范了  
	reverse('add2', args=(4,5))  # u'/add/4/5/'  
	reverse('add2', args=(444,555))  # u'/add/444/555/'

reverse 接收 url函数中的 name参数的值作为第一个参数，我们在代码中就可以通过 reverse() 来获取对应的网址（这个网址可以用来跳转，也可以用来计算相关页面的地址），只要对应的url的name不改，就不用改代码中的网址。




