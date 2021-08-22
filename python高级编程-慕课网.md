# 一切皆对象

## 一切皆对象

函数和类也是对象，属于python一等公民

- 可以赋值给一个变量
- 可以添加到集合对象中
- 可以作为参数传递给函数
- 可以当作函数返回值



## type、class、object

```python
a = 1
b = 'abc'

print(type(1))
print(type(int))
print(type(b))
print(type(str))
# type生成class，class生成obj


class Student:
    pass

class MyStudent(Student):
    pass

stu = Student()
print(type(stu))
print(type(Student))

# object是最顶层基类
print(int.__bases__)  # object
print(Student.__bases__)  # object
print(MyStudent.__bases__)

# type也是一个类，通化市type也是一个对象
print(type.__bases__)
print(object.__bases__)
print(type(object))
```

三者关系

![image-20210711165941843](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210711165941843.png)

object是type的实例

type继承了object

type还可以实例化自身



## 常见内置类型

对象的三个特征

- 身份（地址）
- 类型
- 值



1.None

全局只有一个

2.数值

- int
- float
- complex
- bool

3.迭代类型

4.序列类型

- list
- bytes、bytearray、memoryview(二进制序列)
- range
- tuple
- str
- array

5.映射dict

6.集合

- set
- frozenset

7.上下文管理类型（with）

8.其他

- 模块
- class和实例
- 函数
- 方法
- diamagnetic
- object对象
- type
- ellipsis
- notimplemented类型



# 魔法函数

Python里面的魔法函数，是以带双下划线开头和结尾，可以帮助类增强一些功能。这样方法可以在特定的情况下被Python调用,而几乎不用直接调用。

例如

```python
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __getitem__(self, item):
        return self.employee[item]

company = Company(["a", "b", "c"])

# __getitem__ 方法使对象可以遍历
for i in company:
    print(i)
```



## 魔法函数一览

### 非数学运算

字符串表示

```python
__repr__
__str__
```

集合序列相关

```python
__len__
__getitem__
__setitem__
__delitem__
__contains__
```

迭代相关

```python
__iter__
__next__
```

可调用

```python
__call__
```

with上下文管理器

```python
__enter__
__exit__
```

数值转换

```python
__abs__
__bool__
__init__
__float__
__hash__
__index__
```

元类相关

```python
__new__
__init__
```

属性相关

```python
__getattr__
__setattr__
__getattrbute__
__setattrbute__
__dir__
```

属性描述符

```python
__get__
__set__
__delete__
```

协程

```python
__await__
__aiter__
__anext__
__aenter__
__aexit__
```



### 数学运算

不是重点

一元运算符

```python
__neg__  # -
__pos__  # +
__abs__
```

二元运算符

```python
__lt__
__le__
__eq__
__ne__
__gt__
__ge__
```

算术运算符

```python
__add__  # +
__sub__  # -
__mul__  # *
__truediv__  # /
__floordiv__  # //
__mod__  # %
__divmod__  # 或divmod()
__pow__  # **，或pow()
__round__  # 或round()
```

等...



# 深入类和对象

## 鸭子类型和多态

当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

在鸭子类型中，关注的不是对象类型本身，而是它是如何使用的。我们可以编写一个函数，它接受一个类型为鸭的对象，并调用它的走和叫方法。在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的走和叫方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。

```python
class Cat(object):
    def say(self):
        print("I am a cat")

class Dog(object):
    def say(self):
        print("I am a dog")

class Duck(object):
    def say(self):
        print("I am a duck")

animal_list = [Cat, Dog, Duck]
for animal in animal_list:
    animal().say() 
```



## 抽象基类

abc模块

有时候，我们检查某个类是否具有某种方法，可以使用hasattr：

```python
lass Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list

    def __len__(self):
        return len(self.employee)

com = Company(['a', 'b'])
print(len(com))
print(hasattr(com, "__len__"))
```

现在需要强制某个子类必须实现某些方法，如实现了一个web框架，继承cache（redis，cache， memorycache），则需要设计一个抽象基类，指定子类必须实现某些方法：

```python
# 如何去模拟一个抽象基类
class CacheBase():
    def get(self, key):
        raise NotImplementedError

    def set(self, key, value):
        raise NotImplementedError

class RedisCache(CacheBase):
    def set(self, key, value):
        pass


redis_cache = RedisCache()
redis_cache.set('key', 'value')
```

