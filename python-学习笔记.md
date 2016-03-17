# python学习笔记

## python `__eq__`, `__ne__`函数

如果我们定义了某个类型的 `__eq__` 函数，如果a==b,并不意味着 a != b 是 False ，他们是相互没有关系的。
所以我们在重写某个类型的 `__eq__` 函数的时候最好也写一个 `__ne__` 函数，这样才能确保能够得到我们想要的结果

在比较的过程中可能会出现异常，因为如果比较的对象的类型不同或者没有这个属性，那么就会出现错误，所以在比较的时候最好做类型检查，以防止这种情况出现
测试如下：

    class A:
        def __init__(self,x):
            self.x = x
        def __eq__(self, other):
            try:
                return self.x == other.x
            except AttributeError:
            return False
        def __ne__(self, other):
            try:
                return self.x != other.x
            except AttributeError:
            return False
    
    >>> a = A(1)
    >>> b = A(1)
    >>> c = A(2)
    >>> a==b
    True
    >>> a==c
    False
    >>> a==1
    False
    >>> x = [1,2,b,4,5]
    >>> x.index(2)
    1
    >>> x.index(b)
    2
    >>> x.index(a)
    2
    >>> x.index(c)
    Traceback (most recent call last):
      File "<ipython-input-34-a8e2fa474c61>", line 1, in <module>
      x.index(c)
    ValueError: <__main__.A object at 0x7f10ae4c0b10> is not in list
    
### python函数可变参数
关于args和kwargs
>* `*args`: 任意多个无key的参数，它是一个tuple
>* `**kwargs`: 任意多个有key的参数，它是一个dict

也就是说 `*args` 依靠参数的位置来对应参数值，而 `**kwargs` 使用键值对的形式

测试代码如下：

    def foo(*args, **kwargs):
        print('args=', args)
        print('kwargs=', kwargs)
        print('**********************')


    if __name__ == '__main__':
        foo(1, 2, 3)
        foo(a=1, b=2, c=3)
        foo(1, 2, 3, a=1, b=2, c=3)
        foo(1, 'b', 'c', a=1, b='b', c='c')
执行结果如下：

    args= (1, 2, 3)
    kwargs= {}
    **********************
    args= ()
    kwargs= {'b': 2, 'a': 1, 'c': 3}
    **********************
    args= (1, 2, 3)
    kwargs= {'b': 2, 'a': 1, 'c': 3}
    **********************
    args= (1, 'b', 'c')
    kwargs= {'b': 'b', 'a': 1, 'c': 'c'}
    **********************
