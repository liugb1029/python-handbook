####Django安装和部署

1. 下载Django：
   pip3 install django
2. 创建一个django project
   django-admin.py startproject mysite
   manage.py ----- Django项目里面的工具，通过它可以调用django shell和数据库等。
   settings.py ---- 包含了项目的默认设置，包括数据库信息，调试标志以及其他一些工作的变量。
   urls.py ----- 负责把URL模式映射到应用程序。
3. 在mysite目录下创建应用
   python manage.py startapp blog
4. 启动django项目
   默认端口是8000
   python manage.py runserver 8080


####静态文件配置
1. 在项目下settings.py
STATIC_URL = '/static/'  **静态文件路径别名**
增加如下代码
```python
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
    os.path.join(BASE_DIR, 'static/app01'),  # http://127.0.0.1:8000/static/timer.js 无需带app01
]
```

**这里的static就是在项目目录下的目录static，一般命名成这样。**
2. 调用css、js静态文件 路径开头使用别名

```html
<script type="text/javascript" src="/static/jquery-3.4.1.js"></script>
```

3.一般情况都是在static目录下针对各个应用创建各自的目录，比如app01，html文件中引入css和js文件。

```html
<script type="text/javascript" src="/static/jquery-3.4.1.js"></script>
<script type="text/javascript" src="/static/app01/timer.js"></script>
```
