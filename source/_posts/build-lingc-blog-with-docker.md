---
title: 借助docker搭建个人hexo博客
abbrlink: 916ec1b6
date: 2019-12-13 22:40:38
tags:
---

> 感谢开源社区！我又爱上了写博客。

> 本文介绍借助docker搭建一个hexo博客，支持git hook在文章更新时自动触发博客系统重新部署，整站支持https，支持CDN分发静态资源，支持 www 形式的url 301 到non-www形式以避免seo权重分散；

<!--more-->

docker真的太棒了，身边前端朋友都在学docker。 我也想学，除了工作上将自己负责的编辑器全部容器化后，觉得博客系统也适合用docker，来来回回研究了几天，实现的功能不多，但是坑却不少。好久不写文章，文笔不好，就直接开始吧。

## 实现功能
1. 通过`webhook`在`git`仓库有`push`操作时自动触发服务器上的`hexo`站点更新； 
2. 根据`docker`配置自动为容器服务配置`nginx vhosts`和域名；
3. 自动为所绑定的域名申请免费`ssl证书`使站点可以以`https`访问；
4. 在`nginx`中配置`http`的`url`访问跳转到`https`方式；如`http://linguochi.com`会被重定向到`https://linguochi.com`
5. 使用`express`的`redirect`将 `带www的url`访问跳转到 `不带www的地址`；如`https://www.linguochi.com`会被重定向到`https://linguochi.com`；
6. 结合5和6，最终实现`http://linguochi.com`，`http://www.linguochi.com`，`https://www.linguochi.com`全部跳转到`https://linguochi.com`;
7. `hexo`、`git webhook`、`nginx`、`ssl证书分配`、`express redirect`均生成docker镜像，并用`docker-composer`编排，使得迁移站点更加方便；
8. 接入腾讯云的CDN服务和对象存储COS服务，因为比起直接主机存储与访问，CDN和COS不仅便宜而且更快。

## 步骤
### 购买个人服务器
今年双十一其他的都没买，就花了702元续费了4年我的祖传屌丝配置服务器（`1核1G内存按流量计费`的带宽），花了300元霸占了我的专有域名9年。觉得是这几年最值的一次双十一剁手。
想想家里的电脑都是`8核16G内存`起步了。。
阿里云、腾讯云我都买过，腾讯云比较便宜，套路也少；传说阿里云技术比较强悍，但是对我一个只用来存静态博客系统，跑脚本的人来说，只需要服务器不宕机，就足够了。

### 搭建hexo博客
`hexo`是很热门的开源博客系统，实现将`markdown`渲染成静态资源包，我们只需要将静态资源包放到服务器上，配置个`nginx`路径就可以访问。
`markdown`文档又可以直接托管在`git`服务上，也就是只要`git`服务不倒闭，就算服务器炸了，我们的博客都不会丢。

### 自动更新hexo博客
小白玩`hexo`是每写一篇文章，就生成一次静态资源包，然后手工搬运到服务器上，效率很低；

后来，有了`hexo`部署到`gitpage`的插件，通过一条`deploy`命令就可以将构建好的静态资源包托管在`gitpage`上，然后指定 `***.github.io`的域名去访问博客。
这个方式有一些好处：`github`是免费的，`gitpage`也是免费的，一切都是`免费`的。
当然也有一些不能忍受的：`github`在国内功夫网下，`速度不理想`。

再后来，受CI(持续集成)启发，可以实现写文章->`push到git`->触发webhook自动构建静态资源，自动部署；

### 将博客放在个人服务器上
没有什么比独占一个东西并且拥有绝对使用权更舒服的事情了。不到20元/月的服务器，又能装`docker`，又能跑脚本，还要什么自行车！腾讯云主机由于是国内的服务器，要部署`web服务`，域名需要`备案`。从申请备案到通过，腾讯云+通信管理局一共花了7天。速度算快的了，现如今不需要再等腾讯给你邮寄个幕布拍照了，一切都在小程序上`电子化`了，节约了很多时间。
备案还是相当有必要的，很多国内的其他服务，比如`CDN`、对象存储等服务，也都需要绑定的域名是备案的。

