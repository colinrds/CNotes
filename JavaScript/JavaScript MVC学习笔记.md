[toc]

## 初识JS MVC

### 什么是MVC

MVC 是一种设计模式，它将应用划分为3个部分：数据（模型）、展现层（视图）、用户交互层（控制器）。一个事件的发生是这样的过程：

- 用户和应用产生交互
- 控制器的事件处理器被触发
- 控制器从模型中请求数据，并将其交给视图
- 视图将数据呈现给用户

### 模型

模型用来存放应用的所有数据对象。

模型不必知晓视图和控制器的细节，模型只需包含数据及直接和这些数据相关的逻辑。

任何事件处理代码、视图模板，以及那些和模型无关的逻辑都应当隔离在模型之外。将模型和视图的代码混在一起，是违反MVC架构原则的。模型是最应该从你的应用中解耦出来的部分。

当控制器从服务器抓取数据或创建新的记录时，它就将数据包装成模型实例。也就是说，我们的数据是面向对象的（object oriented），任何定义在这个数据模型上的函数或逻辑
都可以直接被调用。

因此，不要这样做：

```
var user = users["foo"];
destroyUser(user);
```

上面的代码没有命名空间的概念，并且不是面向对象的。如果在应用中定义了另一个destoryUser()函数的话，两个函数就会产生冲突。我们应当确保全局变量和函数的个数尽可能少.

而要这样做：

```
var user = User.find("foo");
user.destroy();
```

上面的代码中，destory()函数是存放在命名空间User的实例中的。这种代码更加清晰，而且非常容易做继承，类似destory()的这种函数就不用在每个模型中都定义一遍了。

### 视图

视图层是呈现给用户的，用户与之产生交互。在JavaScript 应用中，视图大都是由HTML、CSS和JavaScript模板组成的。除了模板中简单的条件语句之外，视图不应当包含任何其他逻辑。

这并不是说MVC不允许包含视觉呈现相关的逻辑，只要这部分逻辑没有定义在视图之内即可。我们将视觉呈现逻辑归类为“视图助手”（helper）：和视图有关的独立的小型工具函数。

反例——formatDate()函数直接插入视图：

```
// template.html
<div>
    <script>
        function formatDate(date) {
            /* ... */
        };
    </script>

    ${ formatDate(this.date) }
</div>
```

应该这样做——所有视觉呈现逻辑都包含在helper变量中，这是一个命名空间，可以防止冲突并保持代码清晰、可扩展：

```
// helper.js
var helper = {};
helper.formatDate = function(){ /* ... */ };

// template.html
<div>
    ${ helper.formatDate(this.date) }
</div>
```

### 控制器

控制器是模型和视图之间的纽带。控制器从视图获得事件和输入，对它们进行处理（很可能包含模型），并相应地更新视图。当页面加载时，控制器会给视图添加事件监听，比如监听表单提交或按钮点击。然后，当用户和应用产生交互时，控制器中的事件触发器就开始工作了。

下面用jQuery实现一个例子：

```
var Controller = {};

// 使用匿名函数来封装一个作用域
(Controller.users = function($){

    var nameClick = function(){
        /* ... */
    };

    // 在页面加载时绑定事件监听
    $(function(){
        $("#view .name").click(nameClick);
    });
})(jQuery);
```

上面的代码创建了users控制器，这个控制器是放在Controller变量下的命名空间。然后用了一个匿名函数封装了一个作用域，以避免对全局作用域造成污染。当页面加载时，程序给视图元素绑定了点击事件的监听。

## 类的使用

### JavaScript的类

JavaScript是基于原型的语言，没有包含内置的类，但是通过JavaScript可以轻易地模拟出经典的类。JavaScript中有构造函数和 new 运算符。构造函数用来给实例对象初始化属性和值。任何JavaScript函数都可以用做构造函数，构造函数必须使用new 运算符作为前缀来创建新的实例。

new 运算符改变了函数的执行上下文，同时改变了return 语句的行为。实际上，使用new和构造函数和传统的实现了类的语言中的使用方法是很类似的：

```
var Person = function(name) {
  this.name = name;
};

// 实例化一个Person
var alice = new Person('alice');

// 检查这个实例
assert( alice instanceof Person );
```

构造函数的命名通常使用驼峰命名法，首字母大写，以此和普通的函数区分开来，这是一种习惯用法。

不要省略new前缀的方式来调用构造函数:

```
// 不要这么做!
Person('bob'); //=> undefined
```

这个函数只会返回undefined，并且执行上下文是window（全局）对象，你无意间创建了一个全局变量name。调用构造函数时不要丢掉new关键字。

