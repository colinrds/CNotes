[toc]

继承是面向对象编程中又一非常重要的概念，JavaScript支持实现继承，不支持接口继承，实现继承主要依靠原型链来实现的。

## 原型链

首先得要明白什么是原型链，在一篇文章看懂proto和prototype的关系及区别中讲得非常详细

原型链继承基本思想就是让一个原型对象指向另一个类型的实例


```
function SuperType() {
  this.property = true
}
SuperType.prototype.getSuperValue = function () {
  return this.property
}
function SubType() {
  this.subproperty = false
}
SubType.prototype = new SuperType()
SubType.prototype.getSubValue = function () {
  return this.subproperty
}
var instance = new SubType()
console.log(instance.getSuperValue()) // true
```

代码定义了两个类型SuperType和SubType，每个类型分别有一个属性和一个方法，SubType继承了SuperType，而继承是通过创建SuperType的实例，并将该实例赋给SubType.prototype实现的。

实现的本质是重写原型对象,代之以一个新类型的实例，那么存在SuperType的实例中的所有属性和方法，现在也存在于SubType.prototype中了。

我们知道，在创建一个实例的时候，实例对象中会有一个内部指针指向创建它的原型，进行关联起来，在这里代码SubType.prototype = new SuperType()，也会在SubType.prototype创建一个内部指针，将SubType.prototype与SuperType关联起来。

所以instance指向SubType的原型，SubType的原型又指向SuperType的原型，继而在instance在调用getSuperValue()方法的时候，会顺着这条链一直往上找。

### 添加方法

在给SubType原型添加方法的时候，如果，父类上也有同样的名字，SubType将会覆盖这个方法，达到重新的目的。 但是这个方法依然存在于父类中。

记住不能以字面量的形式添加，因为，上面说过通过实例继承本质上就是重写，再使用字面量形式，又是一次重写了，但这次重写没有跟父类有任何关联，所以就会导致原型链截断。

```
function SuperType() {
  this.property = true
}
SuperType.prototype.getSuperValue = function () {
  return this.property
}
function SubType() {
  this.subproperty = false
}
SubType.prototype = new SuperType()
SubType.prototype = {
  getSubValue:function () {
   return this.subproperty
  }
}
var instance = new SubType()
console.log(instance.getSuperValue())  // error
```

### 问题

单纯的使用原型链继承，主要问题来自包含引用类型值的原型。

```
function SuperType() {
  this.colors = ['red', 'blue', 'green']
}
function SubType() {
}
SubType.prototype = new SuperType()
var instance1 = new SubType()
var instance2 = new SubType()
instance1.colors.push('black')
console.log(instance1.colors)  // ["red", "blue", "green", "black"]
console.log(instance2.colors) // ["red", "blue", "green", "black"]
```

在SuperType构造函数定义了一个colors属性，当SubType通过原型链继承后，这个属性就会出现SubType.prototype中，就跟专门创建了SubType.prototype.colors一样，所以会导致SubType的所有实例都会共享这个属性，所以instance1修改colors这个引用类型值，也会反映到instance2中。

## 借用构造函数

此方法为了解决原型中包含引用类型值所带来的问题。

这种方法的思想就是在子类构造函数的内部调用父类构造函数，可以借助apply()和call()方法来改变对象的执行上下文


```
function SuperType() {
  this.colors = ['red', 'blue', 'green']
}
function SubType() {
  // 继承SuperType
  SuperType.call(this)
}
var instance1 = new SubType()
var instance2 = new SubType()
instance1.colors.push('black')
console.log(instance1.colors)  // ["red", "blue", "green", "black"]
console.log(instance2.colors) // ["red", "blue", "green"]
```

在新建SubType实例是调用了SuperType构造函数，这样以来，就会在新SubType对象上执行SuperType函数中定义的所有对象初始化代码。

结果，SubType的每个实例就会具有自己的colors属性的副本了。

### 传递参数

借助构造函数还有一个优势就是可以传递参数

```
function SuperType(name) {
  this.name = name
}
function SubType() {
  // 继承SuperType
  SuperType.call(this, 'Jiang')
 
  this.job = 'student'
}
var instance = new SubType()
console.log(instance.name)  // Jiang
console.log(instance.job)   // student
```

### 问题

如果仅仅借助构造函数，方法都在构造函数中定义，因此函数无法达到复用

## 组合继承(原型链+构造函数)

组合继承是将原型链继承和构造函数结合起来，从而发挥二者之长的一种模式。

思路就是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。

这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。