假如我们没实现set方法，则上面只有创建对象并调用方法才会抛出异常，现在想初始化就抛出异常，则可以使用abc模块，使用abc.abstractmethod装饰器：

```python
import abc

class CacheBase(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def get(self, key):
        raise NotImplementedError

    @abc.abstractmethod
    def set(self, key, value):
        raise NotImplementedError

class RedisCache(CacheBase):
    def set(self, key, value):
        pass


redis_cache = RedisCache()
# TypeError: Can't instantiate abstract class RedisCache with abstract methods get
```



## isinstance判断类型



```python
class A:
    pass

class B(A):
    pass

b = B()
print(isinstance(b, B))
print(isinstance(b, A))
print(type(b) is B)
print(type(b) is A)
```



## 类变量和实例变量

```python
class A:
    aa = 1  # 类变量
    def __init__(self, x, y):
        self.x = x
        self.y = y

a = A(2, 3)

# 可以通过对象向上找类变量
print(a.x, a.y, a.aa)
print(A.aa)
# 但是不可以通过类寻找对象变量中
# print(A.x)

A.aa = 11
a.aa = 100
print(a.x, a.y, a.aa)
print(A.aa)
```



## 类属性和实例属性查找顺序

MRO查找属性顺序

```python
class A:
    name = "A"
    # def __init__(self):
    #     self.name = "obj"

a = A()
print(a.name)

class D:
    pass

class C(D):
    pass

class B(D):
    pass

class A(B, C):
    pass

print(A.__mro__)
# (<class '__main__.A'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.D'>, <class 'object'>)
```

```python
class D:
    pass

class E:
    pass

class C(E):
    pass

class B(D):
    pass

class A(B, C):
    pass

print(A.__mro__)
# (<class '__main__.A'>, <class '__main__.B'>, <class '__main__.D'>, <class '__main__.C'>, <class '__main__.E'>, <class 'object'>)
```



## 静态方法类方法

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    def tomorrow(self):
        self.day += 1

    # 静态方法
    @staticmethod
    def parse_from_string(date_str):
        year, month, day = tuple(date_str.split("-"))
        return Date(int(year), int(month), int(day))

    # 类方法
    @classmethod
    def from_string(cls, date_str):
        year, month, day = tuple(date_str.split("-"))
        return cls(int(year), int(month), int(day))


    def __str__(self):
        return "{}/{}/{}".format(self.year, self.month, self.day)

new_day = Date(2021, 7, 12)
new_day.tomorrow()
print(new_day)

# 使用静态方法完成初始化
day2 = Date.parse_from_string("2021-07-13")
print(day2)

# 使用类方法完成初始化
day3 = Date.from_string("2021-07-13")
print(day3)
```



## 数据封装和私有属性

```python
from class_method import Date
import datetime

class User:
    def __init__(self, birthday):
        # __变成私有属性
        self.__birthday = birthday

    def get_age(self):
        return datetime.date.today().year - self.__birthday.year

if __name__ == "__main__":
    user = User(Date(1992, 7, 23))
    print(user.get_age())

    # 变成私有属性就无法直接获取，但是可以使用其他方法获取
    # print(user.birthday)
    print(user._User__birthday)
```



## 自省

自省就是通过一定机制查询到对象的内部结构

```python
class Person:
    """
    人类
    """
    name = "user"

class Student(Person):
    def __init__(self, school):
        self.school = school

if __name__ == "__main__":
    user = Student("aaa")

    # 通过__dict__查询属性
    print(user.__dict__)
    print(user.name)
    # 类的内容更加多一点
    print(Person.__dict__)
    # 还可以动态增加和取出
    user.__dict__["school_addr"] = "SH"
    print(user.school_addr)

    # 通过dir取出对象所有属性，但是没有属性值
    print(dir(user))
```



## super

super可以调用父类的构造方法

```python
class A:
    def __init__(self):
        print("A")

class B(A):
    def __init__(self):
        print("B")
        super().__init__()

if __name__ == "__main__":
    b = B()
# B
# A
```

super并不是直接调用父类的构造方法，而是按照MRO算法寻找构造方法

```python
class A:
    def __init__(self):
        print("A")

class B(A):
    def __init__(self):
        print("B")
        super().__init__()

class C(A):
    def __init__(self):
        print("C")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D")
        super().__init__()

if __name__ == "__main__":
    print(D.__mro__)
    d = D()
