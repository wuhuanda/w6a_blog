---
title: Hexo+阿里云+docker+nginx保姆级搭建教程
date: 2021-03-27 23:21:13
category:
- 教程
tags: 
- Hexo
- 阿里云
- docker
- nginx
---

# 前言

其实很早之前我就想要搭建一个自己的个人站了，但是由于种种原因（主要还是懒癌发作了orz）导致我一直没有去搭建。期间我也尝试过使用三方作为个人站，比如在掘金上发表文章，亦或是用 Notion 做一个“伪·个人站”。掘金的话毕竟属于一个社区，所以发表的文章需要考虑到平台的影响，有很多东西其实并不太适合往上面放。至于 Notion 的话，其一是 Notion 其实是比较偏向团队协作和个人空间的定位，不太适合做全站分享，其二是因为目前它并未在国内架设服务器，东西是好东西，就是使用条件有点苛刻，懂的自然懂。

然后今年开始上网课了，觉得需要总结输出一些东西，有个个人站的话会比较方便。对比了市面上几款个人博客搭建工具最终还是选择了 Hexo，至于几款工具的对比这里就不展开了，我觉得既然你都来看这篇文章了，肯定也是希望搭建一个 Hexo 博客的哈哈哈哈。

Hexo 是基于 node.js 的，搭建起来也比较简单。接下来就跟着我一步步搭建一个属于自己的个人博客吧~

# 开始搭建 Hexo

1. 上面也说了 Hexo 是基于 node.js 框架的，所以我们需要现拥有一个 node 环境。 
node.js 下载地址：[node.js](https://nodejs.org/zh-cn/download/)

2. 安装完成后，我们打开命令窗口，输入 `node -v`，如果返回下图所示，那么 node 就被完美安装成功啦~
![](https://file.clovemu.com/img/20210406083941.png?imgslim)

3. 接下来安装 Hexo
```bash
npm install hexo-cli -g
```

4. 初始化博客
```bash
hexo init my_blog 
```

5. 初始化完成只有切换到刚才创建的 Hexo 目录
```bash
cd my_blog
```

6. 安装依赖
```bash
npm install
```

7. 运行 Hexo
```bash
hexo clean g s
```
> 具体的指令可以在[官方文档](https://hexo.io/zh-cn/docs/commands)中查询

8. 打开浏览器，在地址栏输入 `localhost:4000` 就可以访问到刚搭建好的博客啦！
![](https://file.clovemu.com/img/20210406083958.png?imgslim)

# 将博客公之于众

经过刚才上面的一番折腾，此时我们已经能够自己看到自己的博客了，然而别人却无法看到。很显然，这并不是我们想要的结果，所以我们需要借助某些东西让每个人都能够来欣赏我们的个人博客。本文以阿里云为例，当然你也可以使用其他的云甚至内网穿透都可以（只要你能保证安全 @w@）。

# 开始搭建阿里云环境

首先我们需要一台阿里云主机，其次我们选择一个环境，这里以 CentOS7 为例。

如果服务器没有安装 git 的话，我们需要安装 git。

# 建立 git 仓库

Hexo 可以使用 git 来部署，所以我们可以用 git 来实现一键部署。

切换到 root 用户，然后操作如下：
```sh
useradd git # 添加一个新用户
passwd git # 设置 git 用户的密码
su git # 切换到 git 用户进行后续操作
cd /home/git/
mkdir -p projects/blog # 把项目目录建立起来
mkdir repos && cd repos
git init --bare blog.git # 建立裸库
cd blog.git/hooks
vi post-receive # 创建一个钩子
```

`post-receive` 文件内容如下：
```sh
#!/bin/sh
DIR=/home/git/projects/blog
GIT_DIR=/home/git/repos/blog.git
rm -rf ${DIR}
git clone ${GIT_DIR} ${DIR}
```

保存并退出vim，继续如下操作：
```sh
chmod +x post-receive # 添加可执行操作
exit # 返回到 root 用户
chown -R git:git /home/git/repos/blog.git # 给 git 用户添加权限
```

这样仓库就配置好了，在本地试下能不能把仓库拉下来：
```bash
git clone git@server_ip:/home/git/repos/blog.git
```

如果能拉下来，就说明配置成功了！

# 本地免密登录 SSH

为了避免每次对 git 仓库进行操作时都需要输入密码，我们可以通过以下操作实现免密登录 SSH

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub git@server_ip # 建立信任关系
ssh git@server_ip # 免密登录
```

# 更改 git 用户默认的 shell

为了安全起见，我们应该只允许 git 用户拥有指令操作的权限而不该有登录服务器的权限，所以我们需要修改它默认的 shell

```sh
cat /etc/shells # 查看 git-shell 是否在登录方式里面
which git-shell # 找到 git-shell 的路径
vi /etc/shells
```

然后把刚才找到的 git-shell 路径添加进去，保存。

然后 `vi /etc/passwd`， 把 `git:x:1000:1000::/home/git:/bin/bash` 修改为 `git:x:1000:1000:,,,:/home/git:/usr/bin/git/git-shell`。

如此一番操作下来，git 用户就只能操作 git 等操作而不能通过 ssh 连接上服务器啦。
