# 第一部分 序幕

## 第1章、Python数据模型

什么是Python数据模型？

对属性访问机制的描述

### 什么是特殊方法

特殊方法，又称双下方法，是元对象协议。

元对象指对构建语言本身来讲很重要的对象，协议可以看作是**接口**，也就是说，元对象协议是对象模型的同义词，意思是构建核心语言的API。

### 如何实现特殊方法

False下例用到了namedtuple：一个快速构建只有少数属性没有方法的对象（在下一章有介绍）。

```python
import collections

Card = collections.namedtuple("Card", ['rand', 'suit'])

def spades_high(card):
    suit_values = {"♣": 3, "♦": 2, "♤": 1, "♧": 0}
    # 根据当前卡片的rank值，返回他在ranks中的位置 
    # list.index(x)从列表中找出某个值第一个匹配项的索引位置
    rank_value = FrenchDeck.ranks.index(card.rank)
    return rank_value * len(card) + suit_values[card.suit]


class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = '♣ ♦ ♤ ♧'.split()

    def __init__(self):
        # suit先运行rank再运行
        self._card = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
	
    # 实现len()
    def __len__(self):
        return len(self._card)

    # 实现切片、迭代、随机生成、
    def __getitem__(self, item):
        return self._card[item]

    
if __name__ == '__main__':
    deck = FrenchDeck()
    # 随机抽一个
    print(choice(deck))
    # 支持切片 打印出3♣ 3♦ 3♤ 3♧
    print(deck[1::13])
    
    # 迭代
    for card in deck:
        print(card)
    # 反向迭代,reversed 函数返回一个反转的迭代器
    for card in reversed(deck):
        print(card)
        
    #     
	for card in sorted(deck, key=spades_high):
    	print(card)
```



##### 实现len()

```python
def __len__(self):
    return len(self._card)
```



##### 实现迭代

```python
def __getitem__(self, item):
    return self._card[item]
```

原理：

在用for..in.. 迭代自定义类时，解释器首先会看类有没有实现`__iter__`迭代器协议，`__iter__`只需要返回一个实现了 `__next__()` 方法的迭代器对象即可，比如

```python
def __iter__(self):
    return self
def __next__(self):
    return 1
```

这样，如果对这个对象进行迭代，就会不停地返回1。

如果迭代的对象，没有实现`__iter__`和`__next__`的迭代器协议，就会寻找`__getitem__`来迭代，如果也没找到，就会报错不是迭代对象。



##### 实现排序

- sort

  对list排序

- sorted(iterable, cmp=None, key=None, reverse=False)

  对可迭代对象iterable排序，key代表排序方式

```python
def spades_high(card):
    suit_values = {"♣": 3, "♦": 2, "♤": 1, "♧": 0}
    # 根据当前卡片的rank值，返回他在ranks中的位置 
    # list.index(x)从列表中找出某个值第一个匹配项的索引位置
    rank_value = FrenchDeck.ranks.index(card.rank)
    # 牌号*4 + 花色权重
    return rank_value * len(card) + suit_values[card.suit]

for card in sorted(deck, key=spades_high):
    print(card)
```



##### 实现字符串表示

对于自定义类来说，开发者打断点看一个指向自定义类的实例时，会看到调试器，形如`a = <__main__.Vector object at 0x00000197F398C2B0>`，但是如果你实现了以下两种方法中的一个，你就能在调试器看到更加友好的表示形式（所以说在控制台看到的实例的值，并不是类定义的那个`self`全部，只是一种界面友好的展示方式）。

- `__repr__()`

  给开发人员看的，用哪些关键属性来描述这个类

- `__str__()`

  区别不大，返回的更加友好，`__repr__`像是给开发人员看的。

优先实现`__repr__`，当解释器需要调用str，但又没有实现的时候，解释器会用`__repr__`替代。



##### 实现+*计算

- `__add__()`

  为类带来+运算，返回新的类型（中缀运算符的基本原则就是不改变操作对象）

- `__mul__()`

  为类实现*运算，返回新的类型

##### 实现布尔

默认情况下，类的实例总是被认定为真。

解释器首先去看有没有实现`__bool__()`方法。如果没有，则尝试调用`__len__()`，如果返回0则返回False，否则为Ture。

##### 双下方法汇总表



### 总结

在用for..in.. 迭代自定义类时，解释器首先会看类有没有实现`__iter__`迭代器协议，`__iter__`只需要返回一个实现了 `__next__()` 方法的迭代器对象即可。如果迭代的对象，没有实现`__iter__`和`__next__`的迭代器协议，就会寻找`__getitem__`来迭代，如果也没找到，就会报错不是迭代对象。

在用str()背后的原理：优先实现`__repr__`，当解释器需要调用str，但又没有实现的时候，解释器会用`__repr__`替代。



# 第二部分 数据结构

## 第2章 序列构成的数组

### 2.1 内置序列



可变序列：list、bytearray、array.array、collection.deque、memoryview

不可变序列：tuple、str、bytes

属性差异：

- 不可变序列：
  - `__getitem__`
  - `__contains__`
  - `__iter__`
  - `__reversed__`
  - index
  - count
- 可变序列（额外包括）：
  - insert
  - append
  - extend
  - pop
  - remove
  - `__iadd__`

还可以按照如下方式分类：

- 容器序列

  list、tuple、collections.deque

- 扁平序列

  str、bytes、bytearray、memoryview、array.array

当容器序列里面嵌套可变序列，很容易出现“意外”(后面会介绍)

### 2.2 列表推导和生成器表达式

#### 列表推导式

```python
symbols = '$%^&'
# 先运行for迭代，再进行判断，符合放到list
beyond_ascil = [ord(s) for s in symbols if ord(s)>50]

```

> 冷知识：
>
> [] 、{}、 ()中的代码可以忽略换行



#### 生成器表达式

> 是对包含 yield 关键字的生成器函数的语法糖写法，不理解没关系，后面讲。

虽然也可以用列表推导来初始化元组、数组或其他序列类型，但生成器表达式是更好的选择。

遵循迭代器协议，逐个产生元素，==可以节省内存==。

==把方括号换成圆括号==

```python
symbols = '$%^&'
# 先运行for迭代，再进行判断，符合放到list
beyond_ascil = (ord(s) for s in symbols if ord(s)>50)
```



### 2.3 元组

#### 常见用法

除了不可变的列表，元组还有很多妙用，比如说拆包的应用：

==平行赋值==

```python
tuple_a = (1, 2)

a, b = tuple_a 
#不使用中间变量赋值
a, b = b, a
```

> 对于不感兴趣的数据，可以用占位符`_`代替

==用作参数==

```python
# 此处会将tuple_a拆成元素个数个变量，而不是把tuple_a当作一个变量传进去
# 所以如果foo_a有多个位置参数，而tuple_a只有一个元素，则报错
foo_a(*tuple_a)
```

==处理剩余元素==

如下的例子，首先是一个平行赋值，然后将不确定数量的参数全部放到`rest`中，且rest是个list

```python
>>>a, b, *rest = range(5)
>>>a, b, rest
(0, 1, [2, 3, 4])

# 此处如果不加* 会报错
```

在一次平行赋值中，*只能用在一个变量前面，但是是可以出现在**任意位置**的：

```python
>>>a, *body, b, c = range(5)
```

==嵌套拆包==

```python
tuple_a = (1, 2, (3, 4))
a, b, (c,_) = tuple_a
>>>1,2,3

a, b, c = tuple_a
>>>1 2 (3, 4)

a, b, *c = tuple_a
>>>1 2 [(3, 4)]
```

此时，可得c=3，当然也可以用*c的方式获取剩余值，不过就是一个list：[(3,4)]



#### 具名元组namedtuple

collections.namedtuple是一个工程函数，快速构建只有少数属性没有方法的对象：

```python
Hero = collections.namedtuple("Heroo", ['blood', 'magic'])
galen = Hero(10, 20)
print(galen)

>>>Heroo(blood=10, magic=20)

#会报错AttributeError: can't set attribute
galen.blood = 22
```

> 注意：
>
> collections.namedtuple的第一个参数代表类的名称，而不是变量名代表类的名称。

> 用namedtuple创建的类的实例消耗的内存和元组一致，因为字段名存在对应的类里面。namedtuple创建的类的实例比普通实例小一些，因为不会用`__dict__`存放实例的属性。

具名元组需要两个参数（“类名”，“类的各个字段的名字”），后者可以是由**数个字符串**组成的可迭代对象或者是**空格分隔**的字段名组成的字符串。

```python
# 真正的类名是namedtuple的第一个参数，City是这个类的便利贴，指向这个类
City = collections.namedtuple("City",'name country population coordinates')

# 因为Python支持一等函数，所以此处实测是可以传递函数的。。。
tokyo = City('Tokyo','Japan',36, (11,22) )
# 仍然具有元组的平行赋值特性
a, b, c, d = tokyo

>>>City(name='Tokyo', country='Japan', population=36, coordinates=(11, 22))
```

##### 具名元组常用属性

==_fields==

返回一个包含此类所有字段的元组

```python
tokyo._fields
>>>('name', 'country', 'population', 'coordinates')

# 这样也是可以的
City._fields
```



==_make()==

接受一个可迭代的对象，来生成一个实例

```python
City = collections.namedtuple("City",'name country population coordinates')

delhi_data = ('Delhi NCR', 'IN', 21, (22, 33))
delhi = City._make(delhi_data)

# 上面的写法等同于
delhi = City(*delhi_data)
```



==_asdict()==

将实例的内容更加友好的展示出来，返回一个collections.OrderedDict类型

```python
delhi._asdict()
>>>OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population', 21), ('coordinates', (22, 33))])
```



#### 元组VS列表

P27页



### 2.4 切片

切片和区间会忽略最后一个元素，==好处==如下：

- 快速看出切片\区间元素个数
- 明确开始截至信息时，快速计算长度
- 分割不重叠部分时，书写更加简便eg：l[:2]和l[2:]



#### 切片赋值

要给切片赋值，那么赋值语句的右侧必须是个可迭代对象，即使只有单独的值，也要进行转换。

#### 2.4.3 多维切片和省略

[] 运算符里还可以使用逗号，二维的 numpy.ndarray 就可以用 a[i, 75j] 这种形式来获取，抑或是用 a[m:n, k:l] 的方式来得到二维切片。

要正确处理这种 [] 运算符的话，对象的特殊方法` __getitem__ `和 `__setitem__ `需要以元组的形式来接收 a[i, j] 中的索引。也就是说，如果要得到 a[i, j] 的 值，Python 会调用 `a.__getitem__((i, j))`。

Python 内置的序列类型都是一维的，因此它们只支持单一的索引，成对出现的索引是没有用的。 省略（ellipsis）的正确书写方法是三个英语句号（...），而不是 Unicdoe 码位 U+2026 表示的半个省略号（...）。省略在 Python 解析器眼 里是一个符号，而实际上它是 Ellipsis 对象的别名，而 Ellipsis 对象 又是 ellipsis 类的单一实例。 它可以当作切片规范的一部分，也可 以用在函数的参数清单中，比如 f(a, ..., z)，或 a[i:...]。在 NumPy 中，... 用作多维数组切片的快捷方式。如果 x 是四维数组， 那么 x[i, ...] 就是 x[i, :, :, :] 的缩写。

> 冷知识：
>
> 是的，你没看错，ellipsis 是类名，全小写，而它的内置实例写作 Ellipsis。这其实跟 bool 是小写，但是它的两个实例写作 True 和 False 异曲同工。

### 2.5 序列使用+*

+和*都是不修改原有对象，而是==构建一个新的==序列

```python
a = (3,4)
b = (5,6)
print(a+b)
>>>(3, 4, 5, 6)
```

==警告==：

如果在类似`a*n`这种语句中：a变量里的元素有，对其他可变对象的引用时，生成的结果中对于可变对象的引用，其实指向同一个：

```python
mylist=[[1,1]]
aa = mylist*3

print(aa)
>>>[[1, 1], [1, 1], [1, 1]]

# 可见aa列表有三个变量，但是三个变量指向了同一地址
aa[0][0]=2
print(aa)
>>>[[2, 1], [2, 1], [2, 1]]

# 但是我们可以替换掉其中一个,总结就是一定要把一个List元素看成一个整体
aa[0]=[2,1]
print(aa)
>>>[[2, 1], [1, 1], [1, 1]]

# 但是如果我们是这样创建aa,后续操作aa就如预期一样
# 造成这种区别的原因，应该是mylist*3 是将mylist的内存地址复制了三次，而下面的写法则是申请了三个内存地址
aa = [[1,1]] + [[1,1]] + [[1,1]]

# 以下代码仍能按预期操作，but Why？
# 因为row是局部变量，每次循环都会申请新的内存地址，效果等同于每次append([1,1])
aa = []
for _ in range(3):
    row = [1,1]
    aa.append(row)

    print(aa)
    aa[0][0]=2
    print(aa)
```



### 2.6 序列的增量赋值

即`+=`或者`*=`操作，原理是实现了`__iadd__`方法（就地加法），但是如果没有实现这个方法，就会退一步去寻找`__add__`

对于可变序列`a += b`，a会就地改动，但是如果a没有实现`__iadd__`的话，`a += b`就会变得跟`a = a + b`一样了：先计算`a + b`再赋给`a`。一般来说可变序列都实现了`__iadd__`方法，而不可变序列本身就不会有`__iadd__`方法。

>冷知识：
>
>我们知道运行以下代码是可以的：
>
>```python
>aa = (1,2,[3,4])
>aa[2].append(5)
>```
>
>但是如果是`+=`操作呢？
>
>```python
>aa = (1,2,[3,4])
>aa[2]+=[5]
>```
>
>结果是aa会增加，同时也会报错。。。

`*=`同理，不过是实现了`__imul__`



### 2.7 序列sort和sorted函数



###### list.sort

`list.sort`会==就地排序==，返回值为None。

> Python惯例：
>
> 如果一个方法对对象进行的是就地改动，那它就应该返回None，以让调用者知道传入的参数发生了改变，而且没有产生新的对象。eg：radom.shuffle
>
> 缺点：
>
> 因为返回值是None，所以无法实现连贯接口（fluent interface又称作链式调用）

###### sorted()

它接受任何形式的可迭代对象，新建一个列表作为返回值，

---

以上两个函数都接收参数：{reverse=False, key=None}

- reverse：

  - True

    被排序的序列以降序输出

  - False

    默认值

- key：

  传入一个回调函数，这个函数会被用在序列的每一个元素上，函数应用的结果是排序算法依据的关键字。

  默认值是：恒等函数（identity function）

### 2.8 basic管理已排序列

==暂时跳过，看不懂==

用bisect管理已排序的序列，主要包含两个函数：

- bisect
- insort



### 2.9 除了列表？

#### 2.9.1 数组array

列表十分灵活好用，但是有些业务场景你可能更需要array.array：

- 只包含数字的列表
- 读取和存入文件

array创建时需要一个类型码，表示这个array要存放怎样的数据类型

| 类型码 | C 类型             | Python 类型  | 以字节表示的最小尺寸 |
| :----- | :----------------- | :----------- | :------------------- |
| `'b'`  | signed char        | int          | 1                    |
| `'B'`  | unsigned char      | int          | 1                    |
| `'u'`  | wchar_t            | Unicode 字符 | 2                    |
| `'h'`  | signed short       | int          | 2                    |
| `'H'`  | unsigned short     | int          | 2                    |
| `'i'`  | signed int         | int          | 2                    |
| `'I'`  | unsigned int       | int          | 2                    |
| `'l'`  | signed long        | int          | 4                    |
| `'L'`  | unsigned long      | int          | 4                    |
| `'q'`  | signed long long   | int          | 8                    |
| `'Q'`  | unsigned long long | int          | 8                    |
| `'f'`  | float              | float        | 4                    |
| `'d'`  | double             | float        | 8                    |

示例：

```python
floats = array('d', (random() for i in range(10**7)))
```

#### 2.9.2 内存视图

pass

#### 2.9.3 NumPy、SciPy和Pandas

pass

#### 2.9.4 双向队列

```python
deque = collections.deque([1, 2, 3, 4])
deque.append(5)
# 行为很像list，不过可以从左边操作
deque.appendleft(0)
print(deque)

>>>deque([0, 1, 2, 3, 4, 5])
```

应用场景：

- ==先进先出==

  ```python
      from collections import deque
      dq = deque(range(10), maxlen=10)
      
   >>>deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
  
  	# 队列最右边的n个元素，会被移到队列的左边
  	dq.rotate(3)
   >>>deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
      
      # 当n<0时，最左边的n个元素移动到右边
      dq.rotate(-4)
   >>>deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
      
      # 对一个len(dq)=dq.maxlen的队列做左添加的时候，尾部元素会被删除
      dq.appendleft(-1)
  
  ```

deque对两端的操作做了优化，且线程安全，但是中间删除元素操作会慢。

Python其他队列库：

- queue

  maxlen满员时会锁住，直到移除成员腾出位置，适合控制活跃线程数量

- multiprocessing

  任务管理用的

- asyncio

  异步编程的任务管理

- heapq

  把可变序列当作堆队列或者优先队列用



### 2.10总结

当容器序列存储可变对象时，要格外小心，可能会搞出一些“意外”

元组拆包很好用

具名元组很好用，但是没用起来

+和*都是不修改原有对象，而是构建一个新的序列

增量赋值对于可变序列会就地修改，对不可变序列生成新的

sort会就地排序，返回None



## 第3章 字典和集合

### 3.1 泛映射类型

什么是映射类型？

映射类型也可被称做哈希表，提供了存取数据项及其键和值的方法。比如：dict和dict的变种：defaultdict、OrderedDict。映射类型的key必须是**可散列**的数据类型

> 可以简单理解为“映射类型”是dict的超类，是一种泛指和统称

什么是可散列的数据类型？

如果一个对象是可散列的，那么在这个对象的生命周期中，它的散列值是不可变的，这就要求这个对象需要实现`__hash()__`方法。还要实现`__eq()__`方法，这样才能跟其他键作比较。如果两个可散列对象是相等的，那么他们的散列值一定是一样的。

原子不可变数据类型（str、bytes和数值类型）都是可散列类型



collections.abc模块中的Mapping和MutableMapping两个抽象基类为dict和其他类似的泛映射定义了接口

### 3.2 字典推导

第二章有列表推导和生成器表达式，Python2.7以后支持字典推导。

```python
DIAL_CODES = [(86, 'China'),(91,'India'),(1,'United States')]
country_code = {country: code for code, country in DIAL_CODES}

 >>>{'China': 86, 'India': 91, 'United States': 1}   

code_country = {code:country.upper() for country, code in country_code.items() if code>66}    

>>>{86: 'China', 91: 'India'}
```



### 3.3 常用映射方法

```python
# 移除所有元素
d.clear()

# 实现 k in d
d.__contains__(k)

# 返回k对应的值，如果没有则返回None或者default
d.get(k, default)

# 让d能够以d[k]的方式返回值,（里面的实现估计是d.get吧）
d.__getitem__(k)

# 将迭代器it里的元素设置为映射里的键，如果initial有值，就当作key对应的默认值
# 第二个值的【】不是说传入列表，而是可选的意思，下同
d.fromkeys(it, [initial])

# 返回k对应的值，然后移除这个键值对，如果没有则返回None或者default
d.pop(k, [default])

# 返回k对应的值，如果没有则d[k]=default,然后返回default
d.setdefault(k, [default])

# m可以是映射或者是键值对迭代器，用来更新d对应的条目
d.update(m, [**kargs])

# 在__missing__函数中被调用的函数，用以给未找到的元素设置值
d.default_factory


```

#### update

update处理参数m的方式，跟自定义实现一个可迭代类异曲同工，都是典型的“鸭子类型”：

函数update首先检查m有没有实现keys方法，如果有就把m当作映射对象处理，否则会把m当作包含了（key，value）的迭代器。

```
dict_result.update({key:value})
```

#### ==setdefault==

```python
dict_a = {}
dict_a.setdefault("a",[]).append(5)
# 首先setdefault会查找key为a的值，没有时创建并赋值为空列表，并返回这个空列表。然后在这个空列表中append了一个数字
```

### 3.4 映射的弹性键查询

有时候，我们查询字典的某个key，即使不存在也不期望报错，期望能给我们一个默认值。有两个方法：

- collections.defaultdict

  额外提供的库，在实例化defaultdict时，需要给构造方法提供一个可调用对象，字典中找不到的值，就会调用这个对象，并返回其内容。当然除此以外它就是个dict

- 特殊方法`__missing__`

#### 3.4.1 defaultdict

```python
"""创建一个从单词到其出现情况的映射"""
import sys
import re
import collections

WORD_RE = re.compile(r'\w+')

# index就是我们的字典名，如果查不到key，就会通过调用list的__call__方法，返回一个空[]
# 如果创建的时候，我们没有指定default_factory,此处即list，那么碰到没有的key依然会触发KeyError
index = collections.defaultdict(list) 

with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start()+1
            location = (line_no, column_no)
            index[word].append(location) 
            
    # 以字母顺序打印出结果
    for word in sorted(index, key=str.upper):
        print(word, index[word])

```



警告：

default_factory的可调用对象，只会在`__getitem__` 中被调用，比如说`dd[k]`是在找不到的时候调用default_factory，但是`dd.get(k)`只会和普通dict一样返回None。

其实这背后完全归功于特殊方法`__missing__`



#### 3.4.2 \_\_missing__

所有的映射类型在处理找不到的键的时候，都会牵扯到 `__missing__` 方法。这也是这个方法称作“missing”的原因。虽然基类 dict 并没有定 义这个方法，但是 dict 是知道有这么个东西存在的。也就是说，如果 有一个类继承了 dict，然后这个继承类提供了 `__missing__` 方法， 那么在 `__getitem__` 碰到找不到的键的时候，Python 就会自动调用 它，而不是抛出一个 KeyError 异常。

 `__missing__` 方法只会被 `__getitem__` 调用（比如在表 达式 d[k] 中）。提供 `__missing__` 方法对 get 或者 `__contains__`（in 运算符会用到这个方法）这些方法的使用没 有影响。这也是我在上一节最后的警告中提到，defaultdict 中 的 default_factory 只对 `__getitem__` 有作用的原因。



下面我们来实现一个自定义dict，它能够将数字key，在查询不到的时候，转成str key来查询。

```python
class StrKeyDict0(dict): 
    
    # str类型不用看了直接报错，int的话转成str再调一遍get，它有可能再走到miss里来，此时说明str也没有，直接报错
    def __missing__(self, key):
        if isinstance(key, str): 
            raise KeyError(key)
        return self[str(key)] 
    
    # 会首先走这，有值直接返回，没有会先走__missing__
    def get(self, key, default=None):
        try:
            return self[key] 
        # 报错最终都在这儿，会返回None或者自定义缺省值
        except KeyError:
            return default 
        
    # 这是调用 in 关键字时走的方法，会判断是否直接包含，或者转换成str后是否包含  
    def __contains__(self, key):
		return key in self.keys() or str(key) in self.keys() 


```



为了保持一致性，`__contains__` 方法在这里也是必需的。这是因为 `k in d` 这个操作会调用它，但是我们从 dict 继承到的 `__contains__` 方法不会在找不到键的时候调用 `__missing__` 方 法。__`__contains__`__ 里还有个细节，就是我们这里没有用更具 Python 138 风格的方式——`k in my_dict`——来检查键是否存在，因为那也会导 致 `__contains__`被递归调用。为了避免这一情况，这里采取了更显 式的方法，直接在这个 `self.keys()` 里查询。



### 3.5 字典的变种

总结一下，标准库collections模块中，除了defaultdict之外的不同映射类型

| 类型        | 作用                               |
| ----------- | ---------------------------------- |
| defaultdict | 找不到key时也不会报错              |
| OrderedDict | 带顺序的dict                       |
| ChainMap    | 容纳多个dict，依次遍历             |
| Counter     | 计数                               |
| UserDict    | 标准dict，用来让用户继承写子类的。 |



1. collections.OrderedDict

   这个类在添加键的时候会保持顺序，因此迭代顺序总是保持一致。popitem方法默认删除并返回字典最后一个元素。

