> 在系列文章的[第一篇](https://juejin.im/post/5c07fa04e51d451de968906b)我们谈到过跨域问题产生的原因主要是浏览器的同源策略. 既然同源策略是浏览器的专属, 服务器不会受到相关条件的限制. 那么是不是我们可以通过服务器代理访问其他域名的 api 呢? 如果可以的话, 那岂不是可以想访问谁就访问谁了? 限制, 不存在的...

![2018-12-12-23-32-26](http://img.blog.niubishanshan.top/2018-12-12-23-32-26.png)

ps: 本文涉及到部分后端知识, 需要有一丢丢的 `nodejs` `koa` 基础. 主要用于搭建一个 `web` 服务器, 当然没有基础也没啥关系, 先去 [node](http://nodejs.cn/) [koa](https://koajs.cn/#) 官网看看. 😄

## 创建项目目录

继续上一步, 由于本次我们的前端项目会包含到后端项目中. 所以不需要在 `./fe` 目录下创建前端项目啦, 项目目录如下.

![2018-12-12-23-50-07](http://img.blog.niubishanshan.top/2018-12-12-23-50-07.png)

其中, `serverProxy` 目录表示本次项目的主目录. 其中包含的 `www` 目录即为前端项目的托管目录. `base.js` 为后端主程序, `add.js subtract.js` 分别表示两个第三方服务, 分别提供了计算加法和减法的能力.

## 安装 koa

- 首先执行 `cd be/serverProxy` 将路径切换到 `serverProxy`
- 执行 `npm init -y` 初始化项目
- 执行 `npm i koa -S` 完成 `koa` 安装

验证 `koa` 安装是否完成

- 编辑 `base.js` 写入以下内容

```js
const Koa = require('koa');

const app = new Koa();
const PORT = 1234;

app.use((ctx) => {
  ctx.body = 'Hello World';
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

- 完成后执行 `node base.js` 看到命令行中输出了 `the server is listen:  1234` 说明启动成功
- 浏览器访问 [localhost:1234](http://localhost:1234/)

![2018-12-13-00-05-48](http://img.blog.niubishanshan.top/2018-12-13-00-05-48.png)

此时[代码](https://github.com/luoquanquan/cross-domain/commit/0f610b65d366d75f243c97c10d26b780da2c2cfb)

## 引入 `koa-static` 模块

在之前的几种跨域方式中, 我们总是要通过 `live-server` 启动一个本地的静态资源服务. 保证前端项目代码可以通过 `localhost` 的方式访问. 其实在 `koa` 生态中有现成的库(`koa-static`)可以帮助我们直接在后端项目中创建静态资源服务.

- 首先执行 `npm i koa-static -S` 安装 `koa-static`
- 调整 `base.js`

```js
const Koa = require('koa');

// 引入 koa-static
const koaStatic = require('koa-static');

const app = new Koa();
const PORT = 1234;

// 使用 koa-static 中间件, 并指定静态文件目录为 www
app.use(koaStatic(`${__dirname}/www`));

app.use((ctx) => {
  console.log(ctx.req.url);
  ctx.body = 'Hello World';
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

- 编写前端 `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>ServerProxy 实现跨域</title>
</head>
<body>
    ServerProxy 实现跨域
</body>
</html>
```

- 执行 `node base.js` 重启后端项目
- 浏览器打开 [localhost:1234/index.html](http://localhost:1234/index.html)

![2018-12-13-00-26-23](http://img.blog.niubishanshan.top/2018-12-13-00-26-23.png)

