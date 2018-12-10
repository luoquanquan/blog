系列文章:

- [【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
- [【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)
- [【小哥哥, 跨域要不要了解下】CORS 进阶篇](https://juejin.im/post/5c0b5a8851882548e9380afb)
- [【小哥哥, 跨域要不要了解下】NGINX 反向代理](https://juejin.im/post/5c0e6d606fb9a049f66bf246)

> 打算纯前端做一个[接口测试工具](https://juejin.im/post/5bfd43986fb9a049ed308f1a), 直到遇到
> ![2018-12-05-14-13-31](https://user-gold-cdn.xitu.io/2018/12/6/1677f29fc07225a2?w=2556&h=64&f=png&s=69130)
> 这个报错, 触碰到了知识盲区了, 怎么办???
> ![2018-12-05-14-14-58](https://user-gold-cdn.xitu.io/2018/12/6/1677f29fc03c146e?w=442&h=574&f=png&s=105156)
>
> 还好, 有谷哥和度娘. 原来是[跨域](https://baike.baidu.com/item/AJAX%20%E8%B7%A8%E5%9F%9F%E8%AE%BF%E9%97%AE/15633363)
> 随手整理了一下常用的跨域方式处理方案, 这里马上分享给大家 😋

ps: 为了保证前后端编码的一致性, 本系列文章中涉及部分后端内容. 后端统一使用原生 nodejs 来搞, 请奔走相告.

## 准备工作

为了托管我们的静态页面, 我们需要一个可以提供服务器环境的插件, 这里推荐 `live-server`, 通过命令 `npm i -g live-server` 安装即可. 该插件支持html文件热更新. 那用户体验简直飞起. 一键启动, 只需要在需要托管的目录执行 `live-server .` 即可.
![9150e4e5ly1fvmnifqdj2g207i07iaa4](https://user-gold-cdn.xitu.io/2018/12/6/1677f29fbe8cf261?w=270&h=270&f=gif&s=12603)

ps: `live-server` 依赖 nodejs, 没有安装的小伙伴, 请参照[这篇文章](https://juejin.im/post/5bfd43986fb9a049ed308f1a)安装 nodejs.

## AJAX 访问接口跨域解决方案

首先, 更正几个常见的错误认识:
1. 同源策略是浏览器的行为, 和 js 关系不大.
2. 所谓跨域是指请求发起方页面所在的 url 与访问的 api 存在协议, 域名, 端口中任意一个不同即视为跨域. 并不单单是指域名.
3. 跨域这个东西, 日常工作中并不是很常用. 你想, 谁会闲的没事儿干总是请求人家别人的 api 去.

## jsonp

> 可能有小伙伴会说. 圈圈, 你扯淡, 既然浏览器有跨域限制. 为什么我司项目从 [bootcdn](https://www.bootcdn.cn/), 引入的 jquery 依然跑在信息高速路上, 没有任何低头的意思?

hhh, 😄. 这个质疑提的好. 浏览器同源策略禁止的是 ajax 请求. 然鹅, jquery 是一个 js 文件. 不受该策略的限制.

我尼玛, 那到底是啥限制啥不限制嘛???

![2018-12-05-16-05-22](https://user-gold-cdn.xitu.io/2018/12/6/1677f29fc0bd4074?w=652&h=560&f=png&s=198441)

根据 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) (自备梯子), 对于浏览器的同源策略的解释, 不受限制的外域资源加载情况有以下几种:
- script
- link
- img
- video
- object embed applet
- font-face 有的浏览器允许, 有的禁止
- frame

那么问题来了, 挖掘机学校..., 不好意思走错片场了. 既然有这么多方式可以绕过浏览器同源策略的限制. 那么, 是不是我们可以做一点事情呢 ^_^

答: 是的 😄.

那还不抓紧搞起来?

![6af89bc8gw1f8rf4zbaemg209105fka1](https://user-gold-cdn.xitu.io/2018/12/6/1677f29ff2a80875?w=325&h=195&f=gif&s=697904)

我们使用第一个特例 `script` 一步一步实现跨域访问 (jsonp).

- 首先, 创建本次文章的项目目录
![2018-12-05-17-14-53](https://user-gold-cdn.xitu.io/2018/12/6/1677f29ff2b20c7c?w=512&h=498&f=png&s=26477)
目录中, be 代表是后端项目, fe 代表前端项目. jsonp 目录说明我们是用 jsonp 的方式实现跨域.
- 在项目根目录下执行 `live-server ./fe/jsonp/` 启动前端 web 容器
- 编辑 `./fe/jsonp/` 目录下的 index.html 文件. 代码如下:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>jsonp 实现跨域</title>
</head>
<body>
    <h3>jsonp 实现跨域</h3>
</body>
</html>
```

- 浏览器访问 [localhost:8080](http://localhost:8080/)浏览器如下图说明前端 web 容器部署成功.
![2018-12-05-17-22-21](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a032f80775?w=1220&h=736&f=png&s=48228)

- 编写后端代码, 编写 `be/jsonp/index.js` 文件, 文件内容如下

```js
var http = require('http');
var PORT = 8888

// 创建一个 http 服务
var server = http.createServer(function(request, response) {
    response.end('hello world')
})

// 启动服务, 监听端口
server.listen(PORT, function() {
    console.log('服务启动成功, 正在监听: ', PORT)
})
```

- 编写完成后命令行执行 `node ./be/jsonp/index.js`
命令行中出现![2018-12-05-17-57-06](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a033455791?w=1088&h=82&f=png&s=29058)说明后端程序启动成功.此时可以通过浏览器访问 [localhost:8888](http://localhost:8888/)获得 hello world
![2018-12-05-19-15-36](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a073ed0985?w=1002&h=210&f=png&s=32208)

- 下来, 我们在前端的 `index.html` 中尝试通过 ajax 请求 `http://localhost:8888/` 来获取返回数据, 添加如下代码, 添加以后[代码](https://github.com/luoquanquan/cross-domain/commit/cce37cc50db74d0584f55da393d948c1dcfa7696)

```html
<script>
    var xhr = new XMLHttpRequest()
    xhr.open('GET', 'http://localhost:8888/')
    xhr.send()
</script>
```

回到浏览器, 查看页面控制台, 熟悉的错误出现了. Access to XMLHttpRequest at `http://localhost:8888/` from origin `http://127.0.0.1:8080` has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. 这个错误说明了, 我们是不能通过 ajax 的方式从 `http://127.0.0.1:8080` 访问 `http://localhost:8888/` 的.

既然不能通过 ajax 实现跨域的访问, 同时 mdn 又说 script 标签不受同源策略的限制. 那么, 我们尝试一下用 script 标签引入 `http://localhost:8888/` 试试?

![2018-12-05-19-29-19](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a06e7fa1ba?w=578&h=592&f=png&s=224850)

此时的[代码](https://github.com/luoquanquan/cross-domain/commit/d7ae69f656b1476a25057614264c411f7de13da4), 网络请求没有问题. 知识报了 js 文件不合法的问题. 如果我们把接口返回的数据调整为规范的 js 是不是, 嗯哼???

干起, 修改后端代码, 返回的内容由 `hello world` 改为 `console.log('hello world')`, 修改后的[代码](https://github.com/luoquanquan/cross-domain/commit/bb5b1d2c6a1f3756c4fc7d5dda04e9bc6b30310e)(修改完后端代码以后切记重启服务哈 ^_^)

不得了, 不得了, 返回的结果不紧没有报错, 甚至可以执行. 我们从后端返回的 `hellow world` 成功的答应到了控制台了.

![2018-12-05-20-15-32](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a075dacfd8?w=700&h=550&f=png&s=520755)

试想一下, 如果我们通过 js 文件里定义一个变量用于存放后端返回给前端的数据, 前端插入一个 script 标签, 把后端返回的变量定义执行一把. 那样定义的变量岂不是就可以在全局可以获取到后端定义的变量了. 赶紧试一把 😄

首先修改后端代码, 只需要调整一行.(修改完后端代码以后切记重启服务哈 ^_^)

```js
response.end("var aaaa = {name: 'quanquan', friend: 'guiling'}");
```

其次调整前端代码

```html
<script>
    // 第一次因为还没有引入外部 js 所以打印 undefined
    console.log(window.aaaa)
    // 1 秒后, 外部 js 加载完成, 能打印出后端返回的变量定义
    setTimeout(() => {console.log(window.aaaa)}, 1e3)
</script>
<script src="http://localhost:8888/"></script>
```

当前[代码](https://github.com/luoquanquan/cross-domain/commit/5bdc0dd023f4676a8d6eebb87af73e087661d830), 通过这种方式, 我们能够成功的获取到后端返回的数据. 但是, 接口这个东西时快时慢. 写个定时器轮询? 有点不够 666, 肿么办?

![2018-12-05-21-15-37](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a0744de62a?w=532&h=536&f=png&s=192073)

========================  思考 5 分钟  ========================

![2018-12-05-21-18-13](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a0774655f4?w=614&h=552&f=png&s=126200)

========================   5 分钟已过  ========================

既然, 写在 script 标签上的内容是可以直接执行的. 那么, 如果我们把变量的定义改写成一个函数的执行可不可以呢 ^_^, 试试?

后端(修改完后端代码以后切记重启服务哈 ^_^)

```js
response.end("aaaa({name: 'quanquan', friend: 'guiling'})");
```

前端

```html
<script>
    // 由于后端返回的内容即将调用函数 aaaa, 那我们就预先定义一个呗, 这东西就叫回调函数
    function aaaa(param) {
        console.log('后端返回的参数是: ', param)
    }
</script>
<script src="http://localhost:8888/"></script>
```

结果
![2018-12-05-21-30-30](https://user-gold-cdn.xitu.io/2018/12/6/1677f2a079906415?w=2558&h=826&f=png&s=118779)

此时[代码](https://github.com/luoquanquan/cross-domain/commit/a2e0ba9ca2edcabf03ca549fd018d85c96fa0c64), 目前为止, 我们已经彻底解决了跨域的问题. 很靠谱有木有? 当然木有. 这个玩意儿只是说明了 jsonp 的原理, 并没有实用性. 下一步, 我们做一点封装. 让我们的代码更健壮 💪🏻

最后, 修改一把代码

前端

```js
// 创建 Jsonp 类
// 初始化时传入两个参数, url 是接口的url
// cb 是对于接口返回的参数的处理
function Jsonp(url, cb) {
    this.callbackName = 'jsonp_' + Date.now()
    this.cb = cb
    this.url = url
    this.init()
}

// 初始化方法 用于拼接 url
Jsonp.prototype.init = function() {
    if(~this.url.indexOf('?')) {
        this.url = this.url + '&callback=' + this.callbackName
    } else {
        this.url = this.url + '?callback=' + this.callbackName
    }
    this.createCallback()
    this.createScript()
}

// 创建 script 标签, src 取接口请求的url
Jsonp.prototype.createScript = function() {
    var script = document.createElement('script')
    script.src = this.url
    script.onload = function() {
        this.remove()
    }
    document.body.appendChild(script)
}

// 绑定回调函数
Jsonp.prototype.createCallback = function() {
    window[this.callbackName] = this.cb
}

// 创建 jsonp 实例, 并指定回调函数
new Jsonp('http://localhost:8888/', function(data) {
    console.log(data)
})
```

后端(修改完后端代码以后切记重启服务哈 ^_^)

```js
const http = require('http');
// 新引入了 url 模块, 主要用于解析请求参数
const url = require('url');

const PORT = 8888;

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  // 获取前端请求数据
  const queryObj = url.parse(request.url, true).query;
  // 这里把前端传来的 callback 字段作为后端返回的回调函数的函数名称
  response.end(`${queryObj.callback}({name: 'quanquan', friend: 'guiling'})`);
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

目前[代码](https://github.com/luoquanquan/cross-domain/commit/75331e68e664549144cb98c8086aa38d5cc57310), 至此我们已经能够顺利的获取跨域资源了. 👏🏻.

下集预告: jsonp 是一种传统的跨域解决方案, 关于这种方式的优缺点, 请[度娘](https://www.baidu.com), 下一节, 我们一起学习相对比较现代一点的跨域解决方案. `CORS`, See You
