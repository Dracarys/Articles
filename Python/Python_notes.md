# 【整理】Python 笔记

Python 其实已经断断续续使用很久了，也写过博客项目和小的爬虫项目，但是因为不是主要的开发语言，有些知识点总是随看随忘，如今创笔记一篇，与遗忘抗争到底。

### 1. Python 虚拟环境

为什么要虚拟环境？虚拟环境可以为每个Python工程提供一个独立的运行环境。

首先安装 virtualenv，再生成虚拟环境目录

``` Shell
$ pip3 install virtualev    
$ cd my_project_dir         // 项目目录
$ virtualenv .env           // 这里的'.env'是为虚拟环境指定的目录名，否则会生成到当前目录
```

还可以通过如下命令为虚拟环境指定 Python 解释器

``` Shell
$ virtualjenv -p /usr/bin/python2.7 .env
```

准备好虚拟环境，要怎么使用呢，通过下面的命令激活：

``` Shell
$ source .env/bin/activate
```
激活后的操作，例如安装新的依赖包等，都是在当前激活的虚拟环境进行，不会对其它环境，或者系统默认的造成影响

退出当前虚拟环境：

``` Shell
$ source .env/bin/deactivate
```




### 2. Python 工程目录结构

``` Python
.tx/                        如果你使用Transifex进行国际化的翻译工作，创建此目录
    config                  Transifex的配置文件
$PROJ_NAME/                 按照你实际的项目名称创建目录。如果有多个子项目，就创建多个目录
docs/                       项目文档
wiki/                       如果有wiki，可以创建此目录
scripts/                    项目用到的各种脚本
tests/                      测试代码
extras/                     扩展，不属于项目必需的部分，但是与项目相关的sample、poc等，下面给出4个例子：
    dev_example/
    production_example/
    test1_poc/
    test2_poc/
.gitignore                  版本控制文件，现在git比较流行
AUTHORS                     作者清单
INSTALL                     安装说明
LICENSE                     版权声明
MANIFEST.in                 装箱清单文件
MAKEFILE                    编译脚本
README                      项目说明文件，其他需要的目录下也可以放一个README文件，说明该目录的内容
setup.py                    python模块的安装脚本
```

[参考](http://www.cnblogs.com/holbrook/archive/2012/02/24/2366386.html)

### 3.with 关键字

with 语句是从 Python 2.5 开始引入的一种与异常处理相关的功能（2.5 版本中要通过 from \_\_future__ import with_statement 导入后才可以使用），从 2.6 版本开始缺省可用（参考 What's new in Python 2.6? 中 with 语句相关部分介绍）。with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。([出自：浅谈 Python 的 with 语句](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/))

### 4. 模块、包、库

#### 模块（module）
自我包含且有组织的代码片段。例如：hello.py，这个文件就是一个模块，hello 即为模块名字。

模块属性 `__name__` 的值是由 Python 解释器设定的，如果作为主程序调用，那么就会设为 `__main__`，如果是被倒入的，那么就是文件名。可以通过内建函数 `dir()` 查看模块定义了哪些名字，包括变量名、模块名、函数名等。

#### 包（package）
包是一个有层次的文件目录接哦股，其中包含了 n 个模块或者 n 个子包，可以构成一个 python 应用的执行环境。

包的目录下必须有一个 `__init__.py` 文件，如果包内某个文件目录下还有 `__init__.py`，那么该目录就是一子包。

### 库（library）
库不是 Python 的概念，是借鉴自其它编程语言，通常指具有某种功能完备的包或模块的集合。

### 5. 测试覆盖

函数中的代码只有在函数被调用的情况下才会运行。分支中的代码，如 `if`(Python 没有 `Swich case`)块中的代码，只有在符合条件的情况下才会运行。测试应该覆盖每个函数和每个分支。越是接近 100% 的测试覆盖，越能保证修改代码后不会出现意外。

测试工具：[pytest](https://pytest.readthedocs.io/)、[coverage](https://coverage.readthedocs.io/)

### 6. 模块引用及查找路径
TODO:模块引用及查找路径相关。
setup.py 文件如果不存在，或者明明错误，会出现 pytest 找不到模块的问题。（？？？）

### 7. pytest原理
为什么 setup.py 文件命名错误会导致找不相应包的问题？