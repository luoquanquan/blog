> 在[上一篇文章](https://juejin.im/post/5d2805b86fb9a07ec07febb0)中我们实现了一个简单的 hello world, 这一节我们继续完善我们的项目. 作为实用篇本篇将会添加 react-router redux

> 并通过 react-helmet 实现自定义的 meta 标签, 完善 SEO. 这个实在是不想写了. 下篇吧...

## 加入路由畅游皇冠赌场

### 客户端渲染配置 react-router

来到皇冠赌场的大家那肯定是丈二的和尚, 摸不着头脑呀. 那么路由就应运而生了, 关于路由的原理建议大家看看[这篇文章](https://juejin.im/post/5d2d19ccf265da1b7f29b05f).

![9150e4e5gy1fznxtqxc8sg206y06yab7](http://img.blog.niubishanshan.top/9150e4e5gy1fznxtqxc8sg206y06yab7.gif)

如果你看了还回来了, 那说明还是我们澳门 XXXX 更加的有意思 😹.

谈到赌场无非就是这老四样, 抓牌, 看牌, 洗牌, 码牌~

![2019-07-16-17-28-14](http://img.blog.niubishanshan.top/2019-07-16-17-28-14.png)

那么我们就开始, 创建几个页面. 页面的代码结构如下图所示.

![2019-07-16-17-30-43](http://img.blog.niubishanshan.top/2019-07-16-17-30-43.png)

为了便于各个 level 的小伙伴理解, 这里无耻的运用了拼音命名法. 代码改动在[这里](https://github.com/luoquanquan/react-ssr-show/commit/8aef4f3714d83f5341c64a1956498e891356e5ac)

其次, 安装 react-router-dom 依赖, 并修改 App.jsx 和 client.js 文件, diff 在[这里](https://github.com/luoquanquan/react-ssr-show/commit/9b0fb5352770139622cff9d8e04b027016ef6435)

修改后的 App.jsx 文件
```js
import React from 'react';
// 从 react-router-dom 引入基础组件
import { NavLink, Switch, Route } from 'react-router-dom';

// 引入皇冠赌场的页面
import Home from './Home.jsx';
import Zhuapai from './Zhuapai.jsx';
import Kanpai from './Kanpai.jsx';
import Xipai from './Xipai.jsx';
import Mapai from './Mapai.jsx';

export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }

  render() {
    return (
      <div className="ssr-show">
        <h1>欢迎来到澳门皇冠赌场</h1>

        <NavLink to="/">首页</NavLink>
        <NavLink to="/Zhuapai">抓牌</NavLink>
        <NavLink to="/Kanpai">看牌</NavLink>
        <NavLink to="/Xipai">洗牌</NavLink>
        <NavLink to="/Mapai">码牌</NavLink>

        <Switch>
          <Route path="/" component={Home} />
          <Route path="/Zhuapai" component={Zhuapai} />
          <Route path="/Kanpai" component={Kanpai} />
          <Route path="/Xipai" component={Xipai} />
          <Route path="/Mapai" component={Mapai} />
        </Switch>
      </div>
    );
  }
}
```

修改后的 client.js 为
```js
import React from 'react';
import ReactDOM from 'react-dom';
// 从 react-router-dom 里边导入 BrowserRouter 组件
import { BrowserRouter } from 'react-router-dom';
import App from './components/App.jsx';

// 包装一下 App 组件
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('app'),
);
```

目前为止, 路由就算是配完了. 执行 `npm run build:client` 后, 用浏览器打开 index.html 文件.

![2019-07-15-16-03-01](http://img.blog.niubishanshan.top/2019-07-15-16-03-01.png)

成功就在眼前, 但是难免有一点小小的障碍~

这个报错的大致意思就是, 本地的文件不能用 react-router, 那么我们只能把它放到一个服务器上了. 还是我们的老伙伴 --- `live-server` 直接执行 `live-server ./dist`

浏览器打开 `localhost:8080`

![react-router-err](http://img.blog.niubishanshan.top/react-router-err.gif)

我们不难发现, 点击链接的时候浏览器地址栏有变化, 但是我们并不能体验到从发牌到码牌的一条龙"服务"...

看了下 react-router 的 [官方文档](https://reacttraining.com/react-router/web/api/Route/exact-bool) 原来我们没有指定路径匹配必须得精准匹配. 那么我们加上 `exact` 属性试试咧~

![react-router](http://img.blog.niubishanshan.top/react-router.gif)

此时的代码 [diff](https://github.com/luoquanquan/react-ssr-show/commit/653639940d9d5ecf66f668374727f8f9ae9de00e)

到目前为止, 我我们已经可以畅游澳门皇冠赌场了, 抓牌看牌洗牌码牌样样精通~

### 配置服务端渲染的 react-router

轻松搞定了客户端渲染的 react-router, 服务端渲染的话那就更加的简单了.

- 首先肯定是要引入 react-router-dom 依赖的(ps: ssr 需要导入的是 StaticRouter)
![2019-07-27-09-21-30](http://img.blog.niubishanshan.top/2019-07-27-09-21-30.png)
- 创建路由上线文对象, 并获取到当前用户访问的路径 path
![2019-07-27-09-22-59](http://img.blog.niubishanshan.top/2019-07-27-09-22-59.png)
- 最后, 利用第一步导入的 StaticRouter 组件包裹一下之前生成的服务端渲染组件
![2019-07-27-09-24-17](http://img.blog.niubishanshan.top/2019-07-27-09-24-17.png)

代码[diff](https://github.com/luoquanquan/react-ssr-show/commit/51181e64b06d5eeaae8920f4e09392c6d2fef241)

最后的最后, 我们执行一下 `node index.js`, 浏览器打开 `localhost:9999`

![react-router-ssr](http://img.blog.niubishanshan.top/react-router-ssr.gif)

通过 gif 我们能发现, 我们的服务端渲染是货真价实的服务端渲染了. 查看源代码的 html 字符串没有任何的问题了.(如果有样式该咋办呢 🤔)

细心的同学可能发现了, 我们每次点击链接的时候页面都会整体刷新. 这里就又到了那个经典的面试题[前端：你要懂的单页面应用和多页面应用](https://juejin.im/post/5a0ea4ec6fb9a0450407725c), 我们的目的很简单, 只是需要 ssr 实现首屏的渲染, 之后就由客户端接管, 这样就结合了两者的优点. 需求是有了, 怎么实现咧~

# =================================================

![2019-07-27-09-38-00](http://img.blog.niubishanshan.top/2019-07-27-09-38-00.png)

# =================================================

聪明的同学已经猜到, 只要我们我们在服务端渲染的页面中也引入客户端渲染生成的 `bundle.js` 文件是不是就 OK 了呢.那我们就试试咯

![2019-07-27-09-41-08](http://img.blog.niubishanshan.top/2019-07-27-09-41-08.png)

修改 `server.js` 引入 `koa-static` 用于托管静态文件, 文件整体 [diff](https://github.com/luoquanquan/react-ssr-show/commit/eef7ac6dbda66ac1ac4236dde5453d80994065e0) 如下:

![2019-07-27-09-43-36](http://img.blog.niubishanshan.top/2019-07-27-09-43-36.png)

废话少扯, 继续 `node index.js`, 浏览器打开 `localhost:9999` 搞起~

![react-router-dont-reload](http://img.blog.niubishanshan.top/react-router-dont-reload.gif)

服务端渲染的路由配置, 这就完成了~

![9150e4e5gy1g53l5pua7og208c08c0tr](http://img.blog.niubishanshan.top/9150e4e5gy1g53l5pua7og208c08c0tr.gif)

## 配置 redux, 带上身份畅玩澳门皇冠

### 客户端渲染引入 redux

由于配置 redux 不是我们的重点, 所以这里就不多说了, 简单配置一个 redux 开发环境. 代码 [diff](https://github.com/luoquanquan/react-ssr-show/commit/7f1cb6d181cec0e9cc6903115b8fabadbfd41686)

有疑问的问题, 评论区见~

![2019-07-27-12-26-06](http://img.blog.niubishanshan.top/2019-07-27-12-26-06.png)

### ssr 引入 redux

修改 server.js 文件如下:
```js
import path from 'path';
import Koa from 'koa';
import Router from 'koa-router';
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import serve from 'koa-static';

// 像客户端渲染一样导入 Provider 组件
import { Provider } from 'react-redux';
import App from './components/App.jsx';
import createStore, { init } from './store';

const app = new Koa();
const router = new Router();

const conf = {
  PORT: 9999,
};

const generateHtmlStr = reactDom => `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>

    <div id="app">${reactDom}</div>
    <script src="/dist/bundle.js"></script>
</body>
</html>
`;

router.get('*', (ctx) => {
  const context = {};
  const { url } = ctx.req;

  // 初始化一个 store
  const store = createStore();

  // 手动触发一下 init
  store.dispatch(init());
  // 首先把 React 组件变成一个字符串
  // eslint-disable-next-line
  const rNode = renderToString(
    // 把刚刚创建的 store 作为属性传给 Provider 组件
    <Provider store={store}>
      <StaticRouter location={url} context={context}>
        <App />
      </StaticRouter>
    </Provider>,
  );
  // 然后替换 template 里边的内容
  const domString = generateHtmlStr(rNode);

  // 最后返回 html 字符串
  ctx.body = domString;
});

app.use(serve(path.resolve(__dirname, '../')));
app.use(router.routes(), router.allowedMethods());

app.listen(conf.PORT, () => {
  console.log(`The Server is listening on ${conf.PORT} now, enjoy`);
});
```

代码 [diff](https://github.com/luoquanquan/react-ssr-show/commit/edf5555ca4e0ffd32877b1fcf6415e4008b24840) 如下:
![2019-07-27-17-32-51](http://img.blog.niubishanshan.top/2019-07-27-17-32-51.png)

还是那一套, 先 `node index.js` 再浏览器打开 `localhost:9999`~

![2019-07-27-17-37-54](http://img.blog.niubishanshan.top/2019-07-27-17-37-54.png)

浏览器执行结果如下图:
![redux-ssr](http://img.blog.niubishanshan.top/redux-ssr.gif)

细心的同学不难发现, 这个图片中抓牌的入口总是会闪动一下, 理论上讲我们执行了 store.dispatch(init()); 证明了用户是已经登录的用户, 所以应该是可以抓牌的才对. 这是问啥呢???

![2019-07-27-17-46-45](http://img.blog.niubishanshan.top/2019-07-27-17-46-45.png)

其实原因很简单, 我们的项目在首屏渲染完成以后就有客户端渲染接管了, 所以我们应该在客户端接管的时候把之前服务端渲染的数据保留下来. 怎么搞呢?

- 首先, 获取首屏情况下的状态
- 其次, 升级 server.js 文件, generateHtmlStr 中添加一个参数, 表示当前的状态
- 再次, 更新 html 模板, 在 window 下创建一个 REDUX_DATA 属性, 用于放置服务端渲染时首屏的全局状态
- 最后, 升级客户端相关的代码, 在 createStore 的时候把 REDUX_DATA 作为参数传入. 也正因此, 定义 REDUX_DATA 要放在引入 `bundle.js` 的上方.

这就完了, 通过下边的 gif 可以看出, 状态很好的保存下来了~

![redux-ssr-goog](http://img.blog.niubishanshan.top/redux-ssr-goog.gif)

到了这里, 可能有同学会问, 我们来到皇冠赌场, 全局状态肯定会灰常的多, 不应该只有一个登陆状态. 鉴于此, 我们扩充一下全局状态. 添加一个音乐列表. 一遍欢歌一遍看牌~

下来我们在看牌页面添加一个音乐列表, 我们听着音乐看着牌, 要是再吃着火锅那简直就是人生巅峰了.

![2019-08-16-10-16-02](http://img.blog.niubishanshan.top/2019-08-16-10-16-02.png)

### 在 store 中获取异步数据

- 首先, 我们创建一个 mock.js, 主要就是访问异步接口获取歌曲列表
```js
import axios from 'axios';

export default () => axios('https://music.niubishanshan.top/api/v2/music/toplist')
  .then(({ data }) => data);

// 在 mock.js 中我们引用了高端的 ajax 请求库 axios. 那么就不得不 npm i axios -S 啦
```
- 其次, 升级 store.js 添加歌曲相关的 actions 和 reducer.
![2019-08-16-11-31-25](http://img.blog.niubishanshan.top/2019-08-16-11-31-25.png)
- 由于我们有之前的单 reducer 升级成了两个 reducer, 此时要同步升级之前连接过 redux 的组件 `Header Home Zhuapai`
- 最后, 在看牌页面, 我们引入音乐. party Time
![2019-08-16-11-40-48](http://img.blog.niubishanshan.top/2019-08-16-11-40-48.png)

到目前为止, 代码是 [这样](https://github.com/luoquanquan/react-ssr-show/commits/v3.0.0) 的.

废话少说...

![error-kanpai](http://img.blog.niubishanshan.top/error-kanpai.gif)

![1565938746](http://img.blog.niubishanshan.top/1565938746.png)

卧槽, 居然不好使...

仔细看一下, 原来是 action 里边不能包含异步. 这个简单. 我们[升级](https://github.com/luoquanquan/react-ssr-show/commit/b9e91ed8eaf9d0a859bbf2c9df66c4c25dca966c)一下.

然后...

![ok-kanpai](http://img.blog.niubishanshan.top/ok-kanpai.gif)

### 服务端渲染支持首屏展示初始异步数据

> 有的时候, 我们服务端渲染的首屏网页也需要从其他异步接口来获取初始化数据. 此时就需要吧 html 返回给前端前先去访问异步接口. 那么怎么办咧...

- 首先, 给需要预先请求异步数据的组件加一个静态属性[就是需要请求数据的方法](https://github.com/luoquanquan/react-ssr-show/commit/e14c36ffa4822b5861b4020b886bf4cfebee913d)
- 其次, 升级 server.js [就像这样](https://github.com/luoquanquan/react-ssr-show/commit/73bc58850ca9bc2407164e8bf3614f7282a8e5fa)

最后...

![11a88f499af3f3b5d80b756fd](http://img.blog.niubishanshan.top/11a88f499af3f3b5d80b756fd.jpg)

![polyfill-error](http://img.blog.niubishanshan.top/polyfill-error.gif)

这里建议大家复习一下 [polyfill 和 preset 的区别](https://stackoverflow.com/questions/47255455/babel-polyfill-vs-babel-plugins)

反正我就这么[干了一把](https://github.com/luoquanquan/react-ssr-show/commit/ff6c64c0172cc77bba17f036624ac1a0594a2262)

然后就...

![polyfill-ok](http://img.blog.niubishanshan.top/polyfill-ok.gif)

是不是可以痛快的玩耍啦 😺~

![42cdf9611d1efb032b2b5b396](http://img.blog.niubishanshan.top/42cdf9611d1efb032b2b5b396.png)
