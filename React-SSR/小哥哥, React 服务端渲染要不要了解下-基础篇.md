咱也不知道这会儿写这个出来算不算过时, 反正就想写写. 至于有没有人看, 看完点不点赞, 点完赞会不会实践. 咱也不敢问呀, 随手写写吧~

![2019-06-24-16-44-54](http://img.blog.niubishanshan.top/2019-06-24-16-44-54.png)

到了 9102 年, 作为前端扛把子的 React 依然炙手可热. 周边的各种生态更是红的发烫. 每天应付完各种业务需求真的想舒舒坦坦躺上一波...

![2019-06-24-17-12-03](http://img.blog.niubishanshan.top/2019-06-24-17-12-03.png)

偏偏又来了互联网寒冬的夹持, 苦逼的前端 🐶只能把这句不怎么当讲的话埋在心底了~

![2019-06-24-17-13-37](http://img.blog.niubishanshan.top/2019-06-24-17-13-37.png)

这一期, 和大家一同学习 React + Koa 的服务端渲染知识. 文中不免疏漏, 望大佬斧正. 废话不多说, 搞起来~

![6af89bc8gw1f8ss9thu1sg20a305janv](http://img.blog.niubishanshan.top/6af89bc8gw1f8ss9thu1sg20a305janv.gif)

## 首先, 还是要先配一个舒服的开发环境

配置开发环境参考[这里](https://juejin.im/post/5bfd43986fb9a049ed308f1a), 由于咱们创建的是 React 项目, 所以按照这篇文章配置到 eslint 就可以了.

ps: 最新版本的 eslint 更新后和那篇文章命令行步骤有点不一样. 但是总体大同小异. 如果不想解决差异的同学可以安装 eslint 5.9.0 这个版本, 保证和文章中一致性~

以上几步操作的步骤为:

![init](http://img.blog.niubishanshan.top/init.gif)

## Hello world

首先我们创建一个基础的 react 项目, 为了便于理解, 这里不用 `create-react-app`, 而是直接用 webpack 手撸配置. 所以需要大家有一点点的 webpack 基础知识. 如果实在没有也没关系, 咱们讲~

开发环境创建完成后我们看到的项目结构应该添加了 `package.json` 和 `.eslintrc.js` 两个配置文件还有一个 `node_modules` 目录. 接下来就是我们大干一场的时间啦~

![2019-06-24-17-39-11](http://img.blog.niubishanshan.top/2019-06-24-17-39-11.png)

- 首先, 在项目的根目录创建一个 index.js
- 编写 index.js 文件, 文件内容如下.
```js
console.log('hello world')
```
- 到了这一步打开我们的命令行, 输入命令 `node index.js` 回车
![success](http://img.blog.niubishanshan.top/success.gif)

恕我直言, 在座各位都已经晋级为 `node.js` 开发工程师了~

![9150e4e5gy1g08qehjq9dg206j06j0tx](http://img.blog.niubishanshan.top/9150e4e5gy1g08qehjq9dg206j06j0tx.gif)

## 起步, React 版本的 HelloWorld

- 首先安装 React ReactDOM `npm i react react-dom`
- 搭建项目目录如下
![2019-07-11-18-42-03](http://img.blog.niubishanshan.top/2019-07-11-18-42-03.png)
其中, src 目录为项目源码目录, client.js 文件为客户端渲染的入口文件, components 目录下的文件为 react 组件文件.
- 创建并编写 App.jsx 文件
```js
export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }

  render() {
    return (
      <div className="ssr-show">
        <h1>欢迎来到澳门皇冠赌场</h1>
      </div>
    );
  }
}
```
- 编写 client.js 文件
```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';

// eslint-disable-next-line
ReactDOM.render(<App />, document.getElementById('app'));
```
到了这一步, 似乎是没有啥问题了, 但是, 这个东西跑不起来呀...
![2019-07-11-19-02-42](http://img.blog.niubishanshan.top/2019-07-11-19-02-42.png)

之前的步骤根本没有 html 作为依托, 咱们的项目更本就是不好使的. 此时的代码在 [这里](https://github.com/luoquanquan/react-ssr-show/tree/d49fd6796ed0071c6d01b8e81c94649af828a014) 建议大家下载下来尝试配置下 webpack

## 配置 webpack

配置 webpack 其实灰常简单, 只需要三步.
- 配置入口和输出
- 限定编译规则
- 生成 html 文件托管打包成果

根据三步原理创建的 webpack 配置文件如下
```js
const path = require('path');
const HtmlWebPackPlugin = require('html-webpack-plugin');

module.exports = {
  // 客户端渲染的入口文件
  entry: './src/client.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    // 因为目前我们的项目里边只有 js 和 jsx 文件, 所以只配这一条规则就可以啦~
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react'],
          },
        },
      },
    ],
  },
  plugins: [
    // 使用 html webpack plugin 创建 html 文件作为项目产出的依托文件
    new HtmlWebPackPlugin({
      template: './index.temp.html',
    }),
  ],
};
```
最后添加 `npm script`
在 `package.json` 的 script 字段下添加以下代码
`"build:client": "webpack --mode=production"`
![2019-07-11-19-33-46](http://img.blog.niubishanshan.top/2019-07-11-19-33-46.png)

最后执行 `npm run build:client` 结束后用你喜欢的浏览器打开 index.html 文件.
![2019-07-11-19-35-45](http://img.blog.niubishanshan.top/2019-07-11-19-35-45.png)

进行到这里的同学, 恭喜你正在悄悄的超越你们的项目小组长了~ 此时的代码在[这里](https://github.com/luoquanquan/react-ssr-show/tree/6b1aa38d3feae5cabe6db785672c25346baed4ce)
![9150e4e5ly1fq0seoi54ng208c08cq4d](http://img.blog.niubishanshan.top/9150e4e5ly1fq0seoi54ng208c08cq4d.gif)

## 配置服务端渲染

我们已经能实现客户端渲染的 React 项目, 走出了万里长征的第一步. 接下来就是剩下的 9999 步了.
![2019-07-12-11-08-39](http://img.blog.niubishanshan.top/2019-07-12-11-08-39.png)

首先, 在 src 目录下创建 `server.js` 文件作为服务端渲染的入口文件.
其次, 编辑该文件内容如下:
```js
import Koa from 'koa';
import Router from 'koa-router';
import React from 'react';
import { renderToString } from 'react-dom/server';
import App from './components/App.jsx';

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
</body>
</html>
`;

router.get('*', (ctx) => {
  // 首先把 React 组件变成一个字符串
  // eslint-disable-next-line
  const rNode = renderToString(<App />);
  // 然后替换 template 里边的内容
  const domString = generateHtmlStr(rNode);

  // 最后返回 html 字符串
  ctx.body = domString;
});

app.use(router.routes(), router.allowedMethods());

app.listen(conf.PORT, () => {
  console.log(`The Server is listening on ${conf.PORT} now, enjoy`);
});
```

代码和客户端渲染的入口文件 `client.js` 文件大同小异

再次, 改写项目根目录下的 `index.js` 文件, 导入并执行一下刚刚创建的 `server.js` 文件
```js
require('./src/server')
```

最后, 命令行执行 `node index.js`

![2019-07-12-11-18-13](http://img.blog.niubishanshan.top/2019-07-12-11-18-13.png)

![2019-07-12-11-17-18](http://img.blog.niubishanshan.top/2019-07-12-11-17-18.png)

仔细看下报错信息, 原来是 nodejs 不怎么认识 es6 的模块化语法. 给我们的项目入口文件添加个 babel. 修改项目根目录下的`indes.js` 文件如下:
```js
require('@babel/register')({
  presets: ['@babel/env', '@babel/react'],
});
require('./src/server');
```

再执行一下 `node index.js`

![2019-07-12-11-32-27](http://img.blog.niubishanshan.top/2019-07-12-11-32-27.png)

看样子要成功了~

![2019-07-12-11-33-16](http://img.blog.niubishanshan.top/2019-07-12-11-33-16.png)

这个时候浏览器打开 `localhost:9999` 如果你进入了澳门皇冠, 那肯定就值得信赖啦~

然后我们鼠标右键 -> 查看网页源代码 -> 验货
![2019-07-12-12-01-47](http://img.blog.niubishanshan.top/2019-07-12-12-01-47.png)

虽然很简单, 但是货真价实, 正儿八经的服务端渲染. 此时代码在 [这里](https://github.com/luoquanquan/react-ssr-show/tree/87c9508368b055687af7a5defcc26abb59aa376e)

## 后记

通过之前的步骤我们已经学习了使用 koa 实现最最基础的服务端渲染项目(仅仅用到了一个 renderToString 这个 api), 在下篇文章中我们会一起学习在项目中加入 `react-router` `redux` `自定义 mata` 等功能, 并实现其服务端渲染. 第一次接触服务端渲染, 希望各路大佬能不吝赐教~
![2019-07-12-11-51-56](http://img.blog.niubishanshan.top/2019-07-12-11-51-56.png)