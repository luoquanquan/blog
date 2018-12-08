系列文章:

- [【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
- [【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)
- [【小哥哥, 跨域要不要了解下】CORS 进阶篇](https://juejin.im/post/5c0b5a8851882548e9380afb)

> 在[前一篇文章](https://juejin.im/post/5c07fa04e51d451de968906b)中, 我们一起学习了第一种跨域处理方案 `JSONP`. 这种方法相对比较原始, 优点是兼容性好, 就连现代前端没怎么听说过的 `IE 6` 上跑起来都是妥妥的. 然鹅, 它也就这一点优点了. 其缺点有: 只支持 GET 请求, 配置繁琐(前后端都需要调整代码), 在 window 上注册各种回调函数, 开发体验差....
> ![2018-12-07-10-07-21](https://user-gold-cdn.xitu.io/2018/12/7/167885f8190c15f9?w=578&h=552&f=png&s=402218)

## CORS

由于 JSONP 的方案存在诸多缺点且老旧, 这里我们一起学习一种比较现代的跨域问题解决方案---CORS

### 兼容性

从 mdn 官网粘来的兼容性列表如下:
![2018-12-07-10-25-52](https://user-gold-cdn.xitu.io/2018/12/7/167885f819baff5b?w=1824&h=190&f=png&s=36828)

ie 10 都可以跑, 足以满足现代前端开发者的需求了.

### 概念 😳

概念性的东西[在这儿](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS) MDN 偶尔需要梯子, 自备哈.

### 搭建跨域的环境

我们先创建一个跨域的环境, 代码基于我们 jsonp 时候的示例项目 [cross-domain](https://github.com/luoquanquan/cross-domain.git), 首先, 在 fe 和 be 目录下创建 cors 目录. 其次, 分别添加 `index.html` 和 `index.js`. 修改以后的项目目录如下图.
![2018-12-07-14-26-54](https://user-gold-cdn.xitu.io/2018/12/7/167885f81af9f36a?w=520&h=606&f=png&s=50332)

编写前端代码 `fe/cors/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>CORS 实现跨域</title>
</head>
<body>
    <h3>CORS 实现跨域</h3>

    <script>
        var xhr = new XMLHttpRequest()
        xhr.open('GET', 'http://localhost:8888')
        xhr.onreadystatechange = function() {
            if(xhr.readyState === 4 && xhr.status === 200) {
                console.log(xhr.responseText)
            }
        }
        xhr.send()
    </script>
</body>
</html>
```

一个灰常简单的 ajax 请求, 有木有.

后端代码 `be/cors/index.js`

```js
const http = require('http');

const PORT = 8888;

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

此时的[项目代码](https://github.com/luoquanquan/cross-domain/commit/483808571faf3cadc886033c2214a2b6f25b08fa)

### 找点苗头

代码环境准备完成后
- 首先启动后端代码 `node ./be/cors/index.js`
- 其次启动前端 web 容器 `live-server ./fe/cors`
- 打开浏览器, 访问 [http://localhost:8080/](http://localhost:8080/)
- 打开控制台, 切换到 Console tab
- 刷新浏览器

![2018-12-07-16-47-05](https://user-gold-cdn.xitu.io/2018/12/7/167885f81a2a5b8c?w=1498&h=23&f=png&s=16734)
我们细细分析这个熟悉的报错, 前一段告诉我们我们的请求被 block 了. 后边居然直接告诉我们解决方案了, 方案了 `'Access-Control-Allow-Origin' header is present on the requested resource.`, 这么明显的暗示, 难道我们就不试试???
![2018-12-07-17-27-44](https://user-gold-cdn.xitu.io/2018/12/7/167885f819f00e67?w=478&h=508&f=png&s=163171)

### 响应头添加 Access-Control-Allow-Origin

针对浏览器的报错, 我们分析出他是要我们在响应头上添加`Access-Control-Allow-Origin`这个字段,
那么我们修改我们的后端代码.

```js
const http = require('http');

const PORT = 8888;

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  response.setHeader('Access-Control-Allow-Origin', '*');
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

改动后的[代码](https://github.com/luoquanquan/cross-domain/commit/9aff17ce9a1facb0ca643a233bbe30a8741d08e1).

<div style="font-weight: bold; font-size: 20px;">巨大的 PS: 修改过后端代码以后, 一定要<span style="color: red;">重启 node 服务</span></div>

浏览器刷新一下, 我了个乖乖. 好了 😄
![2018-12-07-17-33-48](https://user-gold-cdn.xitu.io/2018/12/7/167885f818a3544d?w=1920&h=361&f=png&s=40239)

开心过后, 我们想一下, jsonp 的缺点是只能支持 GET 请求, 作为`现代`的跨域请求方式. cors 能不能支持其他的请求方式呢?

### 其他请求方式的支持

作为`现代`的跨域问题解决方案, 应该是能解决多种请求方式的. 光说不练假把式. 咱们试试 😄

改动前端代码

```js
// 改动前
xhr.open('GET', 'http://localhost:8888')
// 改动后
xhr.open('POST', 'http://localhost:8888')
```

修改后[代码](https://github.com/luoquanquan/cross-domain/commit/8494d3543b9919bf58ab1c3d9c72d41d97a7cbaa)来浏览器上看一下?

![2018-12-07-18-22-46](https://user-gold-cdn.xitu.io/2018/12/7/167885f8469e427f?w=741&h=342&f=png&s=37334)

木有任何问题, 返回的数据顺利的打印. 没有任何的报错.

趁着兴头试试`PUT`请求

再次改动前端代码

```js
// 改动前
xhr.open('POST', 'http://localhost:8888')
// 改动后
xhr.open('PUT', 'http://localhost:8888')
```

修改后[代码](https://github.com/luoquanquan/cross-domain/commit/8494d3543b9919bf58ab1c3d9c72d41d97a7cbaa)来浏览器上看一下?

![2018-12-07-18-25-55](https://user-gold-cdn.xitu.io/2018/12/7/167885f868b2ef87?w=1522&h=340&f=png&s=54871)

哎呀我滴妈? 很眼熟的错误, 但是不要认错人哈, 这次的报错和之前的报错长的很像, 但是关键词不一样了. 根据之前的经验, 后端添加 `Access-Control-Allow-Methods` 响应头应该好使.

后端代码

```js
const http = require('http');

const PORT = 8888;

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  response.setHeader('Access-Control-Allow-Origin', '*');
  response.setHeader('Access-Control-Allow-Methods', 'PUT');
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

修改后[代码](https://github.com/luoquanquan/cross-domain/commit/b50d23491506b9cd2c52dc36155c57e07c18f2ed)来浏览器上看一下?

![2018-12-07-18-32-14](https://user-gold-cdn.xitu.io/2018/12/7/167885f85a61e806?w=809&h=340&f=png&s=38388)

成功了 😄
![2018-12-07-18-33-45](https://user-gold-cdn.xitu.io/2018/12/7/167885f86da1f8a5?w=578&h=574&f=png&s=176882)

其他的 http 方法和 `PUT` 方法处理的方式是一样的. 举一反三即可.

### 后端说, 你的请求要加一个 token 呀

既然是现代的开发, 那么会话的管理一般是会用 jwt(后续可能会写相关的文章), jwt 一个闪耀的标志就是请求头添加了 jwt token. 明人不说暗话.

![2018-12-07-18-39-07](https://user-gold-cdn.xitu.io/2018/12/7/167885f8755e45eb?w=576&h=538&f=png&s=175877)

修改前端代码:

```js
// 添加了一行
xhr.setRequestHeader('token', 'quanquanbunengshuo')
```

修改后[代码](https://github.com/luoquanquan/cross-domain/commit/b4b372173f6c8c5645b5e8cc4e79b9f41e22a9ce)来浏览器上看一下?

![2018-12-07-18-43-04](https://user-gold-cdn.xitu.io/2018/12/7/167885f880245ae6?w=1633&h=341&f=png&s=57655)

相信大家已经摸清了我的套路, 闲话不扯.

后端代码

```js
// 添加了一行
response.setHeader('Access-Control-Allow-Headers', 'token');
```

修改后[代码](https://github.com/luoquanquan/cross-domain/commit/11686561e729075da6c5d7d1c64470e08ad790e8)来浏览器上看一下?

![2018-12-07-18-45-14](https://user-gold-cdn.xitu.io/2018/12/7/167885f87ff961d2?w=987&h=339&f=png&s=39934)

目前为止, 跨域请求成功了, 请求方式兼容了, 自定义请求头好使了. 是不是大吉大利, 可以吃鸡了呢?

### 致, 被打入冷宫的 Network tab

我们自始至终都在查看浏览器的 Console tab, 作为一个通信性质的文章, 不看一下 Network 明显有点说不过去辣.

![2018-12-07-18-50-42](https://user-gold-cdn.xitu.io/2018/12/7/167885f89557ea13?w=350&h=574&f=png&s=125450)

既然存在, 那肯定是要看看的, 我们把 tab 切换到 Network.

![2018-12-07-18-52-58](https://user-gold-cdn.xitu.io/2018/12/7/167885f8954581c6?w=1919&h=489&f=png&s=81050)

哎呦喂? 两个请求, 一个 OPTIONS 一个 PUT, 这是什么鬼?

下集预告: 刚刚看到了 Network 就出现了血案. 当然如果仅仅是停留在会用 CORS 实现跨域上, 到目前为止已经没有什么问题了, <b>用来面试也是杠杠滴</b>. 下一步, 我们一起探讨 CORS 条件下, `预检请求` 和 `Cookie` 携带那些事儿. 周五码字感觉好累...... 约小姐姐去辣 😄

![9150e4e5ly1fkhkg77t1mg204603w12l](https://user-gold-cdn.xitu.io/2018/12/7/167885f899802433?w=150&h=140&f=gif&s=373835)
