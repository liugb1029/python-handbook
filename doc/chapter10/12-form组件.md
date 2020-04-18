# forms组件

## forms组件介绍

### forms组件功能

form组件的主要功能如下:

- 生成页面可用的HTML标签

- 对用户提交的数据进行校验

- 保留上次输入内容

  

### forms组件常用field

#### 1、field通用字段

| 属性                      | 值                                                           |
| ------------------------- | ------------------------------------------------------------ |
| required=True             | 是否允许为空                                                 |
| widget=None               | HTML插件                                                     |
| label=None                | 用于生成Label标签或显示内容                                  |
| initial=None              | 初始值                                                       |
| help_text=''              | 帮助信息(在标签旁边显示)                                     |
| error_messages=None       | 错误信息 {'required': '不能为空', 'invalid': '格式错误'}     |
| show_hidden_initial=False | 是否在当前插件后面再加一个隐藏的且具有默认值的插件（可用于检验两次输入是否一直） |
| validators=[]             | 自定义验证规则                                               |
| localize=False            | 是否支持本地化                                               |
| disabled=False            | 是否可以编辑                                                 |
| label_suffix=None         | Label内容后缀                                                |

#### 2、其他类型field字段

##### CharField

| 属性            | 值                   |
| --------------- | -------------------- |
| max_length=None | 最大长度             |
| min_length=None | 最小长度             |
| max_length=None | 是否移除用户输入空白 |

Error_message字段：required`, `max_length`, `min_length

##### IntergerField

| 属性      | 值     |
| --------- | ------ |
| max_value | 最大值 |
| min_value | 最小值 |

Error_message字段： required`, `invalid`, `max_value`, `min_value

##### DecimalField

| 属性      | 值     |
| --------- | ------ |
| max_value | 最大值 |
| min_value | 最小值 |

##### 时间格式化

```
BaseTemporalField(Field)
    input_formats=None          时间格式化   
 
DateField(BaseTemporalField)    格式：2015-09-01
TimeField(BaseTemporalField)    格式：11:12
DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
 
DurationField(Field)            时间间隔：%d %H:%M:%S.%f
```

##### RegexField

```
    regex,                      自定制正则表达式
    max_length=None,            最大长度
    min_length=None,            最小长度
    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
```

##### 文件Field

```python
FileField(Field)
    allow_empty_file=False     是否允许空文件
 
ImageField(FileField)      
    ...
    注：需要PIL模块，pip3 install Pillow
    以上两个字典使用时，需要注意两点：
        - form表单中 enctype="multipart/form-data"
        - view函数中 obj = MyForm(request.POST, request.FILES)
```

##### 其他field

```
ChoiceField(Field)
    ...
    choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
    required=True,             是否必填
    widget=None,               插件，默认select插件
    label=None,                Label内容
    initial=None,              初始值
    help_text='',              帮助提示
 
 
ModelChoiceField(ChoiceField)
    ...                        django.forms.models.ModelChoiceField
    queryset,                  # 查询数据库中的数据
    empty_label="---------",   # 默认空显示内容
    to_field_name=None,        # HTML中value的值对应的字段
    limit_choices_to=None      # ModelForm中对queryset二次筛选
     
ModelMultipleChoiceField(ModelChoiceField)
    ...                        django.forms.models.ModelMultipleChoiceField
 
 
     
TypedChoiceField(ChoiceField)
    coerce = lambda val: val   对选中的值进行一次转换
    empty_value= ''            空值的默认值
 
MultipleChoiceField(ChoiceField)
    ...
 
TypedMultipleChoiceField(MultipleChoiceField)
    coerce = lambda val: val   对选中的每一个值进行一次转换
    empty_value= ''            空值的默认值
 
ComboField(Field)
    fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
                               fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])
 
MultiValueField(Field)
    PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用
 
SplitDateTimeField(MultiValueField)
    input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
    input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']
 
FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中
    path,                      文件夹路径
    match=None,                正则匹配
    recursive=False,           递归下面的文件夹
    allow_files=True,          允许文件
    allow_folders=False,       允许文件夹
    required=True,
    widget=None,
    label=None,
    initial=None,
    help_text=''
 
GenericIPAddressField
    protocol='both',           both,ipv4,ipv6支持的IP格式
    unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用
 
SlugField(CharField)           数字，字母，下划线，减号（连字符）
    ...
 
UUIDField(CharField)           uuid类型
    ...
```