```



## mixin

混合模式

1.mixin功能单一 

2.不和基类关联，可以任意基类组合，基类可以不和mixin关联就可以初始化成功

3.在mixin中不使用super用法



## with语句

上下文管理器

参考魔法方法with上下文

```python
class Sample:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 释放资源
        print("exit")

    def do_something(self):
        print("doing someting")

with Sample() as sample:
    # 使用with打开会调用__enter__
    sample.do_something()
    # 结束后会调用__exit__
# enter
# doing someting
# exit
```

使用contextlib实现上下文

```python
import contextlib

@contextlib.contextmanager
def file_open(file_name):
    print("filen open")
    # yild之前相当于__enter__
    yield {}
     # yild之后相当于__exit__
    print("file end")

with file_open("test") as f:
    print("file processing")
```



# 自定义序列类

## 序列类型的分类

通过存储数据类型

- 容器序列：list、tuple、deque，可防放置任意类型数据
- 扁平序列：str、bytes、bytearray、array

序列是否可变

- 可变序列：list、deque、bytearray、array
- 不可变：str、tuple、bytes



## abc继承关系





## +和+=和extend

```python
a = [1, 2]
c = a + [3, 4]
print(c)
a += [3, 4]
print(a)
# [1, 2, 3, 4]
# [1, 2, 3, 4]
```

```python
# +号后面只能加列表
a = [1, 2]
c = a + (3, 4])  # 报错
print(c)
```

```python
# +=后面可以是任意可迭代对象
a = [1, 2]
a += (3, 4)
print(a)
# [1, 2, 3, 4]
```

```python
# extend和append的区别
a = [1, 2]
a.extend(range(3))
print(a)
# [1, 2, 0, 1, 2]
a.append([1,2])
print(a)
# [1, 2, 0, 1, 2, [1, 2]]
```



## 实现可切片对象

```python
import numbers


class Group:
    def __init__(self, name, company, staffs):
        self.name = name
        self.company = company
        self.staffs = staffs

    def __reversed__(self):
        self.staffs.reverse()

    # 实现切片操作
    import numbers
    def __getitem__(self, item):
        cls = type(self)
        if isinstance(item, slice):
            return cls(name=self.name, company=self.company, staffs=self.staffs[item])
        elif isinstance(item, numbers.Integral):
            return cls(name=self.name, company=self.company, staffs=[self.staffs[item]])

    def __len__(self):
        return len(self.staffs)

    def __iter__(self):
        return  iter(self.staffs)

    def __contains__(self, item):
        if item in self.staffs:
            return True
        else:
            return False

group = Group(name = 'hs', company='hope', staffs=['a', 'b', 'c', 'd'])
print(group[:2])

for n in group:
    print(n)

print('a' in group)

reversed(group)
for n in group:
    print(n)
```



## bisect

insort()用于在有序列表中插入元素，另有insort_left()、insort_right()方法，当插入的元素和序列中的某一个元素相同时，该插入到该元素的前面（左边，left），还是后面（右边）。insort()默认为insort_right()方法。

bisect_right用于查找元素在有序列表中的位置。也有bisect_left和bisect_right，用法同insort_left()、insort_right()。

```python
import bisect

# 用来处理已排序的序列

# 二分查找
# 动态维护一个有序的序列
inter_list = []
bisect.insort(inter_list, 3)
bisect.insort(inter_list, 1)
bisect.insort(inter_list, 2)
bisect.insort(inter_list, 6)
bisect.insort(inter_list, 5)
print(inter_list)

print(bisect.bisect(inter_list, 3))
print(bisect.bisect_left(inter_list, 3))
```



## array

数组array相比于list，只可以放指定数据类型



## 列表推导式、生成器表达式、字典推导式

列表推导式

```python
l0 = [i for i in range(21)]
print(l0)

# 带条件
l1 = [i for i in range(21) if i % 2 == 1]
print(l1)

# 带表达式
l2 = [i**2 for i in range(21) if i % 2 == 1]
print(l2)

# 两个条件
l3 = [i**2 if i % 2 == 1 else i + 1 for i in range(21)]
print(l3)
```

生成器表达式

```python
l4 = (i for i in range(21) if i % 2 == 1)
print(type(l4))
for item in l4:
    print(item)
