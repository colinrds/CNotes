[toc]

## 一：理解工厂模式
  
工厂模式类似于现实生活中的工厂可以产生大量相似的商品，去做同样的事情，实现同样的效果;这时候需要使用工厂模式。

简单的工厂模式可以理解为解决多个相似的问题;这也是她的优点;比如如下代码： 

```
function CreatePerson(name,age,sex) {
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sex = sex;
    obj.sayName = function(){
        return this.name;
    }
    return obj;
}
var p1 = new CreatePerson("longen",'28','男');
var p2 = new CreatePerson("tugenhua",'27','女');
console.log(p1.name); // longen
console.log(p1.age);  // 28
console.log(p1.sex);  // 男
console.log(p1.sayName()); // longen

console.log(p2.name);  // tugenhua
console.log(p2.age);   // 27
console.log(p2.sex);   // 女
console.log(p2.sayName()); // tugenhua

// 返回都是object 无法识别对象的类型 不知道他们是哪个对象的实列
console.log(typeof p1);  // object
console.log(typeof p2);  // object
console.log(p1 instanceof Object); // true
```

如上代码：函数CreatePerson能接受三个参数name,age,sex等参数，可以无数次调用这个函数，每次返回都会包含三个属性和一个方法的对象。

工厂模式是为了解决多个类似对象声明的问题;也就是为了解决实列化对象产生重复的问题。

- 优点：能解决多个相似的问题。
- 缺点：不能知道对象识别的问题(对象的类型不知道)。
- 复杂的工厂模式定义是：将其成员对象的实列化推迟到子类中，子类可以重写父类接口方法以便创建的时候指定自己的对象类型。

父类只对创建过程中的一般性问题进行处理，这些处理会被子类继承，子类之间是相互独立的，具体的业务逻辑会放在子类中进行编写。

父类就变成了一个抽象类，但是父类可以执行子类中相同类似的方法，具体的业务逻辑需要放在子类中去实现；比如我现在开几个自行车店，那么每个店都有几种型号的自行车出售。我们现在来使用工厂模式来编写这些代码;

父类的构造函数如下：

```
// 定义自行车的构造函数
var BicycleShop = function(){};
BicycleShop.prototype = {
    constructor: BicycleShop,
    /*
    * 买自行车这个方法
    * @param {model} 自行车型号
    */
    sellBicycle: function(model){
        var bicycle = this.createBicycle(mode);
        // 执行A业务逻辑
        bicycle.A();

        // 执行B业务逻辑
        bicycle.B();

        return bicycle;
    },
    createBicycle: function(model){
        throw new Error("父类是抽象类不能直接调用，需要子类重写该方法");
    }
};
```

上面是定义一个自行车抽象类来编写工厂模式的实列，定义了createBicycle这个方法，但是如果直接实例化父类，调用父类中的这个createBicycle方法,会抛出一个error，因为父类是一个抽象类，他不能被实列化，只能通过子类来实现这个方法，实现自己的业务逻辑，下面我们来定义子类，我们学会如何使用工厂模式重新编写这个方法，首先我们需要继承父类中的成员，然后编写子类;如下代码：

```
// 定义自行车的构造函数
var BicycleShop = function(name){
    this.name = name;
    this.method = function(){
        return this.name;
    }
};
BicycleShop.prototype = {
    constructor: BicycleShop,
    /*
     * 买自行车这个方法
     * @param {model} 自行车型号
    */
    sellBicycle: function(model){
            var bicycle = this.createBicycle(model);
            // 执行A业务逻辑
            bicycle.A();

            // 执行B业务逻辑
            bicycle.B();

            return bicycle;
        },
        createBicycle: function(model){
            throw new Error("父类是抽象类不能直接调用，需要子类重写该方法");
        }
    };
    // 实现原型继承
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
    var BicycleChild = function(name){
        this.name = name;
// 继承构造函数父类中的属性和方法
        BicycleShop.call(this,name);
    };
    // 子类继承父类原型方法
    extend(BicycleChild,BicycleShop);
// BicycleChild 子类重写父类的方法
BicycleChild.prototype.createBicycle = function(){
    var A = function(){
        console.log("执行A业务操作");    
    };
    var B = function(){
        console.log("执行B业务操作");
    };
    return {
        A: A,
        B: B
    }
}
var childClass = new BicycleChild("龙恩");
console.log(childClass);
```

实例化子类，然后打印出该实例, 如下截图所示：