#### 3、Django内置插件

```
TextInput(Input)
NumberInput(TextInput)
EmailInput(TextInput)
URLInput(TextInput)
PasswordInput(TextInput)
HiddenInput(TextInput)
Textarea(Widget)
DateInput(DateTimeBaseInput)
DateTimeInput(DateTimeBaseInput)
TimeInput(DateTimeBaseInput)
CheckboxInput
Select
NullBooleanSelect
SelectMultiple
RadioSelect
CheckboxSelectMultiple
FileInput
ClearableFileInput
MultipleHiddenInput
SplitDateTimeWidget
SplitHiddenDateTimeWidget
SelectDateWidget
```

#### 4、常用选择插件

```
 # 单radio，值为字符串
 user = fields.CharField(
     initial=2,
     widget=widgets.RadioSelect(choices=((1,'上海'),(2,'北京'),))
 )

 # 单radio，值为字符串
 user = fields.ChoiceField(
     choices=((1, '上海'), (2, '北京'),),
     initial=2,
     widget=widgets.RadioSelect
 )

 #单select，值为字符串
 user = fields.CharField(
     initial=2,
     widget=widgets.Select(choices=((1,'上海'),(2,'北京'),))
 )

 #单select，值为字符串
 user = fields.ChoiceField(
     choices=((1, '上海'), (2, '北京'),),
     initial=2,
     widget=widgets.Select
 )

 #多选select，值为列表
 user = fields.MultipleChoiceField(
     choices=((1,'上海'),(2,'北京'),),
     initial=[1,],
     widget=widgets.SelectMultiple
 )


 #单checkbox
 user = fields.CharField(
     widget=widgets.CheckboxInput()
 )


 #多选checkbox,值为列表
 user = fields.MultipleChoiceField(
     initial=[2, ],
     choices=((1, '上海'), (2, '北京'),),
     widget=widgets.CheckboxSelectMultiple
 )
```

## 校验字段功能

针对一个实例：注册用户讲解。

模型：models.py

```
class UserInfo(models.Model):
    name=models.CharField(max_length=32)
    pwd=models.CharField(max_length=32)
    email=models.EmailField()
    tel=models.CharField(max_length=32)
```

模板: register.html:

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>

<form action="" method="post">
    {% csrf_token %}
    <div>
        <label for="user">用户名</label>
        <p><input type="text" name="name" id="name"></p>
    </div>
    <div>
        <label for="pwd">密码</label>
        <p><input type="password" name="pwd" id="pwd"></p>
    </div>
    <div>
        <label for="r_pwd">确认密码</label>
        <p><input type="password" name="r_pwd" id="r_pwd"></p>
    </div>
     <div>
        <label for="email">邮箱</label>
        <p><input type="text" name="email" id="email"></p>
    </div>
    <input type="submit">
</form>

</body>
</html>
```

视图函数：register



```python
# forms组件
from django.forms import widgets

wid_01=widgets.TextInput(attrs={"class":"form-control"})
wid_02=widgets.PasswordInput(attrs={"class":"form-control"})

class UserForm(forms.Form):
    name=forms.CharField(max_length=32,
                         widget=wid_01
                         )
    pwd=forms.CharField(max_length=32,widget=wid_02)
    r_pwd=forms.CharField(max_length=32,widget=wid_02)
    email=forms.EmailField(widget=wid_01)
    tel=forms.CharField(max_length=32,widget=wid_01)



def register(request):

    if request.method=="POST":
        form=UserForm(request.POST)
        1、is_valid会校验字典中的字段是否符合自定义的规则，有一个不符合，就返回false,
        2、待校验的字典中的key值必须包含自定义规则的字段，否则也返回false。
        3、待校验的字典中的key-value值可以比自定义规则的字段多。
        
        form.cleaned_data 只返回校验成功的字段的字典
        form.error  返回校验失败的字段的字典  errordict,每个key的value值是一个errorList
        form.errors.get('pwd')[0]  返回字段为什么校验不成功
        if form.is_valid():
            print(form.cleaned_data)       # 所有干净的字段以及对应的值
        else:
            print(form.cleaned_data)       #
            print(form.errors)             # ErrorDict : {"校验错误的字段":["错误信息",]}
            print(form.errors.get("name")) # ErrorList ["错误信息",]
        return HttpResponse("OK")
    form=UserForm()
    return render(request,"register.html",locals())