```

字典推导式

```python
odd_dict = {"a":1, "b":2, "c":3}
new_dict = {value:key for key, value in odd_dict.items()}
print(new_dict)
```



# set和dict

## dict的abc继承关系

```python
from collections.abc import Mapping, MutableMapping

# dict属于mapping类型

a = {}
print(isinstance(a, Mapping))
```



## dict常用方法

```python
a = {"a":{"num":1}, "b":{"num":2}, "c":{"num":3}}

# copy，浅拷贝
new_dict = a.copy()
new_dict["a"]["num"] = 0
print(new_dict)
print(a)

# 深拷贝
import copy
new_dict = copy.deepcopy(a)
new_dict["a"]["num"] = 100
print(new_dict)
print(a)

# clear，清空
# a.clear()
# print(a)


# fromkeys
new_list = ["a", "b"]
new_dict = dict.fromkeys(new_list, {"num":1})
print(new_dict)

# get，可以防止keyarror错误，没有返回None
print(a.get("a"))
print(a.get("d"))


# setdefault，如果set的key存在，则取出那个key的值，
# 不存在，则设一个默认值，并修改前字典，返回默认值
defult_value = new_dict.setdefault("c", 3)
print(defult_value)
print(new_dict)

# update
new_dict.update({"d": 4}) # 传字典
new_dict.update(e=1,f=2)  # 传iterbale对象
new_dict.update([("g", 6)])
new_dict.update((("h", 7),))

print(new_dict)
```



## dict子类





## set和frozenset

```python
# set 集合
# fronzenset 不可变集合，可以作为dict的key
# 集合无序不重复，内部也是使用hash，行性能很高

# set(可迭代对象)
s1 = set("abcdee")
s2 = set(['a', 'b', 'c', 'd', 'e'])
s3 = {'a', 'b'}
s3.add('c')
print(s1)
print(s2)
print(s3)

s4 = frozenset("abcde")
# s4.add('f')  #AttributeError: 'frozenset' object has no attribute 'add'
print(s4)


# update
s5 = set("def")
s6 = set("ab")
s6.update(s5)
print(s6)

# difference ，差集
s7 = set("abcd")
s8= set("cde")
rs = s7.difference(s8)
rs2 = s7 - s8
print(rs)
print(rs2)

# 其他集合运算
# & 交集
# | 并集
```



## dict和set实现原理

参考：https://blog.csdn.net/weixin_43064185/article/details/107565845







# 对象引用及垃圾回收

## is和==区别

```python
a = [1,2,3,4]
b = [1,2,3,4]
print(a is b)
print(a == b)
print(id(a), id(b))

# 小整数和小字符串，内部优化，声明一个变量后，不会信开辟一个空间
a = 1
b = 1
print(a is b)
```



## del和垃圾回收

python中垃圾回收采用引用计数

```python
a = object()
b = a
del a
print(b)
# print(a)  # a已经找不到了
```



## 传参

```python
def add(a, b):
    a += b
    return a

if __name__ == "__main__":
    a = 1
    b = 2

    c = add(a,b)
    print(c)  # 3
    print(a)  # 1
    print(b)  # 2
# 因为a是不可变对象，所以add函数内部的a重新开辟了一个空间，不影响原来的a
```

```python
def add(a, b):
    a += b
    return a

if __name__ == "__main__":
    a = [1,2]
    b = [3,4]

    c = add(a,b)
    print(c)
    print(a)
    print(b)
# 因为a是可变对象，所以add函数内部的a还是原来的a，原来的a也被修改了
```

```python
def add(a, b):
    a += b
    return a

class Company:
    def __init__(self, name, staffs=[]):
        self.name = name
        self.staffs = staffs

    def add(self, staff_name):
        self.staffs.append(staff_name)

    def remove(self, staff_name):
        self.staffs.remove(staff_name)

if __name__ == "__main__":
    com1 = Company("com1", ["a", "b"])
    com1.add("c")
    com1.remove("a")
    print(com1.staffs)

    com2 = Company("com2")
    com2.add("d")
    print(com2.staffs)

    com3 = Company("com3")
    com3.add("e")
    print(com2.staffs)
    print(com3.staffs)
    print(com2.staffs is com3.staffs)
# 因为com2和com3定义时都没有进行staffs的初始化，使用的默认值，他们时同一个staffs
```



# 元类

## property动态属性

```python
from datetime import date, datetime
class User:
    def __init__(self, name, birthday):
        self.name = name
        self.birthday = birthday
        self._age = 0

    def get_age(self):
        return datetime.now().year - self.birthday.year

    @property
    def age(self):
        return datetime.now().year - self.birthday.year

    @age.setter
    def age(self, value):
        self._age = value

