---
title: 基于react、socket.io、node.js仿微信开发
date: 2018-01-21 23:52:24
tags: ['react', 'node', 'socket.io']
categories: 前端
thumbnail: https://s1.ax1x.com/2020/03/30/GZzE28.png
---

# 前言

这个项目是我自学 react+redux 的第一个项目，并结合自己之前所学的 node+mongodb，来模仿开发微信客户端。利用每天下班时间边学习边写。由于本人技术水平有限，比较适合新手。目前还没有写完。喜欢的话可以帮忙给我 github 点个 star ^\_^

<!-- more -->

## 项目地址

https://github.com/zhouatie/wechat

## 技术栈

react+redux+react-router4+socket.io+axios+node.js+mongodb

## 说明

```
本地启动mongodb服务
分别进入wechat跟server文件夹npm install
wechat里npm run start
server里node app.js 和 chat.js 这两个文件
```

> 开发环境：macbook pro 、vscode、Chrome、node

> 如果 npm install 太慢导致有些 npm 依赖包下载失败 你可以看控制台的报错信息，再手动 npm install 具体的开发包，推荐使用淘宝的注册源，直接运行

`npm install -g cnpm --registry=https://registry.npm.taobao.org`

## 目标功能

- [√] 注册
- [√] 登录
- [√] 添加好友
- [√] 支持私聊
- [√] 消息列表的展示
- [√] 未读消息数量的显示
- [√] axios 数据跨域的设置
- [ ] 群聊
- [√] 上传头像
- [√] 个人信息的编辑
- [ ] 朋友圈

## 部分截图

![私聊](https://github.com/zhouatie/wechat/raw/master/data/wechat_2018-01-14.gif)

![上传头像](https://github.com/zhouatie/wechat/raw/master/data/uploadLogo2018011501.gif)

## 总结

1.之前写 vue 项目的时候，在 main.js 文件中写 express 接口，就行了，就不存在跨域问题。在 create-react-app 启动的静态资源服务中，实在找不到哪里可以写接口，找了好久的 node_modules ，都不知道在哪里下手。好在 create-react-app 中的 package.json 中加上：`proxy:http://localhost:4000`就能解决跨域问题了。

2.在 app.js 页面中，使用的是 express 框架，写 socket.io 不知道为什么会提醒跨域问题，而我前面的登录接口用 axios 跨域就没有问题，而且我在 express 的头部做了 CORS 处理，还是存在跨域问题。所以只能另启了一个 node 服务，采用原生 node.js 编写，跨域就成功了。但是我在新写的服务中，换成用 express 框架，结果也提示了存在跨域问题。目前个人猜测 express 可能有什么跨域机制。

3.在引入 react-router4 的时候遇到了很多疑难杂症，晚上大部分的 react-router4 一下的版本。按照网上来做，好多报错，到处找博客找文章。后来通过 react-router 英文文档的阅读解决了各种报错问题。

4.我是通过 redux 来更新消息列表，中间出现 store 数据更新了，组件却不渲染。后来求助好友后，原来是我强制修改了 state 导致页面无法即使刷新。

5.formdata 上传文件，相当于表单上传，头部为`Content-Type:multipart/form-data`,这点要注意了！

> 注意: Multer 不会处理任何非 multipart/form-data 类型的表单数据。具体见 [multer](https://www.npmjs.com/package/multer)

```javascript
var multer = require('multer');
var upload = multer({ dest: '../wechat/public/logos' }); // dest 指的是图片存到哪个文件夹里

// 上传头像
app.post('/uploadLogo', upload.single('avatar'), (req, res) => {
	User.update(
		{ _id: req.body.id },
		{ $set: { logo: './logos/' + req.file.filename } },
		function() {
			res.send({
				status: 'success',
				url: './logos/' + req.file.filename
			});
		}
	);
});
```

## 参考资料

《深入浅出 React 和 Redux》-- 程墨

《MongoDB 实战（第二版）》

[react-router](https://reacttraining.com/react-router/web/guides/philosophy)

[react](https://reactjs.org/docs/hello-world.html)

[redux 中文文档](http://www.redux.org.cn/index.html)

[mongoose](http://www.nodeclass.com/api/mongoose.html#guide_connections)

[基于 Vue、Nodejs、Socket.io 的聊天应用](https://juejin.im/entry/5923e2242f301e006b2a7827)

[multer](https://www.npmjs.com/package/multer)

> 文章都是学习过程中的总结，如果发现错误，欢迎留言指出