```



## 渲染标签功能 

### 渲染方式1

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
   <!-- 最新版本的 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
</head>
<body>
<h3>注册页面</h3>
<div class="container">
    <div class="row">
        <div class="col-md-6 col-lg-offset-3">

                <form action="" method="post">
                    {% csrf_token %}
                    <div>
                        <label for="">用户名</label>
                        {{ form.name }}
                    </div>
                    <div>
                        <label for="">密码</label>
                        {{ form.pwd }}
                    </div>
                    <div>
                        <label for="">确认密码</label>
                        {{ form.r_pwd }}
                    </div>
                    <div>
                        <label for=""> 邮箱</label>
                        {{ form.email }}
                    </div>

                    <input type="submit" class="btn btn-default pull-right">
                </form>
        </div>
    </div>
</div>



</body>
</html>
```



### 渲染方式2



```python
<form action="" method="post">
                    {% csrf_token %}
                    
                    {% for field in form %}
                        <div>
                            <label for="">{{ field.label }}</label>
                            {{ field }}
                        </div>
                    {% endfor %}
                    <input type="submit" class="btn btn-default pull-right">
                
</form>
```



### 渲染方式3

```python
<form action="" method="post">
    {% csrf_token %}
    
    {{ form.as_p }}
    <input type="submit" class="btn btn-default pull-right">

</form>
```

## 显示错误与重置输入信息功能

### 视图

```python
def register(request):

    if request.method=="POST":
        form=UserForm(request.POST)
        if form.is_valid():
            print(form.cleaned_data)       # 所有干净的字段以及对应的值
        else:
            print(form.cleaned_data)       # 校验正确的字段
            print(form.errors)             # ErrorDict : {"校验错误的字段":["错误信息",]}
            print(form.errors.get("name")) # ErrorList ["错误信息",]
        return render(request,"register.html",locals())
    form=UserForm()
    return render(request,"register.html",locals())
```

### 模板

**注意form表单添加novalidate属性**

```html
<form action="" method="post" novalidate>
    {% csrf_token %}
    
    {% for field in form %}
        <div>
            <label for="">{{ field.label }}</label>
            {{ field }} 
          <span class="pull-right" style="color: red">{{ field.errors.0 }}		 </span>
        </div>
    {% endfor %}
    <input type="submit" class="btn btn-default">

</form>
```





## 局部钩子与全局钩子

### 模板



```python
# forms组件
from django.forms import widgets

wid_01=widgets.TextInput(attrs={"class":"form-control"})
wid_02=widgets.PasswordInput(attrs={"class":"form-control"})

from django.core.exceptions import ValidationError
class UserForm(forms.Form):
    name=forms.CharField(max_length=32,
                         widget=wid_01
                         )
    pwd=forms.CharField(max_length=32,widget=wid_02)
    r_pwd=forms.CharField(max_length=32,widget=wid_02)
    email=forms.EmailField(widget=wid_01)
    tel=forms.CharField(max_length=32,widget=wid_01)


    # 局部钩子 clean_后面必须跟字段名称
    def clean_name(self):
        val=self.cleaned_data.get("name")
        if not val.isdigit():
            return val
        else:
            raise ValidationError("用户名不能是纯数字!")

    # 全局钩子

    def clean(self):
        pwd=self.cleaned_data.get("pwd")
        r_pwd=self.cleaned_data.get("r_pwd")

        if pwd==r_pwd:
            return self.cleaned_data
        else:
            raise ValidationError('两次密码不一致!')


def register(request):

    if request.method=="POST":
        form=UserForm(request.POST)
        if form.is_valid():
            print(form.cleaned_data)       # 所有干净的字段以及对应的值
        else:
            clean_error=form.errors.get("__all__")

        return render(request,"register.html",locals())
    form=UserForm()
    return render(request,"register.html",locals())
```



### 视图



```html
 <form action="" method="post" novalidate>
            {% csrf_token %}

            {% for field in form %}
                <div>
                    <label for="">{{ field.label }}</label>
                    {{ field }}
                    <span class="pull-right" style="color: red">
                          {% if field.label == 'r_pwd' %}
                          <span>{{ clean_error.0 }}</span>
                          {% endif %}
                          {{ field.errors.0 }}
                    </span>
                </div>
            {% endfor %}
            <input type="submit" class="btn btn-default">

</form>
```

