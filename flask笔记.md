# flask笔记

# （一）hello world

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```



# （二）flask配置方法

参考：1.https://zhuanlan.zhihu.com/p/24055329

​			2.https://www.jianshu.com/p/f1d341119df0?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation
​			3.http://www.pythondoc.com/flask/config.html

应用程序需要某种形式的配置。你可能会需要根据应用环境更改不同的设置，比如开关调试模式、 设置密钥、或是别的设定环境的东西。 

跟你如何加载配置无关，有一个配置对象用来维持加载的配置值：[`Flask`](http://www.pythondoc.com/flask/api.html#flask.Flask) 对象的 [`config`](http://www.pythondoc.com/flask/api.html#flask.Flask.config) 属性。 这是Flask自身放置特定配置的地方同时也是扩展放置它们配置值的地方。但是，这里也可以放置你自己的配置 。

 [`config`](http://www.pythondoc.com/flask/api.html#flask.Flask.config) 实际上是字典的一个子类且能够像字典一样被修改 

## 1 直接写入主脚本 

 当你的程序很小的时候，可以直接把配置写在主脚本里 

```python
from flask import Flask

app = Flask(__name__)
app.config['SECRET_KEY'] = 'some secret words'
app.config['DEBUG'] = True
app.config['ITEMS_PER_PAGE'] = 10
```

 使用字典的update方法可以简化代码： 

```python
from flask import Flask

app = Flask(__name__)
app.config.update(
    DEBUG=True,
    SECRET_KEY='some secret words',
    ITEMS_PER_PAGE=10
)
```



## 2 系统环境变量

 一些重要的配置， 比如密码等 ,可以设置在系统环境变量里，又或者放到某个服务器里，用的时候下载配置文件并读取配置。 然后使用os模块的getenv()方法获取，第二个参数作为默认值： 

```shell
set MAIL_USERNAME=me@greyli.com  # windows
export MAIL_USERNAME=me@greyli.com  # *unix
```

 获取变量并写入： 

```python
import os

from flask import Flask

app = Flask(__name__)
app.config['MAIL_USERNAME'] = os.getenv('MAIL_USERNAME', 'me@greyli.com')
```

 如果你使用虚拟环境，设置环境变量时注意要激活虚拟环境，同时不要给变量值加引号：

```python
set MAIL_USERNAME=me@greyli.com  # 结果是'me@greyli.com'
set MAIL_USERNAME='me@greyli.com'  # 结果是"'me@greyli.com'"
```



## 3 单独的配置文件

程序逐渐变大时，配置也逐渐增多，写在主脚本里太占地方，不够优雅，这时你应该已经把表单，路由，数据库模型等等分成独立的文件了。 

config.py

```python
SECRET_KEY = 'some secret words'
DEBUG = True
ITEMS_PER_PAGE = 10
```

 在创建程序实例后使用app.config.from_object导入配置： 

```python
from flask import Flask
import config


app = Flask(__name__)
app.config.from_object(config)
# print(app.config['SECRET_KEY'])
# print(app.config['ITEMS_PER_PAGE'])


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```

使用app.config.from_object导入配置：

```python
from flask import Flask
import config


app = Flask(__name__)
app.config.from_pyfile('config.py')
print(app.config['SECRET_KEY'])
print(app.config['ITEMS_PER_PAGE'])


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```



## 4 创建不同的配置类

大型项目需要多个配置组合，比如开发时的配置，测试的配置，部署的配置……这样我们需要在配置文件里创建不同的配置类，然后在创建程序实例时引入相应的配置类。

最佳实践是创建一个**存储通用配置**的基类，然后为不同的使用使用场景创建新的继承基类的配置类：

 config.py（这里为开发和测试创建了不同的数据库） 

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))


class BaseConfig:  # 基本配置类
    SECRET_KEY = os.getenv('SECRET_KEY', 'some secret words')
    ITEMS_PER_PAGE = 10


class DevelopmentConfig(BaseConfig):
    DEBUG = True
    SECRET_KEY = 'afasjkfnasfhas'
    SQLALCHEMY_DATABASE_URI = os.getenv('DEV_DATABASE_URL', 'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite'))


class TestingConfig(BaseConfig):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.getenv('TEST_DATABASE_URL', 'sqlite:///' + os.path.join(basedir, 'data-test.sqlite'))
    WTF_CSRF_ENABLED = False


config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

 通过from_object()方法导入配置： 

```python
from flask import Flask
from config import config


app = Flask(__name__)
# app.config.from_object(config)
# app.config.from_pyfile('config.py')
app.config.from_object(config['development'])
print(app.config['SECRET_KEY'])


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```

## 5 flask内置配置

```python
{
    'DEBUG': False,    # 是否开启Debug模式
    'TESTING': False,  # 是否开启测试模式
    'PROPAGATE_EXCEPTIONS': None,  # 异常传播(是否在控制台打印LOG) 当Debug或者testing开启后,自动为True
    'PRESERVE_CONTEXT_ON_EXCEPTION': None,  # 一两句话说不清楚,一般不用它
    'SECRET_KEY': None,  # 之前遇到过,在启用Session的时候,一定要有它
    'PERMANENT_SESSION_LIFETIME': 31,  # days , Session的生命周期(天)默认31天
    'USE_X_SENDFILE': False,  # 是否弃用 x_sendfile
    'LOGGER_NAME': None,  # 日志记录器的名称
    'LOGGER_HANDLER_POLICY': 'always',
    'SERVER_NAME': None,  # 服务访问域名
    'APPLICATION_ROOT': None,  # 项目的完整路径
    'SESSION_COOKIE_NAME': 'session',  # 在cookies中存放session加密字符串的名字
    'SESSION_COOKIE_DOMAIN': None,  # 在哪个域名下会产生session记录在cookies中
    'SESSION_COOKIE_PATH': None,  # cookies的路径
    'SESSION_COOKIE_HTTPONLY': True,  # 控制 cookie 是否应被设置 httponly 的标志，
    'SESSION_COOKIE_SECURE': False,  # 控制 cookie 是否应被设置安全标志
    'SESSION_REFRESH_EACH_REQUEST': True,  # 这个标志控制永久会话如何刷新
    'MAX_CONTENT_LENGTH': None,  # 如果设置为字节数， Flask 会拒绝内容长度大于此值的请求进入，并返回一个 413 状态码
    'SEND_FILE_MAX_AGE_DEFAULT': 12,  # hours 默认缓存控制的最大期限
    'TRAP_BAD_REQUEST_ERRORS': False,
    # 如果这个值被设置为 True ，Flask不会执行 HTTP 异常的错误处理，而是像对待其它异常一样，
    # 通过异常栈让它冒泡地抛出。这对于需要找出 HTTP 异常源头的可怕调试情形是有用的。
    'TRAP_HTTP_EXCEPTIONS': False,
    # Werkzeug 处理请求中的特定数据的内部数据结构会抛出同样也是“错误的请求”异常的特殊的 key errors 。
    # 同样地，为了保持一致，许多操作可以显式地抛出 BadRequest 异常。
    # 因为在调试中，你希望准确地找出异常的原因，这个设置用于在这些情形下调试。
    # 如果这个值被设置为 True ，你只会得到常规的回溯。
    'EXPLAIN_TEMPLATE_LOADING': False,
    'PREFERRED_URL_SCHEME': 'http',  # 生成URL的时候如果没有可用的 URL 模式话将使用这个值
    'JSON_AS_ASCII': True,
    # 默认情况下 Flask 使用 ascii 编码来序列化对象。如果这个值被设置为 False ，
    # Flask不会将其编码为 ASCII，并且按原样输出，返回它的 unicode 字符串。
    # 比如 jsonfiy 会自动地采用 utf-8 来编码它然后才进行传输。
    'JSON_SORT_KEYS': True,
    #默认情况下 Flask 按照 JSON 对象的键的顺序来序来序列化它。
    # 这样做是为了确保键的顺序不会受到字典的哈希种子的影响，从而返回的值每次都是一致的，不会造成无用的额外 HTTP 缓存。
    # 你可以通过修改这个配置的值来覆盖默认的操作。但这是不被推荐的做法因为这个默认的行为可能会给你在性能的代价上带来改善。
    'JSONIFY_PRETTYPRINT_REGULAR': True,
    'JSONIFY_MIMETYPE': 'application/json',
    'TEMPLATES_AUTO_RELOAD': None,
}
```



# （三）路由

## 配置路由两种方式

```python
# 配置路由两种方式
from flask import Flask