当使用new关键字来调用构造函数时，执行上下文从全局对象（window）变成一个空的上下文，这个上下文代表了新生成的实例。因此，this关键字指向当前创建的实例。尽管理解起来有些绕，实际上其他语言内置类机制的实现也是如此。默认情况下，如果构造函数中没有返回任何内容，就会返回this——当前的上下文。要不然就返回任意非原始类型的值。

### 创建自己的类模拟库

```
var Class = function(){

    var klass = function(){
      this.init.apply(this, arguments);
    };

    klass.prototype.init = function(){};

    return klass;
};

var Person = new Class;

Person.prototype.init = function(){
  // 基于Person 的实例做初始化
};

// 用法：
var person = new Person;
```

由于 JavaScript 2 规范从未被实现过，class 一直都是保留字。最常见的做法是将变量名 class 改为 _class 或 klass。

### 给类添加函数

在JavaScript 中，在构造函数中给类添加函数和给对象添加属性是一样的：

```
Person.find = function(id){ /*...*/ };
var person = Person.find(1);
```

要想给构造函数添加实例函数，则需要用到构造函数的prototype ：

```
Person.prototype.breath = function(){ /*...*/ };
var person = new Person;
person.breath();
```

一种常用的模式是给类的 prototype 起一个别名fn，写起来也更简单：

```
Person.fn = Person.prototype;
Person.fn.run = function(){ /*...*/ };
```

这种模式在jQuery 的插件开发中是很常见的，将函数添加至jQuery.fn 中也就相当于添加至jQuery 的原型中。

### 给实现了类机制的类库添加方法

给类添加属性和给构造函数添加属性是一样的。

直接给类设置属性和设置其静态成员是等价的：

```
var Person = new Class;

// 直接给类添加静态方法
Person.find = function(id){ /* ... */ };

// 这样我们可以直接调用它们
var person = Person.find(1);
给类的原型设置的属性在类的实例中也是可用的：

var Person = new Class;

// 在原型中定义函数
Person.prototype.save = function(){ /* ... */ };

// 这样就可以在实例中调用它们
var person = new Person;
person.save();
```

这样很难一眼就分辨出类的静态属性和实例的属性。因此我们采用另外一种不同的方法来给类添加属性，这里用到了两个函数extend() 和include() ：

```
var Class = function () {
    var klass = function () {
        this.init.apply(this, arguments);
    };

    klass.prototype.init = function () {};

    // 定义 prototype 的别名
    klass.fn = klass.prototype;

    // 定义类的别名
    klass.fn.parent = klass;

    // 给类添加属性
    klass.extend = function (obj) {
        var extended = obj.extended;
        for (var i in obj) {
            klass[i] = obj[i];
        }
        if (extended) 
            extended(klass)
    };

    // 给实例添加属性
    klass.include = function (obj) {
        var included = obj.included;
        for (var i in obj) {
            klass.fn[i] = obj[i];
        }
        if (included) 
            included(klass)
    };
    return klass;
};
```

这段代码使用extend() 函数来生成一个类，这个函数的参数是一个对象。通过迭代将对象的属性直接复制到类上：

```
var Person = new Class;
Person.extend({
    find: function(id) { /* ... */ },
    exists: functions(id) { /* ... */ }
});
var person = Person.find(1);
include() 函数的工作原理也一样，只不过不是将属性复制至类中，而是复制至类的原型中。换句话说，这里的属性是类实例的属性，而不是类的静态属性。

var Person = new Class;
Person.include({
    save: function(id) { /* ... */ },
    destroy: functions(id) { /* ... */ }
});
var person = new Person;
person.save();
```

同样地，这里的实现支持extended 和included 回调。将属性传入对象后就会触发这两个回调：

```
Person.extend({
    extended: function(klass) {
        console.log(klass, " was extended!");
    }
});
```

这种写法已经可以支持模块了。模块是可重用的代码段，用这种方法可以实现各种继承，用来在类之间共享通用的属性。

```
var ORMModule = {
    save: function(){
        // 共享的函数
    }
};

var Person = new Class;
var Asset = new Class;
Person.include(ORMModule);
Asset.include(ORMModule);
```

### 基于原型的类继承

JavaScript 是基于原型的编程语言，原型用来区别类和实例。原型是一个“模板”对象，它上面的属性被用做初始化一个新对象。任何对象都可以作为另一个对象的原型对象，以此来共享属性。实际上，可以将其理解为某种形式的继承。

当读取一个对象的属性时，JavaScript 首先会在本地对象中查找这个属性，如果没有找到，JavaScript 开始在对象的原型中查找，若还未找到还会继续查找原型的原型，直到查找到Object.prototype。如果找到这个属性，则返回这个值，否则返回undefined。例如，给 Array.prototype 添加了属性，那么所有的 JavaScript 数组都具有了这些属性。

