---
layout: post
title: 'Automated Deployment of Jekyll Blog with Git Webhooks and Flask'
description: "利用Git Webhooks实现jekyll博客自动化部署"
date: 2018-12-15
author: MartyPang
cover: '/assets/img/webhook-cover.png'
tags: [Webhooks, Jekyll, Flask, Nginx]
---

> 如何自动化部署Jekyll博客

# Automated Deployment of Jekyll Blog with Git Webhooks and Flask

在我之前的一篇[博客](https://www.hytheory.com/2018/12/08/using-disqus-api-and-vps-to-provide-comment-module-for-jekyll-blog.html)中说到购置了一台阿里云的轻量级应用服务器并讲述了如何配置Nginx反向代理转发Disqus请求。在完成所有配置后，我发现这条路子还是走不通，原因在于原本托管在github上的jekyll博客采用git pages自动部署的，使用的是https协议，而Nginx这端如果想配置SSL必须在阿里云上绑定域名并购买相应的证书服务。心想，既然域名，证书，VPS都齐全了，不如直接将博客部署在VPS上。部署的过程比较简单，这里不再赘述，给出一篇[参考博客](https://tomisacat.xyz/tech/2017/02/27/Deploy-a-blog-site-with-Jekyll-and-Nginx.html)。
博客迁移完成后，我意识到更新代码是一个巨繁琐的过程：在本地更新了代码或博客，`push`到远程分支，ssh登录到服务器，从远程分支`pull`最新代码，接着`jekyll build`，最后将生成的`_site`文件夹拷贝到指定目录。这一套组合拳下来，也是有点繁琐的。
这篇博客记录了搭建自动化jekyll博客部署的过程，主要结合的技术有Git Webhooks，轻量级Web框架Flask。这里假设你已经具备基础的Nginx部署知识，可参考[反向代理的搭建](https://www.hytheory.com/2018/12/08/using-disqus-api-and-vps-to-provide-comment-module-for-jekyll-blog.html)。

## 什么是Git Webhooks？
Git Webhooks允许服务器接收有关对git仓库执行操作的通知，并允许用户指定服务器在接收到通知时应该执行的操作。一般来说，如果我想通过更改git触发远程操作，webhooks是一个不错的选择。在我们的例子中，我们只对`push`事件感兴趣（任何时候推送代码到远程仓库）。

### 创建一个Github Webhook
如果你的网站托管在Github上，进入`Repo -> Settings -> Webhooks`并点击`Add Webhook`。
![](/assets/img/add-webhook.png)

在上图的页面中，你需要配置在某些操作发生时由Github调用的URL。这里我们还需要在服务器上部署一个简单的应用来处理Github的Post请求，下一小节将会讲到。`Content type`可选json，但这里我对payload并不是很感兴趣，故采用默认的`form-urlencoded`。另外还可以提供一个密码。其`SHA-1`哈希值将添加到`X-Hub-Signature`中。更多细节可以参见Git Webhooks的[官方文档](https://developer.github.com/webhooks/)。完成配置后点击`Add webhook`即可。

## 部署Flask应用
上一小节中，我们为博客repo创建了一个webhook，每次推送新代码到repo时，Github都会给我们的服务器发送一个POST请求，所以我们必须部署一个应用来处理该请求并重新构建jekyll。这里我不想使用一个比较重的web框架，因为我需要的仅仅是来自Github的一个通知。目前也有许多的轻量级Web框架，笔者选了Flask。在`/var/www/`目录下新建应用文件夹：
```shell
mkdir autorebuild -p
cd autorebuild
```
在安装`flask`之前安装虚拟环境`virtualenv`，进入目录创建新的python环境并激活：
```shell
pip install virtualenv
virtualenv venv
source /venv/bin/activate
```
使用`pip`安装`Flask`：
```shell
pip install flask
```
现在我们可以创建一个简单的处理webhook配置中指定请求的flask应用：
```python
from flask import Flask
import subprocess
import os

app = Flask(__name__)

@app.route('/rebuid', methods=['POST'])
def autobuildjekyll():
    script_path = "/var/www/autorebuild/jekyll_rebuild.sh"
    subprocess.call([os.path.expanduser(script_path)])
    return "Success"

if __name__ == "__main__":
    app.run()
```
上面的python代码（假设为refresh.py）创建了一个简单的flask应用，其中的一个endpoint（`/rebuild`）处理Git Webhook发来的`POST`请求。具体来说，`autobuildjekyll`函数调用一个简单的shell脚本。
`jekyll_rebuild.sh`脚本从Git仓库中拉取最新的修改，之后重建我们的站点：
```shell
#!/bin/bash
echo "Pulling latest from Git"
cd ~/hybridtheory/ && git pull

echo "Building Jekyll Site";
jekyll build --source ~/hybridtheory/ --destination /var/www/hytheory.com/ --incremental;
echo "Jekyll Site Built";
```
这里需要注意的是，千万不要省略shell脚本的头部`#!/bin/bash`，否则`subprocess.call()`会报如下错误：`OSError: [Errno 8] Exec format error`。
直接运行`python refresh.py`即可启动应用，默认绑定端口5000。由于我们采用Nginx作为博客的代理服务器，简单的启动一个flask应用是不够的，我们需要使用Gunicorn来启动flask应用。
[Gunicorn](https://gunicorn.org/)是一个Python wsgi http server，来源于Ruby的Unicorn项目，能够与各种wsgi web框架写作，比如Flask。首先进入虚拟环境安装Gunicorn：
```shell
pip install gunicorn
```
用gunicorn启动我们的flask应用：
```shell
gunicorn -b 127.0.0.1:8001 refresh:app
```
这里我们绑定`localhost`的8001端口，仅能内部访问。`refresh`为python文件的名字，`app`为应用名字。
最后我们修改Nginx配置，由于Github发送的是https请求，故我们绑定端口443。具体修改过程不再赘述，这里贴出Nginx的具体配置：
```yaml
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
        ssl_certificate /etc/nginx/sslcert/1613132_www.hytheory.com.pem;
        ssl_certificate_key /etc/nginx/sslcert/1613132_www.hytheory.com.key;

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
        location /refresh {
                proxy_pass http://127.0.0.1:8001;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

至此，所有的配置完成，测试成功。

## 参考博客
- [用 Jekyll/Nginx/Let'sEncrypt 搭建一个博客站点](https://tomisacat.xyz/tech/2017/02/27/Deploy-a-blog-site-with-Jekyll-and-Nginx.html)
- [Automate Jekyll with GitHub Webhooks](https://ryanharrison.co.uk/2018/07/05/jekyll-rebuild-github-webhook.html)
- [ubuntu（Flask + Gunicorn + Nginx 部署）](https://www.cnblogs.com/gjack/p/8066672.html)


