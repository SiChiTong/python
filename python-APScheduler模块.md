# python APScheduler模块

APScheduler是一个定时任务的框架，使用起来很方便，可以很灵活的定制任意时间需要触发的任务

参考网址：
>* https://pypi.python.org/pypi/APScheduler/
>* http://apscheduler.readthedocs.org/en/3.0/
>* 这个链接中有源码和示例：
>* https://bitbucket.org/agronholm/apscheduler/src/c534d51a57638e8a8a51c36d4a4128b89f8beb22?at=master

## 1. 安装
    >pip install APScheduler

## 2. 简单使用示例

### 1. background：

    from datetime import datetime
    import time
    import os

    from apscheduler.schedulers.background import BackgroundScheduler


    def tick():
        print('Tick! The time is: %s' % datetime.now())


    if __name__ == '__main__':
        scheduler = BackgroundScheduler()
        scheduler.add_job(tick, 'interval', seconds=3)
        scheduler.start()
        print('Press Ctrl+{0} to exit'.format('Break' if os.name == 'nt' else 'C'))

        try:
            while True:
                time.sleep(2)
        except (KeyboardInterrupt, SystemExit):
            scheduler.shutdown()


由于是background的，所以需要在后面使用while循环保持主进程不被结束，不然就会直接结束掉，无法完成定时任务

### 2. blocking：

    from datetime import datetime
    import os

    from apscheduler.schedulers.blocking import BlockingScheduler


    def tick():
        print('Tick! The time is: %s' % datetime.now())


    if __name__ == '__main__':
        scheduler = BlockingScheduler()
        scheduler.add_job(tick, 'interval', seconds=3)
        print('Press Ctrl+{0} to exit'.format('Break' if os.name == 'nt' else 'C'))

        try:
            scheduler.start()
        except (KeyboardInterrupt, SystemExit):
            pass

blocking类型的是阻塞的，所以可以一直触发任务


## 3. 添加任务
使用APScheduler主要就是为了设置定时的任务
### 1. add_job:
这个方法用的比较多，也比较通用

    def job_function():
        print("Hello World")

    sched = BlockingScheduler()
    sched.add_job(job_function, 'interval', hours=2)
    sched.start()

这样设置的就是每两个小时执行一次 job_function

### 2. triggers:
    'interval': 每隔一定的时间执行一次
    'date': 可以指定在未来某个时间执行
    'cron': 这个触发机制很灵活，可以是指定的时间，基本可以代替'date'
            具体设置可以参考：
            https://apscheduler.readthedocs.org/en/latest/modules/triggers/cron.html#module-apscheduler.triggers.cron

### 3. 可以使用注解的形式添加任务
    @sched.scheduled_job('cron', id='my_job_id', day='last sun')
    def some_decorated_task():
        print("I am printed at 00:00:00 on the last Sunday of every month!")

与下面的效果相同：

    sched.add_job(job_function, 'cron', day_of_week='mon-fri', hour=5, minute=30, end_date='2014-05-30')
    
## 4. 注意：
在APScheduler启动后，就不可以再手动的添加任务了，也就是在start以后的任务都是无效的，不过，可以通过在任务里面添加任务，利用启动的任务去添加新的任务

    def push_message(msg, user_list):
        """
        消息推送的方法
        :param msg: 推送的内容
        :param user_list: 推送对象的列表，这里是用户的id列表
        :return:
        """
        print('推送消息任务...', user_list)
        send(msg, user_list)


    def exe_job():
        """
        执行推送任务的添加的方法，
        在此方法中动态添加任务
        此方法每隔一定时间会定期执行一次
        从redis缓存中获取需要推送的消息集合
        然后遍历消息集合，添加任务，
        并且从redis消息集合中删除已经添加到任务的消息
        :return:
        """
        # 获取推行消息的有序集合，默认是升序排列
        push_list = get_zset_value(PUSH_QUEUE_KEY)
        if push_list:
            for p in push_list:
                result = eval(p.decode())
                run_date = result['push_date']
                user_list = result['user_list']
                msg = result['push_msg']
                # 添加推送任务
                scheduler.add_job(push_message, 'date', run_date=run_date, args=[msg, user_list])
                # 删除集合中的消息
                remove_zset_value(PUSH_QUEUE_KEY, p)


    if __name__ == '__main__':
        scheduler.add_job(exe_job, 'interval', seconds=0.5)
        print('scheduler is running...')
        scheduler.start()

上面这个例子是利用在APScheduler框架执行定时推送消息的任务，在这个里面，首先添加一个执行的exe_job,APScheduler每隔0.5秒执行一次，
每次执行后都会从redis的缓存中读取待推送的消息通知，然后把这些消息通知添加到任务中，并设定好执行时间，这样到了时间就会自动执行推送的任务，目的达到了


这些都是基本的用法，更多用法可以参考官方的文档API

