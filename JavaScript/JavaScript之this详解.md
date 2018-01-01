
[TOC]

JavaScript作为一种脚本语言身份的存在，因此被很多人认为是简单易学的。然而情况恰恰相反，JavaScript支持函数式编程、闭包、基于原型的继承等高级功能。由于其运行期绑定的特性，JavaScript 中的 this 含义要丰富得多，它可以是全局对象、当前对象或者任意对象，这完全取决于函数的调用方式。JavaScript中函数的调用有以下几种方式：作为对象方法调用，作为函数调用，作为构造函数调用，和使用 apply 或 call 调用。本文就采撷些例子以浅显说明在不同调用方式下的不同含义。

## 全局的this
全局this一般指向全局对象，浏览器中的全局对象就是 window。例如：

```
console.log(this.document === document); 
console.log(this === window); 
this.a = 91;
console.log(window.a); 
```

## 一般函数的 this

```
function f1 () {
    return this;
}
console.log(f1() === window);
```
可以看到一般函数的 this 也指向 window，在 nodeJS 中为 global object

```
function f2 () {
    "use strict";
    return this;
}
console.log(f1() === undefined);
```

严格模式中，函数的 this 为 undefined,因为严格模式禁止this关键字指向全局对象；对于js“严格模式”具体可以看阮一峰先生的Javascript 严格模式详解

## 作为对象方法的函数的 this

```
var o = {
    prop: 37,
    f: function() {
        return this.prop;
    }
};
console.log(o.f()); 
```
上述代码通过字面量创建对象 o。
f 为对象 o 的方法。这个方法的 this 指向这个对象，在这里即对象 o。

```
var o = {
    prop: 37
};

function independent() {
    return this.prop;
}
o.f = independent;
console.log(o.f()); // 37
```

上面的代码，创建了对象 o，但是没有给对象 o，添加方法。而是通过 o.f = independent 临时添加了方法属性。这样这个方法中的 this 同样也指向这个对象 o。

## 作为函数调用
函数也可以直接被调用，此时 this 绑定到全局对象。在浏览器中，window 就是该全局对象。比如下面的例子：函数被调用时，this被绑定到全局对象，接下来执行赋值语句，相当于隐式的声明了一个全局变量，这显然不是调用者希望的。

```
function makeNoSense(x) { 
    this.x = x; 
} 
makeNoSense(5); 
x;
```

对于内部函数，即声明在另外一个函数体内的函数，这种绑定到全局对象的方式会产生另外一个问题。以下面moveTo方法为例，内定义两个函数，分别将 x，y 坐标进行平移。结果可能出乎大家意料，不仅 point 对象没有移动，反而多出两个全局变量 x，y。

```
var point = { 
    x : 0, 
    y : 0, 
    moveTo : function(x, y) { 
        var moveX = function(x) { 
            this.x = x;
        }; 
        var moveY = function(y) { 
            this.y = y;
        }; 
        moveX(x); 
        moveY(y); 
    } 
}; 
point.moveTo(1, 1); 
console.log(point.x) 
console.log(point.x) 
console.log(x)       
console.log(y)   
```    

这属于 JavaScript 的设计缺陷，正确的设计方式是内部函数的this应该绑定到其外层函数对应的对象上，为了规避这一设计缺陷，聪明的JavaScript程序员想出了变量替代的方法，约定俗成，该变量一般被命名为 that。

## 对象原型链上的this

```
var o = {
    f: function() {
        return this.a + this.b;
    }
    };
var p = Object.create(o);
p.a = 1;
p.b = 2;
console.log(p.f()); 
```
通过 var p = Object.create(o) 创建的对象，p 是基于原型 o 创建出的对象。
p 的原型是 o，调用 f() 的时候是调用了 o 上的方法 f()，这里面的 this 是可以指向当前对象的，即对象 p。

## get/set 方法与 this

