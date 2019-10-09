# JavaScript面对对象

面对对象（Object-oriented，OO）的一个标志，就是他们都有类的概念，而通过类可以创建任意多个具有相同属性的方法的对象。ECMA-262把对象定义为："无序属性的集合，其属性可以包含基本知识、对象或者函数。"严格的讲，这就相当于说**对象十一组没有特定顺序的值。对象的每个属性或方法都有一个名字，每个名字都映射到一个值。**

## 1.1 理解对象

创建一个Object的实例，然后再为他添加属性和方法，如下所示。

```javascript
var person = new Object();
person.name = "Nicholas";
person.age = 25;
person.job = "Software Engineer";
person.sayName = function() {
    alert(this.name);
}
```

上面的例子创建了一个名为`person`的对象，并为它添加了三个属性（`name`、`age`和`job`）和一个方法（`sayName()`）。其中，`sayName()`用于显示`this.name`(将被解析成`persong.name`）的值。

使用对象字面量语法可以写成这样：

```javascript
var person = {
    name: "Nicholas",
    age: 25,
    job: "Software Engineer";
    sayName: function() {
        alert(this.name);
    }
}
```

### 1.1.1 属性类型

只有在内部采用的特性（`attribute`），描述属性(`property`)的各种特征。

> 1. 数据属性
>
> 数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有4个描述其行为的特性。

* **[[ Configurable ]]** : 表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。默认为 **true**
* **[[ Enumerable ]]** : 表示能否通过 `for-in` 循环返回属性。默认为 **true**
* **[[ Writable ]]** : 表示能否修改属性的值。默认为 **true**
* **[[ Value ]]** : 包含这个特属性的数据值。读取数据值时，从这个位置读；写入属性值的时候，把新值保存在这个位置。这个特性的默认值为 **undefined**

例如：

```javascript
var person = {
    name: "Nicholas"
}
```

这里创建了一个名为name的属性，为它指定的值是"Nicholas"。也就是说，`[[Value]]`特性将被设置为"Nicholas"，而这个值得任何修改都将反映在这个位置。

要修改属性的默认特性，必须使用 `ES5` 的`Object.defineProperty()` 方法。这个方法接收三个参数：属性所在的对象，属性的名字和一个描述符对象。

```javascript
var person = {};
Object.defineProperty(person, "name", {
    writable: false,
    value: "Nicholas"
});
alert(person.name); // "Nicholas"
person.name = "Greg";
alert(person.name); // "Nicholas"
```

这个例子创建了一个名为 name 的属性，他的值 "Nicholas"是只读的。这个属性的值是不可修改的，如果尝试为它制定新值，则在飞严格模式下，赋值的操作将被忽略；在严格模式下，赋值操作将会导致抛出错误。

类似的规则也适用于不可配置的属性。例如：

```javascript
var person = {};
Object.defineProperty(person, "name", {
    configurable: false,
    value: "Nicholas"
});
alert(person.name); // "Nicholas"
delete person.name;
alert(person.name); // "Nicholas"
```

把configurable 设置为false，表示不能从对象中删除属性。如果把这个属性调用delete，则不会生效（严格模式下报错）。而且，一旦把属性定义为不可配置的。就不能再把它变回成可配置了。此时，再调用 `Object.defineProperty()` 方法修改除 `writable` 之外的特性，都会导致错误：

```javascript
var person = {};
Object.defineProperty(person, "name", {
    configurable: false,
    value: "Nicholas"
});

// 抛出错误
Object.defineProperty(person, "name", {
    configurable: true,
    value: "Nicholas"
})
```

**在调用`Object.defineProperty()` 方法时，如果不制定，configurable、enumerable 和 writable特性的默认值都是false。**

> 2. 访问器属性
>
> 访问器属性不包含数据值；它们包含一对儿`getter`和`setter`函数（这俩都不是必须的）。在读取访问器属性时，会调用getter函数，这个函数负责返回有效的值；在写入访问器属性时，会调用setter函数并传入新值，这个函数负责决定如何处理数据。访问器属性有如下4个特性。

* **[[Configurable]]** : 表示能否通过 `delete` 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。 默认为 true
* **[[Enumerable]]** : 表示能否通过 `for-in` 循环返回属性。对于直接在对象上定义的属性，这个特性的默认值为 true
* **[[Get]]** : 在读取属性时调用的函数。默认值为 undefined。
* **[[Set]]** : 在写入属性时调用的函数。默认值为 undefined。

访问器属性不能直接定义，必须使用 `Object.defineProperty()` 来定义。

```javascript
var book = {
    _year: 2004,
    edition: 1
};
Object.defineProperty(book, "year", {
    get: function() {
        return this._year;
    },
    set: function(newValue) {
        if(newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});
book.year = 2005;
alert(book.edition); // 2
```

不一定非要同时指定 getter 和 setter。只指定 getter 意味着属性时不能写，尝试写入属性会被忽略。

### 1.1.2 定义多个属性

```javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
    get: function () {
      return this._year;
    },
    set: function (newVal) {
      if (newVal > 2004) {
        this._year = newVal;
        this.edition += newVal - 2004;
      }
    }
  }
})
```

注意，这里使用了 `Object.defineProperties` 定义了两个数据属性和一个访问器属性。没有设置configurable 、writable、Enumerable;

### 1.1.3 读取属性的特性

```javascript
var book = {};
Object.defineProperties(book, {
  _year: {
    value: 2004
  },
  edition: {
    value: 1
  },
  year: {
    get: function () {
      return this._year;
    },
    set: function (newVal) {
      if (newVal > 2004) {
        this._year = newVal;
        this.edition += newVal - 2004;
      }
    }
  }
})
var descriptor = Object.getOwnPropertyDescriptor(book, "year");
console.log(descriptor);
console.log(descriptor.enumerable); // false
```

## 1.2 创建对象

### 1.2.1 工厂模式

工厂模式抽象了创建具体对象的过程。

```javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    }
    return o;
}
```

### 1.2.2 构造函数模式

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function () {
    console.log(this.name);
  }
}
var p1 = new Person("Nicholas", 19, "Software Engineer");
var p2 = new Person("Greg", 20, "Tester");
console.log(p1.constructor === p2.constructor);
```

`Person()` 函数取代了 `createPerson()` 函数。两者不同之处：

* 没有显式的创建对象；
* 直接将属性和方法赋值给 `this` 对象；
* 没有 `return` 语句

要创建Person的实例，必须使用new操作符。以这种方式调用构造函数实际上会经历以下4个步骤：

(1). 创建一个新对象；

(2). 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；

(3). 执行构造函数中的代码（为这个新对象添加属性）；

(4). 返回新对象。

p1 和 p2分别保存着 Person的一个不同的实例。这两个对象都有一个constructor属性，该属性指向 Person。使用构造函数的方式意味着可以将他的实例标示为一种特定的类型。

```javascript
console.log(p1 instanceof Person); // true
console.log(p1 instanceof Object); // true
```

**构造函数的问题**

在前面的例子中，p1 和 p2 都有一个名为 `sayName()` 的方法，但那两个方法不是同一个Function实例。然而，创建两个完成同样任务的 Function 实例的确没有必要；况且有 this 对象在，根本不用再执行代码前把函数绑定到特定对象上。

```javascript
var sayName = function () {
  console.log(this.name);
}
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}
var p1 = new Person("Nicholas", 19, "Software Engineer");
p1.sayName();
```

### 1.2.3 原型模式

我们创建的每个函数都有一个`prototype` 属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。换句话说，不必再构造函数中定义对象实例的信息，而是可以将这些信息直接存储到原型对象中，如下面的例子所示。

```javascript
function Person() { }
Person.prototype.name = "Nicholas";
Person.prototype.age = 19;
Person.prototype.sayName = function () {
  console.log(this.name);
}
```

1. 理解原型对象

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor属性。这个属性包含一个指向prototype属性所在函数的指针。`Person.prototype.constructor`指向`Person`。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

![prototype](./prototype.png)

虽然在所有实例中都无法访问到`[[Prototype]]`，但可以通过 `isPrototypeOf()` 方法来确定对象之间是否存在这种关系。从本质上讲，如果`[[Prototype ]]` 指向调用 `isPrototypeOf()` 方法来确定对象之间是否存在这种关系。

```javascript
Object.getPrototypeOf(P1) === Person.prototype
```

如果我们在实例中添加了一个属性，而该属性与实例原型中的一个属性同名，那我们就在实例中创建该属性，该属性将会屏蔽原型中的那个属性。

```javascript
function Person() { }
Person.prototype.name = "Nicholas";

