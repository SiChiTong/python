# python序列化

## 1. python本地序列化
python提供了pickle模块实现序列化

### ①. pickle.dumps()
这个方法将任意对象序列化成一个bytes，然后写入文件

    def fun():
        import pickle
        d = dict(name='aaa', age=20)
        pickle.dumps(d)

也可以使用pickle.dump()直接将对象序列化后写入指定的文件中
    
    def fun():
        import pickle
        f = open('dump.txt', 'wb')
        d = dict(name='aaa', age=20)
        pickle.dump(d, f)
        f.close()


序列化的过程与上面的过程相反，可以使用:
        
    pickle.loads() 将读取到的bytes反序列化
    pickle.load(f) 从一个file-like object中直接反序列化出来
    
    def fun():
        f = open('dump.txt', 'rb')
        d = pickle.load(f)
        f.close()


## 2. 使用json序列化
JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：

JSON类型	               |  Python类型
:----------------------|:----------------------
{}	                   |    dict
[]	                   |    list
"string"	           |    str
1234.56	               |    int或float
true/false	           |    True/False
null	               |    None

json常用的方法有：
>* json.dumps() 将一个对象转成json格式的数据，返回的是一个字符串
>* json.loads() 将一个字符串转成一个python的内置dict类型的数据，返回一个dict

不过如果转换过程中出现了json不默认的类型，则会出错，例如一些自定的class，datetime，等等，这时候需要做一些特殊处理

### ①. 对于一些简单的类型
比如时间类型，可以定义一个cls放在dumps的参数中：
    
    class MyEncoder(json.JSONEncoder):
        """
        将时间类型的转为字符串json.dumps
        """
        def default(self, obj):
            if isinstance(obj, datetime):
                return obj.strftime('%Y-%m-%d %H:%M:%S')
            elif isinstance(obj, date):
                return obj.strftime('%Y-%m-%d')
            elif isinstance(obj, ObjectId):
            # ObjectId是mongodb中bson中的一个类型，也是每个文档的默认的_id的类型
                return str(obj)
            else:
                return json.JSONEncoder.default(self, obj)

使用的时候：

    cfg = json.dumps(cfg, cls=MyEncoder)

### ②. 对于自定义的类：

    class Student(object):
        def __init__(self, name, age, score):
            self.name = name
            self.age = age
            self.score = score

    def student2dict(std):
        return {
            'name': std.name,
            'age': std.age,
            'score': std.score
        }
    s = Student('Bob', 20, 88)
    s = json.dumps(s, default=student2dict)

这样可以针对单个对象进行json序列化，不过，可以将任意class的实例变为dict，因为通常class的实例都有一个__dict__属性，它就是一个dict，
用来存储实例变量.也有少数例外，比如定义了__slots__的class
    
    s = json.dumps(s, default=lambda obj: obj.__dict__)


如果我们要把JSON反序列化为一个Student对象实例，loads()方法首先转换出一个dict对象，
然后，我们传入的object_hook函数负责把dict转换为Student实例：
        
    def dict2student(d):
        return Student(d['name'], d['age'], d['score'])

    json_str = '{"age": 20, "score": 88, "name": "Bob"}'
    s = json.loads(json_str, object_hook=dict2student)



