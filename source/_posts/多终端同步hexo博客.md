---
title: 多终端同步hexo博客
date: 2017-3-21 16:34:56
author: Peter pan
categories: 多终端同步
tags: hexo github
---

多终端同步hexo
<!--more-->









使用hexo是一个很棒的体验，但是由于它是一个静态博客，优点与缺点共存，它存活的生命周期更长，但没有和由于数据库连接，所以想要在多机上更新自己的博客会显得有些麻烦，但是事实上只要回想一下当初搭建时候的步骤，此时便会简化很多。而且有[多种方案可供选择](https://www.zhihu.com/question/21193762)

重部署流程
>因为新电脑上面已经完全没有了hexo的环境，所以要重新部署一下。
>1.先安装 [Node.js](http://nodejs.cn) 和 [Git](https://git-scm.com/downloads)，再安装完这两者后，再通过之前的步骤安装 Hexo。
>2.在public下创建一个文件夹把整一个your_name.github.io 文件夹同步过来cd 到同步的 your_name.github.io 内， 然后再安装 Hexo：

```bash
   npm install -g hexo-cli
```
 **为了部署，还要执行**：


```bash
   npm install hexo-deployer-git --save
```
至此，在新的平台上运行 hexo clean，hexo generate ，hexo server 等命令应该就没有问题了。

Github配置
Step 1：Setting up Git。在 Git Bash 中执行如下代码即可：
```bash
   git config --global user.name "YOUR NAME" #YOUR NAME 是自己取的名字
   git config --global user.email "YOUR EMAIL ADDRESS" #YOUR EMAIL ADDRESS 是自己的 Github 邮箱
```

Step 2：Authenticating with GitHub from Git。
在 Git Bash 下执行如下命令，生成 SSH key
```bash
   ssh-keygen -t rsa -b 4096 -C     "your_email@example.com"
   #your_email@example.com 是你的 Github 注册邮箱，剩下的一路回车即可。
```
接下来将其保存在主目录下，其中id_rsa是你的私钥，不要被他人知道，你所要做的是把那个后缀.pub的公钥添加到你的github账户

将 SSH key 添加到 Github 账户
在 Git Bash 中将 SSH Key 拷贝出来：
```bash
    clip < ~/.ssh/id_rsa.pub
```
此时，SSH Key 已经在我们的剪切板里了。然后登录 Github 帐号，依次点击自己的头像，Settings，SSH and GPG keys， Add SSH key， 在 Title 这里输入 Key 的label，比如 your_name - PC，然后在 Key 里面把 SSH Key 粘贴进去，点击 Add SSH key 大功告成。
测试 SSH 连接，在 Git Bash 中敲入
```bash
   ssh -T git@github.com
```
应该可以看到提示你成功的信息。
Hexo 部署：执行hexo deploy 应该就可以大功告成了。



另外附上一种高手方法，新手慎用：[自动部署更新博客](https://github.com/zjhou/E2P)



​        