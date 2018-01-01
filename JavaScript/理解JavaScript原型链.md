在JS中，原型链有时候让人觉得很胡里花哨，又是prototype、__proto__又是各种指向什么的，让人觉得很头疼。如果你也有这种感觉，或许这篇文章可以帮助到你

## 一、认识原型

1. 先来一串代码

```
var Person = function(msg){
    this.msg = msg;
}
var person1 = new Person("wanger")

person1.constructor===Person;    //true
Person === Person.prototype.constructor; //true
person1.__proto__ === Person.prototype; //true
person1.__proto__.constructor === person1.constructor //true
```

看晕了吧？是不是很胡里花哨？不用担心，其实一张图就能了明白这其中的关系：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_1.jpg)

- 蓝色的是构造函数
- 绿色的是构造函数实例出来的对象
- 橙色的是构造函数的prototype,也是构造函数实例出来的对象的原型（它其实也是一个对象）

2. 这里特别要注意的是prototype与`__proto__`的区别，prototype是函数才有的属性，而`__proto__`是每个对象都有的属性。(`__proto__`不是一个规范属性，只是部分浏览器实现了此属性，对应的标准属性是[[Prototype]])。

## 二、认识原型链

1. 我们刚刚了解了原型，那原型链在哪儿呢？不要着急，再上一张图：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_2.jpg)

通过这张图我们可以了解到,person1的原型链是：

```
person1 ----> Person.prototype ----> Object.prototype ----> null
```

2. 事实上，函数也是一个对象，所以，Person的原型链是：

```
Person ----> Function.prototype ----> Object.prototype ----> null
```

由于Function.prototype定义了apply()等方法，因此，Person就可以调用apply()方法。

3. 如果把原型链的关系都显示清楚，那会复杂一些，如下图：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_3.jpg)

这里需要特别注意的是：所有函数的原型都是Function.prototype,包括Function构造函数和Object构造函数（如图中的标红部分）

## 三、原型链的继承

1. 假设我们要基于Person扩展出Student，Student的构造如下：

```
function Student(props) {
    // 调用Person构造函数，绑定this变量:
    Person.call(this, props);
    this.grade = props.grade || 1;
}
```

但是，调用了Person构造函数不等于继承了Person，Student创建的对象的原型是：

```
new Student() ----> Student.prototype ----> Object.prototype ----> null
```

示意图如下所示：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_4.jpg)

必须想办法把原型链修改为：

```
new Student() ----> Student.prototype ----> Person.prototype ----> Object.prototype ----> null
```

示意图如下所示：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_5.jpg)

那我们应该怎么修改呢？仔细观察两张图的差异，我们会发现，如果我们将Student的prototype改成person1对象不就大功告成了？于是有了下面的代码：

```
Student.prototype = person1 ;
```

但是这时候有个问题:

```
Student.prototype.constructor === Student; //false
```

原来Student.prototype(即person1)的constructor指向的还是Person，这时候还需要我们再改一下代码：

```
Student.prototype.constructor = Student;
```

这样就能把Student的原型链顺利的修改为： new Student() ----> Student.prototype ----> Person.prototype ----> Object.prototype ----> null 了；

完整的代码显示如下：

```
var Person = function(msg){
    this.msg = msg;
}
var Student = function(props) {
    // 调用Person构造函数，绑定this变量:
    Person.call(this, props);
    this.grade = props.grade || 1;
}
var person1 = new Person("wanger")
Student.prototype = person1 ;
Student.prototype.constructor = Student;
```

## 四、用以上原型链继承带来的问题

1. 如果在控制台执行一遍上述的代码，我们会发现一些问题，如图所示：

![image](http://ow2n75eab.bkt.clouddn.com/prototype_proto_6.jpg)

Student.prototype上含有之前person1带有的属性，那么，这样的继承的方法就显得不那么完美了

2. 这个时候，我们可以借助一个中间对象来实现正确的原型链，这个中间对象的原型要指向Person.prototype。为了实现这一点，参考道爷（就是发明JSON的那个道格拉斯）的代码，中间对象可以用一个空函数F来实现：

```
var Person = function(msg){
    this.msg = msg;
}
var Student = function(props) {
    // 调用Person构造函数，绑定this变量:
    Person.call(this, props);
    this.grade = props.grade || 1;
}

// 空函数F:
function F() {
}

// 把F的原型指向Person.prototype:
F.prototype = Person.prototype;

// 把Student的原型指向一个新的F对象，F对象的原型正好指向Person.prototype:
Student.prototype = new F();

// 把Student原型的构造函数修复为Student:
Student.prototype.constructor = Student;

// 继续在Student原型（就是new F()对象）上定义方法：
Student.prototype.getGrade = function () {
    return this.grade;
};

// 创建wanger:
var wanger = new Student({
    name: '王二',
    grade: 9
});
wanger.msg; // '王二'
wanger.grade; // 9

// 验证原型:
wanger.__proto__ === Student.prototype; // true
wanger.__proto__.__proto__ === Person.prototype; // true

// 验证继承关系:
wanger instanceof Student; // true
wanger instanceof Person; // true
```

这其中主要用到了一个空函数F作为过桥函数。为什么道爷会用过桥函数？用过桥函数F(){}主要是为了清空构造的属性。如果有些原Person的构造用不到，那么过桥函数将是一个好的解决方案

这样写的话，Student.prototype上就没有任何自带的私有属性，这是理想的继承的方法

3. 如果把继承这个动作用一个inherits()函数封装起来，还可以隐藏F的定义，并简化代码：

```
function inherits(Child, Parent) {
    var F = function () {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
}
```

封装后，写起来就像这样：

```
var Person = function(msg){
    this.msg = msg;
}
var Student = function(props) {
    // 调用Person构造函数，绑定this变量:
    Person.call(this, props);
    this.grade = props.grade || 1;
}
inherits(Student,Person) ;
```

这样再一封装的话，代码就很完美了。

事实上，我们也可以在inherits中使用Object.create()来进行操作，代码如下：

```
function inherits(Child, Parent) {
    Child.prototype = Object.create(Parent.prototype);
    Child.prototype.constructor = Child;
}
```

如果有兴趣了解Object.create()的其他用法，可以参考我的这篇博客JS中Object.create的使用方法;

## 五、ES6的新关键字class

在ES6中，新的关键字class，extends被正式被引入，它采用的类似java的继承写法，写起来就像这样：

```
class Student extends Person {
    constructor(name, grade) {
        super(msg); // 记得用super调用父类的构造方法!
        this.grade = grade || 1;
    }
    myGrade() {
        alert('I am at grade ' + this.grade);
    }
}
```

这样写的话会更通俗易懂，继承也相当方便。读者可以进入廖雪峰的官方网站详细了解class的用法