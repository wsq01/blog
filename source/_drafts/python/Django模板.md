

Djano 的模板系统将 Python 代码与 HTML 代码解耦，动态地生成 HTML 页面。Django 项目可以配置一个或多个模板引擎，但是通常使用 Django 的模板系统时，应该首先考虑其内置的后端 DTL（`Django Template Language`），Django 模板语言。
# 什么是模板
在 Django 中，模板是可以根据字典数据动态变化的，并且能够根据视图中传递的字典数据动态生成相应的 HTML 网页。Django 中使用 Template 来表示模板，Template 对象定义在`django/template/base.py`文件中，它的构造函数如下所示：
```python
def __init__(self,template_string,origin=None,name=None,engine=None)
```
它只有一个必填的参数：字符串表示的模板代码。
## 模板的配置
首先按照`BookStore/templates`路径创建模板文件夹`templates`，在`settings.py`配置文件中有一个`TEMPLATES`变量：
```python
TEMPLATES = [
  {
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [],  #指定模板文件的存放路径
    'APP_DIRS': True, #搜索APP里面的所有templates目录
    'OPTIONS': {
      'context_processors': [  #context_processors 用于配置模板上下文处理器
        'django.template.context_processors.debug',
        'django.template.context_processors.request',
        'django.contrib.auth.context_processors.auth',
        'django.contrib.messages.context_processors.messages',
      ],
    },
  },
]
```
其中每一项释义如下所示：
`BACKEND`: Django 默认设置，指定了要是用的模板引擎的 Python 路径；
`DIRS`: 一个目录列表，指定模板文件的存放路径，可以是一个或者多个。模板引擎将按照列表中定义的顺序查找模板文件；
`APP_DIRS`: 一个布尔值，默认为`Ture`。表示会在安装应用中的`templates`目录中搜索所有模板文件；
`OPTIONS`: 指定额外的选项，不同的模板引擎有着不同的可选参数。

## 修改settings配置文件
修改`settings.py`文件，设置`TEMPLATES`的`DIRS`值来指定模板的搜索目录为`templates`：
```python
'DIRS': [os.path.join(BASE_DIR, 'templates')]
```
# 模板的加载与响应方式
方式一：通过`loader`获取模板,通过`HttpResponse`进行响应
```python
from django.template import loader
# 1.通过loader加载模板
t = loader.get_template("模板文件名")
# 2.将t转换成HTML字符串
html = t.render(字典数据)
# 3.用响应对象将转换的字符串内容返回给浏览器
return HttpResponse(html)
```
方式二：使用`render`方法直接加载并响应模板
```python
from django.shortcuts import render
return render(request,'模板文件名', 字典数据)
```
下面我们对上述两种方式分别来说明：
```python
#方式一
from django.template import loader # 导入loader方法
from django.shortcuts import render #导入render 方法
def test_html(request): 
  t=loader.get_template('test.html') 
  html=t.render({'name':'张三'}) #以字典形式传递数据并生成html
  return HttpResponse(html) #以 HttpResponse方式响应html
#方式二
from django.shortcuts import render #导入reder方法 
def test_html(request): 
  return render(request,'test.html', {'name': '张三'}) #根据字典数据生成动态模板
```
然后在`templates`目录下创建 test.html 文件并在其中添加如下代码：
```html
<p style="font-size:50px;color:red">{{name}}</p>
```
最后在`BookStore/urls.py`文件的`urlpatterns`列表中为视图函数`test_html()`配置路由映射关系,如下所示：
```python
urlpatterns = [ path('admin/', admin.site.urls), path('test/', views.test_html), ]
```
从上述过程我们不难体会 Django 视图函数的实现流程。首先定义了视图函数`test_html()`，然后在`templates`文件夹中新建了`test.html`文件，使用它作为模板文件；最后我们配置了视图函数的路由映射关系，以上步骤完成后，我们可以通过访问`127.0.0.1/test`得到展示页面。

`renbder`方法的作用是结合一个给定的模板和一个给定的字典，并返回一个渲染后的`HttpResponse`对象。通俗的讲就是把字典格式的内容, 加载进`templates`目录中定义的 HTML 文件, 最终通过浏览器渲染呈现.

