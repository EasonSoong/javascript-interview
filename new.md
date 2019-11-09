# new 操作符

## 作用

用于创建对象的实例，适用于有构造函数的对象。

## 原理

new 的基本原理分为几步：

```
// 创建一个空的原始 js 对象
const newObj = {};
// 将该对象绑定到另一个对象上
newObj.__proto__ = OtherObj.prototype;
// 将该对象作为上下文
OtherObj.call(newObj);
// 返回该对象假如没有其他返回值
return newObj;
```

具体的例子如下：

```
  function Person(name, age) {
    this.name = name;
    this.age = age;
  }

  const newObj = function(constructor, ...args) {
    const newObj = {};
    newObj.__proto__ = constructor.prototype;
    const res = constructor.apply(newObj, args);
    if (!res || !res instanceof Object) {
      return newObj;
    }
  }
```

tips:

* 箭头函数不能使用 new 操作符，因为箭头函数没有 constructor