app = Flask(__name__)

"""
@app.route的流程
    1.执行 decorator = app.route('/index', methods=['POST','GET'])
    3.@decorator，相当于执行 decorator(index)
        执行代码add_url_rule(rule, endpoint, f, **options)     
"""
# 方式一
@app.route('/index', methods=['POST','GET'])
def index():
    return 'Hello World!'


# 方式二
def order():
    return 'order'
app.add_url_rule('/order', view_func=order)


if __name__ == '__main__':
    app.run()
```



## url_for

 给指定的函数构造 URL， 它接受函数名作为第一个参数，也接受对应 URL 规则的变量部分的命名参数。未知变量部分会添加到 URL 末尾作为查询参数 。一般与重定向一起使用。

```python
# url_for反向生成url
from flask import Flask, url_for

app = Flask(__name__)


@app.route('/')
def index():
    print(url_for('n1'))
    print(url_for('n2'))
    print(url_for('user', username='cys'))
    return "index"

@app.route('/login', methods=['POST', 'GET'],endpoint='n1')
def login():
    pass

@app.route('/logout', methods=['POST', 'GET'],endpoint='n2')
def logout():
    pass

@app.route('/user/<username>')
def user(username):
    pass


if __name__ == '__main__':
    app.run()

    
# 结果如下
# /login
# /logout
# /user/cys
```



## 路由转换器

参考：https://cloud.tencent.com/developer/article/1538653

### 1 自带转换器

- int转换器 ` <int:param> ` ：接收整数
- float转换器 ` <float:param> `: 接收浮点数
- string转换器 ` <string:param> `: 接收string类型（**默认则是string转换器**）
- path转换器 ` <path:param> `: 和默认的相似，但也接收斜线

```python
# 转换器
from flask import Flask


app = Flask(__name__)


# 转换器 <int:goods_id>
@app.route('/intconverter/<int:params>')
def int_param(params):
    return "-->> %s" % params

# 设置float转换器
@app.route('/floatconverter/<float:params>')
def float_param(params):
    return "-->> %s" % params

# 设置转换器 <string:goods_id>
@app.route('/strconverter/<string:params>')
def str_param(params):
    return "-->> %s" % params

# 设置path转换器
@app.route('/pathconverter/<path:params>')
def path_param(params):
    return "-->> %s" % params

if __name__ == '__main__':
    app.run()

```





### 2 自定义万能转换器

使用正则表达式

```python
# 自定义转换器
from flask import Flask
from werkzeug.routing import BaseConverter

app = Flask(__name__)


# 1.定义自己的转换器
class RegexConverter(BaseConverter):
    """
    自定义转换器
    """

    def __init__(self, url_map, regex):
        # 调用父类的初始化方法
        super(RegexConverter, self).__init__(url_map)
        # 将正则表达式的参数保存到对象属性中,flask会使用这个属性来进行路由的正则匹配
        self.regex = regex


# 2.将自定义的转换器添加到flask应用中
app.url_map.converters['re'] = RegexConverter


# 转换器
@app.route("/tel/<re(r'1[34578]\d{9}'):tel>")
def goods_detail(tel):
    return "tel is {}".format(tel)



if __name__ == '__main__':
    app.run()

```

手机号码转换器

```python
from werkzeug.routing import BaseConverter
class MobileConverter(BaseConverter):
    def __init__(self,url_map,mobile):
        super(MobileConverter, self).__init__(url_map)
        self.regex = r'1[34578]\d{9}'

app.url_map.converters['mobile'] = MobileConverter # 注册

@app.route("/goods/<mobile:goods>")
def goods_detail(goods):
    return "goods id is {}".format(goods)

```



## 视图函数添加装饰器

注意点：

1.被同一个装饰器装饰的视图函数，他们的endpoint一样，会报错，所以使用@functools.wraps(func)修饰，使他们为原来自己的函数名

2. 装饰器一定要写到@app.route下面，不然每次不生效

```python
# 给视图添加装饰器
from flask import Flask
import functools

app = Flask(__name__)


def wapper(func):
    # 被同一个装饰器装饰的视图函数，他们的endpoint一样，会报错，所以使用@functools.wraps(func)修饰，使他们为原来自己的函数名
    @functools.wraps(func)
    def inner(*args, **kwargs):
        print("inner")
        return func(*args, **kwargs)
    return inner


@app.route('/')
@wapper  # 装饰器一定要写到@app.route下面，不然每次不生效
def index():
    return 'index'


@app.route('/order', methods=['GET', 'POST'])
@wapper
def order():
    return 'order'


if __name__ == '__main__':
    app.run()

```





# （四）CBV和FBV

CBV： 基于类的视图 

FBV：  基于函数的视图 

前面所学都是FBV，下面演示CBV

1.第一种方式

继承 views.View

```python
# CBV示例
from flask import Flask, views
import functools

app = Flask(__name__)


def wapper(func):
    @functools.wraps(func)
    def inner(*args, **kwargs):
        print("inner")
        return func(*args, **kwargs)
    return inner


class IndexView(views.View):
    methods = ['GET']
    decorators = [wapper, ]

    def dispatch_request(self):
        print('Index')
        return 'Index'

app.add_url_rule('/index', view_func=IndexView.as_view(name='index'))  # name是endpoint


if __name__ == '__main__':
    app.run()
```

2.第二种方式

继承 views.MethodView

可以完成反射

```python
# CBV示例
from flask import Flask, views
import functools

app = Flask(__name__)


def wapper(func):
    @functools.wraps(func)
    def inner(*args, **kwargs):
        print("inner")
        return func(*args, **kwargs)
    return inner


class IndexView(views.MethodView):
    methods = ['GET', 'POST']
    decorators = [wapper, ]

    def get(self):
        print('get 请求')
        return 'get'

    def post(self):
        print('post 请求')
        return 'post'

