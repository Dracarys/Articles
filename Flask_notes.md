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



[学习原文](https://dormousehole.readthedocs.io)