# python日期处理模块
python中关于日期的处理操作，包括日期格式化等

## datetime
关于datetime的操作

    t = datetime.datetime.now()
    print(t)
    print(type(t))
    print(t.year)
    print(t.month)
    print(t.day)
    # 转换成字符串
    t = t.strftime('%Y-%m-%d')
    print(t)
    print(type(t))
    # 将字符串格式的日期转换成datetime类型
    t = datetime.datetime.strptime(t, '%Y-%m-%d')
    print(t)
    # 将原有日期添加24小时
    max_date = t+datetime.timedelta(hours=24)
    print(max_date)