app.add_url_rule('/index', view_func=IndexView.as_view(name='index'))  # name是endpoint

if __name__ == '__main__':
    app.run()

```



# （五）请求和响应

## 1 请求

request对象的属性

**1.request.method**  

请求方式:GET  POST

**2.request.args**

request.args  ：get参数字典

request.args.get(键) ： 获取字典中键的值

request.args.getlist(键)  ： 获取字典中键的值，得到一个列表 

**3.request.form**

request.form  ： 获取参数字典 

request.form.get(键)  ： 获取字典中键的值 

**4.request.cookies**

一个包含所有随请求提交的cookies的字典

**5.request.headers**

一个Werkzeug的EnvironHeaders对象，包含首部字段，可以以字典的形式操作

**6.request.path**
base_url

**7.request.files**
MultiDict包含所有上传文件的对象。每个键files都是来自的名称 。每个值都是一个Werkzeug 对象

flask.request.files  ：接收文件字典

flask.request.files.get(键)  ： 获取文件字典中的内容 

 保存文件到本地，方式一 ：

```text
文件对象 = flask.request.files.get(键)
文件对象.save(保存路径)
```

 保存文件到地址，方式二 

```text
with open(路径，'wb') as f:
    内容 = 文件对象.read()
    f.write(内容)
```

 文件的名字与大小 

```text
文件对象.name
文件对象.content_length
```



## 2 响应

可以响应多种内容，方式如下：

**1.字符串**

```python
return 'index'
```

**2.模板**

```python
return render_template('index.html', n1=123)
```

3.**重定向**

```python
return redirect('/index')
```

**4.json**
方式1

```python
return json.jums({})
```

方式2

```python
from flask import Flask, jsonify
return jsonfy({})
```

**5.make_response**

构造make_response对象可以设置cookie，响应头，状态码等

```python
response = make_response(render_template('index.html'), 200)
response.set_cookie("username", "cys")
response.headers['X-someting'] = 'A value'
response.delete_cookie('username')
return response
```



# （六）模板引擎

 参考：https://zhuanlan.zhihu.com/p/23669244

​			https://www.jianshu.com/p/e20e3671c680

## 1 基本语法

- {% ... %} 语句（[Statements](https://link.zhihu.com/?target=http%3A//jinja.pocoo.org/docs/dev/templates/%23list-of-control-structures)）
- {{ ... }} 打印模板输出的表达式（[Expressions](https://link.zhihu.com/?target=http%3A//jinja.pocoo.org/docs/dev/templates/%23expressions)）
- {# ... #} 注释
- \# ... ## 行语句（[Line Statements](https://link.zhihu.com/?target=http%3A//jinja.pocoo.org/docs/dev/templates/%23line-statements)）

## 2 变量

变量类型和获取方法，可以传整型、字符串、列表、字典、函数

```python
#  模板引擎
from flask import Flask, render_template, Markup

app = Flask(__name__)


@app.template_global()
def global_func(a, b):
    "每个模板都可以调用的函数"
    return a + b


def gen_input(value):
    return Markup("<input value='%s'/>"%value)


@app.route('/index')
def index():
    context = {
        'k1':123,
        'k2':[11,22,33],
        'k3':{'name':'oldboy', 'age':18},
        'k4':lambda x:x+1,
        'k5':gen_input  # 只有当前模板才能调用的函数
    }

    return render_template('index.html', **context)

@app.route('/order')
def order():
    context = {
        'k1':123,
        'k2':[11,22,33]
    }

    return render_template('order.html', **context)

if __name__ == '__main__':
    app.run()

```

html如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>{{ k1 }}</h1>
    <h1>{{ k2.0 }} {{ k2[1] }}</h1>
    <h1>{{ k3.name }} {{ k3['name'] }} {{ k3.get('name','aaa') }}</h1>
    <h1>{{ k4(6) }}</h1>
    <h1>{{ k5(666) }}</h1>
    <h1>{{ global_func(1,2) }}</h1>
</body>
</html>
```



## 3 过滤器

 过滤器用来修改变量，使用一个竖线和变量相隔。 

```django
{{ items|join(', ') }}
```

常用的内置过滤器：

- safe 渲染时不转义
- capitalize 首字母大写
- lower 小写
- upper 大写
- title 每个单词的首字母都转换成大写
- trim 去掉首尾空格
- striptags 去掉值里的HTML标签
- default 设置一个默认值，如果变量未定义，就用这个默认值替换。类似这样：

```django
{{ my_variable|default('my_variable is not defined') }}
```

- random(*seq*) 返回一个序列里的随机元素
- truncate(*s*, *length=255*, *killwords=False*, *end='...'*) 截取出指定长度的文章（文章摘要）
- format(*value*, **args*, ***kwargs*) 参考Python的字符串格式化函数
- length 左边如果是列表，输出列表的数量；如果是字符串，则输出字符串的长度



## 4 标签（控制语句）

 语法格式 ：{% 标签名 %} 

 **if**

```django
{% if data.bool %}
    <li>{{ data.bool }}值为真</li>
{% elif True %}
    <li>{{ True }} 值为真</li>
{% else %}
    <li>{{ data.bool }}值为假</li>
{% endif %}
```

**for循环**

```django
{% for i in data.xxxx %}
{# 错误的迭代方法TypeError: 'bool' object is not iterable #}
{#  {% for i in data.bool %}#}
    <li>{{ i }}</li>
{% else %}
    <li>当迭代的变量不存在时 则执行else</li>
{% endfor %}
```



## 5 继承

 语法：
{% extends %} 继承某个模板
{% block %} 挖坑和填坑
{{ super() }} 调用被替换掉的代码 

如下：

base.html：被继承的模板

```html
<!DOCTYPE html>
<html lang="en">
<head>
{% block header %}
  <meta charset="UTF-8">
  {% block meta %}
  {% endblock %}
  <title>{% block title%}首页{% endblock %}</title>
  <style>
    {% block style %}
      p{color:red;}
    {% endblock %}
  </style>
  {% block link %}
  {% endblock %}
  {% block script %}
  {% endblock %}
{% endblock %}
</head>
<body>
<header>头部</header>
{% block con %}
  我是中间的内容部分
{% endblock %}
<footer>尾部</footer>
</body>
</html>
```

extend.html：继承base.html的模板

```html
{% extends 'base.html' %}
{% block title %}
  我的首页
{% endblock %}
{% block style %}
  {{ super() }}
  p{color:green;}
{% endblock %}
{% block con %}
    <p>{{ super() }}</p>
{% endblock %}
```



# （七）装饰器（钩子函数）

## 1 before_request

before_request用于请求前验证

如果before_request里的其中一个函数有返回值，则直接跳到after_request的列表里从后往前验证。

```python
# before_request 和 after_request
from flask import Flask


app = Flask(__name__)


@app.before_request
def xxx1():
    print('前1')

@app.before_request
def xxx2():
    print('前2')


@app.route('/x1', methods=['GET', 'POST'])
def X1():
    print('视图函数x1')
    return '视图函数x1'


if __name__ == '__main__':
    app.run()
```

例子：使用before_request做登录验证

