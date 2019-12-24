---
title: test
---
# 定时任务API列表
在需要使用周期任务的app内，新建tasks.py，同时在settings中安装该app，则celery在启动时会自动发现tasks内已经注册的函数



## celery task

1. 获取所有已经注册的任务（celery启动时主动发现的任务）
    - 接口 http://127.0.0.1:8000/celery_manage/tasks/
    - 方式 get
    - 参数 无
    - 结果
        ```
        {
        'status': True,
         'data': [
                    'celery_manage.celery.debug_task',
                    'cmdb_business_manage.tasks.test',
                    'demo_app.tasks.test'
                ]
        }
        ```
2. 执行一个任务
    - 接口 'http://127.0.0.1:8000/celery_manage/async_apply_task/'
    - 方式 post
    - 参数 json 类型

        |  参数 |是否必需|类型|说明|
        |---|---|---|---|
        | task  |是|string|所添加的任务|
        | option  |否|dict|参数，eta,countdown，expires等如果值为日期需为string类型(%Y-%m-%d %H:%M:%S.%f)格式的日期字符串，如果不传该参数，则默认为立即执行|
        |kwargs|否|dict|传任务所需要的关键字参数|
        |args|否|tuple|传任务所需要的位置参数|

    - 结果
        ```
        {
            "status": true,
            "data": "af151086-d334-`44e6-82d6-657a84464859"
        }
        ```
    - 参数示例
        ```
            {
                'task': 'demo_app.tasks.test',
                'kwargs':{
                    'name':'cc',
                    'age':'22'
                }
                # 'options': {'countdown': 10} //倒计时
                # 'options': {'eta':str(datetime.now()+timedelta(seconds=15)) } // 具体时间

            }
        ```

3. 停止/删除一个任务
    - 接口 'http://127.0.0.1:8000/celery_manage/revoke_task/'
    - 方式 post
    - 参数 json 类型

        | 参数  |是否必需|类型|说明|
        |---|---|---|---|
        |  id |是|int|所要删除任务的id|
    - 参数示例
        ```
        {
            'id': 'c6f16f5e-28f7-48ec-9c2d-9232c52e2d4f'
        }
        ```

4. 修改一个还未执行的任务
    - 接口 'http://127.0.0.1:8000/celery_manage/update_task/'
    - 方式 post
    - 参数

        | 参数  |是否必需|类型|说明|
        |---|---|---|---|
        |  id |是|int|所要修改任务的id|
        |  task |是|str|修改之后的任务|
        |  kwargs |否|dict|关键字参数|
        |  args |否|tuple|位置参数|
        |  options |否|dict|执行所需参数，如果不传，默认为立即执行|
        - 参数示例
            ```{
                'id':'c0d83927-f314-40c5-b63c-2cf5921ab502',
                'task': 'demo_app.tasks.test',
                'kwargs': {
                    'name': 'cc',
                    'age': '2'
                },
                'options': {'countdown': 500}
                }
            ```
    - 结果
        -   ```{'status': True, 'msg': '修改成功', 'data': 'ada954ba-15b6-4011-8713-b13545579236'}```

        - ```{'status': False, 'msg': 'task error','data':''}```
5. 获取任务
    - 接口 http://127.0.0.1:8000/celery_manage/list_tasks/
    - 方式 get
    - 参数 无
    - 结果
        ```
            {
                "4a82a804-a556-404b-a05c-9e4e906bebf8": {
                    "uuid": "4a82a804-a556-404b-a05c-9e4e906bebf8",
                    "name": "demo_app.tasks.test",
                    "state": "SUCCESS",
                    "received": 1570778891.7902446,
                    "sent": null,
                    "started": 1570778906.7771018,
                    "succeeded": 1570778906.7811022,
                    "failed": null,
                    "retried": null,
                    "revoked": null,
                    "args": "[]",
                    "kwargs": "{}",
                    "eta": "2019-10-11T15:28:26.771243",
                    "expires": null,
                    "retries": 0,
                    "result": "None",
                    "exception": null,
                    "timestamp": 1570778906.7811022,
                    "runtime": 0.014999999999417923,
                    "traceback": null,
                    "exchange": null,
                    "routing_key": null,
                    "clock": 17809,
                    "client": null
                }
            }
        ```

## period task
1. 添加一个周期任务
    - 接口 http://127.0.0.1:8000/celery_manage/add_periodic_task/
    - 方式 post
    - 参数 json 类型

        |  参数 |是否必需|类型|说明|
        |---|---|---|---|
        | name  |是|string|所添加的任务的描述（唯一）|
        | task  |是|string|所添加的任务|
        | enabled  |否|bool|是否启用 默认为True|
        | crontab  |否|dict|interval和crontab只能有一个存在|
        | interval  |否|dict|interval和crontab只能有一个存在|
        | kwargs  |否|string|关键字参数json格式的默认为'{}'|
        | args  |否|string|位置参数默认为'()'|
        | expires  |否|string|过期时间 (%Y-%m-%d %H:%M:%S.%f)格式的日期字符串|
    - 结果

        ```
        {
            'status': True,
            'data': {
                'id': 5,
                'task': 'demo_app.tasks.test'

            }
         }
        ```
    - 参数示例
        ```
        data = {
            "name": "cmdb_test_task2",
            "task": "demo_app.tasks.test",
            "enabled": True,

            "crontab": {
                "minute": "*/1"
            },
            "kwargs":'{"mm":1}',
        }
       ```
        crontab内可指定minute,hour,day_of_week,day_of_month,month_of_year，默认为'*'

2. 删除一个周期任务
    - 接口 'http://127.0.0.1:8000/celery_manage/delete_periodic_task/'
    - 方式 post
    - 参数 json 类型

        | 参数  |是否必需|类型|说明|
        |---|---|---|---|
        |  id |是|int|所要删除任务的id|

    - 结果
        - ```{'status': False, 'msg': '未查询到该任务'}```
        - ```{'status': True, 'msg': 'delete ok'}```
        - ```{'status': False, 'msg': 'params error'}```

3. 修改一个周期任务
    - 接口 'http://127.0.0.1:8000/celery_manage/update_periodic_task/'
    - 方式 post
    - 参数 json 类型

        | 参数  |是否必需|类型|说明|
        |---|---|---|---|
        |  id |是|int|所要修改任务的id|
        其他传需要修改的参数
        例如：
        ```
        {
                'id':5,
                "name": "tes212d3",
                "task": "demo_app.tasks.test",
                "enabled":True,
                "crontab": "*/15 * * * * (m/h/d/dM/MY)"
        }
        ```
    - 结果
        - {'status': True, 'msg': '更新成功'}
        - {'status': False, 'msg': '未找到'}

4. 获取周期任务详细信息
    - 接口 'http://127.0.0.1:8000/celery_manage/get_periodic_task/'
    - 方式 get
    - 参数 无
    - 结果
        ```
        {
        "status": true,
        "msg": "查询成功",
        "data": [
            {
                "id": 1,
                "name": "test_1",
                "task": "demo_app.tasks.test",
                "interval_id": 1,
                "crontab_id": null,
                "args": "[]",
                "kwargs": "{}",
                "queue": null,
                "exchange": null,
                "routing_key": null,
                "expires": null,
                "enabled": true,
                "last_run_at": "2019-10-10T17:35:41.285",
                "total_run_count": 360,
                "date_changed": "2019-10-10T17:35:42.285",
                "description": "",
                "interval": "every second",
                "crontab": null
            },
            {
                "id": 2,
                "name": "celery.backend_cleanup",
                "task": "celery.backend_cleanup",
                "interval_id": null,
                "crontab_id": 2,
                "args": "[]",
                "kwargs": "{}",
                "queue": null,
                "exchange": null,
                "routing_key": null,
                "expires": null,
                "enabled": true,
                "last_run_at": null,
                "total_run_count": 0,
                "date_changed": "2019-10-10T17:32:41.782",
                "description": "",
                "interval": null,
                "crontab": "0 4 * * * (m/h/d/dM/MY)"
            }]}
        ```

5. 获取周期任务所有任务名字
    - 接口 'http://127.0.0.1:8000/celery_manage/get_task_name_list/'
    - 方式 get
    - 结果
        ```
            {
            "status": true,
            "msg": "",
            "data": [
                "test_1",
                "celery.backend_cleanup",
                "cmdb_test_task1",
                "tes212d3"
                ]
            }
        ```

