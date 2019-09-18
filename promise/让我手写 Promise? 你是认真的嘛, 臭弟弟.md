> [Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise), es6 新增的用于异步处理的 api, 在一般的情况下: api 其实会用了就行. 但是在某些 "特殊" 的场合有些弟弟就是要要求你手写一个 Promise 你说恶心不恶心...

![2019-08-28-19-06-28](http://img.blog.niubishanshan.top/2019-08-28-19-06-28.png)

## 了解需求

作为一个业务猿, 开始动手干之前一定要了解 PM 的需求, 写一个表单是这样. 写一个 promise 也是同理. 那么这里正好有份[需求文档](https://promisesaplus.com/)供我们参考.

纯英文的文档, 很不爽有没有. 然而, 每次做完需求产品验收时候那帮货总会质疑我的语文水平也不过关. 总之, 有这么个文档, 看不懂就先虾鸡啪写. 撸出来再说...

![2019-09-18-17-11-05](http://img.blog.niubishanshan.top/2019-09-18-17-11-05.png)

当然, 英文的需求文档看起来如果比较吃力的话, 这里还有[中文版本](http://www.ituring.com.cn/article/66566)的需求文档

![8e508a429ef2717ec2f44e60e](http://img.blog.niubishanshan.top/8e508a429ef2717ec2f44e60e.gif)

## 根据印象写一个

我相信, 需求文档是没几个人看完了...

既然大家都是性情中人, 那么咱么就随便写写咯

![c8b6f978f7d320cae5d1583d1](http://img.blog.niubishanshan.top/c8b6f978f7d320cae5d1583d1.gif)

万里长征第一步, 我们先初始化我们的项目, 整个项目中有两个文件 `index.html + promise.js` 他在[这里](https://github.com/luoquanquan/promise-show/commits/v0.0.1)

首先, 我们原生的 Promise 写一段牛逼的测试代码

```js
const p = new Promise((resolve) => {
    resolve('success')
})

p.then(value => {
    console.log('success: ', value)
})
```

创建一个 promise 对象, 并且直接 resolve 掉它. 在浏览器打开 `index.html` 效果如图:
![2019-08-28-19-48-42](http://img.blog.niubishanshan.top/2019-08-28-19-48-42.png)

看上去不是很难呀, 我们着手搞一个我们自己的 `promise.js` 文件, 实现一个相同的功能.

```js
class Promise {
    constructor(executor) {
        this.value = null

        const resolve = value => {
            this.value = value
        }

        executor(resolve)
    }

    then (onFullfilled) {
        onFullfilled(this.value)
    }
}
```

![2019-09-18-19-03-26](http://img.blog.niubishanshan.top/2019-09-18-19-03-26.png)