[toc]

## 一：理解构造函数原型(prototype)机制
    
prototype是javascript实现与管理继承的一种机制，也是面向对象的设计思想.构造函数的原型存储着引用对象的一个指针，该指针指向与一个原型对象，对象内部存储着函数的原始属性和方法；我们可以借助prototype属性，可以访问原型内部的属性和方法。

当构造函数被实列化后，所有的实例对象都可以访问构造函数的原型成员，如果在原型中声明一个成员，所有的实列方法都可以共享它，比如如下代码：

```
// 构造函数A 它的原型有一个getName方法
function A(name){
    this.name = name;
}
A.prototype.getName = function(){
    return this.name;
}
// 实列化2次后 该2个实列都有原型getName方法；如下代码
var instance1 = new A("longen1");
var instance2 = new A("longen2");
console.log(instance1.getName()); //longen1
console.log(instance2.getName()); // longen2
```

原型具有普通对象结构，可以将任何普通对象设置为原型对象; 一般情况下，对象都继承与Object,也可以理解Object是所有对象的超类,Object是没有原型的，而构造函数拥有原型，因此实列化的对象也是Object的实列，如下代码：

```
// 实列化对象是构造函数的实列
console.log(instance1 instanceof A); //true
console.log(instance2 instanceof A); // true

// 实列化对象也是Object的实列
console.log(instance1 instanceof Object); //true
console.log(instance2 instanceof Object); //true

//Object 对象是所有对象的超类，因此构造函数也是Object的实列
console.log(A instanceof Object); // true

// 但是实列化对象 不是Function对象的实列 如下代码
console.log(instance1 instanceof Function); // false
console.log(instance2 instanceof Function); // false

// 但是Object与Function有关系 如下代码说明
console.log(Function instanceof Object);  // true
console.log(Object instanceof Function);  // true
```

如上代码，Function是Object的实列，也可以是Object也是Function的实列;他们是2个不同的构造器，我们继续看如下代码:

```
var f = new Function();
var o = new Object();
console.log("------------"); 
console.log(f instanceof Function);  //true
console.log(o instanceof Function);  // false
console.log(f instanceof Object);    // true
console.log(o instanceof Object);   // true
```

我们明白，在原型上增加成员属性或者方法的话，它被所有的实列化对象所共享属性和方法，但是如果实列化对象有和原型相同的成员成员名字的话，那么它取到的成员是本实列化对象，如果本实列对象中没有的话，那么它会到原型中去查找该成员，如果原型找到就返回，否则的会返回undefined，如下代码测试

```
function B(){
    this.name = "longen2";
}
B.prototype.name = "AA";
B.prototype.getName = function(){
    return this.name;
};

var b1 = new B();
// 在本实列查找，找到就返回，否则到原型查找
console.log(b1.name); // longen2

// 在本实列没有找到该方法，就到原型去查找
console.log(b1.getName());//longen2

// 如果在本实列没有找到的话，到原型上查找也没有找到的话，就返回undefined
console.log(b1.a); // undefined

// 现在我使用delete运算符删除本地实列属性，那么取到的是就是原型属性了，如下代码：
delete b1.name;
console.log(b1.name); // AA
```

## 二：理解原型域链的概念
  
原型的优点是能够以对象结构为载体，创建大量的实列，这些实列能共享原型中的成员(属性和方法);同时也可以使用原型实现面向对象中的继承机制~ 如下代码：下面我们来看这个构造函数AA和构造函数BB，当BB.prototype = new AA(11);执行这个的时候，那么B就继承与A，B中的原型就有x的属性值为11

```
function AA(x){
    this.x = x;
}
function BB(x) {
    this.x = x;
}
BB.prototype = new AA(11);
console.log(BB.prototype.x); //11

// 我们再来理解原型继承和原型链的概念，代码如下，都有注释
function A(x) {
    this.x = x;
}
// 在A的原型上定义一个属性x = 0
A.prototype.x = 0;
function B(x) {
    this.x = x;
}
B.prototype = new A(1);
```

实列化A new A(1)的时候 在A函数内this.x =1， B.prototype = new A(1);B.prototype 是A的实列 也就是B继承于A， 即B.prototype.x = 1;  如下代码：

```
console.log(B.prototype.x); // 1
// 定义C的构造函数
function C(x) {
    this.x = x;
}
C.prototype = new B(2);
```

