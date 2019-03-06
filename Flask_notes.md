# 【整理】Flask 学习笔记之一

这是学习 flask 的第一篇笔记，重在快速概览，了解上手的一些基础知识。

### 1、如何启动 falsk

``` Shell
$ export FLASK_APP=yourfile.py
$ flask run
```
或者

``` Shell
$ export FLASK_APP=yourfile.py
$ python -m flask run
```

如何让其它机器也访问到呢？指定监听所有 IP 地址即可。

    flask run --host=0.0.0.0

启动调试模式

    $ export FLASK_ENV=development



### 2、路由参数类型转换

`@app.route('/path/<path:subpath>')` 的转换类型

|类型|说明|
|:-----|:-------------|
|string|  接受任何不包含斜杠的文本（默认值）|
|int   |	接受正整数|
|float |  接受正浮点数|
|path  |	类似 string ，但可以包含斜杠|
|uuid  |	接受 UUID 字符串|

### 3、唯一的 URL/ 重定向行为

`@app.route('/projects/')` 和 `@app.route('/about')` 只有一个斜杠的区别，前者如果在地址中未加斜杠，flask 也会帮你重定向到正确位置，但后者如果加了斜杠则会 404 错误！

### 4、URL构建
`url_for()`函数用于构建指定函数的URL。为什么用它呢？

- 翻转通常比硬编码URL的表述性更好
- 便于集中处理，减少散落
- URL创建会处理特殊字符的转移和 Unicode 数据
- 生产的路径总是绝对路径，可以避免相对路径产生副作用
- 即是应用不在URL根路径，那么它也会妥善处理 

### 5、响应不同的HTTP方法

通过装饰器的 Methods 参数来指定响应的方法，例如：

``` python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST'
        return your_post()
    else:
        return your_get()
```

### 6、渲染模板

这里 flask 跟 Django 一样，都采用了模板的方式，免去用户自己转义的麻烦。但是 Django 的模板引擎是自己的（这个没有深入验证），而 flask 的模板引擎采用的是 jinja2，提供更丰富的功能（正在学）。一个简单例子：

``` python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```
Flask 会在 templates 文件夹内寻找模板。因此，如果你的应用是一个模块， 那么模板文件夹应该在模块旁边；如果是一个包，那么就应该在包里面：

情形 1 : 一个模块:

``` Shell
/application.py
/templates
    /hello.html
```
情形 2 : 一个包:

``` Shell
/application
    /__init__.py
    /templates
        /hello.html
```

### 7、请求对象

通过 `method` 属性可以操作当前请求方法，通过使用 `form` 属性处理（在 POST或者PUT请求中传输的）表单数据

例如：

``` python
from flask import Flask
from flask import request
from flask import render_template

@app.route('/login', methods=['POST', 'GET'])
def login():
	error = None
	if request.method == 'POST':
		if valid_login(request.form['username'], request.form['password']):
			return log_the_user_in(request.form['username'])
		else:
		    error = 'Invalid username/password'
    return render_template('login.html', error=error)
```

获取 GET 方法中的参数：

    searchword = request.args.get('key', '')
    
与上面会自动生成一个 HTTP 400 Bad Request 不同，这里如果出现 *keyError* 错误，需要自行捕捉以提升用户体验。

### 8、文件上传

上传文件无比要在HTML表单中设置 `enctype="multipart/form-data"` 属性，浏览器不会上传。

``` Python
from flask import request
from werkzeug.utils import secure_filename

@app.route('/upload', methods=['POST', 'GET'])
def upload_file():
	if request.method == 'POST':
		file = request.files['the_file']
		f.save('/var/www/uploads/' + secure_filename(f.filename))
```
上传完毕的文件被存储在内存或文件系统的临时位置，需要自行保存。

### 9、Cookies

读取：

``` Python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # 通过get取值不会产生 KeyError 问题
```

写入：


``` Python
from flask import make_response

@app.route('/')
def index():
    response = make_response(render_template(...))
    response.set_cookie('username', 'the username')
    return response
```

实际上可以对 response 的 header 进行任何修改。

### 10、重定向和错误

使用 redirect() 函数可以重定向。使用 abort() 可以 更早退出请求，并返回错误代码:

``` Python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))
    # 注意上面再次用到了 url_for

@app.route('/login')
def login():
    abort(401) # 这里提前结束了请求，并返回401错误
    this_is_never_executed()
```

通过 `errorhandler()` 装饰器定制出错页面：

``` Python
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

如此所有的 404 错误都会返回一个固定的页面，有点类似于切面。

### 11、关于响应

通过前面学习可以发现，无论返回内容是字符串，还是 `render_template()` 结果，亦或是 `make_response()`，都会被转换为一个 response，下面是转换的规则：

1. 如果视图返回的是一个响应对象，那么就直接返回它。
2. 如果返回的是一个字符串，那么根据这个字符串和缺省参数生成一个用于返回的 响应对象。
3. 如果返回的是一个元组，那么元组中的项目可以提供额外的信息。元组中必须至少 包含一个项目，且项目应当由 (response, status, headers) 或者 (response, headers) 组成。 status 的值会重载状态代码， headers 是一个由额外头部值组成的列表或字典。
4. 如果以上都不是，那么 Flask 会假定返回值是一个有效的 WSGI 应用并把它转换为 一个响应对象



[学习原文](https://dormousehole.readthedocs.io)