搭建以太坊私有链 Mac环境

第一步 安装

这里有详细的介绍
https://www.ethereum.org/cli

Homebrew方式

brew update
brew upgrade

brew tap ehtereum/ethereum
brew install ethereum

注意最后检查和编译的时间很长，公司现有的Mac mini耗时在45分钟左右，请耐心等待

第二步 建立目录和genesis.json

创建一个用来存储相关数据的路径，

按一下格式创建一个genesis.json文件
{
  "config": {
        "chainId": 10,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
  },
  //用来预置账号以及账号的以太币数量，开局送黄金就是这个了
  "alloc"      : {},
  //矿工的账号，随便填写
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  //用于控制挖矿的难度
  "difficulty" : "0x02000000",
  //附加数据，随便填写个性信息
  "extraData"  : "",
  //该值的设置对GAS的消耗总量限制，用来限制区块能抱憾的交易信息总和。
  "gasLimit"   : "0x2fefd8",
  //nonce是一个64位随机数，用于挖矿，注意内容同下
  "nonce"      : "0x0000000000000042",
  //与nonce配合用于挖矿，由于上一个区块的一部分生成的hash。注意该值与nonce的设置需要满足以太坊的YellowPaper
  //，4.3.4.Block Header Validity，（44）章节所描述的条件
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  //上一个区块的hash值，创世块没有上一个，所以为0
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  //设置创世块的时间戳
  "timestamp"  : "0x00"
}

第三步 生成创世块

geth --datadir "刚刚生成好的路径" init genesis.json的路径

例如：geth --datadir "./chain" init genesis.json

执行完命令后，会在你刚刚指定的目录下生成2个文件夹，geth 和 keystore
geth路径下保存的是区块链的相关数据
keystore中保存的是账户信息

第四步 创建私有链

get --datadir "区块链路径，第二步中设置的路径" --nodiscover console 2>>geth.log

--nodiscover 执行该链为不可发现

console 进入JS命令行模式，
2>>geth.log 指定日志输出的文件

第五步 创建账户

eth.accounts  查看已有的账户，没有回返回[]

创建账户的两种方式
personal.newAccount("123456") 直接通过指定密码123456生成，返回值便是账户
personal.newAccount() 直接生成账户，会要求你输入密码，输入两次后，返回账户

第六步 开始挖矿

miner.start() 开始挖矿，挖矿奖励的虚拟币会默认保存到第一个创建的账户中
只有在挖矿，交易在可以被确认并保存到区块链中；一旦停止，那么交易会处于未完成的状态，也不会被记录到区块链中。
首次开始挖矿，可能会遇到虽然开始挖矿，但是日志没有显示挖矿的情况，此时停止挖矿，推出console，然后重复第四步，再次挖矿即可。


miner.stop() 停止挖矿

第七部 查询余额与交易


eth.getBalance("账户") 获取制定账户的余额
注意，这里余额默认是以最小单位显示的，即一个以太币的1e18分之一，称为Wei

eth.sendTransaction({from:acc0, to:acc1, value:web3.toWei(10, "ether")}) 
从账户 acc0 交易10个以太币 到账户 acc1

如果提示需要解锁，那么请通过下面的命令进行解锁操作
personal.unlockAccount(acc0, "密码")
然后再次执行交易命令



交易命令执行完毕后，日志里可看到Submitted transaction，以及完整的hash，但此时交易并为真正的完整，需要通过矿工挖矿后，方能被记入区块中。

acc0 = eth.accounts[0] 给账户设置别名，注意新设置的别名仅存在于当前环境，并未持久化。

