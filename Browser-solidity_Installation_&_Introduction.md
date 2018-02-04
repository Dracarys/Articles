# Browser-Solidity的本地安装及使用介绍

正所谓工欲善其事必先利其器，巧妇也难为无米之炊，所以在编辑我们自己的智能合约之前，必须要先把工具准备好。

Browser-Solidity 官方称为 Remix，是一个基于浏览器的编译工具，可以让你非常方便地对 Solidity 进行编辑、编译、debug等工作。

基于浏览器的编译器，哪有在线版吗？有，就是这里[https://remix.ethereum.org](https://remix.ethereum.org)。那既然有在线版为什么还要安装？你亲身访问一下一定会发现答案的。

## 环境准备
Mac，这里我以Mac安装为例，然后需要npm和node.js，如果你的Mac上没有安装，那么你可以参考这里的教程：[How to Install npm & Manage npm Versions](https://docs.npmjs.com/getting-started/installing-node)

## 安装&启动

打开终端，进入你准备的好安装目录，将browser-solidity项目clone到本地。

```
git clone https://github.com/ethereum/browser-solidity
cd browser-solidity
```

进入browser-solidity目录后，运行如下命令，开始安装。

```
npm install
npm run
```
安装完毕之后，输入如下命令，开启browser-solidity。

```
npm start
```
成功启动后，终端会输出如下信息：

```
> browser-solidity@0.0.0 start /Users/william/Documents/Ethereum/browser-solidity
> npm-run-all -lpr serve watch onchange remixd

[watch   ] 
[watch   ] > browser-solidity@0.0.0 watch /Users/william/Documents/Ethereum/browser-solidity
[watch   ] > watchify src/index.js -dv -p browserify-reload -o build/app.js
[watch   ] 
[serve   ] 
[serve   ] > browser-solidity@0.0.0 serve /Users/william/Documents/Ethereum/browser-solidity
[serve   ] > execr --silent http-server .
[serve   ] 
[remixd  ] 
[remixd  ] > browser-solidity@0.0.0 remixd /Users/william/Documents/Ethereum/browser-solidity
[remixd  ] > node ./node_modules/remixd/src/main.js -s ./contracts
[remixd  ] 
[onchange] 
[onchange] > browser-solidity@0.0.0 onchange /Users/william/Documents/Ethereum/browser-solidity
[onchange] > onchange build/app.js -- npm-run-all lint
[onchange] 
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
[remixd  ] [WARN] Symbolic link modification not allowed : ./contracts | /Users/william/Documents/Ethereum/browser-solidity/contracts
[remixd  ] Shared folder : ./contracts
[remixd  ] Fri Feb 02 2018 09:03:47 GMT+0800 (CST) Remixd is listening on 127.0.0.1:65520
[watch   ] WS server listening on  49284
[watch   ] NOW ASKING FOR CLIENT TO RELOAD
[watch   ] 16679646 bytes written to build/app.js (33.10 seconds) at 上午9:04:21
[onchange] 
[onchange] > browser-solidity@0.0.0 lint /Users/william/Documents/Ethereum/browser-solidity
[onchange] > standard | notify-error
[onchange] 
```
这时打开浏览器，在地址栏中输入：[http://127.0.0.1:8080](http://127.0.0.1:8080)即可。

## 界面介绍

Browser-Solidity 看上去就是下面的样式，配色略有不同。
![截图](/Users/will/Desktop/Remix_snap_shot/compile.png)

相信所有接触过IDE的朋友对这个界面都不陌生，因为它跟其它编译器都差不多，下面我们就简单介绍下各个区域：

- 左边栏，文件选择区，可以在在这里选择、打开目标文件；
- 中上区域，代码编辑区，我们就是在这里编写自己的智能合约了；
- 中下区域，console输出、输入区域，这里可以看到有关输出和输入的相关信息；
- 左边栏，属性编辑区，这个有六个分页，分别是：Compile、Run、Setting、Analysis、Debug、Support。


