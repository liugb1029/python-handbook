# 中间件

## 中间件的概念

中间件顾名思义，是介于request与response处理之间的一道处理过程，相对比较轻量级，并且在全局上改变django的输入与输出。因为改变的是全局，所以需要谨慎实用，用不好会影响到性能。

Django的中间件的定义：

```
Middleware is a framework of hooks into Django’s request/response processing. <br>It’s a light, low-level “plugin” system for globally altering Django’s input or output.

```

如果你想修改请求，例如被传送到view中的**HttpRequest**对象。 或者你想修改view返回的**HttpResponse**对象，这些都可以通过中间件来实现。

可能你还想在view执行之前做一些操作，这种情况就可以用 middleware来实现。

Django默认的`Middleware`：

```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

每一个中间件都有具体的功能。

## 自定义中间件

中间件中一共有四个方法：

```
process_request

process_view

process_exception

process_response
```

### process_request，process_response

当用户发起请求的时候会依次经过所有的的中间件，这个时候的请求时process_request,最后到达views的函数中，views函数处理后，在依次穿过中间件，这个时候是process_response,最后返回给请求者。

![img](/Users/liuguobing/Documents/Devlop/python-luffycity/第10章 Django/images/Middleware-1.png)

上述截图中的中间件都是django中的，我们也可以自己定义一个中间件，我们可以自己写一个类，但是必须继承MiddlewareMixin

需要导入

```python
from django.utils.deprecation  import MiddlewareMixin
```

![Middleware-1](/Users/liuguobing/Documents/Devlop/python-luffycity/第10章 Django/images/Middleware-1.png)

**in views:**

```python
def index(request):
    print("view index.........")
    return HttpResponse('index')
```

**in my-middlewares.py：**

```python
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse

class CustomerMiddleware(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware1 process_request.......')
        # 添加后直接不走后面得中间件request，直接到自己的response。
        # return HttpResponse('forbidden...')

    def process_response(self,request,response):
        print('Customer Middleware1 process_response.......')
        return response

class CustomerMiddleware2(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware2 process_request.......')

    def process_response(self,request,response):
        print('Customer Middleware2 process_response.......')
        return response
```

**结果：**

```
Customer Middleware1 process_request.......
Customer Middleware2 process_request.......
view index.........
Customer Middleware2 process_response.......
Customer Middleware1 process_response.......
```

**注意：**如果当请求到达请求1的时候直接不符合条件返回，即return HttpResponse("forbidden")，程序将把请求直接发给中间件1返回，然后依次返回到请求者，结果如下：

返回请求1的返回值，后台打印如下：(页面显示"forbidden")

```
Customer Middleware1 process_request.......
Customer Middleware1 process_response.......
```

**流程图如下：**

![img](/Users/liuguobing/Documents/Devlop/python-luffycity/第10章 Django/images/middleware-request返回值.png)

### process_view

```
process_view(self, request, callback, callback_args, callback_kwargs)
```

 **my-middlewares.py**修改如下

```python
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse

class CustomerMiddleware(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware1 process_request.......')
        # 添加后直接不走后面得中间件request，直接到自己的response。
        # return HttpResponse('forbidden...')

    def process_response(self,request,response):
        print('Customer Middleware1 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        # callback就是路径对应的视图函数
        print("Customer Middleware1 process_view.......")

class CustomerMiddleware2(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware2 process_request.......')

    def process_response(self,request,response):
        print('Customer Middleware2 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print("Customer Middleware2 process_view.......")
```

结果如下：

```
Customer Middleware1 process_request.......
Customer Middleware2 process_request.......
Customer Middleware1 process_view.......
Customer Middleware2 process_view.......
view index.........
Customer Middleware2 process_response.......
Customer Middleware1 process_response.......
```

下图进行分析上面的过程：

![img](/Users/liuguobing/Documents/Devlop/python-luffycity/第10章 Django/images/middleware-procee-view.png)

当最后一个中间的process_request到达路由关系映射之后，返回到中间件1的process_view，然后依次往下，到达其他的process_views函数，最后通过process_response依次返回到达用户。

process_view可以用来调用视图函数：

```python
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse

class CustomerMiddleware(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware1 process_request.......')
        # 添加后直接不走后面得中间件request，直接到自己的response。
        # return HttpResponse('forbidden...')

    def process_response(self,request,response):
        print('Customer Middleware1 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        # callback就是路径对应的视图函数
        print("Customer Middleware1 process_view.......")
        response = callback(request)
        # 会越过CustomerMiddleware2的process_view
        return response

class CustomerMiddleware2(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware2 process_request.......')

    def process_response(self,request,response):
        print('Customer Middleware2 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print("Customer Middleware2 process_view.......")
```

结果如下：

```
Customer Middleware1 process_request.......
Customer Middleware2 process_request.......
Customer Middleware1 process_view.......
view index.........
Customer Middleware2 process_response.......
Customer Middleware1 process_response.......
```

注意：process_view如果有返回值，会越过其他的process_view以及视图函数，但是所有的process_response都还会执行。

### process_exception

```
process_exception(self, request, exception)
```

只有当视图函数出错时才会触发这个执行。

示例修改如下：

```
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse
from django.middleware.common import CommonMiddleware

class CustomerMiddleware(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware1 process_request.......')
        # 添加后直接不走后面得中间件request，直接到自己的response。
        # return HttpResponse('forbidden...')

    def process_response(self,request,response):
        print('Customer Middleware1 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        # callback就是路径对应的视图函数
        print("Customer Middleware1 process_view.......")
        # response = callback(request)
        # return response

    def process_exception(self,request,exception):
        print('Customer Middleware1 process_exception.......')

class CustomerMiddleware2(MiddlewareMixin):

    def process_request(self,request):
        print('Customer Middleware2 process_request.......')

    def process_response(self,request,response):
        print('Customer Middleware2 process_response.......')
        return response

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print("Customer Middleware2 process_view.......")

    def process_exception(self,request,exception):
        print('Customer Middleware2 process_exception.......')
```

结果如下：

```
Customer Middleware1 process_request.......
Customer Middleware2 process_request.......
Customer Middleware1 process_view.......
Customer Middleware2 process_view.......
view index.........
Customer Middleware2 process_exception.......
Customer Middleware1 process_exception.......
Customer Middleware2 process_response.......
Customer Middleware1 process_response.......
```



流程图如下：

当视图函数出现错误时：

![img](/Users/liuguobing/Documents/Devlop/python-luffycity/第10章 Django/images/middlerware-process-exception.png)

 

 将md2的process_exception修改如下：

```
  def process_exception(self,request,exception):

        print("md2 process_exception...")
        # 页面直接错误信息
        return HttpResponse(exception)
```

就不会执行Middleware1的process_exception

结果如下：

```
Customer Middleware1 process_request.......
Customer Middleware2 process_request.......
Customer Middleware1 process_view.......
Customer Middleware2 process_view.......
view index.........
Customer Middleware2 process_exception.......
Customer Middleware2 process_response.......
Customer Middleware1 process_response.......
```