```
function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}
function SubType(name, job) {
  // 继承属性
  SuperType.call(this, name)
 
  this.job = job
}
// 继承方法
SubType.prototype = new SuperType()
SubType.prototype.constructor = SuperType
SubType.prototype.sayJob = function() {
  console.log(this.job)
}
var instance1 = new SubType('Jiang', 'student')
instance1.colors.push('black')
console.log(instance1.colors) //["red", "blue", "green", "black"]
instance1.sayName() // 'Jiang'
instance1.sayJob()  // 'student'
var instance2 = new SubType('J', 'doctor')
console.log(instance2.colors) // //["red", "blue", "green"]
instance2.sayName()  // 'J'
instance2.sayJob()  // 'doctor'
```

这种模式避免了原型链和构造函数继承的缺陷，融合了他们的优点，是最常用的一种继承模式。

## 原型式继承

借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。

```
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```

在object函数内部，先创建一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回这个临时类型的一个新实例。

本质上来说，object对传入其中的对象执行了一次浅复制。

```
var person = {
  name: 'Jiang',
  friends: ['Shelby', 'Court']
}
var anotherPerson = object(person)
console.log(anotherPerson.friends)  // ['Shelby', 'Court']
```

这种模式要去你必须有一个对象作为另一个对象的基础。

在这个例子中，person作为另一个对象的基础，把person传入object中，该函数就会返回一个新的对象。

这个新对象将person作为原型，所以它的原型中就包含一个基本类型和一个引用类型。

所以意味着如果还有另外一个对象关联了person，anotherPerson修改数组friends的时候，也会体现在这个对象中。

**Object.create()方法**

ES5通过Object.create()方法规范了原型式继承，可以接受两个参数，一个是用作新对象原型的对象和一个可选的为新对象定义额外属性的对象，行为相同，基本用法和上面的object一样，除了object不能接受第二个参数以外。

```
var person = {
  name: 'Jiang',
  friends: ['Shelby', 'Court']
}
var anotherPerson = Object.create(person)
console.log(anotherPerson.friends)  // ['Shelby', 'Court']
```

## 寄生式继承

寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数。

```
function createAnother(o) {
  var clone = Object.create(o) // 创建一个新对象
  clone.sayHi = function() { // 添加方法
    console.log('hi')
  }
  return clone  // 返回这个对象
}
var person = {
  name: 'Jiang'
}
var anotherPeson = createAnother(person)
anotherPeson.sayHi()
```

基于person返回了一个新对象anotherPeson，新对象不仅拥有了person的属性和方法，还有自己的sayHi方法。

在主要考虑对象而不是自定义类型和构造函数的情况下，这是一个有用的模式。

## 寄生组合式继承

在前面说的组合模式(原型链+构造函数)中，继承的时候需要调用两次父类构造函数。

父类

```
function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
```

第一次在子类构造函数中

```
function SubType(name, job) {
  // 继承属性
  SuperType.call(this, name)
 
  this.job = job
}
```

第二次将子类的原型指向父类的实例

```
// 继承方法
SubType.prototype = new SuperType()
```

当使用var instance = new SubType()的时候，会产生两组name和color属性，一组在SubType实例上，一组在SubType原型上，只不过实例上的屏蔽了原型上的。

使用寄生式组合模式，可以规避这个问题。

这种模式通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。

基本思路：不必为了指定子类型的原型而调用父类的构造函数，我们需要的无非就是父类原型的一个副本。

本质上就是使用寄生式继承来继承父类的原型，在将结果指定给子类型的原型。

```
function inheritPrototype(subType, superType) {
  var prototype = Object.create(superType.prototype)
  prototype.constructor = subType
  subType.prototype = prototype
}
```

该函数实现了寄生组合继承的最简单形式。

这个函数接受两个参数，一个子类，一个父类。

第一步创建父类原型的副本，第二步将创建的副本添加constructor属性，第三部将子类的原型指向这个副本。

```
function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}
function SubType(name, job) {
  // 继承属性
  SuperType.call(this, name)

  this.job = job
}
// 继承
inheritPrototype(SubType, SuperType)
var instance = new SubType('Jiang', 'student')
instance.sayName()
```

补充：直接使用Object.create来实现，其实就是将上面封装的函数拆开，这样演示可以更容易理解。

```
function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}
function SubType(name, job) {
  // 继承属性
  SuperType.call(this, name)
 
  this.job = job
}
// 继承
SubType.prototype = Object.create(SuperType.prototype)
// 修复constructor
SubType.prototype.constructor = SubType
var instance = new SubType('Jiang', 'student')
instance.sayName()
```

ES6新增了一个方法，Object.setPrototypeOf，可以直接创建关联，而且不用手动添加constructor属性。

```
// 继承
Object.setPrototypeOf(SubType.prototype, SuperType.prototype)
console.log(SubType.prototype.constructor === SubType) // true
```