2. collections.ChainMap

   这个类型可以容纳不同的字典，在进行键值查找时，会被当作一个整体逐个查找，直到键被找到。

   所以可以用来参数的优先级查找

   ```python
   from collections import ChainMap
   
   
   defaults = {
       'color': 'red',
       'user': 'guest'
   }
   first = {
       'color': 'blue',
       'user': 'root'
   }
   
   combined = ChainMap(first, defaults)
   combined['color']
   
   >>>blue
   ```

   

3. collections.Counter

   `Counter`是一个简单的计数器，most_common(n)返回映射中最常用的n个键和他们的计数。例如，统计字符出现的个数：

   ```python
   from collections import Counter
   c = Counter()
   for ch in 'programming':
        c[ch] = c[ch] + 1
   
   c
   Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})
   
   c.update('hello') # 也可以一次性update
   Counter({'r': 2, 'o': 2, 'g': 2, 'm': 2, 'l': 2, 'p': 1, 'a': 1, 'i': 1, 'n': 1, 'h': 1, 'e': 1})
   
   c.most_common(5)
   [('r', 2), ('o', 2), ('g', 2), ('m', 2), ('l', 2)]
   ```

   

4. collections.UserDict

   标准dict，用来让用户继承写子类的。

### 3.6 子类化dict

> 为什么自定义映射类，以UserDict为基类，要比普通dict要方便？
>
> 因为dict某些方法的实现走了捷径，我们子类需要重写；而UserDict是纯python实现的。

UserDict并不是dict的子类，UserDict有一个data属性，是dict的实例，存放了UserDict最终存储的数据。

下面我们实现一个查询key时都转为字符串类型key的dict

```python
import collections

class StrKeyDict(collections.UserDict):
    
    def __missing__(self, key):
        # 你都找不到key了，还是字符串类型，说明是真的没有
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]
    
    # 存储的数据都在self.data里
    def __contains__(self, key):
        return str(key) in self.data
    
    def __setitem(self, key, item):
        self.data[str(key)] = item


```



### 3.7 不可变映射

自Python3.3，types模块引入了MappingProxyType类型。通过给这个类传入一个映射，返回一个只读的映射视图，eg：

```python
from types import MappingProxyType
d= {1:"A"}
d_proxy = MappingProxyType(d)

```

### 3.8 集合论

包括：可变的set，和它的不可变的姊妹frozenset。集合的本质是唯一对象的聚合，所以可用来去重。

```python
l = ['apple', 'egg', 'egg']
set(l)
>>>{'apple', 'egg'}
```

集合的元素必须是可散列的（原子不可变数据类型）：set是不可散列的，frozenset是可散列的，所以set可以包含frozenset。

中缀运算

```python
# 交集
a & b
# 差集
a - b
# 合集
a | b
```

#### 3.8.1 集合字面量