`C.prototype = new B(2)`; 也就是`C.prototype` 是B的实列，C继承于B；那么`new B(2)`的时候 在B的构造函数内 `this.x = 2`；那么 C的原型上会有一个属性`x =2` 即`C.prototype.x = 2`; 如下代码：

```
console.log(C.prototype.x); // 2
```

下面是实列化 `var d = new C(3)`; 实列化C的构造函数时候，那么在C的构造函数内`this.x = 3`; 因此如下打印实列化后的`d.x = 3`；如下代码：

```
var d = new C(3);
console.log(d.x); // 3
```

删除d.x 再访问d.x的时候 本实列对象被删掉，只能从原型上去查找；由于C.prototype = new B(2); 也就是C继承于B，因此C的原型也有x = 2；即C.prototype.x = 2; 如下代码：

```
delete d.x;
console.log(d.x);  //2
```

删除C.prototype.x后，我们从上面代码知道，C是继承于B的，自身的原型被删掉后，会去查找父元素的原型链，因此在B的原型上找到x =1; 如下代码：

```
delete C.prototype.x;
console.log(d.x);  // 1
```

当删除B的原型属性x后，由于B是继承于A的，因此会从父元素的原型链上查找A原型上是否有x的属性，如果有的话，就返回，否则看A是否有继承，没有继承的话，继续往Object上去查找，如果没有找到就返回undefined 因此当删除B的原型x后，delete B.prototype.x; 打印出A上的原型x=0; 如下代码：

```
delete B.prototype.x;
console.log(d.x);  // 0

// 继续删除A的原型x后 结果没有找到，就返回undefined了;
delete A.prototype.x;
console.log(d.x);  // undefined
```

在javascript中，一切都是对象，Function和Object都是函数的实列;构造函数的父原型指向于Function原型，Function.prototype的父原型指向与Object的原型，Object的父原型也指向与Function原型，Object.prototype是所有原型的顶层;

如下代码：

```
Function.prototype.a = function(){
    console.log("我是父原型Function");
}
Object.prototype.a = function(){
    console.log("我是 父原型Object");
}
function A(){
    this.a = "a";
}
A.prototype = {
    B: function(){
        console.log("b");
    }
}
// Function 和 Object都是函数的实列 如下：
console.log(A instanceof Function);  // true
console.log(A instanceof Object); // true

// A.prototype是一个对象，它是Object的实列，但不是Function的实列
console.log(A.prototype instanceof Function); // false
console.log(A.prototype instanceof Object); // true

// Function是Object的实列 同是Object也是Function的实列
console.log(Function instanceof Object);   // true
console.log(Object instanceof Function); // true

/*
 * Function.prototype是Object的实列 但是Object.prototype不是Function的实列
 * 说明Object.prototype是所有父原型的顶层
 */
console.log(Function.prototype instanceof Object);  //true
console.log(Object.prototype instanceof Function);  // false
```

## 三：理解原型继承机制

构造函数都有一个指针指向原型，Object.prototype是所有原型对象的顶层，比如如下代码：

```
var obj = {};
Object.prototype.name = "tugenhua";
console.log(obj.name); // tugenhua
```

给Object.prototype 定义一个属性，通过字面量构建的对象的话，都会从父类那边获取Object.prototype的属性；

从上面代码我们知道，原型继承的方法是：假如A需要继承于B，那么A.prototype(A的原型) = new B()（作为B的实列） 即可实现A继承于B; 因此我们下面可以初始化一个空的构造函数;然后把对象赋值给构造函数的原型，然后返回该构造函数的实列； 即可实现继承; 如下代码：

```
if(typeof Object.create !== 'function') {
    Object.create = function(o) {
        var F = new Function();
        F.prototype = o;
        return new F();
    }
}
var a = {
    name: 'longen',
    getName: function(){
        return this.name;
    }
};
var b = {};
b = Object.create(a);
console.log(typeof b); //object
console.log(b.name);   // longen
console.log(b.getName()); // longen
```

如上代码:我们先检测Object是否已经有Object.create该方法;如果没有的话就创建一个； 该方法内创建一个空的构造器，把参数对象传递给构造函数的原型，最后返回该构造函数的实列，就实现了继承方式；如上测试代码：先定义一个a对象，有成员属性name='longen'，还有一个getName()方法;最后返回该name属性; 然后定义一个b空对象，使用Object.create(a);把a对象继承给b对象，因此b对象也有属性name和成员方法getName();

理解原型查找原理：对象查找先在该构造函数内查找对应的属性，如果该对象没有该属性的话，