if __name__ == "__main__":
    user = User("a", date(year=1992, month=1, day=1))
    # print(user.get_age())  # 一般不适用这种方法
    print(user.age)
    user.age = 30
    print(user._age)
```



## getattr和getattribute

getattr  在查找不到属性的时候调用

当找不到属性时，自己添加一些逻辑

```python
from datetime import date

class User:
    def __init__(self, name, birthday):
        self.name = name
        self.birthday = birthday

    def __getattr__(self, item):
        # 当找不到属性时，自己添加一些逻辑
        return "not fund attr"


if __name__ == "__main__":
    user = User("a", date(year=1992, month=1, day=1))
    print(user.age)
# not fund attr 
```

```python
# __getattr__ 在查找不到属性的时候调用

from datetime import date

class User:
    def __init__(self, name, birthday, info={}):
        self.name = name
        self.birthday = birthday
        self.info = info

    def __getattr__(self, item):
        # 当找不到属性时，自己添加一些逻辑
        return self.info[item]


if __name__ == "__main__":
    user = User("a", date(year=1992, month=1, day=1), {"company":"hope"})
    print(user.company)
# hope
```

getattribute 只要用点的方式访问属性，都会使用这个方法

getattribute里会调用getattr

尽量不要重写getattribute

```python
from datetime import date

class User:
    def __init__(self, info={}):
        self.info = info

    def __getattr__(self, item):
        # 当找不到属性时，自己添加一些逻辑
        return self.info[item]

    def __getattribute__(self, item):
        return "attribute"


if __name__ == "__main__":
    user = User({"company":"hope"})
    print(user.company)
    print(user.name)
```



## 属性描述符及查找过程

参考：https://www.cnblogs.com/z-x-y/p/10216249.html

https://www.dazhuanlan.com/2019/12/12/5df13606bab79/

保护属性不受修改、属性类型检查和自动更新某个依赖属性的值等

```
一个类如果实现了__get__,__set__,__del__方法(三个方法不一定要全部都实现)，并且该类的实例对象通常是另一个类的类属性，那么这个类就是一个描述符。__get__,__set__,__del__的具体声明如下：
	__get__(self, instance, owner) 
    __set__(self, instance, value)
    __delete__(self, instance)
其中：
    __get__ 用于访问属性。它返回属性的值，或者在所请求的属性不存在的情况下出现 AttributeError 异常。类似于javabean中的get。
    __set__ 将在属性分配操作中调用。不会返回任
    何内容。类似于javabean中的set。
    __delete__ 控制删除操作。不会返回内容。
    instance参数是实际拥有者的实例，如果没有则为None，第二个owner参数是实际所属的类。
 注意：
    只实现__get__方法的对象是非数据描述符，意味着在初始化之后它们只能被读取。而同时实现__get__和__set__的对象是数据描述符，意味着这种属性是可读写的。   
```

写一个类似django里的orm模型里的字段，用于属性设置的参数检查

```python
import numbers


class IntField:
    # 数据描述符
    def __get__(self, instance, owner):
        return self.value

    def __set__(self, instance, value):
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        self.value = value

    def __delete__(self, instance):
        pass

    
class User:
    age = IntField()

    
if __name__ == "__main__":
    user = User()
    user.age = 30
    print(user.age)

    # 字符串就会报错
    # user.age = "abc"
```

属性查找过程

```
如果user时某个类的实例，那么user.age（以及等价的hetattr(user, 'age')）首先调用__getattribute__。如果类自定义的__getattr__方法，那么在__getattribute__抛出AttributeError的时候就会调用到__getattr__，而对于描述符(__get__)的调用，则发生在__getattribute__内部的。

user = User()
user.age
调用顺序如下：
1.如果"age"出现在User或其基类的__dict__中，且age是data descriptor，那么调用其__get__方法，否则
2.如果"age"出现在obj的__dict__中，那么直接返回obj.__dict__['age']，否则
3.如果"age"出现在User或其基类的__dict__中
	3.1如果是non-data descriptor，那么调用其__get__方法，否则
	3.2 返回__dict__['age']
4.如果User有__getattr__方法，调用那个__getattr__方法，否则
5.抛出异常
```

```python
import numbers
from datetime import date, datetime


