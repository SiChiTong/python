# python pymongo模块

## 1. 安装pymongo
    > pip install pymongo
默认安装最新版本的pymongo

## 2. 连接mongodb

    def get_db(db=db):
        conn = pymongo.Connection(host, port)
        conn.admin.authenticate(user, password)
        database = conn[db]
        return database

数据库的配置文件可以单独放在一个配置文件中，这样便于修改

## 3. 使用的时候直接获取数据库，然后操作指定的集合就可以
    >>> db = get_db()
    >>> db.collection.insert(doc)
    ...

## 4. 其他的一些操作基本上和mongodb的操作是一样的

## 5. 区别的地方：
### ①. pymongo sort()
sort的参数不是一个字典或者json格式的数据，而是一个包含tuple的list，如: [('aaa', -1), ('bbb', 1)]

