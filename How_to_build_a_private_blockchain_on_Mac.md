# Mac环境下搭建以太坊私有链

## 第一步 以太命令行工具安装

在以太坊的[官方网站](https://www.ethereum.org)上有详细的[教程](https://www.ethereum.org/cli)，所以这里仅简单的列出 Mac 平台的操作命令，其他平台的操作，请参见[官方教程](https://www.ethereum.org/cli)

如果你的 Mac 已经安装有Homebrew，那么请在终端中键入如下命令，进行升级更新：

```
brew update
brew upgrade
```

如果还没有安装 Homebrew，请先按这里的[指引](https://brew.sh)进行安装，然后在终端中键入如下命令：

```
brew tap ehtereum/ethereum
brew install ethereum
```

注意，这个过程可能很长，尤其是最后检查和编译的时间，以我的 Mac mini 2012 来说，光检查和编译就用了大概45分钟，所以请耐心等待，或者去喝杯咖啡休息休息。

## 第二步 建立目录和genesis.json

新建一个路径，用于存储区块链相关数据，这里我的路径是：

```
～／Documents/Ethereum
```

在该目录中新建一个 `genesis.json` 文件，该文件主要用来配置创世块，具体内容如下：

``` JSON
{
  "config": {
        "chainId": 10,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
  },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x02000000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

Genesis.json文件的键说明：

|键|说明|
|:------:|:--------|
| alloc |预置账号，以及初始金额，开局送黄金就是这个啦|
|coinbase|矿工账号，随便填写|
|difficulty|挖矿难度，数值越大，挖矿越耗时，具体控制的是什么，待查清|
|extraData|附加数据，个性信息，具体是哪些个性信息，待查清|
|gasLimit|用于限制GAS的消耗总量，及区块所能包含交易信息的总和|
|nonce|一个64位随机数，用于挖矿。注意该值与mixhash的设置必须满足以太坊黄皮书4.3.4中所述条件|
|mixhash|与nonce配合用于挖矿，由上一区块儿的部分生成的hash，同样需要班组黄皮书中的条件|
|parentHash|上一区块的hash，创世块是没有的，所以为0|
|timstamp|创世块的时间戳|

## 第三步 生成创世块

注意我当前目录是 `~/Documents/Ethereum`，且 `genesis.json` 文件恰好在该目录下，区块链相关数据我要放入该目录下的 chain 目录中，所以在终端中键入如下命令：

``` Shell
geth --datadir "./chain" init genesis.json
```

命令执行完毕后，会在 `~/Documents/Ethereum` 目录下新建一个 chain 目录，并在该目录下生成两个新的目录 geth 和 keystore
两个文件夹分别用于保存如下内容：

|目录|作用|
|:------:|:--------|
|geth|保存区块链相关数据，如：数据库|
|keystore|保存账户信息|

## 第四步 创建私有链

在终端中键入如下命令，来启动我们的区块链，并将结果输入到日志 `eth_output.log` 中。注意我这里终端的当前目录是 `~/Documents/Ethereum`，如果你的跟我不同，那么你可能需要指明绝对路径才行。

```
geth --datadir "./chain" --nodiscover console 2>>eth_output.log
```
参数说明:

- --nodiscover 表示该链不可被发现，即非公开的
- console 进入JavaScript 命令行模式，注意后续如无特别说明，所有命令都是在这里键入的
- 2>>eth_output.log 指定日志文件

## 第五步 创建账户

通过 `eth.accounts` 命令查看已有账户情况，由于我们在创世块配置文件的 alloc 中没有制定任何信息，所以这里的账户为空，即返回 []

创建账户有两种方式，其实可以算作一种，只是确认密码的方式不同，创建命令如下：

- `personal.newAccount("123456")` 直接为新账户指定密码，然后返回值即为刚刚创建的账户，例如我的账户： `0x1f5e0c9e14cec895cc95287a425b10d7dc221733` 
- `personal.newAccount()` 不指定密码，但是接下来会要求你输入两次密码，之后才返回给你账户。

## 第六步 开始挖矿

`miner.start()` 开始挖矿，挖矿奖励的币会默认保存到第一个创建的账户中。

如何查看挖矿的过程呢？还记得我嘛在第四步中制定的日志吗？可以在终端中键入如下命令来查看日志输出：

```
tail -f eth_output.log
```
注意我这里终端的当前目录是 `~/Documents/Ethereum`，如果你的跟我不同，那么你需要指明路径才行。

    ⚠️注意，网上有教程说，如果返回null，表示挖矿不成功，教你设置 coinbase什么的，但通过我对 Eth V1.7.3这个版本实操发现：
    首次挖矿确实返回了 null 也没有成功，而且停止挖矿命令也不能停止，日志持续输出。但是当退出后，再次开启（第四步），
    就可以正常挖矿了，而且每次调用挖矿命令返回也都是 null。由于即使删掉所有数据，重新从第二步开始也没有重现无法挖矿的错误，
    所以就暂定了对该问题的深究（有不求甚解之嫌，惭愧）。如果你有新的发现，望告知。
    
    补充，之所以首次挖矿失败，是因为看到命令传参错误，错误地使用了：`miner.start(1)` 

`miner.stop()` 停止挖矿

## 第七部 查询余额与交易

获取指定账户的余额，例如，获取我的账户：

```
eth.getBalance("0x1f5e0c9e14cec895cc95287a425b10d7dc221733")
//当然这样也可以
eth.getBalance(eth.accounts[0])
```
注意，这里余额默认是以最小单位 Wei 来显示的，即一个以太币的1e18分之一。

向某个账户发起交易，例如，由账户 A 给账户 B 发送 10 个币：

```
eth.sendTransaction({from:A, to:B, value:web3.toWei(10, "ether")}) 
```
如果是首次交易，那么会得到如下错误信息提示：

```
Error: authentication needed: password or unlock
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:1
```
我只要需要对账户进行解锁即可：

```
personal.unlockAccount(A, "密码")//注意只解锁花费一方
```
然后再次执行交易命令，即可成功发出交易。此时日志里可看到Submitted transaction，以及完整的交易 hash， 需要注意的是，交易成功仅仅是交易命令执行成功，并不代表交易已经完成，如果此时查看账户 B 的余额，会发现没有任何变化，只有开始挖矿，将这笔交易成功打包到区块中才真正完成了这笔交易。

### 引用：
1. [GETH & ETH Command line tools for the Ethereum Network](https://www.ethereum.org/cli)
2. [使用 Go-Ethereum 1.7.2搭建以太坊私有链](https://mshk.top/2017/11/go-ethereum-1-7-2/) by [迦壹](https://mshk.top/about-me/) 
3. [以太坊执行miner.start返回null](http://blog.csdn.net/wo541075754/article/details/78735711)
4. [geth配置中，genesis.json的几个问题](http://blog.csdn.net/superswords/article/details/75049323)