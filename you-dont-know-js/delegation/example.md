### 传统类设计模式

Controller父类：
```javascript
function Controller() {
  this.errors = [];
}
Controller.prototype.showDialog(title, msg){ // 给用户显示标题和消息
};
Controller.prototype.success = function(msg) {
  this.showDialog( "Success", msg );
};
Controller.prototype.failure = function(err) {
  this.errors.push( err );
  this.showDialog( "Error", err );
};
```

LoginController子类：
```javascript
function LoginController() {
  Controller.call( this );
}
// 把子类关联到父类
LoginController.prototype = Object.create( Controller.prototype );
LoginController.prototype.getUser = function() {
  return document.getElementById( "login_username" ).value;
};
LoginController.prototype.getPassword = function() {
  return document.getElementById( "login_password" ).value;
};
LoginController.prototype.validateEntry = function(user,pw) {
  user = user || this.getUser();
  pw = pw || this.getPassword();
  if (!(user && pw)) {
    return this.failure("Please enter a username & password!");
  }
  return true;
};
LoginController.prototype.failure = function(err) {
  Controller.prototype.failure.call(this, "Login invalid: " + err);
};
```
AuthController子类：
```javascript
function AuthController(login) {
  Controller.call( this );
  this.login = login;
}
AuthController.prototype = Object.create( Controller.prototype );
AuthController.prototype.server = function(url,data) {
  return $.ajax( { url: url, data: data } );
};
AuthController.prototype.checkAuth = function() {
  var user = this.login.getUser();
  var pw = this.login.getPassword();
  if (this.login.validateEntry( user, pw )) {
    this.server( "/check-auth",{ user: user, pw: pw} )
      .then( this.success.bind( this ) )
      .fail( this.failure.bind( this ) );
  }
};
AuthController.prototype.success = function() {
  Controller.prototype.success.call( this, "Authenticated!" );
};
AuthController.prototype.failure = function(err) {
  Controller.prototype.failure.call(this, "Auth Failed: " + err);
};
var auth = new AuthController(new LoginController());
auth.checkAuth();
```

所有控制器共享的基础行为是success(..)、failure(..) 和showDialog(..)。子类重写了failure(..) 和success(..)。

AuthController 需要使用LoginController，因此我们实例化后者（new LoginController()）并用一个类成员属性this.login 来引用它，
在继承的基础上进行了一些合成。

### 对象关联风格的行为委托

LoginController：
```javascript
var LoginController = {
  errors: [],
  getUser: function() {
    return document.getElementById("login_username").value;
  },
  getPassword: function() {
    return document.getElementById("login_password").value;
  },
  validateEntry: function(user,pw) {
    user = user || this.getUser();
    pw = pw || this.getPassword();
    if (!(user && pw)) {
      return this.failure("Please enter a username & password!");
    }
    return true;
  },
  showDialog: function(title,msg) {},
  failure: function(err) {
    this.errors.push( err );
    this.showDialog( "Error", "Login invalid: " + err );
  }
};
```

AuthController 委托LoginController:
```javascript
var AuthController = Object.create( LoginController );
AuthController.errors = [];
AuthController.checkAuth = function() {
  var user = this.getUser();
  var pw = this.getPassword();
  if (this.validateEntry( user, pw )) {
    this.server( "/check-auth",{user: user,pw: pw} )
      .then( this.accepted.bind( this ) )
      .fail( this.rejected.bind( this ) );
  }
};
AuthController.server = function(url,data) {
  return $.ajax( {url: url,data: data} );
};
AuthController.accepted = function() {
  this.showDialog( "Success", "Authenticated!" )
};
AuthController.rejected = function(err) {
  this.failure( "Auth Failed: " + err );
};
AuthController.checkAuth();
```

在行为委托模式中，AuthController 和LoginController 只是对象，它们之间是兄弟关系，并不是父类和子类的关系。代码中AuthController 委托了LoginController，
反向委托也完全没问题。