```
function modulus() {
    return Math.sqrt(this.re * this.re + this.im * this.im);
    }
var o = {
    re: 1,
    im: -1,
    get phase() {
        return Math.atan2(this.im, this.re);
    }
    };
Object.defineProperty(o, 'modulus', {
    get: modulus,
    enumerable: true,
    configurable: true
});
```
console.log(o.phase, o.modulus); 
get/set 方法中的 this 也会指向 get/set 方法所在的对象的。

## 构造器中的 this

```
function MyClass() {
    this.a = 25;
}
var o = new MyClass();
console.log(o.a); 
```
new MyClass() 的时候，MyClass()中的 this 会指向一个空对象，这个对象的原型会指向 MyClass.prototype。MyClass()没有返回值或者返回为基本类型时，默认将 this 返回。

```
function C2() {
    this.a = 26;
    return {
        a: 24
    };
}
o = new C2();
console.log(o.a); 
```

因为返回了对象，将这个对象作为返回值

## call/apply 方法与 this

```
function add(c, d) {
    return this.a + this.b + c + d;
}
var o = {
    a: 1,
    b: 3
};
add.call(o, 5, 7); 
add.apply(o, [10, 20]); 
function bar() {
    console.log(Object.prototype.toString.call(this));
}
bar.call(7); 
bar.call(); 
bar.call("7");
bar.call(true);
console.log(add.call(o,5,7));
```

## bind 方法与 this

```
function f() {
    return this.a;
}
var g = f.bind({
    a: "test"
});
console.log(g()); 
var o = {
    a: 37,
    f: f,
    g: g
};
console.log(o.f(), o.g()); 
```
绑定之后再调用时，仍然会按绑定时的内容走，所以 o.g() 结果是 test
##JavaScript中this的些许看似怪异现象

```
<body></body>
    <a href="#" onclick="alert(this.tagName);">click me</a> 
    <a href="javascript:alert(this.tagName);">click me</a>  
    <a href="javascript:alert(this==window);">click me</a>  
    <input type="button" value="this demo">
    
var name = 'somebody';
var angela = {
    name: 'angela',
    say: function () {
        alert("I'm " + this.name);
    }
    };
var btn = document.getElementById('btn');
```

## setTimeout和setInterval也会改变this的指向

```
angela.say();
setTimeout(angela.say, 1000);  
setInterval(angela.say, 1000); 
```

这是因为setTimeout等函数内this默认的指向是Window。这一点类似于曾探所写的《JavaScript设计模式与开发实践》中提到的丢失的this。看下面代码：

```
var obj ={
    myName = 'jeff',
    getName = functon(){
        return this.myName;
    }
    };
console.log(obj.getName()); 
var getName2 = obj.getName;
console.log( getName2() ); 
```

## on…也会改变this的指向

```
angela.say(); 
btn.onclick = angela.say; 
```

## click等回调也会改变this指向

```
$("#btn").click = angela.say;  
$("#btn").click(angela.say);   
```

