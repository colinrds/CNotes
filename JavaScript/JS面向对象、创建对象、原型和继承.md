[toc]

## 1.创建对象

### （1）两个基本方法
创建对象最基本的两个方法是：Object构造函数和对象字面量。

```
//Object构造函数方式
var person = new Object();
person.name = "Jack";
person.age = 12;
person.sayName = function() {
    alert(this.name);
};

//字面量方式
var person = {
    name: "Jack",
    age: 14,
    job: "码农",
    sayName: function() {
        alert(this.name);
    }
};
```

### （2）工厂模式
上述两个基本方法的缺点是：使用同一个接口创建很多对象，会产生大量的复制代码。针对这个缺点，看下面原理是用函数来封装以特定接口创建对象的细节

```
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person1 = createPerson("Jack", 15, "码农");
var person2 = createPerson("rose", 12, "程序媛");
```

函数createPerson能接收参数构建一个包含所有属性的对象，并且可以用很少的代码不断的创建多个对象，但是由于它被函数所封装，暴露的接口不能有效的识别对象的类型（即你不知道是Object还是自定义的什么对象）。

### （3）构造函数模式

```
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        alert(this.name);
    };
}
var person1 = new Person("Jack", 15, "码农"); //满满的java复古风
var person2 = new Person("Rose", 15, "程序媛");
```

与工厂模式相比，构造函数模式用Person()函数代替了createPerson()函数，并且没有显示的创建对象，直接把属性和方法赋值给了this对象。
要创建Person的实例，必须使用new关键字。
person1和person2都是Person的实例，这两个对象都有一个constructor（构造函数）属性，该属性指向Person。 person1.constructor == Person; //true
person1即是Person的实例又是Object的实例，后面继承原型链会总结。
### （4）构造函数的使用

```
//当做构造函数使用
var person1 = new Person("Jack", 15, "码农");
person1.sayName(); //"Jack"
//当做普通函数使用
Person("Jack", 16, "码农"); //注意：此处添加到了window
window.sayName(); //"Jack"
//在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "Jack", 12, "码农");
o.sayName(); //"Jack"
```

第一种当做构造函数使用就不多说了
当在全局作用域中调用Person("Jack",16,"码农");时，this对象总是指向Global对象（浏览器中是window对象）。因此在执行完这句代码后，可以通过window对象来调用sayName()方法，并且返回“Jack”。
最后也可以使用call()或者apply()在某个特殊对象的作用域中调用Person()函数

### （5）存在的问题
在(3)构造函数模式的代码中，对象的方法sayName的功能都一样，就是alert当前对象的name。当实例化Person之后，每个实例（person1和person2）都有一个名为sayName的方法，但是两个方法不是同一个Function实例。不要忘了，js中函数是对象，所以每个实例都包含一个不同的Function实例，然而创建两个功能完全一样的Function实例是完全没有必要的。因此可以把函数定义转移到构造函数外。
如下代码：

```
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName；
}
function sayName() {
    alert(this.name);
}

//实例化对象
var person1 = new Person("Jack", 15, "码农"); //满满的java复古风
var person2 = new Person("Rose", 15, "程序媛");
```

但是这样依然存在问题：

为了让Person的实例化对象共享在全局作用域中定义的同一个sayName()函数，我们把函数sayName()定义在全局作用域中，并通过指针sayName指向构造函数，所以在全局作用域中的sayName()只能被特定对象调用，全局作用域名不符实，且污染全局变量。
并且如果对象需要很多种方法，那么就要定义很多全局函数，对于对象就没有封装性，并且污染全局。

## 2.原型

### （1）原型模式创建对象
js不同于强类型语言的java，java创建对象的过程是由类（抽象）到类的实例的过程，是一个从抽象到具体的过程。
javascript则不同，其用原型创建对象是一个具体到具体的过程，即以一个实际的实例为蓝本（原型），去创建另一个实例对象。
所以用原型模式创建对象有两种方式：

1.Object.create()方法
Object.create:它接收两个参数，第一个是用作新对象原型的对象（即原型），一个是为新对象定义额外属性的对象（可选，不常用）。

```
var Person = {
    name: "Jack",
    job: "码农"
};
//传递一个参数
var anotherPerson = Object.create(Person);
anotherPerson.name //"Jack"
//传递两个参数
var yetPerson = Object.create(Person, {
    name: {
        value: "Rose"
    }
});
yetPerson.name; //Rose

```
2.构造函数方法创建对象

