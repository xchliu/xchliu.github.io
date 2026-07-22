---
layout: default
title: "Python collections"
date: 2016-04-11
categories: Python
---

python中常用的数据类型：dict，set，list，tuple。在一些场景下总是会遇到使用的类型不能满足的情况，事实上，python还提供一些特定功能的数据类型：

## [](#Counter)Counter
dict的子类，用于对迭代对象的计数，同时也具备dict类型大多数方法，而且可以方便地转换成常用的数据类型,比如在搜索结果做类别计数的时候。

- 初始化

```
>>> from collections import Counter
>>> c=Counter()
# list
>>> c=Counter([1,2])
>>> c
Counter({1: 1, 2: 1})
# 直接初始化计数
>>> c=Counter({'a':2,'b':3})
>>> c
Counter({'b': 3, 'a': 2})
>>> c=Counter(a=2,b=10)
>>> c
Counter({'b': 10, 'a': 2})
```

- 操作

  和dict一样，通过key访问Counter对象的值，也可以修改/删除其计数值。

```
>>> c['b']=20
>>> c
Counter({'b': 20, 'a': 2})
>>> c.elements()
<itertools.chain object at 0x11027f290>
>>> list(c.elements())
['a', 'a', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b', 'b']
```

- 方法

elements `无序`返回所有元素，如果你想生成一个特定的list，这个方法很方便。

```
>>> c=Counter(a=2,b=3,d=1)
>>> list(c.elements())
['a', 'a', 'b', 'b', 'b', 'd']
```

- most_common 按计数返回top n

```
>>> c.most_common(1)
[('b', 3)]
```

- subtract 计数减法,也就是对应key的值减法，结果可以为负，可以key值不对称。

```
>>> d=Counter(a=1,b=1,d=1,e=2)
>>> c.subtract(d)
>>> c
Counter({'b': 1, 'a': 0, 'd': -1, 'e': -2})
```

- update 同dict，不过Counter update的是计数值。

```
>>> c
Counter({'b': 1, 'a': 0, 'd': -1, 'e': -2})
>>> c.update(['a','d'])
>>> c
Counter({'a': 1, 'b': 1, 'd': 0, 'e': -2})
```
*其他 dict类包含的属性            

- 运算            

加 对应key的计数相加，保留正数

- 减 对应key的计数相加，保留正数

- 交集 对应key的计数最小值，保留正数

- 并集 对应key的计数最大值，保留正数 

```
>>> c
Counter({'a': 1, 'b': 1, 'd': 0, 'e': -2})
>>> d
Counter({'e': 2, 'a': 1, 'b': 1, 'd': 1})
  >>> c+d
Counter({'a': 2, 'b': 2, 'd': 1})
>>> c-d
Counter()
>>> d-c
Counter({'e': 4, 'd': 1})
>>> c&d
Counter({'a': 1, 'b': 1})
>>> c|d
Counter({'e': 2, 'a': 1, 'b': 1, 'd': 1})
```

## [](#deque)deque
对于常用的list类型，对列表中的元素进行插入，删除，以及添加，会涉及到列表的元素移动或者列表大小的变化。并且list只能从队列末端进行append和pop。

双端队列deque 提供线程安全，内存性能，可从队列两端进行元素插入和删除的高速队列。首尾的快速操作可以做为动态的消息队列。

deque的方法和list类似，多了对端的操作函数，比如append，appendleft。其中maxlen属性用来控制队列最大长度，如果对一个满队列进行插入，则会通过推出对端相应数量的元素来完成。

常用操作：

```
>>> from collections import deque
>>> d=deque([1,2,3])
>>> d
deque([1, 2, 3])
>>> d.append('a')
>>> d
deque([1, 2, 3, 'a'])
>>> d.appendleft('a')
>>> d
deque(['a', 1, 2, 3, 'a'])
>>> d[2]
2
>>> d.reverse()
>>> d
deque(['a', 3, 2, 1, 'a'])
>>> d.rotate(1)
>>> d
deque(['a', 'a', 3, 2, 1])
>>> d.pop()
1
>>> d.popleft()
'a'
>>>
```

## [](#defaultdict)defaultdict
又一个dict的子类，如果你想对字典的values()有一个初始值或者是类型，defaultdict就派上用场了。

```
>>> from collections import defaultdict
>>> d = defaultdict(list)
>>> d['newkey'].append('list')
>>> d
defaultdict(<type 'list'>, {'newkey': ['list']})
```
如果访问defaultdict的key不存在，初始化指定的函数或者类就会被调用，比如上面的就会调用list()，并将返回值作为该key的初始值，于是就可以直接对元素调用list的方法。初始化的方法可以是int，list，set ..等类型，也可以是自定义的函数。

## [](#namedtuple)namedtuple
只能使用下标访问的tuple总是各种麻烦，而且必须加上注释才知道tuple的元素的含义。namedtuple就是为解决这个问题：

- 属性访问

- 自解释

- 快速转换

```
>>> from collections import namedtuple
>>> Servers = namedtuple('servers', ['ip','port'])
>>> s = Servers('1.1.1.1', 22)
>>> s
servers(ip='1.1.1.1', port=22)
>>> s.ip
'1.1.1.1'
>>> s._asdict()
OrderedDict([('ip', '1.1.1.1'), ('port', 22)])
>>> s._replace(port=23)
servers(ip='1.1.1.1', port=23)
>>> s._fields
('ip', 'port')
```

namedtuple的函数都是下划线命名，为的避免和键值冲突，也就要去namedtuple的键值不能是下划线开头。

实例的初始化可以用_make函数，也可以直接`**dict`来完成。

### [](#OrderedDict)OrderedDict
dict的子类，有序字典，顺序取决于元素的插入顺序，需要注意的是如果改变已插入的元素，该元素会的顺序将移到最后。

```
>>> from collections import OrderedDict
>>> d = OrderedDict()
>>> d
OrderedDict()
>>> d['2'] =1
>>> d['1'] =2
>>> d
OrderedDict([('2', 1), ('1', 2)])
>>>
>>> a ={'7':4,'4':3}
>>> d.update(a)
>>> d
OrderedDict([('2', 1), ('1', 2), ('4', 3), ('7', 4)])    
>>> e = OrderedDict(a)
>>> e
OrderedDict([('4', 3), ('7', 4)])
```
和update一样，如果初始的时候指定字典，得到的有序字典顺序不一定和指定的字典一样，因为python中字典是无序的。

- [多元组Multiset](https://en.wikipedia.org/wiki/Multiset)

- [python data types](https://docs.python.org/2.7/library/datatypes.html?highlight=data%20type)

- [python collections](https://docs.python.org/2.7/library/collections.html?highlight=collections#module-collections)