```python
from flask import Flask, request, session, redirect


app = Flask(__name__)
app.secret_key = "sasfangndga"

@app.before_request
def check_login():
    if request.path == '/login':
        return None
    user = session.get('user_info')
    if not user:
        return redirect('/login')


@app.route('/login', methods=['GET', 'POST'])
def login():
    return '视图函数x1'


@app.route('/index', methods=['GET', 'POST'])
def X2():
    print('视图函数x2')
    return '视图函数x2'


if __name__ == '__main__':
    app.run()

```



## 2 after_request

after_request用于请求后验证

after_request效果和django 的process_response是一样的,必须有返回值，没有则报错。

```python
# before_request 和 after_request
from flask import Flask


app = Flask(__name__)


@app.before_request
def xxx1():
    print('前1')

@app.before_request
def xxx2():
    print('前2')


@app.after_request
def yyy1(response):
    print('后1')
    return response

@app.after_request
def yyy2(response):
    print('后2')
    return response


@app.route('/x1', methods=['GET', 'POST'])
def X1():
    print('视图函数x1')
    return '视图函数x1'


@app.route('/x2', methods=['GET', 'POST'])
def X2():
    print('视图函数x2')
    return '视图函数x2'


if __name__ == '__main__':
    app.run()

# 执行顺序如下
# 前1
# 前2
# 视图函数x2
# 后2
# 后1
```

 **flask中间件装饰器执行顺序**： 如果多个app.before_request和app.after_request，  app.before_request是按照从上而下执行（文件的上下），app.after_request是自下而上执行。 



## 3 before_first_request

 第一次来请求操作的装饰器： 

```python
@app.before_first_request
def first(*args,**kwargs):
    pass
'''
只有第一次请求时候才执行的函数装饰器
'''
```



## 4 error_handlers

```python
@app.error_handlers(404)
def error_404(arg):
    '''自定义错误页面，根据状态码定制'''
    return "404错误啦"
```



## 5 template_filter

app.template_filter(‘过滤器名字’)。 

flask 中用@app.template_filter(‘name’)来自定义名字为name的过滤器，name可以不指定，当不指定的时候使用函数的名称。 

```python
@app.template_filter()
def db(a1,a2,a3):
    return a1+a2+a3
'''
效果和django的Filter相似，前端渲染的时候需要注意写法
{{ 1|db(2,3)}} 1是第一个参数，后面是2,3参数。
'''
```



## 6 template_global

声明全局可用的模板函数，见模板引擎-变量里的示例





# （八）闪现

参考：http://www.pythondoc.com/flask/patterns/flashing.html
			https://www.cnblogs.com/1996-11-01-614lb/p/8975842.html

 Flask 的闪现系统提供了一个良好的反馈方式。闪现系统的基本工作方式 是：在且只在下一个请求中访问上一个请求结束时记录的消息。一般我们结合布局模板来 使用闪现系统。 

 需求：有两个函数login 和index ，有一个人在向login页面发起请求，login生成一个错误，放到session，跳转到index显示错误，然后再把session移除，并且这个错去只能执行一次（也就是让你看一次）这个东西就可以用闪现是实现。

示例如下：

```python
from flask import Flask,session,flash,get_flashed_messages


app = Flask(__name__)
app.secret_key = "asklfnasfgasnklsdk"


@app.route("/x1", methods=["GET", "POST"])
def login():
    # session['msg'] = "回复哈哈哈哈哈哈"  #这是基于session做的
    flash("闪现消息1", category='x1')    #这是另一种方法，设置flash，这个内部也是基于session做的，flash其实就是把这个值设置到session上
    flash("闪现消息2", category='x2')#category表示对数据进行分类
    return "视图函数x1"


@app.route("/x2", methods=["GET", "POST"])
def index():
    data = get_flashed_messages(category_filter=['x1']) #这个是取上面我们设置的类似于错误信息的东西，这个其实就是在session上把他上面设置的值拿到并且删除
    #category_filter = ['x1'] 这个意思就是取x1那个对应的数据，两个都要拿就category_filter = ['x1','x1']
    print(data)
    # msg = session.pop('msg')  #这个拿完以后就没有了，这是基于session实现的，看完以后就删除了
    return "视图函数x2"

if __name__ == '__main__':
    app.run()
```



# （九）蓝图

参考：http://www.pythondoc.com/flask/blueprints.html

 Flask 使用了 *蓝图* 的概念在一个应用或者跨应用中构建应用组件以及支持通用模式。 蓝图很好地简化了大型应用工作的方式，并提供给 Flask 扩展在应用上注册操作的核心方法。 一个 `Blueprint` 对象与 `Flask` 应用对象的工作方式很像，但它确实不是一个应用， 而是一个描述如何构建或扩展应用的 *蓝图* 。 

蓝图的作用：

- 把一个应用分解成一系列的蓝图。对于大型的应用是理想化的；一个项目能实例化一个应用， 初始化一些扩展，以及注册一系列的蓝图。
- 以一个 URL 前缀和/或子域在一个应用上注册蓝图。 URL 前缀/子域名中的参数即成为这个蓝图下的所有视图函数的共同的视图参数（默认情况下）。
- 在一个应用中用不同的 URL 规则多次注册一个蓝图。
- 通过蓝图提供模板过滤器、静态文件、模板和其它功能。一个蓝图不一定要实现应用或视图函数。
- 初始化一个 Flask 扩展时，在这些情况中注册蓝图。



## 1 蓝图结构

### 结构1

一般大小的项目使用此结构即可

如下图：

![1623831770852](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1623831770852.png)

```
flaskBluePrint01  项目
-app1  程序子应用
	-static  静态文件
	-templates  模板文件
	-views  视图模块
		-account.py  视图文件
		-admin.py
		-user.py
	- __init__.py  初始化文件：注册应用，实例化对象，使用蓝图
-manage.py  项目启动文件
```



步骤如下：

**1.创建app**

app1/ __ _init_ _ _.py

```python
from flask import Flask

app = Flask(__name__)
```

**2.创建启动文件**

manage.py

```python
from app1 import app


if __name__ == '__main__':
    app.run()
```

**3.创建子视图**

app1/views/ account.py

```python
from flask import Blueprint

ac = Blueprint('ac', __name__)


@ac.route('/login')
def login():
    return 'login'


@ac.route('/logout')
def logout():
    return 'logout'
```

app1/views/admin.py

```python
from flask import Blueprint

ad = Blueprint('ad', __name__)


@ad.route('/home')
def home():
    return 'home'
```

app1/views/ user.py

```python
from flask import Blueprint

us = Blueprint('us', __name__)


@us.route('/info')
def info():
    return 'info'
```

4.**注册蓝图**
app/ __ _init_ _ _.py

```python
from flask import Flask

app = Flask(__name__)


from .views import account
from .views import admin
from .views import user


app.register_blueprint(account.ac)
app.register_blueprint(admin.ad)
app.register_blueprint(user.us)
```



### 结构2

对于一些大型项目可以使用以下数据结构

![1623834506932](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1623834506932.png)



 注:如果在 templates 中存在和 templates_users 有同名模板文件时, 则系统会优先使用 templates 中的文件 

步骤如下：

**1.创建app**

