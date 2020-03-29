---
title: 配置nginx解决vue路由history模式下刷新404问题
date: 2018-09-11 01:08:44
tags: ['nginx']
categories: 前端
thumbnail: https://s1.ax1x.com/2020/03/30/GemqIK.png
---

在 vue 路由模式为 history 的时候，刷新页面会出现 404 问题。我们只需要在服务器配置如果 URL 匹配不到任何静态资源，就跳转到默认的 index.html。

<!-- more -->

```javascript

server {
        listen 8105; // 表示你nginx监听的端口号
        root /home/admin/sites/vue-nginx/dist; // vue打包后的文件夹dist目录
        index index.html;
        location / {
                 try_files $uri $uri/ /index.html;
        }
}

```
