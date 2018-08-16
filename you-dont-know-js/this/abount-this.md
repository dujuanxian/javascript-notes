# 关于this

## 为什么要用this？
this 提供了一种更优雅的方式来隐式“传递”一个对象引用，使函数可以自动引用合适的上下文对象。

## this误解
this在任何情况下都不指向函数的词法作用域，也就是说**不可能使用this来引用一个词法作用域内部的东西。**。每当你想要把this和词法作用域的查找混合使用时，
一定要提醒自己，这是无法实现的。

在JavaScript内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过JavaScript代码访问，它存在于JavaScript引擎内部。
我们不可能通过Javascript代码来操纵作用域。

```javascript
function foo() {
  var a = 2;
  this.bar();
}
function bar() {
  console.log( this.a );
}
foo(); // ReferenceError: a is not defined
```
bar函数最自然的方法是省略前面的this，直接使用词法引用标识符。

## 关于this
**this 是在运行时进行绑定的**，并不是在编译时绑定，它的上下文取决于函数调用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。
当一个函数被调用时，会创建一个执行上下文，this会被记录在这个上下文中。

## 总结
this 既不指向函数自身也不指向函数的词法作用域，它指向什么完全取决于函数在哪里被调用