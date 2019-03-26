# 手写实现 call apply bind

> 三种常用的 js 改变 this 的方式

## call

### call 使用方法

```js
function willCall(param1, param2) {
    console.log('this.a, this.b', this.a, this.b)
    console.log('param1, param2', param1, param2)
}

const obj = {
    a: 1,
    b: 2
}

willCall.call(obj, 'i am regret very much~', 'can you forgive me?')
// this.a, this.b 1 2
// param1, param2 i am regret very much~ can you forgive me?
```

### call 原生实现

不难发现其实 call 就是 `function willCall` 的一个方法. 执行 call 方法时传入的第一个参数就是函数体`willCall`内部的 this, 后续的参数就是函数体 `willCall` 接收的参数~

首先, 要为 function 添加方法则可以直接在 Function 原型对象上添加.

```js
Function.prototype.customCall = function() {
    // 1212
}
```

接下来进一步完善 customCall 方法

```js
// 通过默认参数设置默认上下文为 null
// 通过 restArgs 手机函数在执行 customCall 调用时传入的参数
Function.prototype.customCall = function(context = null, ...restArgs) {
    // 缓存 context 的 func 属性
    const {func} = context;

    // 此时我们调用 customCall 的函数就被挂载到了 context 上作为它的一个方法了
    context.func = this;

    // 接收将 func 作为 context 的方法调用时返回的值
    const ret = context.func(...restArgs);

    // 还原 context 的 func 属性
    context.func = func;

    // 返回函数执行的结果
    return ret;
}

// ==========  测试代码  ========================================================\
function willCall(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
    return 'success';
}

const obj = {
    a: 1,
    b: 2
}

willCall.customCall(obj, 'i am regret very much~', 'can you forgive me?');
```

最后, 讲两句~

这种实现 call 方法的方式其实就是把即将调用的方法添加到要绑定 this 的对象上作为一个方法. 用对象再来调用它自己的方法那 this 妥妥的就是自己咯.

## apply

### apply 使用方法

apply 方法的使用和 call 相当类似. 只是传递参数的方式有点差异~

```js
function willApply(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
}

const obj = {
    a: 1,
    b: 2
}

willApply.apply(obj, ['i am regret very much~', 'can you forgive me?']);
// this.a, this.b 1 2
// param1, param2 i am regret very much~ can you forgive me?
```

不难发现, 在调用 willApply 的时候从第二个参数开始我们统一放到了一个数组里.

### apply 原生实现

```js
// 通过默认参数设置默认上下文为 null
// 通过 restArgs 手机函数在执行 customCall 调用时传入的参数
Function.prototype.customApply = function(context = null, args) {
    // 缓存 context 的 func 属性
    const {func} = context;

    // 此时我们调用 customCall 的函数就被挂载到了 context 上作为它的一个方法了
    context.func = this;

    // 接收将 func 作为 context 的方法调用时返回的值
    const ret = context.func(...args);

    // 还原 context 的 func 属性
    context.func = func;

    // 返回函数执行的结果
    return ret;
}

// ==========  测试代码  ========================================================\
function willApply(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
    return 'success';
}

const obj = {
    a: 1,
    b: 2
}

willApply.customApply(obj, ['i am regret very much~', 'can you forgive me?']);
// this.a, this.b 1 2
// param1, param2 i am regret very much~ can you forgive me?
// "success"
```

这就结了~

不得不说 es6 的 `...` 运算符真的是爽了不少呢.
