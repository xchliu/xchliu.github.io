---
layout: default
title: "Python One line"
date: 2016-04-14
categories: Python
---

很多时候，我们更愿意将代码写成一行，显得更简洁。Python一行代码可以干很多事，有的是为了方便实用，也有的仅仅是为了能在一行代码搞定一堆事情。实现的方式一般是使用lamdba，推导式或者调用一些包。

- 三元条件表达式  a=1 if 1=2 else 0一行搞定条件判断，变量赋值。

- 推导式

```
>>> [ x for x in range(5)]
[0, 1, 2, 3, 4]
>>> [ x for x in range(10) if x%2]
[1, 3, 5, 7, 9]
```

- Httpserver

文件共享利器

```
python -m SimpleHTTPServer 8000
```
列表碾平：

```
flatten = lambda x: [y for l in x for y in flatten(l)] if type(x) is list else [x]
```
[Python Oneliner](http://python-oneliner.readthedocs.org/en/latest/)

[Powerful Python One-Liners](https://wiki.python.org/moin/Powerful%20Python%20One-Liners)