那么javascript会试着从该原型上去查找，如果原型对象中也没有该属性的话，那么它们会从原型中的原型去查找，直到查找的Object.prototype也没有该属性的话，那么就会返回undefined;因此我们想要仅在该对象内查找的话，为了提高性能，我们可以使用hasOwnProperty()来判断该对象内有没有该属性，如果有的话，就执行代码(使用for-in循环查找)：如下：

```
var obj = {
    "name":'tugenhua',
    "age":'28'
};
// 使用for-in循环
for(var i in obj) {
    if(obj.hasOwnProperty(i)) {
        console.log(obj[i]); //tugenhua 28
    }
}
```

如上使用for-in循环查找对象里面的属性，但是我们需要明白的是：for-in循环查找对象的属性，它是不保证顺序的，for-in循环和for循环；最本质的区别是：for循环是有顺序的，for-in循环遍历对象是无序的，因此我们如果需要对象保证顺序的话，可以把对象转换为数组来，然后再使用for循环遍历即可;

下面我们来谈谈原型继承的优点和缺点 

```
// 先看下面的代码：
// 定义构造函数A，定义特权属性和特权方法
function A(x) {
    this.x1 = x;
    this.getX1 = function(){
        return this.x1;
    }
}
// 定义构造函数B，定义特权属性和特权方法
function B(x) {
    this.x2 = x;
    this.getX2 = function(){
        return this.x1 + this.x2;
    }
}
B.prototype = new A(1);
```

`B.prototype = new A(1)`;这句代码执行的时候,B的原型继承于A，因此`B.prototype`也有A的属性和方法,即：`B.prototype.x1 = 1`; `B.prototype.getX1` 方法；但是B也有自己的特权属性x2和特权方法getX2; 如下代码：

```
function C(x) {
    this.x3 = x;
    this.getX3 = function(){
        return this.x3 + this.x2;
    }
}
C.prototype = new B(2);
C.prototype = new B(2);这句代码执行的时候,C的原型继承于B，因此C.prototype.x2 = 2; C.prototype.getX2方法且C也有自己的特权属性x3和特权方法getX3,
var b = new B(2);
var c = new C(3);
console.log(b.x1);  // 1
console.log(c.x1);  // 1
console.log(c.getX3()); // 5
console.log(c.getX2()); // 3
var b = new B(2); 
```

实列化B的时候 b.x1 首先会在构造函数内查找x1属性，没有找到，由于B的原型继承于A，因此A有x1属性，因此B.prototype.x1 = 1找到了;var c = new C(3); 实列化C的时候，从上面的代码可以看到C继承于B，B继承于A，因此在C函数中没有找到x1属性，会往原型继续查找，直到找到父元素A有x1属性，因此c.x1 = 1;c.getX3()方法; 返回this.x3+this.x2 this.x3 = 3;this.x2 是B的属性，因此this.x2 = 2;c.getX2(); 查找的方法也一样，不再解释

prototype的缺点与优点如下：

- 优点是：能够允许多个对象实列共享原型对象的成员及方法，
- 缺点是：
    - 1. 每个构造函数只有一个原型，因此不直接支持多重继承;
    - 2. 不能很好地支持多参数或动态参数的父类。在原型继承阶段，用户还不能决定以

什么参数来实列化构造函数。


## 四：理解使用类继承(继承的更好的方案)

类继承也叫做构造函数继承，在子类中执行父类的构造函数；实现原理是：可以将一个构造函数A的方法赋值给另一个构造函数B，然后调用该方法，使构造函数A在构造函数B内部被执行，这时候构造函数B就拥有了构造函数A中的属性和方法，这就是使用类继承实现B继承与A的基本原理；

如下代码实现demo：

```
function A(x) {
    this.x = x;
    this.say = function(){
        return this.x;
    }
}
function B(x,y) {
    this.m = A; // 把构造函数A作为一个普通函数引用给临时方法m
    this.m(x);  // 执行构造函数A;
    delete this.m; // 清除临时方法this.m
    this.y = y;
    this.method = function(){
        return this.y;
    }
}
var a = new A(1);
var b = new B(2,3);
console.log(a.say()); //输出1, 执行构造函数A中的say方法
console.log(b.say()); //输出2, 能执行该方法说明被继承了A中的方法
console.log(b.method()); // 输出3， 构造函数也拥有自己的方法
```

