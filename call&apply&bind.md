# call & apply & bind 原生实现

apply、call 和 bind 可以改变 this 的指向

## call 原生实现

call 的使用示例：

```
const testObj = {
  name: 'testObj',
  sayHi: function() {
    console.log(this.name);
  },
}; 

const testObj2 = {
  name: 'testObj2',
};

testObj.sayHi(); // testObj
testObj.sayHi.call(testObj2); // testObj2
```

常见普通版本

```
Function.prototype._call = function(ctx, ...parameters) {
  if (typeof ctx === 'object') {
    ctx = ctx || window;
  } else {
    ctx = Object.create(null);
  }
  ctx.fn = this;
  const res = ctx.fn(...parameters);
  delete ctx.fn;
  return res;
};
```

很多人可能觉得这样就已经完成了，但我有一个问题想问问你，如果调用 call 传入的参数包含了 fn 会怎么样？

```
const testObj = {
  name: 'testObj',
  sayHi: function() {
    console.log(this.name);
  },
}; 

const testObj2 = {
  name: 'testObj2',
  fn: function() {
    console.log('i am fn of testObj2')
  },
};

testObj.sayHi.call(testObj2); // testObj2
```

执行完了之后，发现 testObj2 的 fn 属性消失了，发现问题了吗？这是因为我们使用了 fn 作为默认的 key 对 testObj2 进行赋值，在执行完后进行了删除导致的，为了避免发生这种情况，我们应该避免使用 testObj2 中存在的属性作为 key 去操作，我们可以使用 ES6 中的 Symbol 来避免这种情况。完整代码如下：

```
Function.prototype._call = function(ctx, ...parameters) {
  if (typeof ctx === 'object') {
    ctx = ctx || window;
  } else {
    ctx = Object.create(null);
  }
  const fn = Symbol();
  ctx[fn] = this;
  const res = ctx[fn](...parameters);
  delete ctx[fn];
  return res;
};
```

到这里我们已经完成的差不多了，但是这个时候，如果面试官问你：你这个都是用 ES6 实现的，能用 ES5 实现吗？当然能！代码示例如下：

```
Function.prototype._call = function(ctx) {
  if (typeof ctx === 'object') {
    ctx = ctx || window;
  } else {
    ctx = Object.create(null);
  }
  var keys = Object.keys(ctx);

  // 构造参数
  const args = [];
  for (var i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']');
  }
  // 构造唯一 key 值
  var maxKeyLen = 0;
  for (var i = 0; i < keys.length; i++) {
    maxKeyLen = Math.max(maxKeyLen, keys[i].length);
  }
  var uniqueKey = 'x'.repeat(maxKeyLen + 1);
  ctx[uniqueKey] = this;
  var res = eval('ctx[uniqueKey](' + args + ')');
  delete ctx[uniqueKey];
  return res;
};
```

## apply 原生实现

apply 和 call 的原生实现类似，代码示例如下

```
// ES5
Function.prototype._apply = function(ctx, parameters) {
  if (typeof ctx === 'object') {
    ctx = ctx || window;
  } else {
    ctx = Object.create(null);
  }
  
  var args = [];
  for (var i = 1; i < parameters.length; i++) {
    args.push('parameters[' + i + ']');
  }

  var maxKeyLen = 0;
  var keys = Object.keys(ctx);
  for (var i = 0; i < keys.length; i++) {
    maxKeyLen = Math.max(maxKeyLen, keys[i].length);
  }
  var uniqueKey = 'x'.repeat(maxKeyLen);
  ctx[uniqueKey] = this;
  var res = eval('ctx[uniqueKey](' + args + ')');
  delete ctx[uniqueKey];
  return res;
};

// ES6
Function.prototype._apply = function(ctx, parameters) {
  if (typeof ctx === 'object') {
    ctx = ctx || window;
  } else {
    ctx = Object.create(null);
  }

  const key = Symbol();
  ctx[key] = this;
  const res = ctx[key](...parameters);
  delete ctx[key];
  return res;
};
```

## bind 的原生实现

先看一下 bind 的用法：bind 用于改变 this 的指向，并且返回绑定 this 的方法。

```
const obj1 = {
  name: 'obj1',
  fn: function() {
    console.log(this.name);
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn.bind(obj2);
bindFn(); // obj2
```

根据上面的示例，原生实现如下：

```
Function.prototype._bind = function(ctx) {
  // 保存实际调用的方法
  const self = this;
  const bindFn = function() {
    return self.apply(ctx);
  }
  return bindFn;
}

const obj1 = {
  name: 'obj1',
  fn: function() {
    console.log(this.name);
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn._bind(obj2);
bindFn(); // obj2
```

