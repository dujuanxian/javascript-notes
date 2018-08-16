# this的全面解析

## 绑定规则
### 默认绑定
foo函数调用时应用了this 的默认绑定，因此this 指向全局对象。
```javascript
function foo() {
  console.log( this.a );
}
var a = 2;
foo(); // 2
```
**如果使用严格模式，全局对象将无法使用默认绑定，因此this 会绑定到undefined**：
```javascript
function foo() {
  "use strict";
  console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined
```

### 隐式绑定
**当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this 绑定到这个上下文对象**
```javascript
function foo() {
  console.log( this.a );
}
var obj = {
  a: 2,
  foo: foo
};
obj.foo(); // 2
```

**隐式丢失**
```javascript
function foo() {
  console.log( this.a );
}
var obj = {
  a: 2,
  foo: foo
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```
虽然bar 是obj.foo 的一个引用，但是实际上，它引用的是foo 函数本身，因此此时的bar() 其实是应用了默认绑定。

```javascript
function foo() {
  console.log( this.a );
}
function doFoo(fn) {
  // fn 其实引用的是foo
  fn(); // <-- 调用位置！
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```
**参数传递其实就是一种隐式赋值**

**回调函数也会丢失this 绑定**

### 显式绑定
```javascript
function foo() {
  console.log( this.a );
}
var obj = {
  a:2
};
foo.call( obj ); // 2
```

显式绑定仍然无法解决丢失绑定问题

#### 硬绑定
ES5 中提供了内置的方法`Function.prototype.bind`
```javascript
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}
var obj = {
  a:2
};
var bar = foo.bind( obj );
console.log( bar( 3 ) ); // 5
```

### new绑定
在JavaScript 中，构造函数只是一些使用new 操作符时被调用的普通函数。它们并不会属于某个类，也不会实例化一个类。

包括内置对象函数在内的所有函数都可以用new 来调用，这种函数调用被称为构造函数调用。实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

使用new 来调用函数，或者说发生构造函数调用时：
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行[[Prototype]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new 表达式中的函数调用会自动返回这个新对象。

比如`var o1 = new Base()`，Javascript实际执行的是：
```javascript
var o1 = new Object();
o1.[[Prototype]] = Base.prototype;
Base.call(o1);
```
#### new和Object.create的区别
Object.create的实现：
```javascript
Object.create =  function (o) {
    var F = function () {};
    F.prototype = o;
    return new F();
};
```
|比较|new|Object.create|
| --- | ------- | ------- |
|构造函数|保留原构造函数属性|丢失原构造函数属性|
|原型链|原构造函数prototype属性|原构造函数/（对象）本身|
|作用对象|function|function和object||

**在JavaScript 中创建一个空对象最简单的方法都是Object.create(null)**。Object.create(null) 和{} 很像， 但是并不会创建Object.prototype 这个委托，
所以它比{}“更空”。

## 优先级
new绑定 > 显式绑定 > 隐式绑定 > 默认绑定

## this词法（ES6箭头函数）
**箭头函数不使用this 的四种标准规则，而是根据外层（函数或者全局）作用域来决定this。**
```javascript
function foo() {
  return (a) => {
    console.log( this.a );
  };
}
var obj1 = {
  a:2
};
var obj2 = {
  a:3
};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是3 ！
```

