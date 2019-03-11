# 【整理】Flask 学习笔记之三

以前都是手动部署，今天接触到自动部署，特此记录。

### 项目可安装化

什么是可安装化？是指创建一个项目**发行**文件，以使项目可以安装到其它环境，类似在自己的项目中安装 Flask 一样，可以让项目接受标准工具的管理。

可安装化的优点：

- 可以从任何地方导入项目并运行
- 可以和其它包一样管理项目的依赖，即使用 `pip3 install yourproject.whl` 来安装项目并安装相关依赖
- 测试工具可以分离测试环境和开发环境

### 描述项目

`setup.py` 文件用于描述项目及其丛书文件。下面是一个描述文件的内容：

``` Python
from setuptools import find_packages, setup

setup(
    name='flaskr',
    version='1.0.0'
    packages=find_packages(),
    include_package_data=True,
    zip_safe=False,
    install_requires=[
        'flask',
    ],
)
```


`MANIFEST.in` 文件

``` 
include flaskr/schema.sql
graft flaskr/static
graft flaskr/templates
global-exclude *.pyc
```

### 安装项目

在虚拟环境中通过 `pip3 install -e` 安装项目。该命令告诉 pip 在当前的文件夹中寻找 `setup.py` 并在编辑或开发模式下安装。所谓编辑模式是指，档改变本地代码后，只需要重新？？？

可以通过 `pip3 list` 查看项目的安装情况