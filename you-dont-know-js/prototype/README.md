## [[Prototype]]
JavaScript 中的对象有一个特殊的[[Prototype]] 内置属性，其实就是对于其他对象的引用

对于默认的[[Get]] 操作来说，如果无法在对象本身找到需要的属性，就会继续访问对象的[[Prototype]] 链

所有普通的[[Prototype]] 链最终都会指向内置的Object.prototype

### 属性设置和屏蔽
对于`myObject.foo = "bar";`，如果属性名foo 既出现在myObject 中也出现在myObject 的[[Prototype]] 链上层， 那么就会发生屏蔽。
myObject.foo 总是会选择原型链中最底层的foo 属性。

## “类”
JavaScript 中只有对象。

在JavaScript 中，不能创建一个类的多个实例，只能创建多个对象，它们[[Prototype]] 关联的是同一个对象。

new Foo() 会生成一个新对象，这个新对象的内部链接[[Prototype]] **关联**的是Foo.prototype 对象。
```javascript
var a = new Object();
a.[[Prototype]] = Foo.prototype;
Foo.call(a);
```
**委托**：
也被称作原型继承，但继承意味着复制，并不合适，因此称为委托更为合适。

可以看到通过“构造函数”调用new Foo() 创建的对象也有一个.constructor 属性，指向“创建这个对象的函数”。
`Foo.prototype.constructor === Foo; // true`

**在JavaScript 中对于“构造函数”最准确的解释是，所有带new 的函数调用**。函数不是构造函数，但是当且仅当使用new 时，函数调用会变成“构造函数调用”。

## 继承
要创建一个合适的关联对象，我们必须使用Object.create(..) 而不是使用具有副作用的Foo(..)。这样做唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉，
不能直接修改已有的默认对象。

ES6 添加了辅助函数Object.setPrototypeOf(..)，可以用标准并且可靠的方法来修改关联。`Object.setPrototypeOf( Bar.prototype, Foo.prototype );`

#### 反射
检查一个实例的委托关联通常被称为内省（或者**反射**）。

`a instanceof Foo;`

instanceof 回答的问题是：在a 的整条[[Prototype]] 链中是否有指向Foo.prototype 的对象？

`Foo.prototype.isPrototypeOf( a );`

isPrototypeOf(..) 回答的问题是：在a 的整条[[Prototype]] 链中是否出现过Foo.prototype ？

`b.isPrototypeOf( c );`

`Object.getPrototypeOf( a ) === Foo.prototype;`

`a.__proto__ === Foo.prototype;`

访问 a.__proto__ 时，实际上是调用了a.__proto__()（调用getter 函数）。虽然getter 函数存在于Object.prototype 对象中，但是它的this 指向对象a，
所以和Object.getPrototypeOf( a ) 结果相同。

## 对象关联
[[Prototype]] 机制就是存在于对象中的一个内部链接，这个链接的作用是：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在[[Prototype]] 
关联的对象上进行查找。这一系列对象的链接被称为“**原型链**”。

Object.create(null) 会创建一个拥有空（ 或者说null）[[Prototype]]链接的对象，这个对象无法进行委托。由于这个对象没有原型链，
所以instanceof 操作符（之前解释过）无法进行判断，因此总是会返回false。这些特殊的空[[Prototype]] 对象通常被称作“字典”，
它们完全不会受到原型链的干扰，因此非常适合用来存储数据。

## 小结
如果要访问对象中并不存在的一个属性，[[Get]] 操作就会查找对象内部[[Prototype]] 关联的对象。这个关联关系实际上定义了一条“原型链”，
在查找属性时会对它进行遍历。

关联两个对象最常用的方法是使用new 关键词进行函数调用，通常被称为“构造函数调用”。

对象之间的关系不是复制而是委托。