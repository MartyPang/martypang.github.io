---
layout: post
title: '在境外VPS上搭建反向代理转发Disqus API 请求'
description: "反向代理实现墙内使用Disqus"
date: 2018-12-08
author: MartyPang
cover: '/assets/img/disqus-cover.png'
tags: [Disqus, Jekyll, VPS, Nginx]
---

> 如何科学使用Disqus

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
# 设置fix-pathinfo为0
cgi.fix-pathinfo=0
```

之后配置Nginx。
```shell
sudo vim /etc/nginx/sites-available/default
```
修改default文件如下，`domain_or_ip`指定使用PHP processor。
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name server_domain_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
保存，重载nginx服务。
```shell
sudo systemctl reload nginx
```

#### 1. 部署disqus-php-api
首先将disqus-php-api代码clone到本地。
```shell
git clone https://github.com/fooleap/disqus-php-api.git
```

将api目录下的php文件拷贝到Nginx配置文件中所示的root目录下（`/var/www/html`）即可。
```shell
cp -r ./disqus-php-api/api/* /var/www/html
# 进入root目录以便后续操作
cd /var/www/html
```

目录下`config.php`为调用官方disqus api所需的配置文件。主要修改如下字段：
```
define('DISQUS_PUBKEY', 'pubkey');
define('DISQUS_USERNAME', 'martypang');
define('DISQUS_EMAIL', 'pang.shuaifeng@gmail.com');
define('DISQUS_PASSWORD', 'password');
define('DISQUS_WEBSITE', 'https://martypang.github.io');
define('DISQUS_SHORTNAME', 'shortname');
define('DISQUS_APPROVED', true);
```

在修改之前，你可能需要在disqus上申请一个app以获取公钥。注册app网址，[点我](https://disqus.com/api/applications/)，根据提示填写相应信息即可。

#### 2. 修改博客页面disqus配置
之前我们已经讲了如何配置原生的disqus，加入一段js即可。使用fooleap的php api实现反向代理的disqus也很简单。首先把clone下来的代码中的dist文件夹下的`iDisqus.min.css`，`iDisqus.min.css.map`，`iDisqus.min.js`和`iDisqus.min.js.map`放入jekyll博客的相应目录即可（分别复制到`/assets/css/`和`assets/js/`）。然后在相应的html头部引入。

接着把原生disqus配置的js：
```javascript
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
```
直接替换如下即可：
```javascript
<div id="comment"></div>
<script type="text/javascript">
  $(document).ready(function () {
    var disq = new iDisqus('comment', {
        forum: 'hybridtheory',
        api: 'https://domian.com',
        site: 'https://martypang.github.io',
        url: location.pathname,
        mode: 1,
        timeout: 2000,
        init: true,
      });
    disq.count();
  });
</script>
```
`iDisqus`具体的配置详见[项目仓库](https://github.com/fooleap/disqus-php-api)。`mode`为1表示检测能否访问 Disqus，若能则加载 Disqus 原生评论框，超时则加载简易评论框。

