系列文章:

- [【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
- [【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)
- [【小哥哥, 跨域要不要了解下】CORS 进阶篇](https://juejin.im/post/5c0b5a8851882548e9380afb)
- [【小哥哥, 跨域要不要了解下】NGINX 反向代理](https://juejin.im/post/5c0e6d606fb9a049f66bf246)

> 原本本系列文章是不打算详写 NGINX 反向代理的. 至于为什么不想写呢? 当然是因为我不太会咯~~
![2018-12-10-17-47-38](https://user-gold-cdn.xitu.io/2018/12/10/167985b1ef655cd0?w=590&h=556&f=png&s=428450)
> 但是, 不久前有大佬点了这道菜, 那当然就得上啦 😄

## 代理是个啥

既然要聊反向代理, 那首先得知道代理是个啥吧? 嗯.

![2018-12-10-17-51-05](https://user-gold-cdn.xitu.io/2018/12/10/167985b1ce2c93ff?w=526&h=554&f=png&s=141451)

### 正向代理

比如, 你买束花, 想要给隔壁工位的测试妹子小丽表白. 但是又怕被人家直面拒绝太没面子. 于是你把鲜花委托给平时和小丽一起的测试小伙伴小红. 让她帮忙把花送给小丽. 这就是一个简单的代理过程, 小红作为代理帮你把花送给了小丽, 当然这种情况在现实中并不推荐使用, 因为难以避免中间商赚差价 😂.

在上面的例子中, 你作为客户端(请求方), 想要向服务方(小丽)发起请求. 但是碍于面子你主动找到了第三方(小红)作为代理向服务方发送请求, 这种情况就是常说的`正向代理`. 正向代理在互联网中的使用主要是科学上网, 你想访问谷歌但是碍于防火墙你只能通过vpn服务器作为代理才能访问. 这个时候一般也要找值得信赖的vpn厂商, 避免中间商赚差价 😄.

![2018-12-10-19-09-12](https://user-gold-cdn.xitu.io/2018/12/10/167985b206554946?w=1768&h=1078&f=png&s=2286941)

### 反向代理

关于反向代理的例子, 那就比较多啦. 比如, 孤独的你躺在床上夜不能寐. 于是乎, 拿出手机, 点亮了屏幕, 拨通 `10086`, 中国移动就会随机分配一个当前处于空闲的客服MM, 你可以和客服MM聊聊天, 问问她家住哪里, 有没有男朋友, 她的微信号, 她的手机号, 星座, 八字.......

在这个例子中, 中国移动就充当了反向代理的角色. 你只需要拨打 `10086`. 至于会不会分配到 MM 会分配到哪个 MM 在接通之前你都是不知道的. 反向代理在互联网中的使用主要是实现负载均衡. 当你访问某个网站的时候, 反向代理服务器会从当前网站的所有服务器中选择一个空闲的服务器为你响应. 用于均衡每台服务器的负载率.

![2018-12-10-19-09-56](https://user-gold-cdn.xitu.io/2018/12/10/167985b21a298a74?w=2094&h=1198&f=png&s=3034786)

## 修改 hosts 完成域名绑定

mac 用户直接执行 `vim /private/etc/hosts` 在 hosts 文件最后添加一行:

```nginx
127.0.0.1 a.com
```

这一句是什么意思呢? 就是告诉我们的电脑访问 `a.com` 的时候, 无需请求 DNS, 直接指向我们本机.

ps: win 环境下, hosts 文件在 `C:\Windows\System32\drivers\etc` 文件夹下. 如果没有权限修改, 把 hosts 文件先拷贝到别的位置, 通过编辑器打开并添加最后一行内容以后再剪切到原来的位置替换即可.

验证: 打开命令行窗口执行 `ping a.com`, 如果访问的 ip 为 127.0.0.1 说明我们的域名绑定就完成啦 ^_^

![6af89bc8gw1f8r5kn9ezgg202s02smzp](https://user-gold-cdn.xitu.io/2018/12/10/167985b1c6c774b8?w=100&h=100&f=gif&s=104157)

## 安装 nginx

> 要做 NGINX 反向代理, 肯定要安装 [nginx](http://nginx.org/), 本文安装步骤示例环境为 mac, win 的小伙伴, 可以百度一下嗷, 这个东西大同小异.

- 安装 brew 命令, 执行 `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
- 安装 nginx, 执行 `brew install nginx`
- 启动 nginx `nginx`, 如果报没有权限, 执行 `sudo nginx`

nginx 启动后, 浏览器打开 [localhost:8080](http://localhost:8080), 即可验证. 出现以下界面说明安装成功.
![2018-12-10-19-31-35](https://user-gold-cdn.xitu.io/2018/12/10/167985b43cdaf74d?w=2554&h=520&f=png&s=113493)

## nginx 配置初探

配置完 hosts 域名已经能够成功绑定. 现在如果我们访问 `a.com` 实际上是会访问到我们的自己的电脑辣. 那还不抓紧试一下?

浏览器访问 [a.com](http://a.com)

![2018-12-10-19-58-29](https://user-gold-cdn.xitu.io/2018/12/10/167985b14897aed9?w=790&h=544&f=png&s=294825)

这是什么鬼????
![2018-12-10-19-57-20](https://user-gold-cdn.xitu.io/2018/12/10/167985b1a15ca9c7?w=2554&h=1412&f=png&s=124776)

为什么会 **无法访问此网站** 呢? 我们下载安装完 nginx 还没有做任何配置. 接下来, 我们稍微配置一下就 OK:

- 命令行切换到 nginx 配置目录下 `cd /usr/local/etc/nginx/servers`
- 创建并编辑配置文件 `vim test.conf`, 在配置文件中粘贴以下内容

```shell
server {
    # 监听80端口号
    listen 80;

    # 监听访问的域名
    server_name a.com;

    # 根据访问路径配置
    location / {
        # 把请求转发到 https://www.baidu.com
        proxy_pass https://www.baidu.com;
    }
}
```

- 保存文件, 并执行 `ngins -s reload` 重启 nginx
- 回到浏览器, 打开 a.com 的页签, 强制刷新.
![2018-12-10-20-09-04](https://user-gold-cdn.xitu.io/2018/12/10/167985b43cfa74e4?w=2552&h=1258&f=png&s=120825)

恭喜你已经完成了第一个 nginx 配置.

![9150e4e5gy1ftr3ghw3rdg208c08cwg3](https://user-gold-cdn.xitu.io/2018/12/10/167985b1dbffb16d?w=300&h=300&f=gif&s=69588)

## 创建跨域环境

> 通过一系列的折腾, 我们已经可以通过 nginx 将`a.com` 转发到百度. 完成了第一步, 接下来我们创建跨域的 Case 并一步一步通过 nginx 配置实现跨域.

首先, 项目前后端添加 nginx 目录, 用户存放前后端代码. 代码结构如下图所示.
![2018-12-10-19-38-25](https://user-gold-cdn.xitu.io/2018/12/10/167985b2102c597b?w=524&h=674&f=png&s=54258)

其次编写前后端代码:

前端代码(`./fe/nginx/index.html`):

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
        xhr.open('GET', 'http://localhost:8888/api/getFriend')
        xhr.setRequestHeader('token', 'quanquanbunengshuo')
        xhr.withCredentials = true;
        xhr.onreadystatechange = function() {
            if(xhr.readyState === 4 && xhr.status === 200) {
                console.log(xhr.responseText)
                console.log(xhr.getAllResponseHeaders())
            }
        }
        xhr.send()
    </script>
</body>
</html>
```

编写完前端代码以后, 启动前端 web 容器. `live-server ./fe/nginx`
![2018-12-10-20-17-56](https://user-gold-cdn.xitu.io/2018/12/10/167985b23be6227f?w=1168&h=138&f=png&s=49185)

命令行中出现了黄色警告, 通知我们 8080 端口已经被占用, 这又是为什么呢? 大家请思考一哈.
![2018-12-10-20-21-29](https://user-gold-cdn.xitu.io/2018/12/10/167985b2513751c6?w=576&h=594&f=png&s=125819)

我们重新指定一个端口`live-server ./fe/nginx --port=9999` 哈哈, 换一个指令, 依旧是那么顺畅. ^_^

后端代码:

```js
const http = require('http');

const PORT = 8888;

// 创建一个 http 服务
const server = http.createServer((request, response) => {
  console.log(request.headers)
  response.end("{name: 'quanquan', friend: 'guiling'}");
});

// 启动服务, 监听端口
server.listen(PORT, () => {
  console.log('服务启动成功, 正在监听: ', PORT);
});
```

启动后端服务 `node ./be/nginx/index.js`

## 完善 nginx 配置

> 前后端代码已经准备完成, 这一步我们就来点干货. 完成最后的配置.

- 首先, 修改 nginx 配置, 把百度地址替换成本地的前端地址

```shell
server {
    # 监听80端口号
    listen 80;

    # 监听访问的域名
    server_name a.com;

    # 根据访问路径配置
    location / {
        # 把请求转发到 http://127.0.0.1:9999
        proxy_pass http://127.0.0.1:9999;
    }
}
```

- 修改完成 nginx 配置文件以后, 切记执行 `nginx -s -reload` 重启 nginx.
- 访问[a.com](http://a.com)
![2018-12-10-20-34-13](https://user-gold-cdn.xitu.io/2018/12/10/167985b2666fb873?w=2556&h=980&f=png&s=159892)

熟悉的报错又出现了...

- 修改前端项目中的接口地址

```javascript
// 接口地址修改为当前域名下 /api 路劲下的 getFriend
xhr.open('GET', '/api/getFriend')
```

- 修改 nginx 配置文件

```shell
server {
    # 监听80端口号
    listen 80;

    # 监听访问的域名
    server_name a.com;

    # 根据访问路径配置
    location / {
        # 把请求转发到 http://127.0.0.1:9999
        proxy_pass http://127.0.0.1:9999;
    }

    # 监听根目录下的 /api 路径
    location /api/ {
        # 把请求转发到 http://127.0.0.1:8888
        proxy_pass http://127.0.0.1:8888;
    }
}
```

新加的对于 api 路径的监听的意思就是把关于后端 api 的请求转发到后端项目上(哈哈, 当然这就是为啥好多后端接口都是要有 `/api` 开头的啦). 重启 nginx 以后, 再次刷新浏览器, 后端返回的结果已经成功的打印到了控制台, 本次跨域访问任务完成.
![2018-12-10-20-46-57](https://user-gold-cdn.xitu.io/2018/12/10/167985b27e613762?w=2554&h=1234&f=png&s=190087)

细心的小伙伴肯定发现了, 控制台还有一个报错. 这个是因为我们的项目中用到了 `live-server` 这个工具需要 websocket 导致的. 我们可以通过添加以下配置解决.

```shell
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

![2018-12-10-20-50-39](https://user-gold-cdn.xitu.io/2018/12/10/167985b2b48598ec?w=2556&h=1072&f=png&s=155531)

报错消失 😄, 此时完整的 nginx 配置文件为

```shell
server {
    # 监听80端口号
    listen 80;

    # 监听访问的域名
    server_name a.com;

    # 根据访问路径配置
    location / {
        # 把请求转发到 http://127.0.0.1:9999
        proxy_pass http://127.0.0.1:9999;

        # 兼容websocket
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # 监听根目录下的 /api 路径
    location /api/ {
        # 把请求转发到 http://127.0.0.1:8888
        proxy_pass http://localhost:8888;
    }
}
```

前后端代码地址为: [github](https://github.com/luoquanquan/cross-domain/commit/f38f56689fdac1526244ecadaa979a52c9c4a7ea)

## 总结

至此, 我们已经通过 nginx 反向代理的方式实现了跨域访问 api, 在系列文章[第一篇](https://juejin.im/post/5c07fa04e51d451de968906b)对于跨域的解释为: 跨域源于同源策略, 是浏览器保证用户安全的行为. 我们使用的 nginx 反向代理实际上是对浏览器的一种 "哄骗", 让它认为自己访问到的是同域的 api. 实际上是在服务端做了个调包, 这个道理就如同你拨打 10086 你就认定了给你分配到的一定是中国移动的客服MM(客服GG也是有可能出现的 😄)而中国移动的客服MM就是一个很安全的聊天对象, 没有必要再进行限制...

下集预告: 终于蹩脚的码完了最后一行, 作为生产环境中最常用的 nginx 反向代理, 比我想象的要简单很多很多. 由于涉及到诸多配置的步骤. 有写的不明白的地方还望小伙伴们评论区一起讨论. 下一节预计聊聊服务端代理 `ServerProxy` 这个也是我要做的[接口测试工具](https://juejin.im/post/5bfd43986fb9a049ed308f1a)需要用到的技术方案, See you.