让子类继承父类的属性的方法：

先定义一个构造函数，然后将父类的新实例赋值给构造函数的原型：

```
// 父，动物大类
var Animal = function(){};
Animal.prototype.breath = function(){
    console.log('breath');
};

// 子，狗类
var Dog = function(){};

// Dog 继承了Animal
Dog.prototype = new Animal;
Dog.prototype.wag = function(){
    console.log('wag tail');
};
```

检查继承是否生效了：

```
var dog1 = new Dog;
dog1.wag();
dog1.breath(); // 继承的属性
```

### 给“类”库添加继承

通过传入一个可选的父类来创建新类，这个可以作为创建类的基础模板：

```
var Class = function(parent){
    var klass = function(){
        this.init.apply(this, arguments);
    };

    // 改变klass 的原型
    if (parent) {
        var subclass = function() { };
        subclass.prototype = parent.prototype;
        klass.prototype = new subclass;
    };

    klass.prototype.init = function(){};

    // 定义别名
    klass.fn = klass.prototype;
    klass.fn.parent = klass;
    klass._super = klass.__proto__;
    /* include/extend 相关的代码…… */

    return klass;
};
```

如果将parent 传入Class 构造函数，那么所有的子类则共享同一个原型。这种创建临时匿名函数的小技巧避免了在继承类的时候创建实例，这里暗示了只有实例的属性才会被继承，而非类的属性【我没读懂这句和上面代码的关系，感觉已经晕了 = =】。设置对象的proto ；属性并不是所有浏览器都支持，类似Super.js的类库则通过属性复制的方式来解决这个问题，而非通过固有的动态继承的方式来实现。

通过给Class 传入父类来实现简单的继承：

```
var Animal = new Class;
Animal.include({
    breath: function(){
        console.log('breath');
    }
});

var Cat = new Class(Animal);

// 用法
var tommy = new Cat;
```

### 函数调用

在JavaScript中，函数和其他东西一样都是对象。和其他对象不同的是，函数可调用。函数内上下文，如this 的取值，取决调用它的位置和方法。

除了使用方括号可以调用函数之外，还有其他两种方法可以调用函数：apply() 和 call()。两者的区别在于传入函数的参数的形式。

apply() 函数有两个参数：第1个参数是上下文，第2个参数是参数组成的数组。如果上下文是null，则使用全局对象代替。例如：

```
function.apply(this, [1, 2, 3])
```

call()的第1个参数是上下文，后续是实际传入的参数序列:

```
function.call(this, 1, 2, 3);
```

JavaScript 中允许更换上下文是为了共享状态，尤其是在事件回调中。jQuery 在其API 的实现中就利用了apply() 和call() 来更改上下文，比如在事件处理程序中或者使用each() 来做迭代时。

```
$('.clicky').click(function(){
    // ‘this’指向当前节点
    $(this).hide();
});

$('p').each(function(){
    // ‘this’指向本次迭代
    $(this).remove();
});
```

为了访问原始上下文，可以将this 的值存入一个局部变量中，这是一种常见的模式，比如：

```
var clicky = {
    wasClicked: function(){
        /* ... */
    },

    addListeners: function(){
        var self = this;
        $('.clicky').click(function(){
            self.wasClicked()
        });
    }
};

clicky.addListeners();
```

可以用apply来将这段代码变得更干净一些，通过将回调包装在另外一个匿名函数中，来保持原始的上下文：

```
var proxy = function(func, thisObject){
    return(function(){
        return func.apply(thisObject, arguments);
    });
};

var clicky = {
    wasClicked: function(){
        /* ... */
    },
    addListeners: function(){
        var self = this;
        $('.clicky').click(proxy(this.wasClicked, this));
    }
};
```

上面的例子中在点击事件的回调中指定了要使用的上下文；jQuery中调用这个函数所用的上下文就可以忽略了。实际上jQuery也包含了实现了这个功能的API——jQuery.proxy() ：

```
$('.clicky').click($.proxy(function(){ /* ... */ }, this));
```

使用apply()和call()还有其他很有用的原因，比如“委托”。可以将一个调用委托给另一个调用，甚至可以修改传入的参数：

```
var App {
    log: function(){
        if (typeof console == "undefined") return;

        // 将参数转换为合适的数组
        var args = jQuery.makeArray(arguments);

        // 插入一个新的参数
        args.unshift("(App)");

        // 委托给console
        console.log.apply(console, args);
    }
};
```