class IntField:
    # 数据描述符
    def __get__(self, instance, owner):
        print("__get__")
        return self.value

    def __set__(self, instance, value):
        print("__set__")
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < 0:
            raise ValueError("positive value need")
        self.value = value

    def __delete__(self, instance):
        pass


class NoDataIntField:
    # 非数据属性描述符
    def __get__(self, instance, owner):
        print("__get__")
        return  self.value


class User:
    age = IntField()


if __name__ == "__main__":
    user = User()
    user.age = 30
    # 因为User里的age是数据描述符。所以使用了IntField的__get__方法
    # 所以__dict__里没有age
    print(user.__dict__)
    print(getattr(user, "age"))
```

```python
import numbers
from datetime import date, datetime


class IntField:
    # 数据描述符
    def __get__(self, instance, owner):
        print("__get__")
        return self.value

    def __set__(self, instance, value):
        print("__set__")
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < 0:
            raise ValueError("positive value need")
        self.value = value

    def __delete__(self, instance):
        pass


class NoDataIntField:
    # 非数据属性描述符
    def __get__(self, instance, owner):
        print("__get__")
        return  self.value


class User:
    # age = IntField()
    age = NoDataIntField()

if __name__ == "__main__":
    user = User()
    user.age = 30
    # 因为User里的age是非数据描述符。没有使用NoDataIntField的__get__方法
    # 所以__dict__有age
    print(user.__dict__)
    print(getattr(user, "age"))
```

```python
import numbers
from datetime import date, datetime


class IntField:
    # 数据描述符
    def __get__(self, instance, owner):
        print("__get__")
        return self.value

    def __set__(self, instance, value):
        print("__set__")
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < 0:
            raise ValueError("positive value need")
        self.value = value

    def __delete__(self, instance):
        pass


class NoDataIntField:
    # 非数据属性描述符
    def __get__(self, instance, owner):
        print("__get__")
        return  self.value


class User:
    age = IntField()
    # age = NoDataIntField()

if __name__ == "__main__":
    user = User()
    user.__dict__["age"] = 30
    print(user.__dict__)
    # 虽然在user的__dict__中有age属性，但是user.age还是按照原来的路线查找，还是找不到
    print(user.age)
```



## _ _new_ _ _和_ _ _init_ _

new是在实例创建**之前**被调用的，因为它的任务就是创建实例然后返回该实例对象，是个静态方法。

init是当实例对象创建完成后被调用的，然后设置对象属性的一些初始值，通常用在初始化一个类实例的时候。是一个实例方法。

__new__的返回值（实例）将传递给__init__方法的第一个参数，然后__init__给这个实例设置一些参数。如果new不反悔对象，则不会调用init函数。

```python
class User:
    def __new__(cls, *args, **kwargs):
        print("__new__")
        pass

    def __init__(self, name):
        print("__init__")
        self.name = name


if __name__ == "__main__":
    user = User("a")
# __new__
```

new返回对象

```python
class User:
    def __new__(cls, *args, **kwargs):
        print("__new__")
        return super().__new__(cls)

    def __init__(self, name):
        print("__init__")
        self.name = name


if __name__ == "__main__":
    user = User("a")
# __new__
# __init__
```





# 元类

## 自定义元类

元类是创建类的类

通过一个方法创建类

```python
# 类也是对象，type创建的对象即为类

def create_class(name):
    if name == "user":
        class User:
            def __str__(self):
                return "user"
        return User
    elif name == "company":
        class Company:
            def __str__(self):
                return "company"
        return Company


if __name__ == "__main__":
    Myclass = create_class("user")
    my_obj = Myclass()
    print(my_obj)
```



type可以看作是一个元类，可以创建类

通过type创建类

```python
if __name__ == "__main__":
    User = type("User", (), {})
    my_obj = User()
    print(my_obj)
```

通过type创建带属性的类

```python
if __name__ == "__main__":
    User = type("User", (), {"name":"user"})
    my_obj = User()
    print(my_obj)
    print(my_obj.name)
```

通过type创建带方法的类

```python
def say(self):
    return "i am user"
    # return self.name

if __name__ == "__main__":
    User = type("User", (), {"name":"user", "say":say})
    my_obj = User()
    print(my_obj)
    print(my_obj.say())
```

通过type创建带父类的类

```python
def say(self):
    return "i am user"
    # return self.name

