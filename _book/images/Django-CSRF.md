#### Django中CSRF原理及应用详解


 Web开发中十分重要的一项内容就是Web安全，在我们进行服务端构建的时候，最常见的几种Web攻击无非是下面的这几种：

```html
1.注入（SQL注入）
2.跨站脚本攻击（XSS）
3.跨站请求伪造（CSRF）
4.开放重定向
```

今天主要就CSRF做一个简单的总结，结合Python Web框架Django 做一个应用

CSRF跨站点请求伪造(Cross—Site Request Forgery)

##### **一、CSRF是什么**

`跨站请求伪造（CSRF）`与跨站请求脚本正好相反。跨站请求脚本的问题在于，客户端信任服务器端发送的数据。跨站请求伪造的问题在于，服务器信任来自客户端的数据。

##### 二、无CSRF时存在的隐患

跨站请求伪造是指攻击者通过HTTP请求将数据传送到服务器，从而盗取回话的cookie。盗取回话cookie之后，攻击者不仅可以获取用户的信息，还可以修改该cookie关联的账户信息。

##### 三、Form提交（CSRF）

那么在Django中CSRF验证大体是一个什么样的原理呢？下面通过一个小例子来简单说明一下：

我们把Django中CSRF中间件开启（在settings.py中）

```python
'django.middleware.csrf.CsrfViewMiddleware'
```

在app01的views.py中写一个简单的后台

```python
from django.shortcuts import render,redirect,HttpResponse

# Create your views here.

def login(request):
    if request.method == "GET":
        return render(request,'login.html')
    elif request.method == "POST":
        user = request.POST.get('user')
        pwd = request.POST.get('pwd')
        if user == 'root' and pwd == "123123":
            #生成随机字符串
            #写到用户浏览器cookie
            #保存在服务端session中
            #在随机字符串对应的字典中设置相关内容
            request.session['username'] = user
            request.session['islogin'] = True
            if request.POST.get('rmb',None) == '1':
                #认为设置超时时间
                request.session.set_expiry(10)
            return redirect('/index/')
        else:
            return render(request,'login.html')

def index(request):
    #获取当前随机字符串
    #根据随机字符串获取对应的信息
    if request.session.get('islogin', None):
        return render(request,'index.html',{'username':request.session['username']})
    else:
        return HttpResponse('please login ')

def logout(request):
    del request.session['username']
    request.session.clear()
    return redirect('/login/')
```

在templates中写一个简单的登陆,创建两个文件（login.html,index.html）登陆成功跳转index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="post">
        <input type="text" name="user" />
        <input type="text" name="pwd" />
        <input type="checkbox" name="rmb" value="1" /> 10s免登录
        <input type="submit" value="提交" />
    </form>
</body>
</html>
```

这是浏览器的样式：

![](images/Django-CSRF-1.png)

那么如果这个时候，我们点击登陆提交，django会因为无法通过csrf验证返回一个403：

![](images/Django-CSRF-2.png)

而csrf验证其实是对http请求中一段随机字符串的验证，那么这段随机字符串从何而来呢？这个时候我们尝试把login.html做一个修改添加一句 {% csrf_token %}：    

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="post">
        {% csrf_token %}
        <input type="text" name="user" />
        <input type="text" name="pwd" />
        <input type="checkbox" name="rmb" value="1" /> 10s免登录
        <input type="submit" value="提交" />
    </form>
</body>
</html>
```

这个时候我们再通过浏览器元素审查，就会发现一段新的代码：

![](images/Django-CSRF-3.png)

Django在html中创建一个基于input框value值的随机字符串 ，这个时候我们再次输入后台设置的账号密码，进行提交，会发现提交成功，进入了我们的index.html界面当中：

![](images/Django-CSRF-4.png)

这就是csrf的基本原理，如果没有这样一段随机字符串做验证，我们只要在另一个站点，写一个表单，提交到这个地址下，是一样可以发送数据的，这样就造成了极大的安全隐患。而我们新添加的csrf_token就是在我们自己的站点中，设置的隐藏参数，用来进行csrf验证。

##### 四、Ajax提交 （CSRF）

​        上面是一个基于form表单提交的小例子，csrf中间件要验证的随机字符串存放在了form表单当中，发送到了后台，那么如果我们使用的是**ajax异步请求**，是不是也要传送这样一个类似的字符串呢？答案是肯定的，但这个字符串又该来自哪里？其实它来自cookie，在上面的那个小例子当中，我们打开F12元素审查，会发现，在cookie中也存放这样一个csrftoken:

![](images/Django-CSRF-6.png)

所以下面呢我们就再聊一下ajax请求中如何进行。

首先我们引入两个js文件放在工程项目的static当中，这两个文件是jquery的库文件，方便我们进行请求操作：

![](images/Django-CSRF-7.png)

 然后我们把之前的login.html做一个修改：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="post">
        {% csrf_token %}
        <input type="text" name="user" />
        <input type="text" name="pwd" />
        <input type="checkbox" name="rmb" value="1" /> 10s免登录
        <input type="submit" value="提交" />
        <input id="btn" type="button" value="按钮">
    </form>
 
    <script src="/static/jquery-1.12.4.js"></script>
    <script src="/static/jquery.cookie.js"></script>
    <script>
        $(function () {
            $('#btn').click(function () {
                $.ajax({
                    url:'/login/',
                    type:"POST",
                    data:{'username':'root','pwd':'123123'},
                    success:function (arg) {
                        
                    }
                })
            })
        })
    </script>
