## 类的机制
### 构造函数
类实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为构造函数。

JavaScript 中父类和子类的关系只存在于两者构造函数对应的.prototype 对象中，因此它们的构造函数之间并不存在直接联系，从而无法简单地实现两者的
相对引用（在ES6 的类中可以通过super 来“解决”这个问题）。

### 多重继承
有些面向类的语言允许你继承多个“父类”。JavaScript 要简单得多：它本身并不提供“多重继承”功能。

## 混入
在继承或者实例化时，JavaScript 的对象机制并不会自动执行复制行为。简单来说，JavaScript 中只有对象，并不存在可以被实例化的“类”。
**一个对象并不会被复制到其他对象，它们会被关联起来**。

### 显示混入
#### 混合复制
JavaScript有一个方法来模拟类的复制行为，这个方法就是混入。

```javascript
function mixin( sourceObj, targetObj ) {
  for (var key in sourceObj) {
    // 只会在不存在的情况下复制
    if (!(key in targetObj)) {
      targetObj[key] = sourceObj[key];
    }
  }
  return targetObj;
}
```

由于两个对象引用的是同一个函数，因此这种复制（或者说混入）实际上并不能完全模拟面向类的语言中的复制。

**JavaScript 中的函数无法（用标准、可靠的方法）真正地复制，所以你只能复制对共享函数对象的引用**。

如果你向目标对象中显式混入超过一个对象，就可以部分模仿多重继承行为，但是仍没有直接的方式来处理函数和属性的同名问题。

#### 寄生继承
```javascript
function Vehicle() {
  this.engines = 1;
}
Vehicle.prototype.drive = function() { 
  // ...
}
function Car() {
  var car = new Vehicle();
  car.wheels = 4;
  var vehDrive = car.drive;
  car.drive = function() {
    vehDrive.call( this );
    // ...
  }
  return car;
}
var myCar = new Car();
```

### 隐式混入
```javascript
var Something = {
  cool: function() {
  }
};
var Another = {
  cool: function() {
    Something.cool.call( this );
  }
};
```

## 小结
类意味着复制。传统的类被实例化时，它的行为会被复制到实例中。类被继承时，行为也会被复制到子类中。多态看起来似乎是从子类引用父类，
但是本质上引用的其实是复制的结果。

JavaScript 并不会（像类那样）自动创建对象的副本。显式混入实际上无法完全模拟类的复制行为，因为对象只能复制引用，无法复制被引用的对象或者函数本身。
