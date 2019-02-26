> 自信失败并不要紧, 回到公司继续划水, 直到有一天. 从不过问前端技术的傻领导蹦出来一个P: '我感觉咱们的页面白屏时间有点长呀!!!'. 小样, 本来我是不想忽悠你的, 你竟然把我逼到这个份儿上了昂.
![2019-02-20-21-51-33](http://img.blog.niubishanshan.top/2019-02-20-21-51-33.png)

> 在[上一步](https://juejin.im/post/5c4e81c3f265da613c0a2952)中我们提到: 为了在浏览器端实现 Common 的模块化效果, webpack 把所有的 js 文件打包到了一个 bundle.js 文件里, 随着项目复杂度提升, bundle.js 文件也越来越大. 没想到这货甩给我一句这话.
![2019-02-20-21-48-24](http://img.blog.niubishanshan.top/2019-02-20-21-48-24.png)

装逼装过头了, 知识不够用了, 怎么办? 得抓紧学习一波了. 搞起...

![2019-02-26-20-00-24](http://img.blog.niubishanshan.top/2019-02-26-20-00-24.png)

在很久很久以前, 我还在培训机构学习着 HTML5 开发技术, 上课打盹的时候偶然听到培训讲师说了一句说: React 就是 facebook 前端团队在开发 instagram 的时候觉得市面上没有好用的前端框架就自己搞出来的一套东西. 很明显 instagram 应该是这个世界上早期的单页面应用之一了. 废话少说, 打开网页[来一发](https://www.instagram.com/)给我的傻领导看看世界顶尖级互联网公司的 大 大大 大 bundle.js ...

![2019-02-26-20-22-50](http://img.blog.niubishanshan.top/2019-02-26-20-22-50.png)

WTF!!! 居然没有 bundle.js 这么优秀的 js 文件的, 最大的 js 文件 gzip 之后只有 240K. 这是啥? 为什么没了 bundle? 这是怎么实现的? 培训班老师没教鸭~

![2019-02-26-20-29-12](http://img.blog.niubishanshan.top/2019-02-26-20-29-12.png)

经过我和度娘还有谷哥的一致商议, 确定了通过 webpack 实现 instagram 这种代码分包(英文名 code split)有两种方式:

## 第一种: 通过 `require.ensure` 实现代码分包处理

光说不练假把式, 管他什么效果, 先百度粘一段代码跑起来再说.

修改我们的项目代码 `index.js`
```js
require.ensure([], (require) => {
    const foo = require('./foo');
    console.log(foo);
})

console.log('我是高级前端工程师~');
export default {};
```

就是把 `import foo from './foo'` 替换成了 `require.ensure` 的写法.

`require.ensure` 能接收三个参数

参数名 | 参数类型 | 是否必填
--- | --- | ---
依赖列表 | array | 是
回调函数 | func | 是
模块名称 | string | 否

再次执行 `npm run build` 编译后的代码目录如下图所示.

![2019-02-26-20-50-26](http://img.blog.niubishanshan.top/2019-02-26-20-50-26.png)

为了演示分包加载的效果, 我们手动添加了一个 `index.html` 用于引入打包生成的 js 文件[代码地址](https://github.com/luoquanquan/webpack-show/commit/9decb3a0b29d003a669727c1f97dd9e2669fa76d)