这个例子中首先构建了一个参数数组，然后将参数添加进去，最后将这个调用委托给了console.log()。arguments变量是解释器内置的当前调用的作用域内用来保存参数的数组。但它并不是真正的数组，比如它是不可变的，因此需要通过jQuery.makeArray()将其转换为可用的数组。


### 控制“类”库的作用域

在类和实例中都添加proxy函数，可以在事件处理程序之外处理函数的时候保持类的作用域。下面是不用proxy的办法：

```
var Class = function(parent){
    var klass = function(){
        this.init.apply(this, arguments);
    };

    klass.prototype.init = function(){};
    klass.fn = klass.prototype;

    // 添加一个proxy 函数
    klass.proxy = function(func){
        var self = this;
        return(function(){
            return func.apply(self, arguments);
        });
    }

    // 在实例中也添加这个函数
    klass.fn.proxy = klass.proxy;
    return klass;
};
```

下面是用proxy()函数来包装函数，以确保它们在正确的作用域中被调用：

```
var Button = new Class;
    Button.include({
    init: function(element){
        this.element = jQuery(element);

        // 代理了这个click 函数
        this.element.click(this.proxy(this.click));
    },
    click: function(){ /* ... */ }
});
```

如果没有使用proxy将click()的回调包装起来，它就会基于上下文this.element来调用，而不是Button，这会造成各种问题。在新版本的JavaScript——ECMAScript 5（ES5）——中同样加入了bind()函数用以控制调用的作用域。bind()基于函数进行调用的，用来确保函数是在指定的this值所在的上下文中调用的。例如：

```
Button.include({
    init: function(element){
        this.element = jQuery(element);
        // 绑定这个click 函数
        this.element.click(this.click.bind(this));
    },
    click: function(){ /* ... */ }
});
```

这个例子和proxy()函数是等价的，它能确保click() 函数基于正确的上下文进行调用。老版本的浏览器不支持bind()，但可以手动实现它，兼容性也不错，直接扩展相关对象的原型，这样就可以像在ES5 中使用bind()，在任意浏览器中调用它。下面是一段实现了bind() 函数的代码：

```
if (!Function.prototype.bind) {
    Function.prototype.bind = function (obj) {
        var slice = [].slice,
        args = slice.call(arguments, 1),
        self = this,
        nop = function () {},
        bound = function () {
        return self.apply( this instanceof nop ? this : (obj || {}),
            args.concat(slice.call(arguments)));
        };

        nop.prototype = self.prototype;
        bound.prototype = new nop();
        return bound;
    };
}
```

如果浏览器原生不支持bind()，则仅仅重写了Function 的原型。现代浏览器则可以继续使用内置的实现。

对于数组来说这种“打补丁”式的做法非常有用，因为在新版本的JavaScript 中，数组增加了很多新的特性。可以使用es5-shi项目，它涵盖了ES5 中新增的尽可能多的特性。

### 添加私有函数

目前上面为“类”库添加的属性都是“公开的”，可以被随时修改。关于给“类”添加私有属性，JavaScript支持不可变属性，但在主流浏览器中并未实现，没办法直接利用这个特性。可以利用JavaScript 匿名函数来创建私有作用域，这些私有作用域只能在内部访问：

```
var Person = function(){};

(function(){
    var findById = function(){ /* ... */ };
    Person.find = function(id){

    if (typeof id == "integer")
        return findById(id);
    };
})();
```

上面的代码将类的属性都包装进一个匿名函数中，然后创建了局部变量（findById），这些局部变量只能在当前作用域中被访问到。Person变量是在全局作用域中定义的，可以在任何地方都能访问到。定义变量的时候不要丢掉var运算符，如果丢掉var就会创建全局变量。如果需要定义全局变量，可以在全局作用域中定义，或者定义为window的属性：

```
(function(exports){
    var foo = "bar";

    // 将变量暴露出去
    exports.foo = foo;
})(window);

assertEqual(foo, "bar");
```

### “类”库
jQuery 本身并不支持类，但通过插件的方式可以轻易引入类的支持，比如HJS。HJS 允许通过给$.Class.create传入一组属性来定义类：

```
var Person = $.Class.create({

    // 构造函数
    initialize: function(name) {
        this.name = name;
    }
});
```

可以在创建类的时候传入父类作为参数，这样就实现了类的继承：

```
var Student = $.Class.create(Person, {
    price: function() { /* ... */ }
});

var alex = new Student("Alex");
alex.pay();
```

可以直接给类挂载属性：

```
Person.find = function(id){ /* ... */ };
```

HJS 的API 中同样包含一些工具函数，比如clone() 和equal() ：