var p1 = new Person();
var p2 = new Person();
p1.name = "Greg";
console.log(p1.name); // "Greg"
console.log(p2.name); // "Nicholas"
```

**原型与in操作符**

有两种方式使用 in 操作符：单独使用和在 `for-in` 循环中使用。在单独使用时，in 操作符会在通过对象能够访问给定属性时返回 true ，无论该属性是在实例中还是原型中。

```javascript
var o = {
  toString: function () {
    return "My Object";
  }
}
for (var prop in o) {
  if (prop == "toString") {
    alert("Found toString");
  }
}
```

要取得对象上所有可枚举的实例属性，可以使用ES5的 `Object.keys()` 方法。

```javascript
function Person() { }
Person.prototype.name = "Nicholas";
Person.prototype.age = 23;
Person.prototype.sayName = function () {
  alert(this.name);
}
var keys = Object.keys(Person.prototype);
console.log(keys);

var p1 = new Person();
p1.name = "Greg";
p1.age = 24;
var p1Keys = Object.keys(p1);
console.log(p1Keys);
```

如果重写整个原型对象，那么情况不一样了。

```javascript
function Person() { }
var friend = new Person();
Person.prototyp = {
  constructor: Person,
  name: "Nicholas",
  age: 12,
  job: "Software Engineer",
  sayName: function () {
    alert(this.name);
  }
}
friend.sayName(); // error
```

### 1.2.4 组合使用构造函数模式和原型模式

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ["Tom", "Jerry"];
}
Person.prototype = {
  constructor: Person,
  sayName: function () {
    console.log(this.name)
  }
}
var p1 = new Person("Nicholas", 23, "Software");
var p2 = new Person("Greg", 28, "Doctor");

console.log(p1.friends === p2.friends); // false
console.log(p1.sayName === p2.sayName); // true
```

