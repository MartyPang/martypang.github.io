---
layout: post
title: 'Tendermint User Guide'
description: "Tendermint使用教程"
date: 2018-05-06
author: MartyPang
cover: '/assets/img/tendermint.jpg'
tags: Tendermint BFT Blockchain
---

> 如何配置使用Tendermint

### Tendermint安装
从[Github](https://github.com/tendermint/tendermint/releases)上下载最新二进制包（tendermint_0.22.0_linux_amd64.zip）。
解压缩到任意位置。

```shell
unzip tendermint_0.22.0_linux_amd64.zip -d ~/tendermint
```

设置tmhome路径，默认为~/.tendermint。可通过设置环境变量TMHOME修改。编辑~/.bashrc文件，添加：

```shell
export TMHOME=$HOME/tendermint/tmhome
```

修改tendermint配置，不打包空块。打开`$TMHOME/config/config.toml`，修改配置：

```
# EmptyBlocks mode and possible interval between empty blocks in seconds 
######################### Modified by SSS #############################
create_empty_blocks = false
# create_empty_blocks_interval = 0
```
修改连接配置：

```
abci = "grpc"
```

### Tendermint使用
常用命令：
- tendermint init
- tendermint node
- tendermint unsafe_reset_all

tendermint init一般只使用一次，除非改了config，需要重新初始化使配置生效。在运行tendermint node之前，需要启动我们的app(即一个实现了check，deliver，commit等方法的应用)。tendermint unsafe_reset_all可以重置tendermint环境，删除区块数据。

### Tendermint多节点配置
以A，B两个节点双节点tendermint配置为例。
首先在A机器上，打开`$TMHOME/config`下的genesis.json文件，清空validator列表的内容。然后将priv_validator.json中的内容复制到validators列表中，复制完毕后增加一项`"power":10`。将B节点的priv_validator.json同样复制到A节点的validator列表中，也增加power配置。

接着将A机器的genesis.json替换B节点的genesis.json文件。

最后A节点启动时，使用命令：

```shell
tendermint node --p2p.persistent_peers=Host_B:46656
```

同理，B节点在启动时，使用命令：

```shell
tendermint node --p2p.persistent_peers=Host_A:46656
```
附genesis.json文件示例。

```json
{
    "genesis_time": "0001-01-01T00:00:00Z",
    "chain_id": "upchain",
    "validators": [
        {
            "address": "D7519DF1B809C2E5B90400B9DCD685B84230F89F",
            "pub_key": {
                "type": "ed25519",
                "data": "B617D84892C15D2E32C02141D2BD31BE6160975D548C810397065BA9D877CB38"
            },
            "last_height": 0,
            "last_round": 0,
            "last_step": 0,
            "last_signature": null,
            "power": 10,
            "priv_key": {
                "type": "ed25519",
                "data": "12FE9E17003E101CDA14520B8C85323FE03077ED0A63CEA65114504F8DB43405B617D84892C15D2E32C02141D2BD31BE6160975D548C810397065BA9D877CB38"
            }
        },
        {
            "address": "BE2357FE11A8231D9EF98C621177E2582F08178B",
            "pub_key": {
                "type": "ed25519",
                "data": "A6D46DA3315241D40F37634B8BB171FB989A6C0F7014EDEC41D1DBF8BDCA88C8"
            },
            "last_height": 0,
            "last_round": 0,
            "last_step": 0,
            "last_signature": null,
            "power": 10,
            "priv_key": {
                "type": "ed25519",
                "data": "4F0B0717CBDAF3A44B404EC92F06CD72F738EFB2DBDD1636F767327251B246B0A6D46DA3315241D40F37634B8BB171FB989A6C0F7014EDEC41D1DBF8BDCA88C8"
            }
        }
    ],
    "app_hash": ""
}
```
