- [函数定义](#函数定义)
- [函数调用](#函数调用)
- [函数的实参和形参](#函数的实参和形参)
- [作为值得函数](#作为值得函数)
- [作为命名空间的函数](#作为命名空间的函数)
- [闭包](#闭包)
- [函数属性、方法和构造函数](#函数属性、方法和构造函数)
- [函数式编程](#函数式编程)**(记忆&不完全函数未完成)**

---

#### 函数定义

函数使用function关键字来定义，可以用在函数定义表达式或者函数声明语句中。

> 需要注意的是，用function声明的函数会被提前，但是通过表达式的方式定义的函数只是变量提前了，赋值操作没提前，因此不能在表达式定义之前调用这个函数

##### 嵌套函数
```javascript
function hypotenuse(a, b){
    function square(x){
        return x*x
    }
    
    return Math.sqrt(square(a) + square(b));
}
```
嵌套函数的有趣之处在意变量作用域规则：它们可以访问嵌套它们（或者多重嵌套）的函数的参数和变量。

#### 函数调用

- 作为函数
- 作为方法
- 作为构造函数
- 通过apply()方法和call()方法调用

##### 函数调用

作为普通的函数调用， 函数的返回值成为调用表达式的值。如果该函数返回是因为解释器到达结尾，返回值就是undefined。如果有return 语句，则返回return的值。

以函数形式调用的函数通常不使用this关键字。不过，“this”可以用来判断当时是不是严格模式

```javascript
var isStrict = (function(){ return !this }());
```

#####方法调用

> 方法是保存在对象属性中的js函数

方法调用中的this指向的是当前对象。

```javascript
var calculator = {
    operand1: 1,
    operand2: 2,
    add: function(){
        this.result = this.operand1 + this.operand2
    },
    child: {
        operand1: 3,
        operand1: 4,
        add: function(){
            //此处的this只的是child这个上下文
            this.result = this.operand1 + this.operand2
        }
    }
};
calculator.add();
calculator.child.add();
calculator.result   //--> 2

```

**方法和this关键字是面向对象编程的核心**，任何函数只要作为方法调用实际上都会传入一个隐式的实参--这个实参就是一个对象，方法调用的母体就是这个对象。通常来讲，基于那个对象的方法可以执行多种操作，方法调用的语法很清晰的表明函数基于对象进行操作。
```javascript
rect.setSize(width. height);
setRectSize(rect, width, height);
```

假设这两行代码的功能完全一样，都作用于一个假定的对象rect。第一行的方法调用语法非常清晰的表明这个函数执行的载体是rect对象，函数中所有操作都是基于这个对象的。

> 方法链----当方法的返回值是一个对象，这个对象还可以再调用它的方法。这种方法调用序列中每次的调用结果都是另外一个表达式的组成部分。

> 当方法并不需要返回值时，最好直接返回this。如果在设计的API中一直采用这种方法（每个方法都返回this）， 使用API就可以进行“链式调用”风格的编程了。例如： shape.setX(100).setY(100).setOutline('red').draw();

> 不要将方法的链式调用和构造函数的链式调用混为一谈

this是一个关键字，不是变量也不是属性名，不能给this赋值。

this没有作用域的限制，嵌套函数不会从调用它的函数中继承this。如果嵌套函数作为方法调用，其this值指向调用它的对象。如果嵌套函数作为函数调用，this指向全局对象在严格模式下是undefined。

调用嵌套函数时，如果需要访问外部函数的this值，需要将this赋值给一个变量。

```javascript
var o = {
    m: function(){
        console.log(this === o);
        var self = this;
        
        function f(){
            console.log(this === o);
            console.log(self === o);
        };
        
        f();
    }
}
```

##### 构造函数

> 如果函数或者方法调用之前带有关键字new， 它就构成构造函数。构造函数调用和普通的函数调用以及方法调用在实参处理、调用上下文和返回值方面都有不同。

如果构造函数调用在圆括号内的一组实参列表，先计算这些实参表达式，然后传入函数内，这和函数调用方法调用是一致的。

凡是没有形参的构造函数调用都可以省略括号。

构造函数调用创建一个新的空对象，这个对象继承自构造函数的prototype属性。构造函数试图初始化这个新创建的对象，并将这个对象用做其调用上下文，因此构造函数可以使用this关键字来引用这个新创的对象。

注意，尽管构造函数看起来像一个方法调用，它依然会使用这个新对象作为调用上下文，在表达式new o.m()中，调用的上下文是new的新对象而不是o

构造函数通常不使用return关键字，它们通常初始化新对象，当构造函数的函数体执行完毕时，它会显式返回。在这种情况下，构造函数调用表达式的计算结果就是这个新对象的值。

如果构造函数显式的使用return语句返回一个对象，那么调用表达式的值就是这个对象。如果构造函数使用return语句但是没有执行返回值，或者返回一个原始值，那么它将忽略返回值，同时使用这个新对象作为调用结果。

##### 间接调用

> 函数对象也有两个方法， call和apply，可以间接的调用函数。两个方法都允许显式指定调用所需的this值，任何函数可以作为任何对象的方法来调用，即使这个函数不是那个对象的方法。两个方法都可以制定法调用的参数。call()方法使用它自有的实参列表作为函数的实参，apply()则要求以数组的形式传入参数。


#### 函数的实参和形参
> js 中的函数定义并未指定函数形参的类型，函数调用也没有对传入的实参做任何类型检查。 函数调用实际上都不检查传入形参的个数

##### 可选形参

> 当调用函数的时候传入的参数比函数声明时指定的形参个数少，那么剩下的形参都将设置为undefined值。因此在调用函数时心肝是否可选以及是否可以省略应当保持较好的适应性。可以给省略的参数设置一个默认值

```javascript
function getPropertyNames(o, a){
    a = a || [];
    for (var prop in o) a.push(prop);
    return a;
}

```

##### 可变长的实参列表：实参对象

> 为了解决传入的实参个数超过形参个数导致无法获取未命名的值时，标识符arguments通过指向实参对象的引用，解决了这个问题。

实参对象有一个重要的用处，可以让函数操作任意数量的实参。（Math.max()的功能与之类似）

```javascript
function max(/******/){
    var max = Number.NEGATIVE_INFINITY;
    for (var i = 0; i < arguments.length; i++){
        arguments[i] > max ? max = arguments[i] : max = max;
    }
    
    return max;
}

var largest = max(1,10,100,1111); //--> 1111
```

上面这种函数也被称为“不定实参函数”，这个术语源自古老的C语言。

arguments并不是真正的数组，它是一个实参对象。每个实参对象都是以数字为所以呢的一组元素以及length属性。

在非严模式下，当一个函数包含若干形参，实参对象的数组元素是函数形参对用实参的别名，实参对象中以数字索引，并且形参名称可以认为是相同变量的不同命名。通过实参名字来修改实参值得花，通过arguments[]数组也可以获取到更改后的值。

```javascript
function f(x){
    console.log(x);
    arguments[0] = null;
    console.log(x);
}
```

**在es5中移除了实参对象的这个特殊特性。在严格模式下，arguments变成一个保留字，无法使用arguments作为形参名或者局部变量名，不能给arguments赋值**

**callee和caller属性**

除了数组元素，实参对象还定义了callee和caller属性。**在严格模式下，对这两个属性的读写操作都会产生类型错误**。在非严格下，callee属性指代当前正在执行的函数。**caller是非标准的**，但大多数浏览器都实现了这个属性，它指代调用当前正在执行的函数的函数。通过caller属性可以访问调用栈。callee属性在某些时候很有用，比如在匿名函数中通过callee来递归调用自身。

```javascript
var factorial = function(x){
    if(x <= 1) return 1;
    return x*arguments.callee(x-1);
}
```

##### 将对象属性用作实参

当一个函数包含超过三个形参时，建议以对象形式传入，还不用考虑实参的顺序。


##### 实参类型

js方法的形参并没有声明类型，可以在抛出异常前尽可能的转换类型或者停止代码运行。

```javascript
function flexisum(a){
    var total = 0;
    for(var i = 0; i < arguments.length ; i++ ){
        var element = arguments[i],
            n;
            if(element = null) continue; //忽略null个undefined实参
            if(isArray(element)
                n = flexisum.apply(this, element); //递归计算累加
            else if(typeof element === 'function')
                n = Number(element());  //如果是函数就调用ta并作类型转换
            else
                n = Number(element);    //否则直接强转类型
            
            if(isNaN(n)){
                throw Error(element + 'is not a number')
            }
            total += n;
    }
    return total;
}
```

#### 作为值的函数

> 函数不仅是一种语法，也是值。可以将函数赋值给变量，存储在对象的属性或数组的元素中，作为参数传入另一个函数等

```javascript
function square(x){ return x*x }
```
这个定义创建了一个新的函数对象，并将其赋值给变量square。函数的名字实际上是看不见的，square仅仅是变量的名字，这个变量指代函数对象。函数还可以赋值给其他的变量，并且扔可以正常工作。
```javascript
var s = square;
square(4)  === s(4);
```

#### 自定义函数属性
js中的函数并不是原始值，而是一种特殊的对象，函数其实是可以拥有属性的。例如：

```javascript
uniqueInteger.counter = 0;

function uniqueInteger(){
    return uniqueInteger.counter++;
}
```

下面的函数使用了自身的属性来缓存上一次的计算结果：
```javascript
function factorial(n){
    if(isFinite(n) && n > 0 && Math.round(n)){  //有限的正整数
        if(!(n in factorial)){                  //如果没有缓存结果
            facrotial[n] = n * factorial(n - 1);//计算并缓存
        }
        return factorial[n];                    //返回缓存结果
    }else{
        return NaN; //如果输入有误，则返回NaN
    }
}
fatorial[1] = 1;    //初始化缓存以保存这种基本情况
```

#### 作为命名空间的函数

> 在js中我们无法声明只在一个代码块内可见的变量，因此我们常常定义一个函数用作临时的命名空间，在这个命名空间内定义的变量不会污染到全局命名空间

```javascript
//定义一个扩展函数，用来将第二个以及后续的参数复制到第一个参数
//这里处理的IE的bug：在多数ie中如果o属性拥有一个不可枚举的同名属性，则for/in循环不会枚举对象o的可枚举属性
//将不会正确的处理toString的属性
//除非我们显式检测它
var extend = (function(){
    for (var p in {toString: null}){
        return function extend(o){
            for(var i = 1; i < arguments.length; i++){
                var source = arguments[i];
                for(var prop in source){o[prop] = source[prop]};
            }
            
            return o;
        };
    }
    
    return function patched_extend(o){
        for(var i = 1; i < arguments.length; i++){
            var source = arguments[i];
            for(var prop in source) o[prop] = source[prop];
            
            for(var j = 0; j < protoprops.length; j++){
                prop = protoprops[j];
                if(source.hasOwnProperty(prop)) o[prop] = source[prop];
            }
        }
        
        return o
    };
    
    var protoprops = ['toString', 'valueOf', 'constructor', 'hasOwnproperty', 'isPrototypeOf', 'propertyIsEnumerable', 'toLocaleString']
}())
```

####闭包

和其他变成语言一样，函数的执行依赖于变量作用域，**这个作用域在函数定义时决定的**。为了实现这种词法作用域，javascript函数对象的内部状态不仅包含函数的代码逻辑，还必须引用当前的作用域链。函数对象可以通过作用域链相互关联，函数体内部的变量都可以保存在函数作用域内，这种特性成为**“闭包”**。

从技术角度上说，所有的js函数都是闭包： **它们都是对象，都关联到作用域链。** 丁辉大多数函数时的作用域链在调用函数时依然有效，单着不影响闭包。当调用函数时闭包所指向的作用域链和定义函数时的作用域链不是同一个作用域链时，事情就变得非常微妙。当一个函数嵌套了另一个函数，外部函数将嵌套的函数对象作为返回的时候往往会法伤这种事情。有很多强大的编程技术都利用了这类嵌套的函数闭包，以至于这种编程模式在JavaScript中非常常见。当第一次碰到闭包时会非常让人费解。

理解闭包首先要了解嵌套函数的词法作用域规则。看下面的代码。。

```javascript
var scope = 'global scope';
function checkscope(){
    var scope = 'local scope';
    function f(){
        return scope;
    }
    return f();
}
checkscoe();                        //-->'local scope'
```

checkscope()函数声明了一个局部变量，并定义了一个函数f()，函数f()返回了这个变量的值，最后将函数f()的执行结果返回。

```javascript
var scope = 'global scope';
function checkscope(){
    var scope = 'local scope';
    function f(){
        return scope;
    }
    return f;
}
checkscoe()();                      //-->'local scope'
```
返回的还是local scope!!!回想下词法作用域的基本原则：JavaScript函数的执行到了作用域链，这个作用域链式函数定义的时候创建的。嵌套函数的f()定义在这个作用域里，其中的变量scope一定是局部变量，不管在何时何地执行f()，这种绑定在执行f()时依然有效。因此在最后一行代码返回“local scope”。简言之，**闭包的这个特性可以捕捉到局部变量（和参数），并一直保存下来**，看上去像这些变量绑定到了再其中定义它们的外部函数。

在8.4.1中定义了uniqueInteger()函数，这个函数用自身属性来保存每次的值，这可能会被人为的清0，而闭包不会。可以用闭包重写这个函数。

```javascript
var uniqueInteger = (function(){
    var count = 0;
    return function(){
        return count++;
    };
}());
uniqueInteger();    //0
uniqueInteger();    //1
uniqueInteger();    //2
```

这段代码定义了一个立即调用的函数，返回值赋值给变量uniqueInteger。函数体内返回了另一个函数，这个一个嵌套函数，我们将它赋值给变量定义的count。当外部函数返回之后，其他代码都无法访问count变量，只有内部函数才能访问它。

**像count一样的私有变量不是只能在一个单独的比包内，在同一个外部函数内定义的多个嵌套函数都能访问它，这多个嵌套函数都共享一个作用域链。**

```javascript
function counter(){
    var n = 0;
    return {
        count: function(){
            return n++;
        },
        reset: function(){
            n = 0
        }
    }
}

var c = counter(),
    d = counter();

c.count();      //0
d.count();      //0 两个互不干扰
c.reset();      //0 reset()和count()共享状态
d.count();      //1
a.count();      //0
```

counter()函数返回了一个“计数器”对象，对象的两个方法属性都可以访问私有变量n。**每次调用counter()都会创建一个新的作用域链和一个新的私有变量**。因此调用counter()两个，则会得到两个计数器对象，而且彼此包含不同的四右边哦啊宁。

从技术角度看，其实可以将这个闭包合并为属性存取器方法getter和setter。

```javascript
function counter(n){
    typeof Number(n) === 'number' ? n = Number(n) : n = 0
    return {
        get count(){return n++;}
        //setter不允许n递减
        set count(m){
            if( m >= n) n = m;
            else throw Error('can only be set to l larger value')
        }
    }
}
```

需要注意的是这个版本的counter()函数并未声明局部变量，只是用参数n来保存私有状态，属性存取器方法可以访问n，这样的话counter()的函数就能指定私有变量的初始值了。

例8-4是这种使用闭包技术来共享私有状态的通用做法。下面的例子定义了一个私有变量，以及两个嵌套函数来获取和设置这个变量的值。它将这些嵌套函数添加为指定对象的方法：

例8-4：利用闭包实现的私有属性存取器方法
```javascript
//这个函数给对象o增加了属性存取器方法
//方法名给get<name>和set<name>
//如果提供一个判定函数，setter方法就会用它来检测参数的合法性，然后存储它
//如果判定函数返回false,setter方法抛出一个异常
//
//这个韩式一个非同寻常之处就是getter和setter函数所操作的属性并没有存储在对象o中
//相反，这个值仅仅是保存在函数的局部变量中
//getter和setter方法同样是局部函数，因此可以访问这个局部变量
//也就是说，对于两个存取器方法来说这个变量是私有的
//没有方法绕过存取器方法来设置和修改这个值

function addPrivateProperty(o, name, predicate){
    var value;  //这是一个属性值
    
    //getter方法返回这个值
    o['get' + name] = function(){
        return value;
    }
    
    //setter方法首先检测值是否合法，不合法则抛出异常
    o['set' + name] = function(v){
        if(predicate && !predicate(v)){
            throw Error()
        }else{
            value = v
        }
    }
}

var o = {};
addPrivateProperty(o, "Name", function(x){return typeof x == 'string'});

o.setName('James');
console.log(a.getName());
```

在同一个作用域链中定义两个闭包，这两个闭包共享同样的私有变量或变量。但是要小心那些不希望共享的变量往往共享给了其他闭包。

```javascript
function constfunc(v){return function(){return v}}

var funcs = [];
for(var i = 0; i < 10; i++){
    funcs[i] = constfunc(i);
}

funcs[5]() //-->5
```
这段代码利用循环创建了很多歌闭包，当写这种类似的代码是会翻一个错误：试图将循环代码移入定义这个闭包的函数之内：

```javascript
//返回一个函数组成的数组，它们的返回值是0-9
function constfuncs(){
    var funcs = [];
    for(var i = 0; i < 10; i++){
        funcs[i] = function(){ return i };
    }
    return funcs
}

var funcs = consfuncs();
funcs[5]()  //10
```

上面这段代码创建了10个闭包，但是这些闭包都是在同一个函数调用中定义的，因此它们共享变量i。当constfuncs()返回时，变量i的值是10，所有的闭包都共享这一个值，因此，数组中的函数的返回值都是同一个值。
、关联到闭包的作用域链都是“活动的”，嵌套的函数不会讲作用域内的私有成员复制一份，也不会对所绑定的变量生成静态快照。

书写闭包时还需注意的是，this是js的关键字，而不是变量。正如之前讨论的，每个函数调用都包含一个this值，如果闭包在外部函数里是无法访问this的，除非在外部函数将this转存为一个变量：

```javascript
var self = this;
```

绑定arguments的问题与之类似。arguments并不是一个关键字，倒在调用每个函数时都会自动声明它，由于闭包具有自己所绑定的arguments，因此闭包内饰无法直接访问外部函数的arguments的，除非像this一样处理：

```javascript
var outerArguments = arguments;
```

#### 函数属性、方法和构造函数
在js程序中，函数是值。对函数执行typeof运算会返回字符串"function"，但函数是js重特殊的对象。也可以拥有属性和方法。甚至可以用Function()构造函数来创建新的函数对象。

##### length属性
在函数体里，arguments.length表示传入函数的实参的个数。而函数本身的length属性则有这不同含义。函数的length是只读属性，它代表函数形参的数量。这是的参数指的是“形参”而非实参，也就是函数定义时给的参数个数。

```javascript
//这个函数使用arguments.callee，因此不能在严格模式下工作
function check(args){
    var actual = args.length,           //实参的真实个数
        expected = args.callee.length   //期望的实参个数
    if(actual !== expected){
        throw Error('...')              //如果不同则抛出异常
    }
}

function f(x, y, z){
    check(arguments);                   //检查实参个数和期望的实参个数是否一致
    return x + y + z;                   //再执行后续逻辑
}
```

##### prototype属性
每一个函数都包含一个prototype属性，这个属性是指向一个对象的引用，这个对象称为“原型对象”。每一个函数都包含不同的原型对像。当将函数用作构造函数的时候，新创建的对象会从原型上继承属性。6.1.3节讨论了原型和prototype属性。

##### call()方法和apply方法()

call()和apply()的第一个实参是要调用函数的木对象，它是调用上下文在函数体内通过this来获得对它的引用。

```javascript
f.call(o);
f.apply(o)
```

在ECMAscript5的严格模式中，call()和apply()的第一个实参都会变成this的值，哪怕传入的实参是原始值甚至是null或undefined。在非严格模式中，传入null和undefined会被全局对象代替。而其他原始值则会被相应的包装对象所带一。

call()和apply()的不同之处在于传入的实参，call是按顺序传入实参，而apply则以数组形式传入实参

需要注意的是，传入apply()的参数数组可以是类数组对象也可以是真是数组，实际上，可以将当前函数的arguments数组直接传入apply()来调用另一个函数：

##### bind()方法

bind()方法是ECMAscript5中新增的方法。这个方法主要作用是将函数绑定到某个对象。

```javascript
//这是待绑定的函数
function f(y){ return this.x + y }
var o = { x: 1 },           //将要绑定的对象
    g = f.bind(o);          //通过调用g(x)来调用o.f(x)

g(2)    //-->3
```

还可以通过以下代码实现这种绑定：
```javascript
function bind(f, o){
    if(f.bind) return f.bind();
    else return function(){
        return f.apply(o, arguments);
    }
}
```

zaiECMAScript5中的bind()方法不仅仅是将函数绑定至一个对象，它还附带一些其他应用：除了第一个实参以外，传入bind()的实参也会绑定至this，这个附带的应用是一种常见的函数式编程技术，有时也被成为“柯里化”。
```javascript
var sum = function(x, y){
    return x + y    //返回两个参数的和值
},
    succ = sum.bind(null, 1);

succ(2) //-->3: x绑定到1，并传入2作为实参y

function f(y, z){
    return this.x + y + z;  //另外一个做累加计算的函数
}
var g = f.bind({x: 1}, 2);
g(3)    //-->6: this.x绑定到1，y绑定2，z绑定到3
```

我们可以绑定this的值并在es3中实现这个附带的应用。例8-5中模拟实现了标准的bind()方法

注意，我们将这个方法另存为Function.prototype.bind,以便所有的函数对象都继承他。

例8-5：es3版本的bind()方法实现
```javascript
if(!Function.prototype.bind){
    Function.prototype.bind = function(o/* ,args*/){
        //将this和arguments的值保存至变量中
        //以便在后面的嵌套函数中可以使用
        var self = this,
            boundArgs = arguments;
            
        //bing()方法的返回值是一个函数
        return function(){
            //创建一个实参列表，将传入bind()的第二个及后续的实参都传入这个函数
            var args = [],
                i;
            for(i = 1; i < boundArgs.length; i++){
                args.push(boundArgs[i]);
            };
            
            for(i = 0; i < arguments.length; i++){
                args.push(arguments[i])
            };
            
            //现在将self作为o的方法来调用，传入这些实参
            return self.apply(o, args);
        }
    }
}
```

bind()方法返回的函数是一个闭包，这个闭包的外部函数中声明了self和boundArgs变量，这两个变量在闭包里用到。尽管定义闭包的内部函数已经从外部函数中返回，而且调用这个闭包逻辑的时刻要在外部函数返回之后（在闭包中照样可以正确访问这两个变量）。

ECMAScript5定义的bind()方法也有一些特性是es3代码无法模拟的。

bing()方法返回的是一个函数对象，这个函数对象的length属性是绑定函数的形参个数减去绑定实参的个数（length的属性不能小于零）。

再者，es5的bind()可以顺带用构造函数。如果bind()返回的函数用作构造函数，将忽略传入bind()的this，原始函数就会以构造函数的形式调用，其实参也已经绑定。有bind()方法所返回的函数并不包含prototype属性(普通函数固有的prototype属性是不能删除的)，并且将这些绑定的函数用作构造函数所创建的对象从原始的未绑定的构造函数中继承prototype。

同样，在使用instanceof运算符时，绑定构造函数和未绑定构造函数并无两样。

##### toString()方法

函数也有toString()方法，实际上，大多数的toString()方法的实现都是返回函数的完整源码，内置函数则返回"[ native code ]"的字符串作为函数体

##### Function()构造函数

不管是用过函数定义语句还是函数直接量表达书，函数的定义都要使用function关键字。单数函数还可以通过Function()构造函数来定义：

```javascript
var f = new Function("x", "y", "return x*y");
var f = function(x, y){ return x*y };       //这两个函数几乎等价
```

Function()构造函数可以传入任意数量的字符串实参,最后一个实参多表示的文本就是函数体；它可以包含任意的JavaScript语句。传入构造函数的其他所有的实参字符是指定函数的形参名称的字符串。如果定义的函数不包含任何参数，只需给构造函数简单的传入一个字符串--函数体--即可

Function()构造函数并不需要通过传入实参以指定函数名。就像函数直接量一样，Function()构造函数创建一个匿名函数。

关于Function()构造函数有几个点要注意：

- Function()构造函数允许Javascript在运行时动态的创建并编译函数
- 每次调用Function()构造函数都会解析函数体，并创建新的函数对象。如果是一个循环或者多次调用的函数中执行这个构造函数，执行效率会受到影响。相比之下，循环=中的潜逃函数和函数定义表达式不会每次执行时都重新编译。
- 最后一点，它所创建的函数并不是使用词法作用域，相反，**函数体代码的编译总是会在顶层函数执行**：

```javascript
var scope = "global";
function constructFunction(){
    var scope = "local";
    return new Function('return scope');    //无法捕获局部作用于
}

constructFunction()();  //-->'global'
```

我们可以将Function()构造函数认为是在全局作用域中执行的eval()，Function()构造函数在实际编程过程中很少用。

##### 可调用的对象

在7.11节中提到的“类数组对象”并不是真正的数组，但大部分场景下，可以将其当做数组来对待。“可调用的对象”是一个对象，可以在函数调用表达式中调用这个对象。所有的函数都是可调用的，但并非所有可调用对象都是函数。

可调用对象在两个js实现中不能算作函数。首先，IE Web浏览器（ie8之前的版本）实现了客户端方法（如document.getElementById()）,使用了可调用的宿主对象，而不是内置函数对象。IE中的这些方法在其他浏览器中也存在，但它们本质上不是Function对象。IE9将它们实现为真正的函数，因此这类可调用的对象将越来越罕见。

另外一个常见的可调用对象是RegExp，可以直接调用RegExp对象。这比调用它的exec()方法更快一些。在js中这是一个非标准特性。

对RegExp执行typeof，有的返回'function'有的返回'object'

下面的函数可以检测一个对象是否是真正的函数对象：
```javascript
function isFunction(x){
    return Object.prototype.toString.call(x) === '[object function]'
}
```

#### 函数式编程

和Lisp、Haskell不同，JavaScript并非函数式编程语言，但是在js中可以像操控对象一样操控函数。

es5中的数组方法（如map和reduce）就适用于函数式编程。

##### 使用函数处理数组

假设有一个数组，元素都是数字。用非函数式编程的风格来和函数式编程的风格来计算平均值和标准差：
```javascript
var data = [1, 1, 3, 5, 5],
    total = 0;
    
for(var i = 0; i < data.length; i++){
    total += data[i]
}

var mean = total/data.length;

//计算标准差
total = 0;
for(var i = 0; i < data.length; i++){
    var deviation = data[i] - mean;
    total = deviation*deviation
}

var stddev = Math.sqrt(total/(data.length - 1));
```

用数组方法map()和reduce()同样可以实现：
```javascript
var sum = function(x, y){ return x + y },
    square = function(x){ return x * x },
    data = [1, 1, 3, 5, 5],
    mean = data.reduce(sum)/data.length,
    deviations = data.map(function(x){ return x - mean }),
    stddv = Math.sqrt(deviations.map(square),reduce(sum)/(data.length))
```

##### 高阶函数

高阶函数就是操作函数的函数，它接收一个或多个函数作为参数，并返回一个新的函数：

```javascript
function not(f){
    return function(){                              //返回一个新函数
        var result = f.apply(this, arguments);      //调用f()
        return !result;                             //对结果求反
    }
}

var even = function(x){
    return x%2 === 0;                               //判断x是否是偶数
}

var odd = not(even);                                //一个新的函数，所做的事情和even相反

[1,3,5,7].every(odd);                               //true每个都是奇数
```

not()函数就是一个高阶函数，它接收一个函数作为参数，并返回一个新函数。看下一个mapper()函数：

```javascript
function mapper(f){
    return function(a){
        return map(a, f)
    }
}

var increment = function(x){ return x + 1 }
var incrementer = mapper(increment);

incrementer([1, 2, 3]); //-->[2,3,4]
```

还有个更常见的例子：
```javascript
//返回一个f(g(...))的新函数
//返回的函数h()将它所有的实参传入g(),然后将g()的返回值传入f()
//调用f()和g()时的this值和调用h()时的this值是同一个this
function compose(f, g){
    return function(){
        //需要给f()传入一个参数，所以使用f()的call()方法
        //需要给g()传入更多的参数，所以用g()的apply()方法
        return f.call(this, g.apply(this, arguments))
        
    }
}

var square = function(x){ return x * x },
    sum = function(x, y){ return x + y },
    squareofsum = compose(square, sum);
    
squareofsum(2, 3);  //25
```

##### 不完全函数

函数f()的bind()方法返回一个新函数， 给新函数传入特定的上下文和一组指定的参数，然后调用函数f()。我们说它把函数“绑定至”对象并传入一部分参数。传入bind()的参数都是放在传入原始函数的实参列表开始的地方。

我们可以将传入bind()的实参放在右侧：
```javascript
//实现一个工具函数将数组对象（或对象）转换成真正的数组
//将arguments对象转换为数组
function array(a, n){
    return Array.prototype.slice.call(a, n || 0);
}

//这个函数的实参传递到左侧
function partialLeft(f /*, ...*/){
    var args = arguments;   //保存外部的实参数组
    return function(){
        var a = array(args, 1);
        a = a.concat(array(arguments));
        
        return f.apply(this, a)
    }
}

//这个函数的实参传递至右侧
function partialRight(f /*, ...*/){
    var args = arguments;
    return function(){
        var a = array(arguments);
        a = a.cancat(array(args, 1));
        
        return f.apply(this, a);
    }
}

//这个函数的实参被用做模板
//实参列表中的undefined值都被填充
function partial(f /*, ...*/){
    var args = arguments;
    return function(){
        var a = array(args, 1)
        var i = 0,
            j = 0;
        for(; i < a.length; i++){
            if(a[i] === undefined){
                a[i] = arguments[j++]
            }
        }
        a = a.concat(array(arguments, j));
        return f.apply(this, a)
    }
    
}

//这函数带头三个实参
var f = function(x, y, z){
    return x * (y - z);
}

//注意三个不完全调用之间的区别
partialLeft(f, 2)(3, 4);            //-2
partialRight(f, 2)(3, 4);           //6
partial(f, undefined, 2)(3, 4);     //-6
```


##### 记忆