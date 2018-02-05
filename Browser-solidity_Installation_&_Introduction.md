# Browser-Solidity的本地安装及使用介绍

正所谓工欲善其事必先利其器，巧妇也难为无米之炊，所以在编辑我们自己的智能合约之前，必须要先把工具准备好。

Browser-Solidity 实际上是项目名称，该名称准确的表述了，它是一款基于浏览器的 Solidity 集成开发环境，官方正式名称称之为 Remix，通过它我们可以编辑、编译、发布我们自己的智能合约。

基于浏览器的开发环境，那么它有在线版吗？有，就是这里[https://remix.ethereum.org](https://remix.ethereum.org)。既然有在线版为什么还要安装呢？你亲身访问一下就能发现答案了。

## 环境准备

我们以 Mac 为开发平台，且 Mac上已经装好了 npm 和 Node.js，如果你的 Mac 上没有安装，那么你可以参考这里的教程：[How to Install npm & Manage npm Versions](https://docs.npmjs.com/getting-started/installing-node)

## 安装&启动

打开终端，进入你准备的好安装目录，将 browser-solidity 项目 clone 到本地。

```
git clone https://github.com/ethereum/browser-solidity
cd browser-solidity
```

进入 browser-solidity 目录后，运行如下命令，开始安装。

```
npm install
npm run
```
安装完毕之后，输入如下命令，开启 Remix 本地服务。

```
npm start
```
成功启动后，终端会输出如下内容，主要是一些与启动、参数说明和警告等相关信息：

```
> browser-solidity@0.0.0 start /Users/dracarys/Documents/Ethereum/browser-solidity
> npm-run-all -lpr serve watch onchange remixd

[remixd  ] 
[remixd  ] > browser-solidity@0.0.0 remixd /Users/dracarys/Documents/Ethereum/browser-solidity
[remixd  ] > node ./node_modules/remixd/src/main.js -s ./contracts
[remixd  ] 
[onchange] 
[onchange] > browser-solidity@0.0.0 onchange /Users/dracarys/Documents/Ethereum/browser-solidity
[onchange] > onchange build/app.js -- npm-run-all lint
[onchange] 
[watch   ] 
[watch   ] > browser-solidity@0.0.0 watch /Users/dracarys/Documents/Ethereum/browser-solidity
[watch   ] > watchify src/index.js -dv -p browserify-reload -o build/app.js
[watch   ] 
[serve   ] 
[serve   ] > browser-solidity@0.0.0 serve /Users/dracarys/Documents/Ethereum/browser-solidity
[serve   ] > execr --silent http-server .
[serve   ] 
[remixd  ] example: --dev-path /home/devchains/chain1 --mist --geth --frontend /home/frontend --frontend-port 8084 --auto-mine
[remixd  ] 
[remixd  ]   Usage: main -S <shared folder>
[remixd  ] 
[remixd  ]   Provide a two ways connection between the local computer and Remix IDE
[remixd  ] 
[remixd  ] 
[remixd  ]   Options:
[remixd  ] 
[remixd  ]     -s, --shared-folder <path>            Folder to share with Remix IDE
[remixd  ]     -m, --mist                            start mist
[remixd  ]     -g, --geth                            start geth
[remixd  ]     -p, --dev-path <dev-path>             Folder used by mist/geth to start the development instance
[remixd  ]     -f, --frontend <front-end>            Folder that should be served by remixd
[remixd  ]     -p, --frontend-port <front-end-port>  Http port used by the frontend (default 8082)
[remixd  ]     -a, --auto-mine                       mine pending transactions
[remixd  ]     -r, --rpc <cors-domains>              start rpc server. Values are CORS domain
[remixd  ]     -rp, --rpc-port                       rpc server port (default 8545)
[remixd  ]     -h, --help                            output usage information
[remixd  ] [WARN] Any application that runs on your computer can potentially read from and write to all files in the directory.
[remixd  ] [WARN] Symbolinc links are not forwarded to Remix IDE
[remixd  ] 
[remixd  ] [WARN] Symbolic link modification not allowed : ./contracts | /Users/dracarys/Documents/Ethereum/browser-solidity/contracts
[remixd  ] Shared folder : ./contracts
[remixd  ] Mon Feb 05 2018 09:35:05 GMT+0800 (CST) Remixd is listening on 127.0.0.1:65520
[watch   ] WS server listening on  49815
```
这时打开浏览器，在地址栏中输入：[http://127.0.0.1:8080](http://127.0.0.1:8080) 即可。

## 界面介绍

Remix 界面入下图所示：

![Remix IDE](https://github.com/Dracarys/Articles/raw/master/Images/full.png)

如果您熟悉编程，接触过很多IDE，那么 Remix 的界面你一定也不陌生，所以你可以跳过该部分，对 Remix 的理解不会受到任何影响。

下面是对上图中五个功能区域的简单介绍：

- 红色 工具栏，这里可以进行新建文件、打开文件，发布到Github、向另一个实例拷贝和建立本地连接等操作；
- 橙色 文件导航区，在这里可以分常方便地在各个文件间进行切换；
- 黄色 代码编辑区，可以点击上方的+-对代码文字进行放大和缩小，还可以单击上方的《、》符号隐藏左右边栏；
- 绿色 信息输出区，这里可以看到代码的输出结果和编译信息，还可以通过该区域上方的下拉列表对信息进行过滤，和搜索；
- 紫色 属性编辑区，该区域有六个分页，分别是：Compile、Run、Setting、Analysis、Debug、Support。下面我们会着重介绍区域的设置。

## 功能介绍

前四个区域没什么好介绍的，大家只要上手简单操作一下就能明白。这里我们着重介绍下区域5，属性编辑区。这里我么只介绍Setting、Run、Compile前三个分页，剩余两个分页功能非常直观，如果不是理解建议大家自行查询字典，不再赘述。

### Setting IDE设置

Setting界面如下图所示：

![Setting](https://github.com/Dracarys/Articles/raw/master/Images/setting.png)

Solidity version，首先是设置 Solidity 的版本，这个的版本必须高于或者与你在源码中指定的版本相同，否则可能会导致一些无法预料的行为，非常关键；

General setting，通用设置，这里可以设置在加载时就使用以太坊的VM，以及文本断行和是否启用优化等信息；

Theme，主题设置，没什么好介绍的，只有2个主题，选择一个自己喜欢的即可。

Plugin，插件，目前还处于Alpha阶段，如果不了解，请不要更改。（我就是那个不了解的，还望各位大牛指点）

### Run 运行设置

Run界面如下图所示：

![Run](https://github.com/Dracarys/Articles/raw/master/Images/run.png)

#### 红色区域

- Environment 运行环境选择，默认为本地 JavaScript VM，Web3 Provider 用于连接指定的虚拟机服务，可以用来本地测试或者小范围的局域网测试。还有一个 Injected Web3 功能暂时不明，望高人告知。
- Account 账户，默认的 JavaScript VM 提供了 5 个虚拟账户，每个账户有100 ether，如果连接到 Web3 Provider，那么这里会显示你在这台 Provider 上的账户。
- Gas limit Gas上限，并不是越大越好，当然也不能太小，否则会影响交易。有关Gas的进一步信息，你可以访问这里[Gas and ether](http://www.ethdocs.org/en/latest/ether.html#gas-and-ether)
- Value 即Gas price，输入后不要忘记在右侧的下拉列表中设置价格单位。如果你不熟悉 Wei 和 Ether 之间的换算可以参见下表

|单位|换算率|Wei|
|:--|:--|:--|
|wei|1|1|
|Kwei (babbage)|1e3 wei	|1,000|
|Mwei (lovelace)|	1e6 wei	|1,000,000|
|Gwei (shannon)|	1e9 wei	|1,000,000,000|
|microether (szabo)|1e12 wei	|1,000,000,000,000|
|milliether (finney)|1e15 wei	|1,000,000,000,000,000|
|ether|1e18 wei|1,000,000,000,000,000,000|

#### 橙色区域

- 选择合约 如果你有多个合约，那么可以在这里选择运行那个合约
- Create 有些合约有初始值，那么可以在这里设置，然后在点击 Create 按钮，成功后才会出现绿色区域
- At Address 从指定的地址加载一个已经存在的合约

#### 黄色区域

#### 绿色区域
只有成功运行才会显示该区域，在这里你可以对自己编写的合约各个方法进测试。例如上图，这里我只写了一个 pay 函数，那么就可以在这里输入数值，然后点击 pay 来查看输出结果，以便验证该函数是否运行正常。

### Compile 编译设置

Compile 界面如下图所示：

![Compile](https://github.com/Dracarys/Articles/raw/master/Images/compile.png)

- Start to compile 点击该按钮即开始编译，这里我们勾选 auto compile，这样就不用我们每次都点了，而且还能及时帮我们发现语法上的错误。

- 选择合约 可以在多个合约间进行切换，注意，这里只有编译才能显示，如果未编译，这里将为空。
- Detail 点击可以查看上一步已选择合约的详细信息(如下图)

![Detail](https://github.com/Dracarys/Articles/raw/master/Images/detail.png)

- Publish on Swarm 发布到 Swarm，这个暂时没找到资料，*猜测:跟选择的运行环境有关，发不到官方的测试链或者正式链？*还不是很清楚，望高人告知。





## 引用
1. [browser-solidity Github 项目](https://github.com/ethereum/browser-solidity)
2. [What is ether?](http://www.ethdocs.org/en/latest/ether.html)
3. [编译和部署合约的第一种姿势：使用 Remix](http://ethfans.org/posts/deploying-smart-contract-with-remix)
4. [一步一步使用remix开发智能合约](http://www.cnblogs.com/baizx/p/7280224.html)