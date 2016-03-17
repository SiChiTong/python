# python json格式化
python中一个对象转换成json数据的时候，有时候会出错，对象中包含了没法转换的对象，
此时，就需要我们对那些不能转换的对象做一些额外的处理了：

如下，遇到时间类型无法转换的时候可以用下面的方法处理：
### 1. 加入下面这个方法（这个是处理时间的，有时候需要做自己的处理）

    class JSONEncoder(json.JSONEncoder):
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

### 2. 调用的时候

    cfg = json.dumps(cfg, cls=JSONEncoder)
如上这样既可

