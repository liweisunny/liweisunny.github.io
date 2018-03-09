---
title: Django安装及基本操作指南
date: 2018-02-08 18:01:33
tags: Django环境部署
toc: true
categories: Django
---
**以下操作均在Windows下**

# 配置python环境

参考：[python 环境部署-Windows](http://waisunny.com/2017/11/16/python-%E7%8E%AF%E5%A2%83%E9%83%A8%E7%BD%B2-Windows/)

# 安装PiP

参考：[setuptools和pip安装教程](http://waisunny.com/2017/11/17/setuptools%E5%92%8Cpip%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/)

<!--more-->
# 安装Django

使用PiP安装django：

安装最新的版本的 Django 命令如下：  
 
	pip install django  

安装指定版本的 Django 命令如下：

	pip install django==1.10.3  
在这里推荐使用指定版本的django来安装

安装成功以后的截图：

![](https://i.imgur.com/wZFz8AH.png)

使用 **pip show** 命令来查看当前安装的Django版本：

	pip show django

使用 **pip uninstall** 命令来 卸载 Django：

	pip uninstall django 

# 使用Django

## 创建项目:

使用管理工具 django-admin.py创建项目，命令：

	django-admin.py startproject HelloWorld  
说明："HelloWord"为项目名称，项目路径为当前dos窗口打开的路径

创建完成后我们可以查看下项目的目录结构：

	|-- HelloWorld  
	|   |-- __init__.py  
	|   |-- settings.py  
	|   |-- urls.py  
	|   `-- wsgi.py  
	`-- manage.py 

目录说明：

1. HelloWorld: 项目的容器。
2. manage.py: 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
3. HelloWorld/__init__.py: 一个空文件，告诉 Python 该目录是一个 Python 包。
4. HelloWorld/settings.py: 该 Django 项目的设置/配置。
5. HelloWorld/urls.py: 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
6. HelloWorld/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。
## 启动服务器

接下来我们进入 HelloWorld 项目目录输入以下命令：

	python manage.py runserver  

说明：以上命令执行后会在你本地启动服务，默认端口8000，浏览器输入127.0.0.1:8000或者你的ip+8000就可以看到如下界面：
![](https://i.imgur.com/aVIilKb.jpg)

## 视图和Url配置

参考：[视图和URL配置](http://www.runoob.com/django/django-first-app.html ) 

## 模型

Django规定，如果要使用模型，必须要创建一个app。我们使用以下命令创建一个名为 myapp的 app:

	django-admin.py startapp myapp  

目录结构：

	HelloWorld  
	|-- myapp  
	|   |-- __init__.py  
	|   |-- admin.py  
	|   |-- models.py  
	|   |-- tests.py  
	|   `-- views.py  

我们修改myapp/models.py文件代码：

	HelloWorld/TestModel/models.py: 文件代码：  
 
	from django.db import models  
	class Test(models.Model):  
	    name = models.CharField(max_length=20)  


以上的类名代表了数据库表名，且继承了models.Model，类里面的字段代表数据表中的字段(name)，数据类型则由CharField（相当于varchar）、DateField（相当于datetime）， max_length 参数限定长度。

### 激活模型

接下来在settings.py中找到INSTALLED_APPS这一项，如下：

	INSTALLED_APPS = (  
	    'django.contrib.admin',  
	    'django.contrib.auth',  
	    'django.contrib.contenttypes',  
	    'django.contrib.sessions',  
	    'django.contrib.messages',  
	    'django.contrib.staticfiles',  
	    'myapp',               # 注册app  
	) 

 然后，执行一下命令：

	$ python manage.py makemigrations myapp #让 Django 知道我们在我们的模型有一些变更  
	$ python manage.py migrate myapp #创建表结构 

### 模型变更步骤

记住实现模型变更的三个步骤（每一步都是必须的）：

1. 修改你的模型（在models.py文件中）。
2. 运行python manage.py makemigrations ，为这些修改创建迁移文件,如图所示就是生成的迁移文件
3. 运行python manage.py migrate ，将这些改变更新到数据库中。

说明：可以在执行：

	python manage.py migrate
之前执行：

	python manage.py sqlmigrate myapp1 000 #这个会输出给我们迁移行为将会执行哪些SQL语句；

执行：

	python manage.py check #会检查项目中模型是否存在问题。

<font color=red>**执行命令时可以指定app名称，这样就是作用于某一个 app,推荐这样做。**</font>

 
# 总结

这些都是最基本的操作，入门必备 ，希望对向我一样的初学者有帮助！！！