`rebder()`方法的完整参数格式如下所示：
```
render(request, template_name, context=None, content_type=None, status=None, using=None)
```
以下每个参数的含义如下所示：
* `request`: 是一个固定参数，用于生成响应的请求对象；
* `template_name`: `templates`中定义的文件, 要注意路径名. 比如`'templates\appname\index.html'`, 参数就要写`appname\index.html`；
* `context`: 要传入文件中用于渲染呈现的数据, 默认是字典格式；
* `content_type`: 生成的文档要使用的媒体格式类型。默认为`DEFAULT_CONTENT_TYPE`设置的值；
* `status`: http 的响应代码,默认是 200；
* `using`: 用于加载模板使用的模板引擎的名称。

常见的`content_type`媒体格式：
```
text/html ： HTML 格式
text/plain ：纯文本格式
text/xml ： XML 格式

image/gif ：gif 图片格式
image/jpeg ：jpg 图片格式
image/png：png 图片格式

application/xhtml+xml ：XHTML 格式
application/xml： XML 数据格式
application/atom+xml ：Atom XML 聚合格式
application/json： JSON 数据格式
application/pdf：pdf 格式
application/msword ：Word 文档格式
application/octet-stream ： 二进制流数据（如常见的文件下载）
application/x-www-form-urlencoded ：form 表单数据被编码为 key/value 格式发送到服务器（表单默认的提交数据的格式）。

multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式
```
# 模板变量
我们知道，模板是文本文件，比如我们常见的有 HTML、CSV、TXT 等。Django 模板语言的语法主要分为以下四个部分：变量、标签、过滤器、注释。

Django 模板引擎通过`context_processors`这个上下文处理器来完成字典提供的值（`value`）与模板变量之间的替换，也就是用字典的`vaule`来替换模板文件`test.html`中的变量`{{name}}`，这就好比字典中`key`到`vaule`的映射。而我们无需关心内部细节是如何让实现的，这些由 Django 框架自己完成。

提示：当在模板中遇到变量的时候，会根据视图函数来确定这个变量的值，然后将结果输出。
## 变量的命名规范
Django 对于模板变量的命名规范没有太多的要求，可以使用任何字母、数字和下划线的组合来命名，且必须以字母或下划线开头，但是变量名称中不能有空格或者标点符号。
## 模板的变量语法
如何理解模板的变量语法呢？其实它有四种不同的使用场景：
* 索引`index`查询，如`{{变量名.index}}`，其中`index`为`int`类型即索引下标；
* 字典查询方法，`{{变量名.key}}`其中`key`代表字典的键，如`a['b']`；
* 属性或方法查询，如`{{对象.方法}}`，把圆点前的内容理解成一个对象，把圆点后的内容理解为对象里面的属性或者方法；
* 函数调用，如`{{函数名}}`。

注意：在模板中访问对象方法的时候，方法调用不需要加括号，而且只能够调用不带参数的方法；如果不希望自定义的方法被模板调用可以使用`alters_data=Ture`属性，放在方法的结束位置即可。

```python
def test_html(request):
  a={} #创建空字典，模板必须以字典的形式进行传参
  a['name'] = '张三' 
  a['course'] = ["Python", "C", "C++", "Java"]
  a['b'] = {'name':'张三', 'address':'北京'}
  a['test_hello'] = test_hello
  a['class_obj'] = Website()
  return render(request,'test_html.html', a)
def test_hello():
  return 'HelloWorld'
class Website:
  def Web_name(self):
    return 'Hello，World'
  #Web_name.alters_data=True #不让Website()方法被模板调用
```
其次在`templates`目录下创建名为`test_html`的`html`文件，然后添加以下代码：
```html
<p> 网站名字是{{ name }}</p> //字典查询
<p> 课程包含{{ course.1 }}</p> //索引查询
<p> 变量a是{{ b }}  <p>
<p> a['address']是{{b.address}}  </p>//字典查询
<p> 函数fuction：{{ test_hello }}</p> //函数方法调用
<p> 类实例化对象：{{class_obj.Web_name}} </p> //类方法调用
```
然后在`urls.py`文件中添加路由配置：
```
path('test_html/', views.test_html)
```
## 模板传参语法格式
在视图函数中必须将变量封装到字典中才允许传递到模板上：
```python
#方式1
def xxx_view(request)
  dic = {
    "变量1":"值1",
    "变量2":"值2",
  }
  return render(request, 'xxx.html', dic)
#方式2
def xxx_view(request)
  变量1=值1
  变量2=值2
  return render(request, 'xxx.html', locals())
```
提示：`locals()`返回当前函数作用域内全部局部变量形成的字典。即将变量与值对应形成字典，并把这个字典作为`locals()`的返回值来使用。
# 模板标签
## if标签
```python
{% if 条件表达式1 %}
 ......
{% elif 条件表达式2 %}
......
{% elif 条件表达式3 %}
......
{% else %}
......
{% endif %}
```
在`if`标签中可以使用算术操作符，如`>、<、==、<=`等符号，同时也可以使用逻辑运算符`and、or`来连接多个条件，以及使用`not`对当前条件取反。

