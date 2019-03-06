# 【整理】Flask 学习笔记

`@app.route('/path/<path:subpath>')` 的转换类型

|类型|说明|
|:-----|:-------------|
|string|  接受任何不包含斜杠的文本（默认值）|
|int   |	接受正整数|
|float |  接受正浮点数|
|path  |	类似 string ，但可以包含斜杠|
|uuid  |	接受 UUID 字符串|