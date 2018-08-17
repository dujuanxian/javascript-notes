## 面向委托的设计

### 委托理论
```javascript
Task = {
  setID: function(ID) { this.id = ID; },
  getID: function() { console.log( this.id ); }
};
XYZ = Object.create( Task );
XYZ.prepareTask = function(ID) {
  this.setID( ID );
};
XYZ.getTaskDetails = function() {
  this.getID();
};
```

Task 和XYZ它们是对象。XYZ的[[Prototype]] 委托了Task 对象。

相比于面向类（或者说面向对象），我会把这种编码风格称为“**对象关联**”（OLOO，objects linked to other objects）

对象关联风格的不同之处：
1. 通常来说，在[[Prototype]] 委托中最好把状态保存在委托者（XYZ）而不是委托目标（Task）上。
2. 在类设计模式中，利用重写（多态）的优势，在子类中重写父类方法。在委托行为中则恰好相反：会尽量避免在[[Prototype]] 链的不同级别中使用相同的命名。
3. XYZ 中的this.setID(ID)；首先会寻找XYZ 自身是否有setID(..)，但是XYZ 中并没有这个方法名，因此会通过[[Prototype]] 委托关联到Task。
此外，由于调用位置触发了this 的隐式绑定规则，运行时this 仍然会绑定到XYZ  
