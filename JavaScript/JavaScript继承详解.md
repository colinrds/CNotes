[toc]

本篇结合原型链详细介绍一下JavaScript的继承。

通常除非小应用，那像JavaScript继承一文中那样直接写写代码就行了。如果是大型应用或者库函数，对于继承这种稍显复杂的代码结构，通常会封装成一个inherit函数。例如：
```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}
function Child(n) {}    //空白的子构造函数

inherit(Child, Parent); //继承
```
现在我们来实现inherit。

## 模式一：默认模式，将原型对象指向父对象

```
function inherit(Child, Parent) {
    Child.prototype = new Parent();    //原型对象指向父对象
}
```
例子：

```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}
function Child(n) {}     //空白的子构造函数

function inherit(Child, Parent) {
    Child.prototype = new Parent(); //原型对象指向父对象
}

inherit(Child, Parent); //继承

var c1 = new Child("Jack");
console.log(c1.name);   //Adam
c1.say();               //Adam
```

原型链图：

![image](http://olphpb1zg.bkt.clouddn.com/Klass1.png)

见上面的结果为Adam。这就是该模式的缺点之一，即无法将子构造函数的参数给父构造函数。这个缺点很致命，因此通常我们不用该模式。即使你能保证父子构造函数都不需要参数，那从结果上看是OK的，但效率是低下的，例如你再创建一个子对象：

```
var c2 = new Child("Betty");
console.log(c2.name);   //Adam
c2.say();               //Adam
```

两个子对象c1和c2，都分别新建了一个父对象，因此存在两个父对象。这是该模式的缺点之二，即每个子对象都会重复地创建父对象，效率不高。

## 模式二：借用构造函数

该方法解决了模式一中无法通过子构造函数传递参数给父构造函数的问题：

```
function Child(a, b, c, d) {        //子构造函数
    Parent.apply(this, arguments);    //借用父构造函数
}
```
例子：

```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}

function Child(n) {     //子构造函数
    Parent.apply(this, arguments);    //借用父构造函数
}

var c2 = new Child("Patrick");
console.log(c2.name);   //Patrick
c2.say();               //error,未定义
```

结果看出Child的参数顺利传入了，但say方法会报未定义的错。原因就是该模式并没有将prototype指向Parent，只不过借用了一下Parent的实现。因此看似是继承，其实不然，从原型链角度来看，两者毫无关系。Child的实例对象里自然就没有Parent原型中的say方法。图示如下：

![image](http://olphpb1zg.bkt.clouddn.com/Klass2.png)

总结一下该模式：子类只是借用了父类构造函数的实现，从结果上看，获得了一个父对象的副本。但子类对象和父类对象是完全独立的，不存在修改子类对象的属性值影响父对象的风险。缺点是该模式某种意义上讲，其实不是继承，无法从父类的prototype中获得任何东西

## 模式三：借用和设置原型

本模式是上面两个模式的结合体，借鉴了上面两种模式的特点：

```
function Child(a, b, c, d) {          //子构造函数
    Parent.apply(this, arguments);    //参照模式二，借用父构造函数
}
Child.prototype = new Parent();        //参照模式一，将原型对象指向父对象
```

这就是JavaScript继承一文中推荐的继承模式。子对象既可获得父对象本身的成员副本，又能获得原型的引用。子对象能传参数给父构造函数，也能安全地修改自身属性。

例子：

```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}

function Child(name) {   //子构造函数
    Parent.apply(this, arguments);
}
Child.prototype = new Parent();

var c4 = new Child("Patrick");
console.log(c4.name);    //Patrick
console.log(c4.say());   //Patrick
delete c4.name;
console.log(c4.say());   //Adam
```

![image](http://olphpb1zg.bkt.clouddn.com/Klass3.png)

该模式通常用用就可以了，但不是完美的。缺点和模式二的缺点二一样，多个子对象都会重复地创建父对象，效率不高。另外从例子的结果和图中都可以看出，有两个name属性，一个在父对象中，一个在子对象中。你delete子对象中的name后，父对象的name会显现出来，这可能会出bug。而且对效率狂来说，冗余的属性会看着不舒服。

## 模式四：共享原型

为了克服模式三需要重复创建父对象的缺点，该模式不调用构造函数，即任何需要继承的成员都放到原型里，而不是放置在父构造函数的this中。等价于对象共享一个原型

```
function inherit(Child, Parent) {
    Child.prototype = Parent.prototype;
}
```
例子：

```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}
function Child(n) {
    this.name = n;
}

function inherit(Child, Parent) {
    Child.prototype = Parent.prototype;
}

inherit(Child, Parent); //继承

var c4 = new Child("Patrick");
console.log(c4.name);         //Patrick
console.log(c4.say());        //Patrick
delete c4.name;
console.log(c4.say());        //undefined
```

从结果可以看出，该模式和模式三不同，现在你delete子对象的name属性，就不会将父对象的name属性显现出来了。原型链图：

![image](http://olphpb1zg.bkt.clouddn.com/Klass4.png)

该模式除了需要你仔细斟酌哪些属性和方法需要被继承，抽出来放到父类原型里。而且由于父子对象共享原型，因此双方修改时都要小心，如果子对象不小心修改了原型里的属性和方法，会影响到父对象，反之亦然。例如：

```
Child.prototype.setName = function(n) {
    return this.name = n;
}
c4.setName("Jack");
console.log(c4.name);     //Jack
console.log(c4.say());    //Jack

var c5 = new Parent();
c5.setName("Betty");
console.log(c5.name);     //Betty
console.log(c5.say());    //Betty
```

给子类原型增加一个setName方法。由于父子类共享原型，因此父类对象也自动获得了setName方法。

## 模式五：临时构造函数

为解决模式四中父子对象间耦合度较高的缺点，该模式断开父子对象间的原型的直接链接关系，但同时还能继续受益于原型链的好处

```
function inherit(Child, Parent) {
    var F = function() {};        //空的临时构造函数
    F.prototype = Parent.prototype;
    Child.prototype = new F();
}
```

例子：

```
function Parent(n) {     //父构造函数
    this.name = n || 'Adam';
}
Parent.prototype.say = function() {
    return this.name;
}
function Child(n) {
    this.name = n;
}

function inherit(Child, Parent) {
    var F = function() {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
}

inherit(Child, Parent); //继承

var c6 = new Child("Patrick");
console.log(c6.name);     //Patrick
console.log(c6.say());    //Patrick
delete c6.name;
console.log(c6.say());    //undefined
```

原型链图：

![image](http://olphpb1zg.bkt.clouddn.com/Klass5.png)

与模式四的差别就是，新定义了个空的临时构造函数F()，子类的原型指向该临时构造函数。这样修改子类原型时，实际修改的是修改到了临时构造函数F()，不会影响父类：

```
Child.prototype.setName = function(n) {
    return this.name = n;
}
c6.setName("Jack");
console.log(c6.name);       //Jack
console.log(c6.say());      //Jack

var c7 = new Parent();
c7.setName("Betty");        //error,未定义
```

上面的例子和模式四中相同，但结果不同，子类原型上添加的新方法setName，父类对象无法访问。

该模式非常好，即有效率，还能实现父子解耦。本着精益求精的精神，再为该模式增加三个加分项：

加分项一：添加一个指向父类原型的引用，例如其他语言里的super：

```
function inherit(Child, Parent) {
    var F = function() {};         //空的临时构造函数
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.uber = Parent.prototype; //uber表示super，因为super是保留的关键字
}
```
这样如果你为子类原型添加setName方法后，希望父类对象也能获得该方法，可以：

```
function inherit(Child, Parent) {
    var F = function() {};        //空的临时构造函数
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.uber = Parent.prototype;    //uber表示super，因为super是保留的关键字
}

inherit(Child, Parent); //继承

Child.prototype.setName1 = function(n) {
    return this.name = n;
}
Child.uber.setName2 = function(n) {
    return this.name = n;
}

var c8 = new Child("Patrick");
c8.setName1("Jack");
console.log(c8.name);     //Jack
console.log(c8.say());    //Jack
c8.setName2("Betty");
console.log(c8.name);     //Betty
console.log(c8.say());    //Betty

var c9 = new Parent();
c9.setName1("Andy");      //error，未定义
c9.setName2("Andy");
console.log(c9.name);     //Andy
console.log(c9.say());    //Andy
```

子类给原型的新增方法setName1不会影响父类，父类对象无法使用setName1。但父类对象可以使用子类通过uber给原型的新增方法setName2。

加分项二：重置该构造函数的指针，以免在将来某个时候还需要该构造函数。如果不重置构造函数的指针，那么所有子对象会报告Parent()是它们的构造函数，这没有任何用处：

```
var c10 = new Child();
console.log(c10.constructor.name);          //Parent
console.log(c10.constructor === Parent);    //true
```

虽然我们很少用constructor属性，不改也不影响实际的使用，但作为完美主义者还是改一下吧：

```
function inherit(Child, Parent) {
    var F = function() {};            //空的临时构造函数
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.uber = Parent.prototype;    //uber表示super，因为super是保留的关键字
    Child.prototype.constructor = Child;    //修正constructor属性
}

inherit(Child, Parent); //继承

var c11 = new Child();
console.log(c11.constructor.name);          //Child
console.log(c11.constructor === Parent);    //false
```

加分项三：临时构造函数F()不必每次继承时都创建，仅创建一次以提高效率：

```
var inherit = (function() {
    var F = function() {};
    return function(Child, Parent) {
        F.prototype = Parent.prototype;
        Child.prototype = new F();
        Child.uber = Parent.prototype;
        Child.prototype.constructor = Child;
    }
}());
```

## 模式五，你可以在开源的YUI库，或其他库中看到类似模式五的身影。例如Klass

- Klass

Klass是一种代码结构，模拟传统OO语言的Class。继承时能像传统OO语言的Class一样，子类构造函数调用父类的构造函数。作为一种代码结构，它有一套命名公约，如initialize，_init等，创建对象时这些方法会被自动调用。

例如：

```
var klass = function (Parent, props) {
    var Child, F, i;

    //1.新构造函数
    Child = function (Parent, props) {
        if(Child.uber && Child.uber.hasOwnProperty("__construct")) {
            Child.uber.__construct.apply(this, arguments);
        }
        if(Child.prototype.hasOwnProperty("__construct")) {
            Child.prototype.__construct.apply(this, arguments);
        }    
    };    

    //2.继承
    Parent = Parent || Object;
    F = function() {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.uber = Parent.prototype;
    Child.prototype.constructor = Child;

    //3.添加实现方法
    for(i in props) {
        if(props.hasOwnProperty(i)) {
            Child.prototype[i] = props[i];
        }
    }

    return Child;
};
```
看一下上面的Klass代码结构。它有两个参数，分别是父类和子类需要扩展的字面量形式的属性。

第一部分是为子类生成构造函数：如果父类存在构造函数，先调用父类构造函数。如果子类存在构造函数，再调用子类构造函数。（由于PHP的影响，一个潜规则是，类的构造函数最好命名为__construct）。在最后return出生成的构造函数

第二部分是继承，参照模式五，不赘述。

第三部分是为子类添加需要扩展的属性。

现在我们的代码中就可以不再纠结于用哪种模式来实现继承了，直接用Klass。

例如创建一个不继承自任何类的新类：

```
var Person = klass(null, {
    __construct: function (n) {
        this.name = n;
    },
    sayHi: function() {
        console.log("hi " + this.name);
    }
});

var p1 = new Person("Jack");
p1.sayHi();    //hi Jack
```

上面代码用klass创建了一个Person的新类，没有继承自任何类，意味着继承Object类（源码中的Parent = Parent || Object;语句）。构造函数里创建name属性，并提供了一个sayHi的方法

现在扩充一个Man类：

```
var Man = klass(Person, {
    __construct: function (n) {
        console.log("I am a man.");
    },
    sayHi: function() {
        Man.uber.sayHi.call(this);
    }
});
var m1 = new Man("JackZhang");    //I am a man.
m1.sayHi();                       //hi JackZhang
console.log(m1 instanceof Person);    //true
console.log(m1 instanceof Man);       //true
```

用库的klass的话，虽然比较方便，让JavaScript无比接近传统OO语言，让新手也能快速上手进行开发。但其实不建议用klass，因为让容易让人产生一种JavaScript也有类的错觉，其实它只是一种模拟类的代码结构，如果你对JavaScript的原型链不害怕的话，还是避免用klass比较好

- 原型继承

上面介绍的五种模式和klass都属于基于类型的继承，在JavaScript继承一文中还介绍了用Object.create()基于对象的继承，也叫原型继承。用法很简单，这里看一下它的本质。

原型继承不涉及类，对象都是继承自其它对象，即要继承的话，先要有一个父类对象：

```
function create(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

var parent = {
    name: "Papa"
};
var child = create(parent);
console.log(child.name);    //Papa
```

原型链图：


也不是必须使用字面量来创建父对象（虽然用字面量比较常见），也可以用构造函数来创建父对象，这样的话，自身的属性和构造函数的原型的属性都将被继承：

```
function Parent() {
    this.name = "Papa";
}
Parent.prototype.getName = function() {
    return this.name;
};

var papa = new Parent();
var child = create(papa);
console.log(child.name);         //Papa
console.log(child.getName());    //Papa
```
不论用字面量还是构造函数方式创建父对象都可以，甚至你可以只继承父类的原型对象：

```
var child = create(Parent.prototype);
console.log(typeof child.name);       //undefined
console.log(typeof child.getName);    //function
```

ES5里定义成Object.create()，基于对象的继承我们直接用该方法就行了。

## 总结

如果基于对象继承用Object.create()。如果基于类型继承，平时一些快速应用，或小应用，用模式三实现继承就够了。复杂应用或大型程序用模式五。如果你做代码库，可以用模式五定义inherit函数，或定义Klass。
