# 【整理】Python 笔记

### 1. Python虚拟环境

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




### 2. Python工程目录结构

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