### 采用https方式访问
![https](https://cos.linguochi.com/2019/12/20/15768081792628.jpg)
单纯喜欢浏览器地址栏对`https`前面显示的那把锁，但搜索引擎对`https`站点加权比`http`多。
`https`需要申请一个`ssl证书`，腾讯有提供一年有效期的`免费ssl证书`，需要手动申请，手动填写在`nginx配置文件`里，不够优雅。
通过`jrcs/letsencrypt-nginx-proxy-companion`这个镜像可以结合`jwilder/nginx-proxy`镜像自动给`hexo`站点配置`vhost`域名，自动为该域名申请`ssl证书`。配置了这两个容器后，只需要在需要配置域名的容器`environment`分别配置`VIRTUAL_HOST`、`LETSENCRYPT_HOST` 、`LETSENCRYPT_EMAIL` 就可以指向对应域名并申请证书。
```shell
lingc-server:
    image: linguochi/hexo-blog
    container_name: hexo-blog
    depends_on:
      - letsencrypt
    environment:
      - VIRTUAL_HOST=linguochi.com
      - LETSENCRYPT_HOST=linguochi.com
      - LETSENCRYPT_EMAIL=linguochi@gmail.com
    volumes:
      - /opt:/app
```
### 对象存储
用对象存储主要是解决写`markdown`文档时图片的存储问题。
古老的方式是，将图片保存到`hexo`仓库的一个路径下，然后拷贝图片路径，再到`markdown`里粘贴，我对这个方式恐惧到写文档不敢贴图。
但是，目前有些解决方案可以是贴图过程变得相当简单。比如我用的`mweb`。可以将腾讯云、七牛云、阿里云等的对象服务配置好。之后，贴图过程只需要，随便截一张图或者复制一张图片，然后在`mweb`编辑区里面粘贴。它会自己把剪切板中的图片上传到对象存储上，生成对象链接并拼出相应的包含图片路径的`markdown`格式。
![mweb文件存储配置](https://cos.linguochi.com/2019/12/20/15768118060774.jpg)

### CDN部署
并不是我的博客有多少访问量，只是因为穷。按流量收费的服务器，1GB是8毛钱。但是如果把它步在CDN上，1GB是2毛钱，每个月前10GB还免费；而且CDN还能智能匹配访客最适合的接入节点，大大提高站点打开速度，简直不能太香。
在腾讯云上开通cdn服务很简单，不需要对服务器现有代码做任何改动。
配置需要开通CDN的域名，指定相应的回源地址，选择是否启动缓存、`https`等；然后静等部署。
![CDN域名](https://cos.linguochi.com/2019/12/20/15768114227712.jpg)
部署完毕后，将CNAME地址配置到域名解析里面就可以了。
![域名解析](https://cos.linguochi.com/2019/12/20/15768115179024.jpg)

### 301跳转
`301跳转`可以将多个分散的url指向一个，对`SEO`十分有用。
在不考虑`https`的时候，`301跳转`很容易，只需要域名解析里面配置一个`显性URL`就可以。如
![域名解析301跳转](https://cos.linguochi.com/2019/12/20/15768121366194.jpg)
但是上了https之后，在腾讯云上直接用域名解析就不能达到效果了。也尝试了在nginx里面做跳转，但是需要填写ssl证书信息，比较麻烦。考虑了一番，直接多开了一个容器，容器内运行了一个`express`的node服务，将链接捕获到之后进行重定向。
```javascript
const express = require('express');
const app = express();
const port = 80;

app.all('*', function(req, res) {
  return res.redirect(301, `https://linguochi.com${req.path}`);
});

app.listen(port, () => console.log(`Redirector ${port}!`));
```
这个服务绑定的域名是`www.linguochi.com`,同样也上了https，所以当以`https://www.linguochi.com`前缀访问站点的时候，也能跳转到`不带www`的https域了。

### docker化
`docker`的使用，是这次部署个人博客中最有成就感的事情。docker一直只闻其名，不知道它具体能干啥。在个人工作里面，有次我所负责的项目需要到一个完全连不了外网的服务器上部署，当时的我还不懂用docker，于是`nginx、MongoDB、Redis、node.js`等几种依赖的安装搞了一天！现在用上docker之后，把本地上的容器全打成镜像save起来，拷贝到别的服务器load后，服务就跑起来了，十分方便。
这个博客本身，也把所有能容器化的服务都容器化了，编排在`docker-compose`里面，对我的服务器而言，它能访问外网，我所做的，就只需要把git仓库拉下来，`docker-compose up`一下就可以了。附上这个博客的搭建docker：
[blog-docker地址](https://github.com/linguochi/blog-docker) 