提示：`elif`和`else`这两个标签是可选的，`elif`标签可以不止一个，但是`else`标签只有一个，同时也可以都不出现在`if`标签中，只使用`if`与`endif`。
## for标签
`for`标签用于对可迭代对象进行遍历，包括列表、元组等。
```python
{% for 变量 in 可迭代对象 %}
    ... 循环语句
{% empty %}
    ... 可迭代对象无数据时填充的语句
{% endfor %}
```
在`for`标签还提供了内置变量`forloop`，我们可以访问这个变量的属性从而获取`for`循环迭代过程中的一些信息：

| 变量	              | 描述 |
| :--: | :--: |
| forloop.counter	    | 用来计数，查看当前迭代第几个元素（从1开始索引） |
| forloop.counter0	  | 用来计数，查看当前迭代第几个元素（从0开始索引） |
| forloop.revcounter	| 表示当前循环中剩余的未被迭代的元素数量（从1开始索引） |
| forloop.revcounter0	| 表示当前循环中剩余的未被迭代的元素数量（从0开始索引） |
| forloop.first	      | 如果当前迭代的是第一个元素，则为True |
| forloop.last 	      | 如果当前迭代的是最后一个元素，则为True |
| forloop.parentloop	| 在嵌套循环中，用来引用外层循环的 forloop |

## url标签
`url`标签可以避免在模板中使用硬编码的方式插入要访问的`url`地址。

所谓硬编码就是将数据直接嵌入到程序或其他可执行对象的源代码中，比如我们修改了视图的访问地址，如果模板中采用的是硬编码的话，那么也需要对模板中的访问地址`url`进行修改，让它们保持数据的一致，但是这样对于采用 MTV 设计模式的 Django 框架来说是极其不方便的。`url`标签就很好的避免了这一点，它的使用语法格式如下：
```
{% url 'app_name:url_name' args1 args2 %}
```
其中`app_name`代表我们创建的应用的名字此处是`index`；`url_name`是`url`自定义的别名，可以在配置路由地址时通过`path`的`name`属进行设置。而后面的`args1、args2`参数是用于定义动态的`url`即带有查询的字符串的`url`。

首先我们在`urls.py`文件中 为`url`设置别名：
```
path('Hello_MyWeb/', views.Hello_MyWeb, name='hello')
```
然后我们在`templates`目录下创建一个名为`test_url`的`html`文件：
```html
<p><a href="{% url 'hello' %}" >点我看C语言中文网</a></p>
```
最后我们在`views.py`文件中创建一个`test_url`函数：
```python
def test_url(request):
  return render(request, 'test.url')
```
在浏览器地址栏访问`127.0.0.1:8000/test_url/`，通过点击可以跳转到`Hello_MyWeb`页面。如果你想跳转到其他的页面，只需要将给相应路由配置`name`属性即可，而我们无需做其他的改动。`name`参数有非常重要作用，`url`的反向解析也是通过它与`reverse()`函数配合使用实现的。

提示：`url`标签主要用来实现页面之间的跳转，在分布式路由中使用`{% url 'app_name:url_name' args1 args2 %}`即应用名`:url`别名，在本例中是`index:hello`。
### 给定参数的动态url
如果是带参数的`url`我们要怎么处理它呢？首更改`path`路由函数映射关系：
```python
path('Hello_MyWeb/<int:id>', views.Hello_MyWeb, name='hello')
```
改动`Hello_MyWeb`视图函数，为其添加`id`参数：
```python
def Hello_MyWeb(request, id):
```
再把模板中的标签改写成如下格式：
```html
<p><a href="{% url 'hello' 1 %}" >跳转链接</a></p>
```
# 自定义标签
## 如何实现自定义标签
自定义标签可以分为三种类型：简单标签（`simple_tag`）、引用标签（`inclusion_tag`）、赋值标签（`assignment_tag`）。

