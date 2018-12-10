---
layout: post
title: '在境外VPS上搭建反向代理转发Disqus API 请求'
description: "反向代理实现墙内使用Disqus"
date: 2018-12-08
author: MartyPang
cover: '/assets/img/disqus-cover.png'
tags: [Disqus, Jekyll, VPS, Nginx]
---

> 如何配置使用Tendermint

# Jekyll博客在墙内科学使用Disqus评论系统

## 原生Disqus评论系统使用
Disqus是一个第三方的评论托管网站，提供第三方账户登陆评论和匿名留言功能，是个人博客评论系统的不二之选。在博客上使用Disqus很简单，只需在Disqus上申请账号（需自备梯子），添加site，复制专属的js代码到post即可。
如图创建新的site，为新的site添加shortname并选择合适的目录和语言。
![](/assets/img/disqus-create-site.png)
接着选择博客平台，以jekyll为例。
![](/assets/img/disqus-jekyll.png)

首先在jekyll项目的`_config.yml`配置文件中设置comment选项为true，并配置shortname。
```yaml
# Comments 评论功能
comments:
  disqus: true
  disqus_url: 'https://sitename.disqus.com/embed.js'
```

接着在`_layouts/post.html`文件中添加如下js，即可正常使用Disqus评论功能。
```javascript
  {% if site.comments.disqus %}
  <script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
      var d = document,
        s = d.createElement('script');
      s.src = '{{ site.comments.disqus_url }}';
      s.setAttribute('data-timestamp', +new Date());

      (d.head || d.body).appendChild(s);
    })();
  </script>
  {% endif %}
```

最后，如果你在一个可以访问Disqus评论的环境或者你的用户也均可访问Disqus，请忽略本文以下内容:)。

## 科学访问Disqus
最近刚好在阿里云购买了一台香港节点轻服务器（便宜的不行，[大家快去买啊~](https://common-buy.aliyun.com/?commodityCode=swas&userCode=1cbk3kmv&regionId=cn-hongkong#/buy)），部署了科学上网神器之后想到自己之前苦于博客无法使用disqus评论，有了这台vps，可以搞个反向代理转发disqus API请求。程序员交友网站一番搜索，发现已经有好多repository干了这个事了，就不再重复造轮子。以下使用的是fooleap写的[disqus-php-api](https://github.com/fooleap/disqus-php-api)，该库通过一个能翻墙的VPS设置Nginx反向代理server，利用php curl转发对Disqus的API调用，并返回结果。这篇post记录下自己配置反代server的过程。

### 准备材料
首先当然是需要一台境外的VPS主机，去搬瓦工，Vultr或者最近免费薅一年羊毛的Google Cloud置办均可。博主自己撸了阿里云24一个月的香港节点轻服务器，已经能满足基本需要吧。

### 搭建过程
基本的搭建过程有如下几步：
- 在vps服务器上安装配置Nginx和PHP；
- 部署Disqus-php-server，并修改反向代理server配置；
- 修改Jekyll博客页面disqus配置；

#### 0. 安装Nginx和PHP服务
不同服务器系统安装配置nginx和php大同小异，这里以Ubuntu 16.04为例，直接使用apt安装nginx和php。
```shell
sudo apt-get install nginx php-fpm php-mysql
```

php有个安全漏洞，需要配置cgi默认安全配置，设置`php.ini`的`cgi.fix-pathinfo`选项值为1即可。
```shell
sudo vim /etc/php/fpm/php.ini
# vim搜索cgi.fix-pathinfo
\cgi.fix-pathinfo
```


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

## Tendermint使用
常用命令：
- tendermint init
- tendermint node
- tendermint unsafe_reset_all

tendermint init一般只使用一次，除非改了config，需要重新初始化使配置生效。在运行tendermint node之前，需要启动我们的app(即一个实现了check，deliver，commit等方法的应用)。tendermint unsafe_reset_all可以重置tendermint环境，删除区块数据。

## Tendermint多节点配置
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
