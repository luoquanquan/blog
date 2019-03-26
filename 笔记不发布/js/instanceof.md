# instanceof 实现

instanceof 通过判断对象的 \_\_proto\_\_ 实现对象类型的判断, 直到访问的对象原型为 null 为止

```js
class Person { }

class Man extends Person { }

const quanquan = new Man();

quanquan instanceof Object;  // true
quanquan instanceof Person;  // true
quanquan instanceof Man;     // true
```

## 手写一个 instanceof 函数

```js
/**
 * 原生 js 实现 instanceof
 * @param {Object} 要判断类型的对象
 * @param {Function} 判断类型的构造函数
 * @return {Boolean} 该对象是否属于指定的类型
 */
function customInstanceof(index, target) {
    // 实例的 __proto__ 会指向构造函数的原型
    index = index.__proto__

    while(index) {
        if(index === target.prototype) {
            return true;
        }
        index = index.__proto__
    }

    // 直到遍历到 __proto__ 指向 null 的时候, 仍然没有找到和 target.prototype 相等时,
    // 证明 index 不属于 target 类型
    return false;
}
```

## 验证代码

```js
function customInstanceof(index, target) {
    index = index.__proto__
    while(index) {
        if(index === target.prototype) {
            return true;
        }
        index = index.__proto__
    }
    return false;
}

class Person { }

class Man extends Person { }

const quanquan = new Man();

customInstanceof(quanquan, Man)    // true
customInstanceof(quanquan, Person) // true
customInstanceof(quanquan, Object) // true
customInstanceof(quanquan, Array)  // false
```
