---
layout: post
title: 'Migrate Jekyll blog to a VPS'
description: "从GitHub Page迁移Jekyll博客到个人服务器"
author: Marty Pang
categories: 
  - Tutorial
  - Blog
tags: 
  - Jekyll
  - VPS
last_modified_at: 2019-06-17T09:42:19-05:00
---

阿里云香港节点拔网线半个月后终于忍不了了（原因都懂），退款重新开了一台。搭环境，迁博客这一套组合拳下来也要费不少功夫，这篇博客记录一下全过程，以备不时之需。(ノへ￣、)

* 目录
{:toc}

# Setting Jekyll Environment

我们首先搞定[jekyll](https://jekyllrb.com/)环境。Jekyll是ruby提供的一个静态网站生成器，GitHub Pages就是由Jekyll支持的，网上也有许多关于如何使用GitHub部署Jekyll博客的文章，支持自定义的域名，提供SSL。如果觉得GitHub Pages够用，那么就可以略过以下内容了。

开始之前，先更新软件包列表，以确保软件包及其依赖项都是最新的。

```shell
sudo apt-get update
```

## nodejs

如果使用`jekyll server`部署项目，需要安装nodejs依赖，否则会报错。只使用Nginx的话，可以跳过此步。

nodejs安装方式官方文档都写得很详细，可以自行查看官网[nodejs.org](www.nodejs.org)。这里给出源码安装的几条命令。

```shell
wget https://nodejs.org/dist/v10.16.0/node-v10.16.0.tar.gz
tar -zxvf node-v10.16.0.tar.gz
cd node-v10.16.0
./configure
make
make install
```

## Install Ruby with rbenv

使用apt亦可安装ruby，安装的版本貌似是2.3的，之前配jekyll的时候貌似出过问题。这里采用rbenv来管理ruby，可以轻松切换不同版本的ruby环境。

**从`rbenv`仓库克隆一份源码**

```shell
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

**添加环境变量**

用vim编辑当前用户的配置脚本`~/.bashrc`，在文件末尾添加两行：

```
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

或者直接用`echo`：

```shell
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

添加`~/.rbenv/bin`到`PATH`方便使用rbenv命令。`eval "$(rbenv init -)"`以便rbenv自动卸载。

`source ~/.bashrc`使配置文件生效，应用于当前shell。

**安装`ruby-build`插件**

`ruby-build`添加了`rbenv install`命令，简化了ruby的安装过程。

```shell
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

**安装ruby 2.5.1**

`rbenv install -l`可以列出所有可安装的ruby版本，这里我们安装2.5.1。

```shell
rbenv install 2.5.1
```

完成安装后，使用`global`命令将ruby 2.5.1配置为默认的版本。

```shell
rbenv global 2.5.1
```

到此，ruby的安装结束，你可以使用`ruby -v`查看版本。

如果使用gemfile和bundler，需要额外安装依赖。

```shell
gem install bundler
gem install rails
```

## Jekyll

接下来就可以使用`gem`安装jekyll了。

```shell
gem install jekyll bundler
```

至此，jekyll环境配置结束。

# Install Nginx

本身`jekyll server`命令即可开启站点服务，但由于服务器上用Nginx部署了调用disqus API的服务（详见[]()），索性jekyll站点也用Nginx做服务。

安装Nginx很简单：

```shell
apt-get install nginx
service nginx start
```

以上两条命令就可以开启Nginx服务。

# Deploy your site

## Adjust Nginx configuration

进入`/etc/nginx/sites-available`目录，复制一份defaut配置，文件名为你的域名，例如：

```shell
cp default hytheory.com
```

打开`hytheory.com`配置文件，将默认的server配置改为如下：

```
server {
 listen 80 default_server;
 listen [::]:80 default_server;
        server_name www.hytheory.com;
        return 301 https://$server_name$request_uri;

 root /var/www/hytheory.com;

 # Add index.php to the list if you are using PHP
 index index.html index.htm index.nginx-debian.html;

 location / {
  # First attempt to serve request as file, then
  # as directory, then fall back to displaying a 404.
  try_files $uri $uri/ =404;
 }
}
```

之后进入`/etc/nginx/sites-enabled`目录，建一个指向上述配置文件的软链接：

```shell
ln -s /etc/nginx/sites-available/hytheory.com /etc/nginx/sites-enabled/hytheory.com
```

使用`nginx -t`测试配置是否正确。

## build Jekyll blog

根据`hytheory.com`文件中配置的root目录，我们把jekyll blog部署到相应的`/var/www/hytheory.com`目录。

```shell
jekyll build --source ~/hybridtheory/ --destination /var/www/hytheory.com/
```

如果已经做了域名解析，那么至此你的jekyll博客应该是处于可访问状态的。

可参考我另一篇文章，[如何使用GitHub的webhook，自动build博客](https://www.hytheory.com/tutorial/blog/deploy-jekyll-site-automatically-with-git-webhook/)。如果想在墙内使用Disqus服务的，参考[如何科学使用Disqus](https://www.hytheory.com/tutorial/blog/using-disqus-api-and-vps-to-provide-comment-module-for-jekyll-blog/)

## Go HTTPS

在为域名开启HTTPS之前，首先需要有证书。一般云服务商都提供SSL证书服务，以阿里云为例，可以购买免费的[Symantec证书](https://common-buy.aliyun.com/?spm=5176.2020520163.cas.3.47dbNchlNchluv&commodityCode=cas#/buy)，个人建站够用。

![Symantec](/images/20190617/symantec.png){:  .align-center}

证书购买后下载到本地，传输到服务器上。之后配置Nginx，开启SSL。修改`hytheory.com`：

```
server {
 listen 80 default_server;
 listen [::]:80 default_server;
        server_name www.hytheory.com;
        return 301 https://$server_name$request_uri;
}

server {
 listen 443 default_server;
 listen [::]:443 default_server;

 ssl on;
 ssl_certificate path/to/pem;
 ssl_certificate_key path/to/key;

 ssl_session_timeout 5m;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;        
 ssl_prefer_server_ciphers on;
 ssl_session_cache shared:SSL:10m;

 root /var/www/hytheory.com;

 # Add index.php to the list if you are using PHP
 index index.html index.htm index.nginx-debian.html;

 server_name hytheory.com www.hytheory.com;

 location / {
  # First attempt to serve request as file, then
  # as directory, then fall back to displaying a 404.
  try_files $uri $uri/ =404;
 }
}
```

至此，个人VPS上搭建Jekyll博客教程结束。相关参考：

- [Jekyll博客在墙内科学使用Disqus评论系统](https://www.hytheory.com/tutorial/blog/using-disqus-api-and-vps-to-provide-comment-module-for-jekyll-blog/)
- [Automated Deployment of Jekyll Blog with Git Webhooks and Flask](https://www.hytheory.com/tutorial/blog/deploy-jekyll-site-automatically-with-git-webhook/)

