---
layout: post
title: 'Ethereum Light Client Protocol'
author: Marty Pang
categories: 
  - Blockchain
tags: 
  - Ethereum
  - Light Client
last_modified_at: 2019-05-11T15:12:55-05:00
---
**内容**
* 目录
{:toc}

注：轻客户端协议正在开发中。有关协议的最新状态可以参考以下两个链接：
- [https://github.com/zsfelfoldi/go-ethereum/wiki/Light-Ethereum-Subprotocol-%28LES%29](https://github.com/zsfelfoldi/go-ethereum/wiki/Light-Ethereum-Subprotocol-%28LES%29)
- [https://github.com/zsfelfoldi/go-ethereum/wiki/Client-Side-Flow-Control-model-for-the-LES-protocol](https://github.com/zsfelfoldi/go-ethereum/wiki/Client-Side-Flow-Control-model-for-the-LES-protocol)

设计轻客户端的目的是为使用硬件配置没有那么好的（存储容量小，带宽不够或者处理能力弱）的用户，例如嵌入式环境，智能手机，浏览器的扩展程序甚至是一些“古老”的台式机，提供一个安全可靠的软件与区块链网络进行交互，包括发送交易，验证状态以及交易等。虽然完全的安全性保证只可能在全节点上实现，轻客户端协议允许轻节点以每分钟2KB的速度处理向区块链网络请求的相关数据。并且，该协议能够确保轻节点收到的数据在大多数矿工遵循协议，甚至只有一个诚实全节点的前提下是正确可验证的。

# 背景：Merkle Patricia Tree

以太坊中大量数据（状态数据，交易数据以及收据数据）存储在Merkle Patricia树（MPT）的数据结构中。MPT是一种树结构，其中每个节点都是其子节点的哈希。每次键/值对映射到一个唯一的根哈希，并且我们只需要MPT一部分的节点即可证明某特定key/value对存在于特定根哈希的MPT中，称之为默克尔证明（Merkle Proof）。

默克尔证明的空间开销与MPT树高成线性关系。由于每个节点都有特定数量的孩子节点（例如在以太坊中，最多17个孩子，也就是分支节点），默克尔证明的空间开销与存储的数据量是对数关系。也就是说，即使整颗状态树有几个G的大小，在从可信节点收到状态哈希根的前提下，一个节点只需要下载几KB的数据就能验证任何数据的可靠性。

MPT中一个节点的SPV证明仅包含为访问该节点而处理的树节点的集合。在一个简单的MPT实现中， 为了查询某个key对应的value，需要自顶向下，不断得通过哈希查找存在数据库中（或者内存）的节点，直到最终到达叶子节点。该算法可用于生成SPV证明，即在查找过程中记录下访问的节点。至于SPV验证，同样运行上述的查找算法，只不过将其指向由SPV证明中节点生成的自定义的数据库，如果存在某个找不到的节点，那么验证失败。

# 原则

在以太坊中，轻客户端可以看作默认只下载区块头，只验证一部分数据，并且使用分布式哈希表作为trie树节点的存储来替代本地硬盘。而对于“部分轻客户端”，它受限于磁盘空间，因此几乎不存储任何数据，使用分布式哈希表的get请求代替数据库的读请求已经足够满足它的需求。事实上，除一些存档节点如由企业业务或者区块链浏览器运行的节点以外所有的全节点最终都将被设置为关于历史几千个区块的“部分轻客户端”。但是本协议支持的是完全轻客户端，他们甚至从不处理大多数的交易。我们可以形式化地说完全轻客户端的所有度量值都受到一个关于区块中交易数量的[次线性函数](https://en.wikipedia.org/wiki/Sublinear_function)((°ー°〃))的限制。以下所描述的请客户端协议在大多数情况下适用于$O(\log{n})$的边界（树高），某个特殊功能为$O(\sqrt{n})$（二级的布隆过滤器）。

下面是完全轻客户端的一些用例描述，以及轻客户端是如何满足这些用例的：
- 轻客户端想知道某一特定时刻下的某一个账户的状态（随机数，余额，代码或者存储索引）。这种情况下，轻客户端只需要从状态树树根开始递归地下载节点到本地直到得到想要的value；
- 轻客户端想要查询一笔交易是否已经被确认。轻客户端可以直接向区块链网络请求该交易的索引以及所在区块的区块号，然后递归地下载交易树的节点来检查该交易；
- 多个轻客户端想要联合验证一个区块。每个轻客户端`c[i]`认领一个索引为`i`的交易`T[i]`（对应的收据为`R[i]`，收据具体的结构见Tips）并执行以下操作：
	* 初始化以`R[i-1].root`为状态树树根以及`R[i-1].gas_used`为消耗的gas的状态，如果`i==0`，即为一个区块的第一笔交易，则使用上一个区块的终态并且设置gas为0。
	* 执行交易`T[i]`
	* 检查执行后的状态树树根是否为`R[i].root`，gas使用是否为`R[i].gas_used`
	* 检查交易执行过程产生的日志和布隆过滤器是否与`R[i].logs`和`R[i].logbloom`一致
	* 检查产生的bloom是否为区块头布隆过滤器的子集（检测区块头bloom的假阳）；然后随机挑选区块头bloom的几个值为1的索引`idx`，向其他节点获取在`idx`索引处值为1的交易级别的bloom，如果没有收到回复，则表明该区块验证失败
- 轻客户端希望监视记录的事件。这里描述的协议如下：
	* 轻客户端获取所有的区块头，检查包含与轻客户端想要监视的主题或者地址列表相匹配的布隆过滤器的区块
	* 一旦找到一个可能匹配的的区块头，轻客户端下载所有的交易收据，检查布隆过滤器匹配的交易
	* 找到可能匹配的交易后，轻客户端检查其日志的RLP是否匹配

前三个轻客户端协议需要对数级别的数据访问与计算；而第四个协议由于布隆过滤器是一个二级结构，则需要$\sim O(\sqrt{n})$。当然如果轻客户端针对想要的交易索引愿意依赖于全节点并且可以停止依赖某个丢失一些交易的节点，$O(\sqrt{n})$可以优化到$O(\log{n})$。第一个协议对于简单地检查状态很有用。而第二个协议则应用于顾客-商家场景中用以验证交易。第三个协议则允许多个以太坊轻客户端在几乎不存在信任关系的环境中共同验证一个区块。例如在比特币中，矿工可以创建一个区块，并且获得超额的交易费，轻客户端是无法自行检测到这种情况的，也无法看到一个诚实全节点验证这个区块无效。而在以太坊中，如果一个区块是无效的，那么它必定包含某个无效的状态转换，因此该交易将无法通过轻客户端的验证。


第四个协议的应用场景：DAPP想要追踪一些需要高效验证但不需要持久化的事件。一个例子是去中心化的exchange日志或者钱包记录交易（注意，轻客户端协议需要区块头级别的coinbase和叔块进行扩充）。

## Tips

交易收据的结构可以参见源码[receipt.go](https://github.com/ethereum/go-ethereum/blob/master/core/types/receipt.go)与[黄皮书4.3.1](https://ethereum.github.io/yellowpaper/paper.pdf)。从high-level来看，receipt包含cumulative gas used表示当交易发生时已经使用的gas，logs表示交易执行过程产生的日志信息，bloom表示由log信息构成的布隆过滤器以及status表示交易状态。

```go
type Receipt struct {
	// Consensus fields: These fields are defined by the Yellow Paper
	PostState         []byte `json:"root"`
	Status            uint64 `json:"status"`
	CumulativeGasUsed uint64 `json:"cumulativeGasUsed" gencodec:"required"`
	Bloom             Bloom  `json:"logsBloom"         gencodec:"required"`
	Logs              []*Log `json:"logs"              gencodec:"required"`

	// Implementation fields: These fields are added by geth when processing a transaction.
	// They are stored in the chain database.
	TxHash          common.Hash    `json:"transactionHash" gencodec:"required"`
	ContractAddress common.Address `json:"contractAddress"`
	GasUsed         uint64         `json:"gasUsed" gencodec:"required"`

	// Inclusion information: These fields provide information about the inclusion of the
	// transaction corresponding to this receipt.
	BlockHash        common.Hash `json:"blockHash,omitempty"`
	BlockNumber      *big.Int    `json:"blockNumber,omitempty"`
	TransactionIndex uint        `json:"transactionIndex"`
}
```

以下是各字段的解释。

```
blockHash: String, 32 Bytes - hash of the block where this transaction was in.
blockNumber: Number - block number where this transaction was in.
transactionHash: String, 32 Bytes - hash of the transaction.
transactionIndex: Number - integer of the transactions index position in the block.
from: String, 20 Bytes - address of the sender.
to: String, 20 Bytes - address of the receiver. null when its a contract creation transaction.
cumulativeGasUsed: Number - The total amount of gas used when this transaction was executed in the block.
gasUsed: Number - The amount of gas used by this specific transaction alone.
contractAddress: String - 20 Bytes - The contract address created, if the transaction was a contract creation, otherwise null.
logs: Array - Array of log objects, which this transaction generated.
status : String - '0x0' indicates transaction failure , '0x1' indicates transaction succeeded.
```

# 结语

本文翻译自以太坊官方wiki给出的Light Client Protocol：[https://github.com/ethereum/wiki/wiki/Light-client-protocol](https://github.com/ethereum/wiki/wiki/Light-client-protocol)

