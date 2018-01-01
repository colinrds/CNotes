[toc]

## 1.面向对象

### 1.1 对象概述

- 面向对象特性：封装、继承、抽象、多态
- 类作用：通过类可以创建多个具有相同属性和方法的对象
- 对象定义：无序属性的集合，其属性可以包含基本值、对象或者函数
- 对象补充：每个对象都是基于一个引用类型创建，每个对象都继承于Object

### 1.2 对象属性

#### 1.2.1 对象属性概述

- 对象类型包括两个部分：数据属性和访问器属性，其又由4个描述特性组成(即属性的属性)
- ES5新增Object.defineProperty(obj, prop, descriptor)方法修改单个属性默认特性
- ES5新增Object.defineProperties(obj, props)方法用于批量修改属性默认特性

#### 1.2.2 数据属性

- 数据属性包含一个数据值的位置，在这个位置对值进行读写
- 数据属性由4个描述其行为的特性： 
    - Configurable：表示能否通过delete删除属性，能够修改属性特性，或者能否把属性修改为访问器属性，默认为true
    - Enumerable：表示能否通过for-in循环返回属性，默认为true；当循环时，实际上是执行Object.keys获取属性名，使用者必须通过obj[key]的方式手动获取值
    - Writable：表示能否修改属性的值，默认为true；当手动设置为false时，该属性只读，可以理解为一个不可变的常量
    - Value：即该属性的值，属性定义都是key-value的形式，读写属性的值都是操作该特性，默认为undefined（即当只是定义var obj；而没有赋值时，obj默认undefined）


Object.defineProperty(obj, prop, descriptor)方法用于修改属性默认的特性,使用如下：

```
var blog = {};
Object.defineProperty(blog,"author",{
  enumerable: false,//该属性不可在对象属性for-in循环中被枚举出
  configurable: false,//属性特性不可再修改
  writable: false,//该属性只读，值不可变
  value: 'kira'//该属性值默认为"kira"
})
```

为了验证特性是否生效，我们做一下测试验证一下
验证enumerable: false

```
for(var key in blog){
   console.log(key);
}
>>> undefined
```

由此可见，author属性无法在对象属性遍历时获取到，即不可枚举