### 1.2.5 动态原型模式

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    if(typeof this.sayName != "function"){
        Person.prototype.sayName = function() {
            console.log(this.name)
        }
    }
}
```

### 1.2.6 寄生构造函数模式

```javascript
function Person(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
    console.log(this.name);
  }
  return o;
}
var friend = new Person("Nicholas", 21, "Software Engineer")
```

### 1.2.7 稳妥构造函数模式

稳妥对象，指的是没有公共属性，而且其他方法也不引用this的对象。

```javascript
function Person(name, age, job) {
  var o = new Object();
  o.sayName = function () {
    alert(name);
  }
  return o;
}
```

## 1.3 继承

继承是OO语言中一个最为人津津乐道的概念。许多OO语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。

### 1.3.1 原型链

原型链作为实现继承的主要方法，基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

每个构造函数都有一个原型对象，原型对象包含一个指向构造函数的指针（constructor），而实例包含一个指向原型对象的内部指针（`__proto__`）。**如果让原型对象等于另一个类型的实例，此时的原型对象将包含另一个原型的指针，相应的，另一个原型中也包含着另一个构造函数的指针。如此层层递进，就构成了实例与原型的链条。这就是所谓的原型链的基本概念。**

```javascript
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function () {
  return this.property;
}
function subType() {
  this.subproperty = false;
}
// 继承了SuperType
subType.prototype = new SuperType();
subType.prototype.getSubValue = function () {
  return this.subproperty;
}
var instance = new subType();
console.log(instance.getSuperValue()); // true
```

