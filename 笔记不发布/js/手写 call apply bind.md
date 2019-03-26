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

## bind

bind 是一个很有意思的用法, 它返回的是一个包装函数, 相当于把当前的函数 wrapper 了一层, 并且可以分两次传递参数~

### bind 的用法

```js
function willBind(param1, param2) {
    console.log('this.a, this.b', this.a, this.b)
    console.log('param1, param2', param1, param2)
}

const obj = {
    a: 1,
    b: 2
}

const boundFunc = willBind.bind(obj, 'i am regret very much~');
boundFunc('can you forgive me?')
// this.a, this.b 1 2
// param1, param2 i am regret very much~ can you forgive me?
```

### bind 原生实现

```js
Function.prototype.customBind = function(context, ...restArgs) {
    // 这里能把 this(就是你最终要执行的函数)
    // 和 context 保存起来形成一个闭包
    // restArgs 为执行 bind 时传入的已知参数
    const self = this;
    return function(...args) {
        return self.apply(context, [...restArgs, ...args])
    }
}

// ==========  测试代码  ========================================================\
function willBind(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
    return 'success'
}

const obj = {
    a: 1,
    b: 2
}

const bound = willBind.customBind(obj, 'i am regret very much~');
bound('can you forgive me?')
// this.a, this.b 1 2
// param1, param2 i am regret very much~ can you forgive me?
```

看上去已经没有了任何的问题. 但是, 既然 bind 方法返回的是一个函数, 那么肯定是可以通过 new 来调用的, 废话少说我们试一下~

```js
Function.prototype.customBind = function(context, ...restArgs) {
    // 这里能把 this(就是你最终要执行的函数)
    // 和 context 保存起来形成一个闭包
    // restArgs 为执行 bind 时传入的已知参数
    const self = this;
    return function(...args) {
        return self.apply(context, [...restArgs, ...args])
    }
}

// ==========  测试代码  ========================================================\
function willBind(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
    this.a = 3;
}

const obj = {
    a: 1,
    b: 2
}

const bound = willBind.customBind(obj, 'i am regret very much~');
new bound('can you forgive me?');
console.log(obj);
// this.a, this.b 3 2
// param1, param2 i am regret very much~ can you forgive me?
// {a: 3, b: 2}
// 前两行输出没有任何的问题, 但是第三行打印的日志我们不难发现, 我们居然隐式修改了 obj 对象. 可怕😂
```

然鹅, 我们试一下原生的 bind 函数的处理方式.

```js
const obj = {a: 666};
function b() {
    // 在这里直接 console.log(this) 会更加直观. 我是懒得写了
    // 有兴趣的同学可以试下
    this.a = 1234;
}

const B = b.bind(obj);
new B();
console.log(obj);  // {a: 666}
B();
console.log(obj); // {a: 1234}
```

原生的 bind 函数中, 在使用 new 调用的时候, 并没有改变 this 的指向, 所以我们执行 `new B()` 之后打印 obj 的值并没有发生改变.只有直接执行 `B()` 的时候才有了变化.

```js
Function.prototype.customBind = function(context, ...restArgs) {
    const self = this;
    return function F(...args) {
        // 如果方法是通过 new 调用的, 不改变 this 指向, 按照常规的
        if(this instanceof F) {
            return new self(...restArgs, ...args);
        }

        return self.apply(context, [...restArgs, ...args]);
    }
}

// ==========  测试代码  ========================================================\
function willBind(param1, param2) {
    console.log('this.a, this.b', this.a, this.b);
    console.log('param1, param2', param1, param2);
    this.a = 3;
}

const obj = {
    a: 1,
    b: 2
}

const bound = willBind.customBind(obj, 'i am regret very much~');
new bound('can you forgive me?');
console.log(obj);
// this.a, this.b undefined undefined
// param1, param2 i am regret very much~ can you forgive me?
// {a: 1, b: 2}
```

看上去貌似,,, 完美~

最后, 再用我们的 `new B` 方法测试下?

```js
Function.prototype.customBind = function(context, ...restArgs) {
    const self = this;
    return function F(...args) {
        // 如果方法是通过 new 调用的, 不改变 this 指向, 按照常规的
        if(this instanceof F) {
            return new self(...restArgs, ...args);
        }

        return self.apply(context, [...restArgs, ...args]);
    }
}

// ==========  测试代码  ========================================================\
const obj = {a: 666};
function b() {
    console.log(this);
    this.a = 1234;
}

const B = b.customBind(obj);

// 对于下边的结果, 没啥好说的了, 自己看看吧~
new B(); // b {a: 1234}
console.log(obj);  // {a: 666}
B(); // {a: 666}
console.log(obj); // {a: 1234}
```