Django 为我们提供了自定义的机制，我们可以通过使用 Python 代码来自定义标签来，最后使用`{% load %}`标签进行加载。但是在自定义标签之前，需要我们做一些准备工作，如下所示：
* 创建专门的应用来装载自定义标签或者在项目原始`app`上进行自定义，在这里我们依旧使用原有的`index`应用；
* 在`index`应用下创建名为`templatetags`（名字不能变） 的 Python 包；
* 在新建的 Python 包中新建一个名为`index_tags.py`文件，该文件命名时避免与内置标签与过滤器名字冲突；
* 在`INSTALLED_APPS`列表中注册`app`，因为`index`应用之前已经注册，所以就无须操作了，若是新建的`app`就需要注册。

给`index_tags.py`文件命名时，需要注意不能与 Django 内置的标签或者过滤器名字冲突，如同 Python 中命名不可以使用关键字一样，所以我们在命名时应该尽量使用带有下划线的命名方式，这样可以确保名字不冲突。

上述操作完成后，我们就可以使用`{% load index_tags %}`加载自定义标签了，`load`标签将加载指定的的自定义标签，但是在`templatetags`目录中自定义标签或者过滤器的数量是没有限制的，你可以根据自己实际需求进行构建。

提示：`{% load xxx%}`将会载入给定模块名下的标签或者过滤器，而不是`app`应用下的中所有标签和过滤器。

要在模块内自定义标签，该模块必须包含一个名为`register`的模板层变量，且它的值是`template.Library`的实例，所有的标签和过滤器都是在其中注册的。所以我们需要打开`index_tags.py`文件，并在文件顶部加上如下代码：
```python
from django import template
register = template.Library()
```
## 实现自定义简单标签
简单标签通过接收参数，对输入的参数做一些处理并返回结果。
```python
#注册自定义简单标签
@register.simple_tag
def addstr_tag(strs):
  return 'Hello'%strs
```
`addstr_tag`函数使用`register.simple_tag`进行装饰，目的是能够将`addstr_tag`注册到模板系统中。然后我们就可以使用`{% load %}`加载自定义的标签了，使用如下方式：
```
{% load index_tags %}
```
加载之后我们就可以使用我们的自定义标签了，通过举例看一下实际的效果：
```
In [1]: from django.template import Template,Context
In [2]: t=Template("""
   ...: {% load index_tags %}
   ...: {% addstr_tag 'Django BookStore' %}
   ...: 
   ...: """)
   ...: t.render(Context())
Out[2]: 'Hello Django BookStore'
```
上述就是一个简单标签的实现过程，自定义不同类型的标签它们的过程是一样的，而且我们还可以通过`name`参数给自定义的标签其别名，这样在使用`load`加载时就可以直接使用别名了：
```python
@register.simple_tag(name='abc')
```
## 实现自定义引用标签
这种类型的标签可以对其他模板进行渲染，然后将渲染结果输出。
1)定义模板文件
在`BookStore/templates`中定义模板文件 inclusion.html ，并在 body 中编写如下代码：
```html
<p>{{ hello }}</p>
```
在`index_tags`中自定义引用标签：
```python
#注册自定义引用标签
@register.inclusion_tag('inclusion.html', takes_context=True)
#定义函数渲染模板文件 inclusion.html
def add_webname_tag(context, namestr): #使用takes_context=True此时第一个参数必须为context
  return {'hello':'%s %s'%(context['varible'], namestr)}
```
我可以看出，引用标签使用`register.inclusion_tag`来注册，它的第一个参数用来指定要被渲染的模板文件，`takes_context=True`参数可以让我们访问模板的当前环境上下文，并将当前环境上下文中的参数和值作为字典传入到函数的`contex5`参数中，当使用`take_context=True`时，注册标签函数的第一个参数必需为`context`。