class BaseClass:
    def answer(self):
        return "i am baseclass"

if __name__ == "__main__":
    User = type("User", (BaseClass, ), {"name":"user", "say":say})
    my_obj = User()
    print(my_obj)
    print(my_obj.answer())
```



下面自定义一个元类，要继承type

```python
class MetaClass(type):
    pass


class User(metaclass=MetaClass):
    pass


if __name__ == "__main__":
    my_obj = User()
    print(my_obj)
```

创建一个类的过程如下：先User类是否有metaclass，没有再找父类是否有metaclass，没有再去模块中找metaclass，再没有再使用type创建User类

```python
class MetaClass(type):
    pass


class BaseClass(metaclass=MetaClass):
    def answer(self):
        return "i am baseclass"


class User(BaseClass):
    pass


if __name__ == "__main__":
    my_obj = User()
    print(my_obj)
    print(my_obj.answer())
```

可以以通过metaclass在实例化类时可以做些检查等其他事

```python
class MetaClass(type):
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, *args, **kwargs)


class User(metaclass=MetaClass):
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name


if __name__ == "__main__":
    my_obj = User("a")
    print(my_obj)
```



## 元类实现orm

```python
import numbers

class Field:
    pass


class IntField(Field):

    def __init__(self,  db_column, min_value=None, max_value=None):
        self._value = None
        self.db_column = db_column
        self.min_value = min_value
        self.max_value = max_value
        if min_value is not None:
            if not isinstance(min_value, numbers.Integral):
                raise ValueError("min_value must be int")
            elif min_value < 0:
                raise ValueError("min_value must be positive int")
        if max_value is not None:
            if not isinstance(max_value, numbers.Integral):
                raise ValueError("min_value must be int")
            elif max_value < 0:
                raise ValueError("min_value must be positive int")
        if min_value is not None and max_value is not None:
            if min_value > max_value:
                raise ValueError("min_value must be smaller than max_value")

    def __get__(self, instance, owner):
        print("__get__")
        return self._value

    def __set__(self, instance, value):
        print("__set__")
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < self.min_value or value > self.max_value:
            raise ValueError("value must between min_value and max_value")
        self._value = value


class CharField(Field):
    def __init__(self, db_column, max_length=None):
        self._value = None
        self.db_column = db_column
        if max_length is None:
            raise ValueError("max_length must not be None")
        self.max_length = max_length

    def __get__(self, instance, owner):
        return self._value

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise ValueError("string value need")
        if len(value) > self.max_length:
            raise ValueError("value length excess len of max_length")


class ModelMetaClass(type):
    def __new__(cls, name, bases, attrs, **kwargs):
        if name == "BaseModel":
            return super().__new__(cls, name, bases, attrs, **kwargs)

        fields = {}
        for key, value in attrs.items():
            if isinstance(value, Field):
                fields[key] = value

        attrs_meta = attrs.get("Meta", None)
        _meta = {}
        db_table = name.lower()
        if attrs_meta is not None:
            table = getattr(attrs, "db_table", None)
            if table is not None:
                db_table = db_table
        _meta["db_table"] = db_table
        attrs["_meta"] = _meta
        attrs['fields'] = fields
        del attrs["Meta"]

        return super().__new__(cls, name, bases, attrs, **kwargs)


class BaseModel(metaclass=ModelMetaClass):
    def __init__(self, *args, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)
            return super().__init__()

    def save(self):
        fields = []
        values = []
        for key, value in self.fields.items():
            db_column = value.db_column
            if db_column is None:
                db_column = key.lower()
            fields.append(db_column)
            value = getattr(self, key)
            values.append(str(value))
        sql = "insert into {db_table}({fields} value({values})".format(db_table=self._meta["db_table"],
                                                                fields=",".join(fields),
                                                                values=",".join(values))
        pass


class User(BaseModel):
    name = CharField(db_column="name", max_length=100)
    age = IntField(db_column="age", min_value=0, max_value=100)

    class Meta:
        db_table = "user"


if __name__ == "__main__":
    user = User()
    user.name = "aa"
    user.age = 29
    user.save()
```





# 迭代器和生成器

## 迭代协议

迭代器是访问集合内元素的一种方式，一般用来遍历数据

迭代器和以下表访问的方式不同，迭代器不能返回的，迭代器提供了一种惰性的访问数据的方式





