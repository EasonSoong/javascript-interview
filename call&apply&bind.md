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