> 什么是字面量？
>
> 在计算机科学中，字面量（literal）是用于表达[源代码](https://baike.baidu.com/item/源代码/3969)中一个固定值的表示法（notation）。几乎所有[计算机编程语言](https://baike.baidu.com/item/计算机编程语言/5581937)都具有对基本值的字面量表示，诸如：[整数](https://baike.baidu.com/item/整数/1293937)、[浮点数](https://baike.baidu.com/item/浮点数/6162520)以及[字符串](https://baike.baidu.com/item/字符串/1017763)；而有很多也对[布尔类型](https://baike.baidu.com/item/布尔类型/9517367)和[字符](https://baike.baidu.com/item/字符/4768913)类型的值也支持字面量表示；还有一些甚至对[枚举类型](https://baike.baidu.com/item/枚举类型/2978296)的元素以及像[数组](https://baike.baidu.com/item/数组/3794097)、[记录](https://baike.baidu.com/item/记录/14312145)和对象等复合类型的值也支持字面量表示法。

要创建空集合，只能使用`set()`，`{}`只能创建空字典。

像{1, 2,  3}这种创建方法，比`set([1, 2,  3])`更加高效可读。

#### 3.8.2 集合推导

pass

#### 3.8.3 集合操作

P72页

### 3.9 dict和set的背后

Python如何用散列表实现dict类型的？

pass

==不要对字典同时进行迭代和修改==，如果想扫描并修改一个字典，最好分成两部：首先对字典进行迭代，得出需要处理的元素，将这些元素，和修改后的值放在一个新的字典中，迭代完成后用这个字典update原字典。

## 第4章 文本和字节序列

### 4.1 字符问题

字符串是一个字符序列；字符的定义是：**Unicode**字符；

Unicode [规范](https://www.unicode.org/) 旨在列出人类语言中用到的每个字符，并赋予每个字符唯一的编码。该规范持续进行修订和更新以添加新的语言和符号。

Unicode标准把字符的标识和具体字节的表述做了如下区分：

- 字符的表示，即码位，是0~1 114 111的数字（十进制），在Unicode标准中以4~6个十六进制的数字表示，而且加前缀“U+”。例如，字母A的码位是U+0041，欧元是U+20AC。
- 字符的具体表述取决于所用的编码。编码是在码位和字节序列之间转换时使用的算法。在UTF-8编码中，A（U+0041）的码位编码成单个字节`\x41`，而在UTF-16LE编码中编码成两个字节`\x41\x00`。

把码位转换成字节序列的过程是编码；把字节序列转换成码位的过程是解码。

```python
s = 'cafeの'
# 字符串可以被编码成字节序列
b = s.encode('utf8')
>>>b'cafe\xe3\x81\xae'

# 字节可以被解码成字符
b.decode('utf8')
```

> 提示：
>
> 可以把字节序列想象中晦涩难懂的机器语言，Unicode字符串是人类语言。把机器语言编译成人类语言就是decode解码，解密机器语言。



### 4.2 字节概要

bytes或bitearray对象 的各个元素是介于0~255（含）之间的整数。

> bytes可以理解为byte的组合，获取单个byte时，元素是一个0~255之间的数字，对bytes切片时，获取的还是bytes，看下面这个例子。

```python
cafe = bytes('cafeの',encoding='utf-8')
>>>b'cafe\xe3\x81\xae'

# 元素是整数
cafe[0]
>>>99

# bytes对象的切片还是bytes
cafe[:1]
>>>b'c'
```

> 冷知识：
>
> s[0]==s[:1]只对str这个序列类型成立，想想如果s=[0,1,2] 那么s[0]=0   s[:1]=[0]

Q：同样是获取变量的第一个元素，但是返回的数据却不同，为什么？

A：这些二进制整数，的字面量表示法，如果有对应的ASCII文本，那么这个数字就会显示对应的文本。这些二进制整数的值可能有下列三种不同的方式显式：

- 可打印的ASCII范围内的字节（空格到~），使用ASCII字符本身。
- 制表符、换行符、回车符和\对应的字节，使用转义序列\t、\n、\r、和\\\。
- 其他字节的值，使用十六进制转义序列（例如，\x00是空字节）

> 只能显示26个基本拉丁字母、阿拉伯数字和英式标点符号，因此只能用于显示现代美国英语
>
> 所以'cafeの'对应的bytes的类型，前四位都可以显示出来

pass

#### 结构体和内存视图

pass

### 4.3 基本的编解码器

UTF编码的设计目的就是处理每一个Unicode码位，常见的编码如下：

1. latin1（即iso8859_1）

   一种重要的编码，是其他编码的基础，例如cp1252和Unicode（注意，latin1与cp1252的字节值一样）

2. cp1252

   微软定制的latin1超集

3. cp437

4. gb2312

   编码简体中文的陈旧标准，亚洲使用较广泛的多字节编码

5. utf-8

   Web最常见的8位编码，与ASCII兼容

6. utf-16le

### 4.4 编码问题

一般有以下错误：

- UnicodeEncodeError

  把字符串转换成二进制序列时

- UnicodeDecodeError

  把二进制序列转换为字符串

- SyntaxError

  源码的编码与预期不符

> 冷知识：
>
> 当你不确定什么编码时，可以使用Chaedet库，可以判断用的那种编码格式



## 第5章 一等函数

一等对象的定义是什么？

- 在运行时创建
- 能赋值给变量或数据结构中的元素
- 能作为参数传给函数
- 能作为函数的返回结果

> 冷知识：
>
> `foo.__doc__`函数的这个属性会返回函数的描述，就是`"""describe"""`中的内容

### 5.1 把函数当对象

```python
def foo(a):
    return a

aa = map(foo, range(10))


```

`map(function, iterable1, iterable2...)`

map函数第一个参数接收一个函数，然后迭代后面传入的序列，对序列应用函数。

map函数返回的也是一个迭代器，需要遍历时才会真正运行函数。



### 5.2 高阶函数

什么是高阶函数？

函数的参数是函数，或者函数的返回值是函数，就是高阶函数。eg：map()、sorted()、filter()、 reduce、 apply（已过时）

简单介绍一下sorted()：

```python
    fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry']
    sorted(fruits, key=len)
	>>> ['fig', 'apple', 'cherry', 'raspberry', 'strawberry']
```

任何单参数的函数都能作为key的值。

```python
	# 按照元素的最后一个字符作为比较方式
    def foo(word):
    	return word[-1]
    sorted(fruits, key=foo)
    >>>['apple', 'fig', 'strawberry', 'cherry', 'raspberry']
    
```

因为列表推导和生成器表达式的存在，map和filter没那么重要了。eg：

```python
list(map(foo, range(6)))

[foo(n) for n in range(6)]
```

很明显，后者的可读性会更好。

### 5.3 匿名函数

用lambda关键字创建匿名函数。但lambda函数的定义体中不能赋值，也不能使用while和try等语句。

比如说：

```python
def foo(word):
    return word[-1]
sorted(fruits, key=foo)

# 可以简写为
sorted(fruits, key=lambda word:word[-1])

```

> 冷知识：
>
> lambda其实只是一个语法糖，与def语句一样，lambda表达式会创建函数对象。

### 5.4 可调用对象

如何判断一个对象能否调用？可以使用内置的`callable()`函数。返回True或者False

除了用户定义的函数，调用运算符（即`()`），还有其他七种对象可以调用：

1. **用户定义的函数**

​	使用`def`或者`lambda`表达式创建

2. **内置函数**

​	使用C语言（CPython）实现的函数，如`len`或者`time.strftime`

3. **内置方法**

​	使用C语言实现的方法，如dict.get

4. **方法**

​	在类的定义体中定义的函数

5. **类**

   调用类时会运行类的`__new__`方法创建一个实例，然后运行`__init__`方法，初始化实例，最后把实例返回给调用方。因为Python没有new运算符，所以调用类相当于调用函数。

   pass（有点复杂）

 6. **类的实例**

    如果类定义了`__call__`方法，那么它的实例可以作为函数调用

7. **生成器函数**

    使用yield关键字的函数或方法。调用生成器函数返回的是生成器对象。

### 5.5 自定义可调用类型

任何Python对象都可以表现的像函数，只要实现实例方法`__call__`。

看下面的例子

```python
class BingoCage:
    def __init__(self, items):
        self._item = list(items)
        random.shuffle(self._item)

    def pick(self):
        return self._item.pop()

    def __call__(self):
        return self.pick()


bingo = BingoCage(range(20))
bingo.pick()
>>>3
bingo()
>>>7
```



> 小知识：
>
> range()只允许传入一个int类型，然后在python3中返回一个迭代器对象，不想是迭代器对象的话，可以list(range(10))，这样就是完整的了。
>
> random.random()随机返回0~1之间的实数



### 5.6 ==函数内省==

把函数视为对象处理的另一个证据：运行时内省。

上面说过`__doc__`可以返回函数的注释，使用dir函数可以探知函数具有下述属性：

```python
def foo(word):
    return word[-1]

dir(foo)
>>>['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']

```

其中大部分属性都是Python对象共有的，此处重点关注把函数视作对象相关的几个属性：



- `__dict__`

  和用户自定义类时候一样，函数使用`__dict__`属性存储它的用户属性。这相当于一种基本形式的注解。一般来说为函数随意赋予属性不是很常见的做法，但是Django框架就这么做了。

> 冷知识：
>
> 以5.5中的BingoCage类和5.6的foo函数为例
>
> ```python
> BingoCage.__dict__
> >>>{'__module__': '__main__', '__init__': <function BingoCage.__init__ at 0x000002A3D3DD2378>, 'pick': <function BingoCage.pick at 0x000002A3D3DD2400>, '__call__': <function BingoCage.__call__ at 0x000002A3D3DD2488>, '__dict__': <attribute '__dict__' of 'BingoCage' objects>, '__weakref__': <attribute '__weakref__' of 'BingoCage' objects>, '__doc__': None}
> 
> bingo = BingoCage(list((2,2)))
> bingo.__dict__
> >>>{'_item': [2, 2]}
> 
> foo.__dict__
> >>>{}
> 
> foo.__dict__ = dict(aa=2)
> foo.__dict__
> >>>{'aa': 2}
> ```
>
> 不难看出，类和实例的`__dict__`方法返回的值不同，实例的`__dict__`存储了实例中的属性值。下面的例子也能看出，函数确实也是对象，也包含`__dict__`方法，只是我们几乎不会去用它。
>
> 其实上面的例子也能看出来，连我们写的类也是对象。。。



Q：都是返回对象的属性`__dict__`和`dir()`有什么区别吗？

A：

-  `dir()`函数会自动寻找一个对象的所有属性(包括从父类中继承的属性)，包括`__dict__`中的属性，返回的是一个list。

- `__dict__`并不包含其父类的属性，是`dir()`的子集，返回的是一个dict。



再来看一下函数和类相比，独占的属性有哪些？

```python
class C:pass
obj = C()
def foo():pass
#减号是中缀运算符，两个集合取差集，结果是函数独占的
sorted(set(dir(foo))-set(dir(obj)))
>>>['__annotations__', '__call__', '__closure__', '__code__', '__defaults__', '__get__', '__globals__', '__kwdefaults__', '__name__', '__qualname__']

# 结果是类独占的
sorted(set(dir(obj))-set(dir(foo)))
>>>['__weakref__']
```

详解函数独占的方法：

| 名称             | 类型           | 说明                                      |
| ---------------- | -------------- | ----------------------------------------- |
| \__annotations__ | dict           | 参数和返回值的注解                        |
| \__call__        | method-wrappe  | 实现 () 运算符；即可调用对象协议          |
| \__closure__     | tuple          | 函数闭包，即自由变量的绑定（通常是 None） |
| \__code__        | code           | 编译成字节码的函数元数据和函数定义体      |
| \__defaults__    | tuple          | 形式参数的默认值                          |
| \__get__         | method-wrapper | 实现只读描述符协议（参见第 20 章）        |
| \__globals__     | dict           | 函数所在模块中的全局变量                  |
| \__kwdefaults__  | dict           | 仅限关键字形式参数的默认值                |
| \__name__        | str            | 函数名称                                  |
| \__qualname__    | str            | 函数的限定名称，如 Random.choice          |



### 5.7 函数传参类型

Python的参数处理机制极为灵活，可以概括为：

- 位置参数（又叫定位参数）

  即根据位置确定这个参数，对应函数的哪个参数

- 关键字参数

  即根据关键字直接就能确定

  

这两个组合会衍生出各种类型的参数处理机制，包括：

- 默认参数（位置参数给予默认值）
- 可变参数（某个参数前加`*`，这个参数就变成可变参数）
- 可变关键参数（某个参数前加`**`，这个参数就变成可变关键参数）

- 命名关键字参数（参数中加`*`，`*`后面的都会成为命名关键字参数）

我们先从简单的开始：

1. **位置参数**

   ```python
   def foo(a):
       return a 
   foo(5)
   >>>5
   ```

   非常简单

   **默认参数**

   ```python
   def foo(a=5):
       return a 
   foo()
   >>>5
   ```

   但是默认参数和位置参数在一起时，必须位置参数在默认参数前面

   ```python
   # 会报错，提示“默认参数后面跟随非默认参数”
   def foo(a=5,b):
       return a 
   
   # 正确
   def foo(b,a=5):
       return a 
   ```

   默认参数还必须指向一个==原子不可变数据类型==

   后面又说明

2. **关键字参数**

   也是非常简单

   ```python
   def foo(a):
       return a 
   foo(a=5)
   >>>5
   ```

   

3. **可变参数**

   位置参数的变体，不再强制声明有几个参数（Python的包容性），就可以使用可变参数。甚至可以不传参数。传入的参数会在函数体中自动组装为一个tuple。

   ```python
   def foo(*args):
       return args
   # 直接传入了两个值
   foo(1,2)
   # 会把传入的值组装成一个元组处理
   >>>(1, 2)
   
   # 如果传入的是一个元组呢？
   foo((1,2))
   # 仍然看作一个整体，组装成一个元组处理
   # 为什么只有一个元素的元组后面会跟一个逗号，因为不跟逗号，python会把括号当作运算符中的括号处理
   >>>((1, 2),)
   
   # 如果你传入一个可迭代的，但是不想当作整体看待呢？等价于第一种写法
   foo(*(1,2))
   >>>(1, 2)
   
   ```

   > 冷知识：
   >
   > 一定要区别对待调用函数时的`*`和函数定义中的`*`，它俩不是必须成对出现的。比如有个函数定义为`def foo(a, b, c)`，那么传参数的时候，你完全可以写成`foo(*(1, 2, 3))`。
   >
   > 出现在函数定义体中的`*`，是告诉我们，这个函数接收无数个位置参数。
   >
   > 出现在调用中的`*`，是一种拆包，展开可迭代对象，映射到参数

4. ==**可变关键字参数**==

   关键字参数的变体，不再强制声明有哪些关键字参数（Python的包容性）。

   既然有组装成tuple的，那自然有组装成dict的，在定义函数体时，在参数名前面加`**`，就会让传入的0个或多个关键字参数组装成一个dict

   ```python
   def foo_dict(**kwargs):
       return kwargs
   
   foo_dict(aa=2)
   >>>{'aa': 2}
   ```

   Q：那么，我可以直接传一个字典吗？

   A：当然不可以，调用`foo_dict()`函数时，接收的是关键字参数，你可以传字典，但是这个字典得有key啊。不然的话，你就需要在dict前加`**`，代表对字典拆包，这样一个字典就变成了一组关键字参数了。

   

   Q：如果我传的可变关键字参数中，既有前面以显式写出来的关键字，又有未知的关键字会发生什么？

   A：显式写出来的关键字仍然会赋值到到已经写出来的，然后剩下未知的会放在一个dict中

   

   Q：如果我传的可变关键字参数中，既有前面以显式写出来的关键字，且前面也传了关键字参数会发生什么？

   A：会报错，提示`foo() got multiple values for keyword argument '参数'`

   

5. **仅限关键字参数**

   这个是Python3新增的，什么意思呢？就是说：只能通过关键字参数传值，且不可缺省。使用方法如下：

   ```python
   def foo_key(*, a, b):
       return a, b
   foo_key(a=1,b=2)
   >>> (1, 2)
   
   # 当然我们可以用**拆一个字典，但是值必须对应
   dict_key = dict(a=1,b=3)
   foo_key(**dict_key)
   >>>(1, 3)
   
   ```

   Q：为什么这样呢？

   A：虽然书中没有解释，但是根据笔者的观察，当你不指定关键字参数，即用位置参数传值时，比如`foo_key(1, 2, 3)`，不论多少都会被`def foo_key(*, a, b)` 中的第一个\*捕获，但\*后面没有变量名，也就是虽然捕获但是并没有保存，当你用关键字参数时，就会显式的赋值给函数了。（笔者猜测，并没有考据来源）

下面看一个比较灵活的实例：

```python
def tag(name, *content, cls=None, **attrs):
    """生成一个或多个HTML标签"""
    if cls is not None:
        attrs['class'] = cls
    if attrs:
        attr_str = ''.join(' %s="%s"' % (attr, value)
                           for attr, value
                           in sorted(attrs.items()))
    else:
        attr_str = ''
    if content:
        return '\n'.join('<%s%s>%s</%s>' %
                         (name, attr_str, c, name) for c in content)
    else:
        return '<%s%s />' % (name, attr_str)

```

函数`tag()`的name参数可以是位置参数也可以是关键字参数，`*content`因为有`*`号，所以是可变参数，`cls`因为在可变参数后面，所以是仅限关键字参数，最后`attrs`是可变关键字参数。

```python

>>> tag('br') ➊
'<br />'

>>> tag('p', 'hello') ➋
'<p>hello</p>'

>>> print(tag('p', 'hello', 'world'))
<p>hello</p>
<p>world</p>

>>> tag('p', 'hello', id=33) ➌
'<p id="33">hello</p>'

>>> print(tag('p', 'hello', 'world', cls='sidebar')) ➍
<p class="sidebar">hello</p>
<p class="sidebar">world</p>

>>> tag(content='testing', name="img") ➎
'<img content="testing" />'

>>> my_tag = {'name': 'img', 'title': 'Sunset Boulevard',
... 'src': 'sunset.jpg', 'cls': 'framed'}

>>> tag(**my_tag) ➏
'<img class="framed" src="sunset.jpg" title="Sunset Boulevard" />'

```



❶ 传入单个定位参数，生成一个指定名称的空标签。

❷ 第一个参数后面的任意个参数会被 *content 捕获，存入一个元组。

❸ tag 函数签名中没有明确指定名称的关键字参数会被 \**attrs 捕 获，存入一个字典。

❹ cls 参数只能作为关键字参数传入。 

❺ 调用 tag 函数时，即便第一个定位参数也能作为关键字参数传入。 

❻ 在 my_tag 前面加上 \**，字典中的所有元素作为单个参数传入，同名键会绑定到对应的具名参数上，余下的则被 **attrs 捕获。



### 5.8 获取参数信息

函数内省到底应用场景是什么？我们以HTTP 微框架 Bobo为例

```python
import bobo
@bobo.query('/')
def hello(person):
 return 'Hello %s!' % person

```

装饰器会在后面介绍，`bobo`会内省函数`hello()`，发现它需要一个名为`person`的变量。它是如何做到的？

我们知道函数其实也是对象，独占`__defaults__`属性，值是一个元组，存放了定位参数和关键字参数的默认值。仅限关键字的默认值在独占属性`__kwdefaults__`中。

但是上面的属性存的都是默认值，那参数名称呢？在`__code__`属性中，它的值是一个code对象引用，自身也有很多属性（即可以`foo.__code__.co_varnames`这样）。

**详解code属性**

```python
def clip(text, max_len=80):
    """在max_len前面或后面的第一个空格处截断文本
     """
    end = None
    if len(text) > max_len:
        space_before = text.rfind(' ', 0, max_len)
        if space_before >= 0:
            end = space_before
        else:
            space_after = text.rfind(' ', max_len)
            if space_after >= 0:
                end = space_after
     if end is None: # 没找到空格
        end = len(text)
     return text[:end].rstrip()

```

提取函数的参数信息：

```python
# 元组，定位参数和关键字参数的默认值
clip.__defaults__
>>>(80,)


clip.__code__
<code object clip at 0x...>

# 返回元组，获取函数的参数，还有函数中的变量。像是“**kwargs”，会返回kwargs
clip.__code__.co_varnames
>>>('text', 'max_len', 'end', 'space_before', 'space_after')

# int，获取函数有几个参数，像是“**kwargs”，算是一个
clip.__code__.co_argcount
>>>2
```

但是还是太复杂，我们需要先通过`foo.__code__.co_argcount`获取参数数量，再通过`foo.__code__.co_varnames`根据数量截取，这一点也不Python。还好我们有更好的方法：inspect模块

```python
from inspect import signature

sig = signature(foo_dict)
str(sig)
>>>(text, max_len=80)

for name, param in sig.parameters.items():
	print(param.kind, ':', name, '=', param.default)
    
>>>POSITIONAL_OR_KEYWORD : text = <class 'inspect._empty'>
POSITIONAL_OR_KEYWORD : max_len = 80

```

完美，且一致。

kind属性的值是_ParmeterKind类中的5个值之一，如下：

- POSITIONAL_OR_KEYWORD

  可以通过定位参数和关键参数传入的形参（占多数）

- VAR_POSITIONAL

  定位参数元组

- VAR_KEYWORD

  关键字参数字典

- KEYWORD_ONLY

  仅限关键字参数

- POSITIONAL_ONLE

  仅限定位参数

#### 5.8.1 ==校验参数==

inspect.Signature对象有个bind方法，它可以把任意个参数绑定到签名的形参上，所用的规则与实参到形参的匹配方式一样。框架可以使用这个方法在真正调用函数前验证参数。

以5.7节的`tag()`函数为例，校验传给tag函数的参数：

```python
import inspect

sig = inspect.signature(tag)
my_tag = {'name': 'img', 'title': 'Sunset Boulevard',
          'src': 'sunset.jpg', 'cls': 'framed'}
bound_args = sig.bind(**my_tag)
for name, value in bound_args.arguments.items():
    print(name, '=', value)
    
del my_tag['name']
# 此时会报错
bound_args = sig.bind(**my_tag)
>>>TypeError: missing a required argument: 'name'



```



### 5.9 函数注解

Q：什么是函数注解？

A：你们不是一直说「动态类型一时爽，代码重构火葬场」，那我给你加个函数注解吧

```python
# 重写5.8的函数
def clip(text:str, max_len:'int > 0'=80) -> str:
    """在max_len前面或后面的第一个空格处截断文本
    """
    end = None
    if len(text) > max_len:
        space_before = text.rfind(' ', 0, max_len)
        if space_before >= 0:
            end = space_before
        else:
            space_after = text.rfind(' ', max_len)
            if space_after >= 0:
                end = space_after
     if end is None: # 没找到空格
        end = len(text)
     return text[:end].rstrip()

```

函数声明的各个参数，可以在`:`之后增加注解表达式。如果参数有默认值，注解放在参数名和`=`号之间。

如果想要注解返回值，在`)`和`:`之间添加`->`和表达式。

==注解不会做任何处理==，只是存储在函数的`__annotations__`属性（dict）中。Python不做检查，不强制，不验证，啥也不做。那有啥用？给**IDE、框架和装饰器**用。

如何使用这些注解元数据：

pass

### 5.10 函数式编程

Q：什么是函数式编程？

A：面向数学的抽象（即表达式），有引用透明、没有副作用、并行不死锁的优点



Python不是为了编程函数式编程语言，但是在`operator`和`functools`等包的支持下，也可以函数式编程。

#### 5.10.1 operator模块

pass

#### 5.10.2 functools.partial

pass

### 5.11 本章小结

pass



## 第6章 一等函数实现设计模式

在《设计模式：可复用面向对象软件的基础》书中共有23种设计模式，其中有16种在动态语言中“不见或者简化了”。

Norvig建议在有一等函数的语言中重新看待“*策略*”“*命令*”“模板方法”和“*访问者模式*”。通常我们可以把这些模式中设计的某些类的实例替换成简单的函数，从而减少样本代码。本章将使用函数对象重构“策略”模式，以及简化“命令”模式。

### 6.1 重构策略模式

能够用函数重构策略模式须满足：

- 具体的策略不需要维护内部状态（策略类不需要属性），只是处理上下文中的数据。

#### 6.1.1 经典策略模式

*定义一系列算法，把它们一一封装起来，并且使它们可以相互替换。本模式使得算法可以独立于使用它的客户而变化。*

电商领域有个功能明显的“策略”模式，即有多个优惠规则且只能选一个优惠时，选择最佳优惠规则。

涉猎的专有名词有：

- *上下文*

  把一些计算委托给实现不同算法的可互换组件，它提供服务。在电商示例中，上下文就是*order*，它会根据不同算法计算促销折扣。

- *策略*

  实现不同算法的组件共同的接口。

- *具体策略*

  “策略”的具体子类。

下面是用Python实现经典策略模式。在这个示例中，实例化订单之前，系统会以某种方式选择一种促销折扣策略，然后把它传给Order构造方法。具体怎么选择策略，不再这个模式的职责范围内

```python
from abc import ABC, abstractmethod
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')

class LineItem:
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
		self.price = price
	def total(self):
		return self.price * self.quantity
    
class Order: # 上下文
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion
        
    def total(self):
		if not hasattr(self, '__total'):
			self.__total = sum(item.total() for item in self.cart)
        return self.__total
    
    # 计算金额，按照传进来的策略方法
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount
    
    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

    
    
class Promotion(ABC) : # 策略：抽象基类
    @abstractmethod
    def discount(self, order):
	"""返回折扣金额（正值）"""
    
class FidelityPromo(Promotion): # 第一个具体策略
    """为积分为1000或以上的顾客提供5%折扣"""
    def discount(self, order):
		return order.total() * .05 if order.customer.fidelity >= 1000 else 0
    
class BulkItemPromo(Promotion): # 第二个具体策略
    """单个商品为20个或以上时提供10%折扣"""
    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity >= 20:
            discount += item.total() * .1
        return discount
    
class LargeOrderPromo(Promotion): # 第三个具体策略
    """订单中的不同商品达到10个或以上时提供7%折扣"""
    def discount(self, order):
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
        	return order.total() * .07
        return 0

```

> 把一个类定义成抽象基类最简单的方式就是继承abc.ABC，然后抽象方法用@abstractmethod装饰，函数体不用写东西，pass也不用写。



使用Order类示例

```python
    joe = Customer('John Doe', 0)
    ann = Customer('Ann Smith', 1100)
    # 购物车
    cart = [LineItem('banana', 4, .5),
            LineItem('apple', 10, 1.5),
            LineItem('watermellon', 5, 5.0)]
    
    # 最后一个参数，是传进去具体的策略
    joe_order = Order(joe, cart, FidelityPromo())
    joe_due = joe_order.due()
    
    ann_order = Order(ann, cart, FidelityPromo())
    ann_due = ann_order.due()

```



#### 6.1.2 函数实现策略模式

在6.1.1示例中，每个具体的策略都是一个类，且只定义了一个方法`discount`。每个策略实例也没有状态（实例属性）。它们看起来就是普通的函数，下面的示例把具体策略换成了简单的函数，而且去掉了抽象类`Promo`。

```python
from collections import namedtuple
Customer = namedtuple('Customer', 'name fidelity')

class LineItem:
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price
    def total(self):
    	return self.price * self.quantity
    
class Order: # 上下文
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion	
        
    def total(self):
        if not hasattr(self, '__total'):
        	self.__total = sum(item.total() for item in self.cart)
        return self.
    
    def due(self):
        if self.promotion is None:
        	discount = 0
        else:
        	discount = self.promotion(self) 
        return self.total() - discount
    
    # 返回类的描述时，会计算共计花费和优惠后计费
    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

def fidelity_promo(order): 
	"""为积分为1000或以上的顾客提供5%折扣"""
	return order.total() * .05 if order.customer.fidelity >= 1000 else 

def bulk_item_promo(order):
    """单个商品为20个或以上时提供10%折扣"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
        discount += item.total() * .1
    return discount

def large_order_promo(order):
    """订单中的不同商品达到10个或以上时提供7%折扣"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
    	return order.total() * .07
	return 0




```



使用Order类示例

```python
    joe = Customer('John Doe', 0)
    ann = Customer('Ann Smith', 1100)
    # 购物车
    cart = [LineItem('banana', 4, .5),
            LineItem('apple', 10, 1.5),
            LineItem('watermellon', 5, 5.0)]
    # 注意，函数不带括号
    joe_order = Order(joe, cart, fidelity_promo)
    joe_due = joe_order.due()
    
    ann_order = Order(ann, cart, fidelity_promo)
    ann_due = ann_order.due()


```



#### 6.1.3 选择最佳策略

```python
promos = [fidelity_promo, bulk_item_promo, large_order_promo]

def best_promp(order):
    """选择可用最佳策略
    """
	return max(promo(order) for promo in promos)
```

我们把具体策略的函数放到列表promos中，然后列表推导，选出最大的。

习惯函数是一等对象后，自然而然就会构建这种存储函数的数据结构。

#### 6.1.4 找出模块全部策略

很明显，在6.1.3中我们需要手动维护promos列表，这一点也不Python，更好的做法呢？

- `globals()`
- 策略放在一个模块中，然后使用高阶内省函数`inspect()`

1. `globals()`

   在Python中，模块也是一等对象，而且标准库提供了处理模块的函数，比如`globals`函数：

   `globals()`返回一个字典，表示当前的全局符号表，这个符号表始终针对当前模块（对函数或方法来说，是指定义他们的模块，而不是调用他们的模块）。

   使用`globals()`自动找到所有的`_promos`函数：

   ```python
    # globals()返回一个字典,必须用列表推导式写，不然报错，原因暂时不懂pass
    promos = [globals()[name] for name in  globals() 
              if name.endswith('_promos') and name != 'best_promp']
   
   ```



2. `inspect()`

   `inspect.getmembers`函数用于获取对象（这里是promotions模块，模块也是对象）的属性，可选的可以传入一个参数`inspect.isfunction`，这样就可以筛选对象中的函数。

   在使用这个方法之前，我们需要在一个单独的模块中保存所有的策略，把`best_promp`排除在外（感觉也可以写在多个模块中，你调几次不就行了）。

   ```python
   # 先要导入你的模块
   import promotions as pro
   
   # 实测这个不用列表推导式不会报错
   promos = [func for name, func in inspect.getmembers(pro, inspect.isfunction)]
   
   ```

   

> 冷知识：
>
> 可以用`globals()`或者`inspect()`内省当前或者其他模块的函数或者类

但是，和传统的策略模式相比，我们都是对代码隐形假设这是符合约定的，比如说promotions模块不会出现其他函数，所以6.1.4中的方法只是强调模块内省的一种用途，而不是“策略”模式的完整方案，更严格的方法应该是后面介绍的“装饰器”模式。



### 6.2“命令”模式

> 命令模式是回调机制面向对象的替代品

Q：何为命令模式？

A：简单来说“黑话模式”，是用来解耦“调用操作对象”的调用者，和“提供实现对象”的接收者。比如说我们去餐厅吃饭时的顾客（调用者）和厨师（实现者）。

Q：这个模式是如何解耦的？

A：在顾客（调用者）和厨师（实现者）之间，增加一个command对象（服务员），让他实现只有一个方法（execute）的接口（将厨师会做的菜概括成菜单）：调用实现者（厨师）中的方法执行所需的操作（要做哪些菜）。这样调用者（顾客）就无需了解接收者的接口（厨师是怎么做菜的），而且不同的实现者（厨师）可以适应不同的Command子类（同一份菜单同一套黑话）。

Q：用Python实现此模式有何不同？

A：我们可以不用为厨师提供一个command对象实例，而是直接给它一个函数。或者呢我们可以实现Command类的`__call__`方法，这样我们就可以直接调用Command类的实例：可调用对象command()了，例如：

```python
class MacroCommand:
    """一个执行一组命令的命令"""
    def __init__(self, commands):
    	self.commands = list(commands) # ➊
        
    def __call__(self):
        for command in self.commands: # ➋
        	command()



```

Q：复杂的命令模式包含“取消”等操作，Python如何实现？

A：如果是MacroCommand这种写法，再加行为就行了，如果是一等对象的写法，可以用闭包保存函数内部状态。



### 6.3 本章小结

1. 很多情况下，在Python中使用一等函数或者是可调用对象实现回调更自然。

2. 把一个类定义成抽象基类最简单的方式就是继承abc.ABC，然后抽象方法用@abstractmethod装饰，函数体不用写东西，pass也不用写。
3. 可以用`globals()`或者`inspect()`内省当前或者其他模块的函数或者类。
4. 对接口编程，而不是对实现编程。
5. 优先使用对象组合，而不是类继承。



## 第7章 函数装饰器和闭包

Q：什么是函数装饰器？

A：用于在“源码”中标记函数，以某种方式增强函数的行为。依靠闭包实现的。

> 闭包还是回调式异步编程和函数式编程风格的基础。

### 7.1 装饰器基础知识

装饰器是可调用的对象（实现了`__call__`的一切对象），其参数是另一个函数（被装饰的函数）。

装饰器可能会处理被装饰的函数，然后把它返回，或者将其替换成另一个函数或者可调用对象。

```python
@decorate
def target():
    print('running target()')

# 上面的代码等价于
def target():
    print('running target()')
target =  decorate(target)
```

上面代码执行完后，得到的target不一定是原来的target函数，而是decorate(target)返回的函数。

为了进一步验证，我们可以看一下下面的例子：

```python
def deco(func):
    def inner():
        print('running inner()')
    return inner

@deco
def target():
    print('running target()')


if __name__ == '__main__':
	target()
>>> running inner()

	# 代码审查，返回__repr__的内容
	target
>>> <function deco.<locals>.inner at 0x00000222660A6D90> 
```

可以看到控制台输出的内容，完全不是`target()`函数的内容。

> 为什么出现这种现象呢？
>
> 上面的代码等价于：`deco(target)`，虽然我们把target函数传给了func变量，但是我们并没有使用func变量，反而是返回了inner函数。

严格来说，装饰器只是语法糖。

### 7.2 何时执行装饰器

在导入模块时，会立即运行。

```python
registry = [] 

def register(func): 
    print('running register(%s)' % func) 
    registry.append(func)
    return func 

@register 
def f1():
	print('running f1()')
    
@register
def f2():
	print('running f2()')
    
def f3(): 
	print('running f3()')
    
def main(): 
    print('running main()')
    print('registry ->', registry)
    f1()
    f2()
    f3()
    
    
if __name__=='__main__':
	main() 

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
running register(<function f1 at 0x0000015123895D08>)
running register(<function f2 at 0x0000015123895D90>)
running main()
registry -> [<function f1 at 0x0000015123895D08>, <function f2 at 0x0000015123895D90>]
running f1()
running f2()
running f3()
```

从运行结果可以看出，register 在模块中，其他函数之前运行了两次。

> 如果register函数中有定义`func()`，那么也会立刻运行。

上例单纯为了演示装饰器，有两个地方不妥：

- 装饰器通常在一个模块中，然后应用到其他模块的函数上
- 这个装饰器对函数未作任何修改就返回了，一般会在其内部定义一个函数，然后返回这个函数



Q：如果我的装饰器函数上，本身也带着一个装饰器函数会发生什么？

A：简单修改一下上面的例子，其他保持不变。可以看到，是按照源代码中@foo的数量运行的。。。。

```python
def aa(func):
    print('running aa(%s)' % func)
    return func

@aa
def register(func):
    print('running register(%s)' % func)
    registry.append(func)
    return func


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
running aa(<function register at 0x000002B9184D3950>)
running register(<function f1 at 0x000002B9184F0840>)
running register(<function f2 at 0x000002B9184F0E18>)
running main()
registry -> [<function f1 at 0x000002B9184F0840>, <function f2 at 0x000002B9184F0E18>]
running f1()
running f2()
running f3()
```



### 7.3 改进策略模式

改进点就是本来是通过反射获取所有的“策略”，现在是通过装饰器将所有的“策略”放到list中，然后遍历所有list求最佳策略



### 7.4 变量作用域规则

看如下代码：

```python
b = 6

def f2(a):
    print(a)
    print(b)
    b = 9

```

会直接提示报错，Why？

Python不要求声明变量，但是假定在函数定义体中==赋值的变量==是局部变量。

因为我们给b赋值了所以b变成了局部变量，如果没有这句赋值是ok的，或者在`f2()`函数下面加上`global b`，使变量b变成全局变量（在函数外定义的变量本身就是全局变量）。

如果函数外定义了一个可变类型变量时，那么在函数内，对容器内的某个值赋值不会报错，但是对容器本身重新赋值会让这个变量变成局部变量。

> 冷知识：
>
> ```python
> b = [6]
> 
> 
> def f2(a):
>     print(a)
>     print(b)
>     b[0] = 2
> 
> ```
>
> 这样就不会报错了又！Why？
>
> 因为你并没有去改变容器本身，也就是没有改变容器的内存地址，也就是没有开辟新的内存。
>
> ```python
> b = [6]
> 
> 
> def f2(a):
>     print(a)
>     print(b)
>     b= [2]
> 
> ```
>
> 这样就会报错！！！

### 7.5 闭包

什么是闭包？

指延伸了作用域的函数。使其能够访问函数定义体外定义的，又不是全局变量的变量！

看例子：

```python
def make_averager():
    series = []
    
    def averager(new_value):
        # 对于可变类型，可以直接使用，因为不会给对象赋值
        series.append(new_value)
        total = sum(series)
        return total/len(series)
    return averager

# 此时avg中存放的是函数averager()
avg = make_averager()
# 等价于averager(10)
avg(10)
avg(20)




----------------------第二种写法----------------------------
# 单纯为了演示闭包,这种写法,在业务上说不通,因为不能像上面那样重复调用
def make_averager(new_value):
    series = []
    
    def averager():
        # 对于可变类型，可以直接使用，因为不会给对象赋值
        series.append(new_value)
        total = sum(series)
        return total/len(series)
    return averager()


avg = make_averager(10)

```



在这个例子中，我们称series是averager函数的**自由变量**，指未在本地作用域中绑定的变量。

```python
# 审查函数make_averager

# 局部变量
avg.__code__.co_varnames
>>>('new_value', 'total')

# 自由变量 一个元组也要加逗号，不然括号会当作运算符处理
avg.__code__.co_freevars
>>>('series',)

# 获取自由变量保存的值
avg.__closure__[0].cell_contents
>>>[10, 20]
```



### 7.6 nonlocal声明

在上面的例子中，自由变量是一个list，而且存储了历史数据，如果我们不需要历史数据重新设计呢？



```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        count += 1
        total += new_value
        return total / count

    return averager

```

但是上述代码会报错，因为`count+=1`等价于`count = count + 1`，也就是说会给变量count赋值，导致count变成局部变量，进而导致等号右边的count变量没有定义就使用了。

Q：难道自由变量类型只能是可变类型的吗？

A：Python3引入了`nonlocal`声明，可以把一个变量标记成自由变量。

所以上面的代码需要在`averager()`函数下面添加`nonlocal count, total`即可。



Q：nonlocal和global声明，都可以让局部变量变成外部变量，那有什么区别？

A：

- nonlocal关键字只能用于嵌套函数中，最上层的函数使用nonlocal修饰变量必定会报错。
- global 无任何限制。



> 冷知识：
>
> 变量的判定优先级：
>
> 1. 当前作用域局部变量
> 2. 外层作用域变量
> 3. 当前模块中的全局变量
> 4. python内置变量
>
> 也就是说，你在一个函数中引用一个变量，它最可能是1其次是2...



### 7.7 ==实现一个装饰器==

实现一个计算函数运行时间的装饰器

```python
import time

def clock(func):
    def clocked(*args): 
        t0 = time.perf_counter()
        result = func(*args) 
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
	return clocked 

@clock
def foo_aa(a):
    print(a)

    
if __name__ == '__main__':
    foo_aa(2)
    
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>    
[0.00000910s] foo_aa(2) -> None
```

Q：如何做到的？

```python
# 首先上面的装饰器是语法糖，等价于
foo_aa = clock(foo_aa)

# 因为clock函数返回了clocked函数
# 此时foo_aa指向的内存里的函数是clocked，且包含自由变量func，func存储的是函数foo_aa：
foo_aa = clocked(*args)

# 所以当运行foo_aa(2)时，其实就是运行了
clocked(2)

```



但是上面的装饰器有几个缺点：

- 不支持关键参数
- 遮盖了被装饰函数的`__name__`和`__doc__`属性

改进后的装饰器：

```python
import time
import functools

def clock(func):
    # 将func的属性复制到clocked中
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - t0
        name = func.__name__
        arg_lst = []
        if args:
        	arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
        	pairs = ['%s=%r' % (k, w) for k, w in sorted(kwargs.items())]
        	arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
        return result
    return clocked


```



### 7.8 自带的装饰器

内置了`property`、`classmethod`和`staticmethod`



#### 7.8.1 functools.lru_cache

functools.lru_cache是非常实用的装饰器，实现了备忘功能。

它可以把耗时的函数的结果保存起来，避免了传入相同参数时的重复计算。

`LRU`三个字母是“Least Recently Used”的缩写，同时表明了缓存不会无限增长，一段时间不用的条目会被扔掉。

```python
# 必须带括号！
@functools.lru_cache()
def fibonacci(n):
    if n < 2 :
        return n
    return fibonacci(n-2) + fibonacci(n-1)

```



有可选的两个参数来配置：`functools.lru_cache(maxsize=128, typed=False)`

- maxsize指定存储的结果数量，缓存满了后，旧的会被扔掉。为了最佳性能，maxsize的值应该设为2的幂。
- typed=True会把不同参数类型得到的结果分开保存，即`1.0`和`1`区分开来。

lru_cache使用字典存储结果，而key就是被装饰函数的定位参数和关键参数，所以被装饰函数的所有==参数必须是可散列的==。

#### 7.8.2 singledispatch泛函数

因为Python不支持重载函数，所以我们不能用不同的签名实现函数重载。在Python中常见的作法是把函数变成一个分派函数即使用`if/elif/elif`，调用对应的函数但是这样不利于模块的用户扩展，时间一长，分派函数会变得很长，与专门函数之间的耦合也很紧密。

Python3.4新增的`functools.singledispatch`装饰器，可以把整体方案拆成多个模块。使用`@singledispatch`装饰的函数会变成*泛函数*：根据第一个参数的类型，以不同方式执行相同操作的同一组函数，具体做法如下：

```python
from functools import singledispatch
from collections import abc
import numbers
import html

# @singledispatch标记处理object类型的基函数
@singledispatch
def htmlize(obj):
    content = html.escape(repr(obj))
    return '<pre>{}</pre>'.format(content)

# 各个专门函数使用@foo.register(type)装饰
@htmlize.register(str)
def _(text): 
    content = html.escape(text).replace('\n', '<br>\n')
    return '<p>{0}</p>'.format(content)

# 专门函数的名称不重要，统一使用“_”
@htmlize.register(numbers.Integral)
def _(n):
	return '<pre>{0} (0x{0:x})</pre>'.format(n)

# 可以叠放
@htmlize.register(tuple) 
@htmlize.register(abc.MutableSequence)
def _(seq):
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>' + inner + '</li>\n</ul>'


```

这个机制的优点是：

- 为不是自己写的或者不能修改的类添加自定函数
- 可以在系统的任何地方和模块中注册专门函数





> 冷知识：
>
> 上面的示例代码中出现了一个abc.MutableSequence的类型，这是abc模块中定义的一些抽象基类，用于判断一个具体的类是否具有特定的接口；一个abc.MutableSequence的类型，这是abc模块中定义的一些抽象基类，用于判断一个具体的类是否具有特定的接口；常用的抽象基类有:

| 抽象基类                                                     | 继承自                                                       | 抽象方法                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------------------- |
| [`Container`](https://docs.python.org/zh-cn/3/library/#collections.abc.Container) |                                                              | `contains`                                   |
| [`Hashable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Hashable) |                                                              | `hash`                                       |
| [`Iterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterable) |                                                              | `iter`                                       |
| [`Iterator`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterator) | [`Iterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterable) | `next`                                       |
| [`Reversible`](https://docs.python.org/zh-cn/3/library/#collections.abc.Reversible) | [`Iterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterable) | `reversed`                                   |
| [`Generator`](https://docs.python.org/zh-cn/3/library/#collections.abc.Generator) | [`Iterator`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterator) | `send`, `throw`                              |
| [`Sized`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sized) |                                                              | `len`                                        |
| [`Callable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Callable) |                                                              | `call`                                       |
| [`Collection`](https://docs.python.org/zh-cn/3/library/#collections.abc.Collection) | [`Sized`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sized),[`Iterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Iterable),[`Container`](https://docs.python.org/zh-cn/3/library/#collections.abc.Container) | `contains`,`iter`,`len`                      |
| [`Sequence`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sequence) | [`Reversible`](https://docs.python.org/zh-cn/3/library/#collections.abc.Reversible),[`Collection`](https://docs.python.org/zh-cn/3/library/#collections.abc.Collection) | `getitem`,`len`                              |
| [`MutableSequence`](https://docs.python.org/zh-cn/3/library/#collections.abc.MutableSequence) | [`Sequence`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sequence) | `getitem`,`setitem`,`delitem`,`len`,`insert` |
| [`ByteString`](https://docs.python.org/zh-cn/3/library/#collections.abc.ByteString) | [`Sequence`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sequence) | `getitem`,`len`                              |
| [`Set`](https://docs.python.org/zh-cn/3/library/#collections.abc.Set) | [`Collection`](https://docs.python.org/zh-cn/3/library/#collections.abc.Collection) | `contains`,`iter`,`len`                      |
| [`MutableSet`](https://docs.python.org/zh-cn/3/library/#collections.abc.MutableSet) | [`Set`](https://docs.python.org/zh-cn/3/library/#collections.abc.Set) | `contains`,`iter`,`len`,`add`,`discard`      |
| [`Mapping`](https://docs.python.org/zh-cn/3/library/#collections.abc.Mapping) | [`Collection`](https://docs.python.org/zh-cn/3/library/#collections.abc.Collection) | `getitem`,`iter`,`len`                       |
| [`MutableMapping`](https://docs.python.org/zh-cn/3/library/#collections.abc.MutableMapping) | [`Mapping`](https://docs.python.org/zh-cn/3/library/#collections.abc.Mapping) | `getitem`,`setitem`,`delitem`,`iter`,`len`   |
| [`MappingView`](https://docs.python.org/zh-cn/3/library/#collections.abc.MappingView) | [`Sized`](https://docs.python.org/zh-cn/3/library/#collections.abc.Sized) |                                              |
| [`ItemsView`](https://docs.python.org/zh-cn/3/library/#collections.abc.ItemsView) | [`MappingView`](https://docs.python.org/zh-cn/3/library/#collections.abc.MappingView),[`Set`](https://docs.python.org/zh-cn/3/library/#collections.abc.Set) |                                              |
| [`KeysView`](https://docs.python.org/zh-cn/3/library/#collections.abc.KeysView) | [`MappingView`](https://docs.python.org/zh-cn/3/library/#collections.abc.MappingView),[`Set`](https://docs.python.org/zh-cn/3/library/#collections.abc.Set) |                                              |
| [`ValuesView`](https://docs.python.org/zh-cn/3/library/#collections.abc.ValuesView) | [`MappingView`](https://docs.python.org/zh-cn/3/library/#collections.abc.MappingView),[`Collection`](https://docs.python.org/zh-cn/3/library/#collections.abc.Collection) |                                              |
| [`Awaitable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Awaitable) |                                                              | `await`                                      |
| [`Coroutine`](https://docs.python.org/zh-cn/3/library/#collections.abc.Coroutine) | [`Awaitable`](https://docs.python.org/zh-cn/3/library/#collections.abc.Awaitable) | `send`, `throw`                              |
| [`AsyncIterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.AsyncIterable) |                                                              | `aiter`                                      |
| [`AsyncIterator`](https://docs.python.org/zh-cn/3/library/#collections.abc.AsyncIterator) | [`AsyncIterable`](https://docs.python.org/zh-cn/3/library/#collections.abc.AsyncIterable) | `anext`                                      |
| [`AsyncGenerator`](https://docs.python.org/zh-cn/3/library/#collections.abc.AsyncGenerator) | [`AsyncIterator`](https://docs.python.org/zh-cn/3/library/#collections.abc.AsyncIterator) | `asend`, `athrow`                            |





### 7.9 叠放装饰器

```python

@d1
@d2
def foo():
    print("foo")

# 等同于
def foo():
    print("foo")
    
foo=d1(d2(foo))

```



### 7.10 参数化装饰器

Q：如何让一个装饰器接收参数呢？

A：创建一个装饰器工厂函数！把参数传给它，然后返回一个装饰器，再把它应用到要装饰的函数上，如下。

```python
# set对象，添加和删除速度更快
registry =set()

def register(active=True):
    # decorate函数才是真正的装饰器函数
    def decorate(func):
        if active:
            registry.add(func)
        else:
            # 移除指定的集合元素,没有也不会报错
            registry.discard(func)
		return func
    return decorate

@registry(active=False)
def f1():
    print('running f1()')
    
@registry()
def f2():
    print('running f2()')


# 不使用装饰器，写法如下
# registry()运行会返回decorate，即decorate(foo)
registry()(foo)
```









### 7.11本章小结



审查装饰器

```python

# 局部变量
foo.__code__.co_varnames
>>>('new_value', 'total')

# 自由变量 一个元组也要加逗号，不然括号会当作运算符处理
foo.__code__.co_freevars
>>>('series',)

# 获取自由变量保存的值
foo.__closure__[0].cell_contents
>>>[10, 20]
```



变量的判定优先级：

1. 当前作用域局部变量
2. 外层作用域变量
3. 当前模块中的全局变量
4. python内置变量



`@functools.lru_cache(maxsize=128, typed=False)`装饰器可以用来提高性能

`@singledispatch`可以将函数变成单分派函数，然后foo.register装饰器分派处理。

用工厂模式的理念，实现在装饰器中添加参数。



## 第8章 对象引用和垃圾回收

变量是一个标注,而不是盒子。



### 8.1 变量不是盒子

在Java中我们经常能听到人们说，“变量是盒子”。但是这么说其实有碍于人们理解面向对象语言中的引用式变量。Python变量类似于Java中的引用式变量，因此最好把它们理解为附在对象上的标注。

如果变量是盒子，那就无法解释下面的代码：

```python
a = list((1,2,3))
b = a
a.append(4)
b
>>>[1,2,3,4]
```

同样，Python的复制语句，应该始终先读右边，在此之后左边的变量才会绑定到对象上，这就像给对象贴标签！

因为变量只不过是标注，所以无法阻止为对象贴上多个标注，贴的多个标注，就是*别名*

### 8.2 标识、相等性和别名



```python
    
    charles = {'name': 'Rohan', 'born': 1995}
    lewis = charles
    # 两者的id也相同
    id(lewis)
	lewis is charles
    >>> True
    
    # 如果出现一个变量指向的对象内容一致
    leo = {'name': 'Rohan', 'born': 1995}
    # 因为dict类的__eq__方法就是这样写的
    charles == leo
    >> True
    
    # 两者并不是同一对象，id()也不相等
    leo is charles
    >> False
    
```



指向同一对象的不同变量称作*别名*，在上例中，即`lewis`和` charles`变量。

> 扩展：
>
> 每个变量都有标识、类型和值。很明显在Python中，这三个属性都会变。但对象一旦创建，它的标识绝不会变，可以把它理解为对象在内存中的地址。id()返回的就是对象标识的整数标识， is 运算符比较的也是对象的标识。



#### 8.2.1 ==和is的取舍

`==`运算符比较的是两个对象的值（对象保存的数据），而`is`比较的是对象的标识。

一般的，我们更容易关注值，所以 `==` 出现的频率要高一些。

在变量和单例值（singleton data value）进行比较时，应该使用`is`，目前最常检查变量绑定的值是不是None，推荐的写法是：

```python
x is None
```

`is`运算符比`==`速度快，因为它不能重载（相同函数的不同签名），所以Python不会寻找并调用特殊方法（pass），而是直接比较两个整数ID。`==`则是语法糖，多数内置类型使用更有意义的方法覆盖了`__eq__`方法。

#### 8.2.2 元组的相对不可变

元组与多数Python集合（列表、字典、集，等等）一样，保存的是对象的引用。如果引用的元素是可变的话，即便元组不可变，元素依然可变。也就是说元组的不可变性其实是指tuple数据结构的物理内容（即保存并的引用）不可变，与引用的对象无关。

> 冷知识：
>
> str、bytes、和array.array等单一类型序列是扁平的，它们保存的不是引用，而是在连续的内存中保存数据本身（字符、字节和数字）

```python

t1 = (1,2,[30,40])
t2 = (1,2,[30,40])

t1 ==t2
>True

t1 is t2
> False

# 但是并没有修改t1的ID
t1[-1].append(50)
>False

# 如果是这样情况就不一样了
t2 = tuple(t1)
t2 is t1
> True

# 修改的其实是同一个
t2[-1].append(50)
t1 ==t2
>True

```

### 8.3 ==默认浅复制==

复制列表或内置的多数可变集合，最简单的方式是使用内置的类型构造方法（eg:`list()`、 `dict()`）

```python
    l1 = [1,1]
    l2 = list(l1)
    l1 is l2
    > False
    l1 == l2
    > True

```

==因为构造方法或是`[:]`做的都是浅复制，即复制了最外层的容器，副本中的元素是源容器中的引用。==如果源容器的元素都是不可变的还好，否则就会出现意外。

```python
if __name__ == '__main__':
    class Demo:
        a = [1, 2, [33]]


    a = Demo()
    b = copy.copy(a)

    d = copy.copy(a.a)
    print(id(Demo.a),id(a.a),id(b.a),id(d))
    print('说明a b 中的变量a  和 Demo.a指向同一个内存地址，而d新开辟了一个地址',)
    print("为什么b不像d，或者是d不像b；因为b是对引用变量a做浅复制，引用变量a中的内容如果是不可变类型，则新开辟内存，可变类型则指向同一内存地址")
    print("引用变量a中的内容是Demo.a 是可变类型，所以b直接指向Demo.a指向的内存地址")

    print(id(Demo.a[2]),id(a.a[2]), id(b.a[2]), id(d[2]))
    print('说明a b d中的list中的list， 永远指向同一个内存地址')

    a.a[2] = [555]
    print(id(a.a[2]), id(b.a[2]), id(d[2]))
    print("a b list 中的list 不再指向 原来的地址，而是新开辟的地址")
    
    # 深刻理解复制最外层的容器，当你copy一个类的时候，实例就是最外层的容器，里面的list一定指向同一引用！
```





将下面代码粘贴到[Python Tutor - Visualize Python](http://www.pythontutor.com/visualize.html#mode=edit)，能可视化这个流程。

```python
l1 = [3, [66, 55, 44], (7, 8, 9)]
# l2拷贝了l1最外层的容器，也就是说[3, 指向相同地址1, 指向相同地址2]
l2 = list(l1)  # ➊

l1.append(100)  # ➋
l1[1].remove(55)  # ➌

print('l1:', l1)
>[3, [66, 44], (7, 8, 9), 100]
print('l2:', l2)
>[3, [66, 44], (7, 8, 9)]

l2[1] += [33, 22]  # ➍
print('l1:', l1)
>l1: [3, [66, 44, 33, 22], (7, 8, 9), 100]
    
l2[2] += (10, 11)  # ➎
print('l2:', l2)
>l2: [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]

```

`l2 = list(l1)`   `l2`变量指向的对象中，可变元素是指向`l1`中的地址的

![image-20210521095214688](图片/流畅的Python/image-20210521095214688.png)

`l1.append(100)` 可以看到`l1`变量指向的最外层容器新增了一位，`l2`无变化

![image-20210521095407969](图片/流畅的Python/image-20210521095407969.png)

`l1[1].remove(55) ` `l1`的第二个元素是一个可变元素，这个可变元素指向了一个list，并移除了一个元素。因为`l2`变量的第二个元素也指向了这个list，所以`l2`也变了。

![image-20210521095552921](图片/流畅的Python/image-20210521095552921.png)

`l2[1] += [33, 22]`，对于可变对象来说`+=`会就地改变对象。

![image-20210521095922957](图片/流畅的Python/image-20210521095922957.png)

`l2[2] += (10, 11)`，对于元组来说`+=`会创建一个新的元组，然后重新绑定到`l2[2]`上

![image-20210521100124113](图片/流畅的Python/image-20210521100124113.png)

现在可以看出，l2的变量内容似乎远远超出我们的期望，所以我们需要深复制

#### DIY深复制or浅复制

copy模块的`deepcopy`和 `copy`让我们能够为任意对象做深复制和浅复制。

让我们通过一个例子来看一下：

```python
class Bus:
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)
```

控制台调试：

```python
>>> import copy
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)
>>> bus3 = copy.deepcopy(bus1)

# 如何理解此处三个id都不一样呢？因为浅复制复制了最外层的容器，确实是新开辟了内存，但是如果容器内部又有可变类型的容器的话，这些内部容器的地址是指向同一个的
# 此处推测，copy.copy做的是浅复制，bus1.__dict__可以看到属性对应的值是一个可变容器列表，所以在copy的时候，这个值指向了同一地址。
>>> id(bus1), id(bus2), id(bus3)
(4301498296, 4301499416, 4301499752)
>>> bus1.drop('Bill')
>>> bus2.passengers
['Alice', 'Claire', 'David'] 

# 这里可以看出端倪来
>>> id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
(4302658568, 4302658568, 4302657800) 
>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David'] 
```

##### 循环引用

如果有循环引用，deepcopy怎么办？先让我们看一个例子，什么是循环引用：

```python
    list_1 = [11,2]
    list_2= [list_1, 33]
    list_1.append(list_2)
	list_1
>>>>[11, 2, [[...], 33]]
	list_2
>>>>[[11, 2, [...]], 33]
```

可以看到引用部分，用`...`也就是ellipsis （2.4.3 多维切片和省略 中有介绍）

> pass
>
> 这里自己有点迷糊
>
> ```python
>     a = [1, 2]
>     b = [a, 3]
>     a.append(b)
> 	# 现在b里面是什么？
> ```

### 8.4 函数参数作为引用时

Python唯一支持的参数传递模式是*共享传参*。多数面向对象语言都采用这一模式，包括Ruby、Smalltalk和 Java（Java的引用类型是共享传参，基本类型是按值传参）

共享传参的意思就是，函数内部的参数名是函数外部变量中引用的副本，也就是别名。

所以这种情况下，如果传进来的参数是可变变量，那么函数就有可能修改变量的内容。我们来看个例子

```python
def foo(a, b):
    a += b
    return a

if __name__ == '__main__':
    x = 1
    y = 2
    z = foo(x, y)
    # 此时x还是1，z=3
    
    
    x = [1,2]
    y = [3, 4]
    z = foo(x, y)
    # 此时x为[1,2，3，4] 已经被修改了！
    # 传入字典也会修改。
    
```

#### 8.4.1 参数默认值的陷阱

> 当使用可变类型变量作为参数默认值时，可能会发生意料之外的事情。

看下面一个例子，相对于Bus类，我们似乎做了一些改善，代码更加简洁了。

```python
class HauntedBus:
    def __init__(self, passengers=[]):
        self.passengers = passengers

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)
        
if __name__ == '__main__':
    print(dir(HauntedBus.__init__))
    # ['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__'，...]
    # 在第五章一等函数中，我们说过函数在Python中也是对象，__init__这个函数可以看到和普通函数一样，用__defaults__存储了函数默认值
    
    print(HauntedBus.__init__.__defaults__)
    # ([],)
    # 可以看到默认值是一个list
    
    a=[1111]
    bus1 = HauntedBus(a)
    bus2 = HauntedBus()
    b = [3512]
    bus3 = HauntedBus(b)
    bus4 = HauntedBus()
    print(HauntedBus.__init__.__defaults__[0] is bus2.passengers)
    # True
    # 这句说明，在没有默认值时，实例bus2的passengers是类HauntedBus中函数__init__中存储默认值的别名，即他们指向同一地址
    
    # bus1和bus3说明在传入默认值的时候，如果修改传入的可变类型，同样也修改了实例的属性
    # bus2和bus4的地址是一致的，都指向函数默认值，所以容易造成意外
    print(id(a),id(bus1.passengers),id(bus2.passengers),id(b),id(bus3.passengers),id(bus4.passengers))
    # 2332890193472 2332890193472 2332890193664 2332890193408 2332890193408 2332890193664
        
```

通过上例我们可以看出：

对于默认参数是可变类型时，有两种情况：

- 如果传入默认值

  行为可以符合预期，但是修改传入的变量，同样会修改类的属性

- 如果无默认值

  所有无默认值的实例的默认参数，全部指向了同一地址。



#### 8.4.2 防御可变参数

Q：如果一个函数接受一个字典，foo还在内部修改了这个字典，那么这个副作用要不要体现到外部呢？

A：应该根据业务判断，要开发者和维护者达成共识。

看下面的例子：

```python
class TwilightBus:
    """让乘客销声匿迹的校车"""

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = passengers

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)


if __name__ == '__main__':
    basketball_team = ['Sue', 'Tina', 'Maya', 'Diana', 'Pat']
    bus = TwilightBus(basketball_team)
    bus.drop('Tina')
    bus.drop('Pat')
    
    # 可以看到两个人从车上下来，直接从球队消失了
    # 这违背了设计接口的最佳实践，即“最少惊讶原则”
    print(basketball_team)
    # ['Sue', 'Maya', 'Diana']


```

正确的做法是Bus类维护自己的乘客列表，如下：

```python
class TwilightBus:

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            # 做了一次浅复制
            self.passengers = list(passengers)
```

> 其实笔者觉得这样依然有问题，比如说，如果球队是一个嵌套列表（emmm比如说有个女的在车上怀孕了），那么操作这个嵌套列表，依然会改动到Bus上的列表，最完美的就是深复制吧。



### 8.5 del和垃圾回收

> 对象绝不会自行销毁；然而，无法得到对象时，可能会被当作垃圾回收。

Python中`del`语句删除的是”便利贴“，而不是对象。`del`命令可能会导致对象当作垃圾回收，仅当删除的变量保存的是对象的最后一个引用（即这个变量没有贴在身上的便利贴了）。重新绑定也可能导致对象的引用数量归零，从而对象被销毁。

>有个`__del__`特殊方法，但它不会销毁实例，不应该在代码中调用，也很少自己实现`__del__`方法。
>
>Python解释器自己会在合适的机会调用`__del__`方法

```python
    import weakref

    s1 = {1, 2, 3}
    s2 = s1


    def bye():
        print('我没了')


    ender = weakref.finalize(s1, bye)
    print(ender.alive)
    # True

    del s1
    print(ender.alive)
    # True
    
    s2 = 'hh'
    # 我没了
    print(ender.alive)
    # False
```

上述例子说明，del不会删除对象。

上述例子，你是否疑惑，我们这不是把`s1`传给`finalize`函数了吗？为了监控对象必须要有引用。这是因为finalize持有对象的弱引用

### 8.6 弱引用

正是因为有引用，对象才会在内存中存在。当对象的引用数量归零后， 垃圾回收程序会把对象销毁。但是，有时需要引用对象，而不让对象存在的时间超过所需时间。这经常用在*缓存*中。 

弱引用不会增加对象的引用数量。引用的目标对象称为所指对象 （referent）。因此我们说，弱引用不会妨碍所指对象被当作垃圾回收。 弱引用在缓存应用中很有用，因为我们不想仅因为被缓存引用着而始终保存缓存对象。

> 冷知识：
>
> 在控制台会话中，Python控制台会自动把`_`变量绑定到结果部位None的表达式结果上。所以我们在控制台中演示本身就会有影响，这也说明：微观管理内存，往往会得到意想不到的结果，不明显的隐式复制会为对象创建新引用。

下面代码我们不在控制台运行：

```python
if __name__ == '__main__':
    import weakref
    a_set = {0, 1}
    # 创建一个弱引用对象
    wref = weakref.ref(a_set)
    # 审查这个对象
    print(wref)
    # <weakref at 0x000001D777652860; to 'set' at 0x000001D7774B3C80>

    # 调用它，会返回被引用的对象，这里我们可以看到，是{0, 1}
    print(wref())
    # {0, 1}
    
    a_set = {2, 3, 4}
    # {0, 1}没有强引用了
    print(wref())
    # None
    
    print(wref() is None)
    # True
```

不要手动创建并处理`weakref.ref()`实例，多数情况下使用weakref集合

#### 8.6.1 weakref集合

常用的有:

- weakref.WeakValueDictionary()
- weakref.WeakKeyDictionary()
- weakref.Set()

WeakValueDictionary 类实现的是一种可变映射，里面的值是对象的弱引用。被引用的对象在程序中的其他地方被当作垃圾回收后，对应的键会自动从 WeakValueDictionary 中删除。因此，WeakValueDictionary 经常用于缓存。

```python
class Cheese:
    def __init__(self, kind):
        self.kind = kind

    def __repr__(self):
        return 'Cheese(%r)' % self.kind

if __name__ == '__main__':
    import weakref
    stock = weakref.WeakValueDictionary()
    catlog = [Cheese('A'), Cheese('B'), Cheese('C')]

    for cheese in catlog:
        stock[cheese.kind] = cheese
    sorted(stock.keys())
    # ['A', 'B', 'C']
    
    # for循环中的变量是全局变量,除非显式删除,负责不会消失
    del catlog
    sorted(stock.keys())
    # ['C']
    
    del cheese
    sorted(stock.keys())
    # []

```

> 冷知识:
>
> for循环中的变量是全局变量,除非显式删除,负责不会消失



#### 8.6.2 弱引用的局限

不是每个Python对象都可以作为弱引用的目标(或所指对象)，基本的`list`和`dict`实例就不能作为所指对象（`set`可以），但是他们的子类可以

```python
class MyList(list):
    """list的子类，实例可以作为弱引用的目标"""
    
a_list = MyList(range(10))

# a_list可以作为弱引用的目标
wref_to_a_list = weakref.ref(a_list)

```



### 8.7 不可变类型的小心机

对于元组`t`来说，`t[:]`不创建副本，而是返回了同一对象的引用，`tuple(t)`同样如此。

```python
if __name__ == '__main__':
    a = [1,2]
    b = list(a)
    print(a == b) # True
    print(a is b)# False

    c = (1,2,3)
    d = tuple(c)
    print(c == d)# True
    print(c is d)# True

    e = (1,2,3)
    print(c == e)# True
    print(c is e)# True
    
    # 此处c = (1,2,[3])
    f = (1,2,[3])
    print(c == f)# True
    print(c is f)# False 因为[3] 和[3] 的内存地址就不一样
    
    g = copy.deepcopy(c)
    print(c == g)# True
    print(c is g)# False
```



### 8.8 本章小结

`weakref.WeakValueDictionary()`用作缓存







## 第9章 Python风格的对象

得益于Python的数据模型，自定义类型的行为可以像内置类型那样自然。实现如此自然的行为靠的不是继承，而是“*鸭子类型*”：我们只需要按照预定行为实现对象所需的方法即可。

接续第一章，我们来实现更多Python类型中常见的特殊方法。

我们将开发一个简单的二维欧几里得向量类型

### 9.1 对象表示形式

每门面向对象的语言都有至少一种获取对象字符串表示形式的标准方法。Python有两种方式：

- `repr()`

  便于开发者理解的方式返回对象的字符串表示形式。

- `str()`

  便于用户理解的方式返回对象的字符串表示形式。

为了实现上述方式，我们需要分别实现`__repr()__`和`__str()__`两种特殊方法。



### 9.2 实现向量类

我们将实现一个Vector2d类，期望它的实例具有以下行为：

```python
>>> v1 = Vector2d(3, 4)
	# 可以直接访问实例的分量
>>> print(v1.x, v1.y)
3.0 4.0

	# 实例可以拆包成变量元组
>>> x, y = v1
>>> x, y
(3.0, 4.0)
	
    # repr函数会直接调用类的实例，得到的结果类似于构建时的源码
>>> v1 
Vector2d(3.0, 4.0)

	# 用eval，只是为了说明repr()得到的构建时源码十分准确
>>> v1_clone = eval(repr(v1)) 
>>> v1 == v1_clone 
True
	
    # print会调用str()，返回一个有序对
>>> print(v1) 
(3.0, 4.0)

	# 会调用__bytes__方法，生成实例的二进制表示形式
>>> octets = bytes(v1) 
>>> octets
b'd\\x00\\x00\\x00\\x00\\x00\\x00\\x08@\\x00\\x00\\x00\\x00\\x00\\x00\\x10@'

	# 会调用__abs__方法，返回实例的模
>>> abs(v1) 
5.0

	# 调用__bool__方法，模为0返回False
>>> bool(v1), bool(Vector2d(0, 0)) 
(True, False)

```



下面开始我们的实现

```python
from array import array
import math

class Vector2d:
    typecode = 'd' 
    def __init__(self, x, y):
        self.x = float(x) 
        self.y = float(y)
    
    # __iter__是返回一个迭代器，之前的例子是返回self，然后实现类的__next__,此处返回了生成器表达式
    def __iter__(self):
    	return (i for i in (self.x, self.y)) 
    
    def __repr__(self):
        class_name = type(self).__name__
        # {!r}是python转义字符，用来获取分量pass
        # self可以理解为实例，*看起来像是拆包self，所以会走__iter__方法
        return '{}({!r}, {!r})'.format(class_name, *self) 
    
    # tuple()接收一个可迭代的对象，所以会对self拆包，所以会走__iter__方法
    def __str__(self):
    	return str(tuple(self)) 
    
    # ord()以一个字符（长度为1的字符串）作为参数，返回对应的 ASCII 数值
    # pass
    def __bytes__(self):
        # 为什么self.typecode
    	return (bytes([ord(self.typecode)]) + 
    			bytes(array(self.typecode, self))) 
    
    # 实现==语法糖
    def __eq__(self, other):
	    return tuple(self) == tuple(other)

    
    def __abs__(self):
    	return math.hypot(self.x, self.y)
    
    def __bool__(self):
    	return bool(abs(self)) 



```



我们的`Vector2d(3, 4) == [3, 4]`可以视作特性，也可以视作缺陷。



### 9.3 ==备选构造方法==

我们可以通过`bytes()`方法把Vector2d实例转换成字节序列；同理，也应该能从字节序列转换成Vector2d，下面增加此功能。

最核心的就是`return cls(*memv)`这行代码简单来说，就是创建并返回了一个新的实例

```python
# 类方法使用classmethod装饰器修饰
# 不传入self，而是传入cls类本身
@classmethod
def frombytes(cls, octets):
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(*memv)

```



### 9.4 staticmethod

两者都可以直接访问而不需要实例化（因为两者都没有传进去self），`classmethod`第一个参数必须代表类本身，所以它可以在方法体里面创建新的类，并返回。所以`staticmethod`对应Java中的静态方法，而`classmethod`是额外特殊功能



`classmethod`用法：定义操作类，而不是操作实例的方法。`classmethod`的第一个参数是类本身，而不是实例。`classmethod`最常见的用途是定义备选构造方法。类方法的第一个参数`cls`也是约定（Python不介意具体怎么命名）。

`staticmethod`装饰器也会改变方法的调用方式。但是第一个参数不是特殊的值。静态方法就是普通的函数，只是==碰巧在类的定义体中==。

以下是一个简单的对比：

```python

class Demo:
	# 虽然此处是可变参数，但是元组args的第一个值一定是Demo类本身
    @classmethod
    def klassmeth(*args):
        return args

    @staticmethod
    def statmeth(*args):
        return args

# 首先返回的一定是Demo类 
Demo.klassmeth(5)
>>> (<class '__main__.Demo'>, 5)

# 和普通函数无异
Demo.staticmethod(5)
>>> (5,)
```

### 9.5 格式化显示

- format(my_job, format_spec) 
- str.format()

有两种实现`format()`函数和`str.format()`方法。如果两个都陌生，优先学习`format()`函数，因为它只使用了==格式规范微语言==。再去学习`str.format()`。

什么是格式规范微语言？

为内置类型提供了专门的表示代码。

- `b`和`x`分别代表二进制和十六进制的`int`类型。

  ```python
  	format(42, 'b')
  >>> '101010'
  ```

- `f`表示小数形式的`float`类型。

  ```python
      format(1/223, '0.4f')
  >>> '0.0045'
  
      format(1/223, '0.3f')
  >>> '0.0044843'
  ```

- `%`表示百分数形式。

  ```python
  	format(1/223, '0.4%')
  >>> '0.4484%'
  
  
  ```

  

格式微语言支持上述的吗？No，是可以扩展的，比如说

```python
from datetime import datetime

now = datetime.now()
format(now, '%Y-%m-%d %H:%M:%S')

```



> 冷知识：
>
> 如果类没有定义`__format__`方法，从object继承的方法会返回`str(my_object)`，但是传入格式说明符，则会抛出`TypeError`。



剩余的pass



### 9.6 实现可散列的类

目前Vector2d的实例时不可散列的，因此不能放入集合（set）中。为了可散列我们需要：

- 实现`__hash__`方法和`__eq__`方法
- ==属性不可变==

现在的Vector2d实例的属性可以随意变更，我们不期望这样的行为，为此需要把属性设为只读，如下：

```python
class Vector2d:
    typecode = 'd'
    # 一个或两个前导下划线，把属性设置为私有（有利也有弊）
    def __init__(self, x, y):
        self.__x = float(x) 
        self.__y = float(y)
        
    # 把一个方法变成属性一样调用（不用在函数后面加括号了），属性的名字就是函数的名字
    # 此时装饰器还会生成一个@x.setter装饰器，用来设置属性
    @property 
    def x(self):
    	return self.__x
    
    @property 
    def y(self):
    	return self.__y
    
    def __iter__(self):
    	return (i for i in (self.x, self.y))

```



接下来再实现hash方法：

```python
# 这个方法应该返回一个整数
def __hash__(self):
    # 表面是调的属性x，其实是调用的，经过@property装饰，才变成属性的函数def x(self)
	return hash(self.x) ^ hash(self.y)

# 理想情况下，还要考虑对象属性的散列值__eq__方法
```

Q：为什么可散列，还必须要让属性只读？

A：

> 冷知识：
>
> 创建可散列的类型，不一定非要保护实例属性。只要正确的实现`__hash__`方法和`__eq__`方法即可。但是实例的散列值绝不应该变，此处我们实例的散列值是根据属性计算出来的，因此借机提到了只读属性。



### 9.7 私有和受保护属性

- 私有属性：

  `__attribute` 有名称改写机制

- 受保护的属性：

  `_property` 口头约定，不做任何处理

**名称改写机制**

为了避免用户在继承一个类的时候，覆盖了父类的属性，我们可以用`__mood`这种形式命名。

我们访问一个实例的`__dict__`，可以获取这个实例的所有属性，我们会发现`__mood`在`__dict__`中存储的名称变成了`_Class__mood`，即在`__mood`前面加上了“下划线”+“当前类名”。

但是不会对单下划线做这种处理，即`_mood`在`__dict__`中不会改变！

这就是为什么说双下划线是一种十分自私的行为的原因。有的人不喜欢名称改写机制，所以约定使用单下划线。

> 冷知识：
>
> 在模块中单下划线的变量，在from mymod import * 时不会被导入，但是from mymod import _mood可以导入。



### 9.8 节省类的空间

> 只有存在类方法(静态方法)、类属性、实例属性、实例方法中至少一项，才会有`__dict__`属性。[参考链接](https://segmentfault.com/a/1190000022748098)

默认情况下，Python在各个实例的`__dict__`字典中存储实例属性。因为字典会消耗大量的内存，如果要处理属性不多的实例，通过`__slot__`，能节省大量内存，因为它让解释器在==元组==中存储实例属性，而不是通过字典`__dict__`。

> 注意：
>
> 继承自超类的`__slot__`的属性没有效果，Python只会使用各个类中定义的`__slot__`属性。

对我们的`Vector2d`增加如下代码

```python
class Vector2d:
    # 实例中不能出现__slot__没有定义的其他属性
    __slot__ = ('__x', '__y')
	...
```

> 冷知识：
>
> 如果在`__slot__`元组中添加`__dict__`，那么实例属性保存在元组中，动态创建的属性保存在`__dict__`。

> 注意：
>
> 如果类中定义了`__slot__`，那要记得把`__weakref__`加进来，以让自定义类支持弱引用。
>
> 每个子类都要定义`__slot__`，因为解释器会忽略继承`__slot__`属性。



### 9.9 覆盖类属性

在`Vector2d`类中，有一个有个`typecode`属性，但是我们在使用时是直接使用的`self.typecode`。我们的实例`self`其实是没有这个值的，所以`self.typecode`默认获取的是`Vector2d.typecode`类属性的值。

```python
class Vector2d:
    typecode = 'b'
    
    def __bytes__(self):
    	return (bytes([ord(self.typecode)]) + 
    			bytes(array(self.typecode, self))) 
```

对于`Vector2d`类的实例，我们可以修改这个类属性，且只对当前实例有效。

如果想修改所有类的`typecode`属性，可以这么做：

```python
Vector2d.typecode = 'f'
```

然而有种风格更Python，而且效果持久，更具有针对性。类属性是公开的，所以会被子类继承，于是经常会创建一个子类，只用于定制类的数据属性。

```python
class ShortVector2d(Vector2d):
    typecode = 'f'
```



### 9.10 本章小结



## 第10章 序列的修改、散列和切片

*不要检查它是不是鸭子：检查它的叫声像不像鸭子、它的走路姿势像不像鸭子*

### 10.1 自定义序列类型

我们打算使用组合的模式实现一个新类Vector，它是一个不可变的扁平序列，实例中的元素都是浮点数，还与前一章定义的Vector2d类兼容。



### 10.2 与Vector2d类兼容

为了能够构建多维的向量，我们可以让`__init__`方法接收任意个参数（通过`*args`）；但是序列类型的构造方法最好接受可迭代的对象为参数，因为所有的内置序列类型都是这样做的。

我们来看一下我们的第一版代码

```python
from array import array
import reprlib
import math


class Vector:
    typecode = 'd'

  	# _components受保护的，会把可迭代传到一个数组里
    # array指定编码类型，只能存放同一类型的数据，支持可变序列的所有操作
    def __init__(self, components):
        self._components = array(self.typecode, components)

    # 构建迭代器，后面讲
    def __iter__(self):
        return iter(self._components)
	
    # reprlib模块可以生成有限的表示形式。
    # reprlib.repr()内的元素如果超过6个，其生成的字符串就会使用...省略部分
    def __repr__(self):
        components = reprlib.repr(self._components)
        # components存的值形式大致是：array('d', [0.0, 1.0, 2.0, 3.0])
        # 这行代码去掉了 array('d',  和 )
        components = components[components.find('['):-1]
        return 'Vector({})'.format(components)

    
    def __str__(self):
        return str(tuple(self))
	
    # 返回格式形如 b'd\x01\x02\x03'
    def __bytes__(self):
        # ord()接收一个字符，返回对应的ASCII或者Unicode数值
        return (bytes([ord(self.typecode)]) + # 形如b'd'
                bytes(self._components))	# 形如b'\x01\x02\x03'

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    # 因为是未知个个数，先算平方，再算和，再sqrt开平方
    def __abs__(self):
        return math.sqrt(sum(x * x for x in self))

    
    def __bool__(self):
        return bool(abs(self))

    
    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        # 先把变量中的内容传到memoryview中，然后.cast再编码成我们的类型
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(memv)


```







### 10.3 协议和鸭子类型

Python中创建功能完善的序列类型无需使用继承，只需实现符合序列协议的方法即可。那么，这里的协议是什么呢？

在面向对象编程中，协议是非正式的接口，只在文档中定义，不在代码中定义。例如，Python的序列协议只需要`__len__`和`__getitem__`两个方法即可。在第一章代码如下：

```python
import collections

Card = collections.namedtuple("Card", ['rand', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = '♣ ♦ ♤ ♧'.split()

    def __init__(self):
        # suit先运行rank再运行
        self._card = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
	
    # 实现len()
    def __len__(self):
        return len(self._card)

    # 实现切片、迭代、随机生成、
    def __getitem__(self, item):
        return self._card[item]
```



### 10.4 可切片

协议是非正式的，没有强制力的。接下来我们在Vector类中实现序列协议。

```python
class Vector:
    # 省略其他行
    
    def __len__(self):
        return len(self._components)
    
    def __getitem__(self, index):
        return self._components[index]

```

现在我们的Vector可以实现len()、切片等操作：

```python
if __name__ == '__main__':
    v1 = Vector([3,4,5])
    print(v1) # (3.0, 4.0, 5.0)
    print(len(v1)) # 3
    print(v1[1]) # 4.0
    print(v1[1:3]) # array('d', [4.0, 5.0])

```

但是现在Vector的切片返回的并不是Vector类的实例，而是array，因为我们在`__getitem__`中的切片，完全是交给array的切片处理的，这样会缺失大量功能。



#### 10.4.1 ==切片原理==

> [参考链接](https://zhuanlan.zhihu.com/p/79752359)

上代码

```python
class MySeq:
    def __getitem__(self, item):
        return item

    
if __name__ == '__main__':
    s = MySeq()
    # 很容易理解
    s[1]			# 1
    
    # slice是内置类型
    s[1:4]			# slice(1, 4, None)
    
    s[1,4,2]		# (1, 4, 2)
    
    s[1:4:2, 9]		# (slice(1, 4, 2), 9)
    
    s[1:4:2, 7:9]	# (slice(1, 4, 2), slice(7, 9, None))
    
```



`slice`是什么？让我们审查一下它

```python
if __name__ == '__main__':
    dir(slice) 
    
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__',
 '__format__', '__ge__', '__getattribute__', '__gt__',
 '__hash__', '__init__', '__le__', '__lt__', '__ne__',
 '__new__', '__reduce__', '__reduce_ex__', '__repr__',
 '__setattr__', '__sizeof__', '__str__', '__subclasshook__',
 
 # 这一行格外注意
 'indices', 'start', 'step', 'stop']

```



`indices` 方法开放了内置序列实现的棘手逻辑，用于优雅地处理缺失索引和负数索引，以及长度超过目标序列的切片。

这个方法 会“整顿”元组，把 start、stop 和 stride 都变成非负数，而且都落在指定长度序列的边界内。

举个例子，假设又个长度为5的序列`'ABCDE'`：

```python
if __name__ == '__main__':    
    a = 'ABCDE'
    b = slice(None, 10, 2).indices(len(a))
    
    # a[:10:2] 等价于a[0:5:2]
    print(b) # (0, 5, 2)
    
	
    c = slice(-3, None, None).indices(len(a))
    # 就是说a[-3:] 等价于a[2:5:1]
    print(c) # (2, 5, 1)
    
```



虽然在本例中，我们没有用到`indices` 方法，因为我们可以依赖于array类型的`self._components`处理。

但是在没有底层序列类型作为依靠是，使用`indices` 方法能节省大量的时间。

其实`range()`函数，也可以切片

```python
if __name__ == '__main__':
    a = []
    b = slice(2,10,3)
    c = b.indices(30)
    for i in range(*c):
        a.append(i)
    print(a)
	
    # [1, 3, 5, 7, 9]

```







#### 10.4.2 返回正确类型的切片

当`[]`中是切片语法，Python通过语法解析，实际上传入了一个内置类型`slice`，这个类有`start`, `stop`, `step`三个属性，缺省值都是`None`。然后交给`__getitem__`处理

```python
class Vector:
    # 省略其他行
    
    def __len__(self):
        return len(self._components)
    
    def __getitem__(self, index):
        cls = type(self)
        
        #经打断点测试，如果传进来一个[1:3]这样的值，进到这个函数的时候，index==slice(1, 3, None)
        if isinstance(index, slice):
            return cls(self._components[index])
        elif isinstance(index, numbers.Integral):
            return self._components[index]
        else:
            msg = '{cls.__name__} indices must be integers'
 			raise TypeError(msg.format(cls=cls)) 
        
```

大量使用`isinstance`可能表明面向对象设计的不好（泛函数可解决），不过在`__getitem__`中使用是合理的。

> 冷知识：
>
> `isinstance(index, numbers.Integral)`中`numbers.Integral`是抽象基类，在``isinstance`中使用抽象基类做测试能让API更加灵活且易更新。但是



### 10.5 动态存取属性

现在Vector支持多维了，那我们如何获取具体向量的值呢？或许我们可以实现获取前几个值，比如`v1.x , v1.y , v1.z , v1.t`。虽然我们可以像以前那样，用`@property`装饰器编写四个变量，但是太麻烦，我们可以使用`__getattr__`方法。

一个类的属性查找失败后，解释器会调用`__getattr__`方法。接下来我们来实现我们的第一版动态存储代码：

```python
shortcut_names = 'xyzt'

def __getattr__(self, name):
    # 获取Vector，后面用
    cls = type(self) 
    
    # 我们的属性 都是一个字母
    if len(name) == 1: 
        # find() 方法检测字符串中是否包含子字符串，并返回索引值
        # 为什么用cls.shortcut_names，而不是直接self.shortcut_names呢？ pass
        pos = cls.shortcut_names.find(name) 
        if 0 <= pos < len(self._components): 
            return self._components[pos]
        
        # 其他情况则抛异常
     msg = '{.__name__!r} object has no attribute {!r}' 
     raise AttributeError(msg.format(cls, name))

```



但是，只实现这个是不够的。试想一下，如果你通过`v1.x `修改实例的第一个值，它能做到吗？显然不能。

为了避免这个矛盾，我们需要继续添加代码。在Vector2d类中，我们通过`@property`属性避免了开发人员手动设定值，在这我们需要实现`__setattr__`方法。

```python
def __setattr__(self, name, value):
    cls = type(self)
    # 只处理单字符
    if len(name) == 1: 
        
        # 'xyzt'让它不能设置
        if name in cls.shortcut_names: 
            error = 'readonly attribute {attr_name!r}'
        # 小写的一个报错    
        elif name.islower(): 
            error = "can't set attributes 'a' to 'z' in {cls_name!r}"
        else:
            error = '' 
        if error: 
            msg = error.format(cls_name=cls.__name__, attr_name=name)
            raise AttributeError(msg)
     
    # 如果不是单字符，那让它可以正确设定值
    super().__setattr__(name, value)


```



> 注意：
>
> 在9.8章我们学过，使用`__slot__`可以让实例中不能出现`__slot__`没有定义的其他属性，不建议这么做，当且仅当内存严重不足的时候，再这么做。

所以，我们在实现`__getattr__`方法的时候，要注意也实现`__setattr__`方法。



### 10.6 散列和快速等值测试

我们要再次实现`__hash__` 和`__eq__`方法，从而可散列。

- `__hash__`

  1. 首先将向量的各个分量hash运算
  2. 将运算后的结果一次异或，即`hash(v1[0]) ^ hash(v1[1]) ^ hash(v1[2])`

- `__eq__`

  为了能够快速比较两个实例：

  1. 首先对双方应用`len()`，有问题直接出局
  2. 然后用`zip()`函数



Q：为什么这样就可以保证唯一了？

A：这里应用了归约思想。但是我也没搞懂！pass



在这里，我们使用`functools.reduce()`函数实现 hash，原理如下：

reduce() 函数的第一个参数是接受两个参数的函数，第二个参数是一个可迭代的对象。假如有个接受两个参数的 $fn$ 函数和一个 lst 列表。调用 reduce(fn, lst) 时，$fn$ 会应用到第一对元素上，即 $fn(lst[0], lst[1])$，生成第一个结果 $r1$。然后，$fn$ 会应用到 $r1$ 和下一个元素 上，即$fn(r1, lst[2])$，生成第二个结果$r2$。接着，调用 $fn(r2, lst[3])$，生成 $r3$……直到最后一个元素，返回最后得到的结果 $rN$。

```python

from array import array
import reprlib
import math
import functools 
import operator 

class Vector:
    typecode = 'd'
    
    def __eq__(self, other): 
        if len(self) != len(other):
            return False
        # zip可以同时迭代两个以上的对象，某个对象迭代完了也不停止，对应返回None，不报错
        # 返回的是一个元组类型，支持拆包，如下
        for a, b in zip(self, other):
            if a != b:
                return False
            return True # ➍

    
    def __hash__(self):
        # 生成器表达式
        hashes = (hash(x) for x in self._components) 
        # operator模块以函数的形式提供了全部中缀运算符 
        # operator.xor(a, b)返回 a 和 b 按位异或的结果
        # reduce是对hashes迭代，最后的一位参数是序列为空的时候的缺省值
        return functools.reduce(operator.xor, hashes, 0) 


```



### 10.7 格式化

pass





### 10.8 本章小结

```Python
# reprlib模块可以生成有限的表示形式。友好的表达式
# reprlib.repr()内的元素如果超过6个，其生成的字符串就会使用...省略部分

# zip可以同时迭代两个以上的对象，某个对象迭代完了也不停止，对应返回None，不报错
```


当`[]`中是切片语法，Python通过语法解析，实际上传入了一个内置类型`slice`，这个类有`start`, `stop`, `step`三个属性，缺省值都是`None`。然后交给`__getitem__`处理



## 第11章 接口:从协议到抽象基类

*抽象类表示接口*



### 11.1 Python中的接口协议

在Python中没有`interface`关键字，而且除了抽象基类，每个类都有接口：类实现和魔术方法等。

按照定义，受保护的属性和私有属性不在接口中，不要违背这些约定。



### 11.2 Python喜欢序列

序列的抽象基类是`Sequence`，正式接口有：

```python
__getitem__ __containers__ __iter__ __reversed__ index count 
```

看下面的例子，虽然我们没有实现`__iter__`但是我们依然可以使用`for in `迭代，因为Python会尝试从0开始迭代对象（后背机制）。

虽然没有实现`__container__`，但依然可以使用`in`运算符。

```python
class Foo:
    
    # 支持切片的最大索引数为2
    def __getitem__(self, pos):
    	return range(0, 30, 10)[pos]
```



### 11.3 猴子补丁

第一章的扑克牌有个重大缺陷，无法洗牌。如果一个类行为像序列，那么Python解释器可以通过`random.shuffle(cls)`函数，给自定义类就地打乱序列。

```python
Card = collections.namedtuple("Card", ['rand', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = '♣ ♦ ♤ ♧'.split()

    def __init__(self):
        # suit先运行rank再运行
        self._card = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
	
    # 实现len()
    def __len__(self):
        return len(self._card)

    # 实现切片、迭代、随机生成、
    def __getitem__(self, item):
        return self._card[item]
```



让我们直接运行试一下

```python
if __name__ == '__main__':
    card = FrenchDeck()
    random.shuffle(card)
    
# 'FrenchDeck' object does not support item assignment
```

报错提示我们“不支持为元素赋值”。也就是说我们还需要提供`__setitem__`方法。

不要忘了我们这一章是讲“猴子补丁”，所以我们可以在运行时，甚至是控制台弥补这个缺陷

```python
>>> def set_card(deck, position, card):
... 	deck._cards[position] = card
...
>>> FrenchDeck.__setitem__ = set_card
>>> shuffle(deck) 

```



我们能够打补丁的关键是，我们知道 FrenchDeck 类的属性是`_cards`



### 11.4 接口和协议的世界观

介绍完Python风格的协议接口后，我们来讲讲为什么要在Python中引入抽象基类。

鸭子类型在很多情况下都十分有用，在生物分类中，属和种就是用**表型系统学**进行分类的，大致意思就是长得挺像的那应该差不离是近亲，鸭子类型的比喻也是比较贴切的。但是，平行进化往往会导致不相关的种产生相似的特征，编程语言中也有这种“偶然的相似性”，比如说下面的经典示例：

```python
class Artist:
    def draw(self): ...

class Gunslinger:
    def draw(self): ...

class Lottery:
    def draw(self): ...
```

我们不能因为上面的函数签名都一致，而说他们有相同的抽象（功能）。

那么在生物分类中，是用**支序系统学**解决这个问题的，即按照从共同祖先继承的特征分类（DNA鉴定）。

我们分的这么清楚有啥好处嘛？当然，你抓住一只动物烹饪时，和啥比较像会决定你怎么烹饪它或者不吃。养动物时，散养还是圈养（对病原体的抗性）DNA的作用就大多了。。。

所以建议在**鸭子类型**的基础上，增加**白鹅类型**。即：只要`cls`是抽象基类，就可以使用`isinstance(obj, cls)`

> pass 剩下的看不懂了

抽象基类就是：有子类继承时，有必须实现的方法（接口），也有些共通的就实现在了抽象基类中了。



### 11.5 定义抽象基类的子类

现在，我们打算把扑克牌定义为`collections.MutableSequence` 的子类

```python
import collections

Card = collections.namedtuple("Card", ['rand', 'suit'])

class FrenchDeck2(collections.MutableSequence):
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = '♣ ♦ ♤ ♧'.split()

    def __init__(self):
        # suit先运行rank再运行
        self._card = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
	
    # 实现len()
    def __len__(self):
        return len(self._card)

    # 实现切片、迭代、随机生成、
    def __getitem__(self, item):
        return self._card[item]

    # 为了实现洗牌，只需要再实现这个
    def __setitem__(self, position, value):
        self._cards[position] = value
        
    # 我们现在是继承自MutableSequence，这是MutableSequence的一个抽象方法 
    def __delitem__(self, position):
        del self._cards[position]

    # MutableSequence的第三个抽象方法 
    def insert(self, position, value):
        self._cards.insert(position, value)

```



FrenchDeck2也继承了基类一些已经实现的方法，比如从 `Sequence` 继承了几个拿来即用的具体方 法：`__contains__、__iter__、__reversed__、index 、count`。FrenchDeck2 从 `MutableSequence` 继承了 `append、extend、pop、remove 、__iadd__`。



### 11.6 标准库中的抽象基类

大多数抽象基类在`collections.abc`中定义

#### 11.6.1 collections.abc

标准库中有两个`abc`模块，一个在`collections.abc`中；还一个`abc.ABC`类，但不用导入，除非你想定义新的抽象基类。

Python3.4 在`collections.abc`模块定义了16个抽象基类，还有多重继承，后面讲。

![image-20210601211858641](图片/image-20210601211858641.png)



- Iterable、Container 和 Sized 　

  各个集合应该继承这三个抽象基类，或者至少实现兼容的协议。Iterable 通过 `__iter__` 方法支持迭代，Container 通过 `__contains__` 方法支持 in 运算符，Sized 通过 `__len__` 方法支持 len() 函数。 

- Sequence、Mapping 和 Set

  这三个是主要的不可变集合类型，而且各自都有可变的子类。

- MappingView 　

  在 Python 3 中，映射方法 .items()、.keys() 和 .values()返回的对象分别是 ItemsView、KeysView 和 ValuesView 的实例。 前两个类还从 Set 类继承了丰富的接口，包含 3.8.3 节所述的全部运算符。

- Callable 和 Hashable 　

  这两个抽象基类与集合没有太大的关系，只不过因为 collections.abc 是标准库中定义抽象基类的第一个模块，而它们 又太重要了，因此才把它们放到 collections.abc 模块中。我从未 见过 Callable 或 Hashable 的子类。这两个抽象基类的主要作用是 为内置函数 isinstance 提供支持，以一种安全的方式判断对象能不 能调用或散列。 若想检查是否能调用，可以使用内置的 callable() 函数；但是没有类似的 hashable() 函数，因此测试对象是否可散列，最好使用 isinstance(my_obj, Hashable)。 


- Iterator 　

  注意它是 Iterable 的子类。我们将在第 14 章详细讨论。 继 collections.abc 之后，标准库中最有用的抽象基类包是 numbers。下面就来介绍。



#### 11.6.2 抽象基类的数字塔

number包定义的是数字塔，其中`Number`是位于最顶端的超类：

- Number
- Complex
- Real
- Rational
- Integral

因此想要检查一个数是不是整数，可以使用`isinstance(x, number.Integral)`；检查一个值是不是浮点可以使用`isinstance(x, number.Real)`

> 注意：
>
> decimal.Decimal 没有注册为 numbers.Real 的虚拟子 类，这有点奇怪。没注册的原因是，如果你的程序需要 Decimal 的精度，要防止与其他低精度数字类型混淆，尤其是浮点数。
>
> https://zhuanlan.zhihu.com/p/157227878



### 11.7 自定义抽象基类

我们应该创建现有抽象基类的子类，或者使用现有的抽象基类注册。需要自己从头编写的情况少之又少。

我们来自定义一个抽象基类Tombola，它能不重复的随机从有限的集合中取出东西，直到没东西

```python
import abc

# 自己定义的抽象基类要继承自 abc.ABC
class Tombola(abc.ABC): 
    
    
    # 装饰抽象方法，一般方法内只有文档字符串
    # 当然你可以有具体方法，但是子类必须重写抽象方法，但是子类又可以用super()调用抽象方法，为子类添加功能，而不是从头开始写。
    @abc.abstractmethod
    def load(self, iterable): 
        """从可迭代对象中添加元素。"""
        
        
    # 这个方法要求抛出LookupError异常这一事实，是接口的一部分，但是不能要求，只能在文档中说明
    @abc.abstractmethod
    def pick(self): 
        """随机删除元素，然后将其返回。
        如果实例为空，这个方法应该抛出`LookupError`。
        """
        
        
    # 抽象基类也可以包含具体的方法    
    def loaded(self): 
        """如果至少有一个元素，返回`True`，否则返回`False`。"""
        # 抽象基类中的具体方法，只能依赖抽象基类定义的接口。
        return bool(self.inspect()) 
    
    
    # 这个方法很笨拙，只是为了强调抽象基类可以提供具体方法，只要依赖其他接口就行。
    # 它的子类当然可以重写这个方法，但不是强制要求，你有个笨拙的方法了。
    # 其实这是静态类型中设计模式对应的 模板方法模式
    def inspect(self):
        """返回一个有序元组，由当前元素构成。"""
        items = []
        while True: 
            try:
                items.append(self.pick())
            except LookupError:
                break
        self.load(items) 
        return tuple(sorted(items))

```



#### 异常层级结构

[中文对照](https://www.runoob.com/python/python-exceptions.html)

```yaml
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
 ├── StopIteration
 ├── ArithmeticError
 │ ├── FloatingPointError
 │ ├── OverflowError
 │ └── ZeroDivisionError
 ├── AssertionError
 ├── AttributeError
 ├── BufferError
 ├── EOFError
 ├── ImportError
 ├── LookupError 
 │ ├── IndexError 
 │ └── KeyError 
 ├── MemoryError
 ... etc.
```

IndexError 是 LookupError 的子类，尝试从序列中获取索引超 过最后位置的元素时抛出。 

使用不存在的键从映射中获取元素时，抛出 KeyError 异常。



#### 11.7.1 抽象基类详解

- `@classmethod` 修饰的方法，第一个参数是`cls`，常用作备选构造器，也称作类方法
- `@staticmethod` 对应Java中的静态方法
- `@abc.abstractmethod` 是装饰抽象方法的，有这个装饰器的类必须继承自`abc.ABC`

那抽象 类方法怎么定义？

本来呢，Python是有@abstractclassmethod、@abstractstaticmethod 和 @abstractproperty 三个装饰器，但是因为装饰器可以堆叠，也就废除了。

```python
class MyABC(abc.ABC):
    
    # abstractmethod 和装饰的函数之间绝对不能有装饰器，毕竟有装饰器那就变味了
	@classmethod
    @abc.abstractmethod
    def an_abstract_classmethod(cls, ...):
        pass

```



#### 11.7.2 实现自定义抽象类子类



下面这个子类，继承了Tombola抽象基类笨重的`loaded()`方法和`inspect()`方法。这里是想表达：

我们可以偷懒，直接继承基类不怎么理想的具体方法，只要我们正确实现了`pick()`和`load()`方法。

```python
import random
from tombola import Tombola

class BingoCage(Tombola): 
    
    def __init__(self, items):
        # 用了一个别的random的API，不要介意
        self._randomizer = random.SystemRandom()
        self._items = []
        self.load(items)
        
    def load(self, items):
        self._items.extend(items)
        # 类似于random.shuffle()函数，就地打乱传入的可迭代实例
        self._randomizer.shuffle(self._items) 
        
        
    # 按照基类的文档中说的，会抛出LookupError异常
    def pick(self): 
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')
    
    # 自己额外添加的方法
    # 把实例本身当作函数调用，会直接吐出一个数
    def __call__(self): 
        self.pick()
```



让我们再来看另一个实现，

```python
import random
from tombola import Tombola

class LotteryBlower(Tombola):
    
    def __init__(self, iterable):
        # 接受任何可迭代的对象
        # 我们开辟了新的内存，保存可迭代对象，避免发生意外。
        # 但是如果可迭代对象中包含可迭代对象，我们仍不能保证不发生意外哦。
        self._balls = list(iterable) 
        
        
    def load(self, iterable):
        self._balls.extend(iterable)
        
        
    def pick(self):
        # 为了按照抽象基类 文档说明中的那样，我们包裹了异常，然后抛出我们自定义的异常
        try:
            position = random.randrange(len(self._balls)) 
        except ValueError:
                raise LookupError('pick from empty LotteryBlower')
        return self._balls.pop(position) 
    
    
    # 我们重写了抽象基类的具体实现，避免调用inspect，达到优化速度
    def loaded(self): 
        return bool(self._balls)
    
    # 同上
    def inspect(self): 
        return tuple(sorted(self._balls))

```



#### 11.7.3 自定义抽象类的==虚拟子类==

白鹅类型的一个基本特性：即便不继承，也有办法把一个类注册为抽象基类的虚拟子类。当我们这么做的时候，我们保证注册的类忠实的完成了抽象基类定义的接口，而Python会相信我们，从而不做检查。如果说谎了，运行时的异常也是能被捕获的。

> 这个特性，怎么说呢，就像我保证，人和猪的基因是完全一致的，请相信我猪也是人的一种。然后Python就相信你了，等到Python让猪玩手机的时候，就报错了，因为猪没有玩手机的能力。



注册虚拟子类的方法是在抽象基类上调用`register` 方法，或许和你知道的注册方式不一样，别着急，你所知的方法其实是语法糖。

```python
from random import randrange
from tombola import Tombola

# 现在TomboList就是 Tombola 的虚拟子类了
@Tombola.register
class TomboList(list):
    
    def pick(self):
        # 因为扩展自list，所以这地方相当于对一个list的实例判断 true/false 所以会调用list的__bool__方法
        if self:
            position = randrange(len(self))
            # 同样继承自 list 的pop
            return self.pop(position)
        else:
            raise LookupError('pop from empty TomboList')
    
    # 这个写法很耐人寻味，它不是实例也可以访问
    load = list.extend 
            
    def loaded(self):
        return bool(self)
    
    def inspect(self):
        return tuple(sorted(self))

# Python3.3及以前版本的写法    
# Tombola.register(TomboList) 
```



注册以后，可以使用issubclass 和 isinstance 函数判断 TomboList 是不是 Tombola 的子类：

```python
# issubclass比较的是类
issubclass(TomboList, Tombola)
    
# isinstance比较的是实例    
t = TomboList(range(100))
isinstance(t, Tombola)

```



类的继承关系，在一个特殊属性`__mro__` 中存放。这个属性的作用很简单：按顺序列出类及其超类，Python会按照这个顺序搜索方法。你会发现TomboList 的`__mro__`  只列出了“真实”的超类，因此也就没有从虚拟超类中继承任何方法。



### 11.8 验证虚拟子类

这一章我们用到了：

- `__subclasses__()` 　这个方法返回类的直接子类列表，不含虚拟子类。
- `_abc_registry` 　只有抽象基类有这个数据属性，其值是一个 WeakSet 对象，即抽象类注册的虚拟子类的弱引用。



pass



### 11.9 使用register的方式注册

虽然现在可以用装饰器的方式注册虚拟子类，但是更常见的做法还是把它当作函数。

官方把内置类型 tuple、str、range 和 memoryview 注册为 Sequence 的虚拟子类的方式：

```python
Sequence.register(tuple)
Sequence.register(str)
Sequence.register(range)
Sequence.register(memoryview)

```



### 11.10 鹅的行为有可能像鸭子

即使不注册，抽象基类也能把一个类识别为==虚拟子类==。比如：

```python
class Struggle:
    
    def __len__(self): 
        return 23

    
>>> isinstance(Struggle(), abc.Sized)
True
>>> issubclass(Struggle, abc.Sized)
True

```

我们可以看到我们没有注册，也没有继承，`__mro__` 显示是Object，但是却确认是 abc.Sized 的虚拟子类。

因为 abc.Sized 实现了一个叫 `__subclasshook__` 的方法。来看一下源码

```python

class Sized(metaclass=ABCMeta):
    __slots__ = ()
    
    @abstractmethod
    def __len__(self):
        return 0
    
    @classmethod
    def __subclasshook__(cls, C):
        
        if cls is Sized:
            if any("__len__" in B.__dict__ for B in C.__mro__): 
                return True 
            
        return NotImplemented
```







### 11.11 本章小结

`random.shuffle()` 可以就地打乱一个实现了`__getitem__`和`__setitem__`的自定义类

`__mro__` ：按顺序列出类及其超类，Python会按照这个顺序搜索方法。

`__subclasses__()`这个方法返回类的直接子类，不含虚拟子类。

`__abs_registry`是抽象基类独占属性，值是一个WeakSet对象，即抽象基类注册的虚拟子类的弱引用





## 第12章 继承的优缺点

*推出继承的初衷，是让新手也能使用只有专家才能设计出来的框架。*

你讲会在本章看到：

- 子类化内置类型的缺点
- 多重继承的顺序



### 12.1 子类化内置类型很麻烦

在Python2.2以前，内置类型都不能子类化。之后可以了，但是子类化的内置类型不会调用用户覆盖的特殊方法。

```python
class DoppelDict(dict):
    # 重写了内置dict的设置值的方法
    def __setitem__(self, key, value):
    	super().__setitem__(key, [value] * 2) 

        
        
# 用字面量方法创建字典，也就是走__init__ 方法，显然子类的init并没有走子类重写的__setitem__
>>> dd = DoppelDict(one=1) 
>>> dd
{'one': 1}

# []运算符调用了我们重写的__setitem__
>>> dd['two'] = 2 
>>> dd
{'one': 1, 'two': [2, 2]}


# 继承自dict的update方法也没有走我们重写的__setitem__
>>> dd.update(three=3) 
>>> dd
{'three': 3, 'one': 1, 'two': [2, 2]}


```



原生类型的这种行为违背了面向对象编程的一个基本原则：始终应该从实例（self）所属的类开始搜索方法，只有`__missing__`方法是特例（3.4.2），它会从实例处开始搜索。

不仅像上例中的`update()`方法没有调用我们重写的`__setitem__()`，`get()`方法也不会调用我们重写的`__getitem__()`

```python
class DoppelDict(dict):
    def __getitem__(self, item):
        return 42
    
if __name__ == '__main__':
    ad = AnswerDict(a='foo')
    
    # 这与我们的期望相符
    ad['a'] # 42
    
    # 这说明get方法没走我们覆盖的__getitem__
    ad.get("a") # foo

    d = {}
    # dict.update 方法忽略了 AnswerDict.__getitem__ 方法
    d.update(ad)
    d['a']	# foo
    d	# {'a': 'foo'}

```



本节所述的问题只发生在 C 语言实现的内置类型内部的方法委托 上，而且只影响直接继承内置类型的用户自定义类。如果子类化使用 Python 编写的类，如 UserDict 或 MutableMapping，就不会受此影响。



### 12.2 多重继承和方法解析顺序

任何实现多重继承的语言都要处理潜在的命名冲突，这种冲突由不相关的祖先类使用了相同的方法名引起的。这种冲突被成为“菱形问题”。

```python
class A:
    def ping(self):
        print('ping:', self)
class B(A):
    def pong(self):
        print('pong:', self)
class C(A):
    def pong(self):
        print('PONG:', self)
        
        
class D(B, C):
    def ping(self):
        super().ping()
        print('post-ping:', self)
        
    def pingpong(self):
        self.ping()
        super().ping()
        self.pong()
        super().pong()
        C.pong(self)

```



B和C都继承自A，且B、C都各自实现了自己的`pong()`方法。D继承自B、C，那如果调用D的`pong()`，会调用谁的呢？

```python
>>> from diamond import *
>>> d = D()
>>> d.pong
# 可以看到打印了小写的pong，也就是调用了B中的方法
pong: <diamond.D object at 0x10066c278>
    
# 那如果我就是想调用C的呢，可以直接调，我们传一个实例进去即可！    
>>> C.pong(d)
PONG: <diamond.D object at 0x10066c278>

```



那么，为什么是B中的方法呢？

Python会按照特定的顺序遍历继承图，这个顺序叫做方法解析顺序（**Method Resolution Order**），这个顺序以元组的形式存储在了类中名为`__mro__` 的属性。

```python
>>> D.__mro__
(<class 'diamond.D'>, <class 'diamond.B'>, <class 'diamond.C'>,
<class 'diamond.A'>, <class 'object'>)
```



如果想把方法委托给超类，推荐的方法是使用内置的`super()`函数，除非你想绕过解析顺序，直接调用某个方法的超类，那可以使用`C.pong(self)` 这种写法

> 我试了一下super().super().ping() 不行  哈哈哈哈哈

### 12.3 多重继承的应用

pass



### 12.4 ==处理多重继承==

使用多重继承容易得出令人费解和脆弱的设计，下面是一些多重继承的建议：

1. **把接口继承和实现继承区分开**

   一开始就要明确，你这个子类是为了什么而创建的：

   - 继承接口，创建子类型，实现“是什么”的关系
   - 继承实现，通过宠用避免代码重复

   通过继承重用代码是实现细节，通常可以换用组合和委托模式。而接口继承则是框架的支柱。

2. **使用抽象基类显式表示接口**

   在Python中，如果一个类的作用是定义接口，应该明确把它定义为抽象基类，即继承自`abc.ABC`

3. **通过混入重用代码**

   如果一个类的作用，是为多个不相关的子类提供方法实现，从而实现重用，但体现不出来这些子类间有什么关系，那么应该把那个类明确定义为混入类（**mixin class**）。这只是一种宏观上的描述，便于开发人员理解业务关系，我们并没有定义什么新的类型。混入类绝对不能实例化，而具体类不能只继承混入类。

4. **在名称中明确指明混入**

   在Python中，没有明确的声明可以将类声明为混入类，所以强烈在名称中加入`...Mixin` 后缀，如 `AbcMixin` 类

5. **抽象基类可以作为混入，反之不行**

   因为抽象基类是可以有具体的实现方法的，所以抽象基类可以是混入的一种表达形式。

   又因为抽象基类可以作为子类的唯一父类，但是混入绝对不可，所以说反之不行。

6. **不要子类化多个具体类**

   > 能实例化的称为具体类

   具体类可以没有具体父类，或者最多只有一个具体父类。也就是说，具体类的父类中除了这有一个具体父类之外，期许的都是抽象基类或混入。例如下面代码，如果 Alpha 是具体类，那么 Beat 和 Gamma 必须是抽象基类或者混入。

   ```python
   class MyConcreteClass(Alpha, Beta, Gamma):
   	"""这是一个具体类，可以实例化。"""
       # ……更多代码……
   
   ```

7. **为用户提供聚合类**

   如果抽象基类和混入的组合对客户代码非常有用，那就提供一个类，使易于理解的方式，把他们结合起来。称之为聚合类（aggregate class）。

8. **优先使用对象组合，而不是类继承**

   > 什么是委托？
   >
   > 将A的实例在B类中生成，并且转化为B的一个私有属性，当我们需要访问A的属性的时候，假如我们只暴露B出来，这时候就只能通过B类来访问A类，这就达到了委托的效果。像是适配器模式
   >
   > 什么是组合？
   >
   > 组合也是关联关系的一种，一种比聚合关系强的关系。组合关系中的部分类不能独立于整体类存在。整体类和部分类有相同的生命周期。
   >
   > 那什么是聚合呢？
   >
   > 聚合是一种强的关联关系。是整体和个体之间的关系。例如，汽车类与引擎类，轮胎类之间的关系就是整体与个体之间的关系。与关联关系一样，聚合关系也是通过实例变量实现的。但是关联关系涉及的两个类在同一层次，而聚合关系中两个类处在不平等的层次上，一个代表整体，一个代表部分。
   >
   > 那什么是关联呢？
   >
   > 关联是类与类之间的联接，使一个类知道另一个类的属性和方法。关联可以是双向，也可以是单向的。一般使用成员变量来实现。
   >
   > 什么是依赖？
   >
   > 依赖关系是类与类之间的联接。一个类依赖于另一个类的定义。如，一个人(Person)可以买车(Car)和房子(House),Person类依赖于Car和House的定义，因为Person引入了Car和House。与关联不同的是，Person类中没有Car和House的属性，Car和House的实例是以参量的方式传入到buy()方法中的。一般而言，依赖关系在Java语言中体现为局部变量，方法形参，或者对静态方法的调用。

   组合和委托可以代替混入，把行为提供给不同的类，但是不能取代接口继承去定义类型层次结构。



### 本章小结

直接实例化Python内置类型,除了`__missing__`方法，其他方法都不会按照实例所属的类的方法开始寻找，即你重写了一个魔术方法，另个魔术方法如果需要调这个方法也不会走你的，而是应该走原生的。



### 第13章 正确重载运算符

pass







## 第14章 可迭代对象

*当程序中出现了设计模式，表明了我对问题的抽象还不够深刻*

根据四人书的那本《设计模式》所说：**迭代器**用于从集合中取出元素；而**生成器**用于“凭空”生成元素。

> 在Python中，通常把**迭代器**和**生成器**视作同一概念

在Python中，所有集合都是可以迭代的，迭代器用于支持：

- for循环
- 构建和扩展集合类型 
- 逐行遍历文本文件 
- 列表推导、字典推导和集合推导 
- 元组拆包 
- 调用函数时，使用 * 拆包实参



### 14.1 序列

我们来自己实现一个序列，为了完善序列协议，我们实现了`__getitem__`和 `__len__`

```python
import re
import reprlib

# 正则表达式
RE_WORD = re.compile('\w+')


class Sentence:
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text) 
        
    def __getitem__(self, index):
        return self.words[index] 
    
    def __len__(self): 
        return len(self.words)
    
    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text) 
```



为什么序列可以迭代？

解释器需要迭代对象 x 时，会自动调用 iter(x)。

内置的 iter 函数有以下作用。

​	 (1) 检查对象是否实现了 `__iter__ `方法，如果实现了就调用它，获取 一个迭代器。

​	 (2) 如果没有实现 `__iter__ `方法，但是实现了 `__getitem__` 方法， Python 会创建一个迭代器，尝试按顺序（从索引 0 开始）获取元素。

​	 (3) 如果尝试失败，Python 抛出 TypeError 异常，通常会提示“C object is not iterable”（C 对象不可迭代），其中 C 是目标对象所属的类。



对实现了 `__getitem__` 的类就可以迭代，完全是为了兼容；未来可能摒弃。这也是鸭子类型的极端形式：

- 只要实现 特殊的 `__iter__` 方法
- 或者实现 `__getitem__` 方法，且 `__getitem__` 方法的参数是从 0 开始的整数（int）

那就认为对象可迭代。



在白鹅类型 理论中，可迭代对象的定义简单些，不过不够灵活：如果对象实现了`__iter__` 方法，那么就认为对象是可迭代的。也不用注册，因为`abc.Iterable` 类实现了`__subclasshook__`。



### 14.2 可迭代对象与迭代器

- 可迭代对象必须实现`__iter__` 方法，但是不能实现`__next__` 方法。

  > 因为从一个可迭代的实例中，你可以获取多个独立的迭代器。如果`__iter__`返回自身，我们做不到独立的维护多个迭代器。

-  迭代器必须实现`__next__` 方法，`__iter__` 方法应该返回自身。

这两个的关系是：

Python从可迭代的对象中获取迭代器。

比如下例，s是一个可迭代对象，背后有可迭代器只是看不到。

```python
s = 'ABC'
for char in s:
	print(char)
```

如果没有`for` 语句，我们则不得不这么写：

```python

if __name__ == '__main__':
    s = 'ABC'
	
    # iter 接收一个可迭代的对象，返回一个迭代器
    it = iter(s) 
    while True:
        try:
            # 在迭代器上使用next() 获取写一个
            print(next(it)) 
            
        #StopIteration异常表明迭代器到头了 
        #for 循环会自动处理迭代器中的 StopIteration异常
        except StopIteration: 
            # 释放引用
            del it 
            break

```



标准的迭代器接口：

- `__next__`

  返回下一个可用的元素，如果没有元素了，抛出  StopIteration异常

- `__iter__`

  一般在类的实现中，会写返回 `self`，也就是说 实例本身就是个可迭代对象，此时就会调用实例的`__next__`方法。

代码中的函数：

- `next()`

  返回传入迭代器，的下一个元素

- `iter()`

  其实就是调用了传入类型的，`__iter__` 方法，所以可以对 `iter()` 的返回值，使用 `next()` 函数



`__iter__`接口是在`Iterable`类中定义的，然后在 `collections.abc.Iterator` 抽象类中实现，实现的内容就是返回类本身；`collections.abc.Iterator` 抽象类还定义了`__next__`抽象方法。下面是`Iterator`的源码：

```python
class Iterator(Iterable):
    __slots__ = ()
    
    # 新增了一个抽象类方法，要求返回下一个迭代的元素，并在没有可迭代元素时，抛出StopIteration异常
    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    # 实现了Iterable的 iter方法
    def __iter__(self):
        return self
    
    # 定义了一个类方法，会在isinstance 的时候调用
    # 如果任何其他类实现了next和iter，那么这个类的实例可以被检查为是一个Iterator
    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            if (any("__next__" in B.__dict__ for B in C.__mro__) and
                any("__iter__" in B.__dict__ for B in C.__mro__)):
                return True
        return NotImplemented

```

> 实测：
>
> `isinstance(self,Iterator)` 首先检测的是类的实例，self 如果传一个 cls 是不行的，传入类应该用 issubclass 方法；
>
> 其次，这个检查必须要求这两个方法都实现，缺一不可；
>
> 最后极端鸭子类型，即如果只实现`__getitem__` 并不会判断为 True。




**迭代器是这样的对象：实现了无参数的 `__next__` 方法，返回序 列中的下一个元素；如果没有元素了，那么抛出 StopIteration 异 常。Python 中的迭代器还实现了 `__iter__` 方法，因此迭代器也可以 迭代。**









### 14.3 典型的迭代器

本节来实现四人书《设计模式》中的迭代器设计模式。这种写法不Python，但是我们可以通过这一节明确可迭代的集合和迭代器对象之间的关系。

下面我们按照设计模式中描述的迭代器设计模式写一个：

```python

import re
import reprlib

RE_WORD = re.compile('\w+')

class Sentence:
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)
        
    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)
    
    # 上一版我们是通过实现getitem和len实现的，现在我们只实现iter，返回一个可迭代对象
    def __iter__(self): 
        # 根据设计模式，我们只返回一个实例化的迭代器
    	return SentenceIterator(self.words) 

    
class SentenceIterator:
    def __init__(self, words):
        self.words = words 
        # 用于确定下一个要获取的单词
        self.index = 0 
    def __next__(self):
        try:
            word = self.words[self.index] 
        except IndexError:
            raise StopIteration() 
        self.index += 1 
        return word 
    
    # 对于这个实例来说其实没必要实现这个，但是这么做是对的，因为迭代器应该实现两个，而且能够通过 issubclass 检查
    def __iter__(self): 
		return self


```



或许你在想，为什么不在 Sentence 类中，再写个`__next__` 方法，然后将`__iter__` 返回 `self`。这样就让 这个类 自己既是可迭代对象又是自身的迭代器。但是！这种想法非常糟糕！这也是常见的==反模式==！

在《设计模式》一书中，关于迭代器，在“适用性”一节中讲到：

- 访问一个聚合的内容而无需暴露它的内部表示
- 支持对聚合对象的多种遍历
- 为遍历不同的聚合结构提供一个统一的接口

为了能支持多种遍历，我们必须能从同一个可迭代的实例中获取多个独立的迭代器，且各自维护自己的状态，因为迭代器模式的正确实现是：每次调用`iter(my_iterable)`都能新建一个独立的迭代器。

以上我们讨论了典型的迭代器模式，接下来看一下更Python的迭代器。



### 14.4 Python的迭代器模式

就是将上例中`__iter__` 函数返回的迭代器，用生成器函数代替。

> 为什么可以用生成器函数代替？
>
> 生成器也是迭代器，对生成器函数调用next() 同样可返回值，且是惰性返回。

```python
import re
import reprlib

RE_WORD = re.compile('\w+')


class Sentence:
     def __init__(self, text):
         self.text = text
         self.words = RE_WORD.findall(text)
            
     def __repr__(self):
     	return 'Sentence(%s)' % reprlib.repr(self.text)
    
     def __iter__(self):
         # 迭代self.words
         for word in self.words:
            # 产出当前的 word
         	yield word 
            
         # 这个return可以不写，不论写不写都不会报StopIteration异常   
         return 
        
# 完成！ 不需要定义可迭代的对象
```



典型的迭代器 `__iter__`  方法是创建了一个迭代器并返回。

此处的迭代器 `__iter__`  方法是创建了一个生成器对象并返回。



#### 生成器函数

> 生成器是迭代器，它实现了迭代器的协议，所以对生成器使用`next()`，可以返回下一个值，且没有值时会抛出StopIteration异常。



只要Python函数的定义提中有`yield` 关键字，该函数就是生成器函数。调用生成器函数会返回一个生成器对象，即生成器函数就是生成器工厂。

来看一下生成器的行为：

```python
def gen_123():
    yield 1
    yield 2
    yield 3


if __name__ == '__main__':
    # gen_123 是函数对象
    print(gen_123)
    # <function gen_123 at 0x00000268E16E0160>
    
    # 但是调用时，返回的是一个生成器对象
    print(gen_123())
    # <generator object gen_123 at 0x00000268E16C6740>
    
    # 生成器是迭代器，会生成传给yield关键字的表达式的值
    for i in gen_123():
        print(i)
        # 1 2 3
        
    a = gen_123()
    print(next(a))
    print(next(a))
    print(next(a))
    
    # 生成器函数的定义体执行完毕后，生成器对象会抛出StopIteration
    print(next(a))
    # StopIteration

```

生成器函数会创建一个生成器对象，包装生成器函数的定义体。把生成 器传给 next(...) 函数时，生成器函数会向前，执行函数定义体中的 下一个 yield 语句，返回产出的值，并在函数定义体的当前位置暂 停。最终，函数的定义体返回时，外层的生成器对象会抛出 StopIteration 异常——这一点与迭代器协议一致。

> 冷知识：
>
> 生成器函数定义体中的 return 语句会触发生成器对象抛 出 StopIteration 异常。



让我们再看一个例子

```python
def gen_AB():
    print('start')
    yield 'A'
    print('继续')
    yield 'B'
    print('end')


if __name__ == '__main__':
    # 迭代时，for循环的机制与 c= iter(gen_AB()) 一样，然后每次迭代时调用next(c)
    for c in gen_AB():
        # 从结果能看出，此处的print语句 在 生成器函数的print之后执行
        print("-->", c)

        
start
--> A
继续
--> B
end
```



到达生成器函数定义体的末尾时，生成器对象抛出 StopIteration 异 常。for 机制会捕获异常，因此循环终止时没有报错。



### 14.5 惰性迭代器

懒惰的反义词是急迫，其实，惰性求值（lazy evaluation） 和及早求值（eager evaluation）是编程语言理论方面的技术术语。



在进一步精简就是用生成器表达式代替 yield 关键字



### 14.9 标准库中的生成器函数

- `itertools.count()`

  如果不写参数，会从0开始生成整数，提供可选的`start`和`step` 参数，`list(itertools.count())` 电脑直接爆炸。

- `itertools.takewhile(条件函数, 生成器)`

  在条件不为False 时，不停的吐出后面生成器的值。所以联动上面的，可以惰性生成想要的数



### 14.12 深入iter函数

前面说过，在Python中迭代对象 `x` 时，会调用`iter(x)`。`iter()`还有一个鲜为人知的用法，第一个参数传一个函数（或者是实现了`__call__`的对象），函数没有参数且返回一个迭代器；第二个参数是一个哨符，当函数返回这个值的时候，迭代器会触发`StopIteration` 异常，且不产出这个哨符。







### 本章小结



## 第15章 上下文管理器与else

- with 语句能避免错误和减少样本代码
- for、while、try 后面其实也可以 跟 else



### 15.1 你不知道的else

else 子句的行为如下：

- for

  仅当`for` 循环运行完毕，且没有被break语句中止时，才运行`else` 块

- while

  仅当`while` 循环因为条件为 假 而退出，且没有被 break 语句中止时，才运行 `else` 块

- try

  仅当 try 块中没有抛异常 才运行 `else` 块

在以上的情况下，如果发生了异常或者 return、break 或 continue 语句导 致控制权跳到了复合语句的主块之外，else 子句也会被跳过。

---

应用场景：

1. 不用设置控制标志

   ```python
    for item in my_list:
        if item.flavor == 'banana':
        	break
           
    # 不需要引入额外的标志判断       
    else:
    	raise ValueError('No banana flavor found!')
   
   ```



2. try 异常处理

   一开始，你可能觉得没必要在 try/except 块中使用 else 子句。毕竟，在下述代码片段中，只有 dangerous_call() 不抛出异常，after_call() 才会执行，对吧？

   ```python
    try:
        dangerous_call()
        after_call()
    except OSError:
    	log('OSError...')
   ```

   然而，after_call() 不应该放在 try 块中。为了清晰和准确，try 块中应该只抛出预期异常的语句。因此，像下面这样写更好：

   ```python
    try:
        dangerous_call()
    except OSError:
    	log('OSError...')
    else:
    	after_call()
   ```

   现在很明确，try 块防守的是 dangerous_call() 可能出现的错误， 而不是 after_call()。而且很明显，只有 try 块不抛出异常，才会 执行 after_call()。

   在 Python 中，try/except 不仅用于处理错误，还常用于控制流程。 为此，Python 官方词汇表（https://docs.python.org/3/glossary.html#termeafp）还定义了一个缩略词（口号）。



### 15.2 with

> 为了避免 try/finally 语句的样板代码，而存在的。

上下文管理器对象存在的目的时管理 with 语句，就像迭代器存在是为了管理 for 语句。

with 语句的目的是简化 try/finally 模式。这种模式用于保证一段代码运行完毕后执行某项操作，即便那段代码由于异常、return 语句 或 sys.exit() 调用而中止，也会执行指定的操作。finally 子句中的代码通常用于释放重要的资源，或者还原临时变更的状态。

上下文管理器协议包括`__enter__`和 `__exit__` 两个方法。with 语句开始运行时，会在上下文管理器对象上调用 `__enter__` 方法。with语句运行结束后，会在上下文管理器对象上调用  `__exit__` 方法，以此扮演 finally 子句的角色。

> 注意：
>
> 不论控制流程以哪种方式退出 with 块，都会在上下文管理器对象上调用`__exit__`方法，而不是在`__enter__`方法返回的对象上调用。



解释器调用`__enter__` 方法时，除了隐式的 `self` 外不会传入任何参数。传给 `__exit__`的三个参数如下：

- exc_type 　异常类（例如 ZeroDivisionError）。 
- exc_value 　异常实例。有时会有参数传给异常构造方法，例如错误消息，这些 参数可以使用 exc_value.args 获取。 
- traceback 　traceback 对象。



### 15.3 contextlib模块中的实用工具

pass



### 

## 第16章 协程

和生成器函数相比，协程的`yield`  关键字通常出现在表达式的右边（例如，`datum = yield`）

从根本上把 `yield` 视作控制流程的方式，这样就好理解协程了。



### 16.1 生成器如何进化成协程

> 生成器函数的作用不大，在增加了`send()`方法后，变成了协程，就是说生成器可以只用来当迭代器那么用，当你发送数据给生成器函数，那么就变成了协程。为了避免这种不好区分的现象，新版本的Python已经正式区分了生成器和协程；协程函数额外增加`async`关键字标记协程函数，并用`await`当作让步标志。
>
> 所以对于高于Python3.5的用户来说，可以不认为协程由生成器进化的，而是两个东西。

函数中有 `yield` 关键字，就会变成生成器函数。当生成器的调用方除了使用`next()` 拿数据，还使用 `.send(...)` 发送数据，发送的数据会成为生成器函数中 `yield` 表达式的值。因此，生成器可以作为协程使用。协程是指一个过程，这个果撑与调用发协作，产出由调用方提供的值。

除了`.send()` 外，还有：

- `.throw(...)`

  让调用方抛出异常

- `.close()`

  终止生成器

- `.send(...)`

  发送数据，发送的数据会成为生成器函数中 `yield` 表达式的值



为了更好的使用协程，Python对生成器函数做了两处改动：

- 如果在生成器中给`return` 提供值，会抛出`SyntaError` 异常
- 引入 `yield from` 句法，为了节省生成器委托给生成器的样板代码



### 16.2 用作协程的生成器的基本行为



```python
def simple_coroutine():
    print('开始')
    # yield在表达式中使用，如果yield只需接收数据，那么初始的产出值是None，这个值是隐式指定的，因为yield关键字右边没有表达式
    x = yield
    print('传输',x)


if __name__ == '__main__':
    a = simple_coroutine()
    
    # 是生成器对象
    print(a)
    # <generator object simple_coroutine at 0x000001DE72B46890>
    
    # 因为生成器还没启动，也就没法在yield语句暂停，也就无法给它发数据，所以首先我们要先调用next()函数
    next(a)
    # 开始
    
    # 给发送数据后，yield表达式会计算出42；现在协程会恢复，一直运行到下一个yield表达式，或者终止
    a.send(42)
    # 传输 42

    # 这里，控制权流动到协程定义体的末尾，导致生成器像往常一样抛出 StopIteration 异常。
    # 因为它没找到下一个yield关键字，所以运行到定义体的末尾了
    # StopIteration
```

> 可以看到对于生成器函数，直接call 其实并没有运行函数，我们需要`next()` 才会使生成器函数运行到 yield 处



协程可以身处四种状态中的一个。当前状态可以使用`inspect.getgeneratorstate(...)` 函数确定，该函数返回下述字符串中的一个：

- 'GEN_CREATED' 　等待开始执行。 

- 'GEN_RUNNING' 　解释器正在执行。 

  > 只有在多线程应用中才能看到这个状态。此外，生成器对象在自己身上调用 getgeneratorstate 函数也行，不过这样做没什么用。 

- 'GEN_SUSPENDED' 　在 yield 表达式处暂停。 

- 'GEN_CLOSED' 　执行结束。

---

仅当协程处于暂停状态时才能调用`send` 方法，因此始终要调用`next(my_coro)`  或 `my_coro.send(None)`激活协程。

如果在创建协程后，在未激活时send 一个非 None 值都会报错。

最先调用`next(my_coro)` 函数，常称为”预激“协程，即让协程向前执行到第一个 `yield` 表达式，准备好作为活跃协程。

下面我们看个例子，来更好地理解协程

```python
def simple_coro2(a):
    print('-> Started: a =', a)
    b = yield a
    print('-> Received: b =', b)
    c = yield a + b
    print('-> Received: c =', c)


if __name__ == '__main__':
    from inspect import getgeneratorstate
    
    my_coro2 = simple_coro2(14)
    getgeneratorstate(my_coro2)	# 'GEN_CREATED'
    
    # 预激活协程，首先你要记住yield关键字是往外抛数据的
    # 所以在激活的同时，抛出了a，同时协程暂停
    x = next(my_coro2)	# x=a=14
    getgeneratorstate(my_coro2)	# 'GEN_SUSPENDED'
    
    # 在暂停的地方恢复运行，要把b = yield a 这个代码切断了看
    # 此时send的值赋给了b，协程开始运行到下一个yield关键字
    # 发现yield关键字， 并把 a+b的值跑出来
    y = my_coro2.send(28)	# y=a+b=14+28=42
    
    # 将99 赋给c，继续运行到下一个yield关键字
    # 因为到定义体的末尾也没有yiele了，所以抛异常
    z = my_coro2.send(99)	# 没有返回值了哦
    getgeneratorstate(my_coro2)	# 'GEN_CLOSED'

```



### 16.3 使用协程计算移动平均值

我们之前用闭包和装饰器两种方式实现了计算平均值，现在我们用协程来实现相同的功能。

```python

def averager():
    total = 0.0
    count = 0
    average = None
    
    # 无限循环，只要不断地send 就会生成结果；除非调用 close()方法
    while True:
        # yield用于暂停执行协程，同时把结果发给调用方
        term = yield average
        total += term
        count +=1
        average = total/count


```



我们使用协程计算平均值之前，需要预激，但这一步容易忘记，样板代码就很不Python，我们可以使用装饰器预激



### 16.4 预激协程的装饰器



```python
from functools import wraps

def coroutine(func):
"""装饰器：向前执行到第一个`yield`表达式，预激`func`"""

	# wraps装饰器避免被装饰的函数内省信息被覆盖
    @wraps(func)
    def primer(*args,**kwargs): ➊
         gen = func(*args,**kwargs) ➋
         next(gen) ➌
         return gen ➍
	return primer

```



所以我们只需要在上一节的代码上加上这个装饰器，即可预激活：

```python
@coroutine
def averager():
	pass
```



使用`yield from` 句法调用协程时，会自动预激，此时就不能使用上面的装饰器，这个句法后面介绍。



### 16.5 终止协程和异常处理



终止协程的一种方式是：发送某个哨符值，让协程退出。内置的`None` 和`Ellipsis` 等常量经常用作哨符值，甚至还有人`.send(StopIteration)` 。

更一般的做法是：

- `generator.throww(exc_type[, exc_value[, traceback]])`

  使生成器在暂停的 yield 表达式处抛出指定异常，但如果生成器有`try` 处理异常，那么生成器会照常执行到下一个 yield 表达式。

- `generator.close()`

  使生成器在暂停的 yield 表达式处抛出GeneratorExit 异 常。如果生成器没有处理这个异常，或者抛出了 StopIteration 异 常（通常是指运行到结尾），调用方不会报错。如果收到 GeneratorExit 异常，生成器一定不能产出值。

```python

class DemoException(Exception):
 """为这次演示定义的异常类型。"""

def demo_exc_handling():
    print('-> coroutine started')
    while True:
        try:
            x = yield
        # 我们会处理DemoException异常
        except DemoException: 
            print('*** DemoException handled. Continuing...')
        # 没有异常都会走这儿    
        else: 
            print('-> coroutine received: {!r}'.format(x))
    
    # 其实，永远也运行不了
 	raise RuntimeError('This line should never run.') 

```





### 16.4 让协程返回值

写一个只在最后停止协程的时候，才返回值的协程

```python
from collections import namedtuple
Result = namedtuple('Result', 'count average')

def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        # 哨符值 是 None
        if term is None:
            break 
        total += term
        count += 1
        average = total/count
    # return表达式的值会偷偷传给调用方，赋给StopIteration异常的一个属性。
    # 这样做有点不合常理，但是能保证生成器最后的返回是个StopIteration异常。
    return Result(count, average) 
```



```python
    a = averager()
    next(a)
    a.send(5)
    a.send(10)

    try:
        a.send(None)
    except StopIteration as e:
        # 对于最后的异常带的值我们要这么获取！
        print(e.value)
```



那就没有什么优雅的处理方式吗？

有！那就是 `yield from` ，它像`for` 一样会捕获 StopIteration 异常，还会把value 属性的值，变成 yield from 表达式的值。

### 16.7 使用 yield from

`yield from` 是全新的语言结构，它的作用比 yield 要多很多。在其他语言中，类似的结构使用了`await` 关键字。

**`yield from x`   表达式对`x` 对象做的第一件事，就是调用 `iter(x)` ，从中获取迭代器，因此 `x` 可以是任何可迭代对象。**

`yield from` 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来。为了更好理解业务代码，介绍几个术语：

- 委派生成器

  包含`yield from <iterable>` 表达式的生成器函数。（中间商的感觉）

- 子生成器

   `<iterable>` 中获取的生成器，也就是上文说的“子生成器”

- 调用方

  客户端，就是预激和 send 的



```python
from collections import namedtuple

Result = namedtuple('Result', 'count average')


# 子生成器 被 yield from 指向的生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        # 客户端的值会通过委派生成器，最终赋值给 term
        term = yield
        # 哨符值
        if term is None:
            break
        total += term
        count += 1
        average = total / count

    # 终止时，这个return 会被 yield from捕获且返回
    return Result(count, average)


# 委派生成器，中间商
def grouper(results, key):
    # 这个循环有迷糊性，对业务实现无任何帮助，甚至可以优化掉，下面有解释。
    while True:
        # yield from会首先调用averager()的iter，且自动预激 averager()子生成器
        # 如果子生成器没有停止或异常，那么表达式右边永远也不会走
        results[key] = yield from averager()


# 客户端代码，即调用方；业务的源头
def main(data):
    results = {}
    for key, values in data.items():
        # group是生成器对象，result是空字典，此处还没运行grouper中的内容
        group = grouper(results, key)

        # 这里没有yield from 没有预激,所以需要手动激活
        # 这会先运行 grouper()函数 , grouper()函数运行至迭代averager()函数,并在yield处停下,且无返回值
        next(group)
        for value in values:
            # value赋值给averager函数的term,且继续循环一次while至 yield 关键字暂停,无返回值
            # 在for循环中重复
            group.send(value)
            
        # None通过中间商赋值给averager函数的term,导致break,走完函数体,返回namedtuple且被yield from 捕获,作为返回值
        # 此时指针来到grouper函数,results[key]获得返回值;指针期望在下一个yield关键字处停住,或者走完生成器函数grouper抛出StopIteration异常
        # 因为当前指针在while循环体中,所以循环;发现yield from 关键字,进入子生成器函数averager中
        # 走到yield处停住,且没有返回值。所以这个while挺具有迷惑性，只是为了发送None时，让指针在grouper函数中能找到下一个yield关键字好停下
        # 因为send(None)在大for循环的最后，所以马上group变量会申请新的生成器函数，而上次循环的那个停在averager yield关键字处的生成器会被垃圾回收
        # 所以其实有个优化，可以去掉grouper函数的while循环，然后在那条语句后面添一条 yield ，让它停住不会走完代码导致委派生成器抛出Error
        group.send(None)

    # print(results) # 如果要调试，去掉注释
    report(results)


# 输出报告
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print('{:2} {:5} averaging {:.2f}{}'.format(result.count, group, result.average, unit))


data = {
    'girls;kg':
        [40.9, 38.5, 44.3, 42.2, 45.2, 41.7, 44.5, 38.0, 40.6, 44.5],
    'girls;m':
        [1.6, 1.51, 1.4, 1.3, 1.41, 1.39, 1.33, 1.46, 1.45, 1.43],
    'boys;kg':
        [39.0, 40.8, 43.2, 40.8, 43.1, 38.6, 41.4, 40.6, 36.3],
    'boys;m':
        [1.38, 1.5, 1.32, 1.25, 1.37, 1.48, 1.25, 1.49, 1.46],
}


if __name__ == '__main__':
    main(data)

```



###   16.8 yield from的意义

`yield from` 表达式的值是子生成器终止时传给 `StopIteration` 异常的第一个参数。

客户端传给中间商的异常，除了`GeneratorExit` 之外，都会传给子生成器的`throw()` 方法。如果子生成器调用`throw()` 方法时抛出 `StopIteration` 异常，中间商恢复运行，捕获这个异常。子生成器的其他异常会向上冒泡，传给中间商，没处理的话继续向上冒泡。

如果客户端把`GeneratorExit` 传给 中间商，或者在客户端上调用中间商的`close()` ，那么就会在子生成器上调用` close()` 方法，如果子生成器有` close()` 方法的话，子生成器有` close()` 方法，但是报错了的话异常会网上冒泡，传给中间商。如果子生成器有` close()` 方法，且没有报错，那么中间商就会抛出`GeneratorExit` 异常。

让我们通过一段伪代码，来看一下`RESULT = yield from EXPR` 这行代码背后运行了什么逻辑。

```python
# 首先yield from 会对目标调用 iter()获取子生成器的迭代器
_i = iter(EXPR)
try:
    # 然后对子生成器进行预激，返回值保存在_y 中
	_y = next(_i) 
except StopIteration as _e:
    # 任何时候，子生成器如果抛出StopIteration异常，都会被 yield from 捕获，保存在_r 中
	_r = _e.value 
 # 预激后没有什么异常会走这里    
else:
    # 这个while循环会阻塞中间商，使其成为客户端和子生成器的通道
	while 1: 
        # 表达式右边：抛出预激获取的数据，然后代码会停住
        # 当客户端send值时，会将客户端发送的值赋给_s，代码恢复运行
		_s = yield _y 
		try:
            # 尝试向子生成器转发客户端发送来的数据_s，使子生成器代码恢复运行
			_y = _i.send(_s) 
        # 如果子生成器 抛出StopIteration异常，捕获并将值保存在_r中。
        # 上面在预激时，如果抛出StopIteration，也会这么做
        except StopIteration as _e:
            _r = _e.value
            # 跳出while循环
            break
# 走完代码返回的数据，一定是因为子生成器抛出StopIteration，不论是预激还是在send的时候 
RESULT = _r 


```



我们粗略的了解了 yiled from 的工作原理，当然这是在客户端不调用`.throw(...)` 和`.close(...)` 方法的情况下。如果要调用这两个方法，子生成器可能只是个生成器不支持`.throw(...)` 和`.close(...)` 方法，因此 yield from 结构必须要能处理这种情况，来看看它背后处理的伪代码：

```python
# 仍然是使用iter()函数获取目标的迭代器
_i = iter(EXPR) 
try:
    # 仍然预激，没毛病
	_y = next(_i) 
except StopIteration as _e:
    # 捕获StopIteration异常的值，没区别
	_r = _e.value
else:
    # 运行这个循环，中间商会阻塞
    while 1: 
        try:
            # 表达式右半边，抛出预激的返回值，并暂停在这
             _s = yield _y 
                
        # 这是处理当客户端发送GeneratorExit用于关闭中间商和子生成器的；
        # 也可以用.close，但是这不是考虑到子生成器可能没实现这个嘛
        except GeneratorExit as _e: 
            try:
                # 尝试将子生成器的迭代器的close函数赋给一个变量,一等函数才能做到这样
                _m = _i.close
            # 对象没有这个属性会报AttributeError异常
            except AttributeError:
                pass
            else:
                # 走到这儿说明有，那么运行它
                _m()
            # raise是手动引发一个异常，此处是GeneratorExit
            raise _e
        
        # 这是用来处理，如果客户端是通过.throw(...) 方法传进来的异常。
        # 同样，子生成器可以只是一个迭代器，而没有throw()方法可以调用
        except BaseException as _e: 
            # exc_info() 方法会将当前的异常信息以元组的形式返回
            _x = sys.exc_info()
            try:
                # 一样，尝试调用，看有没有子生成器有没有throw
                _m = _i.throw
            except AttributeError:
                # 没有就向上冒泡
                raise _e
            else: 
                try:
                    # 如果子生成器有throw方法，则调用
                    _y = _m(*_x)
                except StopIteration as _e:
                        _r = _e.value
                		break
        # 传进来的值没有问题                
     	else: 
            try: 
                # 如果客户端传进来的是None，就调用子生成器的next()
                if _s is None:
                    _y = next(_i)
                    
                # 排除上面的特例，终于可以传值了    
                else:
                    _y = _i.send(_s)
            # 如果送值后，子生成器抛出StopIteration，那么捕获它
            except StopIteration as _e:
                _r = _e.value
                break
# 最终 yield from 会捕获到子生成器StopIteration异常的值           
RESULT = _r
```



### 16.9 应用场景是什么？

兄弟们我们学了这些个东西到底是为了什么啊，心态崩了啊。

仿真！是协程的经典应用。





```python
Event = collections.namedtuple('Event', 'time proc action')


# 每辆出租车会调用这个函数，传进来ident出租车编号，trips要载几次客，start_time离开车库的时间
def taxi_process(ident, trips, start_time=0): 
    """每次改变状态时创建事件，把控制权让给仿真器"""
    
    # 新生成一辆出租车时，会返回一个Event，action是离开车库，然后此处代码停住
    # 想要继续激活这个代码时，客户端会send()函数，发送一个时间，然后time接收数据，代码继续进行
    time = yield Event(start_time, ident, 'leave garage') 
    # 这个是表示这辆车要拉几趟车嘛
    for i in range(trips): 
        # 第一次进来，会返回一个Event，表示这辆车拉到客了，然后暂停
        # 客户端再调用函数会send()函数的时候，会将值传给time，代码继续运行
        time = yield Event(time, ident, 'pick up passenger') 
        # 在右边表达式处停住，产出Event，表示乘客下车
        # 再次send()会赋值给time
        time = yield Event(time, ident, 'drop off passenger') 
        
    # 完成次数后会产出，干完活回家了
    # 再次send()的时候，并没有赋值给变量，因为代码走完了，所以会抛出StopIteration异常，出租车干完活了。
    yield Event(time, ident, 'going home') 
    # 出租车进程结束 
```



ok，	让我们看看如何在控制台调用上面的代码：

```python

>>> from taxi_sim import taxi_process
# 生成第一辆taxi
>>> taxi = taxi_process(ident=13, trips=2, start_time=0)
# 别忘了预激，我们并没有用yield from
>>> next(taxi) 
Event(time=0, proc=13, action='leave garage')
# 在控制台中 _ 绑定的是前一个结果即taxi变量，此处等价于taxi.time；
#  +7 意味着七个单位时间后
>>> taxi.send(_.time + 7) 
# 返回值
Event(time=7, proc=13, action='pick up passenger') 
>>> taxi.send(_.time + 23) 
Event(time=30, proc=13, action='drop off passenger')
>>> taxi.send(_.time + 5) 
Event(time=35, proc=13, action='pick up passenger')
>>> taxi.send(_.time + 48) 
Event(time=83, proc=13, action='drop off passenger')
>>> taxi.send(_.time + 1)
Event(time=84, proc=13, action='going home') 
# 预设taxi是要拉两次客，上面的返回也看到已经回家了，此时再send 就会抛出StopIteration异常
>>> taxi.send(_.time + 10) 
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
StopIteration
```



但是代码肯定不能在控制台里调用把吧，让我们再封装一下：

```python
def taxi_process(ident, trips, start_time=0): 
    """和上面的一致，此处省略"""
    pass


Event = collections.namedtuple('Event', 'time proc action')

# 下面的类，构造参数需要传入的实例如下
procs_map = {0: taxi_process(ident=0, trips=2, start_time=0),
             1: taxi_process(ident=1, trips=4, start_time=5),
             2: taxi_process(ident=2, trips=6, start_time=10)}


def compute_duration(previous_action):
    """使用指数分布计算操作的耗时"""
    if previous_action in ['leave garage', 'drop off passenger']:
        # 新状态是四处徘徊
        interval = SEARCH_DURATION
    elif previous_action == 'pick up passenger':
        # 新状态是行程开始
        interval = TRIP_DURATION
    elif previous_action == 'going home':
        interval = 1
    else:
        raise ValueError('Unknown previous_action: %s' % previous_action)
        
    return int(random.expovariate(1/interval)) + 1




class Simulator:
    
    # events属性存储Event实例；procs是一个字典，用来存储出租车编号和Event协程生成器的一一映射
    def __init__(self, procs_map):
        # deque是双向队列，而queue.PriorityQueue是优先级队列，pop总是返回队列中优先级最高的元素，默认数字越小优先级越高，此处也就是时间单位为优先级
        # 优先队列是离散事件仿真系统的基础组件
        self.events = queue.PriorityQueue()
        # 传进来的procs_map本来就是一个map，因为业务上我们会对回家的taxi删除，所以我们使用自己的副本
        self.procs = dict(procs_map)

    # 接收一个仿真结束的时间    
	def run(self, end_time): 
        """排定并显示事件，直到时间结束"""
        
        # 排定各辆出租车的第一个事件
        for _, proc in sorted(self.procs.items()): 
            # 预激各个协程，使每个协程向前执行到yield关键字处，此处还返回了一个Event namedtuple对象，并赋值给了first_event
            first_event = next(proc) 
            
            # 将事件添加到优先队列events中
            self.events.put(first_event)
            
            # 这个仿真系统的主循环时间标志，从0开始
            sim_time = 0 
            
            # 当仿真时间小于结束时间时运行
            while sim_time < end_time: 
                # 如果优先队列events是空的，那么跳出循环
                if self.events.empty(): 
                    print('*** end of events ***')
                    break
                    
                # 获取优先队列events中，time最小的元素 # 感觉这个优先队列是基于第一个元素排序的
                # get会从优先队列里删除这个元素（当然要删除，不然你不就每次都get那一个了吗
                current_event = self.events.get() 
                # 拆包Event namedtuple元素，并更新仿真钟的时间sim_time
                sim_time, proc_id, previous_action = current_event 
                # 显示当前数据
                print('taxi:', proc_id, proc_id * ' ', current_event) 
                
                # 根据Event元素的proc_id，获取对应的协程对象
                active_proc = self.procs[proc_id] 
                # 将action传进一个函数，这个函数根据当前的action生成一个随机时间单位，并返回
                next_time = sim_time + compute_duration(previous_action) 
                try:
                    # 向当前的协程对象发送下一个时间，并返回下一个Event对象；或在完成时产出StopIteration
                    next_event = active_proc.send(next_time) 
                except StopIteration:
                    # 说明这辆出租车完成任务了，在我们维护的procs字典中删除这个协程
                    del self.procs[proc_id] 
				# 说明能正常send，将返回的事件，记录在events优先队列中
                else:
                    self.events.put(next_event) 
            # while的else只会在循环体正常运行完，才会走        
            # 说明当前的仿真钟时间已经大于传进来的停止时间了      
            else: 
                msg = '*** end of simulation time: {} events pending ***'
                print(msg.format(self.events.qsize()))


```



### 16.10 本章小结

我们使用生成器代替线程和回调，实现并发。首次一窥事件驱动型框架的运作方式：在单个线程中使用一个主循环驱动协程执行并发活动。使用协程做面向事件编程时，协程会不断把控制权让步给主循环，激活并向前运行其他协程，从而执行各种并发活动。

使用协程能够以新方式组织代码，不过与递归和多态一样，要花点时间才能习惯这种写法。









## 第17章 使用future处理并发

future 指一种对象，表示异步执行的操作。是`concurrent.futures`模块和`asyncio`包的基础。

表示可能已经完成或者尚未完成的延迟计算。

### 17.1 concurrent.futures线程

`concurrent.futures`模块的主要特色是`ThreadPoolExecutor` 和 `ProcessPoolExcutor` 类，这两个类实现的接口能分别在不同的线程或进程中执行可调用的对象。这两个类在内部维护着一个工作线程或进程池，以及要执行的任务队列。



下面是用多线程实现下载网络资源的代码

```python
from concurrent import futures


# 设定ThreadPoolExecutor类最多使用几个线程
MAX_WORKERS = 20 


# 将图片保存至本地
def save_flag(img, filename): 
    DEST_DIR = 'downloads/'
    path = os.path.join(DEST_DIR, filename)
    with open(path, 'wb') as fp:
        fp.write(img)

# 下载cc国家的国旗
def get_flag(cc): 
    # 下载国旗的网站
    BASE_URL = 'http://flupy.org/data/flags' 
    url = '{}/{cc}/{cc}.gif'.format(BASE_URL, cc=cc.lower())
    resp = requests.get(url)
    return resp.content

# 显示一个字符串，然后刷新 sys.stdout，这样能在一行消息中看到进度
def show(text): 
     print(text, end=' ')
     sys.stdout.flush()


# 下载一个的代码
def download_one(cc): 
     image = get_flag(cc)
     show(cc)
     save_flag(image, cc.lower() + '.gif')
     return cc

# 下载一个图像的函数；这是在各个线程中执行的函数。
def download_many(cc_list):
    # 取最小的,避免创建多余的线程
    workers = min(MAX_WORKERS, len(cc_list)) 
    
    # 按照设定的线程数实例化ThreadPoolExecutor类;
    # 当退出上下文时executor.__exit__方法会调用executor.shutdown(wait=True)方法,它会在所有线程执行完毕前阻塞线程
    # 就是说会在所有线程数跑完业务后，才会执行executor.__exit__方法；即我同时跑了好几个线程，但是我主线程还是会阻塞等待
    with futures.ThreadPoolExecutor(workers) as executor: 
        
        # executor.map()返回值是一个迭代器,迭代器的__next__方法会去调用future的result方法,最终得到的是future的结果
        res = executor.map(download_one, sorted(cc_list)) 
    # 返回线程数量    
    return len(list(res)) 

# 入口函数,传入上面的函数
def main(download_many): 
    t0 = time.time()
	POP20_CC = ('CN IN US ID BR PK NG BD RU JP MX PH VN ET EG DE IR TR CD FR').split() 
    count = download_many(POP20_CC)
    elapsed = time.time() - t0
    msg = '\n{} flags downloaded in {:.2f}s'
    print(msg.format(count, elapsed))


if __name__ == '__main__':
    main(download_many) 
```



> ```python
> # with 上下文管理，退出时会调用executor.__exit__ 导致阻塞，默认是不会阻塞的
> with futures.ThreadPoolExecutor(workers) as executor:     
>    	res = executor.map(download_one, sorted(cc_list)) 
> 
> # 这是不阻塞的写法    
> executor = futures.ThreadPoolExecutor(workers)
> res = executor.map(download_one, sorted(cc_list)) 
> ```

这一章说用future处理并发，但是我们的 future 实例对象呢？

自Python3.4起，标准库中有两个名为`Future`的类：

- concurrent.futures.Future
- asyncio.Future

这两个类的作用相同：实例都表示可能已经完成或者尚未完成的延迟计算。`future` 封装待完成的操作，可以放入队列，完成的状态可以查询，得到结果（异常）后可以获取结果（异常）。

通常情况下我们不应该创建`future`，而只能由并发框架（`concurrent.futures`或`asyncio` ）实例化。原因很简单：`future`表示终将发生的事情，而确定某件事会发生的唯一方式是执行的时间已经安排明白了。因此，只有排定把某件事交给 `concurrent.futures.Executor` 子类处理时，才会创建 `concurrent.futures.Future` 实例。例如，`Executor.submit() `方法的参数是一个可调用的对象，调用这个方法后会为传入的可调用对象”安排档期“，并返回一个`future`。 

客户端代码不应该改变`future`的状态，并发框架在`future`表示的延迟计算结束后会改变`future`的状态，而我们无法控制计算何时结束。

这两种类的`future`都有两种方法：

- `.done()` 方法，这个方法不阻塞，返回的是布尔值，指明`future`链接的可调用对象是否已经开始执行。客户端代码通常不会询问`future`是否已经运行结束，而是会等待通知。因此，这两个`Future`类都有`.add_done_callback()`方法：这个方法只有一个参数，类型是可调用对象，在`future`运行结束后回调。

- `result()`方法，在`future`运行结束后调用时，两个类的行为一致：返回可调用对象的结果（或异常）。

  在`future`未结束时：

  - `concurrent.futures.Future` 实例的`.result()`方法会阻塞调用方所在的线程，即一直等到有结果可返回；方法接收可选参数`timeout`，超时会抛出`TimeoutError`异常。
  - `asyncio.Future` 学到了再说吧。



让我们来看看返回`future`实例的写法，只替换上面代码中的核心方法，其他代码保持不变：

```python
def download_many(cc_list):
    # 这次只选取前五个国家代码
    cc_list = cc_list[:5] 
    # 强制使用3个线程
    with futures.ThreadPoolExecutor(max_workers=3) as executor: 
        to_do = []
        # 国家代码排序，使输出顺序与输入一致
        for cc in sorted(cc_list): 
            # executor.submit()方法给参数中的可调用对象排档期。返回一个future对象
            future = executor.submit(download_one, cc)
            # 存储future
            to_do.append(future) 
            msg = 'Scheduled for {}: {}'
            # 打印国家代号，和对应的future
            print(msg.format(cc, future)) 
 		results = []
        # as_completed() 方法是一个生成器，在没有任务完成的时候，会一直阻塞，当子线程中的任务执行完后，直接用 result() 获取返回结果
        for future in futures.as_completed(to_do): 
            # 获取future的结果
            res = future.result() 
            msg = '{} result: {!r}'
            print(msg.format(future, res)) 
            results.append(res)
            
 	return len(results)
```



运行代码后的输出：

```python
$ python3 flags_threadpool_ac.py
Scheduled for BR: <Future at 0x100791518 state=running> 
Scheduled for CN: <Future at 0x100791710 state=running>
Scheduled for ID: <Future at 0x100791a90 state=running>
# 因为强制使用了三个线程，所以后两个pending中    
Scheduled for IN: <Future at 0x101807080 state=pending> 
Scheduled for US: <Future at 0x101807128 state=pending>
    
# 第一个CN是download_one函数打印的，后面的是as_completed打印的
CN <Future at 0x100791710 state=finished returned str> result: 'CN' 
# 可以看到二三个线程的download_one函数几乎同时完成，后面的是   as_completed输出的 
BR ID <Future at 0x100791518 state=finished returned str> result: 'BR' 
<Future at 0x100791a90 state=finished returned str> result: 'ID'
IN <Future at 0x101807080 state=finished returned str> result: 'IN'
US <Future at 0x101807128 state=finished returned str> result: 'US'
5 flags downloaded in 0.70s
```



### 17.2 阻塞型I/O 和GIL

因为全局解释器锁GIL存在，一个Python进程不能同时使用多个CPU核心，所以对于CPU密集型任务，多线程是假的。但是对于I/O密集型程序，阻塞型I/O函数会释放GIL，再运行一个线程，从而从中受益。



### 17.3 concurrent.futures进程

如果要进行CPU密集型处理，使用这个模块能绕开GIL，利用所有可用的CPU核心。

我们只需要将17.1线程中的代码，稍微一改：

```python
def download_many(cc_lisy):
    # 不需要传入核心数，默认是cpu数量
    with futures.ProcessPoolExecutor() as executor:
        pass

```

如果在多个进程中维护同一变量

```python
with multiprocessing.Manager() as manager:
    # queue 就是一个全局锁的list变量
    queue = manager.list()
    with futures.ProcessPoolExecutor(max_workers=2) as executor:
        executor.submit(non_block_consumer, queue)
        executor.submit(do_queue_job, queue)
```







### 17.4 Executor.map方法详解







### 本章总结



`lelo`库和`python-parallelize`库，可以使用多个进程处理并行任务。

`lelo`包定义了一个`@parallel` 装饰器，可以应用到任何函数上，把函数变成非阻塞：调用被装饰的函数时，函数在一个新的进程中执行。

`python-parallelize`包提供了一个`parallelize`生成器，能把`for`循环分配给多个CPU。





map 写法

```python
executor = futures.ThreadPoolExecutor(workers)
# 返回结果是个生成器，next(...) 的话，会调用各个future的result，导致阻塞
res = executor.map(download_one, sorted(cc_list)) 
# 按顺序迭代
for i in res:
    result = i.result()

# 第二种写法
# 当退出上下文时executor.__exit__方法会调用executor.shutdown(wait=True)方法,它会在所有线程执行完毕前阻塞线程
# 就是说会在所有线程数跑完业务后，才会执行executor.__exit__方法；即我同时跑了好几个线程，但是我主线程还是会阻塞等待
with futures.ThreadPoolExecutor(workers) as executor: 
    res = executor.map(download_one, sorted(cc_list)) 
```



submit写法

```python
executor = futures.ThreadPoolExecutor(workers)
res1 = executor.submit(download_one, sorted(cc_list)) 
res2 = executor.submit(download_one, sorted(cc_list)) 

list_res = [res1,res2]
# as_completed会阻塞，参数中的future，哪个先完成迭代哪个
for i in futures.as_completed(list_res):
    res = i.result()


```



进程写法就是把`ThreadPoolExecutor` 替换成`ProcessPoolExecutor`



## 第18章 asyncio包处理并发

*并发是指一次处理多件事。*

*并行是指一次做多件事。*

*二者不同，但是有联系。*

*一个关于结构，一个关于执行。*

*并发用于定制方案，用来解决可能（但未必）并行的问题。*



真正的并行需要多个核心。笔记本有4个CPU，同时能做的事不能超过4件，但是有超过100多个进程在同时运行。因此实际上大多数过程都是并发处理，而不是并行处理。

本章介绍`asyncio` 包，用事件驱动的协程实现并发。





### 18.1 线程与协程对比

我们实现一个小动画由新开辟的线程实现,然后阻塞主线程,看看效果

```python
import threading
import itertools
import time
import sys

# 定义一个简单的可变对象；go属性用于从外部控制线程
class Signal: 
 	go = True
    
#  这个函数会在单独线程中运行；signal是简单类的实例
def spin(msg, signal): 
    write, flush = sys.stdout.write, sys.stdout.flush
    # 无限循环，itertools.cycle()函数会从指定序列中反复不断地生成元素
    for char in itertools.cycle('|/-\\'): 
        status = char + ' ' + msg
        write(status)
        flush()
        # 实现动画地原因，使用退格符把光标移回来
        write('\x08' * len(status)) 
        time.sleep(.1)
        # go属性不为True，则跳出
        if not signal.go: 
            break
    # 使用空格清除状态消息，把光标移回开头。        
 	write(' ' * len(status) + '\x08' * len(status)) 
    
# 假设是一个耗时任务    
def slow_function(): 
     # 假装等待I/O一段时间
     time.sleep(3)
     return 42
    
def supervisor(): 
     signal = Signal()
     # 创建一个从线程对象
     spinner = threading.Thread(target=spin, args=('thinking!', signal))
     print('spinner object:', spinner) # 打印出一个线程对象
     # 启动线程
     spinner.start() 
     # 阻塞主线程，为了观察从线程的动画
     result = slow_function() 
     # 会导致从线程break动画
     signal.go = False 
     # 等待线程结束
     spinner.join() 
     return result

def main():
     result = supervisor() 
     print('Answer:', result)
    
if __name__ == '__main__':
 	main()

```

Python没有提供终止线程的API。若想关闭线程，必须给线程发送消息，上例中我们使用`Signal`类的`go` 属性，线程最终会注意到，然后干净的退出。



下面来看如何使用`@asyncio.coroutine` 装饰器替代线程，实现相同的行为。

> 使用`asyncio` API 的协程在定义体中必须使用`yield from` ，而不是`yield`。`asyncio` 的协程要由调用方驱动，并由调用方通过`yield from` 调用，或者把协程传给`asyncio` 包中的某个函数，比如`asyncio.async(...)` 或是下面介绍的：



```python
import asyncio
import itertools
import sys

# 交给asyncio处理的协程，就用这个装饰器装饰
@asyncio.coroutine 
def spin(msg): # 不像线程，有关闭线程的参数
    
    write, flush = sys.stdout.write, sys.stdout.flush
    for char in itertools.cycle('|/-\\'):
        status = char + ' ' + msg
        write(status)
        flush()
        write('\x08' * len(status))
        try:
            # 类似于线程中的time.sleep(.1),不过不会阻塞事件循环
            yield from asyncio.sleep(.1)
        # 如果此函数再次激活时抛出了此异常，是因为客户端发出了取消请求    
        except asyncio.CancelledError: 
            break
    write(' ' * len(status) + '\x08' * len(status))
    
    
@asyncio.coroutine
def slow_function(): 
    # 假装等待I/O一段时间
    yield from asyncio.sleep(3) # asyncio.sleep()会让指针交给主循环，在休眠结束时恢复这个协程
    return 42

@asyncio.coroutine
def supervisor(): 
    # asyncio.async()函数会用Task对象包装其中的函数spin并立即返回Task对象，并会给spin函数安排一个运行档期
    spinner = asyncio.async(spin('thinking!')) 
    # 打印一下Task函数看看
    print('spinner object:', spinner) 
    
    # 驱动 模拟IO函数slow_function运行，因为slow_function使用了yield from asyncio.sleep(3)表达式，会把控制权交给了主循环
    result = yield from slow_function() 
    # Task对象调用.cancle方法，会在协程当前暂停的yield处抛出asyncio.CancelledError异常；协程可以捕获此异常，可以延迟取消甚至拒绝
    spinner.cancel() 
    return result

def main():
    # 获取事件循环引用
    loop = asyncio.get_event_loop() 
    # 驱动supervisor协程，让它运行完毕；返回值是调用的返回值。
    result = loop.run_until_complete(supervisor()) 
    loop.close()
    print('Answer:', result)
    
if __name__ == '__main__':
 	main()


```









