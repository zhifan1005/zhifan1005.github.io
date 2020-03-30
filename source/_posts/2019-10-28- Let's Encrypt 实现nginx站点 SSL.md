---
title:  Let's Encrypt 实现nginx站点 SSL
date: 2019-10-28 22:57:28
tags: 
- SSL
categories: Server
---

@[toc]

---

## 1.安装 Certbot

> Certbot 其实就是维护 Let's Encrypt 的 Package，在 Ubuntu上，我们可以这样安装：

>首先安装 Nginx：

```shell
sudo apt-get install nginx

```

>以上过程，等待安装完毕就好。

>然后添加 package repository


```shell
sudo add-apt-repository ppa:certbot/certbot

```

>如果出现如下报错

***add-apt-repository: command not found***


```shell
#解决方式
sudo apt install software-properties-common
sudo apt-get update

```

>这个过程中，等待验证完毕，按下 ENTER 就好。然后更新 apt 源数据：


```shell
sudo apt-get update

```

>最后，安装 Certbot 的 Nginx package：


```shell
sudo apt-get install python-certbot-nginx

```
## 2.配置 Nginx

>安装完 Nginx 和 Certbot 之后，需要简单配置 Nginx 以便于 Let's Encrypt 能起作用：

```shell
sudo vi /etc/nginx/sites-available/default

```

>使用 vi 编辑器打开 /etc/nginx/sites-available/default，可以直接删除里面的所有内容，然后再添加下面的配置：

```shell

server {
    listen 80;
    listen [::]:80;
    server_name your-domain.com www.your-domain.com;
}

```

```shell

***注意这里的 your-domain.com 换成你自己的域名。***

>保存退出之后，执行以下命令来检测 Nginx 的配置文件是否有错：
```

```shell
sudo nginx -t

```

>如果出现类似 syntax ok 这样的语句，就说明 Nginx 的配置文件没有问题。之后就是重新加载 Nginx 的配置文件了

## 3.签发 SSL 证书

>前面的两大步配置完成，就可以使用 Let's Encrypt 签发 SSL 证书了

```shell
sudo certbot --nginx -d your-domian.com -d www.your-domain.com

```

***注意这里的 your-domain.com 换成你自己的域名。***

>如果你第一次运行 certbot 命令的话，你需要在弹出的窗口输入你的邮箱地址还有需要接受 Let's Encrypt 的协议，这样之后，你大概会看到下面的文字：

```shell
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):

```

>在上面这里选择 1 或者 2，我推荐大家直接选择 2，因为这个会直接将你的 nginx 文件配置好并且是会将 http 跳转到 https 的。

>选择完毕之后，等待 SSL 生成完毕，就会有类似这样的输出：


```shell
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/your-domain.com/fullchain.pem. Your cert will
   expire on 2017-12-29. To obtain a new or tweaked version of this
   certificate in the future, simply run certbot again with the
   "certonly" option. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:
   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```
***

***然后在上面的文字中，这个 /etc/letsencrypt/live/your-domain.com/fullchain.pem 路径很重要，就是你的 SSL 证书路径。***

***

***其实到这里，访问 your-domain.com 应该就可以看到 https 的效果了。***
***
## 4.自动更新证书
>因为 Let's Encrypt 签发的 SSL 证书有效期只有 90 天，所有在过期之前，我们需要自动更新 SSL 证书，而如果你使用最新的 certbot 的话，Let's Encrypt 会帮你添加自动更新的脚本到 /etc/cron.d 里，你只需要去检测一下这个命令是否生效就OK！
***


```shell
sudo certbot renew --dry-run

```

>如果这个命令你没看到什么 error 的话，那就是没什么问题了

---
