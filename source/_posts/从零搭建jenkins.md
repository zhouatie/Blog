---
title: docker从零搭建jenkins
date: 2020-04-02 04:16:41
tags: ['docker', 'jenkins']
categories: 持续集成
thumbnail: https://s1.ax1x.com/2020/04/02/GGPu79.jpg
---

## 前言

写这篇文章的灵感来源于最近公司的 jenkins 部署老是失败，各种原因。在项目非常赶的情况下，我每天还要抽半天时间去排查 jenkins 上的问题。所以决定在我们前端服务器上搭建个 jenkins 给测试部署。并部署到前端服务器上。文章是边操作边写出来的，踩遍了坑。不知道大家会不会也遇到这些问题。反正我都把解决步骤写在里面了。

本文主要内容是介绍 jenkins 的搭建与使用。至于是安装在服务器上还是本文通过 docker 安装 jenkins 不是很重要，默认读者会使用 docker。如果不是很了解`docker`可看我的[docker 从入门到实战](https://zhouatie.github.io/blog/2019/07/07/docker从入门到实战-基础篇/)博客

<!-- more -->

## 安装 jenkins

`docker pull docker.io/jenkins/jenkins:latest`

安装成功后使用`docker images`查看镜像

```shell
github docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jenkins/jenkins     latest              59f8784e08ee        2 days ago          619MB
```

在启动 Jenkins 前，需要先创建一个 Jenkins 的配置目录，并且挂载到 docker 里的 Jenkins 目录下

新建一个文件夹如我的`/Users/zhouatie/Desktop/github/front-end/practise-jenkins`

并给该文件夹授权`sudo chown -R 1000 /Users/zhouatie/Desktop/github/front-end/practise-jenkins`

> 这里有个很神奇的点就是网上都说要授权，所以我授权了，但是还是提示`Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions`。后来我新建了个文件夹不做授权处理就可以了。可查阅[stackoverflow](https://github.com/jenkinsci/docker/issues/177) 这里面一位朋友就是遇到相同的问题 授权了也没用。

执行以下命令构建容器

```shell
docker run -itd -p 8080:8080 -p 50000:50000 --name jenkins  -v /Users/zhouatie/Desktop/github/front-end/practise-jenkins:/var/jenkins_home docker.io/jenkins/jenkins:latest

```

执行`docker ps`查看后台启动的容器情况

```shell
➜  front-end git:(master) ✗ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                              NAMES
3385ad0e42fe        jenkins/jenkins:latest   "/sbin/tini -- /usr/…"   5 minutes ago       Up 5 minutes        0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
```

可以看到 jenkins 容器已经启动成功了。这个时候访问下页面吧。浏览器输入 `localhost:8080`

可以看到如下界面

<img src="https://s1.ax1x.com/2020/04/02/G8xocQ.jpg" alt="G8xocQ.jpg" style="zoom:50%;" />

这个时候我们就可以到刚才管理数据卷的文件夹里找了(我本地是`/Users/zhouatie/Desktop/github/front-end/practise-jenkins/secrets/initialAdminPassword`)，`cat`下这个文件可以看到输出`28023d3751214bd6aadc0dd83c168325`，把这个密码复制到管理员密码输入框中并点击继续。

loading 转了半天，有种不详的预感。结果不出意外显示 jenkins 离线。所以我又开始上网搜[新版本 jenkins 安装时显示离线问题](https://yq.aliyun.com/articles/347536)

解决步骤

1. 浏览器输入`http://localhost:8080/pluginManager/advanced`

   划到最下面可以看到

   [![G8zfbR.md.jpg](https://s1.ax1x.com/2020/04/02/G8zfbR.md.jpg)](https://imgchr.com/i/G8zfbR)

2. 将截图中的地址替换为`http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json`这个代理服务器

再刷新下`http://localhost:8080`页面如下

![G8zvVI.jpg](https://s1.ax1x.com/2020/04/02/G8zvVI.jpg)

3. 点击安装推荐的插件

   结果不出意外如下图

   ![GGpocD.jpg](https://s1.ax1x.com/2020/04/02/GGpocD.jpg)

   然后我又开始`google`了。找到了这个[安装 jenkins 时出现 No such plugin: cloudbees-folder 的解决办法](https://www.cnblogs.com/changjianblog/p/10916098.html)

   结果按照前几个步骤失败，作者建议重启，好吧，重启就重启，执行下 `docker restart jenkins`

   终于又成功进入下载页了。

   > 安装真的超级慢，心急如焚，不知道是不是 docker 的原因。因为文章是直接边操作边写的。在想要不要直接跨过这个安装界面，直接打开自己跑在前端服务器上的 jenkins 了开始介绍了

![GGpdkq.jpg](https://s1.ax1x.com/2020/04/02/GGpdkq.jpg)

还没等他提示完全失败，这个时候我就又开始`google`了。实在没辙了，这的太难了。所以我重启了个`jenkins`容器后，选择自选插件。然后什么也不选，进了页面后，可以在如下截图地方下载，我是将上面推荐的全部勾选后进行下载，结果还很快。

![GGCv6S.jpg](https://s1.ax1x.com/2020/04/02/GGCv6S.jpg)

4. 安装完插件后重启下，可以看到界面如下

   ![GGPpwj.jpg](https://s1.ax1x.com/2020/04/02/GGPpwj.jpg)

## 构建

1. 点击右上角的新建任务

<img style="max-width: 50%;float: left;" src="https://s1.ax1x.com/2020/04/02/GGP9Ts.jpg" alt="GGP9Ts.jpg" border="0" />

1. 选择第一个自由风格模式

   <img style="max-width: 50%;float: left;" src="https://s1.ax1x.com/2020/04/02/GGPPkn.jpg" alt="GGPPkn.jpg" border="0" />

2. 确点后进入如下页面，并点击配置

   <img style="max-width: 50%;float: left;" src="https://s1.ax1x.com/2020/04/02/GGPV6U.jpg" alt="GGPV6U.jpg" border="0" />

3. 因为部分插件装失败了，我就以在装成功的 jenkins 配置界面截图为例

   <img src="https://s1.ax1x.com/2020/04/02/GGPmm4.jpg" alt="GGPmm4.jpg" border="0" />

- **参数化构建**：这里主要提下参数化构建，这里对应的值都可以在下面【构建】执行 shell 中获取到部署的时候用户手动选择或者填入的参数。
- **源码管理**：主要是让 jenkins 从你的 git 仓库中拉代码，`credentials`需要选择有该仓库权限的账号，可以手动试下
- **构建触发器**：意思就是触发条件，比如`git`上的`webhook`，就可以触发`jenkins`部署。具体可查阅`google`
- **构建**：这个是重点，这里可以执行你的脚本，比如你是一个 vue 项目，可以根据上面配置的参数化配置，获取是否需要安装依赖等。可看我图中`shell`脚本，非常好理解。