![image](http://ow2n75eab.bkt.clouddn.com/561794-20160218145257206-1.png)

```
console.log(childClass.name);  // 龙恩

// 下面是实例化后 执行父类中的sellBicycle这个方法后会依次调用父类中的A

// 和B方法；A方法和B方法依次在子类中去编写具体的业务逻辑。

childClass.sellBicycle("mode"); // 打印出  执行A业务操作和执行B业务操作
```

上面只是"龙恩"自行车这么一个型号的，如果需要生成其他型号的自行车的话，可以编写其他子类，工厂模式最重要的优点是：可以实现一些相同的方法，这些相同的方法我们可以放在父类中编写代码，那么需要实现具体的业务逻辑，那么可以放在子类中重写该父类的方法，去实现自己的业务逻辑；使用专业术语来讲的话有2点：第一：弱化对象间的耦合，防止代码的重复。在一个方法中进行类的实例化，可以消除重复性的代码。第二：重复性的代码可以放在父类去编写，子类继承于父类的所有成员属性和方法，子类只专注于实现自己的业务逻辑。


## 二：理解单体模式

单体模式提供了一种将代码组织为一个逻辑单元的手段，这个逻辑单元中的代码可以通过单一变量进行访问。

单体模式的优点是：

1.可以用来划分命名空间，减少全局变量的数量。
2.使用单体模式可以使代码组织的更为一致，使代码容易阅读和维护。
3.可以被实例化，且实例化一次。

什么是单体模式？单体模式是一个用来划分命名空间并将一批属性和方法组织在一起的对象，如果它可以被实例化，那么它只能被实例化一次。

但是并非所有的对象字面量都是单体，比如说模拟数组或容纳数据的话，那么它就不是单体，但是如果是组织一批相关的属性和方法在一起的话，那么它有可能是单体模式，所以这需要看开发者编写代码的意图；

下面我们来看看定义一个对象字面量(结构类似于单体模式)的基本结构如下：

```
// 对象字面量
var Singleton = {
    attr1: 1,
    attr2: 2,
    method1: function(){
        return this.attr1;
    },
    method2: function(){
        return this.attr2;
    }
};
```

如上面只是简单的字面量结构，上面的所有成员变量都是通过Singleton来访问的，但是它并不是单体模式；因为单体模式还有一个更重要的特点，就是可以仅被实例化一次，上面的只是不能被实例化的一个类，因此不是单体模式；对象字面量是用来创建单体模式的方法之一；

- 使用单体模式的结构如下demo

我们明白的是单体模式如果有实例化的话，那么只实例化一次，要实现一个单体模式的话，我们无非就是使用一个变量来标识该类是否被实例化，如果未被实例化的话，那么我们可以实例化一次，否则的话，直接返回已经被实例化的对象。

如下代码是单体模式的基本结构：

```
// 单体模式
var Singleton = function(name){
    this.name = name;
    this.instance = null;
};
Singleton.prototype.getName = function(){
    return this.name;
}
// 获取实例对象
function getInstance(name) {
    if(!this.instance) {
        this.instance = new Singleton(name);
    }
    return this.instance;
}
// 测试单体模式的实例
var a = getInstance("aa");
var b = getInstance("bb");

// 因为单体模式是只实例化一次，所以下面的实例是相等的

console.log(a === b); // true

// 由于单体模式只实例化一次，因此第一次调用，返回的是a实例对象，当我们继续调用的时候，b的实例就是a的实例，因此下面都是打印的是aa；

console.log(a.getName());// aa

console.log(b.getName());// aa
```

上面的封装单体模式也可以改成如下结构写法：

```
// 单体模式
var Singleton = function(name){
    this.name = name;
};
Singleton.prototype.getName = function(){
    return this.name;
}
// 获取实例对象
var getInstance = (function() {
    var instance = null;
    return function(name) {
        if(!instance) {
            instance = new Singleton(name);
        }
        return instance;
    }
})();

// 测试单体模式的实例

var a = getInstance("aa");
var b = getInstance("bb");

// 因为单体模式是只实例化一次，所以下面的实例是相等的

console.log(a === b); // true

console.log(a.getName());// aa

console.log(b.getName());// aa
```

- 理解使用代理实现单列模式的好处

比如我现在页面上需要创建一个div的元素，那么我们肯定需要有一个创建div的函数，而现在我只需要这个函数只负责创建div元素，其他的它不想管，也就是想实现单一职责原则，就好比淘宝的kissy一样，一开始的时候他们定义kissy只做一件事，并且把这件事做好，具体的单体模式中的实例化类的事情交给代理函数去处理，这样做的好处是具体的业务逻辑分开了，代理只管代理的业务逻辑，在这里代理的作用是实例化对象，并且只实例化一次; 创建div代码只管创建div，其他的不管；如下代码：

```
// 单体模式
var CreateDiv = function(html) {
    this.html = html;
    this.init();
}
CreateDiv.prototype.init = function(){
    var div = document.createElement("div");
    div.innerHTML = this.html;
    document.body.appendChild(div);
};
// 代理实现单体模式
var ProxyMode = (function(){
    var instance;
    return function(html) {
        if(!instance) {
            instance = new CreateDiv("我来测试下");
        }
        return instance;
    } 
})();
var a = new ProxyMode("aaa");
var b = new ProxyMode("bbb");
console.log(a===b);// true
```

- 理解使用单体模式来实现弹窗的基本原理

下面我们继续来使用单体模式来实现一个弹窗的demo；我们先不讨论使用单体模式来实现，我们想下我们平时是怎么编写代码来实现弹窗效果的; 比如我们有一个弹窗，默认的情况下肯定是隐藏的，当我点击的时候，它需要显示出来；如下编写代码：

```
// 实现弹窗
var createWindow = function(){
    var div = document.createElement("div");
    div.innerHTML = "我是弹窗内容";
    div.style.display = 'none';
    document.body.appendChild('div');
    return div;
};
document.getElementById("Id").onclick = function(){
    // 点击后先创建一个div元素
    var win = createWindow();
    win.style.display = "block";
}
```

如上的代码；大家可以看看，有明显的缺点，比如我点击一个元素需要创建一个div，我点击第二个元素又会创建一次div，我们频繁的点击某某元素，他们会频繁的创建div的元素，虽然当我们点击关闭的时候可以移除弹出代码，但是呢我们频繁的创建和删除并不好，特别对于性能会有很大的影响，对DOM频繁的操作会引起重绘等，从而影响性能；因此这是非常不好的习惯；我们现在可以使用单体模式来实现弹窗效果，我们只实例化一次就可以了；如下代码：

```
// 实现单体模式弹窗
var createWindow = (function(){
    var div;
    return function(){
        if(!div) {
            div = document.createElement("div");
            div.innerHTML = "我是弹窗内容";
            div.style.display = 'none';
            document.body.appendChild(div);
        }
        return div;
    }
})();
document.getElementById("Id").onclick = function(){
    // 点击后先创建一个div元素
    var win = createWindow();
    win.style.display = "block";
}
```

- 理解编写通用的单体模式

上面的弹窗的代码虽然完成了使用单体模式创建弹窗效果，但是代码并不通用，比如上面是完成弹窗的代码，假如我们以后需要在页面中一个iframe呢？我们是不是需要重新写一套创建iframe的代码呢？比如如下创建iframe：

```
var createIframe = (function(){
    var iframe;
    return function(){
        if(!iframe) {
            iframe = document.createElement("iframe");
            iframe.style.display = 'none';
            document.body.appendChild(iframe);
        }
        return iframe;
    };
})();
```

我们看到如上代码，创建div的代码和创建iframe代码很类似，我们现在可以考虑把通用的代码分离出来，使代码变成完全抽象，我们现在可以编写一套代码封装在getInstance函数内，如下代码：

```
var getInstance = function(fn) {
    var result;
    return function(){
        return result || (result = fn.call(this,arguments));
    }
};
```

如上代码：我们使用一个参数fn传递进去，如果有result这个实例的话，直接返回，否则的话，当前的getInstance函数调用fn这个函数，是this指针指向与这个fn这个函数；之后返回被保存在result里面；现在我们可以传递一个函数进去，不管他是创建div也好，还是创建iframe也好，总之如果是这种的话，都可以使用getInstance来获取他们的实例对象；

如下测试创建iframe和创建div的代码如下：

```
// 创建div
var createWindow = function(){
    var div = document.createElement("div");
    div.innerHTML = "我是弹窗内容";
    div.style.display = 'none';
    document.body.appendChild(div);
    return div;
};
// 创建iframe
var createIframe = function(){
    var iframe = document.createElement("iframe");
    document.body.appendChild(iframe);
    return iframe;
};
// 获取实例的封装代码
var getInstance = function(fn) {
    var result;
    return function(){
        return result || (result = fn.call(this,arguments));
    }
};
// 测试创建div
var createSingleDiv = getInstance(createWindow);
document.getElementById("Id").onclick = function(){
    var win = createSingleDiv();
    win.style.display = "block";
};
// 测试创建iframe
var createSingleIframe = getInstance(createIframe);
document.getElementById("Id").onclick = function(){
    var win = createSingleIframe();
    win.src = "http://cnblogs.com";
};
```

## 三：理解模块模式

我们通过单体模式理解了是以对象字面量的方式来创建单体模式的；比如如下的对象字面量的方式代码如下：

```
var singleMode = {
    name: value,
    method: function(){
                
    }
};
```

模块模式的思路是为单体模式添加私有变量和私有方法能够减少全局变量的使用；如下就是一个模块模式的代码结构：

```
var singleMode = (function(){
    // 创建私有变量
    var privateNum = 112;
    // 创建私有函数
    function privateFunc(){
        // 实现自己的业务逻辑代码
    }
    // 返回一个对象包含公有方法和属性
    return {
        publicMethod1: publicMethod1,
        publicMethod2: publicMethod1
    };
})();
```

模块模式使用了一个返回对象的匿名函数。在这个匿名函数内部，先定义了私有变量和函数，供内部函数使用，然后将一个对象字面量作为函数的值返回，返回的对象字面量中只包含可以公开的属性和方法。这样的话，可以提供外部使用该方法；由于该返回对象中的公有方法是在匿名函数内部定义的，因此它可以访问内部的私有变量和函数。

- 我们什么时候使用模块模式？

如果我们必须创建一个对象并以某些数据进行初始化，同时还要公开一些能够访问这些私有数据的方法，那么我们这个时候就可以使用模块模式了。

- 理解增强的模块模式

增强的模块模式的使用场合是：适合那些单列必须是某种类型的实例，同时还必须添加某些属性或方法对其加以增强的情况。比如如下代码：

```
function CustomType() {
    this.name = "tugenhua";
};
CustomType.prototype.getName = function(){
    return this.name;
}
var application = (function(){
    // 定义私有
    var privateA = "aa";
    // 定义私有函数
    function A(){};

    // 实例化一个对象后，返回该实例，然后为该实例增加一些公有属性和方法
    var object = new CustomType();

    // 添加公有属性
    object.A = "aa";
    // 添加公有方法
    object.B = function(){
        return privateA;
    }
    // 返回该对象
    return object;
})();
```

下面我们来打印下application该对象；如下：

```
console.log(application);
```

![image](http://ow2n75eab.bkt.clouddn.com/561794-20160218145257206-2.png)

继续打印该公有属性和方法如下：

```
console.log(application.A);// aa

console.log(application.B()); // aa

console.log(application.name); // tugenhua

console.log(application.getName());// tugenhua
```

## 四：理解代理模式

代理是一个对象，它可以用来控制对本体对象的访问，它与本体对象实现了同样的接口，代理对象会把所有的调用方法传递给本体对象的；代理模式最基本的形式是对访问进行控制，而本体对象则负责执行所分派的那个对象的函数或者类，简单的来讲本地对象注重的去执行页面上的代码，代理则控制本地对象何时被实例化，何时被使用；我们在上面的单体模式中使用过一些代理模式，就是使用代理模式实现单体模式的实例化，其他的事情就交给本体对象去处理；

代理的优点：

1. 代理对象可以代替本体被实例化，并使其可以被远程访问；
2. 它还可以把本体实例化推迟到真正需要的时候；对于实例化比较费时的本体对象，或者因为尺寸比较大以至于不用时不适于保存在内存中的本体，我们可以推迟实例化该对象；

我们先来理解代理对象代替本体对象被实例化的列子；比如现在京东ceo想送给奶茶妹一个礼物，但是呢假如该ceo不好意思送，或者由于工作忙没有时间送，那么这个时候他就想委托他的经纪人去做这件事，于是我们可以使用代理模式来编写如下代码：

```
// 先申明一个奶茶妹对象
var TeaAndMilkGirl = function(name) {
    this.name = name;
};
// 这是京东ceo先生
var Ceo = function(girl) {
    this.girl = girl;
    // 送结婚礼物 给奶茶妹
    this.sendMarriageRing = function(ring) {
        console.log("Hi " + this.girl.name + ", ceo送你一个礼物：" + ring);
    }
};
// 京东ceo的经纪人是代理，来代替送
var ProxyObj = function(girl){
    this.girl = girl;
    // 经纪人代理送礼物给奶茶妹
    this.sendGift = function(gift) {
        // 代理模式负责本体对象实例化
        (new Ceo(this.girl)).sendMarriageRing(gift);
    }
};
// 初始化
var proxy = new ProxyObj(new TeaAndMilkGirl("奶茶妹"));
proxy.sendGift("结婚戒"); // Hi 奶茶妹, ceo送你一个礼物：结婚戒
```

代码如上的基本结构，TeaAndMilkGirl 是一个被送的对象(这里是奶茶妹)；Ceo 是送礼物的对象，他保存了奶茶妹这个属性，及有一个自己的特权方法sendMarriageRing 就是送礼物给奶茶妹这么一个方法；然后呢他是想通过他的经纪人去把这件事完成，于是需要创建一个经济人的代理模式，名字叫ProxyObj ；他的主要做的事情是，把ceo交给他的礼物送给ceo的情人，因此该对象同样需要保存ceo情人的对象作为自己的属性，同时也需要一个特权方法sendGift ，该方法是送礼物，因此在该方法内可以实例化本体对象，这里的本体对象是ceo送花这件事情，因此需要实例化该本体对象后及调用本体对象的方法(sendMarriageRing).

最后我们初始化是需要代理对象ProxyObj；调用ProxyObj 对象的送花这个方法(sendGift)即可；

对于我们提到的优点，第二点的话，我们下面可以来理解下虚拟代理，虚拟代理用于控制对那种创建开销很大的本体访问，它会把本体的实例化推迟到有方法被调用的时候；比如说现在有一个对象的实例化很慢的话，不能在网页加载的时候立即完成，我们可以为其创建一个虚拟代理，让他把该对象的实例推迟到需要的时候。

- 理解使用虚拟代理实现图片的预加载

在网页开发中，图片的预加载是一种比较常用的技术，如果直接给img标签节点设置src属性的话，如果图片比较大的话，或者网速相对比较慢的话，那么在图片未加载完之前，图片会有一段时间是空白的场景，这样对于用户体验来讲并不好，那么这个时候我们可以在图片未加载完之前我们可以使用一个loading加载图片来作为一个占位符，来提示用户该图片正在加载，等图片加载完后我们可以对该图片直接进行赋值即可；下面我们先不用代理模式来实现图片的预加载的情况下代码如下：

**第一种方案：不使用代理的预加载图片函数如下**

```
// 不使用代理的预加载图片函数如下
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    var img = new Image();
    img.onload = function(){
        imgNode.src = this.src;
    };
    return {
        setSrc: function(src) {
            imgNode.src = "http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif";
            img.src = src;
        }
    }
})();
// 调用方式
myImage.setSrc("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```

如上代码是不使用代理模式来实现的代码；

**第二种方案：使用代理模式来编写预加载图片的代码如下：**

```
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    return {
        setSrc: function(src) {
            imgNode.src = src;
        }
    }
})();
// 代理模式
var ProxyImage = (function(){
    var img = new Image();
    img.onload = function(){
        myImage.setSrc(this.src);
    };
    return {
        setSrc: function(src) {
                         myImage.setSrc("http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif");
        img.src = src;
        }
    }
})();
// 调用方式
ProxyImage.setSrc("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```

第一种方案是使用一般的编码方式实现图片的预加载技术，首先创建imgNode元素，然后调用myImage.setSrc该方法的时候，先给图片一个预加载图片，当图片加载完的时候，再给img元素赋值，第二种方案是使用代理模式来实现的，myImage 函数只负责创建img元素，代理函数ProxyImage 负责给图片设置loading图片，当图片真正加载完后的话，调用myImage中的myImage.setSrc方法设置图片的路径；他们之间的优缺点如下：

1. 第一种方案一般的方法代码的耦合性太高，一个函数内负责做了几件事情，比如创建img元素，和实现给未加载图片完成之前设置loading加载状态等多项事情，未满足面向对象设计原则中单一职责原则；并且当某个时候不需要代理的时候，需要从myImage 函数内把代码删掉，这样代码耦合性太高。
2. 第二种方案使用代理模式，其中myImage 函数只负责做一件事，创建img元素加入到页面中，其中的加载loading图片交给代理函数ProxyImage 去做，当图片加载成功后，代理函数ProxyImage 会通知及执行myImage 函数的方法，同时当以后不需要代理对象的话，我们直接可以调用本体对象的方法即可；

从上面代理模式我们可以看到，代理模式和本体对象中有相同的方法setSrc,这样设置的话有如下2个优点：

1. 用户可以放心地请求代理，他们只关心是否能得到想要的结果。假如我门不需要代理对象的话，直接可以换成本体对象调用该方法即可。
2. 在任何使用本体对象的地方都可以替换成使用代理。

当然如果代理对象和本体对象都返回一个匿名函数的话，那么也可以认为他们也具有一直的接口；比如如下代码：

```
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    return function(src){
        imgNode.src = src; 
    }
})();
// 代理模式
var ProxyImage = (function(){
    var img = new Image();
    img.onload = function(){
        myImage(this.src);
    };
    return function(src) {
                myImage("http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif");
        img.src = src;
    }
})();
// 调用方式
ProxyImage("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```

- 虚拟代理合并http请求的理解：

比如在做后端系统中，有表格数据，每一条数据前面有复选框按钮，当点击复选框按钮时候，需要获取该id后需要传递给给服务器发送ajax请求，服务器端需要记录这条数据，去请求，如果我们每当点击一下向服务器发送一个http请求的话，对于服务器来说压力比较大，网络请求比较频繁，但是如果现在该系统的实时数据不是很高的话，我们可以通过一个代理函数收集一段时间内(比如说2-3秒)的所有id，一次性发ajax请求给服务器，相对来说网络请求降低了, 服务器压力减少了;

```
// 首先html结构如下：
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id="1"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id = "2"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id="3"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id = "4"/>
</p>
```

一般的情况下 JS如下编写

```
<script>
    var checkboxs = document.getElementsByClassName("j-input");
    for(var i = 0,ilen = checkboxs.length; i < ilen; i+=1) {
        (function(i){
            checkboxs[i].onclick = function(){
                if(this.checked) {
                    var id = this.getAttribute("data-id");
                    // 如下是ajax请求
                }
            }
        })(i);
    }
</script>
```

下面我们通过虚拟代理的方式，延迟2秒，在2秒后获取所有被选中的复选框的按钮id，一次性给服务器发请求。

通过点击页面的复选框，选中的时候增加一个属性isflag，没有选中的时候删除该属性isflag，然后延迟个2秒，在2秒后重新判断页面上所有复选框中有isflag的属性上的id，存入数组，然后代理函数调用本体函数的方法，把延迟2秒后的所有id一次性发给本体方法，本体方法可以获取所有的id，可以向服务器端发送ajax请求，这样的话，服务器的请求压力相对来说减少了。

代码如下：

```
// 本体函数
var mainFunc = function(ids) {
    console.log(ids); // 即可打印被选中的所有的id
    // 再把所有的id一次性发ajax请求给服务器端
};
// 代理函数 通过代理函数获取所有的id 传给本体函数去执行
var proxyFunc = (function(){
    var cache = [],  // 保存一段时间内的id
        timer = null; // 定时器
    return function(checkboxs) {
        // 判断如果定时器有的话，不进行覆盖操作
        if(timer) {
            return;
        }
        timer = setTimeout(function(){
            // 在2秒内获取所有被选中的id，通过属性isflag判断是否被选中
            for(var i = 0,ilen = checkboxs.length; i < ilen; i++) {
                if(checkboxs[i].hasAttribute("isflag")) {
                    var id = checkboxs[i].getAttribute("data-id");
                    cache[cache.length] = id;
                }
            }
            mainFunc(cache.join(',')); // 2秒后需要给本体函数传递所有的id
            // 清空定时器
            clearTimeout(timer);
            timer = null;
            cache = [];
        },2000);
    }
})();
var checkboxs = document.getElementsByClassName("j-input");
for(var i = 0,ilen = checkboxs.length; i < ilen; i+=1) {
    (function(i){
        checkboxs[i].onclick = function(){
            if(this.checked) {
                // 给当前增加一个属性
                this.setAttribute("isflag",1);
            }else {
                this.removeAttribute('isflag');
            }
            // 调用代理函数
            proxyFunc(checkboxs);
        }
    })(i);
}
```

- 理解缓存代理：

缓存代理的含义就是对第一次运行时候进行缓存，当再一次运行相同的时候，直接从缓存里面取，这样做的好处是避免重复一次运算功能，如果运算非常复杂的话，对性能很耗费，那么使用缓存对象可以提高性能;我们可以先来理解一个简单的缓存列子，就是网上常见的加法和乘法的运算。代码如下：

```
// 计算乘法
var mult = function(){
    var a = 1;
    for(var i = 0,ilen = arguments.length; i < ilen; i+=1) {
        a = a*arguments[i];
    }
    return a;
};
// 计算加法
var plus = function(){
    var a = 0;
    for(var i = 0,ilen = arguments.length; i < ilen; i+=1) {
        a += arguments[i];
    }
    return a;
}
// 代理函数
var proxyFunc = function(fn) {
    var cache = {};  // 缓存对象
    return function(){
        var args = Array.prototype.join.call(arguments,',');
        if(args in cache) {
            return cache[args];   // 使用缓存代理
        }
        return cache[args] = fn.apply(this,arguments);
    }
};
var proxyMult = proxyFunc(mult);
console.log(proxyMult(1,2,3,4)); // 24
console.log(proxyMult(1,2,3,4)); // 缓存取 24

var proxyPlus = proxyFunc(plus);
console.log(proxyPlus(1,2,3,4));  // 10
console.log(proxyPlus(1,2,3,4));  // 缓存取 10
```

## 五：理解职责链模式

优点是：消除请求的发送者与接收者之间的耦合。

职责连是由多个不同的对象组成的，发送者是发送请求的对象，而接收者则是链中那些接收这种请求并且对其进行处理或传递的对象。请求本身有时候也可以是一个对象，它封装了和操作有关的所有数据，基本实现流程如下：

1. 发送者知道链中的第一个接收者，它向这个接收者发送该请求。
2. 每一个接收者都对请求进行分析，然后要么处理它，要么它往下传递。
3. 每一个接收者知道其他的对象只有一个，即它在链中的下家(successor)。
4. 如果没有任何接收者处理请求，那么请求会从链中离开。

我们可以理解职责链模式是处理请求组成的一条链，请求在这些对象之间依次传递，直到遇到一个可以处理它的对象，我们把这些对象称为链中的节点。比如对象A给对象B发请求，如果B对象不处理，它就会把请求交给C，如果C对象不处理的话，它就会把请求交给D，依次类推，直到有一个对象能处理该请求为止，当然没有任何对象处理该请求的话，那么请求就会从链中离开。

比如常见的一些外包公司接到一个项目，那么接到项目有可能是公司的负责项目的人或者经理级别的人，经理接到项目后自己不开发，直接把它交到项目经理来开发，项目经理自己肯定不乐意自己动手开发哦，它就把项目交给下面的码农来做，所以码农来处理它，如果码农也不处理的话，那么这个项目可能会直接挂掉了，但是最后完成后，外包公司它并不知道这些项目中的那一部分具体有哪些人开发的，它并不知道，也并不关心的，它关心的是这个项目已交给外包公司已经开发完成了且没有任何bug就可以了；所以职责链模式的优点就在这里：

消除请求的发送者(需要外包项目的公司)与接收者(外包公司)之间的耦合。

下面列举个列子来说明职责链的好处：

天猫每年双11都会做抽奖活动的，比如阿里巴巴想提高大家使用支付宝来支付的话，每一位用户充值500元到支付宝的话，那么可以100%中奖100元红包，

充值200元到支付宝的话，那么可以100%中奖20元的红包，当然如果不充值的话，也可以抽奖，但是概率非常低，基本上是抽不到的，当然也有可能抽到的。

我们下面可以分析下代码中的几个字段值需要来判断：

1. orderType(充值类型)，如果值为1的话，说明是充值500元的用户，如果为2的话，说明是充值200元的用户，如果是3的话，说明是没有充值的用户。
2. isPay(是否已经成功充值了): 如果该值为true的话，说明已经成功充值了，否则的话 说明没有充值成功；就当作普通用户来购买。
3. count(表示数量)；普通用户抽奖，如果数量有的话，就可以拿到优惠卷，否则的话，不能拿到优惠卷。

```
// 我们一般写代码如下处理操作
var order =  function(orderType,isPay,count) {
    if(orderType == 1) {  // 用户充值500元到支付宝去
        if(isPay == true) { // 如果充值成功的话，100%中奖
            console.log("亲爱的用户，您中奖了100元红包了");
        }else {
            // 充值失败，就当作普通用户来处理中奖信息
            if(count > 0) {
                console.log("亲爱的用户，您已抽到10元优惠卷");
            }else {
                console.log("亲爱的用户，请再接再厉哦");
            }
        }
    }else if(orderType == 2) {  // 用户充值200元到支付宝去
        if(isPay == true) {     // 如果充值成功的话，100%中奖
            console.log("亲爱的用户，您中奖了20元红包了");
        }else {
            // 充值失败，就当作普通用户来处理中奖信息
            if(count > 0) {
                console.log("亲爱的用户，您已抽到10元优惠卷");
            }else {
                console.log("亲爱的用户，请再接再厉哦");
            }
        }
    }else if(orderType == 3) {
        // 普通用户来处理中奖信息
        if(count > 0) {
            console.log("亲爱的用户，您已抽到10元优惠卷");
        }else {
            console.log("亲爱的用户，请再接再厉哦");
        }
    }
};
```

上面的代码虽然可以实现需求，但是代码不容易扩展且难以阅读，假如以后我想一两个条件，我想充值300元成功的话，可以中奖150元红包，那么这时候又要改动里面的代码,这样业务逻辑与代码耦合性相对比较高，一不小心就改错了代码；这时候我们试着使用职责链模式来依次传递对象来实现；

如下代码：

```
function order500(orderType,isPay,count){
    if(orderType == 1 && isPay == true)    {
        console.log("亲爱的用户，您中奖了100元红包了");
    }else {
        // 自己不处理，传递给下一个对象order200去处理
        order200(orderType,isPay,count);
    }
};
function order200(orderType,isPay,count) {
    if(orderType == 2 && isPay == true) {
        console.log("亲爱的用户，您中奖了20元红包了");
    }else {
        // 自己不处理，传递给下一个对象普通用户去处理
        orderNormal(orderType,isPay,count);
    }
};
function orderNormal(orderType,isPay,count){
    // 普通用户来处理中奖信息
    if(count > 0) {
        console.log("亲爱的用户，您已抽到10元优惠卷");
    }else {
        console.log("亲爱的用户，请再接再厉哦");
    }
}
```

如上代码我们分别使用了三个函数order500，order200，orderNormal来分别处理自己的业务逻辑，如果目前的自己函数不能处理的事情，我们传递给下面的函数去处理，依次类推，直到有一个函数能处理他，否则的话，该职责链模式直接从链中离开，告诉不能处理，抛出错误提示，上面的代码虽然可以当作职责链模式，但是我们看上面的代码可以看到order500函数内依赖了order200这样的函数，这样就必须有这个函数，也违反了面向对象中的 开放-封闭原则。下面我们继续来理解编写 灵活可拆分的职责链节点。

```
function order500(orderType,isPay,count){
    if(orderType == 1 && isPay == true)    {
        console.log("亲爱的用户，您中奖了100元红包了");
    }else {
        //我不知道下一个节点是谁,反正把请求往后面传递
        return "nextSuccessor";
    }
};
function order200(orderType,isPay,count) {
    if(orderType == 2 && isPay == true) {
        console.log("亲爱的用户，您中奖了20元红包了");
    }else {
        //我不知道下一个节点是谁,反正把请求往后面传递
        return "nextSuccessor";
    }
};
function orderNormal(orderType,isPay,count){
    // 普通用户来处理中奖信息
    if(count > 0) {
        console.log("亲爱的用户，您已抽到10元优惠卷");
    }else {
        console.log("亲爱的用户，请再接再厉哦");
    }
}
// 下面需要编写职责链模式的封装构造函数方法
var Chain = function(fn){
    this.fn = fn;
    this.successor = null;
};
Chain.prototype.setNextSuccessor = function(successor){
    return this.successor = successor;
}
// 把请求往下传递
Chain.prototype.passRequest = function(){
    var ret = this.fn.apply(this,arguments);
    if(ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor,arguments);
    }
    return ret;
}
//现在我们把3个函数分别包装成职责链节点：
var chainOrder500 = new Chain(order500);
var chainOrder200 = new Chain(order200);
var chainOrderNormal = new Chain(orderNormal);

// 然后指定节点在职责链中的顺序
chainOrder500.setNextSuccessor(chainOrder200);
chainOrder200.setNextSuccessor(chainOrderNormal);

//最后把请求传递给第一个节点：
chainOrder500.passRequest(1,true,500);  // 亲爱的用户，您中奖了100元红包了
chainOrder500.passRequest(2,true,500);  // 亲爱的用户，您中奖了20元红包了
chainOrder500.passRequest(3,true,500);  // 亲爱的用户，您已抽到10元优惠卷 
chainOrder500.passRequest(1,false,0);   // 亲爱的用户，请再接再厉哦
```

如上代码;分别编写order500，order200，orderNormal三个函数，在函数内分别处理自己的业务逻辑，如果自己的函数不能处理的话，就返回字符串nextSuccessor 往后面传递，然后封装Chain这个构造函数，传递一个fn这个对象实列进来，且有自己的一个属性successor，原型上有2个方法 setNextSuccessor 和 passRequest;setNextSuccessor 这个方法是指定节点在职责链中的顺序的，把相对应的方法保存到this.successor这个属性上，chainOrder500.setNextSuccessor(chainOrder200);chainOrder200.setNextSuccessor(chainOrderNormal);指定链中的顺序，因此this.successor引用了order200这个方法和orderNormal这个方法，因此第一次chainOrder500.passRequest(1,true,500)调用的话，调用order500这个方法，直接输出，第二次调用chainOrder500.passRequest(2,true,500);这个方法从链中首节点order500开始不符合，就返回successor字符串，然后this.successor && this.successor.passRequest.apply(this.successor,arguments);就执行这句代码；上面我们说过this.successor这个属性引用了2个方法 分别为order200和orderNormal，因此调用order200该方法，所以就返回了值，依次类推都是这个原理。那如果以后我们想充值300元的红包的话，我们可以编写order300这个函数，然后实列一下链chain包装起来，指定一下职责链中的顺序即可，里面的业务逻辑不需要做任何处理;

- 理解异步的职责链

上面的只是同步职责链，我们让每个节点函数同步返回一个特定的值”nextSuccessor”，来表示是否把请求传递给下一个节点，在我们开发中会经常碰到ajax异步请求，请求成功后，需要做某某事情，那么这时候如果我们再套用上面的同步请求的话，就不生效了，下面我们来理解下使用异步的职责链来解决这个问题;我们给Chain类再增加一个原型方法Chain.prototype.next，表示手动传递请求给职责链中的一下个节点。

如下代码：

```
function Fn1() {
    console.log(1);
    return "nextSuccessor";
}
function Fn2() {
    console.log(2);
    var self = this;
    setTimeout(function(){
        self.next();
    },1000);
}
function Fn3() {
    console.log(3);
}
// 下面需要编写职责链模式的封装构造函数方法
var Chain = function(fn){
    this.fn = fn;
    this.successor = null;
};
Chain.prototype.setNextSuccessor = function(successor){
    return this.successor = successor;
}
// 把请求往下传递
Chain.prototype.passRequest = function(){
    var ret = this.fn.apply(this,arguments);
    if(ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor,arguments);
    }
    return ret;
}
Chain.prototype.next = function(){
    return this.successor && this.successor.passRequest.apply(this.successor,arguments);
}
//现在我们把3个函数分别包装成职责链节点：
var chainFn1 = new Chain(Fn1);
var chainFn2 = new Chain(Fn2);
var chainFn3 = new Chain(Fn3);

// 然后指定节点在职责链中的顺序
chainFn1.setNextSuccessor(chainFn2);
chainFn2.setNextSuccessor(chainFn3);

chainFn1.passRequest();  // 打印出1，2 过1秒后 会打印出3
```

调用函数 chainFn1.passRequest();后，会先执行发送者Fn1这个函数 打印出1，然后返回字符串 nextSuccessor;

接着就执行return this.successor && this.successor.passRequest.apply(this.successor,arguments);这个函数到Fn2，打印2，接着里面有一个setTimeout定时器异步函数，需要把请求给职责链中的下一个节点，因此过一秒后会打印出3;

职责链模式的优点是：

1. 解耦了请求发送者和N个接收者之间的复杂关系，不需要知道链中那个节点能处理你的请求，所以你只需要把请求传递到第一个节点即可。
2. 链中的节点对象可以灵活地拆分重组，增加或删除一个节点，或者改变节点的位置都是很简单的事情。
3. 我们还可以手动指定节点的起始位置，并不是说非得要从其实节点开始传递的.

缺点：职责链模式中多了一点节点对象，可能在某一次请求过程中，大部分节点没有起到实质性作用，他们的作用只是让

请求传递下去，从性能方面考虑，避免过长的职责链提高性能。

## 六：命令模式的理解

命令模式中的命令指的是一个执行某些特定事情的指令。

命令模式使用的场景有：有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道请求的操作是什么，此时希望用一种松耦合的方式来设计程序代码;使得请求发送者和请求接受者消除彼此代码中的耦合关系。

我们先来列举生活中的一个列子来说明下命令模式：比如我们经常会在天猫上购买东西，然后下订单，下单后我就想收到货，并且希望货物是真的，对于用户来讲它并关心下单后卖家怎么发货，当然卖家发货也有时间的，比如24小时内发货等，用户更不关心快递是给谁派送，当然有的人会关心是什么快递送货的; 对于用户来说，只要在规定的时间内发货，且一般能在相当的时间内收到货就可以，当然命令模式也有撤销命令和重做命令，比如我们下单后，我突然不想买了，我在发货之前可以取消订单，也可以重新下单（也就是重做命令）;比如我的衣服尺码拍错了，我取消该订单，重新拍一个大码的。

- 命令模式的列子

记得我以前刚做前端的那会儿，也就是刚毕业进的第一家公司，进的是做外包项目的公司，该公司一般外包淘宝活动页面及腾讯的游戏页面，我们那会儿应该叫切页面的前端，负责做一些html和css的工作，所以那会儿做腾讯的游戏页面，经常会帮他们做静态页面，比如在页面放几个按钮，我们只是按照设计稿帮腾讯游戏哪方面的把样式弄好，比如说页面上的按钮等事情，比如说具体说明的按钮要怎么操作，点击按钮后会发生什么事情，我们并不知道，我们不知道他们的业务是什么，当然我们知道的肯定会有点击事件，具体要处理什么业务我们并不知道，这里我们就可以使用命令模式来处理了：点击按钮之后，必须向某些负责具体行为的对象发送请求，这些对象就是请求的接收者。但是目前我们并不知道接收者是什么对象，也不知道接受者究竟会做什么事情，这时候我们可以使用命令模式来消除发送者与接收者的代码耦合关系。

我们先使用传统的面向对象模式来设计代码：

假设html结构如下：

```
<button id="button1">刷新菜单目录</button>
<button id="button2">增加子菜单</button>
<button id="button3">删除子菜单</button>
```

JS代码如下：

```
var b1 = document.getElementById("button1"),
    b2 = document.getElementById("button2"),
    b3 = document.getElementById("button3");

// 定义setCommand 函数，该函数负责往按钮上面安装命令。点击按钮后会执行command对象的execute()方法。
var setCommand = function (button, command) {
    button.onclick = function () {
        command.execute();
    }
};
// 下面我们自己来定义各个对象来完成自己的业务操作
var MenuBar = {
    refersh: function () {
        alert("刷新菜单目录");
    }
};
var SubMenu = {
    add: function () {
        alert("增加子菜单");
    },
    del: function () {
        alert("删除子菜单");
    }
};
// 下面是编写命令类
var RefreshMenuBarCommand = function (receiver) {
    this.receiver = receiver;
};
RefreshMenuBarCommand.prototype.execute = function () {
    this.receiver.refersh();
}
// 增加命令操作
var AddSubMenuCommand = function (receiver) {
    this.receiver = receiver;
};
AddSubMenuCommand.prototype.execute = function () {
    this.receiver.add();
}
// 删除命令操作
var DelSubMenuCommand = function (receiver) {
    this.receiver = receiver;
};
DelSubMenuCommand.prototype.execute = function () {
    this.receiver.del();
}
// 最后把命令接收者传入到command对象中，并且把command对象安装到button上面
var refershBtn = new RefreshMenuBarCommand(MenuBar);
var addBtn = new AddSubMenuCommand(SubMenu);
var delBtn = new DelSubMenuCommand(SubMenu);

setCommand(b1, refershBtn);
setCommand(b2, addBtn);
setCommand(b3, delBtn);
```

从上面的命令类代码我们可以看到，任何一个操作都有一个execute这个方法来执行操作;上面的代码是使用传统的面向对象编程来实现命令模式的，命令模式过程式的请求调用封装在command对象的execute方法里。我们有没有发现上面的编写代码有点繁琐呢，我们可以使用javascript中的回调函数来做这些事情的，在面向对象中，命令模式的接收者被当成command对象的属性保存起来，同时约定执行命令的操作调用command.execute方法，但是如果我们使用回调函数的话，那么接收者被封闭在回调函数产生的环境中，执行操作将会更加简单，仅仅执行回调函数即可，下面我们来看看代码如下：

代码如下：

```
var setCommand = function (button, func) {
    button.onclick = function () {
        func();
    }
};
var MenuBar = {
    refersh: function () {
        alert("刷新菜单界面");
    }
};
var SubMenu = {
    add: function () {
        alert("增加菜单");
    }
};
// 刷新菜单
var RefreshMenuBarCommand = function (receiver) {
    return function () {
        receiver.refersh();
    };
};
// 增加菜单
var AddSubMenuCommand = function (receiver) {
    return function () {
        receiver.add();
    };
};
var refershMenuBarCommand = RefreshMenuBarCommand(MenuBar);
// 增加菜单
var addSubMenuCommand = AddSubMenuCommand(SubMenu);
setCommand(b1, refershMenuBarCommand);

setCommand(b2, addSubMenuCommand);
```

我们还可以如下使用javascript回调函数如下编码：

```
// 如下代码上的四个按钮 点击事件
var b1 = document.getElementById("button1"),
    b2 = document.getElementById("button2"),
    b3 = document.getElementById("button3"),
    b4 = document.getElementById("button4");
/*
bindEnv函数负责往按钮上面安装点击命令。点击按钮后，会调用
函数
*/
var bindEnv = function (button, func) {
    button.onclick = function () {
        func();
    }
};
// 现在我们来编写具体处理业务逻辑代码
var Todo1 = {
    test1: function () {
        alert("我是来做第一个测试的");
    }
};
// 实现业务中的增删改操作
var Menu = {
    add: function () {
        alert("我是来处理一些增加操作的");
    },
    del: function () {
        alert("我是来处理一些删除操作的");
    },
    update: function () {
        alert("我是来处理一些更新操作的");
    }
};
// 调用函数
bindEnv(b1, Todo1.test1);
// 增加按钮
bindEnv(b2, Menu.add);
// 删除按钮
bindEnv(b3, Menu.del);
// 更改按钮
bindEnv(b4, Menu.update);
```

- 理解宏命令：

宏命令是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。其实类似把页面的所有函数方法放在一个数组里面去，然后遍历这个数组，依次执行该方法的。

代码如下：

```
var command1 = {
    execute: function(){
        console.log(1);
    }
}; 
var command2 = {
    execute: function(){
        console.log(2);
    }
};
var command3 = {
    execute: function(){
        console.log(3);
    }
};
// 定义宏命令，command.add方法把子命令添加进宏命令对象，
// 当调用宏命令对象的execute方法时，会迭代这一组命令对象，
// 并且依次执行他们的execute方法。
var command = function(){
    return {
        commandsList: [],
        add: function(command){
            this.commandsList.push(command);
        },
        execute: function(){
            for(var i = 0,commands = this.commandsList.length; i < commands; i+=1) {
                this.commandsList[i].execute();
            }
        }
    }
};
// 初始化宏命令
var c = command();
c.add(command1);
c.add(command2);
c.add(command3);
c.execute();  // 1,2,3
```

## 七：模板方法模式
    
模板方法模式由二部分组成，第一部分是抽象父类，第二部分是具体实现的子类，一般的情况下是抽象父类封装了子类的算法框架，包括实现一些公共方法及封装子类中所有方法的执行顺序，子类可以继承这个父类，并且可以在子类中重写父类的方法，从而实现自己的业务逻辑。

比如说我们要实现一个JS功能，比如表单验证等js，那么如果我们没有使用上一章讲的使用javascript中的策略模式来解决表单验证封装代码，而是自己写的临时表单验证功能，肯定是没有进行任何封装的，那么这个时候我们是针对两个值是否相等给用户弹出一个提示，如果再另外一个页面也有一个表单验证，他们判断的方式及业务逻辑基本相同的，只是比较的参数不同而已，我们是不是又要考虑写一个表单验证代码呢？那么现在我们可以考虑使用模板方法模式来解决这个问题；公用的方法提取出来，不同的方法由具体的子类是实现。这样设计代码也可扩展性更强，代码更优等优点~

我们不急着写代码，我们可以先来看一个列子，比如最近经常在qq群里面有很多前端招聘的信息，自己也接到很多公司或者猎头问我是否需要找工作等电话，当然我现在是没有打算找工作的，因为现在有更多的业余时间可以处理自己的事情，所以也觉得蛮不错的~ 我们先来看看招聘中面试这个流程；面试流程对于很多大型公司，比如BAT，面试过程其实很类似；因此我们可以总结面试过程中如下：

1. 笔试：(不同的公司有不同的笔试题目)。
2. 技术面试(一般情况下分为二轮)：第一轮面试你的有可能是你未来直接主管或者未来同事问你前端的一些专业方面的技能及以前做过的项目，在项目中遇到哪些问题及当时是如何解决问题的，还有根据你的简历上的基本信息来交流的，比如说你简历说精通JS，那么人家肯定得问哦~ 第二轮面试一般都是公司的牛人或者架构师来问的，比如问你计算机基本原理，或者问一些数据结构与算法等信息；第二轮面试可能会更深入的去了解你这个人的技术。
3. HR和总监或者总经理面试；那么这一轮的话，HR可能会问下你一些个人基本信息等情况，及问下你今后有什么打算的个人规划什么的，总监或者总经理可能会问下你对他们的网站及产品有了解过没有？及现在他们的产品有什么问题，有没有更好的建议或者如何改善的地方等信息；
4. 最后就是HR和你谈薪资及一般几个工作日可以得到通知，拿到offer(当然不符合的肯定是没有通知的哦)；及自己有没有需要了解公司的情况等等信息；

一般的面试过程都是如上四点下来的，对于不同的公司都差不多的流程的，当然有些公司可能没有上面的详细流程的，我这边这边讲一般的情况下，好了，这边就不扯了，这边也不是讲如何面试的哦，这边只是通过这个列子让我们更加的理解javascript中模板方法模式；所以我们现在回到正题上来；

我们先来分析下上面的流程；我们可以总结如下：

首先我们看一下百度的面试；因此我们可以先定义一个构造函数。

```
var BaiDuInterview = function(){};
```

那么下面就有百度面试的流程哦~

- 第一步笔试

那么我们可以封装一个笔试的方法，代码如下：

```
// baidu 笔试

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的笔试题了~");

};
```

- 第二步技术面试：

```
// 技术面试

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人");

}; 
```

- 第三步HR和总监或者总经理面试，我们可以称之为leader面试；代码如下：

```
// 领导面试

BaiDuInterview.prototype.leader = function(){

    console.log("百度leader来面试了");

};
```

- 第四步和HR谈期望的薪资待遇及HR会告诉你什么时候会有通知，因此我们这边可以称之为这个方法为 是否拿到offer(当然不符合要求肯定是没有通知的哦)；

```
// 等通知

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，到现在都不给我通知");

};
```

如上看到代码的基本结构，但是我们还需要一个初始化方法；代码如下：

```
// 代码初始化

BaiDuInterview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```

综合所述：所有的代码如下：

```
var BaiDuInterview = function(){};


// baidu 笔试

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的题目笔试题了~");

};

// 技术面试

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人");

}; 

// 领导面试

BaiDuInterview.prototype.leader = function(){

    console.log("百度leader来面试了");

};

// 等通知

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，到现在都不给我通知");

};

// 代码初始化

BaiDuInterview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```


上面我们可以看到百度面试的基本流程如上面的代码，那么阿里和腾讯的也和上面的代码类似(这里就不一一贴一样的代码哦)，因此我们可以把公用代码提取出来；我们首先定义一个类，叫面试Interview

那么代码改成如下：

```
var Interview = function(){};
```

- 第一步笔试：

我不管你是百度的笔试还是阿里或者腾讯的笔试题，我这边统称为笔试(WrittenTest)，那么你们公司有不同的笔试题，都交给子类去具体实现，父类方法不管具体如何实现，笔试题具体是什么样的 我都不管。代码变为如下：

```
// 笔试

Interview.prototype.writtenTest = function(){

    console.log("我终于看到笔试题了~");

};
```

- 第二步技术面试，技术面试原理也一样，这里就不多说，直接贴代码：

```
// 技术面试

Interview.prototype.technicalInterview = function(){

    console.log("我是技术负责人负责技术面试");

}; 
```

- 第三步领导面试

```
// 领导面试

Interview.prototype.leader = function(){

    console.log("leader来面试了");

};
```

- 第四步等通知

```
// 等通知

Interview.prototype.waitNotice = function(){

    console.log("人力资源太不给力了，到现在都不给我通知");

};
```

代码初始化方法如下：

```
// 代码初始化

Interview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};
```

- 创建子类

现在我们来创建一个百度的子类来继承上面的父类；代码如下：

```
var BaiDuInterview = function(){};

BaiDuInterview.prototype = new Interview();
```

现在我们可以在子类BaiDuInterview 重写父类Interview中的方法；代码如下：

```
// 子类重写方法 实现自己的业务逻辑

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的笔试题了");

}

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人，想面试找我");

}

BaiDuInterview.prototype.leader = function(){

    console.log("我是百度的leader，不想加班的或者业绩提不上去的给我滚蛋");

}

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，我等的花儿都谢了！！");

}

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```

如上看到，我们直接调用子类baiDuInterview.init()方法，由于我们子类baiDuInterview没有init方法，但是它继承了父类，所以会到父类中查找对应的init方法；所以会迎着原型链到父类中查找；对于其他子类，比如阿里类代码也是一样的，这里就不多介绍了，对于父类这个方法 Interview.prototype.init() 是模板方法，因为他封装了子类中算法框架，它作为一个算法的模板，指导子类以什么样的顺序去执行代码。

- Javascript中的模板模式使用场景

虽然在java中也有子类实现父类的接口，但是我认为javascript中可以和java中不同的，java中可能父类就是一个空的类，子类去实现这个父类的接口，在javascript中我认为完全把公用的代码写在父函数内，如果将来业务逻辑需要更改的话，或者说添加新的业务逻辑，我们完全可以使用子类去重写这个父类，这样的话代码可扩展性强，更容易维护。由于本人不是专业java的，所以描述java中的知识点有误的话，请理解~~


## 八：理解javascript中的策略模式

策略模式的定义是：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

使用策略模式的优点如下：

优点：

1. 策略模式利用组合，委托等技术和思想，有效的避免很多if条件语句。
2. 策略模式提供了开放-封闭原则，使代码更容易理解和扩展。
3. 策略模式中的代码可以复用。

- 一：使用策略模式计算奖金；

下面的demo是我在书上看到的，但是没有关系，我们只是来理解下策略模式的使用而已，我们可以使用策略模式来计算奖金问题；

比如公司的年终奖是根据员工的工资和绩效来考核的，绩效为A的人，年终奖为工资的4倍，绩效为B的人，年终奖为工资的3倍，绩效为C的人，年终奖为工资的2倍；现在我们使用一般的编码方式会如下这样编写代码：

```
var calculateBouns = function(salary,level) {
    if(level === 'A') {
        return salary * 4;
    }
    if(level === 'B') {
        return salary * 3;
    }
    if(level === 'C') {
        return salary * 2;
    }
};
// 调用如下：
console.log(calculateBouns(4000,'A')); // 16000
console.log(calculateBouns(2500,'B')); // 7500
```

第一个参数为薪资，第二个参数为等级；

代码缺点如下：

calculateBouns 函数包含了很多if-else语句。

calculateBouns 函数缺乏弹性，假如还有D等级的话，那么我们需要在calculateBouns 函数内添加判断等级D的if语句；

算法复用性差，如果在其他的地方也有类似这样的算法的话，但是规则不一样，我们这些代码不能通用。

- 使用组合函数重构代码

组合函数是把各种算法封装到一个个的小函数里面，比如等级A的话，封装一个小函数，等级为B的话，也封装一个小函数，以此类推；如下代码：

```
var performanceA = function(salary) {
    return salary * 4;
};
var performanceB = function(salary) {
    return salary * 3;
};
        
var performanceC = function(salary) {
    return salary * 2
};
var calculateBouns = function(level,salary) {
    if(level === 'A') {
        return performanceA(salary);
    }
    if(level === 'B') {
        return performanceB(salary);
    }
    if(level === 'C') {
        return performanceC(salary);
    }
};
// 调用如下
console.log(calculateBouns('A',4500)); // 18000
```

代码看起来有点改善，但是还是有如下缺点：

calculateBouns 函数有可能会越来越大，比如增加D等级的时候，而且缺乏弹性。

- 使用策略模式重构代码

策略模式指的是 定义一系列的算法，把它们一个个封装起来，将不变的部分和变化的部分隔开，实际就是将算法的使用和实现分离出来；算法的使用方式是不变的，都是根据某个算法取得计算后的奖金数，而算法的实现是根据绩效对应不同的绩效规则；

一个基于策略模式的程序至少由2部分组成，第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。第二个部分是环境类Context，该Context接收客户端的请求，随后把请求委托给某一个策略类。我们先使用传统面向对象来实现；

如下代码：

```
var performanceA = function(){};
performanceA.prototype.calculate = function(salary) {
    return salary * 4;
};      
var performanceB = function(){};
performanceB.prototype.calculate = function(salary) {
    return salary * 3;
};
var performanceC = function(){};
performanceC.prototype.calculate = function(salary) {
    return salary * 2;
};
// 奖金类
var Bouns = function(){
    this.salary = null;    // 原始工资
    this.levelObj = null;  // 绩效等级对应的策略对象
};
Bouns.prototype.setSalary = function(salary) {
    this.salary = salary;  // 保存员工的原始工资
};
Bouns.prototype.setlevelObj = function(levelObj){
    this.levelObj = levelObj;  // 设置员工绩效等级对应的策略对象
};
// 取得奖金数
Bouns.prototype.getBouns = function(){
    // 把计算奖金的操作委托给对应的策略对象
    return this.levelObj.calculate(this.salary);
};
var bouns = new Bouns();
bouns.setSalary(10000);
bouns.setlevelObj(new performanceA()); // 设置策略对象
console.log(bouns.getBouns());  // 40000
       
bouns.setlevelObj(new performanceB()); // 设置策略对象
console.log(bouns.getBouns());  // 30000
```

如上代码使用策略模式重构代码，可以看到代码职责更新分明，代码变得更加清晰。

- Javascript版本的策略模式

```
//代码如下：
var obj = {
        "A": function(salary) {
            return salary * 4;
        },
        "B" : function(salary) {
            return salary * 3;
        },
        "C" : function(salary) {
            return salary * 2;
        } 
};
var calculateBouns =function(level,salary) {
    return obj[level](salary);
};
console.log(calculateBouns('A',10000)); // 40000
```

可以看到代码更加简单明了；

策略模式指的是定义一系列的算法，并且把它们封装起来，但是策略模式不仅仅只封装算法，我们还可以对用来封装一系列的业务规则，只要这些业务规则目标一致，我们就可以使用策略模式来封装它们；

表单效验

比如我们经常来进行表单验证，比如注册登录对话框，我们登录之前要进行验证操作：比如有以下几条逻辑：

- 用户名不能为空
- 密码长度不能小于6位。
- 手机号码必须符合格式。

比如HTML代码如下：

```
<form action = "http://www.baidu.com" id="registerForm" method = "post">
        <p>
            <label>请输入用户名：</label>
            <input type="text" name="userName"/>
        </p>
        <p>
            <label>请输入密码：</label>
            <input type="text" name="password"/>
        </p>
        <p>
            <label>请输入手机号码：</label>
            <input type="text" name="phoneNumber"/>
        </p>
</form>
```

我们正常的编写表单验证代码如下：

```
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    if(registerForm.userName.value === '') {
        alert('用户名不能为空');
        return;
    }
    if(registerForm.password.value.length < 6) {
        alert("密码的长度不能小于6位");
        return;
    }
    if(!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
        alert("手机号码格式不正确");
        return;
    }
}
```

但是这样编写代码有如下缺点：

1. registerForm.onsubmit 函数比较大，代码中包含了很多if语句；
2. registerForm.onsubmit 函数缺乏弹性，如果增加了一种新的效验规则，或者想把密码的长度效验从6改成8，我们必须改registerForm.onsubmit 函数内部的代码。违反了开放-封闭原则。
3. 算法的复用性差，如果在程序中增加了另外一个表单，这个表单也需要进行一些类似的效验，那么我们可能又需要复制代码了；

下面我们可以使用策略模式来重构表单效验；

第一步我们先来封装策略对象；如下代码：

```
var strategy = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    } 
};
```

接下来我们准备实现Validator类，Validator类在这里作为Context，负责接收用户的请求并委托给strategy 对象，如下代码：

```
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6 
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};
```

Validator类在这里作为Context，负责接收用户的请求并委托给strategys对象。上面的代码中，我们先创建一个Validator对象，然后通过validator.add方法往validator对象中添加一些效验规则，validator.add方法接收3个参数，如下代码：

```
validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
```

registerForm.password 为效验的input输入框dom节点；

minLength:6： 是以一个冒号隔开的字符串，冒号前面的minLength代表客户挑选的strategys对象，冒号后面的数字6表示在效验过程中所必须验证的参数，minLength:6的意思是效验 registerForm.password 这个文本输入框的value最小长度为6位；如果字符串中不包含冒号，说明效验过程中不需要额外的效验信息；

第三个参数是当效验未通过时返回的错误信息；

当我们往validator对象里添加完一系列的效验规则之后，会调用validator.start()方法来启动效验。如果validator.start()返回了一个errorMsg字符串作为返回值，说明该次效验没有通过，此时需要registerForm.onsubmit方法返回false来阻止表单提交。下面我们来看看初始化代码如下：

```
var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```

下面是所有的代码如下：

```
var strategys = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    } 
};
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6 
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};

var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
};
```

如上使用策略模式来编写表单验证代码可以看到好处了，我们通过add配置的方式就完成了一个表单的效验；这样的话，那么代码可以当做一个组件来使用，并且可以随时调用，在修改表单验证规则的时候，也非常方便，通过传递参数即可调用；

给某个文本输入框添加多种效验规则，上面的代码我们可以看到，我们只是给输入框只能对应一种效验规则，比如上面的我们只能效验输入框是否为空，`validator.add(registerForm.userName,'isNotEmpty','`用户名不能为空');但是如果我们既要效验输入框是否为空，还要效验输入框的长度不要小于10位的话，那么我们期望需要像如下传递参数：

```
validator.add(registerForm.userName,[{strategy:’isNotEmpty’,errorMsg:’用户名不能为空’}，{strategy: 'minLength:6',errorMsg:'用户名长度不能小于6位'}])
```

我们可以编写代码如下：

```
// 策略对象
var strategys = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    } 
};
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rules) {
    var self = this;
    for(var i = 0, rule; rule = rules[i++]; ){
        (function(rule){
            var strategyAry = rule.strategy.split(":");
            var errorMsg = rule.errorMsg;
            self.cache.push(function(){
                var strategy = strategyAry.shift();
                strategyAry.unshift(dom.value);
                strategyAry.push(errorMsg);
                return strategys[strategy].apply(dom,strategyAry);
            });
        })(rule);
    }
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
    var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
    if(msg) {
        return msg;
    }
    }
};
// 代码调用
var registerForm = document.getElementById("registerForm");
var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,[
        {strategy: 'isNotEmpty',errorMsg:'用户名不能为空'},
        {strategy: 'minLength:6',errorMsg:'用户名长度不能小于6位'}
    ]);
    validator.add(registerForm.password,[
        {strategy: 'minLength:6',errorMsg:'密码长度不能小于6位'},
    ]);
    validator.add(registerForm.phoneNumber,[
        {strategy: 'mobileFormat',errorMsg:'手机号格式不正确'},
    ]);
    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
// 点击确定提交
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```

> 注意：如上代码都是按照书上来做的，都是看到书的代码，最主要我们理解策略模式实现，比如上面的表单验证功能是这样封装的代码，我们平时使用jquery插件表单验证代码原来是这样封装的，为此我们以后也可以使用这种方式来封装表单等学习；

## 九：Javascript中理解发布--订阅模式

### 发布订阅模式介绍

发布---订阅模式又叫观察者模式，它定义了对象间的一种一对多的关系，让多个观察者对象同时监听某一个主题对象，当一个对象发生改变时，所有依赖于它的对象都将得到通知。

现实生活中的发布-订阅模式；

比如小红最近在淘宝网上看上一双鞋子，但是呢 联系到卖家后，才发现这双鞋卖光了，但是小红对这双鞋又非常喜欢，所以呢联系卖家，问卖家什么时候有货，卖家告诉她，要等一个星期后才有货，卖家告诉小红，要是你喜欢的话，你可以收藏我们的店铺，等有货的时候再通知你，所以小红收藏了此店铺，但与此同时，小明，小花等也喜欢这双鞋，也收藏了该店铺；等来货的时候就依次会通知他们；

在上面的故事中，可以看出是一个典型的发布订阅模式，卖家是属于发布者，小红，小明等属于订阅者，订阅该店铺，卖家作为发布者，当鞋子到了的时候，会依次通知小明，小红等，依次使用旺旺等工具给他们发布消息；

- 发布订阅模式的优点：

1. 支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象。比如上面的列子，小明，小红不需要天天逛淘宝网看鞋子到了没有，在合适的时间点，发布者(卖家)来货了的时候，会通知该订阅者(小红，小明等人)。
2. 发布者与订阅者耦合性降低，发布者只管发布一条消息出去，它不关心这条消息如何被订阅者使用，同时，订阅者只监听发布者的事件名，只要发布者的事件名不变，它不管发布者如何改变；同理卖家（发布者）它只需要将鞋子来货的这件事告诉订阅者(买家)，他不管买家到底买还是不买，还是买其他卖家的。只要鞋子到货了就通知订阅者即可。

对于第一点，我们日常工作中也经常使用到，比如我们的ajax请求，请求有成功(success)和失败(error)的回调函数，我们可以订阅ajax的success和error事件。我们并不关心对象在异步运行的状态，我们只关心success的时候或者error的时候我们要做点我们自己的事情就可以了~

- 发布订阅模式的缺点：

创建订阅者需要消耗一定的时间和内存。

虽然可以弱化对象之间的联系，如果过度使用的话，反而使代码不好理解及代码不好维护等等。

### 如何实现发布--订阅模式？

1. 首先要想好谁是发布者(比如上面的卖家)。
2. 然后给发布者添加一个缓存列表，用于存放回调函数来通知订阅者(比如上面的买家收藏了卖家的店铺，卖家通过收藏了该店铺的一个列表名单)。
3. 最后就是发布消息，发布者遍历这个缓存列表，依次触发里面存放的订阅者回调函数。

我们还可以在回调函数里面添加一点参数，比如鞋子的颜色，鞋子尺码等信息；

我们先来实现下简单的发布-订阅模式；代码如下：

```
var shoeObj = {}; // 定义发布者
shoeObj.list = []; // 缓存列表 存放订阅者回调函数
        
// 增加订阅者
shoeObj.listen = function(fn) {
    shoeObj.list.push(fn);  // 订阅消息添加到缓存列表
}

// 发布消息
shoeObj.trigger = function(){
    for(var i = 0,fn; fn = this.list[i++];) {
        fn.apply(this,arguments); 
    }
}
// 小红订阅如下消息
shoeObj.listen(function(color,size){
    console.log("颜色是："+color);
    console.log("尺码是："+size);  
});

// 小花订阅如下消息
shoeObj.listen(function(color,size){
    console.log("再次打印颜色是："+color);
    console.log("再次打印尺码是："+size); 
});
shoeObj.trigger("红色",40);
shoeObj.trigger("黑色",42);
```

运行结果如下：

![image](http://ow2n75eab.bkt.clouddn.com/561794-20160218145257206-3.png)

打印如上截图，我们看到订阅者接收到发布者的每个消息，但是呢，对于小红来说，她只想接收颜色为红色的消息，不想接收颜色为黑色的消息，为此我们需要对代码进行如下改造下，我们可以先增加一个key，使订阅者只订阅自己感兴趣的消息。代码如下：

```
var shoeObj = {}; // 定义发布者
shoeObj.list = []; // 缓存列表 存放订阅者回调函数
        
// 增加订阅者
shoeObj.listen = function(key,fn) {
    if(!this.list[key]) {
        // 如果还没有订阅过此类消息，给该类消息创建一个缓存列表
        this.list[key] = []; 
    }
    this.list[key].push(fn);  // 订阅消息添加到缓存列表
}

// 发布消息
shoeObj.trigger = function(){
    var key = Array.prototype.shift.call(arguments); // 取出消息类型名称
    var fns = this.list[key];  // 取出该消息对应的回调函数的集合

    // 如果没有订阅过该消息的话，则返回
    if(!fns || fns.length === 0) {
        return;
    }
    for(var i = 0,fn; fn = fns[i++]; ) {
        fn.apply(this,arguments); // arguments 是发布消息时附送的参数
    }
};

// 小红订阅如下消息
shoeObj.listen('red',function(size){
    console.log("尺码是："+size);  
});

// 小花订阅如下消息
shoeObj.listen('block',function(size){
    console.log("再次打印尺码是："+size); 
});
shoeObj.trigger("red",40);
shoeObj.trigger("block",42);
```

上面的代码，我们再来运行打印下 如下：

![image](http://ow2n75eab.bkt.clouddn.com/561794-20160218145257206-4.png)

可以看到，订阅者只订阅自己感兴趣的消息了；

### 发布---订阅模式的代码封装

我们知道，对于上面的代码，小红去买鞋这么一个对象shoeObj 进行订阅，但是如果以后我们需要对买房子或者其他的对象进行订阅呢，我们需要复制上面的代码，再重新改下里面的对象代码；为此我们需要进行代码封装；

如下代码封装：

```
var event = {
    list: [],
    listen: function(key,fn) {
        if(!this.list[key]) {
            this.list[key] = [];
        }
        // 订阅的消息添加到缓存列表中
        this.list[key].push(fn);
    },
    trigger: function(){
        var key = Array.prototype.shift.call(arguments);
        var fns = this.list[key];
        // 如果没有订阅过该消息的话，则返回
        if(!fns || fns.length === 0) {
            return;
        }
        for(var i = 0,fn; fn = fns[i++];) {
            fn.apply(this,arguments);
        }
    }
};
```

我们再定义一个initEvent函数，这个函数使所有的普通对象都具有发布订阅功能，如下代码：

```
var initEvent = function(obj) {
    for(var i in event) {
        obj[i] = event[i];
    }
};
// 我们再来测试下，我们还是给shoeObj这个对象添加发布-订阅功能；
var shoeObj = {};
initEvent(shoeObj);

// 小红订阅如下消息
shoeObj.listen('red',function(size){
    console.log("尺码是："+size);  
});

// 小花订阅如下消息
shoeObj.listen('block',function(size){
    console.log("再次打印尺码是："+size); 
});
shoeObj.trigger("red",40);
shoeObj.trigger("block",42);
```

### 如何取消订阅事件？

比如上面的列子，小红她突然不想买鞋子了，那么对于卖家的店铺他不想再接受该店铺的消息，那么小红可以取消该店铺的订阅。

如下代码：

```
event.remove = function(key,fn){
    var fns = this.list[key];
    // 如果key对应的消息没有订阅过的话，则返回
    if(!fns) {
        return false;
    }
    // 如果没有传入具体的回调函数，表示需要取消key对应消息的所有订阅
    if(!fn) {
        fn && (fns.length = 0);
    }else {
        for(var i = fns.length - 1; i >= 0; i--) {
            var _fn = fns[i];
            if(_fn === fn) {
                fns.splice(i,1); // 删除订阅者的回调函数
            }
        }
    }
};
// 测试代码如下：
var initEvent = function(obj) {
    for(var i in event) {
        obj[i] = event[i];
    }
};
var shoeObj = {};
initEvent(shoeObj);

// 小红订阅如下消息
shoeObj.listen('red',fn1 = function(size){
    console.log("尺码是："+size);  
});

// 小花订阅如下消息
shoeObj.listen('red',fn2 = function(size){
    console.log("再次打印尺码是："+size); 
});
shoeObj.remove("red",fn1);
shoeObj.trigger("red",42);
```

运行结果如下：

![image](http://ow2n75eab.bkt.clouddn.com/561794-20160218145257206-5.png)

### 全局--发布订阅对象代码封装

我们再来看看我们传统的ajax请求吧，比如我们传统的ajax请求，请求成功后需要做如下事情：

1. 渲染数据。
2. 使用数据来做一个动画。

那么我们以前肯定是如下写代码：

```
$.ajax(“http://127.0.0.1/index.php”,function(data){
    rendedData(data);  // 渲染数据
    doAnimate(data);  // 实现动画 
});
```

假如以后还需要做点事情的话，我们还需要在里面写调用的方法；这样代码就耦合性很高，那么我们现在使用发布-订阅模式来看如何重构上面的业务需求代码；

```
$.ajax(“http://127.0.0.1/index.php”,function(data){
    Obj.trigger(‘success’,data);  // 发布请求成功后的消息
});
// 下面我们来订阅此消息，比如我现在订阅渲染数据这个消息；
Obj.listen(“success”,function(data){
   renderData(data);
});
// 订阅动画这个消息
Obj.listen(“success”,function(data){
   doAnimate(data); 
});
```

为此我们可以封装一个全局发布-订阅模式对象；如下代码：

```
var Event = (function(){
    var list = {},
          listen,
          trigger,
          remove;
          listen = function(key,fn){
            if(!list[key]) {
                list[key] = [];
            }
            list[key].push(fn);
        };
        trigger = function(){
            var key = Array.prototype.shift.call(arguments),
                 fns = list[key];
            if(!fns || fns.length === 0) {
                return false;
            }
            for(var i = 0, fn; fn = fns[i++];) {
                fn.apply(this,arguments);
            }
        };
        remove = function(key,fn){
            var fns = list[key];
            if(!fns) {
                return false;
            }
            if(!fn) {
                fns && (fns.length = 0);
            }else {
                for(var i = fns.length - 1; i >= 0; i--){
                    var _fn = fns[i];
                    if(_fn === fn) {
                        fns.splice(i,1);
                    }
                }
            }
        };
        return {
            listen: listen,
            trigger: trigger,
            remove: remove
        }
})();
// 测试代码如下：
Event.listen("color",function(size) {
    console.log("尺码为:"+size); // 打印出尺码为42
});
Event.trigger("color",42);
```

### 理解模块间通信

我们使用上面封装的全局的发布-订阅对象来实现两个模块之间的通信问题；比如现在有一个页面有一个按钮，每次点击此按钮后，div中会显示此按钮被点击的总次数；如下代码：

```
<button id="count">点将我</button>

<div id="showcount"></div>
```

我们中的a.js 负责处理点击操作 及发布消息；如下JS代码：

```
var a = (function(){
    var count = 0;
    var button = document.getElementById("count");
    button.onclick = function(){
        Event.trigger("add",count++);
    }
})();
```

b.js 负责监听add这个消息，并把点击的总次数显示到页面上来；如下代码：

```
var b = (function(){
    var div = document.getElementById("showcount");
    Event.listen('add',function(count){
        div.innerHTML = count;
    });
})();
```

下面是html代码如下，JS应用如下引用即可：

```
<!doctype html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script src="global.js"></script>
 </head>
 <body>
    <button id="count">点将我</button>
    <div id="showcount"></div>
    <script src = "a.js"></script>
    <script src = "b.js"></script>
 </body>
</html>
```

如上代码，当点击一次按钮后，showcount的div会自动加1，如上演示的是2个模块之间如何使用发布-订阅模式之间的通信问题；

其中global.js 就是我们上面封装的全局-发布订阅模式对象的封装代码；

## 十：理解中介者模式
    
先来理解这么一个问题，假如我们前端开发接的需求是需求方给我们需求，可能一个前端开发会和多个需求方打交道，所以会保持多个需求方的联系，那么在程序里面就意味着保持多个对象的引用，当程序的规模越大，对象会越来越多，他们之间的关系会越来越复杂，那现在假如现在有一个中介者(假如就是我们的主管)来对接多个需求方的需求，那么需求方只需要把所有的需求给我们主管就可以，主管会依次看我们的工作量来给我们分配任务，这样的话，我们前端开发就不需要和多个业务方联系，我们只需要和我们主管(也就是中介)联系即可，这样的好处就弱化了对象之间的耦合。

日常生活中的列子：

中介者模式对于我们日常生活中经常会碰到，比如我们去房屋中介去租房，房屋中介人在租房者和房东出租者之间形成一条中介;租房者并不关心租谁的房，房东出租者也并不关心它租给谁，因为有中介，所以需要中介来完成这场交易。

中介者模式的作用是解除对象与对象之间的耦合关系，增加一个中介对象后，所有的相关对象都通过中介者对象来通信，而不是相互引用，所以当一个对象发送改变时，只需要通知中介者对象即可。中介者使各个对象之间耦合松散，而且可以独立地改变它们之间的交互。

实现中介者的列子如下：

不知道大家有没有玩过英雄杀这个游戏，最早的时候，英雄杀有2个人(分别是敌人和自己)；我们针对这个游戏先使用普通的函数来实现如下：

比如先定义一个函数，该函数有三个方法，分别是win(赢), lose(输)，和die(敌人死亡)这三个函数；只要一个玩家死亡该游戏就结束了，同时需要通知它的对手胜利了; 代码需要编写如下：

```
function Hero(name) {
    this.name = name;
    this.enemy = null; 
}
Hero.prototype.win = function(){
    console.log(this.name + 'Won');
}
Hero.prototype.lose = function(){
    console.log(this.name + 'lose');
}
Hero.prototype.die = function(){
    this.lose();
    this.enemy.win();
}
// 初始化2个对象
var h1 = new Hero("朱元璋");
var h2 = new Hero("刘伯温");
// 给玩家设置敌人
h1.enemy = h2;
h2.enemy = h1;
// 朱元璋死了 也就输了
h1.die();  // 输出 朱元璋lose 刘伯温Won
```

现在我们再来为游戏添加队友

比如现在我们来为游戏添加队友，比如英雄杀有6人一组，那么这种情况下就有队友，敌人也有3个；因此我们需要区分是敌人还是队友需要队的颜色这个字段，如果队的颜色相同的话，那么就是同一个队的，否则的话就是敌人；

我们可以先定义一个数组players来保存所有的玩家，在创建玩家之后，循环players来给每个玩家设置队友或者敌人;

```
var players = [];
```

接着我们再来编写Hero这个函数；代码如下：

```
var players = []; // 定义一个数组 保存所有的玩家
function Hero(name,teamColor) {
    this.friends = [];    //保存队友列表
    this.enemies = [];    // 保存敌人列表
    this.state = 'live';  // 玩家状态
    this.name = name;     // 角色名字
    this.teamColor = teamColor; // 队伍的颜色
}
Hero.prototype.win = function(){
    // 赢了
    console.log("win:" + this.name);
};
Hero.prototype.lose = function(){
    // 输了
    console.log("lose:" + this.name);
};
Hero.prototype.die = function(){
    // 所有队友死亡情况 默认都是活着的
    var all_dead = true;
    this.state = 'dead'; // 设置玩家状态为死亡
    for(var i = 0,ilen = this.friends.length; i < ilen; i+=1) {
        // 遍历，如果还有一个队友没有死亡的话，则游戏还未结束
        if(this.friends[i].state !== 'dead') {
            all_dead = false; 
            break;
        }
    }
    if(all_dead) {
        this.lose();  // 队友全部死亡，游戏结束
        // 循环 通知所有的玩家 游戏失败
        for(var j = 0,jlen = this.friends.length; j < jlen; j+=1) {
            this.friends[j].lose();
        }
        // 通知所有敌人游戏胜利
        for(var j = 0,jlen = this.enemies.length; j < jlen; j+=1) {
            this.enemies[j].win();
        }
    }
}
// 定义一个工厂类来创建玩家 
var heroFactory = function(name,teamColor) {
    var newPlayer = new Hero(name,teamColor);
    for(var i = 0,ilen = players.length; i < ilen; i+=1) {
        // 如果是同一队的玩家
        if(players[i].teamColor === newPlayer.teamColor) {
            // 相互添加队友列表
            players[i].friends.push(newPlayer);
            newPlayer.friends.push(players[i]);
        }else {
            // 相互添加到敌人列表
            players[i].enemies.push(newPlayer);
            newPlayer.enemies.push(players[i]);
        }
    }
    players.push(newPlayer);
    return newPlayer;
};
        // 红队
var p1 = heroFactory("aa",'red'),
    p2 = heroFactory("bb",'red'),
    p3 = heroFactory("cc",'red'),
    p4 = heroFactory("dd",'red');
        
// 蓝队
var p5 = heroFactory("ee",'blue'),
    p6 = heroFactory("ff",'blue'),
    p7 = heroFactory("gg",'blue'),
    p8 = heroFactory("hh",'blue');
// 让红队玩家全部死亡
p1.die();
p2.die();
p3.die();
p4.die();
// lose:dd lose:aa lose:bb lose:cc
// win:ee win:ff win:gg win:hh
```

如上代码：Hero函数有2个参数，分别是name(玩家名字)和teamColor(队颜色)，

首先我们可以根据队颜色来判断是队友还是敌人；同样也有三个方法win(赢)，lose(输)，和die(死亡)；如果每次死亡一个人的时候，循环下该死亡的队友有没有全部死亡，如果全部死亡了的话，就输了，因此需要循环他们的队友，分别告诉每个队友中的成员他们输了，同时需要循环他们的敌人，分别告诉他们的敌人他们赢了；因此每次死了一个人的时候，都需要循环一次判断他的队友是否都死亡了；因此每个玩家和其他的玩家都是紧紧耦合在一起了。

下面我们可以使用中介者模式来改善上面的demo；

首先我们仍然定义Hero构造函数和Hero对象原型的方法，在Hero对象的这些原型方法中，不再负责具体的执行的逻辑，而是把操作转交给中介者对象，中介者对象来负责做具体的事情，我们可以把中介者对象命名为playerDirector;

在playerDirector开放一个对外暴露的接口ReceiveMessage，负责接收player对象发送的消息，而player对象发送消息的时候，总是把自身的this作为参数发送给playerDirector，以便playerDirector 识别消息来自于那个玩家对象。

代码如下：

```
var players = []; // 定义一个数组 保存所有的玩家
function Hero(name, teamColor) {
    this.state = 'live';  // 玩家状态
    this.name = name;     // 角色名字
    this.teamColor = teamColor; // 队伍的颜色
}
Hero.prototype.win = function () {
    // 赢了
    console.log("win:" + this.name);
};
Hero.prototype.lose = function () {
    // 输了
    console.log("lose:" + this.name);
};
// 死亡
Hero.prototype.die = function () {
    this.state = 'dead';
    // 给中介者发送消息，玩家死亡
    playerDirector.ReceiveMessage('playerDead', this);
}
// 移除玩家
Hero.prototype.remove = function () {
    // 给中介者发送一个消息，移除一个玩家
    playerDirector.ReceiveMessage('removePlayer', this);
};
// 玩家换队
Hero.prototype.changeTeam = function (color) {
    // 给中介者发送一个消息，玩家换队
    playerDirector.ReceiveMessage('changeTeam', this, color);
};
// 定义一个工厂类来创建玩家 
var heroFactory = function (name, teamColor) {
    // 创建一个新的玩家对象
    var newHero = new Hero(name, teamColor);
    // 给中介者发送消息，新增玩家
    playerDirector.ReceiveMessage('addPlayer', newHero);
    return newHero;
};
var playerDirector = (function () {
    var players = {},  // 保存所有的玩家
        operations = {}; // 中介者可以执行的操作
    // 新增一个玩家操作
    operations.addPlayer = function (player) {
        // 获取玩家队友的颜色
        var teamColor = player.teamColor;
        // 如果该颜色的玩家还没有队伍的话，则新成立一个队伍
        players[teamColor] = players[teamColor] || [];
        // 添加玩家进队伍
        players[teamColor].push(player);
    };
    // 移除一个玩家
    operations.removePlayer = function (player) {
        // 获取队伍的颜色
        var teamColor = player.teamColor,
            // 获取该队伍的所有成员
            teamPlayers = players[teamColor] || [];
        // 遍历
        for (var i = teamPlayers.length - 1; i >= 0; i--) {
            if (teamPlayers[i] === player) {
                teamPlayers.splice(i, 1);
            }
        }
    };
    // 玩家换队
    operations.changeTeam = function (player, newTeamColor) {
        // 首先从原队伍中删除
        operations.removePlayer(player);
        // 然后改变队伍的颜色
        player.teamColor = newTeamColor;
        // 增加到队伍中
        operations.addPlayer(player);
    };
    // 玩家死亡
    operations.playerDead = function (player) {
        var teamColor = player.teamColor,
            // 玩家所在的队伍
            teamPlayers = players[teamColor];

        var all_dead = true;
        //遍历 
        for (var i = 0, player; player = teamPlayers[i++];) {
            if (player.state !== 'dead') {
                all_dead = false;
                break;
            }
        }
        // 如果all_dead 为true的话 说明全部死亡
        if (all_dead) {
            for (var i = 0, player; player = teamPlayers[i++];) {
                // 本队所有玩家lose
                player.lose();
            }
            for (var color in players) {
                if (color !== teamColor) {
                    // 说明这是另外一组队伍
                    // 获取该队伍的玩家
                    var teamPlayers = players[color];
                    for (var i = 0, player; player = teamPlayers[i++];) {
                        player.win(); // 遍历通知其他玩家win了
                    }
                }
            }
        }
    };
    var ReceiveMessage = function () {
        // arguments的第一个参数为消息名称 获取第一个参数
        var message = Array.prototype.shift.call(arguments);
        operations[message].apply(this, arguments);
    };
    return {
        ReceiveMessage: ReceiveMessage
    };
})();
// 红队
var p1 = heroFactory("aa", 'red'),
    p2 = heroFactory("bb", 'red'),
    p3 = heroFactory("cc", 'red'),
    p4 = heroFactory("dd", 'red');

// 蓝队
var p5 = heroFactory("ee", 'blue'),
    p6 = heroFactory("ff", 'blue'),
    p7 = heroFactory("gg", 'blue'),
    p8 = heroFactory("hh", 'blue');
// 让红队玩家全部死亡
p1.die();
p2.die();
p3.die();
p4.die();
// lose:aa lose:bb lose:cc lose:dd 
// win:ee win:ff win:gg win:hh
```

我们可以看到如上代码；玩家与玩家之间的耦合代码已经解除了，而把所有的逻辑操作放在中介者对象里面进去处理，某个玩家的任何操作不需要去遍历去通知其他玩家，而只是需要给中介者发送一个消息即可，中介者接受到该消息后进行处理，处理完消息之后会把处理结果反馈给其他的玩家对象。使用中介者模式解除了对象与对象之间的耦合代码; 使程序更加的灵活.

- 中介者模式实现购买商品的列子

下面的列子是书上的列子，比如在淘宝或者天猫的列子不是这样实现的，也没有关系，我们可以改动下即可，我们最主要来学习下使用中介者模式来实现的思路。

首先先介绍一下业务：在购买流程中，可以选择手机的颜色以及输入购买的数量，同时页面中有2个展示区域，分别显示用户刚刚选择好的颜色和数量。还有一个按钮动态显示下一步的操作，我们需要查询该颜色手机对应的库存，如果库存数量小于这次的购买数量，按钮则被禁用并且显示库存不足的文案，反之按钮高亮且可以点击并且显示假如购物车。

HTML代码如下：


选择颜色:

```
<select id="colorSelect">
    <option value="">请选择</option>
    <option value="red">红色</option>
    <option value="blue">蓝色</option>
</select>
<p>输入购买的数量: <input type="text" id="numberInput"/></p>
你选择了的颜色：<div id="colorInfo"></div>
<p>你输入的数量: <div id="numberInfo"></div> </p>
<button id="nextBtn" disabled="true">请选择手机颜色和购买数量</button>
```

首先页面上有一个select选择框，然后有输入的购买数量输入框，还有2个展示区域，分别是选择的颜色和输入的数量的显示的区域，还有下一步的按钮操作；

我们先定义一下：

假设我们提前从后台获取到所有颜色手机的库存量

```
var goods = {
    // 手机库存
    "red": 6,
    "blue": 8
};
```

接着 我们下面分别来监听colorSelect的下拉框的onchange事件和numberInput输入框的oninput的事件，然后在这两个事件中作出相应的处理

常规的JS代码如下：

```
// 假设我们提前从后台获取到所有颜色手机的库存量
var goods = {
    // 手机库存
    "red": 6,
    "blue": 8
};
/*
我们下面分别来监听colorSelect的下拉框的onchange事件和numberInput输入框的oninput的事件，
然后在这两个事件中作出相应的处理
*/
var colorSelect = document.getElementById("colorSelect"),
    numberInput = document.getElementById("numberInput"),
    colorInfo = document.getElementById("colorInfo"),
    numberInfo = document.getElementById("numberInfo"),
    nextBtn = document.getElementById("nextBtn");

// 监听change事件
colorSelect.onchange = function (e) {
    select();
};
numberInput.oninput = function () {
    select();
};
function select() {
    var color = colorSelect.value,   // 颜色
        number = numberInput.value,  // 数量
        stock = goods[color];  // 该颜色手机对应的当前库存

    colorInfo.innerHTML = color;
    numberInfo.innerHTML = number;

    // 如果用户没有选择颜色的话，禁用按钮
    if (!color) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "请选择手机颜色";
        return;
    }
    // 判断用户输入的购买数量是否是正整数
    var reg = /^\d+$/g;
    if (!reg.test(number)) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "请输入正确的购买数量";
        return;
    }
    // 如果当前选择的数量大于当前的库存的数量的话，显示库存不足
    if (number > stock) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "库存不足";
        return;
    }
    nextBtn.disabled = false;
    nextBtn.innerHTML = "放入购物车";
}
```

上面的代码虽然是完成了页面上的需求，但是我们的代码都耦合在一起了，目前虽然问题不是很多，假如随着以后需求的改变，SKU属性越来越多的话，比如页面增加一个或者多个下拉框的时候，代表选择手机内存，现在我们需要计算颜色，内存和购买数量，来判断nextBtn是显示库存不足还是放入购物车；代码如下：

HTML代码如下：

选择颜色:

```
<select id="colorSelect">
    <option value="">请选择</option>
    <option value="red">红色</option>
    <option value="blue">蓝色</option>
</select>
<br/>
<br/>
```

选择内存：

```
<select id="memorySelect">
    <option value="">请选择</option>
    <option value="32G">32G</option>
    <option value="64G">64G</option>
</select>
<p>输入购买的数量: <input type="text" id="numberInput"/></p>
你选择了的颜色：<div id="colorInfo"></div>
你选择了内存：<div id="memoryInfo"></div>
<p>你输入的数量: <div id="numberInfo"></div> </p>
<button id="nextBtn" disabled="true">请选择手机颜色和购买数量</button>
```

JS代码变为如下：

```
// 假设我们提前从后台获取到所有颜色手机的库存量
var goods = {
    // 手机库存
    "red|32G": 6,
    "red|64G": 16,
    "blue|32G": 8,
    "blue|64G": 18
};
/*
我们下面分别来监听colorSelect的下拉框的onchange事件和numberInput输入框的oninput的事件，
然后在这两个事件中作出相应的处理
 */
var colorSelect = document.getElementById("colorSelect"),
    memorySelect = document.getElementById("memorySelect"),
    numberInput = document.getElementById("numberInput"),
    colorInfo = document.getElementById("colorInfo"),
    numberInfo = document.getElementById("numberInfo"),
    memoryInfo = document.getElementById("memoryInfo"),
    nextBtn = document.getElementById("nextBtn");

// 监听change事件
colorSelect.onchange = function () {
    select();
};
numberInput.oninput = function () {
    select();
};
memorySelect.onchange = function () {
    select();
};
function select() {
    var color = colorSelect.value,   // 颜色
        number = numberInput.value,  // 数量
        memory = memorySelect.value, // 内存
        stock = goods[color + '|' + memory];  // 该颜色手机对应的当前库存

    colorInfo.innerHTML = color;
    numberInfo.innerHTML = number;
    memoryInfo.innerHTML = memory;
    // 如果用户没有选择颜色的话，禁用按钮
    if (!color) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "请选择手机颜色";
        return;
    }
    // 判断用户输入的购买数量是否是正整数
    var reg = /^\d+$/g;
    if (!reg.test(number)) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "请输入正确的购买数量";
        return;
    }
    // 如果当前选择的数量大于当前的库存的数量的话，显示库存不足
    if (number > stock) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = "库存不足";
        return;
    }
    nextBtn.disabled = false;
    nextBtn.innerHTML = "放入购物车";
}
```

一般的代码就是这样的，感觉使用中介者模式代码也类似，这里就不多介绍了，书上的代码说有优点，但是个人感觉没有什么很大的区别，因此这里就不再使用中介者模式来编写代码了。