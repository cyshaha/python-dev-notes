python面试题



1.celery哪里用到



2.mysql和redis、mongodb比较，优劣势

相比较MySQL，MongoDB以一种直观文档的方式来完成数据的存储。

mongodb相比mysql有高度多样化的数据类型。

MongoDB文档自然映射到现代的面向对象编程语言。使用MongoDB可以避免将代码中的对象转换为关系表的复杂对象关系映射（ORM）层。

mysql更适合事务关系的数据。



3.redis持久化



4.哪里用到装饰器





5.python垃圾回收机制，循环引用会怎么样？

参考：[记一次面试问题——Python 垃圾回收机制 · TesterHome](https://testerhome.com/topics/16556)



python 采用的是**引用计数**机制为主，**标记 - 清除**和**分代收集**两种机制为辅的策略。

**引用计数：**当一个对象被创建或者被引用时,该对象的引用计数就会加1,当对象被销毁时相应的引用计数就会减1,一旦引用计数减为0时,表示该对象已经没有被使用.可以将其所占用的内存资源释放掉。

无法解决循环引用问题

**分代回收：**以空间换时间的操作方式，Python 将内存根据对象的存活时间划分为不同的集合，每个集合称为一个代，Python 将内存分为了 3“代”，分别为年轻代（第 0 代）、中年代（第 1 代）、老年代（第 2 代），他们对应的是 3 个链表，它们的垃圾收集频率与对象的存活时间的增大而减小。新创建的对象都会分配在年轻代，年轻代链表的总数达到上限时，Python 垃圾收集机制就会被触发，把那些可以被回收的对象回收掉，而那些不会回收的对象就会被移到中年代去，依此类推，老年代中的对象是存活时间最久的对象，甚至是存活于整个系统的生命周期内。分代回收是建立在标记清除技术基础之上。分代回收同样作为 Python 的辅助垃圾收集技术处理那些容器对象。

**标记清除（Mark—Sweep）**：是一种基于追踪回收（tracing GC）技术实现的垃圾回收算法。它分为两个阶段：第一阶段是标记阶段，GC 会把所有的『活动对象』打上标记，第二阶段是把那些没有标记的对象『非活动对象』进行回收。那么 GC 又是如何判断哪些是活动对象哪些是非活动对象的呢？

![image-20210713234844208](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210713234844208.png)

对象之间通过引用（指针）连在一起，构成一个有向图，对象构成这个有向图的节点，而引用关系构成这个有向图的边。从根对象（root object）出发，沿着有向边遍历对象，可达的（reachable）对象标记为活动对象，不可达的对象就是要被清除的非活动对象。根对象就是全局变量、调用栈、寄存器。 mark-sweepg 在上图中，我们把小黑圈视为全局变量，也就是把它作为 root object，从小黑圈出发，对象 1 可直达，那么它将被标记，对象 2、3 可间接到达也会被标记，而 4 和 5 不可达，那么 1、2、3 就是活动对象，4 和 5 是非活动对象会被 GC 回收。

标记清除算法作为 Python 的辅助垃圾收集技术主要处理的是一些容器对象，比如 list、dict、tuple，instance 等，因为对于字符串、数值对象是不可能造成循环引用问题。Python 使用一个双向链表将这些容器对象组织起来。不过，这种简单粗暴的标记清除算法也有明显的缺点：清除非活动的对象前它必须顺序扫描整个堆内存，哪怕只剩下小部分活动对象也要扫描所有对象。



6.实现一个类的只读属性

```python
class Computer:
    def __init__(self, name, mem, cpu):
        self.__name = name

    @property
    def name(self):  # 只读， getter方法
        return self.__name


pc2 = Computer('admin', '8G', 8)
print(pc2.name )
# pc2.name = "aa"  # 修改会报错AttributeError: can't set attribute
```





7.常用的魔法方法

```
__repr__
__str__
__iter__
__call__
__new__
__init__
__getattr__
__setattr__
__dir__
__lt__
__le__
```





8.自己实现一个上下文管理类

contexlib



9.实际用到的python内部包有哪些

math	os	sys	datetime	json



10.实现一个，从0-9里取数，10000个不重复的四位数验证码





11.打印出100以内的质数

```python
import math
def func_get_prime(n):
    prime_number_list = []
    for i in range(2,n):
        for j in range(2, i):
            if i % j == 0:
                break
        else:
            prime_number_list.append(i)
    return prime_number_list

print(func_get_prime(100))

# 第二层循环：可以改为 for j in range(2, i//2 + 1)，因为每个数字都不能整除大于自己一半以上的数字
# 更进一步，第二层循环：可以改为 for j in range(2, int(math.sqrt(x)) + 1)，因为一个数字可以整除他的平方根加一以上的数，那一定可以整除平方根或平方根以下的数
```



```python
import math

def func_get_prime(n):
    return filter(lambda x: not [x % i for i in range(2, int(math.sqrt(x)) + 1) if x % i == 0], range(2, n + 1))
# [x % i for i in range(2, int(math.sqrt(x)) + 1) if x % i == 0]
# 上面这段代码的意思是从2到int(math.sqrt(x)) + 1遍历，如果发现x除以i余数不为0的，就加到列表里。
# 然后用filter过滤列表为空的即为素数
```



12.如何把[1， 1.4， 2.0， 3.6， 4.0]转为[1， 1.4， 2， 3.6， 4]



13.求1-100的和有哪些方法

```python
from functools import reduce

res = reduce(lambda x,y:x+y, range(1,101))
print(res)
```





14.一个列表[1,2,3,4,5,6]，转化为一个列表，其中奇数加一，偶数乘以2

```python
l = [1,2,3,4,5,6]
a = [x+ 1 if x % 2 == 1 else x*2 for x in l]
print(a)
# 一个条件的，可以把条件放后面，a = [x for x in l if x % 2 == 0]
```



15.手写一个装饰器





16.下面代码打印顺序

```python
from collections import abc

def decorator(func):
    print("a")
    def wrapper(*args, **kwargs):
        print("b")
        return func(*args, **kwargs)
        print("c")
    return wrapper

@decorator
def worker(*args, **kwargs):
    print("d")

if __name__ == "__main__":
    print("开始")
    worker()
# a
# 开始
# b
# d
# c没打印因为提前return了
```



17.求最长连续子数组

如l = [1, 2, 4, 8,  9, 10]，sublist = [8, 9, 10]

```python
l = [1, 2, 4, 8,  9, 10]

def soluation(l):
    if len(l) <= 1:
        return l
    max_length_sub_list = []
    start = 0
    end = 0
    for i in range(1, len(l)):
        sub_list = []
        if l[i] - l[i-1] > 1:
            end = i
            sub_list = l[start:end]
            # print(sub_list)
            start = i
        if i == len(l) - 1:
            sub_list = l[start:]
        if len(sub_list) > len(max_length_sub_list):
            max_length_sub_list = sub_list

    return max_length_sub_list

print(soluation(l))
```



18.如何防止sql注入





19.有使用过数据库连接池吗



20.怎么读取一个5g大小文件的最后5行，内存只有2g





21.列表内部的实现是什么样的





23.字典内部如何实现





24.数据插入太慢怎么办





25.python如何实现一个抽象基类，什么时候使用抽象基类











