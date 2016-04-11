# python with

`with` 语句是一个用来包裹一个由上下文管理器方法执行代码段。这个类似于将`try...except...finally`
代码封装后重用。

上下文管理器需要实现上下文管理协议，上下文管理协议只需要实现 `__enter__()` 和 `__exit__()` 方法。

使用方法：
    
    ```python
    with context_expression [as target(s)]:
        with-body
    ```

## 1. with语句执行步骤

1. context expression 经过计算后获得 context manager
2. context manager 加载 `__exit__()` 以备后用
3. context manager 调用 `__enter__()` 方法
4. 如果`with`中含有 `as` 语句，`__enter__()`方法的返回值赋给`as`后面的变量（例如 `target(s)`），
5. 执行语句体（`with-body`）
6. 调用 context manager 的 `__exit__()` 方法. 如果在执行语句体过程中发生了异常，那么异常的
`type`,`value` 和 `traceback` 会作为参数传递给`__exit__()`，这个可以通过 `sys.exc_info()`
方法获取. 否则传递三个 `None` 作为`__exit__()`方法作为参数

如果语句体是由于异常导致退出，并且`__exit__()`方法的返回值是 `False`，异常信息将被重新抛出。
如果返回值是 `True`，则异常不会被抛出，会正常执行 `with` 表达式下面的代码
 
如果语句体不是因为异常导致的退出，那么`__exit__()`方法的返回值将被忽略，并且执行正常类型的退出

**注意**：

1. `with` 表达式保证如果`__enter__()` 方法返回没有异常，那么`__exit__()`方法将总是会被执行.
因此，如果在参数赋值到 `target` 列表的时候出现异常，这个时候的处理和第六步中在语句体中发生异常做相同的处理。

2. 在语句体中执行了`return`操作，`__exit__()`方法调用的时候参数全是`None`，并且原来的返回值会被忽略

3. 如果在`__enter__()`方法中出现了异常，那么`__exit__()`方法是不会调用的，直接就抛出了异常信息

4. 一般情况下，对于会抛出异常的语句体还是需要使用 `try...except...` 来捕获整个 `with` 语句的异常

## 2. 自定义上下文管理器

    class WithDemo(object):
        def __init__(self):
            pass
    
        def __enter__(self):
            print('__enter__()...')
            # raise Exception("enter error...")
            return self
    
        def __exit__(self, exc_type, exc_val, exc_tb):
            print(exc_type, exc_val, exc_tb)
            print('__exit__()...')
            raise Exception("exit error...")
            # return True
    
    with WithDemo() as demo:
        print('with body start...')
        print('demo:', demo)
        return "hello python..."
        # raise Exception("with body error...")
        print('with body end...')
        
## 3. contextlib
`python`标准库自带的模块，带有`contextmanager`装饰器和`closing`函数，用来创建上下文管理器

### 1. contextmanager

    @contextmanager
    def demo():
        print("__enter__() method...")
        yield "hello world..."
        print("__exit__() method...")
        
`contextmanager` 必须装饰一个生成器，而且只能由一个`yield`，否则报错.其中`yield`前面的语句是在
`__enter__()`中执行，之后的代码在 `__exit__()`中执行。`yield` 参数就是`__enter__()`方法的返回值

### 2. closing
`closing`实现代码如下：

    class closing(object):
        def __init__(self, thing):
            self.thing = thing
        def __enter__(self):
            return self.thing
        def __exit__(self, *exc_info):
            self.thing.close()

测试代码如下：

    class Demo(object):
        def __init__(self):
            pass
        def close(self):
            pass
    
    with closing(Demo()):
        pass
使用`closing`的关键是需要实现`close`方法，否则会报错.

        
### 3. 多个with语句
在python2.x中，有个`nested`方法，可以同时执行多个`with`语句，不过在python3.1以后就没有了，
要执行多个语句可以使用`,`分割开，如下：

    with A() as a, B() as b:
        suite
这个和下面这样是相等的：
    
    with A() as a:
        with B() as b:
            suite