如果在say中用了this，this会绑定在angela上么？显然这里不是，赋值以后，函数是在回调中执行的，this会绑定到$(“#btn”)元素上。这个函数被完整复制到onclick属性（现在成为了函数）。因此如果这个even thandler被执行，this将指向HTML元素;因此，结果显示的是”I’m button”。而，匿名函数可以调整this指向,EG:

```
$("#btn").click(function(){ 
    angela.say();  
});
```

这是JavaScript新手们经常犯的一个错误，为了避免这种错误，许多JavaScript框架都提供了手动绑定 this 的方法。比如Dojo就提供了lang.hitch，该方法接受一个对象和函数作为参数，返回一个新函数，执行时this绑定到传入的对象上。使用 Dojo，可以将上面的例子改为： 1

```
button.onclick = lang.hitch(angela, angela.say);
```

其实在我们使用比较多的jQuery也提供了对应的解决方案：jQuery.proxy(function, scope).返回一个新函数，并且这个函数始终保持了特定的作用域。其作用跟Dojo就提供了lang.hitch类似，具体可以参考这里。其中有一例如下：

```
<div>Click Here<\ div=""></\></div>
var obj = {
  name: "John",
  test: function() {
    alert( this.name );
    $("#test").unbind("click", obj.test);
  }
};
$("#test").click( jQuery.proxy( obj, "test" ) );
```
在新版的 JavaScript 中，已经提供了内置的 bind 方法供大家使用。
匿名函数调整this指向比如：

```
setTimeout(function () { angela.say(); }, 1000); 
setInterval(function () { angela.say(); }, 1000) 
btn.onclick = function () { angela.say(); };     
setTimeout(function () { alert(this == window); }, 1000);
btn.onclick = function () { alert(this == btn); }
```

匿名函数赋值给了click属性(好吧，现在成了函数)，此时这个匿名函数指向的即是Html属性。因此所调用的函数(比如angela.say())this上下文没有被更改，所以其打印出来的结果就是’I’m angela’。事实上，也用这样的方法来消解this在回调函数中不堪使用的’特色’(毕竟一旦出了问题难以调试)。

```
$("#btn").click(function(){ 
    if(window == this){
        alert("window == this");
    }else{
        alert("window != this")  
    }
    alert(this.name); 
    angela.say();    
});
```

## 将this指向的对象保存到变量(一般用that)

```
var mydemo = {
    name: 'angela',
    say: function () { alert("I'm " + this.name); },
    init: function () {
        var that = this;
        document.getElementById('btn').onclick = function () {
            that.say();  
            this.say();  
        }
    }
    };
mydemo.init();
```

## 第三方库or框架中的this
比如，使用backbone框架中events时间回调中的this，其指向的就是对应的视图，而不是Dom元素，因为该回调时通过events哈希绑定的，实质上也是自对应视图那里callback到对应的函数;

## Javascript中的eval 方法
JavaScript 中的 eval 方法可以将字符串转换为 JavaScript 代码，使用 eval 方法时，this 指向哪里呢？答案很简单，看谁在调用 eval 方法，调用者的执行环境（ExecutionContext）中的 this 就被 eval 方法继承下来了。(悪，还没用过,有待实践下)！ 但是：在严格模式之下，eval的作用域也被改变了。正常模式下，eval语句的作用域，取决于它处于全局作用域，还是处于函数作用域。严格模式下，eval语句本身就是一个作用域，不再能够生成全局变量了，它所生成的变量只能用于eval内部。

```
"use strict";
var x = 2;
console.info(eval("var x = 5; x")); 
console.info(x); 
```

## Es6箭头函数中的 this
随着时间的推移，Es6越发会成为新的主流，当然在之后肯定又有新的譬如正在制定的Es7等等；而对于本文 this 相关的，这Es6备受关注的的箭头函数( 例如()=>{} )，就很有必要谈及了。这种很简洁的使用，让函数内部this具有了上下文相同的执行环境；再也不写 var that = this 这样蹩脚的东东了（当然 that 在很多童鞋笔下可能是 _this , self等等，我的天哪）。也不用借助 eval 或者 $proxy(obj.xx ,obj) 这样蹩脚的第三方替代办法，酷爽。至于这箭头函数内部的this到底为何物，一两句也谈不清楚，可参见 ES6 箭头函数中的 this？你可能想多了（翻译），一探究竟。

后记：由于javascript的动态性（解释执行，当然也有简单的预编译过程），this的指向在运行时才确定，因此在只要足够留心其运行时的上下文，即可无痛挥霍this的强大。上面列举了这么些许，也是要说明另一个问题：即便知道了这个本质，到底还是需要了解各种环境的差异，方可运用自如。比如这篇JavaScript:万恶的this拿命来（一）文章还谈及：node脚本，还有REPL(“读取-求值-输出”循环（Read-Eval-Print Loop，简称REPL）)中使用this的些许区别，可以一览。

