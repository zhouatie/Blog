---
title: docker从入门到实战-实战篇
date: 2019-07-07 01:34:00
categories: 前端
tags: ['docker']
thumbnail: https://s1.ax1x.com/2020/03/30/GeMSsJ.png
---

## 前言

本文是我通过三个星期业余时间学习后而写的文章，对 docker 的了解还处于入门阶段。希望本文能帮忙一些想学习 docker 的朋友快速入门。练习及实战代码都在 github 仓库中。如果我的文章能帮助到你的话，可以给我的[docker 项目](https://github.com/zhouatie/study-docker)点个赞哦

<!--more-->

## docker 实战

本次实战案例是 todolist。技术栈为 vue、node、mysql。具体代码见项目目录[todolist](https://github.com/zhouatie/study-docker/tree/master/todolist),下面就不一一贴代码了。就讲下重点。

下面我就顺着依赖关系来讲，所以先从 mysql 开始讲起

### 构建 mysql

执行：`docker run --name mymysql -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql`

- --name 给 mysql 容器设置匿名
- -d 表示后台运行
- -p 表示将容器的 3306 端口的映射到本地的 3308 端口，如果不设置的话，本地是无法访问该 MySQL 服务的。
- -e MYSQL_ROOT_PASSWORD 设置 root 的账号密码。
- mysql 后面不指定版本话，默认会 latest 版本

在执行该语句之前，假如你之前没有 pull 过 mysql 镜像。docker 在本地找不到你要的镜像就会帮你从 docker 仓库拉取 mysql:latest 镜像。

这个时候容器就启动成功了。

尝试下用 navicat 连接下试试

鼠标放入黄色小三角出现如下报错。

```javascript
2013 - Lost connection to MySQL server at 'reading initial communication packet', system error: 0 "Internal error/check (Not system error)"
```

这是因为 mysql8 以上，都会使用新的验证方式。

不妨查下信息: `select host,user,plugin,authentication_string from mysql.user;`

```javascript
mysql> select host,user,plugin,authentication_string from mysql.user;
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| host      | user             | plugin                | authentication_string                                                  |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| %         | root             | caching_sha2_password | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9                              |
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
)7k44VulAglQJgGpvgSG.ylA/rdbkqWjiqQJiq3DGsug5HIy3 |ord | $A$005$0pU+sGm[on
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
```

plugin 一栏可看到都是 caching_sha2_password。

那么如何才能将其改成可连接的呢？只需要将其 plugin 改成 mysql_native_password 就可以访问了。

`ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';`

你可以先用上面查询账户信息查下是否修改成功了。

修改成功后，可以尝试下用 navicat 连接下 mysql。不出意外的话就能成功连接上了。

当然我下面的例子用 mysql:5.6，方便操作，不需要修改 plugin。

执行命令：`docker run --name mymysql -d -e MYSQL_ROOT_PASSWORD=123456 -p 3308:3306 mysql:5.6`

启动容器后可以执行：`docker exec -it mymysql bash`进入容器

执行：`mysql -uroot -p123456`进入 mysql 控制台

执行：`show databases;`查看 mysql 数据库

```javascript
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
```

执行：`create database todolist;`创建 todolist 应用的数据库

执行：`show databases;`查看刚刚创建的 todolist 数据库

```javascript
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| todolist           |
+--------------------+
4 rows in set (0.00 sec)
```

可以看到数据库中多了个 todolist 数据库

接下来选择该 todolist 数据库

执行：`use todolist;`选中该数据库

创建表:

```javascript
CREATE TABLE list (
    id INT(11) AUTO_INCREMENT PRIMARY KEY,
    text VARCHAR(255),
    checked INT(11) DEFAULT 0
    );
```

执行：`show tables;`查看 todolist 数据库下的表

```javascript
mysql> show tables;
+--------------------+
| Tables_in_todolist |
+--------------------+
| list               |
+--------------------+
1 row in set (0.00 sec)
```

执行：`describe list;`查看表

```javascript
mysql> describe list;
+---------+--------------+------+-----+---------+----------------+
| Field   | Type         | Null | Key | Default | Extra          |
+---------+--------------+------+-----+---------+----------------+
| id      | int(11)      | NO   | PRI | NULL    | auto_increment |
| text    | varchar(255) | YES  |     | NULL    |                |
| checked | int(11)      | YES  |     | 0       |                |
+---------+--------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)
```

执行：`insert into list set checked = 0, text = 'haha';` 往表中插入一条数据；

执行：`select * from list;`

```javascript
mysql> select * from list;
+----+------+---------+
| id | text | checked |
+----+------+---------+
|  1 | haha |       0 |
+----+------+---------+
1 row in set (0.01 sec)
```

一切正常

### 构建 node

mysql 服务启动好了，接下来就是启动 node 服务，并连接刚启动的 mysql 服务了。

话不多说，直接上代码，解释看注释

```javascript
// index.js
const mysql = require('mysql'); // mysql包
const express = require('express');
const app = express();
const bodyParser = require('body-parser'); // post请求需要引入的包
app.use(bodyParser.json());

// mysql配置（用于连接刚启动的mysql服务）
const opt = {
	host: 'localhost',
	user: 'root',
	port: '3308',
	password: '123456',
	database: 'todolist'
};

const connection = mysql.createConnection(opt);

const sqlFn = sql => {
	return new Promise((resolve, reject) => {
		connection.query(sql, (err, results, filelds) => {
			if (err) throw err;
			resolve(results);
		});
	});
};

connection.connect(async err => {
	if (err) throw err;
	console.log('mysql connncted success!');
});

// todolist 列表查询
app.get('/getList', async (req, res) => {
	const sql = `SELECT * FROM list`;

	const data = await sqlFn(sql);

	res.json({
		code: 0,
		data,
		message: 'success'
	});
});

// todolist 插入数据
app.post('/insert', async (req, res) => {
	const sql = `INSERT INTO list SET checked = ${req.body.checked}, text = '${req.body.text}'`;
	const data = await sqlFn(sql);

	res.json({
		code: 0,
		data,
		message: 'success'
	});
});

app.listen(3000);
```

执行: `node index.js`后，控制台输入

```javascript
➜  server git:(master) ✗ node index.js
mysql connncted success!
```

表示 node 服务连接 mysql 服务成功；

浏览器可以访问下`localhost:3000/getList`

```javascript
{"code":0,"data":[{"id":1,"text":"haha","checked":0}],"message":"success"}
```

页面将会出现刚才我们 sql 插入到数据库的数据

既然代码没有问题，那么我们接下来就把他构建成镜像。

**构建之前需要把代码中 opt 的 host localhost 改为自己主机的 ip。因为容器启动的话，连接 mysql 需要通过主机的 3308 端口访问。**

在当前文件夹新建名为 Dockerfile 的文件

```sh
# 基于最新的 node 镜像
FROM node:8
# 复制当前目录下所有文件到目标镜像 /app/ 目录下
COPY . /todolist/server
# 修改工作目录
WORKDIR /todolist/server
# 安装依赖
RUN ["npm", "install"]
# 启动 node server
ENTRYPOINT ["node", "index.js"]

```

执行：`docker build -t mynode .`，生成 node 镜像

- -t:表示给该镜像添加版本，这里不写默认为 latest

可通过`docker images`查看本地的镜像。

```javascript
➜  server git:(master) ✗ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mynode              latest              3e8de2825063        4 seconds ago       898MB
```

可以看到第一个镜像就是我们刚刚构建的镜像

```javascript
➜  server git:(master) ✗ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mynode              latest              3e8de2825063        4 seconds ago       898MB
```

接下来运行基于这个镜像的容器

执行：`docker run --name mynode -d -p 4000:3000 mynode`

- --name 给 node 容器起一个容器匿名
- -d 表示后台运行
- -p 4000:3000 表示访问本地 4000 代理容器内的 3000 端口服务
- mynode 是上面我们 build 构建的镜像

启动成功后，访问下`localhost:4000/getList`

```javascript
{"code":0,"data":[{"id":1,"text":"haha","checked":0}],"message":"success"}
```

可以看到页面输出我们上面执行 sql 语句插入的数据

上面贴的代码只是基础的查看列表与插入数据两个方法，其他的请参考[server](https://github.com/zhouatie/study-docker/tree/master/todolist/server/index.js)

### 构建 vue

todolist 的静态页面，我是通过 vue-cli3 搭建的。
执行`vue create app` 创建项目。

进入项目根目录执行`npm run serve`页面是否正常执行。

然后编写简易的具有增删改查的 todolist 应用。具体代码见[todolist](https://github.com/zhouatie/study-docker/tree/master/todolist)。

**注意 vue.config.js 中的 devServer 配置的 target: 'http://127.0.0.1:4000'代理到我们刚启动的node容器**

页面启动成功后，访问`localhost:8080`

可以看到页面加载成功，列表也成功渲染出上面构建 mysql 时，sql 插入的数据。

本地启动静态并请求服务端成功后，接下来也将静态页面打包成镜像，然后启动静态页面容器

**在打包镜像容器之前，记得先将 vue.config.js 中的 devServer 配置的 target: 'http://<主机 ip 地址>:4000'代理到我们刚启动的 node 容器**

编写 Dockerfile

```shell
# 基于最新的 node 镜像
FROM node:8
# 复制当前目录下所有文件到目标镜像 /app/ 目录下
COPY . /todolist/app
# 修改工作目录
WORKDIR /todolist/app
RUN npm config set registry https://registry.npm.taobao.org && npm install
# RUN ["npm", "install"]
# 启动 node server
ENTRYPOINT ["npm", "run", "serve"]
```

cd 到静态页面的根目录执行: `docker build -t static .`

执行：`docker run --name static -d -p 9000:8080 static`启动静态容器

打开浏览器访问`localhost:9000`,可以看到页面成功渲染出列表页。

至此，mysql、node、vue 容器均已互通。代码需要完善的地方详见[todolist](https://github.com/zhouatie/study-docker/tree/master/todolist)

### docker-compose

我们不可能每次部署一个应用，需要手动启动好几个服务。这个时候就需要使用[docker-compose](https://yeasy.gitbooks.io/docker_practice/content/compose/introduction.html)

关于命令就不介绍了，这里贴下链接
[docker-compose 命令](https://yeasy.gitbooks.io/docker_practice/content/compose/commands.html)

在根目录下新建`docker-compose.yml`配置文件

```shell
version: '2'
services:

  static:
    build: ./app/
    container_name: static
    ports:
     - 7001:8080
    depends_on:
      - nodejs
      - db

  nodejs:
    build:
      context: ./server/
      dockerfile: sleep-dockerfile
    container_name: nodejs
    ports:
      - 4000:3000
    environment:
      - IS_START_BY_COMPOSE=1
    command: sh ./sleep.sh
    depends_on:
      - db

  db:
    image: mysql:5.6
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
```

具体配置，详见上面贴的链接。下面我介绍下我所写的配置

- version: docker-compose 的版本
- services: 代表你用 docker-compose 启动的几个服务
- build: 指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
- container_name: 指定容器名称。
- ports: 相当于 docker run 启动容器的-p，<主机 ip:容器 ip>，这样主机通过这个端口去访问容器中的服务。
- environment: 只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。简而言之，就是在容器内可以获取给的环境变量。
- depends_on: 解决容器的依赖、启动先后的问题。（但是它有个问题就是只是等所依赖的服务开启启动，并不是等依赖的服务启动成功后在构建当前的服务。
- command: 覆盖容器启动后默认执行的命令

接下来解释下上面用 compose 启动的服务

> 值得一提的是，mysql 的 Dockerfile 中做了创建数据库与表的操作。会涉及到关闭数据库密码登录功能。因为关闭之后，操作数据库就不需要输入密码了。等表建完之后在恢复密码。
> nodejs 中的 sleep.sh 脚本是因为 depends_on 这个依赖只是单纯的等待其他服务开始启动，并不是等待依赖的服务启动完成之后才开始构建自身的服务。这个时候 node 初始化会在 mysql 启动完成之前启动。这样的话，node 启动的时候连接 mysql 就会报错，导致 node 服务挂掉。所以引用了 sleep.sh，让 node 延迟一段时间启动。

最后在`docker-compose`文件所在的目录下执行`docker-compose build`

再执行：`docker-compose up`就可以启动 todolist 这个应用的所有服务了。
