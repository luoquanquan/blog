> 有了顺手的开发环境, 就完成了万里长城第一步. 接下来, 我们就用仅前段的方式实现测试 api 调用, 把接口返回的结果展示到我们的页面上. ready Go ^_^

## 引入 element-ui

- 登录 element [官网](http://element-cn.eleme.io/#/zh-CN)
- 查看安装[文档](http://element-cn.eleme.io/#/zh-CN/component/installation)
- 按照文档的意思, 首先执行 `npm i element-ui -S`
- 按照文档提示, 在 `main.js` 中引入 element 代码库, 并使用. 修改后 `main.js` 代码如下图所示
![2018-12-03-19-53-59](https://user-gold-cdn.xitu.io/2018/12/4/167798825762af0b?w=1560&h=780&f=png&s=157582)
- 引入完成后就可以去 `App.vue` 里测试一个 `button` 组件小试牛刀啦. 粘贴以下代码替换 App 文件中的代码即可.

```vue
<template>
  <div id="app">
     <el-button>按钮组件, 小试牛刀</el-button>
  </div>
</template>

<script>

export default {
  name: 'App',
};
</script>

<style>
</style>
```

此时, 访问浏览器的 [这个地址](http://localhost:8080/), 记得 `npm run dev` 昂. 如果你的页面中出现一个漂亮的 `button`, 说明你就成功辣 ^_^.
![2018-12-04-20-24-02](https://user-gold-cdn.xitu.io/2018/12/4/1677988256d97fe2?w=2558&h=636&f=png&s=65321)

## 照着自己喜欢的样子搞一套ui

```vue
<template>
  <div class="main-wrap">
    <el-form ref="form" label-width="80px">
      <el-form-item label="接口地址">
        <el-input v-model="url" placeholder="要包含 http OR https 喲">
          <el-button slot="append" icon="el-icon-arrow-right"></el-button>
        </el-input>
      </el-form-item>
      <el-form-item label="请方式">
        <el-select v-model="method" placeholder="请选择接口请求的方式">
          <el-option label="GET" value="GET"></el-option>
          <el-option label="POST" value="POST"></el-option>
        </el-select>
      </el-form-item>
      <h3>请求信息</h3>
      <el-form-item label="请求头">
        <el-input type="textarea" v-model="header" placeholder="key1: value1
key2: value2"></el-input>
      </el-form-item>
      <el-form-item label="请求参数">
        <el-input type="textarea" v-model="params" placeholder="key1: value1
key2: value2"></el-input>
      </el-form-item>
      <h3>响应信息(拖动右下角的三角可以控制展示框的大小)</h3>
      <el-form-item label="响应头">
        <el-input type="textarea" v-model="resHeaders" placeholder="key1: value1
key2: value2"></el-input>
      </el-form-item>
      <el-form-item label="响应数据">
        <el-input type="textarea" v-model="resData" placeholder="key1: value1
key2: value2"></el-input>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>

export default {
  name: 'Main',
  data() {
    return {
      // 接口地址
      url: 'https://api.github.com/orgs/learn-cli-organization/repos',
      // 请求方法
      method: 'GET',
      // 请求头
      header: '',
      // 请求参数
      params: '',
      // 响应头
      resHeaders: '',
      // 响应字段
      resData: '',
    };
  },
  methods: {
  },
};
</script>

<style lang="less" scoped>
.main-wrap {
  width: 100%;
  height: 100%;
  overflow: auto;
}
</style>
```

上边的代码, 是我比较喜欢的样纸. 这个真的不好再解释了, 如果有小伙伴看不懂. 还是那句话. 评论区见 ^_^
![2018-12-04-20-53-32](https://user-gold-cdn.xitu.io/2018/12/4/16779882555b3f40?w=796&h=554&f=png&s=212243)

当然啦 `好好学习, 天天向上` 还是少不了的 [Element官网](http://element-cn.eleme.io/#/zh-CN).

## 引入 axios, 完善代码结构

> axios [官网](https://www.npmjs.com/package/axios) 是一款 AJAX 请求工具, 兼容前后端. 同时也是我的程序员偶像 [尤雨溪](https://weibo.com/p/1005051761511274/home) 推荐. 明人不说废话.

![9150e4e5ly1fsj7mpc8tkg205y05ytc1](https://user-gold-cdn.xitu.io/2018/12/4/16779882563a9644?w=214&h=214&f=gif&s=132949)

- 首先, 执行 `npm i axios -S` 安装 axios
- 根目录下创建 `utils` 文件夹, 并在该文件夹下创建 `index.js` `request.js` 两个文件内容如下.

request.js

```js
import axios from 'axios';
export const request = axios;
```

index.js

```js
export { request } from './request';
```

很简单是不是, 其实开发就是这么简单有趣 😄. 其中, request.js 作为请求层封装, 现在看起来没什么用, 但是如果项目复杂的话可以做公共配置. index.js 作为 utils(工具箱的意思) 的统一出口想外部导出工具方法.

- 其次, 调整一下 components 目录下的代码. 此时代码结构如下:
![2018-12-04-21-18-51](https://user-gold-cdn.xitu.io/2018/12/4/1677988264877189?w=504&h=1348&f=png&s=106549)
相对比较清晰, components 下的三个文件分别对应了单页面应用的三个部分. 这里也亮一下代码, 让大家放心.

header.vue

```vue
<template>
  <div class="header-wrap">
    在线 http 接口测试工具 <i class="el-icon-success"></i>
  </div>
</template>

<script>
export default {
  name: 'Header',
};
</script>

<style>
.header-wrap {
  width: 100%;
  height: 50px;
  text-align: center;
  line-height: 50px;
  color: #333;
}
</style>
```

footer.vue

```vue
<template>
  <div class="footer-wrap">
    项目纯属娱乐, 返回的结果不一定是对的, 请自行斟酌~~
  </div>
</template>

<script>
export default {
  name: 'Footer',
};
</script>

<style>

.footer-wrap {
  width: 100%;
  height: 50px;
  line-height: 50px;
  text-align: center;
  color: #333;
}
</style>
```

App.vue

```vue
<template>
  <div id="app">
    <Header></Header>
    <Main></Main>
    <Footer></Footer>
  </div>
</template>

<script>

import Header from './components/header';
import Footer from './components/footer';
import Main from './components/main';

export default {
  name: 'App',
  components: {
    Header,
    Footer,
    Main,
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
```

为啥没有 main.vue 呢? 因为就是我们尝试搞 UI 的那套, 我不会告诉你粘贴过来, 改一条 css 就能跑
![2018-12-04-21-25-01](https://user-gold-cdn.xitu.io/2018/12/4/1677988257cfd980?w=1542&h=364&f=png&s=87108)
- 最后, 我们在主文件中引入封装好的 axios `import { request } from '@/utils';`
- 为第一个 input 右侧的放大镜 icon 添加点击事件. <el-button slot="append" icon="el-icon-arrow-right" **@click="onSubmit"**></el-button>
- 在 methods 里添加 onSubmit 方法:

```js
methods: {
    async onSubmit() {
      const headers = {};
      const params = {};
      // 解析请求头
      this.header
        .split('\n')
        .map(header => header.split(':').map(item => item.trim()))
        .forEach((header) => {
          if ([header[0]] && header[1]) headers[header[0]] = header[1];
        });
      // 解析请求参数
      this.params
        .split('\n')
        .map(param => param.split(':').map(item => item.trim()))
        .forEach((param) => {
          if ([param[0]] && param[1]) params[param[0]] = param[1];
        });
      console.log('headers: ', headers, 'params: ', params);
      try {
        // 使用我们封装的 ajax 请求工具, 直接向测试 api 发起请求.
        const { headers: resHeaders, data } = await request(
          Object.assign({}, {
            url: this.url,
            method: this.method,
            headers,
          }, this.method === 'GET' ? { params } : { data: params }));
        // 将响应的响应头写到响应头展示框
        this.resHeaders = Object.keys(resHeaders).map(key => `${key}: ${resHeaders[key]}\n`).join('');
        // 将响应的结果数据写到结果展示框
          // JSON.stringify(data, null, 2) 这个方法可以理解为是做格式化, 让数据更好看
        this.resData = JSON.stringify(data, null, 2);
      } catch (e) {
        console.log(e);
        this.$message({
          message: '接口请求异常, 请联系 ⭕️️️⭕️️️',
          type: 'error',
        });
      }
    },
  }
```

PS: 放上一系列处理以后的代码的[github地址](https://github.com/luoquanquan/vue-fe/tree/0.0.1)

## 最后, 验证

首先, 我们验证一个在[这篇文章](https://juejin.im/post/5bec598d51882579117f61f8)中提过的接口(获取组织 github 项目列表), 效果是这样子的.
![ok-api-3245678hewew](https://user-gold-cdn.xitu.io/2018/12/4/16779882a59a15f0?w=1165&h=636&f=gif&s=2674071)
啥都别说了, 这效果简直 666.
![2018-12-04-21-45-41](https://user-gold-cdn.xitu.io/2018/12/4/16779882a8650770?w=556&h=506&f=png&s=180817)

趁着兴头, 我们要不再测试一个? 来个百度的接口试试.
![baidu-api-3245678hewew](https://user-gold-cdn.xitu.io/2018/12/4/16779882a5e63e53?w=1165&h=636&f=gif&s=3387262)

如上图, 我们抓取到百度首页获取搜索推荐的接口, 接口 url 如下. 请求类型为 GET
`https://www.baidu.com/s?ie=utf-8&mod=1&isid=3A58FCCEBA622663&ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd=%E4%B8%BA%E4%BB%80%E4%B9%88%E5%8D%95%E8%BA%AB%E7%8B%97%E6%B2%A1%E6%9C%89%E5%A5%B3%E6%9C%8B%E5%8F%8B&rsv_pq=e4fac4eb00025263&rsv_t=2b97p3R1eUvOQAKm0wyNxRv9QYRSkcIvfh0h4EeDGfbqTK%2BSSH6%2BG63KrsE&rqlang=cn&rsv_enter=1&rsv_sug3=34&rsv_sug1=33&rsv_sug7=100&rsv_sid=27938_1456_21125_20719&_ss=1&clist=&hsug=&csor=11&pstg=5&_cr1=29797`

最后, 把刚刚获取到的接口地址拿到我们的工具里试一把 😄
![bad-api-3245678hewew](https://user-gold-cdn.xitu.io/2018/12/4/16779882ac2e206f?w=1165&h=636&f=gif&s=887238)
哎呀我滴妈, 报错了 😓

下面贴出完整的报错信息, 供参考

```shell
Access to XMLHttpRequest at 'https://www.baidu.com/s?ie=utf-8&mod=1&isid=3A58FCCEBA622663&ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd=%E4%B8%BA%E4%BB%80%E4%B9%88%E5%8D%95%E8%BA%AB%E7%8B%97%E6%B2%A1%E6%9C%89%E5%A5%B3%E6%9C%8B%E5%8F%8B&rsv_pq=e4fac4eb00025263&rsv_t=2b97p3R1eUvOQAKm0wyNxRv9QYRSkcIvfh0h4EeDGfbqTK%2BSSH6%2BG63KrsE&rqlang=cn&rsv_enter=1&rsv_sug3=34&rsv_sug1=33&rsv_sug7=100&rsv_sid=27938_1456_21125_20719&_ss=1&clist=&hsug=&csor=11&pstg=5&_cr1=29797' from origin 'http://localhost:8080' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

综上, 测试访问 github 接口的时候没有任何问题, 但是当我们尝试访问百度接口的时候, 出现了报错. 这是为什么呢? 错误信息如上, 大家可以百度一下[试试呗](https://www.baidu.com)

下集预告: 由于访问百度接口报错, 仅仅通过前端技术就要实现一个接口测试工具的方案是行不通的. 所以我们要考虑解决当前出现的问题. 处理跨域问题, 我们下集再见.