```
var alex = new Student("Alex");
var bill = alex.clone();
assert( alex.equal(bill) );
```

还有 Spine 同样实现了类，通过直接在页面中引入spine.js（http://maccman.github.com/spine/spine.js）来使用它：

```
<script src="http://maccman.github.com/spine/spine.js"> </script>
<script>
    var Person = Spine.Class.create();

    Person.extend({
        find: function() { /* ... */ }
    });

    Person.include({
        init: function(atts){
        this.attributes = atts || {};
        }
    });

    var person = Person.init();
</script>
```

## 事件的基本操作

### 监听事件

绑定事件监听的函数是addEventListener()，有 3 个参数：type（比如click），listener（比如callback）及seCapture。使用前两个参数可以给一个 DOM 元素绑定一个函数，当特定的事件（比如点击）被触发时执行这个函数：

```
var button = document.getElementById("createButton");
button.addEventListener("click", function(){ /* ... */ }, false);
```

可以使用removeEventListener() 来移除事件监听，参数和传入addEventListener() 的一样。如果监听的函数是匿名函数，没有任何引用指向它，在不销毁这个元素的前提下，这个监听是无法被移除的：

```
var div = document.getElementById("div");
var listener = function(event) { /* ... */ };
div.addEventListener("click", listener, false);
div.removeEventListener("click", listener, false);
```

带入listener函数的第 1 个参数是event对象，通过event象可以得到事件的相关信息，比如时间戳、坐标和事件宿主元素（target）。它同样包含很多方法来停止事件冒泡和阻止事件的默认行为。

不同的浏览器对事件类型的支持有些差异，但所有现代浏览器都支持这些事件：

- click
- dblclick
- mousemove
- mouseover
- mouseout
- focus
- blur
- change （表单输入框特有）
- submit （表单特有）


### 事件顺序

如果一个节点和它的一个父节点都绑定了相同事件类型的回调，当事件触发时哪个回调会先执行？

浏览器不同有不同的默认执行顺序，分为两种：

- 事件捕捉（capturing），从顶层的父节点开始触发事件，从外到内传播。
- 事件冒泡（bubbling），从最内层的节点开始触发事件，逐级冒泡直到顶层节点，从内向外传播。

W3C将对这两种事件模型的支持都加入标准规范之中。根据W3C型，事件首先被目标元素所捕捉，然后向上冒泡。

可以自行选择要注册的事件处理程序的调用类型，捕捉或冒泡，通过给addEventListener()传入第3个参数useCapture 来设置。如果addEventListener() 的最后一个参数是true，事件处理程序以捕捉模式触发；如果是false，事件处理程序以冒泡模式触发：

```
// 给最后一个参数传入false，来设置事件冒泡
button.addEventListener("click", function(){ /* ... */ }, false);
```

