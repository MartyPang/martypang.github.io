---
layout: post
title: 'Hyperledger Fabric User Guide'
description: "Hyperledger Fabric 环境配置"
date: 2017-10-28
author: MartyPang
cover: '/assets/img/fabric.png'
tags: [Hyperledger Fabric, Fabric, Blockchain, Smart Contract]
---

> Hyperledger Fabric环境配置以及智能合约的使用

# Hyperledger Fabric 安装、配置及使用

## 安装Docker

###  Window版本安装

在docker[官方下载页](https://www.docker.com/products/docker#/windows)下载安装程序，安装即可。
### Ubuntu、debian系列

#### 系统要求

Docker 支持以下版本的 [Ubuntu](https://www.ubuntu.com/server) 和 [Debian](https://www.debian.org/intro/about) 操作系统：

* Ubuntu Xenial 16.04 (LTS)
* Ubuntu Trusty 14.04 (LTS)
* Ubuntu Precise 12.04 (LTS)
* Debian testing stretch (64-bit)
* Debian 8 Jessie (64-bit)
* Debian 7 Wheezy (64-bit)（*必须启用 backports*)

Ubuntu 发行版中，LTS（Long-Term-Support）长期支持版本，会获得 5 年的升级维护支持，这样的版本会更稳定，因此在生产环境中推荐使用 LTS 版本。

Docker 目前支持的 Ubuntu 版本最低为 12.04 LTS，但从稳定性上考虑，推荐使用 14.04 LTS 或更高的版本。

Docker 需要安装在 64 位的 x86 平台或 ARM 平台上（如[树莓派](https://www.raspberrypi.org/)），并且要求内核版本不低于 3.10。但实际上内核越新越好，过低的内核版本可能会出现部分功能无法使用，或者不稳定。

#### 使用脚本自动安装

Docker 官方为了简化安装流程，提供了一套安装脚本，Ubuntu 和 Debian 系统可以使用这套脚本安装：

```bash
$ curl -sSL https://get.docker.com/ | sh
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 安装在系统中。

不过，由于伟大的墙的原因，在国内使用这个脚本可能会出现某些下载出现错误的情况。国内的一些云服务商提供了这个脚本的修改版本，使其使用国内的 Docker 软件源镜像安装，这样就避免了墙的干扰。

##### 阿里云的安装脚本

```bash
$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

##### DaoCloud 的安装脚本

```bash
$ curl -sSL https://get.daocloud.io/docker | sh
```

#### 手动安装
添加官方Docker仓库的GPG key到系统

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加Docker仓库到APT源

```bash
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

添加成功后，更新 apt 软件包缓存。

```bash
$ sudo apt-get update
```

##### 安装 Docker

在一切准备就绪后，就可以安装最新版本的 Docker 了。

```bash
$ sudo apt-get install -y docker-ce
```

如果系统中存在旧版本的 Docker （`lxc-docker`, `docker.io`），会提示是否先删除，选择是即可。

##### 启动 Docker 引擎

##### Ubuntu 16.04、Debian 8 Jessie/Stretch

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

##### 建立 docker 用户组

默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```bash
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```bash
$ sudo usermod -aG docker $USER
```

#### 参考文档

* [Docker 官方 Ubuntu 安装文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
* [Docker 官方 Debian 安装文档](https://docs.docker.com/engine/installation/linux/debian/)

## 安装Docker-compose
### 使用[pip](http://pip-cn.readthedocs.io/en/latest/installing.html)进行安装
安装 docker-compose（推荐为 1.7.0 及以上版本）。

```bash
$ sudo pip install docker-compose>=1.7.0
```

### 使用docker github仓库进行安装

```bash
$ sudo curl -o /usr/local/bin/docker-compose -L "https://github.com/docker/compose/releases/download/1.15.0/docker-compose-$(uname -s)-$(uname -m)"
```

设置权限

```bash
$ sudo chmod +x /usr/local/bin/docker-compose
```

验证是否安装成功

```bash
$ docker-compose -v
```

## 安装部署Fabric 1.0

### clone Hyperledger Fabric samples
选取一个路径存放Fabric samples执行以下命令。

```bash
$ git clone https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
```

fabric samples包含一些基本的网络部署例子以及chaincode实例，建议初学者下载学习。

### Download Platform-specific Binaries

```bash
$ curl -sSL https://goo.gl/eYdRbX | bash
```

上述curl命令会下载并执行一个bash脚本，该脚本会下载并提取用于在fabric平台构建网络的工具存放于bin目录下，包括：

- `cryptogen`
- `configtxgen`
- `configtxlator`, and
- `peer`


之后，脚本会从Docker Hub下载Hyperledger Fabric docker 镜像。This may take a while.

镜像下载完成后，可查看fabric镜像。

```sh
hyperledger/fabric-ca                 latest              5f30bda5f7ee        7 days ago          238MB
hyperledger/fabric-ca                 x86_64-1.0.1        5f30bda5f7ee        7 days ago          238MB
hyperledger/fabric-tools              latest              259847d24868        7 days ago          1.34GB
hyperledger/fabric-tools              x86_64-1.0.1        259847d24868        7 days ago          1.34GB
hyperledger/fabric-couchdb            latest              dd645e1e92c7        7 days ago          1.48GB
hyperledger/fabric-couchdb            x86_64-1.0.1        dd645e1e92c7        7 days ago          1.48GB
hyperledger/fabric-kafka              latest              cbdc916590a0        7 days ago          1.3GB
hyperledger/fabric-kafka              x86_64-1.0.1        cbdc916590a0        7 days ago          1.3GB
hyperledger/fabric-zookeeper          latest              eb07e5cc9674        7 days ago          1.31GB
hyperledger/fabric-zookeeper          x86_64-1.0.1        eb07e5cc9674        7 days ago          1.31GB
hyperledger/fabric-orderer            latest              bbf2708c9487        7 days ago          179MB
hyperledger/fabric-orderer            x86_64-1.0.1        bbf2708c9487        7 days ago          179MB
hyperledger/fabric-peer               latest              abb05def5cfb        7 days ago          182MB
hyperledger/fabric-peer               x86_64-1.0.1        abb05def5cfb        7 days ago          182MB
hyperledger/fabric-javaenv            latest              2bd60859415d        7 days ago          1.42GB
hyperledger/fabric-javaenv            x86_64-1.0.1        2bd60859415d        7 days ago          1.42GB
hyperledger/fabric-ccenv              latest              7e2019cf8174        7 days ago          1.29GB
hyperledger/fabric-ccenv              x86_64-1.0.1        7e2019cf8174        7 days ago          1.29GB
hyperledger/fabric-baseimage          x86_64-0.3.1        21e9f22bc020        5 weeks ago         987MB
hyperledger/fabric-baseos             x86_64-0.3.1        21e9f22bc020        5 weeks ago         987MB
yeasy/hyperledger-fabric-base         1.0.0               21e9f22bc020        5 weeks ago         987MB
```


### 利用basic-netwok配置的网络测试chaincode

本节利用fabric samples中basic-network测试从网络启动到chaincode_example02部署，调用的过程。2.4.4节将详细说明配置自定义区块链网络的过程。
basic-network中部署了一个包含一个channel名为mychannel只有一个org的区块链网络。

#### 启动网络

首先，进入basic-network目录。

```bash
$ cd basic-network
```

若无错误信息，以下命令将启动五个容器，分别是一个ca，一个orderer，一个peer，一个couchdb，一个用于交互的cli。

```bash
$ docker-compose -f docker-compose.yml up -d ca.example.com orderer.example.com peer0.org1.example.com couchdb cli
```

另外打开一个terminal，利用docker查看容器的详细信息。

```bash
$ docker ps
```

终端输出信息应该如下。

```bash
CONTAINER ID        IMAGE                                     COMMAND                  CREATED             STATUS              PORTS                                            NAMES
9587c35e3aba        hyperledger/fabric-peer:x86_64-1.0.0      "peer node start"        5 hours ago         Up 2 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
53f540b3e450        hyperledger/fabric-orderer:x86_64-1.0.0   "orderer"                5 hours ago         Up 2 minutes        0.0.0.0:7050->7050/tcp                           orderer.example.com
734b2a7f48b0        hyperledger/fabric-ca:x86_64-1.0.0        "sh -c 'fabric-ca-..."   5 hours ago         Up 2 minutes        0.0.0.0:7054->7054/tcp                           ca.example.com
3bad98f4da9e        hyperledger/fabric-tools:x86_64-1.0.0     "/bin/bash"              5 hours ago         Up 2 minutes                                                         cli
82aa4d729db0        hyperledger/fabric-couchdb:x86_64-1.0.0   "tini -- /docker-e..."   5 hours ago         Up 2 minutes        4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
```

#### 创建channel

```bash
$ docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
```

#### 加入channel

```bash
$ docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block
```

#### 进入peer容器

```bash
$ docker exec -it cli bash
```

#### 安装chaincode

```bash
$ peer chaincode install -p github.com/chaincode_example02 -n mycc -v 1.0 
```

终端打印如下输出表示chaincode安装成功。

```bash
[chaincodeCmd] install -> DEBU 00d Installed remotely response:<status:200 payload:"OK" > 
```

#### 实例化chaincode

```bash
$ peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}'
```

#### query chaincode

```bash
$ peer chaincode query -n mycc -c '{"Args":["query", "a"]}' -C mychannel
```

可以看到查询结果为：

`Query Result: 100`

#### invoke chaincode

```bash
$ peer chaincode invoke -n mycc -c '{"Args":["invoke","a", "b","50"]}' -C mychannel
```

再次执行上一步的查询命令，可以看到查询结果为50。

`Query Result: 50`

### 配置网络

#### 指定path

将工具位置加入到环境变量中。

```bash
export PATH=$GOPATH/src/github.com/hyperledger/fabric/build/bin:${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}
```

#### cryptogen生成加密文件

```bash
$ cryptogen generate --config=./crypto-config.yaml //crypto-config.yaml是配置文件，可自行需改
```

执行完成后，生成crypto-config文件夹，内有order节点、peer节点的organization的配置信息。

#### configtxgen为orderer生成genesis block

```bash
$ configtxgen -profile $PROFILE_NAME -outputBlock ./config/genesis.block //$PROFILE_NAME在configtx.yaml命名
```

执行完成后，生成创世块genesis.block。

#### configtxgen 生成channel tx

```bash
$ configtxgen -profile $PROFILE_NAME -outputCreateChannelTx ./config/channel.tx -channelID $CHANNEL_NAME //$PROFILE_NAME,$CHANNEL_NAME在configtx.yaml命名
```

执行完成后，生成channel configuration transaction channel.tx。
