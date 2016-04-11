# python super
python 新式类的继承中，在需要调用继承的类中的方法，就需要用到`super`来调用了

首先，经典类中的调用可以直接使用类名，否则会出错；新式类如果也使用类名调用的话，可能会出现
基类多次调用的情况，这个时候就需要使用 `super` 来调用了

**很重要的一点是 `super` 不要简单的想的就是父类，其实它指向的是 `MRO` 中下一个类**

所以这个`super`还是跟python中类的`MRO`有很大关系的

这里有个链接 [Python’s super() considered super!](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/)

`super` 所做的事情如下：

    ```python
    def super(cls, inst):
        mro = inst.__class__.mro()
        return mro[mro.index(cls)+1]
    ```

参数:

1. inst：inst其实是调用`super`的类的实例，所以生成的`MRO`列表也就是调用类的`MRO`列表
2. cls：通过 cls 定位在当前`MRO`中的`index`，并返回对应的类