app/ __ _init_ _ _.py

```python
from flask import Flask

app = Flask(__name__)
```

**2.创建启动文件**

manage.py

```python
from app1 import app


if __name__ == '__main__':
    app.run()
```

**3.创建子视图子应用和子视图**

app1/admin/_ _ init_ _.py

```python
from flask import Blueprint

ad=Blueprint('ad', __name__)

from . import views
```

app1/admin/views.py

```python
from . import ad

@ad.route('/home')
def home():
    return 'home'
```

app1/web/_ _ init_ _.py

```python
from flask import Blueprint


wb=Blueprint('wb', __name__)

from .views import *
```

app1/web/views.py

```python
from . import wb


@wb.route('/login')
def login():
    return 'login'


@wb.route('/logout')
def logout():
    return 'logout'
```



4.**注册蓝图**

app/ __ _init_ _ _.py

```python
from flask import Flask

app = Flask(__name__)
app.debug=True

from .admin import ad
from .web import wb


app.register_blueprint(ad)
app.register_blueprint(wb)
```





## 2 添加前缀

如下，修改app1|views| admin.py文件

```python
from flask import Blueprint

ad = Blueprint('ad', __name__, url_prefix='/admin')


@ad.route('/home')
def home():
    return 'home'
```

也可以在注册时候添加，如下：

```python
from flask import Flask

app = Flask(__name__)


from .views import account
from .views import admin
from .views import user


app.register_blueprint(account.ac)
app.register_blueprint(admin.ad, url_prefix="/admin")
app.register_blueprint(user.us)
```



## 3 静态文件

 一个蓝图可以通过 static_folder 关键字参数提供一个指向文件系统上文件夹的路 径，来公开一个带有静态文件的文件夹。这可以是一个绝对路径，也可以是相对于蓝图文件夹的路径: 

```python
ad=Blueprint('ad', __name__, static_folder='static_admin')
```



## 4 模板

```python
ad=Blueprint('ad', 
             __name__, 
             static_folder='static_admin', 
             template_folder='templates_admin')
```



## 5 构建 URLs

 当你想要从一个页面链接到另一个页面，你可以像通常一个样使用 `url_for()` 函数，只是你要在 URL 的末端加上蓝图的名称和一个点(`.`)作为前缀: 

```python
url_for('admin.index')
```

此外，如果你在一个蓝图的视图函数或是模板中想要从链接到同一蓝图下另一个端点， 你可以通过对端点的只加上一个点作为前缀来使用相对的重定向:

```python
url_for('.index')
```



# （十）flask上下文

参考：https://zhuanlan.zhihu.com/p/26097310
			https://www.cnblogs.com/hq82/p/10451483.html
			https://www.jianshu.com/p/7a7efbb7205f



![1624281696776](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1624281696776.png)











# （十一）使用redis存session

使用flask_session模块

更改配置文件：

settings.py

```python
from datetime import timedelta
from redis import Redis


class Config():
    DEBUG = True
    SECRET_KEY = "AKL;FGMASDFAS"
    PERMANENT_SESSION_LIFETIME = timedelta(minutes=20)
    SESSION_REFRESH_EACH_REQUEST = True  # 是否应该为每一个请求设置cookie，默认为True，如果为False则必须显性调用set_cookie函数
    SESSION_TYPE = "redis"  # 使用redis存储session


class ProductionConfig(Config):
    SESSION_REDIS = Redis(host="127.0.0.1", port='6379')


class DevelopmentConfig(Config):
    SESSION_REDIS = Redis(host="127.0.0.1", port='6379')


class TestConfig(Config):
    pass
```

acconts.py

```python
from flask import Blueprint, request, render_template, session, redirect
from uuid import uuid4

ac = Blueprint('ac', __name__)


@ac.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    user = request.form.get('user')
    pwd = request.form.get('pwd')

    if user == 'cys' and pwd == '123':
        uid = str(uuid4())
        session.permanent = True
        session['user_info'] = {'id':uid, 'name':user}
        return redirect('/index')
    else:
        return render_template('login.html', msg='用户名或密码错误')


@ac.route('/index')
def logout():
    return 'welcome'
```

_ _init_ __.py

```python
from flask import Flask
from flask_session import Session


from .views import account
from .views import admin
from .views import user

def create_app():
    app = Flask(__name__)
    app.config.from_object('settings.DevelopmentConfig')

    app.register_blueprint(account.ac)
    app.register_blueprint(admin.ad, url_prefix="/admin")
    app.register_blueprint(user.us)

    # 将session替换成redis sesion
    Session(app)


    return app
```



# （十二）数据库连接池

参考：https://www.cnblogs.com/wangkun122/articles/8992637.html

通过DBUtils实现数据库连接池

安装：

```shell
pip install DBUtils==1.2
```

注意：python3现在回安装最新2.0版本的，from DBUtils.PersistentDB import PersistentDB这样导入时会找不到模块，要安装低版本到1.2版本。



**模式一**

 为每个线程创建一个连接，线程即使调用了close方法，也不会关闭，只是把连接重新放到连接池，供自己线程再次使用。当线程终止时，连接自动关闭 

```python
from DBUtils.PersistentDB import PersistentDB
import pymysql
POOL = PersistentDB(
    creator=pymysql,  # 使用链接数据库的模块
    maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
    setsession=[],  # 开始会话前执行的命令列表。如：["set datestyle to ...", "set time zone ..."]
    ping=0,
    # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
    closeable=False,
    # 如果为False时， conn.close() 实际上被忽略，供下次使用，再线程关闭时，才会自动关闭链接。如果为True时， conn.close()则关闭链接，那么再次调用pool.connection时就会报错，因为已经真的关闭了连接（pool.steady_connection()可以获取一个新的链接）
    threadlocal=None,  # 本线程独享值得对象，用于保存链接对象，如果链接对象被重置
    host='127.0.0.1',
    port=3306,
    user='root',
    password='123',
    database='pooldb',
    charset='utf8'
)

def func():
    conn = POOL.connection(shareable=False)
    cursor = conn.cursor()
    cursor.execute('select * from tb1')
    result = cursor.fetchall()
    cursor.close()   #这里的关闭没有真正的关闭了，线程在使用的话还是使用的哪一个链接
    conn.close()

func()
```

**模式二**
 创建一批连接到连接池，供所有线程共享使用。 

 由于pymysql、MySQLdb等threadsafety值为1，所以该模式连接池中的线程会被所有线程共享。 

