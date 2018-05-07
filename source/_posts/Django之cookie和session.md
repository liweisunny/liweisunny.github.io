---
title: Django之cookie和session
date: 2018-02-22 18:01:33
tags: Django基础
toc: true
categories: Django
---
再说Django中的cookie和session是如何使用之前，我们先来介绍一下cookie和session是什么，明确一点，cookie和session是通用的，不仅仅局限于Django等python web框架，在其他语言，框架中均有应用。

本文先介绍cookie和session的原理，然后介绍它们在Django中是如何使用以及如何配置的。

cookie与session的详细介绍：[http://waisunny.com/2018/02/21/cookie%E4%B8%8Esession%E5%89%96%E6%9E%90/](http://waisunny.com/2018/02/21/cookie%E4%B8%8Esession%E5%89%96%E6%9E%90/)

<!--more-->

# Django实现的COOKIE

## 获取cookie
	request.COOKIES['key']
	request.get_signed_cookie(key, default=RAISE_ERROR, salt='', max_age=None)
	    #参数：
	        default: 默认值
	           salt: 加密盐
	        max_age: 后台控制过期时间

## 设置cookie
	rep = HttpResponse(...) 或 rep ＝ render(request, ...) 或 rep ＝ redirect()
	 
	rep.set_cookie(key,value,...)
	rep.set_signed_cookie(key,value,salt='加密盐',...)　

### 参数说明：

	def set_cookie(self,  key,                 键
	　　　　　　　　　　　　 value='',            值
	　　　　　　　　　　　　 max_age=None,        超长时间
	　　　　　　　　　　　　 expires=None,        超长时间
	　　　　　　　　　　　　 path='/',           Cookie生效的路径，浏览器只会把cookie回传给带有该路径的页面，这样可以避免将
	                                         cookie传给站点中的其他的应用。 / 表示根路径，根路径的cookie可以被任何url的页面访问
	　　　　　　　　　　　　 
	                      domain=None,         Cookie生效的域名， 你可用这个参数来构造一个跨站cookie。
	                                          如， domain=".example.com"， 所构造的cookie对下面这些站点都是可读的：
	                                          www.example.com 、 www2.example.com 和an.other.sub.domain.example.com 。
	                                          如果该参数设置为 None ，cookie只能由设置它的站点读取。
	
	　　　　　　　　　　　　 secure=False,        如果设置为 True ，浏览器将通过HTTPS来回传cookie。

	　　　　　　　　　　　　 httponly=False       只能http协议传输，无法被JavaScript获取（不是绝对，底层抓包可以获取到也可以被覆盖）
	　　　　　　　　　　): pass

## 删除cookie
	response.delete_cookie("cookie_key",path="/",domain=name)

## 补充
由于cookie存储在客户端，所以我们也可以通过js和jquery操作cookie

	<script src='/static/js/jquery.cookie.js'>
	 
	</script> $.cookie("key", value,{ path: '/' });

# Django实现的SESSION

## 基本操作

	1 设置Sessions值
	          request.session['session_name'] ="admin"
	2 获取Sessions值
	          session_name = request.session["session_name"]
	3 删除Sessions值
	          del request.session["session_name"]
	4 检测是否存在session值
	          if "session_name" is request.session :
	
	5 get(key, default=None)
	 
		fav_color = request.session.get('fav_color', 'red')
	 
	6 pop(key)
	 
		fav_color = request.session.pop('fav_color')
	 
	7 keys()
	 
	8 items()
	 
	9 setdefault()
	 
	10 flush() 删除当前的会话数据并删除会话的Cookie。这用于确保前面的会话数据不可以再次被用户的浏览器访问
	           例如，django.contrib.auth.logout() 函数中就会调用它。
	 
	11 用户session的随机字符串
	   request.session.session_key
	  
	   将所有Session失效日期小于当前日期的数据删除
	   request.session.clear_expired()
	  
	   检查 用户session的随机字符串 在数据库中是否
	   request.session.exists("session_key")
	  
	   删除当前用户的所有Session数据
	   request.session.delete("session_key")
	  
	   request.session.set_expiry(value)
	   	* 如果value是个整数，session会在些秒数后失效。
	   	* 如果value是个datatime或timedelta，session就会在这个时间后失效。
	   	* 如果value是0,用户关闭浏览器session就会失效。
	   	* 如果value是None,session会依赖全局session失效策略。


## 流程解析图
![](https://i.imgur.com/POtapVA.png)


## session存储的相关配置
### 数据库配置（默认）

	Django默认支持Session，并且默认是将Session数据存储在数据库中，即：django_session 表中。
	  
	配置 settings.py
	  
	    SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
	      
	    SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
	    SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
	    SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
	    SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
	    SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
	    SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
	    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
	    SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）

### 缓存配置

	配置 settings.py
	  
	    SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
	    SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
	  
	  
	    SESSION_COOKIE_NAME ＝ "sessionid"                        # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
	    SESSION_COOKIE_PATH ＝ "/"                                # Session的cookie保存的路径
	    SESSION_COOKIE_DOMAIN = None                              # Session的cookie保存的域名
	    SESSION_COOKIE_SECURE = False                             # 是否Https传输cookie
	    SESSION_COOKIE_HTTPONLY = True                            # 是否Session的cookie只支持http传输
	    SESSION_COOKIE_AGE = 1209600                              # Session的cookie失效日期（2周）
	    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                   # 是否关闭浏览器使得Session过期
	    SESSION_SAVE_EVERY_REQUEST = False                        # 是否每次请求都保存Session，默认修改之后才保存

### 文件配置

	配置 settings.py
	  
	    SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
	    SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()        
	    SESSION_COOKIE_NAME ＝ "sessionid"                          # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
	    SESSION_COOKIE_PATH ＝ "/"                                  # Session的cookie保存的路径
	    SESSION_COOKIE_DOMAIN = None                                # Session的cookie保存的域名
	    SESSION_COOKIE_SECURE = False                               # 是否Https传输cookie
	    SESSION_COOKIE_HTTPONLY = True                              # 是否Session的cookie只支持http传输
	    SESSION_COOKIE_AGE = 1209600                                # Session的cookie失效日期（2周）
	    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                     # 是否关闭浏览器使得Session过期
	    SESSION_SAVE_EVERY_REQUEST = False                          # 是否每次请求都保存Session，默认修改之后才保存


# 登录应用
每当我们使用一款浏览器访问一个登陆页面的时候，一旦我们通过了认证。服务器端就会发送一组随机唯一的字符串（假设是123abc）到浏览器端，这个被存储在浏览端的东西就叫cookie。而服务器端也会自己存储一下用户当前的状态，比如login=true，username=hahaha之类的用户信息。但是这种存储是以字典形式存储的，字典的唯一key就是刚才发给用户的唯一的cookie值。那么如果在服务器端查看session信息的话，理论上就会看到如下样子的字典

	{'123abc':{'login':true,'username:hahaha'}}

因为每个cookie都是唯一的，所以我们在电脑上换个浏览器再登陆同一个网站也需要再次验证。那么为什么说我们只是理论上看到这样子的字典呢？因为处于安全性的考虑，其实对于上面那个大字典不光key值123abc是被加密的，value值{'login':true,'username:hahaha'}在服务器端也是一样被加密的。所以我们服务器上就算打开session信息看到的也是类似与以下样子的东西

	{'123abc':dasdasdasd1231231da1231231}

## 案例
项目地址：[https://github.com/liweisunny/DjangoBasis/blob/master/Demo_Login/login/views.py](https://github.com/liweisunny/DjangoBasis/blob/master/Demo_Login/login/views.py)

# 总结
django中操作cookie和seesion的方法差不多就这些，自己写个小案例练习一下，明白cookie和session的使用机制才是王道。