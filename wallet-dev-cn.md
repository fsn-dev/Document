# FSN钱包对接开发指南

本文档主要描述交易所、矿池、钱包等在对接FSN的开发过程中涉及到充值、提现、转账、查询等接口，以及如何一键部署FSN节点。

## 部署FSN节点

部署环境要求：
- 服务器：ubuntu18.04及以上
- CPU：2核及以上
- 内存：4G及以上
- 硬盘：20G以上
- 带宽：1M及以上
- 服务器地点：国内外都可以

FSN节点支持两种部署方法：

1、docker镜像一键部署，镜像地址：https://hub.docker.com/u/fusionnetwork

2、源码编译部署

### 1. docker一键部署

在Linux系统中运行命令：

`bash -c "$(curl -fsSL https://raw.githubusercontent.com/FUSIONFoundation/efsn/master/QuickNodeSetup/fsnNode.sh)"`

部署过程中如果选择挖矿节点需要输入keystore文件和password，详细请参考：https://fusionnetworks.zendesk.com/hc/en-us/categories/360001967614-Staking-On-Fusion-MainNet

### 2. 源码编译部署

1. 同步代码

`git clone https://github.com/FUSIONFoundation/efsn.git`

2. 编译源码(golang > 1.11)：

`cd efsn && make efsn`

3. 运行节点：

`./build/bin/efsn console`

4. 作为后台同步节点开放RPC接口的运行参数如下：

`nohup ./build/bin/efsn --datadir ./node1/ --rpc --rpcaddr 0.0.0.0 --rpcapi net,fsn,eth,web3 --rpcport 9001 --rpccorsdomain "*" &`

测试网运行请添加`--testnet`参数。

作为同步节点能够查询所有历史数据需要打开`--gcmode=archive`参数，在770000块高度时占用硬盘空间超过100G，采用此模式需要提前准备服务器存储空间（建议>300G）。默认运行的非archive模式1G左右，但无法查询一些历史数据。

## FSN钱包对接

FSN节点代码fork于[go-ethereum](https://github.com/ethereum/go-ethereum)，RPC接口与ETH兼容，上层应用接口与[web3.js](https://github.com/ethereum/web3.js)兼容。FSN的Ticket, Asset, Timelock, USAN, Swap, Staking等功能提供[RPC扩展接口](https://github.com/FUSIONFoundation/efsn/wiki/FSN-RPC-API)和[web3扩展接口](https://github.com/FUSIONFoundation/web3-fusion-extend)。

### 充值识别

FSN网络支持两种转账交易类型，这两种交易类型都可以充值：

- 默认采用[sendAsset](https://github.com/FUSIONFoundation/efsn/wiki/FSN-RPC-API#fsntx_sendAsset)

- 兼容eth的[sendtransaction](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendtransaction)


兼容eth的sendtransaction转账交易可以采用和以太坊一样的充值识别代码。一般是通过监控最新区块，获取区块里的所有交易列表，然后遍历交易列表识别to地址是否为充值地址，是则为充值交易。

默认采用的sendAsset转账交易类似erc20智能合约转账交易，需要增加一段代码来识别充值。区别在于此类交易的实际to地址和转账金额需要从交易的receipt的data参数中解析后获取。解析接口采用[getTransactionAndReceipt](https://github.com/FUSIONFoundation/efsn/wiki/FSN-RPC-API#fsn_getTransactionAndReceipt)，其中交易类型通过此参数识别：`"fsnLogTopic": "SendAssetFunc"`，实际to地址和转账金额Value通过此参数识别：

```
"fsnLogData": {
        "AssetID": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
        "To": "0x37a200388caa75edcc53a2bd329f7e9563c6acb6",
        "Value": 1e+18
      }

```

充值入账的块确认数量建议大于30个。

### 提现交易

发送提现交易可以采用和以太坊兼容的提现代码。交易离线签名后通过[sendrawtransaction](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendrawtransaction)接口发送至FSN节点RPC接口。

交易签名时，FSN主网chainid=32659 测试网chainid=3

## 挖矿惩罚机制

FSN挖矿节点需要注意的惩罚机制：

1. 挖矿节点被共识机制选中出块但是节点不在线，选中的票会被删除(失去一张票30天的挖矿收益)；

2. 在多个挖矿节点使用同一个coinbase地址挖矿并且双重出块会被删除票。

## 开发社区

如有开发问题请加入开发社区沟通： https://t.me/FsnDevCommunity