大多数情况下是使用冒泡模式，如果对此不太确定，可以给addEventListener()`的最后一个参数传入false。

### 取消事件冒泡

当事件冒泡时，可以通过stopPropagation()数来终止冒泡，这个函数是event对象中的方法。比如这段代码，任何父节点的事件回调都不会触发：

```
button.addEventListener("click", function(e){
    e.stopPropagation();
    /* ... */
}, false);
```

jQuery 还支持stopImmediatePropagation()函数，用来阻止后续所有的事件触发——哪怕这些事件是注册在同一个节点元素上的。

### 阻止浏览器的默认行为

浏览器给事件赋予了默认行为。比如，点击一个链接时，浏览器的默认行为是载入新页面，当点击一个复选框时，浏览器会将其选中（或取消选中）。在事件传播阶段（之后）会触发这些默认行为，在任何一个事件处理程序中都可以阻止默认行为。可以通过调用event对象的preventDefault()函数来阻止默认行为，也可以通过在回调中返回false来实现同样的效果：

```
bform.addEventListener("submit", function(e){
    /* ... */
    return confirm("Are you super sure?");
}, false);
```

如果调用confirm()返回false（用户点击了对话框的取消按钮），这个事件回调函数就返回false，这样就会取消事件，阻止表单的提交。

## 事件操作的拓展

### 事件对象

event 对象还包含很多有用的属性。W3C 范中包含的大部分属性都列在下面，更多信息参照完整的标准规范。

事件类型：

- bubbles ：布尔值，表示事件是否通过DOM 以冒泡形式触发。

事件发生时，反映当前环境信息的属性：

- button ：表示（如果有）鼠标所按下的按钮。
- ctrlKey ：布尔值，表示Ctrl 键是否按下。
- altKey ：布尔值，表示Alt 键是否按下。
- shiftKey ：布尔值，表示Shift 键是否按下。
- metaKey ：布尔值，表示Meta 键是否按下。

表示键盘事件的属性：

- isChar ：布尔值，表示当前按下的键是否表示一个字符。
- charCode ：表示当前按键的unicode 值（仅对keypress 事件有效）。
- keyCode ：表示非字符按键的unicode 值。
- which ：表示当前按键的unicode 值，不管当前按键是否表示一个字符。

事件发生时的环境参数：

- pageX，pageY ：事件发生时相对于页面（如viewport 区域）的坐标。
- screenX，screenY ：事件发生时相对于屏幕的坐标。

和事件相关的元素：

- currentTarget ：事件冒泡阶段所在的当前DOM 元素。
- target,originalTrget ：原始的DOM 元素。
- relatedTarget ：其他和事件相关的DOM 元素（如果有的话）。

不同的浏览器对这些属性的兼容性也不同，尤其是那些不兼容W3C 的浏览器。

### 事件库

手动处理众多浏览器的差异性吃力不讨好。jQuery 的API 提供了bind()函数用来跨浏览器绑定事件监听。在一个jQuery 实例上调用此函数，传入事件名称和回调函数：

```
jQuery("#element").bind(eventName, handler);
```

给一个元素注册点击事件：

```
jQuery("#element").bind("click", function(event) {
    // ...
});
```

jQuery 提供了一些常用事件的快捷方法，比如click、submit 和mouseover：

```
$("#myDiv").click(function(){
// ...
});
```

注意: 使用这个方法之前要确保DOM 元素是存在的。例如，应当在页面载入完成后绑定事件，因此需要绑定window 的load 事件，然后添加监听：

```
jQuery(window).bind("load", function() {
    $("#signinForm").submit(checkForm);
});
```

比监听window 的load 事件更好的方法，即DOMContentLoaded。当DOM 构建完成时触发这个事件，这时图片和样式表可能还未加载完毕。这也就是说这个事件一定会在用户和页面产生交互之前触发。并不是所有的浏览器都支持DOMContentLoaded，因此jQuery 将它融入了ready() 函数，这个函数是兼容各个浏览器的：

```
jQuery.ready(function($)){
    $("#myForm"). bind("submit", function(){ /*...*/});
});
```

实际上，可以不用ready() 函数而直接将回调函数写入jQuery 对象。

```
jQuery(function($){
    // 当页面内容可用时调用
});
```

### 切换上下文

关于事件有一点经常让人感到迷惑就是调用事件回调函数时上下文的切换。当使用浏览器内置的addEventListener() 时，上下文从局部变量切换为目标HTML 元素：

```
new function(){
    this.appName = "wem";

    document.body.addEventListener("click", function(e){
        // 上下文发生改变，因此appName 是undefined
        alert(this.appName);
    }, false);
};
```

要想保持原有的上下文，需要将回调函数包装进一个匿名函数，然后定义一个引用指向它。即使用代理函数来保持当前的上下文。这在jQuery 中也是一种很常用的模式，包括一个proxy() 函数，只需将指定的上下文传入函数即可：

```
$("signinForm").submit($.proxy(function(){ /* ... */ }, this));
```

### 自定义事件

除了浏览器内置的事件之外，也可以触发和绑定自定义事件。这是架构库的一个好方法——也是jQuery 的大多数插件所使用的模式。大多数浏览器厂商均未实现W3C 标准中的自定义事件，可以使用诸如jQuery 或Prototype 的类库来使用这个特性。jQuery 中可以使用trigger() 函数来触发自定义事件。可以通过命名空间的形式来管理事件名称，命名空间中的单词用点号分隔，比如：

```
// 绑定自定义事件
$(".class").bind("refresh.widget",function(){});

// 触发自定义事件
$(".class").trigger("refresh.widget");
```

通过给trigger() 传入一个额外的参数来给事件处理程序传入数据。数据会以附加参数 的形式带入回调：

```
$(".class").bind("frob.widget", function(event, dataNumber){
    console.log(dataNumber);
}); 

