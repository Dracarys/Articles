# 【整理】Flask 学习笔记之一



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







[参考](https://dormousehole.readthedocs.io)