[toc]

ECMAScript只支持继承，不支持接口实现，而实现继承的方式依靠原型链形成。

## 1.使用原型链继承
```
function Box() { //被继承的函数叫做超类型(父类，基类)
this.name = 'Lee';
}

function Desk() { //继承的函数叫做子类型(子类，派生类)
this.age = 100;
}

function Table() {
this.level = 'AAAAA';
}

//通过原型链继承，超类型实例化后的对象实例，赋值给子类型的原型属性
//new Box()会将Box构造里的信息和原型里的信息都交给Desk
//Desk的原型，得到的是Box的构造+原型里的信息
Desk.prototype = new Box();
Table.prototype = new Desk();
var box = new Box();
alert(box.constructor);
var desk = new Desk();
//alert(desk.age);
//alert(desk.name);

var table = new Table();
//alert(table.level);
//alert(desk.age);
//alert(desk.name);
function Box() { //被继承的函数叫做超类型(父类，基类)
this.name = 'Lee';
}

Box.prototype.name = 'Jack';

function Desk() { //继承的函数叫做子类型(子类，派生类)
this.age = 100;
}

Desk.prototype = new Box(); //通过原型链继承

var box = new Box();
var desk = new Desk();
//alert(desk.name); //就近原则，实例里有，就返回，没有就去查找原型

//子类型从属于自己或者他的超类型
//alert(desk instanceof Object);
alert(desk instanceof Desk);
alert(desk instanceof Box);
alert(box instanceof Desk);
```

## 2.借用构造函数继承（使用对象冒充继承）

```
function Box(name, age) {
this.name = name;
this.age = age;
this.family = ['哥哥','姐姐','妹妹']; //引用类型，放在构造里就不会被共享
}

//Box.prototype.family = '家庭';

function Desk(name, age) {
Box.call(this, name, age) //对象冒充，对象冒充只能继承构造里的信息
}

var desk = new Desk('caibaojian', 100);
alert(desk.family);
desk.family.push('弟弟');
alert(desk.family); //哥哥,姐姐,妹妹,弟弟

var desk2 = new Desk('caibaojian', 100);
alert(desk2.family);//不会共享属性，哥哥,姐姐,妹妹
```

## 3.我们需要原型链+借用构造函数的模式，这种模式称为组合继承

```
function Box(name, age) {
this.name = name;
this.age = age;
this.family = ['哥哥','姐姐','妹妹'];
}

Box.prototype.run = function () {
return this.name + this.age + '运行中...';
};

//构造函数里的方法，放在构造里，每次实例化，都会分配一个内存地址，浪费，所以最好放在原型里，保证多次实例化只有一个地址

function Desk(name, age) {
Box.call(this, name, age) //对象冒充
}

Desk.prototype = new Box(); //原型链继承（因为对象冒充只能继承构造里的信息，所以原型需要加上原型）

var desk = new Desk('Lee', 100);
alert(desk.run());
```