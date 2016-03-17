# python celery模块的使用

Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。
它是一个专注于实时处理的任务队列，同时也支持任务调度。

参考网址：
>* 中文文档的地址
>* http://docs.jinkan.org/docs/celery/index.html
>* 英文网址
>* http://docs.celeryproject.org/en/latest/index.html

## 1. 安装
你可以从 Python Package Index（PyPI）或源码安装 Celery。
用pip安装：

    $ pip install -U celery
用easy_install安装：

    $ easy_install -U celery
    
在安装好了以后，windows下一般会把celery的命令添加到环境变量中，所以在cmd中可以直接使用

## 2. 简单应用
首先你需要一个 Celery 实例，称为 Celery 应用或直接简称应用。
既然这个实例用于你想在 Celery 中做一切事——比如创建任务、管理职程——的入口点，它必须可以被其他模块导入。

首先创建 tasks.py：

    from celery import Celery
    
    app = Celery('tasks', broker='amqp://guest@localhost//')
    
    @app.task
    def add(x, y):
        return x + y
        
Celery 的第一个参数是当前模块的名称，这个参数是必须的，这样的话名称可以自动生成。
第二个参数是中间人关键字参数，指定你所使用的消息中间人的 URL，此处使用了 RabbitMQ，也是默认的选项。
对于 RabbitMQ 你可以写 amqp://localhost ，而对于 Redis 你可以写 redis://localhost .

如果使用redis的话，需要对安装好redis，配置redis也很容易：

    BROKER_URL = 'redis://localhost:6379/0'
URL的格式如下：

    redis://:password@hostname:port/db_number

上面的例子中如果使用redis的话，可以这样配置：

    app = Celery('celerydemo', broker='redis://:smart.53iq.com@192.168.0.62:6379/2')
    
## 运行 Celery 职程服务器
    $ celery -A tasks worker --loglevel=info
上面的命令在windows中，可以直接在cmd中执行，qui中 `tasks` 是上面的创建的 `tasks.py` 文件，所以最好可以定位到那个文件夹下直接执行

在celery的启动中，还有一些其他参数，可以使用 `celery -h` 命令查看
    
    $ celery -h
    
    Options:
      -A APP, --app=APP                 app instance to use (e.g. module.attr_name)
      -b BROKER, --broker=BROKER
                                        url to broker.  default is 'amqp://guest@localhost//'
      --loader=LOADER                   name of custom loader class to use.
      --config=CONFIG                   Name of the configuration module
      --workdir=WORKING_DIRECTORY       Optional directory to change to after detaching.
      -C, --no-color
      -q, --quiet
      --version                         show program's version number and exit
      -h, --help                        show this help message and exit

其中 `workdir=WORKING_DIRECTORY` 是指定celery运行的目录的，这个工作目录需要包括所有的在celery任务中使用到的模块，标准库和第三方安装的模块除外，
然后，从这个目录往下要可以找到celery任务所在的模块，也就是 `-A `所在的模块，这个模块要写全了，其他参数的意思参考官网的文档

以celery任务所在的模块 `celery_handler.py` 为例：
    
    $ (env-api) E:\56iq\api>celery -A handlers.messages.celery_handler --workdir=E:\56iq\api worker --loglevel=info
    
    目录结构如下：
    E:\56iq\api+
               - base
               - ...
               - handlers +
                          - messages+
                                    - celery_handler.py
               - run.py
               - celery_test.py
     
这是一个tornado的应用，celery的测试例子放在了celery_test.py模块中了

window下启动需要定位到项目所在的目录下面开始运行，否则会出现找不到模块的情况，这种情况大部分都是因为运行的目录不对导致的，linux下应该也是类似的


## 调用任务
在celery职程服务器启动的时候，可以在另一个cmd下调用任务，可以直接在python的交互式命令下进行：
调用任务使用`delay()`，这是 `apply_async()` 方法的快捷方式，该方法允许你更好地控制任务执行

    >>> from tasks import add
    >>> add.delay(4, 4)
    
这个任务已经由之前启动的职程执行，并且你可以查看职程的控制台输出来验证。
调用任务会返回一个 AsyncResult 实例，可用于检查任务的状态，等待任务完成或获取返回值（如果任务失败，则为异常和回溯）。


## 保存结果
如果你想要保持追踪任务的状态，Celery 需要在某个地方存储或发送这些状态。可以从内建的几个结果后端选择：SQLAlchemy/Django ORM、 Memcached 、 
Redis 、 AMQP（ RabbitMQ ）或 MongoDB ， 或者你可以自制。

现在以redis作为例子：

    from celery import Celery
    
    url1 = 'redis://:smart.53iq.com@192.168.0.62:6379/0'
    url2 = 'redis://:smart.53iq.com@192.168.0.62:6379/2'
    app = Celery('celerydemo', backend=url1, broker=url2)
    
    
    @app.task
    def add(x, y):
        return x+y

结果的保存和中间件都使用redis来配置

配置好结果后端后，让我们再次调用任务。这次你会得到调用任务后返回的 AsyncResult：

    >>> result = add.delay(4, 4)
ready() 方法查看任务是否完成处理:
    
    >>> result.ready()
    False

