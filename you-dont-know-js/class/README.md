## class
ES6解决的问题
1. 不再引用杂乱的.prototype
2. 不再需要通过Object.create(..) 来替换.prototype
3. 可以通过super(..) 来实现相对多态，这样任何方法都可以引用原型链上层的同名方法
4. class 字面语法不能声明属性（只能声明方法）
5. 可以通过extends 很自然地扩展对象（子）类型

## class陷阱
**class 基本上只是现有[[Prototype]]（委托！）机制的一种语法糖**

class 并不会像传统面向类的语言一样在声明时静态复制所有行为。如果修改或者替换了父“类”中的一个方法，那子“类”和所有实例都会受到影响，
因为它们在定义时并没有进行复制，只是使用基于[[Prototype]] 的实时委托

**class 语法无法定义类成员属性（只能定义方法），如果为了跟踪实例之间共享状态必须要这么做，那你只能使用丑陋的.prototype 语法**
```javascript
class C {
  constructor() {
    C.prototype.count++;
    console.log( "Hello: " + this.count );
  }
}
C.prototype.count = 0;
var c1 = new C(); // Hello: 1
var c2 = new C(); // Hello: 2
```

出于性能考虑，super 并不像this 一样是晚绑定（late bound，或者说动态绑定）的，它在[[HomeObject]].[[Prototype]] 上，
[[HomeObject]] 会在创建时静态绑定。
