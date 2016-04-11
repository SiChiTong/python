# python mro
`MRO` 就是python继承中的父类的搜索顺序

在python2.3以后的版本中使用的`C3`的新的算法，大概在2.2版本之前使用深度优先的算法。

参考python官网的[文档](https://www.python.org/download/releases/2.3/mro/#the-c3-method-resolution-order)

## 1. c3算法
主要解决 本地优先级 和 单调性 的问题

1. **本地优先级**：指声明时父类的顺序，例如C(A,B),如果访问C的对象属性的时候，应该根据声明顺序，
优先查找A类，再查找B类
2. **单调性**：例如C(A,B),在C的解析顺序中，A在B的前面，那么在C的所有子类中，也必须要满足这个顺序


## 2. MRO顺序

    >>> O = object
    >>> class F(O): pass
    >>> class E(O): pass
    >>> class D(O): pass
    >>> class C(D,F): pass
    >>> class B(D,E): pass
    >>> class A(B,C): pass

单个的解析：

    L[O] = O
    L[D] = D O
    L[E] = E O
    L[F] = F O
    
合并起来：

    L[B] = B + merge(DO, EO, DE)

合并规则：
在merge列表中，如果第一个序列mro的第一个类是出现在其它序列，并且也是第一个，或者不出现其它序列，那么这个类就会从这些序列中删除，并合到访问顺序列表中。
另一种说法是：mro序列中头部不在其他序列的尾部，那么删除这个头部，加入到访问序列中，这里的头指的的是列表的第一个元素，其余的都是尾部

再计算C:

    L[C] = C + merge(DO,FO,DF)
         = C + D + merge(O,FO,F)
         = C + D + F + merge(O,O)
         = C D F O

最后计算A：

    L[A] = A + merge(BDEO,CDFO,BC)
         = A + B + merge(DEO,CDFO,C)
         = A + B + C + merge(DEO,DFO)
         = A + B + C + D + merge(EO,FO)
         = A + B + C + D + E + merge(O,FO)
         = A + B + C + D + E + F + merge(O,O)
         = A B C D E F O