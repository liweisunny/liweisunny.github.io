---
title: python黑魔法——迭代器与生成器
date: 2017-12-18 17:44:06
tags: python基础
toc: true
categories: Python
---
在了解Python的数据结构时，容器(container)、可迭代对象(iterable)、迭代器(iterator)、生成器(generator)、列表/集合/字典推导式(list,set,dict comprehension)众多概念参杂在一起，难免让初学者一头雾水，我将用一篇文章试图将这些概念以及它们之间的关系捋清楚。
![](https://i.imgur.com/RLHjcHO.png)

# 容器(container)

容器是一种把多个元素组织在一起的数据结构，容器中的元素可以逐个地迭代获取，可以用in, not in关键字判断元素是否包含在容器中。通常这类数据结构把所有的元素存储在内存中（也有一些特例，并不是所有的元素都放在内存，比如迭代器和生成器对象）在Python中，常见的容器对象有：

list, deque, ….

set, frozensets, ….

dict, defaultdict, OrderedDict, Counter, ….

tuple, namedtuple, …

str

# 