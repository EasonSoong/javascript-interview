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