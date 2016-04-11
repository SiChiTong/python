# python中的else

`else`在python中使用还是挺多的，有些用法还是和其他的语言中不太一样

## 1. if/else
这个是很常用的，条件判断成立则执行`if`下的语句，否则执行`else`下的
    
    ```python
    if condition:
        # executed when condition == True
        ...
    else:
        # executed when condition == False
    ```

## 2. for/else
这个用法的情况会发生在当需要遍历一个集合之类的东西的时候，在找到符合条件的时候`break`出循环，
之后需要判断是否在遍历的时候发生了`break`,可能需要加一个标志位来标记：

    ```python
    car_found_for_repair = False
    for car in cars:
        if needs_repair(car):
            send_for_repair(car)
            car_found_for_repair = True
            break
    if not car_found_for_repair:
        # do something
    ```        

这个时候使用`else`的话，代码就是这样的：
    
    ```python
    for car in cars:
    if needs_repair(car):
        send_for_repair(car)
        break
    else:
        # do something
    ```        
也就是说在`for`循环的时候没有发生`break`，那么`else`的语句就会执行
    
## 3. while/else
这个和上面的`for/else`是一样的
    
## 4. try/except/else
在处理异常的时候的使用，`else` 下的代码会在没有发生异常的时候执行

    ```python
    try:
        # run some code
    except ValueError:
        # run this when ValueError is raised
    else:
        # run this only when no exception is raised
    finally:
        # run this after the code and any exception handling code
    ```