到这里我们实现了 bind 的第一个特性，bind 还支持传入其他参数，与 call 相同，可以传入多个参数。（apply 是传入一个数组）

```
const obj1 = {
  name: 'obj1',
  fn: function(age, sex) {
    console.log(this.name, age, sex);
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn.bind(obj2, 10);
bindFn('man'); // obj2 10 man
```

支持传参版本的 bind 实现如下：

```
Function.prototype._bind = function(ctx, ...args1) {
  const self = this;
  const bindFn = function(...args2) {
    return self.apply(ctx, [...args1, ...args2]);
  }
  return bindFn;
}

const obj1 = {
  name: 'obj1',
  fn: function(age, sex) {
    console.log(this.name, age, sex);
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn._bind(obj2, 10);
bindFn('man'); // obj2 10 man
```

完成了对传参的支持，bind 还有一个特性，bind 后的方法可以作为构造函数，使用 new 操作符调用，此时 bind 绑定的 this 会失效。

```
const obj1 = {
  name: 'obj1',
  fn: function(name, age) {
    this.name = name;
    this.age = age;
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn.bind(obj2);
const res = new bindFn('test', 16);
console.log(obj2.name, res); // obj2 fn {name: "test", age: 16}
```

下面实现一下这个特性：

```
Function.prototype._bind = function(ctx, ...args1) {
  const self = this;
  const bindFn = function(...args2) {
    return self.apply(
      this instanceof bindFn ? this : ctx,
      [...args1, ...args2]
    );
  }
  return bindFn;
}

const obj1 = {
  name: 'obj1',
  fn: function(name, age) {
    this.name = name;
    this.age = age;
  }
};

const obj2 = {
  name: 'obj2',
};

const bindFn = obj1.fn._bind(obj2);
const res = new bindFn('test', 16);
console.log(obj2.name, res); // obj2 bindFn {name: "test", age: 16}
```

到这里已经差不多完成了，但其实还有一个问题

```
const obj1 = {
  name: 'obj1',
  fn: function(age, sex) {
    console.log(this.name, age, sex);
  }
};
obj1.fn.prototype.fName = 'obj1_fn';

const obj2 = {
  name: 'obj2',
};

const bindFn1 = obj1.fn.bind(obj2, 10, 'boy');
const res1 = new bindFn1();
const bindFn2 = obj1.fn._bind(obj2, 10, 'boy');
const res2 = new bindFn2();
console.log(res1.fName, res2.fName); // obj1_fn undefined
```

在使用构造函数返回的对象上，找不到 fName 这个属性了。为了解决这个问题，需要进行原型链的绑定。

```
Function.prototype._bind = function(ctx, ...args1) {
  const self = this;
  const fNop = function() {};
  const bindFn = function(...args2) {
    return self.apply(
      this instanceof bindFn ? this : ctx,
      [...args1, ...args2]
    );
  }
  if (this.prototype) {
    fNop.prototype = this.prototype;
  }
  bindFn.prototype = new fNop();
  return bindFn;
}

const obj1 = {
  name: 'obj1',
  fn: function(age, sex) {
    console.log(this.name, age, sex);
  }
};
obj1.fn.prototype.fName = 'obj1_fn';

const obj2 = {
  name: 'obj2',
};

const bindFn1 = obj1.fn.bind(obj2, 10, 'boy');
const res1 = new bindFn1();
const bindFn2 = obj1.fn._bind(obj2, 10, 'boy');
const res2 = new bindFn2();
console.log(res1.fName, res2.fName); // obj1_fn obj1_fn
```

到这里就基本完成了，但其实 bind 的标准实现还有许多小的细节这里没有考虑到的，下面有一个考虑比较全面的实现版本。

```
Function.prototype._bind = function(ctx, ...args1) {
  // 判断进行绑定的是不是个可执行的方法
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }
  const self = this;
  const bindFn = function(...args2) {
    //把原型链指向要bind操作的函数，也就是原函数。
    //直接执行时this变得未知，所以加上try
    try {
      this.__proto__ = self.prototype;
    } catch(e) {}
    const isNew = self.prototype ? this instanceof self : false;
    return self.apply(
      isNew ? this : ctx,
      [...args1, ...args2]
    );
  }
  //bind后的方法是没有原型的，使其与浏览器原生表现一致
  bindFn.prototype = undefined;
  return bindFn;
}
```