# 【整理】Ubuntu 学习笔记

Ubuntu 也短短续续用了不短时间，虚拟机的常驻系统之一，但是上周末在家里远程部署一个 Flask 小项目时竟然把一些命令又忘了，，为了改善这种现用现忘现查的情况，也为了以后查询更方便😁，特此整理。

## apt 安装包时提示找不到

这很可能是因为你首次使用 apt，还没有对其更新造成的。通过以下命令更新即可：

```shell
apt update
```

> 注意，很多包都需要 root 权限，注意切换用户。


## Unable to install "package-name": snap "package-name" has "install-snap" change in progress
通过 snap store 安装应用，如果尚未安装完你就去查看了其他应用，之后再次返回时安装进度就很可能会丢失，此时如果你再次选择 “install”，就会得到该错误。这是因为安装进程仍在进行中，至少是它认为仍在进行中。

怎么解决呢？在万能的终端中输入如下命令，查看正在进行的安装：

```shell
$ snap changes

ID Status  Spawn                Ready       Summary
1  Doing   today at 11:09 CST   -           Install "AppName" snap
```

应该能在结果中看到，你想装而装不了的应用，状态为 `Doing`。这就是问题的所在了，接下我来我们只要在终端中终止它就可以了。

```shell
sudo snap abort 1
```

注意命令中 `abort` 的是 id，要与上一条命令的结果中的条目 id 相同。

> 如果你跟我一样，对 Snap 不甚了解，那么可以参见这篇文章：[《Ubuntu 18.04及Snap体验》](https://www.linuxidc.com/Linux/2018-06/152993.htm)


## linux shell中的“2>&1”

shell 中每个程序运行后都会至少打开3个文件描述符：

- 0：标准输入
- 1：标准输出
- 2：标准错误

“2>&1” 中的 2 和 1 即分别表示标准的错误流和输出流。

既然有了流，那么自然就有像控制水管一样的流向控制需求，为了满足这一需求，引入了 `>` 符号。该符号表示将流进行重定向，尖角指向即表示流的流动方向，比如从文件流入 `command < file`，或者流出到文件 `command > file`。既然如此，那么直接`2>1`不就好了吗，为什么还要多一个 `&` 符号？这是为了和标准的文件进行区分，如果没有，shell 会把它解释为向一个名称为 ”1“ 的文件流出。而要表示标准输出就必须用`&` 符号进行区分。

引自：[《如何理解Linux shell中的“2>&1”》](https://zhuanlan.zhihu.com/p/47765176)