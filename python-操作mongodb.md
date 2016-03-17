# python操作mongodb


## 1. mongodb日期格式数据
在mongodb中有一种日期格式是ISODate,但是在python中没有这个，所以在查询的时候可能没有办法一直的对应起来，所以需要一定的转换来操作
在mongodb shell中查询日期可以这样来：

    >db.collection.find({'date':{'$gt':ISODate('2000-12-12 00:00:00')}).sort([('date', 1)])

这应该是一个UTC的时间，但是，在python代码中这样做是有错的，首先ISODate类型就没有，如果直接用里面的日期字符串，却查不出结果，
此时，可以使用python的datetime来进行转换：

    d_str = "2000-12-12 00:00:00"
    d= datetime.datetime.strptime(t_str,'%Y-%m-%d %H:%M:%S')
此时再去查找就没有问题了：

    >db.collection.find({'date': {'$gt': d}).sort([('date', 1)])