```python
import pymysqlfrom DBUtils.PooledDB 
import PooledDB
PYMYSQL_POOL = PooledDB(
        creator=pymysql,  # 使用链接数据库的模块
        maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
        mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
        maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
        maxshared=3,   #这个参数没多大用，  最大可以被大家共享的链接
        # 链接池中最多共享的链接数量，0和None表示全部共享。PS: 无用，因为pymysql和MySQLdb等模块的 threadsafety都为1，所有值无论设置为多少，_maxcached永远为0，所以永远是所有链接都共享。
        blocking=True,  # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
        maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
        setsession=[],  # 开始会话前执行的命令列表。如：["set datestyle to ...", "set time zone ..."]
        ping=0,
        # ping MySQL服务端，检查是否服务可用。# 如：0 = None = never, 1 = default = whenever it is requested, 2 = when a cursor is created, 4 = when a query is executed, 7 = always
        host='127.0.0.1',
        port=3306,
        user='root',
        password='123456',
        database='s8day127db',#链接的数据库的名字
        charset='utf8'
    )

def func():
    # 检测当前正在运行连接数的是否小于最大链接数，如果不小于则：等待或报raise TooManyConnections异常
    # 否则
    # 则优先去初始化时创建的链接中获取链接 SteadyDBConnection。
    # 然后将SteadyDBConnection对象封装到PooledDedicatedDBConnection中并返回。
    # 如果最开始创建的链接没有链接，则去创建一个SteadyDBConnection对象，再封装到PooledDedicatedDBConnection中并返回。
    # 一旦关闭链接后，连接就返回到连接池让后续线程继续使用。
    conn = POOL.connection()

    # print(th, '链接被拿走了', conn1._con)
    # print(th, '池子里目前有', pool._idle_cache, '\r\n')

    cursor = conn.cursor()
    cursor.execute('select * from tb1')
    result = cursor.fetchall()
    conn.close()  #这里的关闭也没有彻底关闭，而是将线程放回数据库连接池中去了


func()
```



**应用：使用数据库连接池进行登录验证**

把数据库连接池放进配置文件

settings.py

```python
from datetime import timedelta
from redis import Redis
from DBUtils.PooledDB import PooledDB
import pymysql

class Config():
    DEBUG = True
    SECRET_KEY = "AKL;FGMASDFAS"
    PERMANENT_SESSION_LIFETIME = timedelta(minutes=20)
    SESSION_REFRESH_EACH_REQUEST = True  # 是否应该为每一个请求设置cookie，默认为True，如果为False则必须显性调用set_cookie函数
    SESSION_TYPE = "redis"  # 使用redis存储session

    PYMYSQL_POOL = PooledDB(
        creator=pymysql,  # 使用链接数据库的模块
        maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
        mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
        maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
        maxshared=3,  # 这个参数没多大用，  最大可以被大家共享的链接
        blocking=True,
        maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
        setsession=[], 
        ping=0,
        host='127.0.0.1',
        port=3306,
        user='root',
        password='123456',
        database='flask01',  # 链接的数据库的名字
        charset='utf8'
    )


class ProductionConfig(Config):
    SESSION_REDIS = Redis(host="127.0.0.1", port='6379')


class DevelopmentConfig(Config):
    SESSION_REDIS = Redis(host="127.0.0.1", port='6379')


class TestConfig(Config):
    pass
```



sql_helper.py

使用数据库连接池

```python
import pymysql
from flask import current_app

class SQLHelper():

    @staticmethod
    def open():
        POOL = current_app.config['PYMYSQL_POOL']
        conn = POOL.connection()
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        return conn, cursor

    @staticmethod
    def close(conn, cursor):
        conn.commit()
        cursor.close()
        conn.close()

    @classmethod
    def fetch_one(cls, sql, args):
        conn, cursor = cls.open()
        cursor.execute(sql, args)
        obj = cursor.fetchone()
        cls.close(conn, cursor)

        return obj

    @classmethod
    def fetch_all(cls, sql, args):
        conn, cursor = cls.open()
        cursor.execute(sql, args)
        obj = cursor.fetchall()
        cls.close(conn, cursor)

        return obj
```



account.py

登录验证

```python
from flask import Blueprint, request, render_template, session, redirect
from uuid import uuid4
from ..utils.sql_helper import SQLHelper


ac = Blueprint('ac', __name__)


@ac.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    user = request.form.get('user')
    pwd = request.form.get('pwd')

    # 数据库中查询数据
    obj = SQLHelper.fetch_one("select id,name from users where name=%s and pwd=%s", [user, pwd])

    if user == 'cys' and pwd == '123':
        uid = str(uuid4())
        session.permanent = True
        session['user_info'] = {'id':obj['id'], 'name':user}
        return redirect('/index')
    else:
        return render_template('login.html', msg='用户名或密码错误')


@ac.route('/index')
def logout():
    return 'welcome'
```



# （十三）wtforms

参考：https://www.cnblogs.com/wupeiqi/articles/8202357.html

用于对python web框架做表单验证，适用于任何python框架

安装：

```shell
pip install wtforms
```

例如，使用wtforms进行登录注册验证

项目名：flask_wtforms

文件：app1/views/account.py

```python
from flask import Blueprint, request, render_template, session, redirect
from wtforms import Form, validators, widgets
from wtforms.fields import simple, core, html5
from uuid import uuid4
from ..utils.sql_helper import SQLHelper


ac = Blueprint('ac', __name__)


class LoginForm(Form):
    user = simple.StringField(
        # label='用户名',
        validators=[
            validators.DataRequired(message='用户名不能为空'),
            # validators.length(min=6, max=18, message='用户名长度必须大于%(min)d且小于%(max)d')
        ],
        widget=widgets.TextInput(),
        render_kw={'class':'form-control'}
    )

    pwd = simple.PasswordField(
        # label='密码',
        validators=[
            validators.DataRequired(message='密码不能为空'),
            # validators.Length(min=8, message='用户名长度必须大于%(min)d'),
            # validators.Regexp(regex="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@$!%*?&])[A-Za-z\d$@$!%*?&]{8,}",
            #                   message='密码至少8个字符，至少1个大写字母，1个小写字母，1个数字和1个特殊字符')
        ],
        widget=widgets.PasswordInput(),
        render_kw={'class':'form-control'}
    )

class RegisterForm(Form):
    name = simple.StringField(
        label='用户名',
        validators=[
            validators.DataRequired()
        ],
        widget=widgets.TextInput(),
        render_kw={'class': 'form-control'},
        default='alex'
    )

    pwd = simple.PasswordField(
        label='密码',
        validators=[
            validators.DataRequired(message='密码不能为空.')
        ],
        widget=widgets.PasswordInput(),
        render_kw={'class': 'form-control'}
    )

    pwd_confirm = simple.PasswordField(
        label='重复密码',
        validators=[
            validators.DataRequired(message='重复密码不能为空.'),
            validators.EqualTo('pwd', message="两次密码输入不一致")
        ],
        widget=widgets.PasswordInput(),
        render_kw={'class': 'form-control'}
    )

    email = html5.EmailField(
        label='邮箱',
        validators=[
            validators.DataRequired(message='邮箱不能为空.'),
            validators.Email(message='邮箱格式错误')
        ],
        widget=widgets.TextInput(input_type='email'),
        render_kw={'class': 'form-control'}
    )

    gender = core.RadioField(
        label='性别',
        choices=(
            (1, '男'),
            (2, '女'),
        ),
        coerce=int
    )

    city = core.SelectField(
        label='城市',
        choices=SQLHelper.fetch_all("SELECT id,name FROM city", {}, None),
        coerce = int
    )

    hobby = core.SelectMultipleField(
        label='爱好',
        choices=(
            (1, '篮球'),
            (2, '足球'),
        ),
        coerce=int
    )

    favor = core.SelectMultipleField(
        label='喜好',
        choices=(
            (1, '篮球'),
            (2, '足球'),
        ),
        widget=widgets.ListWidget(prefix_label=False),
        option_widget=widgets.CheckboxInput(),
        coerce=int,
        default=[1, 2]
    )

    def __init__(self, *args, **kwargs):
        super(RegisterForm, self).__init__(*args, **kwargs)
        self.city.choices = SQLHelper.fetch_all("SELECT id,name FROM city", {}, None)  # 每次实例化类时都实时更新数据，防止数据库数据更新后，这里数据未更新

    def validate_pwd_confirm(self, field):
        """
        自定义pwd_confirm字段规则，例：与pwd字段是否一致
        :param field:
        :return:
        """
        # 最开始初始化时，self.data中已经有所有的值

        if field.data != self.data['pwd']:
            # raise validators.ValidationError("密码不一致") # 继续后续验证
            raise validators.StopValidation("密码不一致")  # 不再继续后续验证


@ac.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        form = LoginForm()
        return render_template('login.html', form=form)
    else:
        form = LoginForm(formdata=request.form)
        if not form.validate():  # 验证不通过则返回登录页，提示错误信息
            return render_template('login.html', form=form)
        else:
            # 数据库中查询数据
            obj = SQLHelper.fetch_one("select id,name from users where name=%(user)s and pwd=%(pwd)s", form.data)

            if obj:
                session.permanent = True
                session['user_info'] = {'id': obj['id'], 'name': obj['name']}
                return redirect('/index')
            else:
                return render_template('login.html', form=form, msg='用户名或密码错误')


@ac.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'GET':
        form = RegisterForm()
        return render_template('register.html', form=form)
    else:
        form = RegisterForm(formdata=request.form)
        if form.validate():
            print(form.data)
        else:
            print(form.errors)
        return 'welcome'

@ac.route('/index')
def logout():
    return 'welcome'
```