$(".class").trigger("frob.widget", 5);
```

和内置事件一样，自定义事件同样会沿着DOM 树做冒泡。

## 模型之ORM

### MVC和命名空间

要确保应用中的视图、状态和数据彼此清晰分离，才能让架构更加整洁有序且更加健壮。模型应当从视图和控制器中解耦出来。与数据操作和行为相关的逻辑都应当放入模型中，通过命名空间
进行管理。

在JavaScript 中，通过给对象添加属性来管理一个命名空间，这个命名空间可以是函数，也可以是变量，比如：

```
var User = {
    records: [ /* ... */ ]
};
```

User 的数组数据就在命名空间User.records 中。和user 相关的函数也可以放入User 模型的命名空间里。比如用fetchRemote() 函数来从服务器端获取user 的数据：

```
var User = {
    records: [],
    fetchRemote: function(){ /* ... */ }
};
```

将模型的属性保存至命名空间中的做法可以确保不会产生冲突，这也是符合MVC 原则的，同时也能避免代码变成一堆函数和回调混杂在一起的大杂烩。

可以对命名空间做一点改进，将那些在真实user 对象上的和user 实例相关的函数也都添加进去。假设user 记录包含一个destory() 函数，它是和具体的user 相关的，因此这个函数应当基于User 实例进行调用：

```
var user = new User;
user.destroy()
```

为了做到这一点，应当将User 写成一个类，而不是一个简单对象：

```
var User = function(atts){
    his.attributes = atts || {};
};

User.prototype.destroy = function(){
    /* ... */
};
```

对于那些和具体的user 不相关的函数和变量，则可以直接定义在User 对象中：

```
User.fetchRemote = function(){
    /* ... */
};
```

### 构建对象关系映射（ORM）

对象关系映射（Ojbect-relational mapper，简称ORM）是在除JavaScript 以外的编程语言中常见的一种数据结构。在JavaScript 应用中，对象关系映射也是一种非常有用的技术，它可以用来做数据管理及用做模型。比如使用ORM 可以将模型和远程服务捆绑在一起，任何模型实例的改变都会在后台发起一个Ajax 请求到服务器端。或者将模型实例和HTML 元素绑定在一起，任何对实例的更改都会在界面中反映出来。

现在让来创建一个自定义ORM。

本质上讲，ORM 是一个包装了一些数据的对象层。以往ORM 常用于抽象SQL 数据库，但在这里ORM 只是用于抽象JavaScript 数据类型。这个额外的层有一个好处，可以通过给它添加自定义的函数和属性来增强基础数据的功能。比如添加数据的合法性验证、监听、数据持久化及服务器端的回调处理等，这样会增加代码的重用率。


### 原型继承

这里使用Object.create() 来构造ORM，这里使用基于原型（prototype-based）的继承，而没有用到构造函数和new关键字。

Object.create() 只有一个参数即原型对象，它返回一个新对象，这个新对象的原型就是传入的参数。换句话说，传入一个对象，返回一个继承了这个对象的新对象。

对于不支持的Object.create()的浏览器 ，可以很容易地模拟出这个函数：

```
if (typeof Object.create !== "function")
    Object.create = function(o) {
        function F() {}
        F.prototype = o;
        return new F();
    };
```

现在来创建Model 对象，Model 对象将用于创建新模型和实例：

```
var Model = {
    inherited: function(){},
    created: function(){},
    prototype: {
        init: function(){}
    },

    create: function(){
        var object = Object.create(this);
        object.parent = this;
        object.prototype = object.fn = Object.create(this.prototype);
        object.created();
        this.inherited(object);
        return object;
    },

    init: function(){
        var instance = Object.create(this.prototype);
        instance.parent = this;
        instance.init.apply(instance, arguments);
        return instance;
    }
};
```

create() 函数返回一个新对象，这个对象继承自Model 对象，使用它来创建新模型。
init() 函数返回一个新对象，它继承自Model.prototype——如Model 对象的一个实例：

```
var Asset = Model.create();
var User = Model.create();
var user = User.init();
```

### 添加ORM属性

现在如果给Model 对象添加属性，对于继承的模型来说，这些新增属性都是可访问的：

```
// 添加对象属性
jQuery.extend(Model, {
    find: function(){}
});

// 添加实例属性
jQuery.extend(Model.prototype, {
    init: function(atts) {
        if (atts) this.load(atts);
    },
    load: function(attributes){
        for(var name in attributes)
        this[name] = attributes[name];
    }
});
```

jQuery.extend() 只是代替for 循环手动复制属性的一种快捷方式，这和在load()函数中的做法差不多。现在，对象和实例属性都传播到了单独的模型里：

```
assertEqual( typeof Asset.find, "function" );
```

实际上我们会增加很多属性，因此还需将extend() 和include() 添加至Model 对象中：

```
var Model = {
    /* ……代码片段……*/
    extend: function(o){
        var extended = o.extended;
        jQuery.extend(this, o);
        if (extended) extended(this);
    },

    include: function(o){
        var included = o.included;
        jQuery.extend(this.prototype, o);
        if (included) included(this);
    }
};

// 添加对象属性
Model.extend({
    find: function(){}
});

