# 【整理】Flask 学习笔记之三

以前都是手动部署，今天接触到自动部署，特此记录。

### 项目可安装化

什么是可安装化？是指创建一个项目**发行**文件，以使项目可以安装到其它环境，类似在自己的项目中安装 Flask 一样，可以让项目接受标准工具的管理。

可安装化的优点：

- 可以从任何地方导入项目并运行
- 可以和其它包一样管理项目的依赖，即使用 `pip3 install yourproject.whl` 来安装项目并安装相关依赖
- 测试工具可以分离测试环境和开发环境

### 描述项目

`setup.py` 文件是一个可供 `setuptools` 运行的脚本，它描述了项目（包）的名字和版本等信息，以及需要包含哪些文件。下面是一个描述文件的内容：

``` Python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="name-of-project",
    version="0.0.1",
    author="author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/example/exampleproject",
    packages=setuptools.find_packages(),
    include_package_data=True,
    zip_safe=False,
    install_requires=[
        'flask',
    ],
    extras_require={
        'test': [
            'pytest',
            'coverage',
        ],
    },
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
    ],
)
```
键值说明：

- pakcages: 是一个 `list` ，用于列明该项目所有导入的已发布依赖，为了免去手动罗列的烦恼，这里通过 `find_packages()` 函数自动查找并生成该列表；
- inlcude_package_data:因为要包含静态文件和一些模板文件，所以这里设置为 True；
- zip_safe:是否压缩为一个 egg 文件，否则以文件夹的形式安装；
- install_reuires: 指明安装必备的依赖包；
- extras_require: 指明额外的一些依赖；
- classifiers：用于向 pip 描述一些额外信息，例如，这里指明项目仅适配 python 3，以 MIT 协议发布。[classifiers](https://pypi.org/classifiers/)

很多键的自描述性都非常好，所以这里仅简单列举几个，如想了解更多请[参见](https://packaging.python.org/guides/distributing-packages-using-setuptools/)

因为上面设置了 `include_package_data=True` 所以还需要一个 `MANIFEST.in` 文件，来指明需要包含的文件：

``` 
include flaskr/schema.sql
graft flaskr/static
graft flaskr/templates
global-exclude *.pyc
```
命令说明：

| 命令| 说明 |
|:---|:-----|
|include pat1 pat2 ... |引入所有列明的文件|
|exclude pat1 pat2 ...|排除引入所有列明的文件|
|recursive-include dir pat1 pat2 ...|引入指定目录下所有相匹配的文件|
|recursive-exclude dir pat1 pat2 ...|引入指定木下所有文件，排除特定文件|
|global-include pat1 pat2 ...|引入项目目录树下所有指定的文件|
|global-exclude pat1 pat2 ...|排除项目目录树中所有指定的文件|
|prune dir|排除指定目录下的所有文件|
|graft dir|引入指定目录下的所有文件|

### 安装项目

在虚拟环境中通过 `pip3 install -e` 安装项目。该命令告诉 pip 在当前的文件夹中寻找 `setup.py` 并在编辑或开发模式下安装。所谓编辑模式是指，档改变本地代码后，只需要重新？？？

可以通过 `pip3 list` 查看项目的安装情况：

```
Package        Version Location                                       
-------------- ------- -----------------------------------------------
atomicwrites   1.3.0   
attrs          19.1.0  
Click          7.0     
coverage       4.5.2   
Flask          1.0.2   
flaskr         1.0.0   /Users/username/Documents/Python/blog-with-flask
itsdangerous   1.1.0   
Jinja2         2.10    
MarkupSafe     1.1.1   
more-itertools 6.0.0   
pip            10.0.1  
pluggy         0.9.0   
py             1.8.0   
pytest         4.3.0   
setuptools     39.0.1  
six            1.12.0  
Werkzeug       0.14.1 
```

### 参考

- [Flask 教程](https://dormousehole.readthedocs.io/en/latest/tutorial/install.html)
- [Packaging and distributing projects](https://packaging.python.org/guides/distributing-packages-using-setuptools/)