- [创建对象](#创建对象)
- [属性的查询和设置](#属性的查询和设置)
- [删除属性](#删除属性)
- [检测属性](#检测属性)
- [枚举属性](#枚举属性)
- [属性getter和setter](#属性getter和setter)
- [属性的特性](#属性的特性)
- [对象的三个属性](#对象的三个属性)
- [序列化对象](#序列化对象)
- [对象方法](#对象方法)

---
#### 创建对象
- 对象直接量

```javascript
var a = {},
    b = {x: 0, y: 1},
    c = {x: b.x, y: b.y},
    d = {
        name: 'UMEI',
        year: 2,
        member: {
            'CEO': 'xu',
            'CTO': 'yonnie'
        }
    }
```

- **new**

> 通过new运算符创建一个对象，这里称作构造函数，构造函数用以初始化一个新创建的对象

```javascript
var a = new Object(),
    b = new Array(),
    c = new Date()
```

- **原型**

**每一个javascript对象（null除外）都和一个原型对象相关联，每一个对象都从原型继承属性。**

所有通过对象直接量创建的对象都具有同一个原型对像--**Object.prototype**。

通过new创建的对象的原型就是构造函数的prototype属性的值。因此通过new Object()创建的对象继承Object.prototype，通过new Array()创建的对象继承Array.prototype，通过new Date()创建的继承Date.prototype。

Object.prototype不继承任何属性，没有原型对象。

所有的其他的对象、原型对像都继承自Object.prototype。因此new Date()创建的对象同时继承自Date.prototype和Object.prototype。这一系列链接的原型对象就是所谓的**“原型链”**

- Object.create()

Object.create()可以传入两个参数，第一个参数是这个原型的对象，还有第二个可选参数。

```javascript
var a = Object.create({x: 1, y: 1});
a继承了x和y属性
```
可以通过传入null创建一个没有原型的新对象，但通过这种方式创建的不会继承任何方法和属性。

如果想创建一个普通的空对象，需要传入Object.prototype

可以通过任务原型创建新对象（可以使任意对象可继承）


```javascript
//inherit()返回了一个继承自原型对像p的属性的新对象
//这里使用es5中的Object.create()函数
//如果不存在Object.create()则使用其他方法

function inherit(p){
    if(p == null) throw TypeError(); //p如果是null或者undefined 抛出类型错误
    if(Object.create) //如果存在create方法，则使用create
        return Object.create(p);
    var t = typeof p;
    if(t !== "object" && t !== "function"){
        throw TypeError();
    }
    function f(){};
    f.prototype = p;
    return new f();
}

```
inherit()函数的作用就是防止库函数无意间修改那些不受你控制的对象。当函数读取继承对象的属性时，实际上读取的是继承来的值。如果给继承对象的属性赋值，不会影响原始对象

---

#### 属性的查询和设置

> 可以通过(.)和([])来获取属性的值。如果一个对象的属性名是保留字，必须使用[]来访问

##### 继承

> Javascript对象具有“自有属性”，也有一些属性是从原型对象继承而来。

查询 对象o的属性x，如果o中不存在x，那么就会沿着o的原型链查，直到找到x或者找到一个原型是null的对象位置。

```javascript
var o = {};
o.x = 1;

var p = inherit(o);
p.y = 2;

var q = inherit(p);
p.z = 3;

var s = q.toString();//toString继承自Object.prototype

q.x + q.y; //--> 3, x和y分别继承自o和p

q.x = 4; //会覆盖q继承的x属性，o的x属性不会变
```

属性赋值操作首先检查原型链。例如，如果o继承自一个只读属性x。那么赋值操作是不允许的。如果允许赋值操作，它也总是在原始对象上创建属性或对已有的属性赋值，不会去修改原型链。

在js中，只有在查询属性时才会体会到继承的存在，而设置属性则和继承无关。

##### 属性访问错误

访问一个不存在的属性不会报错，单数如果访问一个不存在的对象的属性就会报错

内置构造函数的原型是只读的。
```javascript
Object.prototype = 1; //不会报错，但是没被修改为1
```

在严格模式中，任何失败的属性设置操作都会抛出一个类型错误。

#### 删除属性

delete运算符可以删除对象的属性，delete只是断开属性和宿主对象的联系，而不会去操作属性中的属性。
```javascript
var a = {p: {x: 1}},
    b = a.p;
    
delete a.p;
console.log(b.x); //--> 删除了a.p属性，但是b.x的值还是1
```

已经删除的属性的引用依然存在，delete只能删除自有属性。

delete不能删除那些可配置性为false的属性。某些内置对象的属性是不可配置的，比如通过变量声明和函数声明创建的全局对象的属性。在严格模式中，删除一个不可配置属性会报一个类型错误。在非严格模式中，delete操作失败会返回false

```javascript
delete Object.prototype     //不能删除，属性不可配置
var x = 1;
delete this.x;              //返回false
delete x;                   //返回false

function f(){

}

delete this.f;              //不能删除全局函数
```

#### 检测属性
> 可以通过in运算符、hasOwnPreperty() 和 propertyInEnumerable()方法来判断某个属性是否存在于某个对象中。

in 运算符，如果对象的自由属性或继承属性中包含这个属性则返回true
```javascript
var o = {x: 1};
'x' in o;           //true
'y' in o;           //false
'toString' in o;    //true 继承的属性
```

hasOwnProperty()方法用来检测是否是自有属性，如果是继承的属性会返回false
```javascript
var o = {x: 1}
o.hasOwnProperty('x');      //true
o.hasOwnProperty('y');      //false
o.hasOwnProperty('toString');   //false
```

propertyIsEnumerable()是hasOwnProperty()的增强版，只有检测到自有属性且这个属性可枚举时才返回true。有些内置属性时不可枚举的

```javascript
var o = inherit({y: 2});
o.x = 1;
o.propertyIsEnumerable('y');        //false
o.propertyIsEnumerable('x');        //true
Object.prototype.propertyIsEnumerable('toString');      //false  toString不可枚举
```

有一种情况只能用in运算符来判断，就是存在并且值为undefined的属性
```javascript
var o = {x: 1, y: undefined};
'x' in o;   //true
'y' in o;   //true
'z' in o;   //false
delete o.x
'x' in o;   //false
```
#### 枚举属性
> for/in 循环可以在循环体中遍历对象中所有可枚举的属性（**包括自有和继承的**），把属性名称赋值给循环变量。**对象继承的内置方法不可枚举**，但在代码中给对象添加的属性都是可枚举的。

```javascript
var o = {
    x: 1,
    y: 2,
    z: 3,
    test: function(){
        console.log('test')
    }
},
    p = Object.create(o);
    
for(i in p){
    console.log(i);     //--> x,y,z,test; 继承自o的属性都是可枚举的, 没有toString
}
    
```

在es5之前，在Object.prototype添加的方法会被枚举出来。

定义一个extend()函数来枚举属性

```javascript
function extend(o, p){
    for (prop in p){
        o[prop] = p[prop];  //遍历p中的所有属性，将属性添加至o中
    }
    return o;
}

/*
*将p中的可枚举属性复制到o中，并返回o
*如果o和p中有同名的属性，o中的属性将不受影响。
*这个函数并不处理getter和setter以及复制属性
*
*/
function merge(o, p){
    for(prop in p){
        if(o.hasOwnProperty(prop)) continue; //过滤掉已经在o中存在的属性
        o[prop] = p[prop]
    }
    
    return o
}

/*
*如果o中的属性在p中没有同名属性，则从o中删除这个 属性
*返回o
*/
function restrict(o, p){
    for(prop in o){
        if(!(prop in p)) delete o[prop]; //如果在p中不存在，则删除之
    }
    return o;
}

/*
*如果o中的属性在p中存在同名属性，则从o中删除这个属性
*返回o
*/
function(o, p){
    for(prop in p){
        delete o[prop]
    }
    return o;
}

/*
*返回一个新对象，这个对象同时拥有o的属性和p的属性
*如果o和p有重名属性，使用p中的属性值
*/
function union(o, p){
    return extend(extend({}, o), p);
}

/*
*返回一个新对象， 这个对象拥有同时再o和p中竖线的属性
*很想求o和p的交集，但p中属性的值被忽略
*/
function intersection(o, p){
    return restrict(extend({}, o), p);
}

/*
*返回一个数组，这个数组包含的是o中可枚举的自有属性的名字
*/
function keys(o){
    if(typeof o !== 'object') throw TypeError();
    var result = [];
    for(var prop in o){
        if(o.hasOwnProperty(prop))
        result.push(prop);
    }
    return result;
}

```
##### 属性getter和setter

> 由getter和setter定义的属性称作“存取器属性”，不同于数据属性，数据属性只有一个简单的值

当程序查询存取器属性的值时，js调用getter方法。当设置存取器属性的值时，调用setter方法。从某种意义上来看，这个方法负责“设置”属性值，可以忽略setter 方法的返回值。

存取器属性和数据属性不同，它不具有可写性。如果只有getter方法，那么是一个只读属性。如果只有setter方法，那么是只写属性，读取时总是返回undefined。

定义存取器属性最简单的方法是使用对象直接量语法的一种扩展写法：

```javascript
var o = {
    //普通的数据属性
    data_drop: 1,
    
    //存取器属性
    get accessor_prop(){
        //函数体
    }，
    
    set accessor_prop(value){
        //函数体
    }
    
}
```

存取器属性定义为一个或两个和属性同名的函数，这个函数定义**没有用function关键字**， 而是用了set、get，没有使用冒号。 注意，与下一个方法或者属性之间有逗号分隔。

```javascript
var p = {
    x: 1.0,
    y: 1.0,
    
    get r(){
        return Math.sqrt(this.x*this.x + this.y*this.y)
    },
    set r(new_value){
        var old_value = this.x*this.x + this.y*this.y,
            ratio = new_value/old_value;
        this.x *= ratio;
        this.y *= ratio;
    },
    
    get theta(){
        return Math.atan2(this.y, this.x);
    }
}
```

跟数据属性一样，存取器属性也是可以继承的

```javascript
var q = inherit(p);
q.x = 1, q.y = 1;
console.log(q.r);   //可以使用继承的存取器属性
```

#### 属性的特性

- 值
- 可写性
- 可枚举性
- 可配置性

存取器属性不具有值特性和可写性，他们的可写性由setter方法决定。因此存取器属性的4个特性是读取、写入、可枚举性和可配置性

为了实现属性特性的查询和设置操作，es5中定义了一个“属性描述符”的对象。

```javascript
//返回{value: 1, writable: true, enumerable: true, configurable: true}
Object.getOwnPropertyDescriptor({x: 1}, 'x');

var a = {
    get random(){
        return Math.random()
    }
}
//{get: function random(), set: undefined, configurable: true, enumerable: true}
Object.getOwnPropertyDescriptor(a, 'random');

//对于继承属性没返回undefined
Object.getOwnPropertyDescriptor({}, 'toString'); //undefined
```

Object.getOwnPropertyDescriptor()只能得到自由属性的描述符，如果想获得继承属性的特性，则需要遍历原型链。

我们可以调用Object.defineProperty()来设置属性的特性，传入要修改的对象、要创建或者修改的属性的名称以及属性描述符对象：

```javascript
var o = {};
Object.defineProperty(o, 'x', {
    value: 1,
    writable: true,
    enumerable: false,
    configurable: true
});

o.x;        //-->1  属性存在
Object.keys(o);     //--> [] 但是不可枚举

//设置属性只读
Object.defineProperty(o, 'x', {
    writable: false
});

o.x = 2 //操作失败，在严格模式中会报错

//虽然属性只读，但是依然可以通过配置来修改属性的值
Object.defineProperty(o, 'x', {
    value: 2
});

o.x;    //2

//将x从数据属性修改成存取器属性
Object.defineProperty(o, 'x', {
    get: function(){
        return 0;
    }
});

```
如果需要同时修改或者创建多个属性则需要使用Object.defineProperties()。第一个参数是对象，第二个参数是一个映射表。

```javascript
var p = Object.defineProperties({}, {
    x: {
        value: 1,
        writable: true,
        enumerable: true,
        configurable: true
    },
    y: {
        value: 1,
        writable: true,
        enumerable: true,
        configurable: true
    },
    r: {
        get: function(){
            return Math.sqrt(this.x*this.x + this.y*this.y)
        },
        enumerable: true,
        configurable: true
    }
})
```

- 如果对象不可扩展，则只能编辑已有的自有属性，不能添加新属性
- 如果属性时不可配置的，则不能修改可配置性和可枚举性
- 如果存取器属性不可配置，则不能修改getter和setter方法，也不能转为数据属性
- 如果数据属性不可配置，则不能转为存取器属性，不能将它的可写性从false改为true，但是可以从true改为false
- 如果数据属性是不可配置且不可写的，则不能修改它的值。如果是可配置的，那么可以通过defineProperty()方法修改值

#### 对象的三个属性

- 原型属性

通过对象直接量创建的对象使用Object.prototype作为原型。

通过new创建的对象使用率构造函数的prototype属性作为原型。

使用Object.create()创建的对象使用第一个参数(也可以是null)作为原型。

在es5中将对象作为参数传入Object.getPrototypeof()可以查询它的原型。在es3中，经常用表达式o.constructor.prototype 来检测一个对象的原型。通过new表达式创建的对象，通常继承一个constructor属性，这个属性指代创建这个函数的构造函数。(更多的细节再[9.2节](#)进一步讨论)


通过对象直接量或者Object.create()创建的对象包含一个名为constructor的属性，这个属性指代Object()的构造函数，constructor.prototype才是真正的原型。要想检测一个对象是否是另一个对象的原型（或者处于原型链中），可以使用isPrototypeOf()方法

```javascript
var p = {x: 1},
    o = Object.create(p);
    
p.isPrototypeOf(o); //true
Object.prototype.isPrototypeOf(o)   //true
```
对象还有一个__proto__属性，但是ie和opera还不支持
- 类属性

es3和es5都没有设置这个属性的方法，只有toString()方法会返回对应的字符串
[object class]

- 可扩展性

对象的可扩展性用以表示是否可以给对象添加新属性。所有内置对象和自定义对象都是显式可扩展的。

es5定义了一个方法--Object.esExtensible()，来判断对象是否是可扩展的。

Object.preventExtensions()可以将对象转换为不可扩展的。一旦转换成不可扩展的对象，无法再转换回可扩展的了！！

preventExtensions()只影响对象本身的可扩展性，如果给它的原型添加属性，这个不可扩展的对象同样能继承新属性。

Object.seal()和preventExtensions()类型，除了能将对象设置为不可扩展的还能降对象的所有自有属性设置成不可配置的，不能给这个属性添加新属性，已有的属性不能删除或者配置，但是已有的可写属性依然可以设置。可以用Object.isSealed()来检测对象是否封闭

Object.freeze()可以“冻结”对象。它将对象设置为不可扩展的，属性设置为不可配置外，还能将对象的所有数据属性设置为只读的。可以通过Object.isFrozen()来检测对象是否冻结。

#### 序列化对象

NaN、Infinity和-Infinity序列化的结果是null，日期对象的序列化结果是ISO格式的日期字符串，但JSON.parse()依然会保留他们的字符串形态，而不是还原成原始的日期对象。

函数、RegExp、Error对象和undefined值不能序列化和还原。

JSON.stringify()只能序列化对象可枚举的自有属性。

JSON.stringify()和JSON.parse()都能接受第二个可选参数，通过传入需要序列化或还原的属性列表来定制自定义的序列化或还原操作。Javascript核心参考中有详细文档

#### 对象方法

- toString()方法
- toLocaleString()方法
- toJSON()方法
- valueOf()方法