上面的代码实现了简单的类继承的基础，但是在复杂的编程中是不会使用上面的方法的，因为上面的代码不够严谨；代码的耦合性高；我们可以使用更好的方法如下：

```
function A(x) {
    this.x = x;
}
A.prototype.getX = function(){
    return this.x;
}
// 实例化A
var a = new A(1);
console.log(a.x); // 1
console.log(a.getX()); // 输出1
// 现在我们来创建构造函数B，让其B继承与A，如下代码：
function B(x,y) {
    this.y = y;
    A.call(this,x);
}
B.prototype = new A();  // 原型继承
console.log(B.prototype.constructor); // 输出构造函数A，指针指向与构造函数A
B.prototype.constructor = B;          // 重新设置构造函数，使之指向B
console.log(B.prototype.constructor); // 指向构造函数B
B.prototype.getY = function(){
    return this.y;
}
var b = new B(1,2);
console.log(b.x); // 1
console.log(b.getX()); // 1
console.log(b.getY()); // 2

// 下面是演示对构造函数getX进行重写的方法如下：
B.prototype.getX = function(){
    return this.x;
}
var b2 = new B(10,20);
console.log(b2.getX());  // 输出10
```

下面我们来分析上面的代码：

在构造函数B内，使用A.call(this,x);这句代码的含义是：我们都知道使用call或者apply方法可以改变this指针指向，从而可以实现类的继承，因此在B构造函数内，把x的参数传递给A构造函数，并且继承于构造函数A中的属性和方法；

使用这句代码：B.prototype = new A();  可以实现原型继承，也就是B可以继承A中的原型所有的方法；console.log(B.prototype.constructor); 打印出输出构造函数A，指针指向与构造函数A；我们明白的是，当定义构造函数时候，其原型对象默认是一个Object类型的一个实例，其构造器默认会被设置为构造函数本身，如果改动构造函数prototype属性值，使其指向于另一个对象的话，那么新对象就不会拥有原来的constructor的值，比如第一次打印console.log(B.prototype.constructor); 指向于被实例化后的构造函数A，重写设置B的constructor的属性值的时候，第二次打印就指向于本身B；因此B继承与构造A及其原型的所有属性和方法，当然我们也可以对构造函数B重写构造函数A中的方法，如上面最后几句代码是对构造函数A中的getX方法进行重写，来实现自己的业务~；

## 五：建议使用封装类实现继承
   
封装类实现继承的基本原理：先定义一个封装函数extend；该函数有2个参数，Sub代表子类，Sup代表超类；在函数内，先定义一个空函数F, 用来实现功能中转，先设置F的原型为超类的原型，然后把空函数的实例传递给子类的原型，使用一个空函数的好处是：避免直接实例化超类可能会带来系统性能问题，比如超类的实例很大的话，实例化会占用很多内存；

如下代码：

```
function extend(Sub,Sup) {
    //Sub表示子类，Sup表示超类
    // 首先定义一个空函数
    var F = function(){};

    // 设置空函数的原型为超类的原型
    F.prototype = Sup.prototype; 

    // 实例化空函数，并把超类原型引用传递给子类
    Sub.prototype = new F();
            
    // 重置子类原型的构造器为子类自身
    Sub.prototype.constructor = Sub;
            
    // 在子类中保存超类的原型,避免子类与超类耦合
    Sub.sup = Sup.prototype;

    if(Sup.prototype.constructor === Object.prototype.constructor) {
        // 检测超类原型的构造器是否为原型自身
        Sup.prototype.constructor = Sup;
    }

}
```

测试代码如下：

```
// 下面我们定义2个类A和类B，我们目的是实现B继承于A
function A(x) {
    this.x = x;
    this.getX = function(){
        return this.x;
    }
}
A.prototype.add = function(){
    return this.x + this.x;
}
A.prototype.mul = function(){
    return this.x * this.x;
}
// 构造函数B
function B(x){
    A.call(this,x); // 继承构造函数A中的所有属性及方法
}
extend(B,A);  // B继承于A
var b = new B(11);
console.log(b.getX()); // 11
console.log(b.add());  // 22
console.log(b.mul());  // 121
```

> 注意：在封装函数中，有这么一句代码：Sub.sup = Sup.prototype; 我们现在可以来理解下它的含义：

比如在B继承与A后，我给B函数的原型再定义一个与A相同的原型相同的方法add();

如下代码

```
extend(B,A);  // B继承于A
var b = new B(11);
B.prototype.add = function(){
    return this.x + "" + this.x;
}
console.log(b.add()); // 1111
```