</body>
</html>
```

这个时候我们打开界面，点击按钮，会发现http请求发送403错误，很明显我们直接发送请求是不合适的，并没有带随机字符串过去：

![](images/Django-CSRF-8.png)

所以，我们应该先从cookie中获取到这个随机字符串，这个随机字符串的名字，我们可以通过之前的验证得出是“csrftoken”：

```html
var csrftoken = $.cookie('csrftoken');
```

这样变量csrftoken取到的就是我们的随机字符串；但是如果后台想要去接收这个随机字符串，也应该需要一个key，那这个key是什么？我们可以通过查找配置文件，通过控制台输出的方式验证这个key：

```python
from django.conf import settings
print(settings.CSRF_HEADER_NAME)
```

​        最后输出的是：

```html
HTTP_X_CSRFTOKEN
```

​         但这里需要注意的是，HTTP_X_CSRFTOKEN并不是请求头中发送给django真正拿到的字段名，前端发过去真正的字段名是：

```html
X-CSRFtoken
```

​        那为什么从Django的控制台输出会得到HTTP_X_CSRFTOKEN呢？其实我们前端的请求头X-CSRFtoken发送到后台之后，django会做一个名字处理，在原来的字段名前家一个HTTP_,并且将原来的小写字符变成大写的，“-”会处理成下划线“_”，所以会有这两个字段的不一样。但本质上他们指向的都是同一个字符串。知道这一点之后，那么我们前端就可以真正地发起含有CSRF请求头的数据请求了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="post">
        {% csrf_token %}
        <input type="text" name="user" />
        <input type="text" name="pwd" />
        <input type="checkbox" name="rmb" value="1" /> 10s免登录
        <input type="submit" value="提交" />
        <input id="btn" type="button" value="按钮">
    </form>


    <script src="/static/jquery-1.12.4.js"></script>
    <script src="/static/jquery.cookie.js"></script>
    <script>
        var csrftoken = $.cookie('csrftoken');
        $(function () {
            $('#btn').click(function () {
                $.ajax({
                    url:'/login/',
                    type:"POST",
                    data:{'username':'root','pwd':'123123'},
                    header:{'X-CSRFtoken':csrftoken},
                    success:function (arg) {
                        
                    }
                })
            })
        })
    </script>

</body>
</html>
```

​       在页面中点击按钮之后，会发现请求成功！

那么这个时候有人会问，难道所有的ajax请求，都需要这样获取一次写进去吗，会不会很麻烦。针对这一点，jquery的ajax请求中为我们封装了一个方法：ajaxSetup，它可以为我们所有的ajax请求做一个集体配置，所以我们可以进行如下改造，这样不管你的ajax请求有多少，都可以很方便地进行csrf验证了：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login/" method="POST">
        {% csrf_token %}
        <input type="text" name="user" />
        <input type="text" name="pwd" />
        <input type="checkbox" name="rmb" value="1" /> 10s免登录
        <input type="submit" value="提交" />
        <input id="btn" type="button" value="按钮">
    </form>


    <script src="/static/jquery-1.12.4.js"></script>
    <script src="/static/jquery.cookie.js"></script>
    <script>
        var csrftoken = $.cookie('csrftoken');
        
        function csrfSafeMethod(method) {
            // these HTTP methods do not require CSRF protection
            return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
        }
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
                    xhr.setRequestHeader("X-CSRFToken", csrftoken);
                }
            }
        });
     
        $(function () {
            $('#btn').click(function () {
                $.ajax({
                    url:'/login/',
                    type:"POST",
                    data:{'username':'root','pwd':'123123'},
                    success:function (arg) {
                        
                    }
                })
            })
        });
    </script>

</body>
</html>
```

##### 五、装饰器配置

​         在讲完基本的csrf验证操作之后，我们还有一个可说的地方。在平时的产品需求当中，并不一定所有的接口验证都需要进行csrf验证，而我们之前采用的是在settings.py中间件配置进行全局配置，那么如果遇到了不需要开启csrf的时候该怎么办呢？

```python
from django.views.decorators.csrf import csrf_exempt,csrf_protect

@csrf_protect
def index(request):
    .....

@csrf_exempt
def login(request):
    .....
```

​       @csrf_protect 是 开启csrf验证的装饰器，@csrf_exempt是关闭csrf验证的装饰器。

##### 六、前后端分离项目手动将csrftoken写入cookie的方法

1. 手动设置，在view 中添加（经常失效，建议采用2,3,4种方法，亲测有效）

```
request.META["CSRF_COOKIE_USED"] = True
```

2. 手动调用 csrf 中的 get_token(request) 或 rotate_token(request) 方法。

   ```python
   from django.middleware.csrf import get_token ,rotate_token
   
   def server(request):
   
       # get_token(request)       // 两者选一
       # rotate_token(request)   // 此方法每次设置新的cookies
        
       return render(request, ‘server.html‘)
   ```

3. 在HTML模板中添加 {% csrf_token %}

4. 在需要设置cookie的视图上加装饰器 ensure_csrf_cookie（）

```python
from django.views.decorators.csrf import ensure_csrf_cookie

@ensure_csrf_cookie
def server(request):

    return render(request,'server.html')
```