任何一个函数都有一个prototype属性（是一个指针），指向通过构造函数创建的实例对象的原型对象，原型对象可以让所有对象实例共享它所包含的属性和方法。
因此不必在构造函数中定义对象实例的信息，而是将这些属性和方法直接添加到原型对象中，从而被实例对象多继承（继承后面总结）

```
//第一步：用构造函数创建一个空对象
function Person() {}
//第二步：给原型对象设置属性和方法
Person.prototype.name = "Jack";
Person.prototype.age = 20;
Person.prototype.job = "码农";
Person.prototype.sayName = function() {
    alert(this.name);
};
//第三步：实例化对象后，便可继承原型对象的方法和属性
var person1 = new Person();
person1.sayName(); //Jack
var person2 = new Person();
person2.sayName(); //Jack
alert(person1.sayName == person2.sayName); //true
```

person1和person2说访问的是同一组属性和同一个sayName()函数。

### （2）理解原型对象
只要创建一个函数，就会为该函数创建一个prototype属性，这个属性指向函数的原型对象。
所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含一个指向prototype属性所在函数的指针。
当调用构造函数创建一个新的实例对象后，该实例内部会有一个指针（`［prototype］/_proto_`）,指向构造函数的原型对象。如下图：

![image](http://olphpb1zg.bkt.clouddn.com/2553832159-5781bfa8367db_articlex.jpeg)

上图中 ：

Person.prototype指向了原型对象，而Person.prototype.construstor又指回了Person。
注意观察原型对象，除了包含constructor属性之外，还包括后来添加的其它属性，这就是为什么每个实例化后的对象，虽然都不包含属性和方法，但是都包含一个内部属性指向了Person.prototype,能获得原型中的属性和方法。

### （3）判断一个实例对象的原型
这个方法叫：Object.getPrototypeOf(),如下例子：

```
alert(Object.getPrototypeOf(person1) == Person.prototype); //true
alert(Object.getPrototypeOf(person1).name); //"Jack"
```

这个方法可以很方便的取得一个对象的原型
还可以利用这个方法取得原型对象中的name属性的值。

### （4）搜索属性的过程
当我们在创建实例化的对象之后，调用这个实例化的对象的属性时，会先后执行两次搜索。
第一次搜索实例person1有name属性吗？没有进行第二次搜索
第二次搜索person1的原型有name属性吗？有就返回。
因此进行一次思考，如果对实例进行属性重写和方法覆盖之后，访问实例对象的属性和方法会显示哪个？实例对象的还是对象原型的？

```
function Person() {}

Person.prototype.name = "Jack";
Person.prototype.age = 20;
Person.prototype.job = "码农";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

person1.name = "Rose";
alert(person1.name); //Rose
alert(person2.name); //Jack

```

当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性。
但是这个属性只会阻止我们访问原型中的那个属性，而不会修改那个属性3.使用delete操作符可以删除实例属性，从而重新访问原型中的属性。
```
function Person() {}

Person.prototype.name = "Jack";
Person.prototype.age = 20;
Person.prototype.job = "码农";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

person1.name = "Rose";
alert(person1.name); //Rose  －－来自实例
alert(person2.name); //Jack  －－来自原型
delete person1.name;
alert(person1.name); //Jack  --来自原型
```
    
### （5）判断访问的到底是对象还是原型属性
hasOwnProperty()可以检测一个属性是存在于实例中，还是原型中，只有在给定属性存在于对象实例中，才会返回true。
```
person1.hasOwnProperty("name");    //假设name存在于原型，返回false
```

in操作符会在通过对象能够访问给定属性时返回true，无论该属性是存在于实例中还是原型中
```
"name" in person1   //true
```
所以通过这两个可以封装一个hasPrototypeProperty()函数确定属性是不是原型中的属性。
```
function hasPrototypeProperty(object,name){
    return !object.hasOwnProperty(name) && (name in object); 
}
```

### （6）更简单的原型语法
前面每次添加一个属性和方法都要写一次Person.prototype，为了简便可以直接这样

```
function Person() {}
Person.prototype = {
    name: "Jack",
    age: 20,
    job: "码农",
    sayName: function() {
        alert("this.name");
    }
};
```

上述代码直接将Person.prototype设置为等于一个以对象字面量形式创建的新对象
上述这么做时：constructor属性就不再指向Person了。
本质上完全重写了默认的prototype对象，因此constructor属性也就变成了新对象的constructor属性（指向Object构造函数）。
因此如果constructor值很重要，可以在Person.prototype中设置回适当的值：
如上例中可以添加：constructor:Person,

### （7）原型的动态性
我们对原型对象所做的任何修改都会立即从实例上反映出来－即使先创建实例对象后修改原型也如此

```
var friend = new Person();
Person.prototype.sayHi = function() {
    alert("Hi");
};
friend.sayHi(); //"Hi"
```

尽管可以随时为原型添加属性和方法，并且修改能立即在实例对象中体现出来，但是如果重写整个原型对象，就不一样了。看下面例子：

```
function Person() {}
var friend = new Person();
Person.prototype = {
    constructor: Person,
    name: "Jack",
    age: 20,
    sayName: function() {
        alert(this.name);
    }
};
friend.sayName(); //error
```

上述代码先创建了一个Person实例，然后又重写了其原型对象，在调用friend.sayName()时发生错误。
因为friend指向的原型中不包含以该名字命名的属性。关系如下图：

### （8）原型对象的问题
省略了为构造函数初始化参数这一环节，结果是所有实例都取得相同的属性，但问题不大，可以为实例对象重写属性来解决。
2.但是，对于包含引用类型值的属性来说，问题就比较突出了，因为引用类型中，属性名只是一个指针，在实例中重写该属性并没有作用。指针始终指向原来的。
如下例子：

```
function Person() {}

Person.prototype = {
    constructor: Person,
    name: "Jack",
    job: "码农",
    friends: ["路人甲", "路人乙", "路人丙"],

};
var person1 = new Person();
var person2 = new Person();

person1.friends.push("路人丁");
alert(person1.friends); //["路人甲","路人乙","路人丙","路人丁"]
alert(person2.friends); //["路人甲","路人乙","路人丙","路人丁"]
alert(person1.friends === person2.friends); //true

```

上面这个，假如每个实例对象的引用值属性不一样，则无法修改。

## 3.组合使用构造函数和原型模式
构造函数模式用于定义实例属性
原型模式用于定义方法和共享的属性
如下代码：

```
function Person(name, age, job) {
    this.name = name;
    this.job = job;
    this.age = age;
    this.friends = ["路人甲", "路人乙"];
}

Person.prototype = {
    constructor: Person,
    sayName: function() {
        alert(this.name);
    }
}
var person1 = new Person("Jack", 20, "码农");
var person2 = new Person("Rose", 20, "程序媛");

person1.friends.push("路人丁");
alert(person1.friends); //["路人甲","路人乙","路人丁"]
alert(person2.friends); //["路人甲","路人乙"]
alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true
```

## 4.寄生构造函数模式
该模式基本思想是创建一个函数，该函数作用仅仅是封装创建对象的代码，然后返回新创建的对象。

```
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}
var friend = new Person("Jack", 16, "码农");
friend.sayName(); //Jack
```

构造函数在不返回值的情况下，默认会返回新对象实例。
通过在构造函数末尾添加一个return语句，可以重写调用构造函数时返回的值。
这个方法的用处是：可以创建一个额外方法的特殊的数组（因为原生对象Array的构造函数不能直接修改）

```
function SpecialArray() {
    //创建数组
    var values = new Array();
    //添加值
    values.push.apply(values, arguments);

    //添加方法
    values.toPipedString = function() {
        return this.join("|");
    };
    //返回数组
    return values;
}
var colors = new SpecialArray("black", "red", "blue");
alert(colors.toPipedString());
```


## 5.原型链

js中原型链是实现继承的主要方法。基本思想是：利用原型让一个引用类型继承另一个引用类型的属性和方法。我们来简单回顾一下以前的内容：

1. 每个构造函数都有一个原型对象
2. 每个原型对象都包含一个指向构造函数的指针：（constructor）
3. 而实例和构造函数都有一个prototype属性指针指向原型对象。
4. 假如现在我们让原型对象（A）等于另一个类型的实例（b），此时相当于这个原型对象（A）整体作为一个实例指向另一个实例的原型对象（b的原型对象B）。
5. 以上就实现了继承。

看下面代码实例：

```
function SuperType() {
    this.property = true; //property是SuperType的实例属性
};

SuperType.prototype.getSuperValue = function() { //getSuperValue是SuperType的原型方法
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//让SuperType继承SubType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue()); //true
```

1. 在上面的代码中，定义了两个类型：SuperType和SubType。每个类型分别有一个属性和方法。
2. 通过创建SuperType的实例，并赋值给了SubType.prototype,从而实现SubType继承了这个的实例
3. 原来存在于SuperType的实例中的所有的属性和方法，现在也存在于SubType.prototype中了。
4. 既然现在SubType的原型对象SubType.prototype是SuperType的实例化对象，那么SuperType的实例属性property就位于SubType.prototype。如下图：
5. 现在instance.constructor现在指向的是SuperType，图中可以看出来。也可以在进行继承之后，再进行如下步骤：SubType.prototype.constructor = Subtype;

![image](http://olphpb1zg.bkt.clouddn.com/3602824876-5783a4fb897aa_articlex.jpeg)

## 6.完整原型链

所有函数的默认原型都是Object的实例，所以下图是上面例子的完整原型链。

![image](http://olphpb1zg.bkt.clouddn.com/1400843372-5783a7d144825_articlex.jpeg)

## 7.重写或添加方法到超类型

（1）重写和添加方法必须在用超类型的实例（new SuperType()）替换原型（SubType.prototype）之后。

```
function SuperType() {
    this.property = true;
};

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//让SuperType继承SubType
SubType.prototype = new SuperType();
//添加新方法
SubType.prototype.getSubValue = function() {
    return this.subproperty;
};
//重写超类型中的方法
SubType.prototype.getSuperValue = function() {
    return false;
};

var instance = new SubType();
alert(instance.getSuperValue()); //false
alert((new SuperType()).getSuperValue()); //我仿照java这么写，居然返回true
```

1. 重写超类型中的方法之后，通过SuperType的实例调用getSuperValue()时，调用的就是这个重新定义的方法。
2. 通过SuperType的实例调用getSuperValue()时，调用的就是超类型中的方法，返回true

（2）通过原型链实现继承时，不能使用对象字面量创建原型方法，这样会重写原型链

```
function SuperType() {
    this.property = true;
};

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//让SuperType继承SubType
SubType.prototype = new SuperType();

SubType.prototype = {
    getSubValue: function() {
        return this.subproperty;
    },
    someOtherMethod: function() {
        return false;
    }
};

var instance = new SubType();
alert(instance.getSuperValue()); //error
```

(3)原型链的问题

1. 在通过原型链进行继承时，原型实际上会变成另一个类型的实例，所以原先的实例属性也就变成了现在的原型属性了。
2. 现在假如原型实例的属性是引用类型的，那么它会直接被添加成现在的对象原型的属性，那么通过这个创建的实例对这个引用类型的属性进行更改时，会立即反映在所有的实例对象上。

看下面代码：
```
function SuperType() {
    this.colors = ["red", "blue", "green"];
};

function SubType() {}

//让SubType继承SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors); //["red","blue","green","black"]
var instance2 = new SubType();
alert(instance2.colors); //["red","blue","green","black"]
```

1. 当SubType通过原型链继承了SuperType之后，SubType.prototype就变成了SuperType的一个实例
2. 此时SubType拥有一个自己的colors属性，就像专门创建了一个SubType.prototype.colors属性一样此时
3. SubType所有的实例话对象都会共享这个colors属性，修改instances1的colors属性会立即在instances2中显示出来。

原型链还有一个问题：在创建子类型的实例时，不能向超类型的构造函数传递参数，实际上是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

## 8.实现继承的其它方法

### （1）借用构造函数

基本思想：

在子类型构造函数的内部调用超类型的构造函数，通过使用call()方法或者apply()方法。

例子：
```
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

function SubType(name, age) {
    //继承了SuperType,同时还传递了参数
    SuperType.call(this, name);
    //再为子类型定义属性
    this.age = age;
}

var instance1 = new SubType("Jack");
alert(instance1.name);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
var instance2 = new SubType();
alert(instance2.colors); //"red,blue,green"
```

上述代码中解决了一个问题，就是引用类型的属性问题，每个实例化的子类型都有自己的特有的属性还存在一个问题，如果方法都定义在构造函数中，那么方法的就不能复用。

### （2）组合继承－最常用的继承模式

组合继承的思路是：

1. 使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对实例属性的继承
2. 这样既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。

例子：
```
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    //继承SuperType的属性
    SuperType.call(this, name);
    this.age = age;
}

//继承SuperType的方法
SubType.prototype = new SuperType();
//定义子类型自己的方法
SubType.prototype.sayAge = function() {
    alert(this.age);
};

var instance1 = new SubType("Jack", 26);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //Jack
instance1.sayAge(); //26
var instance2 = new SubType("Rose", 23);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //Rose
instance2.sayAge(); //23
```

### （3）原型式继承

思路：借助原型可以基于已有的对象创建新对象，还不必因此创建自己的自定义类型如下：
```
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}
```

1. 在object()函数内部先创建一个临时性的函数。
2. 然后将传入的对象作为这个构造函数的原型。
3. 最后返回这个临时类型的饿新实例。

如下：
```
var person = {
    name: "Jack",
    friends: ["路人甲", "路人乙"]
};
var anotherPerson = object(person); //此处调用上方的object方法
anotherPerson.name = "Rose";
anotherPerson.friends.push("路人丙");

var yetPerson = object(person);
yetPerson.name = "Rick";
yetPerson.friends.push("路人丁");

alert(person.friends); //["路人甲","路人乙","路人丙","路人丁"]
```

上述person.friends不仅属于person所有，而且会被anotherPerson和yetPerson共享。还有Object.create()方法，前面已经总结过了。

### （4）寄生式继承

思路：创建一个仅用于封装继承过程的函数。

```
function createAnother(original) {
    var clone = object(original); //调用前面的object()方法
    clone.sayHi = function() {
        alert("hi");
    };
    return clone;
}

//使用
var person = {
    name: "Jack",
    friends: ["路人甲", "路人乙", "路人丙"]
};
var anotherPerson = createAnother(person);
anotherPerson.sayHi(); //"Hi"
```

## 9.寄生组合式继承

### （1）组合继承存在的问题

组合继承是js最常用的继承模式，不过它有自己的不足，组合继承最大的问题在于要调用两次超类型的构造函数，一次是创建超类型的实例赋值给子类型的原型对象时，一次是子类型构造函数内部。

最终子类型会包含超类型对象的全部实例属性，但是我们不得不在调用子类型构造函数时重写这些属性。

看下面例子：
```
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name); //第二次调用
    this.age = age;
}

SubType.prototype = new SuperType(); //第一次调用
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
    alert(this.age);
};
```

1. 第一次调用SuperType构造函数时，SubType.prototype会得到两个属性：name和colors,它们都是SuperType的实例属性，只不过位于SubType的原型中。
2. 当调用SubType的构造函数时，在函数内部又会调用SuperType的构造函数，又在新对象上创建了实例属性name和colors，于是这两个属性就屏蔽了原型中的同名属性。

### （2）解决方法

寄生组合式继承的思想是：不必为了子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型的一个副本而已，本质上就是使用寄生式继承来继承超类型的原型，把返回的结果赋值给子类型的原型。

大家一定还记得上面说的原型式继承吧吧，将一个对象浅赋值给另一个对象，现在也可以把一个超类型的原型赋值给另一个子类型原型

1.回忆一下object()函数的代码
```
function object(o) {
    function F() {}
    F.prototype = 0;
    return new F();
}
```

2.创建一个函数,它接收两个参数：子类型构造函数和超类型构造函数。
```
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}
```

1. 上面的代码第一步创建超类型原型的一个副本
2. 为创建的副本添加constructor属性，弥补因重写原型而失去默认的constructor属性此处的重写发生在object()函数里面，超类型的原型superType.prototype直接赋给了F.prototype,然后object()函数又返回了F的新实例。
3. 把创建新的对象赋值给子类型的原型

3.那么现在来使用一下
```
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
    alert(this.name);
};
function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}
inheritPrototype(subType, SuperType);
SubType.prototype.sayAge = function() {
    alert(this.age);
};
```
上述代码高效率，因为它只调用了一次SuperType的构造函数，因此避免了在SubType.prototype上面创建不必要的、多余的属性，此时原型链还能保持不变。