通过 Django shell 编写如下代码看一下它是如何应用的，如下所示：
```
In [1]: from django.template import Template,Context
In [2]: t=Template("""
   ...: {% load index_tags %}
   ...: {% add_webname_tag '张三' %}
   ...: 
   ...: """)
   ...: t.render(Context({'varible':'Hello'}))
Out[2]: '\n\n<!DOCTYPE html>\n<html lang="en">\n<head>\n    <meta charset="UTF-8">\n    <title>test</title>\n</head>\n<body>\n<p>Hello 张三</p>\n\n</body>\n</html>\n\n'
```
从输出的结果可以得出，引用标签对`inclusion.html`模板进行了渲染，将`{{ hello }}`变量渲染成了`Hello 张三`。
## 实现自定义赋值标签
这个标签类似于简单标签，使用`register.simple_tag`进行注册，但它并不会直接输出结果，而是使用`as`关键字将结果储存在指定的上下文变量中，从而降低了传输上下文的成本。下面在`index_tags.py`中定义`test_as_tag`标签，如下所示：
```python
#注册自定义赋值标签
@register.simple_tag
def test_as_tag(strs):
    return 'Hello Test Tag-%s'%strs
```
使用自定义赋值标签，实例如下所示：
```
In [1]: from django.template import Template,Context
In [2]: t=Template("""
   ...: {% load index_tags %}
   ...: {% test_as_tag '欢迎你' as test %}
   ...: <p>{{ test }}</p>
   ...: """)
   ...: t.render(Context())
Out[2]: '\n\n\n<p>Hello Test Tag-欢迎你</p>\n'
```

# 模板继承
在模板继承中最常用的标签就是`{% block %}`与`{% extends %}`标签，其中`{% block% }`标签与`{% endblock %}`标签成对出现，而`{% extends %}`放在子模板的第一行且必须是模板中的第一个标签，标志着此模板继承自父模板，它们使用方法如下所示：
```python
#定义父模板可被重写内容
{%block block_name%}
...可以被子模板覆盖的内容
{%endblock block_name%}
#继承父模板
{% extends '父模板名称' %}
#子模板重写父模板
{%block block_name%}
...子模板覆盖后呈现的新内容
{%endblock block_name%}
```
需要注意的是子模板不需要重写父模板中的所有 block 标签定义的内容，未重写时，子模板原封不动的使用父模板中的内容。下面我们通过一个简单的例子来看一下具体的实现过程。

首先在 index/templates/index 目录下定义父模板 base.html，代码如下所示：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Welcome {% endblock title %}</title>
</head>
<body>
<!--区域1默认区域不可以被子模板修改-->
<p>尊敬的用户您好：</p>
<hr>
<!--区域2可以被子模板重写-->
{% block content %}
<p>这是主体内容可以被子模板重写</p>
{% endblock content %}
<hr>
<!--区域3可以被子模板重写-->
{% block footer %}
<p>这是结尾的内容也可以被重写</p>
{% endblock footer %}
</body>
</html>
然后在父模板同级路径下定义子模板文件 test.html，代码如下所示：
{% extends 'index/base.html' %}
<!--重写title-->
{% block title %} 欢迎你学习Django教程 {% endblock %}
<!--区域1保持父模板默认状态-->
<!--对父模板的区域2进行重写-->
{% block content %}
{% for item in course %}
<li>{{ item }}</li>
{% endfor %}
{% endblock content %}
{% block footer %}<p>最后希望<span style="color:red">{{ name }}</span>学有所成</p>
{% endblock footer %}
```
在 index/views.py 文件编写视图函数，如下所示：
```python
#定义父模板视图函数
def base_html(request):
    return render(request,'index/base.html')
#定义子模板视图函数
def index_html(request):
    name='xiaoming'
    course=['python','django','flask']
    return render(request,'index/test.html',locals())
```
我们在主路由使用 include 函数为 index 应用建立对应的分发式路由列表，操作步骤如下所示，首先在主路由列表关联 index 应用
```
from django.urls import path,include
from BookStore import views
urlpatterns = [path('index/',include('index.urls'))]
```
然后在 index 应用目录下新建 urls.py 文件，建立主路由对应的分发式路由，代码如下所示：
```python
from django.urls import path
from index import views
urlpatterns=[
#127.0.0.1:8000/index/test 访问子模板
path('test/',views.index_html),
#127.0.0.1:8000/index/base 访问父模板
path('base/',views.base_html)]
```
在浏览器地址栏输入父模板 url 地址进行访问，得到的结果如下所示：

模板继承父模板
图1：模板继承父模板
 
我们在父模板中标记了哪些区域可以被子模板重写覆盖，现在我们访问子模板地址，看看它又是如何的呢？展示结果如下所示：

模板继承子模板
图2：模板继承子模板
 
我们可以看出，子模板对父模板中 {% block %} 包含的内容进行了重写覆盖，这就是模板继承应用。如果在多个模板中出现了大量复杂的代码，那么就应该考虑使用模板继承来减少重复性代码。
