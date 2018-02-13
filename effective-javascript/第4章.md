[上一章](./第3章.md)

# 对象和原型

## 第30条 理解prototype、getPrototypeOf及__proto__之间的不同
- `prototype`属性是new一个构造函数创建出来的对象的原型
- `Object.getPrototypeOf()`是ES5中检索对象原型的标准函数
- `__proto__`是检索对象原型的非标准方法
- js中的类是`一个构造函数`和`一个关联的原型`组成的一种设计模式

## 第31条 使用getPrototypeOf函数而不要使用__proto__属性
- 在ES5中使用`Object.getPrototypeOf`函数, 在支持`__proto__`的非ES5环境中实现`Object.getPrototypeOf`函数.

## 第32条 始终不要修改__proto__属性
- `__proto__`可修改对象原型链接, 会造成严重影响,避免使用.
- 避免修改`__proto__`的原因
  - 可移植性问题.并非所有对象支持.
  - 性能问题.因为它会改变继承结构本身,导致更多的现代js引擎对获取和设置对象属性的行为的深度优化
  - 最大的原因是为了保持行为的可预测性.保持继承层次结构的相对稳定是一个基本的准则.
- 可使用ES5的`Object.create`函数给新对象设置自定义的原型.
- `Object.create()`方法第一个参数传原型, 第二个传要添加的属性,返回一个新对象.
- `Object.create(null)`可以创建一个纯净的对象,不用考虑和原型链上的属性重名问题

## 第33条 使构造函数与new操作符无关
- 构造函数依赖于调用者使用new操作符,如果忘记会灾难性的创建一些全局变量,以及返回无意义的undefined
- 添加严格模式,在调用时会抛出异常,可以帮助调用者尽早发现问题
  ```js
  function User (name, password) {
    "use strict";
    this.name = name;
    this.password = password;
  }
  var u = User("bramble", "pw1234");    // error: this is undefined
  ```
- 通过检查函数的接收者是否是一个正确的构造函数实例来提供一个更健壮的方式
  ```js
  function User(name, password) {
    if(!(this instanceof User)){
      return new User(name, password);
    }
    this.name = name;
    this.password = password;
  }
  // 这种方式有缺点: 需要额外的函数调用,代价有点高.
  // 而且很难适用于可变参数,因为没有一种直接模拟apply方法
  // 将可变参数函数作为构造函数调用的方式
  ```
- 一种更为奇异的方式是利用ES5的`Object.create`函数
  ```js
  function User(name, password) {
    var self = this instanceof User ? this : Object.create(User.prototype);
    self.name = name;
    self.password = password;
    return self;
  }
  // js允许new表达式的结果可以被构造函数中的显示return语句所覆盖.
  ```
- 单参数版本的Object.create函数的简单实现
  ```js
  if(typeof Object.create === "undefined"){
    Object.create = function(prototype){
      function C(){}
      C.prototype = prototype;
      return new C();
    }
  }
  ```

## 第34条 在原型链中存储方法
- 将方法存储在实例对象中将创建该函数的多个副本,因为每个实例对象都有一份副本.会占用更多内存.
- 现代js引擎深度优化了原型查找, 所以将方法复制到实例对象并不一定保证速度有明显的提升.

## 第35条 使用闭包存储私有数据
- 对象和闭包具有相反的策略:对象的属性会被自动地暴露出去,闭包中的变量会被自动地隐藏起来.获取闭包内部结构的唯一方式是该函数显示地提供获取它的路径.
- 通过闭包可在对象中存储真正的私有数据.不是将数据作为对象的属性,而是在构造函数中以变量的方式来存储它.
  ```js
  function User(name, passwordHash){
    this.toString = function(){
      return "[User " + name + "]";
    }
    this.checkPassword = function(password){
      return hash(password) === passwordHash;
    }
  }
  // 该模式的一个缺点: 为了让构造函数的变量在使用它们的方法的作用域内,
  // 这些方法必须置于实例对象中.这回导致方法副本的扩散,
  // 然而这对那些更看重保障信息隐藏的情形来说,这点额外的代价是值得的.
  ```
- 将局部变量作为私有数据从而通过方法实现信息隐藏

## 第36条 只将实例状态存储在实例对象中
- 将可变的实例状态存储在实例对象中

## 第37条 认识到this变量的隐式绑定问题
- 数组的map方法可以传入一个可选的参数作为其回调函数的this绑定
  ```js
  CSVReader.prototype.read = function(str){
    var lines = str.trim().split(/\n/);
    return lines.map(function(line) {
      return line.split(this.regesp);
    }, this);
  };
  ```
- 使用词法作用域的变量来存储这个额外的外部this绑定的引用
  ```js
  CSVReader.prototype.read = function(str){
    var lines = str.trim().split(/\n/);
    var self = this;
    return lines.map(function(line) {
      return line.split(self.regesp);
    });
  };
  ```
- 使用函数的bind方法
  ```js
  CSVReader.prototype.read = function(str){
    var lines = str.trim().split(/\n/);
    return lines.map(function(line) {
      return line.split(this.regesp);
    }.bind(this));
  };
  ```
- this变量的作用域总是由其最近的封闭函数所确定

## 第38条 在子类的构造函数中调用父类的构造函数
- 应当仅仅在子类构造函数中调用父类构造函数，而不是当创建子类原型时调用它
- 在子类构造函数中显式地传入this作为显式的接收者调用父类的构造函数
  ```js
  function Child(){
    Parent.call(this);
  }
  ```
- 使用Object.create函数来构造子类的原型对象以避免调用父类的构造函数
  ```js
  Child.prototype = Object.create(Parent.prototype);
  ```

## 第39条 不要重用父类的属性名
- 留意父类使用的所有属性名,在子类中重用父类的属性名会覆盖.

## 第40条 避免继承标准类
- 继承标准类往往会由于一些特殊的内部属性(如`[[Class]]`)而被破坏.
- 使用属性委托优于继承标准类
  ```js
  function Dir(path, entries){
    this.path = path;
    this.entries = entries;   // array property
  }
  Dir.prototype.forEach = function(f, thisArg){
    if(typeof thisArg === "undefined"){
      thisArg = this;
    }
    this.entries.forEach(f, thisArg);
  };
  ```

## 第41条 将原型视为实现细节
- 避免检查无法控制的对象的原型结构及内部属性的实现

## 第42条 避免使用轻率的猴子补丁
- 避免使用轻率的猴子补丁(为js内置对象原型添加方法)
- 考虑通过将修改置于一个导出函数中,使猴子补丁成为可选的.
- 使用猴子补丁为缺失的标准API提供polyfills.

[下一章](./第5章.md)
