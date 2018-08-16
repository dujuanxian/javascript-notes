# 作用域

## 作用域介绍
_作用域是什么？_ 查找变量存储的地方。变量只能出现在两个地方，一个是给它赋值的地方，一个是检索它原值的地方。

## 词法作用域
**决定变量的词法作用域是在编译过程发生的。** 变量的声明都是通过编译器在编译阶段处理。在执行阶段，JS引擎会对编译阶段生成的变量声明在
词法作用域里进行查找并进行LHS赋值或者RHS检索。

```javascript
1. var foo = "bar"; 
2. function bar () {
3.   var foo = "baz";
4. } 
5. function baz (foo) {
6.  foo = "bam";
7.  bam = "yay";
8. }
9. bar();
10.baz();
```

### 编译函数作用域
在编译阶段，编译器会做如下操作：
- Line1，在全局作用域声明foo，并赋值为undefined；
- Line2，在全局作用域声明function bar；由于bar是一个函数声明，因此会创建一个bar函数作用域；
- Line3，在bar函数作用域声明foo，并赋值为undefined；
- Line4，跳出bar函数作用域，回到全局作用域；
- Line5，在全局作用域声明function baz；由于baz是一个函数声明，因此会创建一个baz函数作用域；
- Line6，在baz函数作用域声明参数foo，，并赋值为undefined；
- Line7，bam不是一个变量声明，在baz函数作用域不做任何处理；
- Line8，跳出baz函数作用域，回到全局作用域；

相同的标识符名称可以在嵌套作用域的多个层中被指定，这称为**遮蔽（shadowing）**。

### 执行函数
在执行阶段，JS引擎会做如下操作：

在执行阶段，已经不需要去考虑var声明和函数声明，这些已经在编译阶段处理了，剩下的只是可执行代码。
    
- Line1，foo是LHS引用，在全局作用域获取foo的声明，并把"bar"赋值给foo
- Line9，执行bar函数，在bar函数作用域获取foo的声明，在Line3并把"baz"赋值给foo
- Line10，执行baz函数，在Line6，从baz函数作用域获取foo的声明，并把"bam"赋值给foo
- 在Line7，在baz函数作用域获取bam的声明，在baz作用域并没有bam的声明。因此上升到全局作用域去获取bam的声明，在全局作用域也并没有bam的声明，
**全局作用域**会隐式创建一个bam变量，并在Line7赋值"yay"给bam。（在严格模式下，会抛出Reference Error）
 
### 严格模式

对于LHS和RHS引用，如果在作用域找不到声明：

|        |严格模式|非严格模式|
|--------|--------|--------|
|LHS|抛出ReferenceError|创建隐式全局变量|
|RHS|抛出ReferenceError|抛出ReferenceError|
 
隐式创建全局变量在严格模式下被禁止，是因为在运行阶段创建变量去修改已经创建的词法作用域，比编译阶段慢得多。大多数情况下严格模式的约束，
是因为使代码更难去优化。

每个Javascript文件是一个独立的程序，所有的文件共享全局作用域。未使用严格模式的文件并不会影响其他文件。但如果将非严格模式的文件和其他文件一起压缩，
便会引起问题。

### 词法作用域 VS 动态作用域
**理论上**，动态作用域在查找变量时，会依据函数被调用的位置（即调用堆栈）决定变量的值。

```javascript
// theoretical dynamic scoping
function foo() {
  console.log(bar); //baz
}
function baz() {
  var bar = "baz";
  foo();
}
baz();
```

### IIFE 理解调用函数表达式
```javascript
function bob() { //...
}
bob();
```
```javascript
function bob() {//...
}
(bob)();
```
```javascript
(function bob() { // 此时bob从函数声明转为函数表达式，因为该行不再以function开始
})();
```
```javascript
(function() {//...
})();
```
**IIFE可以用于创建块作用域**，但是IIFE的return语句并不能返回值到外层作用域，而是返回到了IIFE内部作用域。this在IIFE中也和块作用域期待的不一致。

### 块作用域
```javascript
function diff(x, y) {
  if (x > y) {
    let temp = x;
    x = y;
    y = temp;
  } 
  return y - x;
}
```
temp由于采用let声明，因此具有了块级作用域的属性。如果采用var声明，temp会提升到diff函数作用域，在if块作用域外仍然可以访问。

如果期望的是函数作用域，那更适合采用var声明，比如下面的例子：
```javascript
function lookupRecord(searchStr) {
  try {
    var id = getRecorad(searchStr);
  } 
  catch {
    var id = -1;
  }
  return id;
}
```

### const
const并不是指变量的值不会改变，而是指**const声明的变量不能够再次赋值**