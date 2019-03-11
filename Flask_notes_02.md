# 【整理】Flask 学习笔记之二

Unreal Engine 里也有一个 Blueprint 的概念。不知道有什么联系，UE 研究的还不够深，这里咱不展开，留待日后补习。

### 蓝图（blueprint）

为了在一个或多个应用中，使应用模块化并且支持常用方案，Flask 引入了蓝图概念。蓝图可以极大地简化大型应用并为了扩展提供集中地注册入口。**Blueprint**对象与**Flask**应用对象的工作方式类似，但不是一个真正的应用，它为 Flask 描述了该如何构建和扩展应用。

### 为什么应用蓝图

蓝图的用途：

- 把一个应用分解为一套蓝图。针对大型应用的理想方案：一个项目可以实例化一个应用，初始化多个扩展，并注册需多个蓝图；
- 在一个应用的 URL 前缀和/或子域上注册一个蓝图。URL 前缀和/或子域的参数成为蓝图中所有视图的通用试图参数（默认情况下）；
- 使用不同的 URL 规则在应用中多次注册蓝图；
- 通过蓝图提供模板过滤器、静态文件、模板和其它工具。蓝图不必执行应用或视图函数；
- 当初始化一个 Flask 扩展是，为以上任意一种用户注册一个蓝图

Flask 中的蓝图不是一个可插拔的应用，因为它不是一个真正的应用，而是一套可以注册在应用中的操作，并且可以注册多次。为什么不采用多个应用对象？多应用会导致每个应用都有自己独立的配置，且只能在 WSGI 层中管理应用。而使用蓝图，应用会在 Flask 层中进行管理，共享配置，通过注册按需改变应用对象。

蓝图的缺点 ，除非应用被销毁，否则不能注销蓝图。

感觉 Flask 应用像是一个“经理”，手底下管着一帮“小弟”--蓝图，每个小弟各司其职。

### 创建蓝图

代码如下：

``` Python
from flask import Blueprint, render_template, abort
from jinja2 import TemplateNotFound

# 声明蓝图
simple_page = Blueprint('simple_page', __name__,
                        template_folder='templates')

# 关联蓝图
@simple_page.route('/', defaults={'page': 'index'})
@simple_page.route('/<page>')
def show(page):
    try:
        return render_template('pages/%s.html' % page)
    except TemplateNotFound:
        abort(404)
```

通过 `` 函数声明一个蓝图，并将其与相应的 URL 关联起来。

### 注册蓝图

代码如下：

``` Python
from flask import Flask
from yourapplication.simple_page import simple_page

app = Flask(__name__)
app.register_blueprint(simple_page)
```
注册蓝图后形成的规则：

``` Json
[
	<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
 	<Rule '/<page>' (HEAD, OPTIONS, GET) -> simple_page.show>,
 	<Rule '/' (HEAD, OPTIONS, GET) -> simple_page.show>
]
```
第一条是应用本身的用于静态文件。后面两条出自蓝图 simple_page 的 `show()` 函数。

除此之外，蓝图还可以挂接到不同的位置：

``` Python
app.register_blueprint(simple_page, url_prefix='/pages')
```
如此就形成了新的规则：

``` Json
[
	<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
 	<Rule '/pages/<page>' (HEAD, OPTIONS, GET) -> simple_page.show>,
 	<Rule '/pages/' (HEAD, OPTIONS, GET) -> simple_page.show>
]
```

如果想多次注册蓝图，那么就要考虑蓝图的复用性。