app1/templates/login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页</title>
</head>
<body>
    <h1>登录</h1>
    <form method="post">
        {{ form.user }} {{ form.user.errors[0] }}
        {{ form.pwd }} {{ form.pwd.errors[0] }}
        <input type="submit" value="提交">{{ msg }}
    </form>
</body>
</html>
```

app1/templates/register.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>rigister</title>
</head>
<body>
    <form method="post">
         {% for item in form %}
        <p>{{ item.label }}: {{ item }} {{ item.errors[0] }}</p>
    {% endfor %}
    <input type="submit" value="提交">
    </form>

</body>
</html>
```



# （十四）SQLAlchemy

参考：https://blog.csdn.net/u011146423/article/details/87605812

​			[Python开发【第十九篇】：Python操作MySQL - 武沛齐 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wupeiqi/articles/5713330.html)

​			[SQLAlchemy 学习笔记_JP.Zhang-CSDN博客](https://blog.csdn.net/u011521019/article/details/50793996)

## 1 简介

SQLAlchemy的是Python的SQL工具包和对象关系映射，给应用程序开发者提供SQL的强大功能和灵活性。它提供了一套完整的企业级的持久性模式，专为高效率和高性能的数据库访问，改编成简单的Python的领域语言。
SQLAlchemy是Python界的ORM（Object Relational Mapper）框架，它两个主要的组件： SQLAlchemy ORM 和 SQLAlchemy Core 。

![img](https://img-blog.csdn.net/20160304005104895)

## 2 安装

```shell
pip install SQLAlchemy
```



## 3 基本使用

创建表

类使用声明式至少需要一个__tablename__属性定义数据库表名字，并至少一Column是主键

```python
# SQLAlchemy练习
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String, UniqueConstraint, Index
from sqlalchemy.ext.declarative import declarative_base
# declarative_base类维持了一个从类到表的关系，通常一个应用使用一个base实例，所有实体类都应该继承此类对象

Base = declarative_base()

# 类使用声明式至少需要一个__tablename__属性定义数据库表名字，并至少一Column是主键
# 创建单表
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32))
    extra = Column(String(16))


def init_db():
    # 数据库链接引擎
    engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)
    # 创建表
    Base.metadata.create_all(engine)

def drop_db():
    engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)
    # 删除表
    Base.metadata.drop_all(engine)

if __name__ == "__main__":
    init_db()
    # drop_db()
```

插入数据

```python
# SQLAlchemy练习
import models
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# 数据库链接引擎
engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)

se = sessionmaker(bind=engine)
session = se()

u1 = models.User(name='a1', extra='aa')
u2 = models.User(name='a2', extra='bb')

session.add(u1)
session.add(u2)


session.commit()
# session.rollback() # 回滚
```

session.rollback()表示回滚



## 4 连接

SQLAlchemy 把一个引擎的源表示为一个连同设定引擎选项的可选字符串参数的 URI。URI 的形式是:

```python
dialect+driver://username:password@host:port/database
```


该字符串中的许多部分是可选的。如果没有指定驱动器，会选择默认的（确保在这种情况下 不 包含 + ）。

Postgres:

```python
postgresql://scott:tiger@localhost/mydatabase
```

MySQL:

```python
mysql://scott:tiger@localhost/mydatabaseOracle:
```

oracle:

```python
//scott:tiger@127.0.0.1:1521/sidname
```

SQLite (注意开头的四个斜线):

```python
sqlite:absolute/path/to/foo.db
```



## 5 数据类型

![这里写图片描述](https://img-blog.csdn.net/20161107010358428)

SQLAlchemy列选项：

![img](https://img-blog.csdn.net/20161107010602160)



## 6 执行原生sql

```python
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)

# 执行SQL
cur = engine.execute(
    "select * from users"
)

# 获取第一行数据
re1 = cur.fetchone()
# 获取第n行数据
# re2 = cur.fetchmany(2)
# 获取所有数据
re3 = cur.fetchall()

print(re3)
# 注意：当执行fetchone()后，数据已经被去除一条了，即使fetchall()，取出的数据也是从第二条数据开始的
```

engine.execute默认使用了数据库连接池，也可以使用DBUtils + pymysql做连接池



## 7 增删改查

表增加

models2.py

```python
# sqlalchemy创建多个关联表
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String, UniqueConstraint, Index, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
import datetime


Base = declarative_base()


class Classes(Base):
    __tablename__ = 'classes'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32), nullable=False, unique=True)


class Student(Base):
    __tablename__ = 'student'
    id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(32), nullable=False, unique=True)
    password = Column(String(32), nullable=False)
    ctime = Column(DateTime, default=datetime.datetime.now)  # 注意now后不要加()

    # 外键：对应的班级
    class_id = Column(Integer, ForeignKey('classes.id'))


class Hobby(Base):
    __tablename__ = 'hobby'
    id = Column(Integer, primary_key=True, autoincrement=True)
    caption = Column(String(32), default='篮球')

# 建立多对多的关系
class Student2Hobby(Base):
    __tablename__ = 'student2hobby'
    id = Column(Integer, primary_key=True, autoincrement=True)
    student_id = Column(Integer, ForeignKey('student.id'))
    hobby_id = Column(Integer, ForeignKey('hobby.id'))

    # 增加联合唯一索引
    __table_args__ = (
        UniqueConstraint('student_id', 'hobby_id', name='uni_student_id_hobby_id'),
    )


def init_db():
    # 数据库链接引擎
    engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)
    # 创建表
    Base.metadata.create_all(engine)

def drop_db():
    engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)
    # 删除表
    Base.metadata.drop_all(engine)

if __name__ == "__main__":
    init_db()
    # drop_db()
```



### 增

```python
# sqlalchemy的增删改查

import models2
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# 数据库链接引擎
engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)

se = sessionmaker(bind=engine)
session = se()

# 单条增加
# c1 = models2.Classes(name='python入门')
# session.add(c1)
# session.commit()
# session.close()


# 多条增加
# c2 = [
#     models2.Classes(name='python进阶'),
#     models2.Classes(name='python高级'),
#     models2.Classes(name='python web')
# ]
# session.add_all(c2)
# session.commit()
# session.close()

```



### 删改

```python
# sqlalchemy的增删改查

import models2
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# 数据库链接引擎
engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)

se = sessionmaker(bind=engine)
session = se()



# 删除
# session.query(models2.Classes).filter(models2.Classes.id > 2).delete()
# session.commit()


# 修改
# session.query(models2.Classes).filter(models2.Classes.id > 2).update({"name" : "099"})
session.query(models2.Classes).filter(models2.Classes.id > 2).update({models2.Classes.name: models2.Classes.name + "099"}, synchronize_session=False)
synchronize_session = False  # 对字段执行字符串操作
# session.query(models2.Classes).filter(models2.Classes.id > 2).update({"num": models2.Classes.num + 1}, synchronize_session="evaluate")
# synchronize_session = 'evaluate'  # 对字段执行数值型操作
session.commit()
```



### 查

参考：[Flask-SQLAlchemy - 武沛齐 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wupeiqi/articles/8259356.html)

1.以下的方法都是返回一个新的查询，需要配合执行器使用。

filter(): 过滤，功能比较强大，多表关联查询。
filter_by()：过滤，一般用在单表查询的过滤场景。

order_by()：排序。默认是升序，降序需要导包：from sqlalchemy import * 。然后引入desc方法。比如order_by(desc("email")).按照邮箱字母的降序排序。

group_by()：分组。

2.以下都是一些常用的执行器：配合上面的过滤器使用。

get()：获得id等于几的函数。

   比如：查询id=1的对象。

   get(1)，切记：括号里没有“id=”，直接传入id的数值就ok。因为该函数的功能就是查询主键等于几的对象。

all()：查询所有的数据。

first()：查询第一个数据。

count()：返回查询结果的数量。

paginate()：分页查询，返回一个分页对象。

   paginate(参数1，参数2，参数3)=>参数1：当前是第几页；参数2：每页显示几条记录；参数3：是否要返回错误。

   返回的分页对象有三个属性：items：获得查询的结果，pages：获得一共有多少页，page：获得当前页。

3.常用的逻辑符：

需要导入包才能用的有：from sqlalchemy import * 

not_、and_、or_ 还有上面说的排序desc。

常用的内置的有：in_：表示某个字段在什么范围之中。

4.其他关系的一些数据库查询：

endswith()：以什么结尾。

startswith()：以什么开头。

contains()：包含

```python
# sqlalchemy的增删改查

import models2
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.sql import text


# 数据库链接引擎
engine = create_engine("mysql+pymysql://root:root@127.0.0.1:3306/flask01?charset=utf8", max_overflow=5)

se = sessionmaker(bind=engine)
session = se()


# 查询

r1 = session.query(models2.Classes).all()  # 返回models2.Classes对象

# label相当于sql里的as，取别名，后面结果集使用也是使用别名
r2 = session.query(models2.Classes.name.label('xx'), models2.Classes.name).all()

# 查询课程名字为python入门的课程
r3 = session.query(models2.Classes).filter(models2.Classes.name == "python入门").all()
# 也可使用filter_by，注意filter_by的传参和filter不一样
r4 = session.query(models2.Classes).filter_by(name='python入门').all()

# 查询课程名字为python入门的课程的第一个
r5 = session.query(models2.Classes).filter_by(name='python入门').first()

# 查询id<2且课程名字为python入门的课程
r6 = session.query(models2.Classes).filter(text("id<:value and name=:name")).params(value=2, name='python入门').order_by(models2.Classes.id).all()

# from_statement相当于子查询
r7 = session.query(models2.Classes).from_statement(text("SELECT * FROM Classes where name=:name")).params(name='python入门').all()

print(r7)
session.close()
# 注意：除了r2，其他结果都为models2.Classes对象
```

```python
#　条件
ret = session.query(Users).filter_by(name='alex').all()
ret = session.query(Users).filter(Users.id > 1, Users.name == 'eric').all()
ret = session.query(Users).filter(Users.id.between(1, 3), Users.name == 'eric').all()
ret = session.query(Users).filter(Users.id.in_([1,3,4])).all()
ret = session.query(Users).filter(~Users.id.in_([1,3,4])).all()
ret = session.query(Users).filter(Users.id.in_(session.query(Users.id).filter_by(name='eric'))).all()
from sqlalchemy import and_, or_
ret = session.query(Users).filter(and_(Users.id > 3, Users.name == 'eric')).all()
ret = session.query(Users).filter(or_(Users.id < 2, Users.name == 'eric')).all()
ret = session.query(Users).filter(
    or_(
        Users.id < 2,
        and_(Users.name == 'eric', Users.id > 3),
        Users.extra != ""
    )).all()


# 通配符
ret = session.query(Users).filter(Users.name.like('e%')).all()
ret = session.query(Users).filter(~Users.name.like('e%')).all()

# 限制
ret = session.query(Users)[1:2]

# 排序
ret = session.query(Users).order_by(Users.name.desc()).all()
ret = session.query(Users).order_by(Users.name.desc(), Users.id.asc()).all()

# 分组
from sqlalchemy.sql import func

ret = session.query(Users).group_by(Users.extra).all()
ret = session.query(
    func.max(Users.id),
    func.sum(Users.id),
    func.min(Users.id)).group_by(Users.name).all()

ret = session.query(
    func.max(Users.id),
    func.sum(Users.id),
    func.min(Users.id)).group_by(Users.name).having(func.min(Users.id) >2).all()

# 连表

ret = session.query(Users, Favor).filter(Users.id == Favor.nid).all()
# 默认使用外键进行连表
ret = session.query(Person).join(Favor).all()  # 内连接

ret = session.query(Person).join(Favor, isouter=True).all() # 左连接


# 组合
q1 = session.query(Users.name).filter(Users.id > 2)
q2 = session.query(Favor.caption).filter(Favor.nid < 2)
ret = q1.union(q2).all()

q1 = session.query(Users.name).filter(Users.id > 2)
q2 = session.query(Favor.caption).filter(Favor.nid < 2)
ret = q1.union_all(q2).all()

```



#  （十五）falsk_script

参考：[Flask干货：访问数据库——Flask-Script工具的使用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/269820011)



# （十六）Flask-SQLAlchemy

参考：[Flask-SQLAlchemy详解 - 简书 (jianshu.com)](https://www.jianshu.com/p/f7ba338016b8)













