系列文章:

- [【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
- [【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)
- [【小哥哥, 跨域要不要了解下】CORS 进阶篇](https://juejin.im/post/5c0b5a8851882548e9380afb)
- [【小哥哥, 跨域要不要了解下】NGINX 反向代理](https://juejin.im/post/5c0e6d606fb9a049f66bf246)
- [【小哥哥, 跨域要不要了解下】ServerProxy](https://juejin.im/post/5c153c456fb9a049ca373eed)

> 在系列文章的[第一篇](https://juejin.im/post/5c07fa04e51d451de968906b)我们谈到过跨域问题产生的原因是**浏览器**的同源策略. 那么服务器之间通信就不会受到相关条件的限制. 那么是不是我们可以通过同域服务器帮助访问其他域名的 api 呢? 如果可以的话, 那岂不是可以想访问谁就访问谁? 限制, 不存在的...

![2018-12-12-23-32-26](https://user-gold-cdn.xitu.io/2018/12/16/167b2f356678235e?w=714&h=548&f=png&s=283097)

ps: 本文涉及到部分后端知识, 需要有一丢丢的 `nodejs` `koa` 基础. 主要用于搭建一个 `web` 服务器, 当然没有基础也没啥关系, 先去 [node](http://nodejs.cn/) [koa](https://koajs.cn/#) 官网看看. 回不回来??? 

随你咯 😄

## 创建项目目录

继续上一步, 本文只会创建一个后端项目. 所以不需要在 `./fe` 目录下创建前端项目啦, 项目目录如下.

![2018-12-12-23-50-07](https://user-gold-cdn.xitu.io/2018/12/16/167b2f3562c95617?w=588&h=1270&f=png&s=96963)

其中, `serverProxy` 目录是项目的主目录. `www` 目录即为前端静态文件的托管目录. `base.js` 为后端主程序, `add.js subtract.js` 分别表示两个第三方服务, 分别提供了计算加法和减法的能力.

## 安装 koa

- 首先执行 `cd be/serverProxy` 将路径切换到 `serverProxy`
- 执行 `npm init -y` 初始化为一个 node 项目
- 执行 `npm i koa -S` 完成 `koa` 的安装

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
- 浏览器访问 [localhost](http://localhost:1234/)

![2018-12-13-00-05-48](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35632a07a3?w=2870&h=152&f=png&s=45483)

此时[代码](https://github.com/luoquanquan/cross-domain/commit/0f610b65d366d75f243c97c10d26b780da2c2cfb)

## 引入 `koa-static` 模块

在之前文章中, 我们总是要通过 `live-server` 启动一个本地的静态资源服务. 用于托管前端静态文件. `koa` 生态中有现成的中间件[`koa-static`](https://www.npmjs.com/package/koa-static)可以提供直接在后端项目中创建静态资源服务的能力.

- 首先执行 `npm i koa-static -S` 安装 `koa-static`
- 调整 `base.js`

```js
const Koa = require('koa');

// 引入 koa-static
const koaStatic = require('koa-static');

const app = new Koa();
const PORT = 1234;

// 使用 koa-static 中间件, 并指定静态文件目录为 www
app.use(koaStatic('./www'));

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

- 执行 `node base.js` 重启项目
- 浏览器打开 [localhost:1234/index.html](http://localhost:1234/index.html)

![2018-12-13-00-26-23](https://user-gold-cdn.xitu.io/2018/12/16/167b2f3563d4ed74?w=2876&h=232&f=png&s=65606)

之前准备的 html 页面赫然在目 😄. 至此, 静态文件服务就搭建成功了(相当于我们自己实现了一个 `live-server`)[代码地址](https://github.com/luoquanquan/cross-domain/commit/b8780f2cc32fc4ba55bd90bbadfbb1a603db00f4)

## 通过 ajax 访问当前后端接口

通过 `koa-static` 中间件, 我们搭建了一个自己的静态文件服务器. 接下来演示一个不跨域的请求...

![2018-12-15-21-51-02](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35f606f7e8?w=582&h=556&f=png&s=528743)

- 首先修改后端代码

```js
const Koa = require('koa');
const koaStatic = require('koa-static');

const app = new Koa();
const PORT = 1234;

app.use(koaStatic('./www'));

app.use((ctx) => {
  let ret;
  // 获取本次接收的请求的请求路径
  const path = ctx.req.url;

  // 如果请求路径以api开头, 那么作为接口请求处理
  if (path.startsWith('/api')) {
    // 这样实现的路由不是很优雅, 但是能用 😂
    switch (path) {
      case '/api/getFriend':
        ret = { name: 'quanquan', friend: 'gl' };
        break;
      default:
        ret = { errno: 1, errmsg: '未知接口' };
        break;
    }
  }
  ctx.body = ret;
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

上述代码中定义了 `/api/getFriend` 接口, 通过浏览器访问的如下图:

![2018-12-15-22-09-17](https://user-gold-cdn.xitu.io/2018/12/16/167b2f356cf1e445?w=2120&h=422&f=png&s=72974)
ps: 需要执行 `node base.js` 重启后端项目

接下来修改前端代码. 通过 ajax 的方式访问该接口

修改前端代码:

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

    <script>
        // 一个常规的 ajax, 感兴趣的兄弟们也看看. 手写 ajax 好多面试官还在考
        var xhr = new XMLHttpRequest()
        xhr.open('GET', '/api/getFriend')
        xhr.onreadystatechange = function() {
            if(xhr.readyState === 4 && xhr.status === 200) {
                console.log('接口返回的数据为: ', xhr.responseText)
            }
        }
        xhr.send()
    </script>
</body>
</html>
```

刷新浏览器, 控制台展示如下. 没有报错, 返回的信息前端直接拿到了.
![2018-12-15-22-18-18](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35702ad24c?w=1764&h=540&f=png&s=78884)

原来前后端同域时数据交互这么的简单.
![2018-12-15-22-22-07](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35f6e036b1?w=700&h=580&f=png&s=226830)

前后端跑通阶段[代码](https://github.com/luoquanquan/cross-domain/commit/9b53deab8d22ca5558185347b7f0587db3f443c8)

## 完善第三方服务

项目开发中经常会用到一些基础服务, 比如天气信息, 地理位置信息等等. 这些服务能力一般是通过调用第三方的接口来实现的(你开发一个网站, 先发射一颗气象卫星到天上也不太现实). 这一步我们创建两个第三方服务, 分别提供加法和减法运算.

加法运算服务 `add.js`

```js
const Koa = require('koa');

const app = new Koa();
const PORT = 1111;


app.use((ctx) => {
  // 获取参数
  const { a, b } = ctx.query;
  // 尝试将参数转化为数字后进行加法操作
  const result = Number(a) + Number(b);
  ctx.body = { result };
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

执行命令 `node add.js` 启动程序, 然后浏览器端访问[localhost](http://localhost:1111/?a=1&b=2)得到的结果如下, 说明加法计算服务启动成功.
![2018-12-15-23-18-53](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35f6e0ec45?w=1934&h=380&f=png&s=56429)

减法运算服务 `subtract.js`

```js
const Koa = require('koa');

const app = new Koa();
const PORT = 2222;


app.use((ctx) => {
  // 获取参数
  const { a, b } = ctx.query;
  // 尝试将参数转化为数字后进行减法操作
  const result = Number(a) - Number(b);
  ctx.body = { result };
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

执行命令 `node subtract.js` 启动程序, 然后浏览器端访问[localhost](http://localhost:2222/?a=10&b=2)得到的结果如下, 说明减法计算服务启动成功.
![2018-12-15-23-23-55](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35ff9f48e3?w=2010&h=384&f=png&s=60088)

目前[代码](https://github.com/luoquanquan/cross-domain/commit/feaad9184ab5c6c472ab6196b027634254739ee3)

## 通过后端代理访问第三方服务

创建完加法和减法服务, 我们还是有侥幸心理忍不住在前端项目里访问一下试试, 万一能通了呢? 就不用费事儿研究跨域了, 尝试一下

![2018-12-16-00-57-43](https://user-gold-cdn.xitu.io/2018/12/16/167b2f35faecd16b?w=550&h=554&f=png&s=444919)

修改前端代码中的接口地址 `xhr.open('GET', 'http://localhost:1111/?a=1&b=2')` [完整代码](https://github.com/luoquanquan/cross-domain/commit/3259860ab2c02b9f3df33e42dccb6a26281caad2), 之后直接刷新浏览器(请思考, 为什么修改了 js 文件需要执行 `node ...`  重启服务, 而修改了 html 文件只需要刷新浏览器就可以了呢?).

![2018-12-15-23-34-33](https://user-gold-cdn.xitu.io/2018/12/16/167b2f3600c1ae70?w=2558&h=638&f=png&s=135507)
还是之前的报错, 还是熟悉的味道. 不好使...

回想一下之前的思路. 浏览器有同源策略的限制服务器没有. 我们的前端项目托管在后端项目中所以访问我们自己的后端不跨域. 我们的后端请求第三方服务没有限制. 那么 ^_^

![2018-12-15-23-41-03](https://user-gold-cdn.xitu.io/2018/12/16/167b2f3635478189?w=440&h=590&f=png&s=134468)

- 执行 `npm i axios -S` 安装 axios, 后端通过它来请求目标服务器
- 修改代码

`base.js`

```js
const Koa = require('koa');
const koaStatic = require('koa-static');
const axios = require('axios');

const app = new Koa();
const PORT = 1234;

app.use(koaStatic('./www'));

app.use(async (ctx) => {
  let ret;
  // 获取本次接收的请求的请求路径
  const path = ctx.req.url.split('?')[0];
  console.log('ctx.query.server', ctx.query.server);
  // 如果请求路径以api开头, 那么作为接口请求处理
  if (path.startsWith('/api')) {
    // 这样实现的路由不是很优雅, 但是能用 😂
    switch (path) {
      case '/api/getFriend':
        ret = { name: 'quanquan', friend: 'gl' };
        break;
      // 如果接口需要代理接口路径为 /api/proxy
      case '/api/proxy':
        // axios 直接访问前端给出的目标服务器url, 并将目标服务器返回的数据直接返回给前端
        ret = (await axios.get(ctx.query.server)).data;
        break;
      default:
        ret = { errno: 1, errmsg: '未知接口' };
        break;
    }
  }
  ctx.body = ret;
});

app.listen(PORT, () => {
  console.log('the server is listen: ', PORT);
});
```

前端代码:

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
    <h3>ServerProxy 实现跨域</h3>

    a: <input type="text" id="a" value="1">
    b: <input type="text" id="b" value="2">
    <button id="add">计算加法</button>
    <button id="subtrnct">计算减法</button>

    <div>计算结果为: <span id="ret"></span></div>
    <script>
        var aDom = document.getElementById('a')
        var bDom = document.getElementById('b')
        var addBtn = document.getElementById('add')
        var subtrnctDom = document.getElementById('subtrnct')
        var retDom = document.getElementById('ret')

        function add() {
            if(!a.value.trim() || !b.value.trim()) return
            var xhr = new XMLHttpRequest()
            xhr.open('GET', '/api/proxy' + '?server=' + encodeURIComponent('http://localhost:1111/?a='+ a.value +'&b=' + b.value))
            xhr.onreadystatechange = function() {
                if(xhr.readyState === 4 && xhr.status === 200) {
                    console.log('接口返回的数据为: ', xhr.responseText)
                    retDom.innerHTML = JSON.parse(xhr.responseText).result
                }
            }
            xhr.send()
        }

        function subtrnct() {
            if(!a.value.trim() || !b.value.trim()) return
            var xhr = new XMLHttpRequest()
            xhr.open('GET', '/api/proxy' + '?server=' + encodeURIComponent('http://localhost:2222/?a='+ a.value +'&b=' + b.value))
            xhr.onreadystatechange = function() {
                if(xhr.readyState === 4 && xhr.status === 200) {
                    console.log('接口返回的数据为: ', xhr.responseText)
                    retDom.innerHTML = JSON.parse(xhr.responseText).result
                }
            }
            xhr.send()
        }

        addBtn.addEventListener('click', add)
        subtrnctDom.addEventListener('click', subtrnct)
    </script>
</body>
</html>
```

最后结果:
![server-proxy-dfasdfasfas](https://user-gold-cdn.xitu.io/2018/12/16/167b2f3635504d7a?w=1165&h=622&f=gif&s=475482)

结语: ServerProxy 的原理大概就是这个样子的啦, 通过 ajax 访问同域后端服务, 后端服务访问目标服务并将目标服务返回的内容透传给前端. 当然实际操作起来不会像例子这么简单. 我的另一个系列文章[【手把手带你撸一个接口测试工具】](https://juejin.im/post/5bfd43986fb9a049ed308f1a)将会详细介绍复杂一些的情况, 包括不同的请求类型, 请求头设置以及响应头获取等等. 希望感兴趣的小伙伴继续关注.

关于跨域的其他方式: [document.domain](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/domain) 一行代码可以搞定, 适合同主域名不同子域名的情况. [postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage) 需要添加额外 iframe, 整体实现较为简单, 一个 API 搞定, 感兴趣的同学可以看看[文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage). 还有一些比较小众的做法 `flash` `CSST` 前端打点尝尝用到的 img 标签等等, 这里就不一一列举了, 学无止境...

![2018-12-16-01-38-47](https://user-gold-cdn.xitu.io/2018/12/16/167b2f36356d4fe8?w=556&h=566&f=png&s=230189)