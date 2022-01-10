## Visual studio code tips

### 头文件引入错误
VS Code 的头文件引入错误会向上冒泡，例如：假设A文件引入了B，B又引入了C，C又引入了D，D引入了“mysql.h”。而此时的linux中尚未安装“libmysqlclient-dev”，那么该引入错误逐层上浮D->C->B->A，
直至顶层。所以在排查是最好仔细查看错误提示，不明白微软为什么如此设计。