// 添加实例属性
Model.include({
    init: function(atts) { /* ... */ },
    load: function(attributes){ /* ... */ }
});
```

现在可以创建新的资源并设置一些属性：

```
var asset = Asset.init({name: "foo.png"});
```

### 持久化记录

需要一种保持记录持久化的方法，即将引用保存至新创建的实例中以便任何时候都能访问它。通过在Model 中使用records 对象来实现。当保存一个实例的时候，就将它添加进这个对象中；当删除实例时，和将它从对象中删除：

```
// 用来保存资源的对象
Model.records = {};
    Model.include({
        newRecord: true,
        create: function(){
        this.newRecord = false;
        this.parent.records[this.id] = this;
    },
    destroy: function(){
        delete this.parent.records[this.id];
    }
});
```

更新一个已存在的实例——只需更新对象引用即可：

```
Model.include({
    update: function(){
        this.parent.records[this.id] = this;
    }
});
```

现在创建一个快捷函数来保存实例，这样就不用每次都检查这个实例是否已经保存过或是否需要新创建实例了：

```
// 将对象存入hash 记录中，保持一个引用指向它
Model.include({
    save: function(){
        this.newRecord ? this.create() : this.update();
    }
});
```

用来根据ID 查找资源的find() 函数:

```
Model.extend({
    // 通过ID 查找，找不到则抛出异常
    find: function(id){
        return this.records[id] || throw("Unknown record");
    }
});
```

现在已经成功地创建了一个基本的ORM，运行一下：

```
var asset = Asset.init();
asset.name = "same, same";
asset.id = 1
asset.save();
var asset2 = Asset.init();
asset2.name = "but different";
asset2.id = 2;
asset2.save();
assertEqual( Asset.find(1).name, "same, same" );
asset2.destroy();
```

### 增加 ID 支持

此时每次保存一条记录都必须手动指定一个ID。可以加入自动化处理。首先，我们需要一个方法来自动生成ID，可以使用全局统一标识（Globally UniqueIdentifier，简称GUID）生成器来做这一步。从技术的角度讲，出于API 的原因，JavaScript 无法友好正式地生成128 位的GUID，它只能生成伪随机数。虽然JavaScript 中内置的Math.random() 方法尽管产生的是伪随机数，但也足够用了。

Robert Kieffer 写了一个简单明了的GUID 生成器，它使用Math.random() 来产生一个伪随机数的GUID（http://goo.gl/0b0hu），代码也很简单：

```
Math.guid = function(){
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = Math.random()*16|0, v = c == 'x' ? r : (r&0x3|0x8);
        return v.toString(16);
    }).toUpperCase();
};
```

现在有了生成GUID 的方法，可以很容易将它集成到ORM 里，剩下的只需修改create() 函数了：

```
Model.extend({
    create: function(){
        if ( !this.id ) 
            this.id = Math.guid();
        this.newRecord = false;
        this.parent.records[this.id] = this;
    }
});
```

这样任何新创建的记录都包含一个随机的GUID 作为它们的ID ：

```
var asset = Asset.init();
asset.save();
asset.id //=> "54E52592-313E-4F8B-869B-58D61F00DC74"
```

### 寻址引用

目前创建的ORM 中存在一个与引用相关的bug。当保存或通过find() 查找记录时，所返回的实例并没有复制一份，因此对任何属性的修改都会影响原始资源。改进：

```
var asset = new Asset({name: "foo"});
asset.save();

// 正确传入资源
assertEqual( Asset.find(asset.id).name, "foo" );

// 更改属性，而没有调用update()
asset.name = "wem";

// 出问题了，因为asset 的名字被修改为“wem”
assertEqual( Asset.find(asset.id).name, "foo" );
```

需要修复这个问题，在执行find() 操作的时候新创建一个对象，同样在创建或更新记录的时候需要复制对象：

```
Asset.extend({
    find: function(id){
        var record = this.records[id];
        if ( !record ) throw("Unknown record");
        return record.dup();
    }
    });
    Asset.include({
        create: function(){
        this.newRecord = false;
        this.parent.records[this.id] = this.dup();
    },
    update: function(){
        this.parent.records[this.id] = this.dup();
    },
    dup: function(){
        return jQuery.extend(true, {}, this);
    }
});
```

这里存在另外一个问题，Model.records 是被所有模型所共享的对象：

```
assertEqual( Asset.records, Person.records );
```

但这在合并所有的记录时会有副作用：

```
var asset = Asset.init();
asset.save();
assert( asset in Person.records );
```

解决办法是在创建新的模型时设置一个新的records 对象。Model.create() 是创建新对象的回调，因此我们可以设置任意描述这个模型的对象：

```
Model.extend({
    created: function(){
        this.records = {};
    }
});
```