那么B函数中的add方法会覆盖A函数中的add方法；因此为了不覆盖A类中的add()方法，且调用A函数中的add方法；可以如下编写代码：

```
B.prototype.add = function(){
    //return this.x + "" + this.x;
    return B.sup.add.call(this);
}
console.log(b.add()); // 22
```

`B.sup.add.call(this)`; 中的B.sup就包含了构造函数A函数的指针，因此包含A函数的所有属性和方法；因此可以调用A函数中的add方法；

如上是实现继承的几种方式，类继承和原型继承，但是这些继承无法继承DOM对象，也不支持继承系统静态对象，静态方法等；比如Date对象如下：

```
// 使用类继承Date对象
function D(){
    Date.apply(this,arguments); // 调用Date对象，对其引用，实现继承
}
var d = new D();
console.log(d.toLocaleString()); // [object object]
```

如上代码运行打印出object，我们可以看到使用类继承无法实现系统静态方法date对象的继承，因为他不是简单的函数结构，对声明，赋值和初始化都进行了封装，因此无法继承；

下面我们再来看看使用原型继承date对象；

```
function D(){}
D.prototype = new D();
var d = new D();
console.log(d.toLocaleString());//[object object]
```

我们从代码中看到，使用原型继承也无法继承Date静态方法；但是我们可以如下封装代码继承：

```
function D(){
    var d = new Date();  // 实例化Date对象
    d.get = function(){ // 定义本地方法，间接调用Date对象的方法
        console.log(d.toLocaleString());
    }
    return d;
}
var d = new D();
d.get(); // 2015/12/21 上午12:08:38
```

## 六：理解使用复制继承
   
复制继承的基本原理是：先设计一个空对象，然后使用for-in循环来遍历对象的成员，将该对象的成员一个一个复制给新的空对象里面;这样就实现了复制继承了;如下代码：

```
function A(x,y) {
    this.x = x;
    this.y = y;
    this.add = function(){
        return this.x + this.y;
    }
}
A.prototype.mul = function(){
    return this.x * this.y;
}
var a = new A(2,3);
var obj = {};
for(var i in a) {
    obj[i] = a[i];
}
console.log(obj); // object
console.log(obj.x); // 2
console.log(obj.y); // 3
console.log(obj.add()); // 5
console.log(obj.mul()); // 6
```

如上代码：先定义一个构造函数A，函数里面有2个属性x,y,还有一个add方法，该构造函数原型有一个mul方法，首先实列化下A后，再创建一个空对象obj，遍历对象一个个复制给空对象obj，从上面的打印效果来看，我们可以看到已经实现了复制继承了;对于复制继承，我们可以封装成如下方法来调用:

```
// 为Function扩展复制继承方法
Function.prototype.extend = function(o) {
    for(var i in o) {
        //把参数对象的成员复制给当前对象的构造函数原型对象
        this.constructor.prototype[i] = o[i];
    }
}
// 测试代码如下：
var o = function(){};
o.extend(new A(1,2));
console.log(o.x);  // 1
console.log(o.y);  // 2
console.log(o.add()); // 3
console.log(o.mul()); // 2
```

上面封装的扩展继承方法中的this对象指向于当前实列化后的对象，而不是指向于构造函数本身，因此要使用原型扩展成员的话，就需要使用constructor属性来指向它的构造器，然后通过prototype属性指向构造函数的原型；

复制继承有如下优点：

1. 它不能继承系统核心对象的只读方法和属性
2. 如果对象数据非常多的话，这样一个个复制的话，性能是非常低的;
3. 只有对象被实列化后，才能给遍历对象的成员和属性，相对来说不够灵活;
4. 复制继承只是简单的赋值，所以如果赋值的对象是引用类型的对象的话，可能会存在一些副作用;如上我们看到有如上一些缺点，下面我们可以使用clone(克隆的方式)来优化下：

基本思路是：为Function扩展一个方法，该方法能够把参数对象赋值赋值一个空构造函数的原型对象，然后实列化构造函数并返回实列对象，这样该对象就拥有了该对象的所有成员;代码如下：

```
Function.prototype.clone = function(o){
    function Temp(){};
    Temp.prototype = o;
    return Temp();
}
// 测试代码如下：
Function.clone(new A(1,2));
console.log(o.x);  // 1
console.log(o.y);  // 2
console.log(o.add()); // 3
console.log(o.mul()); // 2
```