![image](http://olphpb1zg.bkt.clouddn.com/uuuy1.png)

验证writable: false
```
blog.author = "sally";blog.author;
>>> "kira"
```

由此可见，author属性的value值真的不可变，即只读不可变

![image](http://olphpb1zg.bkt.clouddn.com/uuuy2.png)

验证configurable: false

```
Object.defineProperty(blog,"author",{
   value: 'sally'//重新定义特性value的值为sally
})
>>> Uncaught TypeError: Cannot redefine property: author
```

由此可见，当重新定义任意特性时，都会直接抛出TypeError的异常

```
console.log(delete blog.author);
blog.author;
>>> false；kira;
```

由此可见，无法通过delete删除该属性

![image](http://olphpb1zg.bkt.clouddn.com/uuuy3.png)

![image](http://olphpb1zg.bkt.clouddn.com/uuuy4.png)

#### 1.2.3 访问器属性

- 访问器属性不会包含值，他包含一个getter和setter函数
- 注意：虽然该属性非必需，但很多框架对于对象的处理都是基于get/set函数，如Vue.js
- 数据属性由4个描述其行为的特性： 
    - Configurable：表示能否通过delete删除属性，能够修改属性特性，或者能否把属性修改为访问器属性，默认为true
    - Enumerable：表示能否通过for-in循环返回属性，默认为true；当循环时，实际上是执行Object.keys获取属性名，使用者必须通过obj[key]的方式手动获取值
    - Get：读取属性(值)，默认值为undefined
    - Set：设置属性(值)，默认值为undefied

```
var blog = {
  articleNum:20
};
Object.defineProperty(blog,"articleNum",{
  get:function () {
    return this.articleNum;
  },
  set:function (addNum) {
    this.articleNum += addNum;
  }
})
```

当对属性进行赋值和取值操作时，会分别调用set和get方法

#### 1.2.4 读取属性特性

ES5新增Object.getOwnPropertyDescriptor(obj, prop)方法用于读取属性的特性

用法如下

```
var descriptor = Object.getOwnPropertyDescriptor(blog,"author");
console.log(descriptor.writable)
console.log(descriptor.enumerable)
console.log(descriptor.configurable)
console.log(descriptor.value)
>>> false;false;false;kira
```

![image](http://olphpb1zg.bkt.clouddn.com/uuuy5.png)

### 1.3 对象属性和方法的来源

- 构造函数内的，供实例化对象复制使用
- 构造函数外的，通过点语法添加，只能被构造函数访问，实例化对象无法访问
- 构造函数原型中的，实例化对象可以通过其原型链间接地访问，并供所有实例化对象共享
- 实例对象创建后，增加自身属性和方法，这些属性和方法只能被该对象访问

## 2.创建对象

### 2.1 字面量方式

#### 2.1.1 字面量方式概述

定义：字面量方式通常用于创建单个对象，因为其会直接实例化一个对象出来，使用如下：

#### 2.1.2 字面量方式使用

```
var blog = { name : "kira", age : 3,sex}
```

#### 2.1.3 字面量方式分析

- 优点： 自带单例模式，简单直接
- 缺陷：无法复用，当创建具有相同属性和方法的对象时，只能复制大量重复代码实现(尤其是属性和方法够多情况下)

例如，同样是具有name、age、sex、home属性，但会有很多重复代码出现，严重的代码冗余

```
var kira = { name : "kira", age : 4, sex : "boy", home:"shanghai"}
var sally = { name : "sally", age : 3, sex : "girl", home:"shanghai"}
var mengmeng = { name : "mengmeng", age : 2, sex : "girl", home:"shanghai"}
```

### 2.2 工厂模式

#### 2.2.1 工厂模式概述

工厂模式属于创建型模式的一种，其通过函数封装了创建具体对象的过程并最终返回一个新的对象

#### 2.2.2 工厂模式使用

```
function createObj(name,age,sex,home) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.set = sex;
  o.home = home;
  o.growUp = function () {
    this.age++;
  }
  return o;
}
var kira = createObj("kira",4,"boy","shanghai");
var sally = createObj("sally",3,"girl","shanghai");
var mengmeng = createObj("mengmeng",2,"girl","shanghai");
```

对比字面量可以很明显的发现重复代码少了很多

#### 2.2.3 工厂模式分析

- 优点： 通过封装创建过程减少代码量
- 缺陷： 无法区分其具体属于哪个类型(都是Object类型)，因为没有定义构造函数

### 2.3 构造函数模式

#### 2.3.1 构造函数模式概述

定义： 构造函数可用来创建特定类型的对象

#### 2.3.2 构造函数模式使用

```
function FamilyMember(name,age,sex,home) {
  this.name = name;
  this.age = age;
  this.sex = sex;
  this.home = home;
  this.growUp = function () {
    this.age++;
  }
}
var kira = new FamilyMember("kira",4,"boy","shanghai");
var sally = new FamilyMember("sally",3,"girl","shanghai");
var mengmeng = new FamilyMember("mengmeng",2,"girl","shanghai");
```

#### 2.3.3 构造函数模式分析

- 优点： 可以具有类型概念
- 缺陷： 若在构造函数内部定义方法时，每个方法都要在每个对象实例上重新创建一遍

```
function FamilyMember() {
  this.growUp = function () {
    this.age++;
  }
  //相等于
  this.growUp = new Function(){};//每次都是新的函数对象
}
```

#### 2.3.4 调用构造函数过程

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象(即this指向新对象)
3. 执行构造函数中的代码，即初始化对象(为新对象添加定义的属性和方法)
4. 返回新对象

#### 2.3.5 支持多种调用方式

由于es5时还不存在定义构造函数的特殊语法(es6开始支持)，因此构造函数也是函数，其支持多种调用方式

```
//当做构造函数调用
var kira = new FamilyMember("kira",4,"boy","shanghai");
//当做普通函数调用
FamilyMember("sally",3,"girl","shanghai");
//在另一对象的作用域中调用
var o = new Object();
FamilyMember.call(o,"mengmeng",2,"girl","shanghai")
或 FamilyMember.apply(o,["mengmeng",2,"girl","shanghai"])
```

#### 2.3.6 构造函数模式 vs 工厂模式

- 与工厂模式相比(或者跟普通方法)，其存在以下特点： 
    - 没有显式地创建对象
    - 直接将属性和方法赋给this对象
    - 没有return语句
    - 函数名必须以大写字母开头(代码规范)
    - 使用时推荐通过new关键字创建新对象

### 2.4 原型模式

#### 2.4.1 原型对象

- 定义：每个函数都有一个原型属性(prototype)指向一个原型对象，该对象包含可以由特定类型的所有实例共享的属性和方法
- 作用：让所有对象实例共享它所包含的原型属性和方法

#### 2.4.2 原型对象、构造函数、对象实例之间的关系(重中之重)

- 原型对象、构造函数、对象实例之间的关系是JS面向对象的最根本所在
- 此处只涉及单类型，多类型的会在继承的原型链实现中进一步阐述
- 比如对象创建、对象属性查找、原型链、继承等JS面向对象非常重要的内容也都是基于以下三条规则，希望读者一定理解清楚

##### 2.4.2.1 构造函数与原型对象

规则1：每个构造函数都有一个原型对象，通过prototype属性指向其原型对象

![image](http://olphpb1zg.bkt.clouddn.com/uuuy6.png)

##### 2.4.2.2 原型对象与构造函数

规则2：所有原型对象都会有一个指向构造函数的指针，通过contructor属性指向该构造函数
补充：通过该构造函数，我们可以继续为原型对象添加其他属性和方法

![image](http://olphpb1zg.bkt.clouddn.com/uuuy7.png)

##### 2.4.2.3 实例与原型对象

规则3：每个实例都有一个指向其构造函数的原型对象的内部指针，非IE下是 `_proto_`

补充：实例才有`_proto_`指针，构造函数才有prototype属性，这个要分清楚，虽然指向一致

![image](http://olphpb1zg.bkt.clouddn.com/uuuy8.png)

> 注意：实例本身是没有实例对象的，也不会有原型对象的拷贝，其指向的是同一个构造函数的原型对象，注意是构造函数的原型对象，即该原型对象属于构造函数！！

> 注意：由于每个实例的原型指针指向的是同一个构造函数的原型对象，即同一个对象，因此当该原型对象的属性变更时，由于指向的是同一个原型对象，自然所有实例对象都会被影响，同时笔者认为这也是共享实现的原理所在

> 注意： 由于IE没有 `_proto_`，因此不推荐在生产阶段直接使用该属性，会有兼容性问题！

#### 2.4.3 使用原型模式创建对象

```
function FamilyMember() {}
FamilyMember.prototype.home = "shanghai";
FamilyMember.prototype.move2NewHome = function (newHome) {
  this.home = newHome;
}
var kira = new FamilyMember();
var sally = new FamilyMember();
console.log(kira.home)
console.log(sally.home)
FamilyMember.prototype.home = "hangzhou";
console.log(kira.home)
console.log(sally.home)

>>> shanghai;shanghai;hangzhou;hangzhou;
```

由此可发现，当变更同一个类型的原型对象的属性值会影响到所有该类型的对象实例

![image](http://olphpb1zg.bkt.clouddn.com/uuuy9.png)

#### 2.4.4 原型字面量对象

- 使用：由于原型本事是以对象形式存在，即可以使用字面量的创建方式简化代码

```
function FamilyMember() {}
FamilyMember.prototype = {
  home:"shanghai",
  move2NewHome : function (newHome) {
    this.home = newHome;
  }
}
```

- 隐患：当没有在原型字面量指定构造函数的话，其构造函数会被重置
- 解决：当构造函数属性必须时，可以通过在原型字面量中显式指明contructor

![image](http://olphpb1zg.bkt.clouddn.com/uuuy10.png)

#### 2.4.5 对象属性/方法查找

- 对象属性/方法的查找遵循如下规则： 
    - 先在自身属性或方法中查找 
        - 找到直接调用
        - 找不到则查找原型对象的属性和方法中是否有： 
            - 如果有则调用
            - 否则若存在原型链的情况下： 
                - 沿着原型链继续找，找到则调用
                - 当一直到Object都没有的话(所有对象都继承自Object)，直接报错

#### 2.4.6 动态原型

- 定义：动态原型指的是可以随时随地的新增原型属性和方法或重置原型，这跟JS语言本身的动态特性有关
- 隐患：虽然动态特性增加了灵活性，但其实也增加了很多出错的机会，比如当对象已创建后又重写了其构造函数的原型对象，那么重写的原型对象切断了现有原型与任何之前已存在的对象实例之间的联系，他们的引用仍然是最初的原型，即重写的原型对象跟之前已创建的对象一点关系都没有

```
function FamilyMember() {}
var kira = new FamilyMember();
FamilyMember.prototype = {
  home:"shanghai",
  move2NewHome : function (newHome) {
    this.home = newHome;
  }
}
kira.move2NewHome("hangzhou");

>>> Uncaught TypeError: kira.move2NewHome is not a function
>>> 重写的原型对象跟之前已创建的对象一点关系都没有，自然没有重写的原型对象的方法
```

![image](http://olphpb1zg.bkt.clouddn.com/uuuy11.png)

#### 2.5.7 原型问题

##### 2.5.7.1 原型属性默认同值的不便

由于省略构造函数传递初始化参数会导致所有实例对象在默认情况下都将取得相同的属性值

```
function FamilyMember() {}
FamilyMember.prototype.home = "shanghai";
var kira = new FamilyMember();
var sally = new FamilyMember();
console.log(kira.home)
console.log(sally.home)
>>> shanghai;shanghai;
```

由此可发现，所有实例对象的home原型属性初始值都是一样的

##### 2.5.7.2 引用类型属性无法私有化

原型属性的共享问题无法实现属性值私有化，尤其是当属性是引用类型的时候，比如每个家庭成员都有自己喜欢的食物，比如萌萌喜欢吃狗狗零食，而这个人不能吃；反过来，人能吃的，狗的不一定适合吃，比如巧克力

```
function FamilyMember() {}
FamilyMember.prototype.foods = ["牛肉","鸡肉","骨头"];
var sally = new FamilyMember();
var mengmeng = new FamilyMember();
sally.foods.push("巧克力");//媳妇添加了巧克力到食谱中
console.log(sally.foods);
mengmeng.foods.push("狗零食");//萌萌添加了狗零食到食谱中
console.log(sally.foods);
>>> ["牛肉", "鸡肉", "骨头", "巧克力"]
>>> ["牛肉", "鸡肉", "骨头", "巧克力", "狗零食"]
```

那么问题来了，狗狗能吃巧克力吗？！人能吃狗零食吗？！有谁可以试一下？...

### 2.6 组合模式

实现：即结合构造函数和原型的方式创建对象

```
//构造函数
function FamilyMember(name) {
    this.name = name;
}
//原型
FamilyMember.prototype = {
  constructor : FamilyMember,//为了构造函数安全，可以选择显式指明构造函数
  home:"shanghai",
  move2NewHome : function (newHome) {
    this.home = newHome;
  }
}
var kira = new FamilyMember("kira");
```

## 3.继承

- 作用：子类继承父类的所有方法和属性，即子类能够拥有父类的所有属性和方法
- 继承方法：面向对象语言一般都支持两种方式：接口继承和实现继承 
    - 接口继承：只继承方法签名，比如Java的implements
    - 实现继承：继承实际的方法，比如Java的extends
- JS的继承：由于JavaScript本身是面向函数的语言， 其没有方法签名，只支持实现继承，且是通过原型链实现的有序单继承

![image](http://olphpb1zg.bkt.clouddn.com/uuuy12.png)

### 3.1 原型链继承

#### 3.1.1 原型链

- 定义：所谓原型链即是通过原型指针(如`_proto_`)串联起来的一个链表
- 形成：每个函数都有一个原型属性prototype指向自身的原型，由该函数创建的对象也有一个`_proto_`指针指向该函数的原型，而函数的原型是一个对象，因此该对象也会有一个`_proto_`指向自己的原型对象，这样逐层有序串联起来直到Object对象的原型，这样就形成了原型链
- 特点：当前执行的对象的所属原型在原型链的链首，Object的原型在原型链的链尾，中间根据层次有序串联，这跟作用域链的原理一致（若有机会笔者会专门开番讲作用域链，前提是媳妇要求）

> 注意： IE中没有`_proto_`这个指针，因此生产环境不能使用，会有兼容性问题

#### 3.1.2 原型链继承

##### 3.1.1.1 原型链继承概述

- 实现： 令一个构造函数的原型等于另一个不同构造函数的实例
- 原理： 利用实例对象具有指向其构造函数原型的指针(`_proto_`)形成原型链，而原型又有指向构造函数的属性，即具有了构造函数的属性和方法
- 好处： 子构造函数创建的实例可以拥有父构造函数所有的属性和方法
- 补充： 实例也可以换成原型对象，但后者有一定使用局限性，后面会介绍到

##### 3.1.1.2 原型链继承使用

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(sex) {
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
//原型链继承
//将子类型构造函数的原型对象指向父构造函数的实例对象
FamilyMember.prototype = new Member();//注意：父实例对象不需要传构造参数
FamilyMember.prototype.walkDog = function(){
    console.log("遛狗去了");
}

var kira = new FamilyMember("boy");
console.log(kira.home)
kira.growUp();
console.log(kira.age)
>>> shanghai;4
```

由此可知，子类型构造器实例能够继承和使用父构造函数的growUp方法和age属性，即继承有效

##### 3.1.1.3 原型链继承原理

我们可以通过查看子构造函数实例的内部结构清晰的看出其的继承体系

![image](http://olphpb1zg.bkt.clouddn.com/uuuy13.png)

- 原理剖析：
    - 子类型构造函数实例具有子类型构造函数的属性和方法，而实例对象又拥有一个原型指针`_proto_`指向其构造函数的原型对象，即父构造函数的实例
    - 由于子类型构造函数的原型对象指向父类型构造函数的实例，因此可以进而获取到父类型构造函数的属性和方法以及父类型构造函数的原型属性和方法，从而继承实现
    - 最终所有对象都会通过的`_proto_`指针形成的原型链指向Object的原型，继承结束(作用域链到链尾)

##### 3.1.1.3 原型链继承问题

- 同使用原型创建对象是一样的引用类型问题，读者可参见本篇的原型创建对象部分
- 在创建子类型构造函数实例时，不能向父类型构造函数中传递参数

不能向父类型构造函数中传递参数

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(sex) {
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.prototype = new Member();
var kira = new FamilyMember("kira");
kira.name;
>>> undefined
```

由此可知，我们无法向Member中传递name参数，即无法向父类型构造函数传递值

为了检验对原型继承的理解，读者可以试着思考以下问题~

- 挑战问题1： 如果子类型构造函数的原型对象指向的是父构造函数的原型对象呢？ 
- 挑战问题2： 如果子类型构造函数的原型对象指向的是父构造函数呢？ 
- 挑战问题3： 能否相互继承？即父构造函数的原型是子类型构造函数实例，子类型构造函数原型是父构造函数实例？ 
- 挑战问题4： 当子类型构造函数的原型对象指向的是父构造函数的实例，那么变更子构造函数的原型，如新增原型属性和方法，会影响到父构造函数的原型对象吗？

### 3.2 借用构造函数继承

#### 3.2.1 借用构造函数继承概述

- 实现：在子类型构造函数的内部调用父类型构造函数
- 原理：在子类型构造函数调用时初始化父类型构造函数，从而具备父类型构造函数的属性和方法
- 好处：能够传递参数，即可以在子类型构造函数中向父类型构造函数传递参数

#### 3.2.2 借用构造函数继承使用

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(name,sex) {
  Member.apply(this,arguments);
  //Member.call(this,name);
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
var kira = new FamilyMember("kira","boy");
console.log(kira.name)
>>> kira;
```

由此可知，子实例可以传递构造参数给父构造函数

#### 3.2.3 借用构造函数继承问题

问题： 与借用构造函数创建对象一样，函数复用就没有意义；同时父类型构造函数的所有原型方法对子类型构造函数不可见，即并没有继承父类型构造器的原型属性和方法

### 3.3 组合继承

#### 3.3.1 组合继承概述

- 实现：将原型链和借用构造函数组合一起使用
- 原理：使用原型链实现对原型属性和方法的继承，通过借用构造函数实现对实例属性和方法的继承
- 好处：通过在原型上定义方法实现函数复用，同时保证每个实例都有自己的属性

#### 3.3.2 组合继承使用

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(name,sex) {
  Member.apply(this,arguments);//父类型构造函数第二次初始化
  //Member.call(this,name);
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.prototype = new Member();//父类型构造函数第一次初始化

var kira = new FamilyMember("kira","boy");
console.log(kira.name)
>>> kira;
```

由此可知，子实例可以传递构造参数给父构造函数

#### 3.3.3 组合继承问题

由于组合继承已经能够有效的实现继承功能，也是最常用的继承方式
但该模式的问题在于父构造函数会被初始化两次：一次是创建子类型构造器原型时，一次是在子类型构造函数内部调用时，虽然子类最终会包含父类所有属性和方法，但不得不每次都要重写一次属性，即二次初始化

### 3.4 变化的组合继承

#### 3.4.1 变化的组合继承概述

- 实现：将一个函数的原型对象指向另一个不同类型函数的原型对象
- 原理：通过将原型对象之间串联起来，形成原型链

好处：避免二次初始化

#### 3.4.2 变化的组合继承使用

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(name,sex) {
  Member.apply(this,arguments);
  //Member.call(this,name);
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.prototype = Member.prototype;
// FamilyMember.prototype = new Member();

var kira = new FamilyMember("boy");
console.log(kira.home)
kira.growUp();
console.log(kira.age)
>>>  shanghai；4
```

由此可见，子类型构造函数实例可以使用父类型构造函数的原型的属性和方法

#### 3.4.3 变化的组合继承问题

由于把父类型构造函数的原型直接赋值给子类型构造函数的原型，那么如果对子类型构造函数的原型做了修改，那么这个修改同时也会影响到父类型构造函数的原型，进而影响父类型构造函数的原型甚至其他继承于父类型构造函数的不同子类型构造函数

### 3.5 原型式继承

#### 3.5.1 原型式继承概述

- 实现：使用临时借用构造函数的方式创建新的对象
- 原理：原型可以基于已有对象创建新对象，前提是必须有一个对象作为另一个对象的基础
- 好处：可以不必创建自定义类型

#### 3.5.2 原型式继承使用

```
function createFamilyMember(member) {
  var F = function () {};//创建一个临时构造函数
  F.prototype = member;
  return new F();
}
```

ES5通过新增Object.create()方法规范化原型式继承

```
var member = {
  name : "kira",
  age : 3,
  growUp : function () {
    this.age++;
  }
}
var kira = createFamilyMember(member)
kira.growUp();
console.log(kira.age);
```

原型式继承的变动，即可以通过定义构造函数实现原型继承

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}

var kira = createFamilyMember(new Member("kira"))
kira.growUp();
console.log(kira.age);
console.log(kira.play());
```

ES5通过新增Object.create()方法规范化原型式继承

```
var member = {
  name : "kira",
  age : 3,
  growUp : function () {
    this.age++;
  },
  foods : ["大闸蟹","小龙虾"]
}
var sally = Object.create(member)
sally.name = "sally";
sally.growUp();
console.log(sally.age)
>>> 4；
```

由此可见，继承了父对象的自身属性和方法

#### 3.5.3 原型式继承问题

问题等同于字面量创建对象
当不想创建构造函数，只想让一个对象与另一个对象保持类型的情况下可使用该方式

### 3.6 寄生式继承

#### 3.6.1 寄生式继承概述

- 实现：创建一个用于封装继承过程的函数，内部增强，并返回该函数
- 原理：在原型式继承基础上增加内部增加的能力
- 好处：当主要考虑对象而不是自定义类型和构造函数的情况，该模式是一种有效模式

#### 3.6.2 寄生式继承使用

```
var member = {
  name : "kira",
  age : 3,
  growUp : function () {
    this.age++;
  },
  foods : ["大闸蟹","小龙虾"]
}
function createFamilyMember(member) {
  var newMember = createObj(member);
  newMember.walkDog = function () {
    console.log("遛狗子啦~~~")
  }
}
function createObj(obj) {
  var F = function () {};
  F.prototype = obj;
  return new F();
}
```

#### 3.6.3 寄生式继承问题

在考虑对象而不是自定义类型和构造函数的情况该，寄生式继承也是一种有用的玩意
无法函数复用，问题等同于构造函数模式

### 3.7 寄生组合式继承(圣杯模式继承)

#### 3.7.1 寄生组合式继承概述

- 实现：寄生式和组合式的结合
- 原理：通过借用构造函数继承属性，通过原型链的混成形式来继承方法
- 好处：集寄生式继承和组合继承的优点于一身，是实现基于类型集成的最有效方式

#### 3.7.2 寄生组合式继承使用

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(name,sex) {
  Member.apply(this,arguments);
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
function createObj(obj) {
  var F = function () {};
  F.prototype = obj;//挑战问题5
  return new F();
}
function extendPrototype(superClassFun,subClassFun) {
  var prototype = createObj(superClassFun.prototype);//创建对象-挑战问题5
  prototype.constructor = subClassFun;//指明构造函数，算是增强对象
  subClassFun.prototype = prototype;//指定对象
}
extendPrototype(Member,FamilyMember);//原型继承
FamilyMember.prototype.walkDog = function () {
  console.log("遛狗子啦~~~")
}
```

![image](http://olphpb1zg.bkt.clouddn.com/uuuy14.png)

## 4 挑战问题

### 4.1 挑战问题1

挑战问题1：如果子类型构造函数的原型对象指向的是父构造函数的原型对象呢？

1. 在原型链模式基础上改动：

```
//在原型链模式基础上我们只做一个改动，改成父构造函数的原型对象
//注意此时是没有使用借用构造函数
FamilyMember.prototype = Member.prototype;
var kira = new FamilyMember("boy");
console.log(kira.home)
kira.growUp();
console.log(kira.age)
>>> shanghai;
>>> VM326:22 Uncaught TypeError: kira.growUp is not a function
```

由此可发现，可以使用父构造器的原型属性，但不可以使用父构造器的自身属性

![image](http://olphpb1zg.bkt.clouddn.com/uuuy15.png)

2. 在组合模式基础上改动：

```
//在组合模式基础上改动
//注意此时是同时使用原型链和借用构造函数
FamilyMember.prototype = Member.prototype;
FamilyMember.prototype.food.push("小笼包");//通过子构造函数向引用类型原型属性变更值
Member.prototype.food;
delete FamilyMember.prototype.foods;//通过子构造函数删除原型属性
console.log(Member.prototype.food)
>>> ["大闸蟹", "小龙虾", "小笼包"]
>>> undefined
```

由此可发现，子类型构造函数对其原型的修改会影响到父类型构造函数，这就尴尬了...

![image](http://olphpb1zg.bkt.clouddn.com/uuuy16.png)

**结论**

当原型对象指向父构造函数的原型对象时，会有两个问题：

- 当只使用原型链继承时，可以使用父构造器的原型属性，但不可以使用父构造器的自身属性
- 由于父构造器的原型对象也是个对象，即引用类型变量，修改子构造器原型的同时也会修改父构造器的原型(因为指向的是同一个对象)，这在继承原则上是非法(子类的任何行为不能对父类产生影响，尤其当父类有众多子类时)

### 4.2 挑战问题2

挑战问题2： 如果子类型构造函数的原型对象指向的是父构造函数呢？

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(sex) {
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.prototype = Member;//原型改成父类型构造器
FamilyMember.prototype.walkDog = function(){
    console.log("遛狗去了");
}

var kira = new FamilyMember("boy");
console.log(kira.home);
kira.growUp();
>>> undefined
>>> Uncaught TypeError: kira.growUp is not a function
```

由此可知，调用父类型构造函数的方法会直接报错，而此时kira.home这个home属性跟父类型构造函数一点关系都没，是对象私有的新增属性，所以才是undefined

![image](http://olphpb1zg.bkt.clouddn.com/uuuy17.png)

**结论**

当子构造函数的原型指向父类型构造函数时：

- 实际上没有并形成继承，不能使用父类型构造函数的任何属性和方法
- 原因在于Member返回的是一个Function函数对象，而函数只有prototype属性，而函数对象的_proto_指针指向的是Function的原型，这样是无法与Member函数的原型串联，即原型链中不会包含Member函数的原型

### 4.3 挑战问题3

挑战问题3： 能否相互继承？即父构造函数的原型是子类型构造函数实例，子类型构造函数原型是父构造函数实例？

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(sex) {
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.walkDog = function () {
    console.log("遛狗子啦~~~")
}
FamilyMember.prototype = new Member();//先正常继承父类型构造函数对象
Member.prototype = new FamilyMember();//然后将父类型构造函数的原型指向子构造函数的对象

var member = new Member();
var familyMember = new FamilyMember();
familyMember.play();
member.walkDog();
>>> 一起high~~~
>>> Uncaught TypeError: member.walkDog is not a function
```

由此可知，由于父类型构造函数对象无法使用(我们认为的)子构造函数原型方法，即其实是无法实现真正意义的相互继承

- 子类型构造函数实例
- 
![image](http://olphpb1zg.bkt.clouddn.com/uuuy18.png)

- 父类型构造函数实例
- 
![image](http://olphpb1zg.bkt.clouddn.com/uuuy19.png)

**结论**

能否实现相互继承：

- 实际上是无法实现真正的相互继承，原因是父构造函数的原型对象被重写
- 重写原型对象会切断先有原型与任何之前已经存在的对象实例之间的联系；它们引用的仍然是最初的原型
- 在例子中由于FamilyMember的原型是Member，而Member的原型又指向FamilyMember，其实最终是Member又指向了Member，即Member->FamilyMember->Member

### 4.4 挑战问题4

挑战问题4： 当子类型构造函数的原型对象指向的是父构造函数的实例，那么变更子构造函数的原型，如新增原型属性和方法，会影响到父构造函数的原型对象吗？

```
function Member(name) {
  this.name = name;
  this.age = 3;
  this.growUp = function () {
    this.age++;
  }
}
Member.prototype.home = "shanghai";
Member.prototype.play = function () {
  console.log("一起high~~~")
}
function FamilyMember(sex) {
  this.sex = sex;
  this.givePhone = function (phone) {
    this.phone = phone;
  }
}
FamilyMember.prototype = new Member();//注意：父实例对象不需要传构造参数
FamilyMember.prototype.walkDog = function(){
    console.log("遛狗去了");
}
var kira = new FamilyMember("boy");
kira;
```

![image](http://olphpb1zg.bkt.clouddn.com/uuuy20.png)

**结论**

会影响到父构造函数的原型对象吗：

- 实际上是不会的，原因在于子类型构造函数的原型指向父类型构造函数的实例，也就是说当像子类型构造函数添加所谓的原型方法时，其实质是像父类型构造函数的实例添加实例方法，即对象添加实例方法

### 4.4 挑战问题5

挑战问题5： createObj方法中为什么传入的是原型对象？为什么F是空构造函数？

- 因为有可能父类型构造函数会比较庞大，而且父类型构造函数会有一些副作用，比如说会执行大量的计算任务