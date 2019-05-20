> 在[前一篇]()文章中, 我们遇到了一个问题 --- [跨域](). 关于跨域的具体知识, 在[上一个系列]()文章中有详细说明. 这里我们尝试解决跨域问题.

## 思路

既然直接通过浏览器实现接口测试有困难, 咋办捏?
![2019-05-20-20-06-07](http://img.blog.niubishanshan.top/2019-05-20-20-06-07.png)

可以用服务器去访问服务器, 在跨域系列文章里边提到过, 实现跨域的方案很多很多. 有风头正硬的 cors 也有明日黄花 jsonp. 这里求小伙伴们不要问我孰优孰略, 技术这个东西只有合不合适, 没有优劣可言. 一句话, 用着爽就好~

这里默认认为大家对跨域知识了然于胸, 这里直接上我们实现接口测试的技术思路
![2019-05-20-20-24-04](http://img.blog.niubishanshan.top/2019-05-20-20-24-04.png)

首先, 开发环境下, 前端项目要访问我们自己的服务器(也就代理服务器)也存在跨域问题. 这里通过 cors 实现跨域访问, 其次就是访问 api 服务器了, 直接用代理服务器带着前端传来的参数去访问即可. 最后就是 api 服务器返回值的处理了. 把 api 服务的响应头和响应 body 直接返回给前端就好了呀. 前端展示两个 textarea.

## 实现后端请求 apiServer

### 初始化一个 node 项目

1. 首先在 github 上创建一个空的项目
![create-repo](http://img.blog.niubishanshan.top/create-repo.gif)
为什么要先创建 github 项目呢?
![2019-05-20-20-33-19](http://img.blog.niubishanshan.top/2019-05-20-20-33-19.png)

1. 本地克隆刚刚创建的项目 `git clone {{你的项目地址}}`
1. 进入你的项目目录`cd {{你的项目名称}}`
1. 初始化 `npm init -y`
1. 如果没有出现意外的话, 项目目录下会创建一个 package.json 文件, 文件内容:
```json
{
  "name": "proxyServer",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/luoquanquan/proxyServer.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/luoquanquan/proxyServer/issues"
  },
  "homepage": "https://github.com/luoquanquan/proxyServer#readme"
}
```
如果之前创建过 node 项目的兄弟们应该知道, 如果自己直接执行 `npm init` 的话也可以生成这个文件, 但是明显不会有各种 github 相关的链接.

### 写一个 hello world

1. 万年第一步, 配上 eslint 没有格式化工具我是没法写代码的
1. 安装 koa 执行 `npm i koa -S`
1. 然后创建一个 `index.js` 把 koa [官网](https://www.npmjs.com/package/koa) 实例代码拷贝过来
1. 最后执行 ``