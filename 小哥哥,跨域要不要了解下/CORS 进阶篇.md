系列文章:

- [【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
- [【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)

## 预检请求的诞生

> 在[前一篇文章](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)结尾, 我们发现使用 CORS 方式实现跨域, 有时候会发送两个请求 `一个 OPTIONS 一个正常请求`, 这个 OPTIONS 是个什么鬼呢?

下面贴一段 MDN 的解释
![2018-12-08-09-17-13](http://img.blog.niubishanshan.top/2018-12-08-09-17-13.png)

众所周知, 后端 API 设计比较流行的范式就是 restful(到 2018 年 12 月 8 日). 在 restful 中分别用不同的 HTTP METHOD 标识后端的 CURD, 对于使用这些可能会更新后端数据的 HTTP METHOD 发出的跨域请求, 浏览器要首先和服务器商定一下当前的域名是不是有执行对应的 CURD 的权限. 于是这个 OPTIONS 类型的 `预检请求` 就诞生了. 那么问题来了 `可能对服务器数据产生副作用的 HTTP 请求方法` 是有那些咧? 不知道么有关系, TIM 队长为我们探探路 😄

![2018-12-08-09-32-33](http://img.blog.niubishanshan.top/2018-12-08-09-32-33.png)

## 简单请求 VS 复杂请求

在 CORS 机制中, 把请求分为了 `简单请求` 和 `复杂请求`, 一个 HTTP 请求若想要让自己成为一个简单请求就要满足以下条件:

- 首先, 请求方式的限制: 请求方式(method) 只能是 `GET POST HEAD` 三者中的一个
- 其次就是请求头字段的限制: 请求头字段必须包含在以下集合中, 包括: `Accept Accept-Language Content-Language Content-Type DPR Downlink Save-Data Viewport-Width Width`.
- 其次就是请求头值的限制: 当请求头中包含 `Content-Type` 的时候, 其值必须为 `text/plain multipart/form-data application/x-www-form-urlencoded(这个是 form 提交默认的 Content-Type)` 三者中的一个.

综上, 只要前端发出的请求满足以上三个条件, 你发出的请求就是简单请求. 那么什么事复杂请求呢? 答: 只要不是简单请求就是复杂请求. 原本以为很难理清的概念, 居然只有三个条件搞定 ^_^.

![2018-12-08-09-58-34](http://img.blog.niubishanshan.top/2018-12-08-09-58-34.png)

再告诉大家一个秘密, 所有的简单请求跨域访问都是不会触发预检请求的哟. 那是复杂请求的专利...

## 预检请求都干了啥 😳

> 对于复杂请求发生跨域访问前, 总是要通过预检请求进行鉴权. 那么鉴权的过程到底是啥么样子的呢? 这一步我们一起来研究一下.

- 首先, 打开[上一节的代码](https://github.com/luoquanquan/cross-domain/commit/11686561e729075da6c5d7d1c64470e08ad790e8)
- 分别执行 `node ./be/cors/index.js` `live-server ./fe/cors` 启动后端服务和前端的 web 容器.
- 浏览器自动打开后打开控制台, 切换到 Network tab 并刷新浏览器. 不出意外的话, 看到的是这个样子的
![2018-12-08-10-09-38](http://img.blog.niubishanshan.top/2018-12-08-10-09-38.png)
- 点击一下第一个 localhost 请求并查看详情
![2018-12-08-10-22-25](http://img.blog.niubishanshan.top/2018-12-08-10-22-25.png)
不难发现, 响应头里标注的几个字段, 就是我们的后端项目里边写的几个.

```js
response.setHeader('Access-Control-Allow-Origin', '*');
response.setHeader('Access-Control-Allow-Methods', 'PUT');
response.setHeader('Access-Control-Allow-Headers', 'token');
response.setHeader('Access-Control-Max-Age', 5);
```

一一对应, 绝非偶然 😄. 那么请求头中标注的两个又是什么意思呢?

浏览器在接受到我们发送的跨域请求的指令时, 会自动判断我们的请求是否属于跨域请求, 如果是的话便会发出预检请求, 预检请求的请求头信息也是浏览器根据我们的请求信息自动添加的. 示例项目中, 因为我们的请求是 `PUT` 类型的, 所以在预检请求的时候会添加 `Access-Control-Allow-Methods: PUT` 来咨询服务器自己是否可以向它发送这种类型的请求. 同理, 由于我们的请求中有自定义请求头 `token` 所以, 在预检请求中, 浏览器要和服务器做是否可以添加自定义请求头的协商. 只有当浏览器和服务器之间的预检请求协商通过了, 浏览器才会继续发送真正的 `AJAX` 请求.

## 老板说, 我不想看到多余的请求

在工作中, 老板往往是不懂技术的. 能看控制台的老板一般是高手了. 面对这种一个 api 发两次请求的情况可能一个程序员笑笑也就过去了, 但是老板就不这么认为了, 一个接口他就要一次请求. 你要把 `圈圈的圈` 跨域文章推荐给老板, 让小哥哥也了解下? 估计你会被 fire 掉. 那肿么办呢?

![2018-12-08-10-42-18](http://img.blog.niubishanshan.top/2018-12-08-10-42-18.png)

面对这种情况, 有两种解决方案.

- 第一种, 可以和后端小哥哥商量一下. 把接口改成简单请求, 预检请求的问题就迎刃而解了.
- 然而, 有时候写好的代码谁都不愿意去改. 后端小哥哥不听话. 这种情况下 `Access-Control-Max-Age` 就派上用场了. 这个响应头的意思是预检请求的有效期. 在指定时间内再次跨域访问接口, 是不需要预检请求的, 单位是 `秒`. 如果我们把有效时间写的非常的长, 那么四不四看上去就像删除了预检请求了呢 ^_^.
- 附加情况, 你老板不懂技术瞎 J8 指挥. 小爷我不干了. 当然这种处理方案比较不推荐.

PS: 使用 `Access-Control-Max-Age` 机制和缓存类似, 所以给老板演示的时候千万不要清理缓存. 不要勾选 Network 下的 `disable cache`. 不说啦, 都是泪...
![2018-12-08-11-13-21](http://img.blog.niubishanshan.top/2018-12-08-11-13-21.png)

## 咱们不能允许所有的人都访问呀

通过 `Access-Control-Allow-Origin`, 可以在后端设置可以跨域访问我们的域名列表, `*` 代表所有的域名都可以跨域访问我们的后端, 这样其实是有隐患的. 为了安全起见, 我们把可以跨域访问的域名限制为我们已知的域名. 老规矩.

![2018-12-08-11-24-04](http://img.blog.niubishanshan.top/2018-12-08-11-24-04.png)

后端代码

```js
// 修改一行代码, 一定要添加协议哟
response.setHeader('Access-Control-Allow-Origin', 'http://127.0.0.1:8080');
```

修改以后浏览器访问 [http://127.0.0.1:8080](http://127.0.0.1:8080)
![2018-12-08-11-27-57](http://img.blog.niubishanshan.top/2018-12-08-11-27-57.png)

如果想要开放多个域名的跨域访问咋办咧?

如果我们有多个业务域名需要跨域访问同一个服务器, 可以把允许的域名列表保存到一个数组里. 接到请求之后先判断当前请求域名是否在我们允许的域名列表里, 如果在的话直接添加到响应头 `Access-Control-Allow-Origin` 下.

后端代码

```js
const http = require('http');

const PORT = 8888;

// 协议名必填, 如果同时存在 http 和 https 就写两条s
const allowOrigin = ['http://127.0.0.1:8080', 'https://www.baidu.com'];

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  const { headers: { origin } } = request;
  if (allowOrigin.includes(origin)) {
    response.setHeader('Access-Control-Allow-Origin', origin);
  }
  response.setHeader('Access-Control-Allow-Methods', 'PUT');
  response.setHeader('Access-Control-Allow-Headers', 'token');
  response.setHeader('Access-Control-Max-Age', 5);
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

此时[代码](https://github.com/luoquanquan/cross-domain/commit/7015daf88efd5eff890f5bcaa40ce15ba1357a1f), 首先访问[http://127.0.0.1:8080](http://127.0.0.1:8080)
![2018-12-08-11-46-03](http://img.blog.niubishanshan.top/2018-12-08-11-46-03.png)
响应结果成功打印, 没有任何问题.

其次访问 [https://www.baidu.com](https://www.baidu.com/), 打开控制台, 执行

```js
xhr = new XMLHttpRequest()
xhr.open('GET', 'http://localhost:8888')
xhr.onreadystatechange = () => {
    xhr.status === 200 && xhr.readyState === 4 && console.log(xhr.responseText)
}
xhr.send()
```

![2018-12-08-11-54-13](http://img.blog.niubishanshan.top/2018-12-08-11-54-13.png)

没有任何报错, 返回结果成功打印. 成功...

## 你的请求怎么没有携带 Cookie

一般情况下, 前端发出的跨域的 ajax OR [fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch) 请求是不会携带 Cookie 的. 但是, 后端小哥哥还要. 咋弄咧? 加上呗.

![2018-12-08-12-05-05](http://img.blog.niubishanshan.top/2018-12-08-12-05-05.png)

前端代码:

```js
// 在 xhr.send 之前添加这一行
xhr.withCredentials = true;
```

添加完以后, 刷新浏览器.

![2018-12-08-12-07-24](http://img.blog.niubishanshan.top/2018-12-08-12-07-24.png)

对于这个报错, 不知道你有没有啥好说的, 反正我是没啥话了...

后端代码:

```js
const http = require('http');

const PORT = 8888;

// 协议名必填, 如果同时存在 http 和 https 就写两条s
const allowOrigin = ['http://127.0.0.1:8080', 'http://localhost:8080', 'https://www.baidu.com'];

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  const { method, headers: { origin, cookie } } = request;
  if (allowOrigin.includes(origin)) {
    response.setHeader('Access-Control-Allow-Origin', origin);
  }
  response.setHeader('Access-Control-Allow-Methods', 'PUT');
  // 允许前端请求携带 Cookie
  response.setHeader('Access-Control-Allow-Credentials', true);
  response.setHeader('Access-Control-Allow-Headers', 'token');
  if (method === 'OPTIONS') {
    console.log('预检请求');
  } else if (!cookie) {
    //  如果不存在 Cookie 就设置 Cookie
    response.setHeader('Set-Cookie', 'quanquan=fe');
  }
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

此时[代码](https://github.com/luoquanquan/cross-domain/commit/3ebead3f6231df75eef65b25c5c4543b19cf49fb), 再次到浏览器看一下.

Cookie 中多了一条
![2018-12-08-12-37-50](http://img.blog.niubishanshan.top/2018-12-08-12-37-50.png)

请求中携带了 Cookie
![2018-12-08-12-37-26](http://img.blog.niubishanshan.top/2018-12-08-12-37-26.png)

通过下边的动图可以看出, 我们前后端 Cookie 传递非常的通畅.

## 我在响应头上给你返回了 Token, 你取出来放在请求头上

工作中常常遇到后端把一些标识放在响应头上返回给前端的 case, 比如用户登录, 后端返回用户的唯一标识放在响应头上. 需要前端获取, 后续的请求都需要把这个标识放在请求头, 用于验证用户的身份.

我们首先修改后端代码:

```js
// 在 response.end() 前添加这一行
response.setHeader('token', 'quanquan');
```

修改前端代码:

```js
xhr.onreadystatechange = function() {
  if(xhr.readyState === 4 && xhr.status === 200) {
    console.log(xhr.responseText)
    // 打印响应数据时同时打印所有响应头
    console.log(xhr.getAllResponseHeaders())
  }
}
```

修改完成后[代码](https://github.com/luoquanquan/cross-domain/commit/a240adeb8566607174b7b4acffa8ec27149d0f35), 浏览器看一下.

console.log 打印出了空行
![2018-12-08-13-26-21](http://img.blog.niubishanshan.top/2018-12-08-13-26-21.png)

但是在 Network Tab 下后端确实返回了响应头 token 字段. 懵逼了...
![2018-12-08-13-28-41](http://img.blog.niubishanshan.top/2018-12-08-13-28-41.png)

原来, `Access-Control-` 系列还有一个响应头 `Access-Control-Expose-Headers`, 我们在后端代码 `response.end(...)` 之前加上 `response.setHeader('Access-Control-Expose-Headers', 'token');`再次会浏览器查看
![2018-12-08-13-31-40](http://img.blog.niubishanshan.top/2018-12-08-13-31-40.png)

成功了 😄.

## 预检请求不返回内容把

我们的响应结果本来应该是在正式的请求中才需要返回的, 但是我们看下预检请求的返回详情发现
![2018-12-08-13-34-09](http://img.blog.niubishanshan.top/2018-12-08-13-34-09.png)

预检请求只是浏览器层面的解析, 前端代码根本拿不到. 这里的内容仅仅是浪费带宽和用户的流量. 所以我们改造一下.预检请求不再返回内容.

后端代码:

```js
const http = require('http');

const PORT = 8888;

// 协议名必填, 如果同时存在 http 和 https 就写两条s
const allowOrigin = ['http://127.0.0.1:8080', 'http://localhost:8080', 'https://www.baidu.com'];

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  const { method, headers: { origin, cookie } } = request;
  if (allowOrigin.includes(origin)) {
    response.setHeader('Access-Control-Allow-Origin', origin);
  }
  response.setHeader('Access-Control-Allow-Methods', 'PUT');
  response.setHeader('Access-Control-Allow-Credentials', true);
  response.setHeader('Access-Control-Allow-Headers', 'token');
  response.setHeader('Access-Control-Expose-Headers', 'token');
  response.setHeader('token', 'quanquan');
  if (method === 'OPTIONS') {
    response.writeHead(204);
    response.end('');
  } else if (!cookie) {
    response.setHeader('Set-Cookie', 'quanquan=fe');
  }
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

此时[代码](https://github.com/luoquanquan/cross-domain/commit/8aae5963cb1afc48014ca355b844be62cc0a6e78), 验证, 就不验证了吧. 好使 😄

下基预告: 前两种跨域方案就算是讲完了, 不少小伙伴吐槽, jsonp 太老, cors 太麻烦.... 那么下一节我们尝试一下 `反向代理`, See you
