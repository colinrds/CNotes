[toc]

是一套可复用的，被众人知晓，经过编目分明的，经验的总结。
作用：使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性

## 模式类型

- 创建型设计模式：解决对象在创建时产生的一系列问题。
- 结构型设计模式：解决类或对象之间组合时产生的一系列问题
- 行为型设计模式：解决类或对象之间的交互以及职责分配的一些列问题

## 23种设计模式
- 创建型模式：单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式。 （5）
- 结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。（7）
- 行为型模式：模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式（Interpreter模式）、状态模式、策略模式、职责链模式(责任链模式)、访问者模式。（11）

## 设计模式，框架，架构与库

- 设计模式：是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结，它强调的是一个设计问题的解决方法
- 框架：软件框架是项目软件开发过程中提取特定领域软件的共性部分形成的体系结构，不同领域的软件项目有着不同的框架类型。框架不是现成可用的应用系统。而是一个半成品，提供了诸多服务，开发人员进行二次开发，实现具体功能的应用系统。
- 架构：简单的说架构就是一个蓝图，是一种设计方案，将客户的不同需求抽象成为抽象组件，并能够描述这些抽象组件之间的通信和调用，偏重于设计。框架比架构更具体，更偏重于技术
- 工具库：是类（方法）的集合，这些类之间可能是相互独立的。可以直接调用它，而不必再写一个。框架也往往是类（方法）的集合；但框架中的各个类并不是孤立的，而框架中的业务逻辑代码是将不同的类“连”在一起，在它们之间建立协作关系

设计模式研究的是针对单一问题的设计思路和解决方法，一个模式可应用于不同的框架和被不同的语言所实现；而框架则是一个应用的体系结构，是一种或多种设计模式和代码的混合体虽然它们有所不同，但却共同致力于使人们的设计可以被重用，在思想上存在着统一性的特点，因而设计模式的思想可以在框架设计中进行应用。

## 工厂模式
### 简单工厂
定义：用来创建一种类型的产品实例，所以他创建的对象单一

特点：

- 通过将创建过程寄生在工厂内，可以解决全局变量污染问题。
- 创建的产品对象单一。
- 对工厂进行改造可以适应不同的需求。

```javascript
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<canvas id="canvas" width="800" height="400" style="background: #000;"></canvas>

<body>

    <script type="text/javascript">
        //简单工厂模式    通过一个 函数，来 创建多个对象。  //处理大量具有相同属性的小对象
         //例如：在 canvas中创建  小球

        var cvs = document.getElementById("canvas");

         //获取上下文对象
        var ctx = cvs.getContext('2d');

        var w = cvs.width;
        var h = cvs.height;

         //小球 构造函数
        var Ball = function (x, y, r) {

            this.x = x;
            this.y = y;
            this.r = r;

        }

         //扩展画小球方法
        Ball.prototype = {

            'draw': function () {

                ctx.beginPath();

                ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false);
                ctx.fillStyle = 'peru';
                ctx.fill();

                ctx.closePath();
            }

        }

         //小球工厂   //通过一个函数来周转 ，把对象在函数内部实例化。 返回实例化对象。  //通过第三方的类完成松耦合。
         //① 可以 减少全局变量 . ② 再次进行对象改造， ③ 返回对象 单一。
        function BallFectory() {

            //将 具有相同属性的小对象 放入 来进行实例化   

            var x = Math.random() * w;
            var y = Math.random() * h;
            var r = Math.random() * 10 + 10;

            var b = new Ball(x, y, r);

            return b; //返回小球对象实例

        }

         //测试
        var b = BallFectory();

        b.draw();

        BallFectory().draw();
    </script>
</body>
```
批量生产

```javascript
<head>
    <meta charset="UTF-8">
    <title></title>
</head>

<canvas id="canvas" width="800" height="400" style="background: #000;"></canvas>

<body>

    <script type="text/javascript">
        //简单工厂模式    通过一个 函数，来 创建多个对象。  //处理大量具有相同属性的小对象
         //例如：在 canvas中创建  小球

        var cvs = document.getElementById("canvas");

         //获取上下文对象
        var ctx = cvs.getContext('2d');

        var w = cvs.width;
        var h = cvs.height;

         //小球 构造函数
        var Ball = function (x, y, r) {

            this.x = x;
            this.y = y;
            this.r = r;

        }

         //扩展画小球方法
        Ball.prototype = {

            'draw': function () {

                ctx.beginPath();

                ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI, false);
                ctx.fillStyle = 'peru';
                ctx.fill();

                ctx.closePath();
            }

        }

         //小球工厂   //通过一个函数来周转 ，把对象在函数内部实例化。 返回实例化对象。  
         //① 可以 减少全局变量 . ② 再次进行对象改造， ③ 返回对象 单一。

         //            第二,三种, 直接对 工厂进行改造
        function BallFectory(num) {

            if (num) {

                var result = [];

                for (var i = 0; i < num; i++) {

                    //reslut.push(BallFectory()); //第二种
                    reslut.push(arguments.callee()); //第三种

                }

                return result;

            } else {

                //将 具有相同属性的小对象 放入 来进行实例化   

                var x = Math.random() * w;
                var y = Math.random() * h;
                var r = Math.random() * 10 + 10;

                var b = new Ball(x, y, r);

                return b; //返回小球对象实例

            }

        }


         //第一种：
         // 批量生产
         // @param {Number} num 需要生产几个.
         //                function ManyBallFentery ( num ) {
         //                    
         //                    //存放  返回小球对象实例
         //                    var reslut = [];
         //                    
         //                    if ( num ) {
         //                        
         //                        for ( var i=0; i<num; i++ ) {
         //                            
         //                            reslut.push(BallFectory());
         //                            
         //                        }
         //                        
         //                    }
         //                    
         //                    return reslut;
         //                    
         //                }
         //                
         //                ManyBallFentery(10);
    </script>

</body>
```

第一种： 
在外部创建一个批量生产的工厂，然后再内部调用工厂，它的问题是需要另外创建一个工厂

第二种：
通过在工厂方法内部进行分支判断，决定创建单个产品或者品量单品。
这种方式在工厂内部调用该厂，依赖了工厂名称。

第三种：
通过在工厂方法内部进行分支判断，决定创建单个产品或者品量单品。
这种方式在工厂内部调用了，arguments.callee解决了对工厂名称的依赖。

###寄生增强工厂
通过对寄生在工厂内部的对象增添方法属性，使在不改变原类的基础上，完成对类的拓展

- 在工厂内部实例化类 这一步叫做寄生
- 对实例化的类拓展方法和属性 这一步叫做增强
- 将这个对象返回。 这一步是工厂。
   
```javascript
<head>
    <meta charset="UTF-8">
    <title></title>
</head>

<body>

    <canvas id="canvas" width="600" height="400" style="border: 1px solid #d98e57;"></canvas>

    <script type="text/javascript">
        var cvs = document.getElementById("canvas");

        var ctx = cvs.getContext('2d');

        var w = cvs.width;
        var h = cvs.height;

        var Ball = function (x, y, r) {

            this.x = x;
            this.y = y;
            this.r = r;

        }

        Ball.prototype = {

            'draw': function () {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI);
                ctx.fillStyle = '#52bf56';
                ctx.fill();
                ctx.closePath();
                ctx.fill();
            }

        }

         // 实现需求，创建的小球不仅仅画在画布里，而且让它们动起来
        function BallFectory(x, y, r) {

            //对类 进行实例化，  -- 寄生
            var x = x || Math.random() * w;
            var y = y || Math.random() * h;
            var r = r || Math.random() * 10 + 10;

            var ball = new Ball(x, y, r);

            //对实例化对象的拓展  -- 增强
            ball.vx = Math.random() * 5;
            ball.vy = Math.random() * 5;
            ball.changeColor = function () {

                return Math.random() > .5 ? '#52bf56' : 'coral';

            }

            ball.move = function () {

                var _this = this;

                setInterval(function () {

                    //判断边界
                    if (_this.x + _this.r > w || _this.x - _this.r < 0) {

                        //方向相反
                        _this.vx *= -1;

                    }
                    if (_this.y + _this.r > h || _this.y - _this.r < 0) {

                        //方向相反
                        _this.vy *= -1;

                    }

                    //添加能量
                    _this.x += _this.vx;
                    _this.y += _this.vy;

                    // 清除上一帧，并重新绘制小球
                    ctx.clearRect(0, 0, w, h);
                    ctx.fillStyle = _this.changeColor();

                    //绘画
                    _this.draw();

                }, 30);

            }

            //返回 实例对象
            return ball;

        }

        var ball = new BallFectory(null, 200);

         //执行动画
        ball.move();
    </script>

</body>
```

### 安全工厂模式
又叫：安全类

特点，不论在调用的时候有没有new关键字，得到的结果都是一样的。

- 判断this是否是只想当前对象的。通过instanceof
- 如果是通过new关键字创建的，就直接对this赋值
- 如果不是，主动创建，并返回该实例。

```javascript
//创建 Book类
//安全工厂
var Book = function ( title ) {
    
    //判断是否调用 new 关键字
    if ( this instanceof Book ) {
        
        this.title = title;
        
    } else {
        
        return new Book(title);    
        
    }
    
}

//var b1 = new Book('设计模式1');
var b2 = Book('设计模式2');

//console.log( b1 ,b1 instanceof Book );  //true
console.log( b2 ,b2 instanceof Book ); //true
```

instanceof运算符可以用来判断某个构造函数的prototype属性所指向的對象是否存在于另外一个要检测对象的原型链上。

### 工厂方法
通过对产品类的抽象使其创建业务主要负责用于创建多类产品的实例
特点：

- 创建多类对象
- 也是对类的再封装

步骤：

- 声明参数
- 循环创建类
- 对类的添加方法
- 将实例化对象返回

```
<head>
    <meta charset="UTF-8">
    <title></title>
</head>

<body>
    <style type="text/css">
        html,
        body {
            height: 100%;
        }
        div {
            position: absolute;
        }
        .rect {
            width: 100px;
            height: 100px;
            background: red;
        }
        .cricle {
            width: 100px;
            height: 100px;
            background: green;
            border-radius: 50%;
        }
        .arrow-top {
            width: 0;
            border-top: 100px solid blue;
            border-bottom: 100px solid transparent;
            border-left: 100px solid transparent;
            border-right: 100px solid transparent;
        }
    </style>
    <script type="text/javascript">
        // 获取页面的宽高
        var w = document.body.offsetWidth;
        var h = document.body.offsetHeight;
         // 创建正方形
        var Rect = function (x, y) {
            // 构建元素
            this.dom = document.createElement('div');
            // 设置坐标
            this.dom.style.left = x + 'px';
            this.dom.style.top = y + 'px';
            // 初始化形状
            this.dom.className = 'rect';
        }
        Rect.prototype = {
            init: function () {
                // 在body种插入形状
                document.body.appendChild(this.dom)
            }
        }

         // 创建圆
        var Cricle = function (x, y) {
            this.dom = document.createElement('div');
            this.dom.style.left = x + 'px';
            this.dom.style.top = y + 'px';
            this.dom.className = 'cricle';
        }
        Cricle.prototype = {
            init: function () {
                document.body.appendChild(this.dom)
            }
        }

         // 创建三角形
        var ArrowTop = function (x, y) {
            this.dom = document.createElement('div');
            this.dom.style.left = x + 'px';
            this.dom.style.top = y + 'px';
            this.dom.className = 'arrow-top';
        }
        ArrowTop.prototype = {
            init: function () {
                console.log(1);
                document.body.appendChild(this.dom);
            }
        }

         // 测试用例
         // new ArrowTop(x, y).init();
         //工厂方法
        function Fectory(type) {

            type = type || 'cricle';

            var x = Math.random() * w;
            var y = Math.random() * h;

            var fectory = null;

            switch (type) {

            case 'rect':

                fectory = new Rect(x, y);

                break;

            case 'cricle':

                fectory = new Cricle(x, y);

                break;

            case 'arrowTop':

                fectory = new ArrowTop(x, y);

                break;

            }

            if (fectory)
                fectory.init();

        }

        Fectory();
        Fectory('arrowTop');
    </script>
</body>
```

## 原型模式
通过将对象的原型指向类的原型实现对该对象原型的属性与方法的共享。


```
var Rect = function( x,y ){
    
    //继承
    Base.apply(this,arguments);
    this.dom.className = 'rect';

}

Rect.prototype = new Base();
```
原型模式一种创建型设计模式
它基于javascript原型链原理实现的
是一种组合式继承
对于处理复杂的类，通过提取公共部分实现对类优化

## 单例模式
### 简单单体模式
只能创建一个实例， 把所有代码和数据组织到一个地方. 全局的资源，公共的数据， 组织到一起.统一的管理.
单体模式 一般不采用new 关键字. 已经存在了对象.
主要用来 划分命名空间(区分代码，降低耦合性)


```
var Singleton={
    attr1: true,
    attr2: 10,
    method1: function(){
        alert(this.attr1);
    },
    method2: function(){
        alert('方法2');
    }
}
```
### 惰性单例
借用闭包创建单体 闭包主要的目的:就是保护数据，不受外界所干扰.
通过闭包将我们的single封装起来，避免外部访问而实例化，这样可保证实例化一次，闭包的返回函数的作用域在闭包里面，所以可以访问到single类对其实例化，这样在执行LazySingle才会对single类实例化

特点：

- 只能被实例化一次
- 推迟了实例化时间


```
var _interface = null;

var LazySingle = (function () {
    
    var Signle = function () {
        // do     
    }
    
    Signle.prototype = {
        version: 2,
        sayHello: function () {
            console.log('1');
        }
    }
    
    return function () {
        
        if ( !_interface ) {
            
            _interface = new Signle();
            
        }
        
        return _interface;
        
    }
    
})()


var s = LazySingle();

s.sayHello();
```
### 分支单体
判读程序的分支 --> 浏览器差异检测

```
var Slington = (function(){
    
    var def = true;

    var More = function(){

        var objA = { //火狐浏览器  内部的一些配置
            attr1: 'ff attr1',
            //属性1
            //方法1
        }
        var objB = {  //ie浏览器 内部的一些配置
            attr2: 'ie attr1',
            //属性1
            //方法1
        }

    }
    return (def) ? new More().objA : new More().objB;

})();
```
## 适配器模式
将一个类（对象）的接口（属性或者方法）转化成另一个类（对象）的接口，以满足用户的需要，使类（对象）之间接口的不兼容问题得以解决。

对被适配的数据的一个分解再封装的一个过程。
这个过程中会造成一定的开销。但远比更改原有业务逻辑成本低。

### 请求数据适配
需要dataAdaptor函数中有适配映射表


```
function DealData ( arr ) {
                
    var div = document.createElement('div');
    var img = new Image();
    var a = document.createElement('a');
    var p = document.createElement('p');
    
    img.src = arr[1];
    a.href = arr[0];
    
    p.innerHTML = arr[2];
    a.appendChild(p);
    a.appendChild(img);
    div.appendChild(a);
    
    document.body.appendChild(div);
    
}


//适配器
function DataAdaptor ( data ) {
    
    var arr = [];
    
    //适配映射表
    var map = {
        'src': 1,
        'href': 0,
        'title': 2
    }
    
    for ( var i in map ) {
        
        //把第三方数据源中的数据，放到自己数据中的数组中的相应位置
        //map[i] -> 自己数据源中的位置
        //i -> 第三方数据源中的属性
        
        arr[map[i]] = data[i];
        
    }
    
    //匹配出来后的数据
//                arr[0] = data.href
//                arr[1] = data.src
//                arr[2] = data.title
    
    return arr;
    
}


//第三方提供数据
//           obj.href  : 链接地址    
//                obj.src   : 图片地址
//                obj.title : 图片标题

$.get('xxx.json',function ( oD ) {
    
    if ( oD && oD.errno === 0 ) {
        
        var newRes = new DataAdaptor( oD.data ); 
        
        DealData(newRes);
        
    }
    
})
```
### 参数适配
使用继承
使用 || 运算符


```
function extend ( targetObj,obj ) {
                
    for ( var i in obj ) {
        
        targetObj[i] = obj[i];
        
    }
    
    return targetObj;
    
}

var Button = function ( param ) {
    
    var btn = document.createElement('button');
    
    //默认参数
    var def = {
        'background': 'tan',
        'color': 'yellow',
        'fontSize': '12px',
        'text': '按钮'
    }
    
    // 适配用户传递的参数与默认的参数
    var def1 = extend(def,param);
    
    for ( var i in def1 ) {
        
        if ( i === 'text' ) {
            
            btn.innerHTML = def1[i];
            
        } else {
            
            btn.style[i] = def1[i];
            
        }
        
    }
    
    document.body.appendChild(btn);
    
}

Button({'background': 'deeppink'});
```
## 组合模式
部分整体模式，将对象表示成树形结构，表示部分整体关系。所以部分与整体的行为习惯达到一致型。

组合模式中只有两种类型对象，组合对象，叶子对象

创建类的步骤：

- 构造函数
- 构造函数继承
- 保留参数
- 初始化数据
- 原型式继承，继承基类方法
- 重写init方法
- 添加其他方法

它是将整体分解成为一个个部分，再有部分重新拼装成一个整体。部分分解的也多，整合结果也就越多。
它的部分与整体之间具有行为的一致性。
部分拼装成整体的过程具有多样性

## 观察者模式
叫消息系统，消息机制，或者发布订阅额模式。通过消息系统实现对象或者类之间的解耦。
解决是一种依赖关系，
解决了，主体与观察者之间的一种依赖关系。
被观察者对象或者类也可以是观察者，观察者也可以是被观察者
观察者内部变化不会影响到被观察者，反过来一样


```
var Observer = (function () {

var __msg = {};

return {
    
    //添加订阅者 
    //@param {String} type  订阅者名字
    //@param {Function} fn  执行函数  
    
    add: function ( type,fn ) {
        
        if ( __msg[type] ) {
            
            __msg[type].push(fn);
            
        }    else {
            
            __msg[type] = [fn];
            
        }
        
        return this;
        
    },

     //执行回调函数
     //@param {String} type
    fire: function ( type,data ) {
        
        if ( __msg[type] ) {
            
            var e = data
            
            for ( var i=0; i<__msg[type].length; ++i ) {
                
                __msg[type][i](e);
                
            }
            
        }
        
        return this;
        
    },
     // 移除回调函数
     // @param {String} type
     // @param {String} fnName
    remove: function ( type,fnName ) {
        
        if ( __msg[type] ) {
            
            for ( var i = __msg[type].length-1; i>=0; --i ) {
                
                if ( __msg[type][i] === fnName ) {
                    
                    __msg[type].splice(i,1);
                    
                }
                
            }
            
        }
        
        return this;
    }
    
}

})()
```
## 策略模式
封装一组算法，使其可以互相替换，这组算法本身具有独立性，不受客户端影响。

特点：

它是行为型模式
每种都是独立的，所他们之间可以相互替换
他解决是使用者与策略算法之间的耦合
算是是独立的方便我们进行单测
算法在使用时候的过程是不一样的，但结果是一样的
tween中使用的是策略模式，价格算法，表单验证.


```
<body>

    <p>价格：<span id="num1"></span>
    </p>
    <p>淘宝价：<span id="num2"></span>
    </p>

    <script type="text/javascript">
        //处理各种价格

        var price = 500;

        var PriceStrategy = (function () {

            var rate = {
                RMBTODOLLAR: 0.1525,
                DOLLARTORMB: 6.5590
            }

            var strategy = {

                'return100': function (price) {

                        var num = parseInt(price / 100) * 10;

                        return price - num;

                    },

                    'percent50': function (price) {
                        return price * .5;
                    },

                    'RMBToDollar': function (price) {
                        return price * rate.RMBTODOLLAR;
                    }

            }

            return function (price, type) {

                if (strategy[type]) {

                    return strategy[type](price);

                }

                return price;
            }

        })();

        document.getElementById('num1').innerHTML = '￥：' + 　price + '  $：' + PriceStrategy(price, 'RMBToDollar');
        document.getElementById('num2').innerHTML = '￥：' + PriceStrategy(price, 'percent50');
    </script>

</body>
//表单验证
var InputStrategy = (function () {

    //策略模型
    var strategy = {

        //判断是否为空
        'notNull': function (val) {

                return /^\s*$/g.test(val) ? '输入的内容不能为空' : 　'';

            },

            //判断是否是数字
            'isNumber': function (val) {

                return /^-?[\d]+(\.[\d])?$/.test(val) ? '' : '输入的不是一个正确的数字';

            },

            //判断电话格式  010-12345678 1234-1234567
            'isTelephoneNumber': function (val) {

                return /^[\d]{3}\-[\d]{8}$|^[\d]{4}\-[\d]{7}$ /.test(val) ? '' : '请输入一个正确的电话号码';

            }

    }

    return {
        //检测表单输入的文本内容是否正确
        //@param {String} val 检测文本
        //@param {String} type 检测算法

        check: function (val, type) {

            if (strategy[type]) {

                return strategy[type](val);

            } else {

                return '没有该算法';

            }

        }
    }

})()


//策略名称 与 页面中的表单的 映射表
var arr = ['notNull', 'isNumber', 'isTelephoneNumber'];

for (var i = 1; i < 4; i++) {

    checkInp('inp' + i, 'err' + i, arr[i - 1]);

}

//执行 检测表单
function checkInp(inpId, errId, type) {

    //添加监听
    document.getElementById(inpId).onblur = function (ev) {

        var val = ev.target.value;

        var reslut = InputStrategy.check(val, type);

        if (reslut) {

            document.getElementById(errId).innerHTML = reslut;

        } else {

            document.getElementById(errId).innerHTML = '';

        }

    }

}
```

## 命令模式
将请求与实现解耦并封装，实现请求对客户端实现参数化。

### 命令模式渲染视图
页面中有好多数据源，要对这些数据源渲染页面，传统方式，渲染页面时候，将视图，数据，业务耦合在一起了，不利于开发，所以要将这些实现东西提取出来封装成一个个命令供使用，来实现代码复用，并简化的创建操作


```
<body>
        
<div class="title" id="title"></div>

<div id="container" class="product"></div>

<script type="text/javascript">
    
    //命令模式:
    // ViewCommand ->cmd & view   return  excute
    
    var titleData = {
        'title': '夏日里的一片温馨',
        'tips': '暖暖的温情带给人们家的感受。'
    }
    
    var productData = [{
        'src': 'img/02.jpg',
        'txt': '绽放的桃花'
    },{
        'src': 'img/03.jpg',
        'txt': '阳光下的温馨'                        
    },{
        'src': 'img/04.jpg',
        'txt': '镜头前的绿色'
    }];
    
    
    var ViewCommand = (function () {
        
        var view = {
            'title': '<div><h2>{#title#}</h2><p>{#tips#}</p></div>',
            'item': '<div><img src="{#src#}" alt="" /><p>{#text#}</p></div>'
        }
        
        var formatString = function ( tpl,data ) {
            
            
            var reslut = tpl.replace(/\{\#([\w]+)\#\}/g,function ( match,key ) {
                
                return data[key] === undefined ? '' : data[key];
                    
            });
            
            return reslut;
        
        }
        
        var html = ''; //缓存格式化后的字符串
        
        var cmd = {
            
            // 对模板格式化
            // tpl 模板
            //data 数据

            'create': function ( tpl,data ) {
                
                var cTpl = view[tpl];
                
                if ( data instanceof Array ) {
                    
                    for ( var i=0; i<data.length; i++ ) {
                        
                        html += formatString(cTpl,data[i]);
                        
                    }
                    
                } else {
                    
                    html += formatString(cTpl,data);
                    
                }
                
            },
            
            //渲染html字符串在页面中
            //id 渲染字符串容器id
            //tpl 模板
            //data 数据

            'display': function ( id,tpl,data ) {
                
                if ( tpl && data ) {
                    
                    this.create( tpl,data );
                    
                }
                
                //获取容器
                var dom = document.getElementById(id);
                dom.innerHTML = html;
                
                html = '';
                
            }
            
        };
        
        return {
            
            // 命令执行方法
            // obj.command 命令名称
            // obj.param   命名参数
            'excute': function ( obj ) {
                
                var command = cmd[obj.command];
                
                //判断是否是 数组
                obj.param = obj.param instanceof Array ? obj.param : [obj.param];
                
                command.apply(cmd,obj.param);
                
                return this;
                
            }
            
        }
        
    })();
    
    
    ViewCommand
        .excute({
            'command': 'display',
            'param': ['title','title',titleData]
        })
        .excute({
            command: 'create',
            param: ['item', {
                src : 'img/01.jpg',
                text : '迎着朝阳的野菊花'
            }]
        })
        .excute({
            command: 'display',
            param: ['container', 'item', productData]
        })
    
</script>


</body>
```
命令模式解决了命令的发起者与命令的实现者之间的耦合
命令的发起者(调用命令的时候) 不必去了解命令是如何实现的以及命令是如何运行的。
所有的在使用具有一致性。
命令模式在一定程度上简化了操作。

### canvas绘图
canvas绘图的 上下文 ctx 一些问题

模块外界访问不到 ctx
ctx很重要，不论操作都需要使用这个变量
一旦更改这个变量，会造成很严重的后果，造成原有功能失效。
把ctx封装成内部方法。 封装成 cmd 对象下的方法， 在excute 命令执行方法中匹配 cmd 中的方法


```
<body>
<canvas id="canvas" width="800" height="500" style="border: 1px solid magenta;"></canvas>

<script type="text/javascript">

var ViewCommand = (function () {
    
    var canvas = document.getElementById("canvas");
    var cvs = canvas.getContext('2d');
    
    var w = canvas.width;
    var h = canvas.height;
    
    var cmd = {
        
        beginPath: function () {
            
            cvs.beginPath();
            
        },
        
        colsePath: function () {
            
            cvs.colsePath;
            
        },
        
        arc: function ( x,y,r ) {
            
            cvs.arc(x,y,r,0,2*Math.PI,false);
            
        },
        
        fill: function () {
            
            cvs.fill();
            
        },
        
        fillStyle: function ( c ) {
            
            cvs.fillStyle = c;
            
        },
        
        stock: function () {
            
            cvs.stroke();
            
        }
        
    }
    
    return {
        
        //命令执行
        excute: function ( arr ) {
            
            cmd[arr[0]].apply(cmd,arr.slice(1));
            
            return this;
            
        },
        
        //扩展
        excuteArr: function ( arrTo ) {
            
            for ( var i=0; i<arrTo.length; i++ ) {
                
                this.excute( arrTo[i] );
                
            }
            
            
        }
        
    }
    
})()

ViewCommand
    .excute(['beginPath'])
    .excute(['arc',100,100,20])
    .excute(['colsePath'])
    .excute(['fillStyle','pink'])
    .excute(['fill'])
    .excuteArr([
        ['beginPath'],
        ['arc',300,300,20],
        ['colsePath'],
        ['fillStyle','yellow'],
        ['stock']
    ]);
    
</script>

</body>
```
## 迭代器模式
### 数组迭代器

```
// 遍历
// arr 遍历的函数
// cb
function each ( arr,cb ) {
    
    for ( var i=0,len=arr.length; i<len; ++i ) {
        
        cb && cb.call(arr,arr[i],i,arr);
        
    }
    
}
```
### 对象迭代器

```
//obj 要被遍历的对象
//cb 遍历回调函数
function each( obj,cb ){
    for( var i in obj ){
        cb && cb.call(obj,obj[i],i);
    }
}
function each (obj, fn) {
    // obj是一个数组时候
    if (obj instanceof Array) {  //判断是否是数组
        //迭代数组
        for (var i = 0, len = obj.length; i < len; i++) {
            fn(obj[i], i, obj)
        }
    // obj是一个对象
    } else { 
        // 迭代对象
        for (var i in obj) {
            fn(obj[i], i, obj)
        }
    }
}
```
### 面向对象中的迭代器
面向对象中有时需要遍历实例化对象中的自身属性，过滤掉原型上的属性，可以通过 hasOwnPrototy 来实现

```
// 遍历实例化对象自身的属性，过滤掉原型上的属性
// obj    实例化对象
// cb     回调函数

function  each ( obj,cb ) {
    for ( var i in obj ) {
        if ( obj.hasOwnPrototy(i) ) {
            cb && cb.call(obj,obj[i],i);
        }
    }
}
```
迭代器的特点

本身并没有删除循环语句，只是将循环语句转移到迭代器的内部，
迭代器模式中，数据对象内部的结构，只需要关注处理函数。
移出了循环语句，使得代码看着更清晰
迭代器模式是将数据源于处理函数之间的解耦
### DOM迭代器

```
<body>
        
    <ul id="list">
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>4</li>
        <li>5</li>
        <li>6</li>
        <li>7</li>
        <li>8</li>
        <li>9</li>
        <li>10</li>
    </ul>
    
    <script type="text/javascript">
        
        var Iter = function ( id,tagName ) {
            
            this._id = document.getElementById(id);
            
            this.items = this._id.getElementsByTagName(tagName);
            
            this.length = this.items.length;
            
            this.index = 0;
            
        }
        
        Iter.prototype = {
            
            'getCurrent': function () {
                
                return this.items[this.index];
                
            },
            
            'first': function () {
                
                this.index = 0;
                
                return this.getCurrent();
                
            },
            
            'last': function () {
                
                this.index = this.length - 1;
                
                return this.getCurrent();
                
            },
            
            'prve': function () {
                
                if ( this.index > 0 ) {
                    
                    this.index --;
                    
                    return this.getCurrent();
                    
                } else {
                    
                    return this.first();
                    
                }
                
                
            },
            
            'next': function () {
                
                if ( this.index < this.length - 1 ) {
                    
                    this.index = this.length + 1;
                    
                    return this.getCurrent();
                     
                } else {
                    
                    return this.last();
                    
                }
                
            },
            
            'get': function ( idx ) {
                
                var num = ( (idx % this.length) + this.length ) % this.length;
                
                return this.items[num];
                
            },
            
            'set': function ( idx ) {
                
                var num = ( (idx % this.length) + this.length ) % this.length;
                
                this.index = num;
                
                return this.items();
                
            },
             //处理某个元素
             // idx 处理的第 i 个元素
             //cb 处理该函数的回调函数
            'dealItem': function ( idx,cb ) {
                
                var dom = this.get(idx);
                
                cb.call(dom,idx,dom);
                
                return this;
                
            },
        
             //处理所有元素
            'each': function ( cb ) {
                
                for ( var i=0; i<this.length; i++ ) {
                    
                    cb.call(this.items[i],i,this.items[i]);
                    
                }
                
                return this;
                
            },
             //过滤某些元素并执行
            'fillter': function ( arr,arrFn,allFn ) {
                
            }
            
            
        }
        
        var list = new Iter('list','li');
        
        list.dealItem(2,function ( i,val ) {
            
            val.style.background = 'pink';
            
        }).each(function ( i,val ) {
            
            val.style.color = 'tan';
            
        });
        
//            list.next().style.background = 'pink';
        
    </script>
    
</body>
```
## 委托模式
是将多个对象接收并处理的请求，统一委托给另外一个对象处理

### 委托模式解决事件数量问题
解决事件绑定数量的问题，为一堆元素绑定事件，通过for循环遍历绑定，无形中绑定了n个事件，这样在内存中造成一定的开销。
可以通过这些元素公共的父元素，对其绑定事件，通过e.target事件触发的目标元素的某些属性来确认是哪些元素需要绑定事件来解决上面的问题。
通常可以判断元素的名称，类名，id，及其属性。
通过对子元素的某些特性判断，来实现对该元素的事件的绑定。


```
var oUl = document.getElementsByTagName('ul')[0];
var li = oUl.getElementsByTagName('li');


oUl.addEventListener('click',function ( e ) {

//                console.log( e );

    if ( e.target.tagName.toLocaleLowerCase() == 'li' ) {
        
        e.target.style.background = 'pink';
        
    }


},false);
```
### 未来元素事件绑定问题
传统式方式通过for循环，只能对现用元素进行事件绑定，当需要在后面添加新的元素的时候，该方案不能对新元素绑定事件。
委托模式，通过将子元素的事件绑定委托给父元素，事件对子元素事件绑定的效果，这样，在父元素中添加新的子元素，同样可以获取绑定事件的效果


```
var oUl = document.getElementsByTagName('ul')[0];
var li = oUl.getElementsByTagName('li');


oUl.addEventListener('click',function ( e ) {

//                console.log( e );

    if ( e.target.tagName.toLocaleLowerCase() == 'li' ) {
        
        e.target.style.background = 'pink';
        
    }
    

},false);

var oLi = document.createElement('li');

oLi.innerHTML = '11';

oUl.appendChild(oLi);
```
### jQuery事件委托
jQuery中给我提供了一个专用来是做事件委托事件绑定方法，delegate，它实质上是通过on方法实现的

```
<script type="text/javascript">

$('ul').delegate('.a','click',function () {
    
    $(this).css('background','tan');
    
});

</script>
```
### 内存外泄
在低版本IE浏览器中，内存只会清理哪些没有被javascript引用的dom元素，所以在对元素定义事件时候，如果将元素清理一定要将该元素绑定的事件解除，因此要在事件内部显性清除事件绑定，但这就要写在原有事件内部，

```
var oDiv = document.getElementsByTagName('div')[0];
            
var oBtn = document.getElementsByTagName('button')[0];

oBtn.onclick = function () {
    
    oBtn.onclick = null;  //手动清除事件绑定
    oDiv.innerHTML = 'info';
    
}
```
更好的解决方案是对该元素的父元素委托绑定事件，这样当清理该元素时候，由于没有事件的绑定，该元素即会被清理

```
var oDiv = document.getElementsByTagName('div')[0];
            
var oBtn = document.getElementsByTagName('button')[0];

oDiv.onclick = function ( ev ) {
    
    if ( ev.target.tagName.toLocaleLowerCase() === 'button' ) {
        
        oDiv.innerHTML = 'info';
        
    }
    
}    
```
### 数据分发
动态页面中，页面中的每个模块会对应一个数据请求，然而如果页面中的这类模块很多，就要发送多个请求，但是并发请求的个数是有限的 ，因此后面的请求就会被堵塞，为了解决这类问题，可以将这些请求委托给父请求统一处理，当接收数据后，解析数据，并派发给各个子模块中，供其使用

```
//对每个模块进行封装
var DealStrategy = {
    
    'banner': function ( data ) {
        
        $('.banner').html(data);
        
    },
    
    'article': function ( data ) {
        
        $('.article').html(data);
        
    },
    
    'aside': function ( data ) {
        
        $('.aside').html(data);
        
    }
    
}

function Deal ( data ) {
    
    for ( var i in data ) {
        
        DealStrategy[i](data[i]);
        
    }
    
}

//发出 get 请求
$.get('data/all.json',function ( res ) {
    
    if ( res.errno === 0 ) {
        
        Deal( res.data );
        
    }
    
});
```
## 节流模式
对重复的业务逻辑进行节流控制，执行最后一次操作并取消其他操作，以提高性能。

特点：

通过计时器延迟程序的执行
通过计时器，使程序异步执行，避免开销大的程序造成的堵塞
条件：

程序可控：即取消后是否可以继续执行
异步执行：即程序是否可以异步执行

### 节流器
通过jQuery的stop方法禁止动画的排队，但是对于滚动这种高频事件，每次执行都会添加动画
通过节流模式，短时间内触发多次动画时，前面动画被取消添加，这样只执行最后一次，来提高性能

节流器通常提供两种使用方式
触发操作
清除操作

```
//对重复的业务逻辑，通常执行最后一个，取消前面的业务逻辑，来实现业务逻辑的优化。
//高频事件， mousemove ，window.onscorll
var Throttle = function () {


    var isClear = arguments[0] , fn;

    if ( isClear !== 'true' ) {

     // 触发操作
     // @param[0] [Function]  表示函数的名称
     // @param[1] [Object] 配置项

        //触发操作
        fn = isClear;
        var o = arguments[1] || {};

        var obj = {
            time: o.time || 200,
            context: o.context || null,
            data: o.data || null
        };  //配置项
        
        //设表先关
        fn._throttle && clearTimeout(fn._throttle);
        
        fn._throttle = setTimeout(function(){

            fn.call(obj.context,obj.data);    

        },obj.time);

    } else {

        // 清除操作
        // @param[0] [Boolean] 表示是否取消操作. true 取消
        // @param[1] [Function] 取消的函数名称

        //取消操作
        fn = arguments[1];

        //清除定时器
        fn._throttle && clearTimeout(fn._throttle);

    }

}

function goBack(){
    console.log(1);
}

//触发
Throttle(goBack,{time: 500,data: {msg: 'hello'}});

//清除操作
// Throttle(true,goBack);
```

### 统计节流
统计是什么，为什么要做统计？ 
统计是为了了解用户对页面使用习惯或者使用方式的，要为页面添加统计来帮助分析用户的使用行为

统计的实现
当用户触发一次交互的时候，想向服务器端发送一条信息，将其记录下来，保存在服务器中。
前端的一些交互，有时候是不需要向服务器端发送请求的，此时，服务器端是不知道这些行为的，所以要向服务器端发送请求告知

统计请求方面的考虑
post请求要比get请求发送的时处理事情要多，在做统计的时候就不考虑post请求
Ajax可以发送get请求，但是要写好多逻辑代码
请求文档是一个get请求，但是请求过来的页面中的信息量比较多
script标签也是get请求，
link标签也是一个get请求
img标签也是get请求
相比较这几种，img的发送成本更低一些，在发送统计请求的时候，用img(0字节的图片作为中转发送get请求)的get请求

请求节流
页面中的一些高频事件，做统计的时候，会不停的发送统计，由于请求的并发次数是有限的 ，不能同时发送这么多的请求。这回造成后面资源的加载延迟，要对这些统计做节流处理

```
var img = new Image();
            
function sendLog ( val ) {
    
    var reslutStr = '';
    for ( var i in val ) {
        
        reslutStr += '&' + i + '=' +  val[i];
        
    }
    
    img.src = 'a.jpg?' + reslutStr;
    
}
```
统计的拼接

一条统计 ?Type=click&date=123
将两条统计拼接在一起 ?Type=click&dat2=123&type=mouseover&date=234，这样的化出现相同的字段了，不能这么拼接，
可以将一条统计作为一个值： ?Log1=typeclick|date123 //将一条数据，作为请求的 一个 id.
拼接两个统计的时候，就可以?Log1=typeclick|date123&log2=typemouserover|date234

//节流处理： 当发送第一个请求的时候，将其缓存下来，当其触发的次数达到规定的次数再发送
//想缓存，需要有缓存容器， dataCache   次数，需要有次数的规定。maxNum
//将一条数据，作为请求的  一个  id.

```
var LogPack = (function () {
    
    var dataCache = [];  //缓存容器
    
    var maxNum = 10; //缓存的次数                
    
    var oImg = new Image();  //请求的触发器

    var itemSplit = '|';
    var keyValSplit = '*';
    
    //发送统计
    function sendLog () {
        
        var logs = dataCache.splice(0,10);
        
        var str = '';
        
        for ( var i=0; i<logs.length; i++ ) {
            
            //拼接 id
//                        'Log1=type*click|date*123&log2=type*mouserover|date*234';
            
            str += '&log' + i + '=';
            
            for ( var j in logs[i] ) {
                
                str += j + keyValSplit + logs[i][j] + itemSplit;
                
            }
            
            str = str.replace(/\|$/,'');
            console.log( str );
        }
        
        oImg.src = 'a.jpg?len=' + logs.length + str;
        
    }

    return function( val ){
        
//                    当没有val 数据,主动触发
            if ( !val ) {
                sendLog();
            }
        
        //统计信息缓存下来
        dataCache.push( val );
        
        //缓存数据条目  大于 定义的条目
        if ( dataCache.length > maxNum ) {
            
            //发送
            sendLog();
            
        }
        
    };
    
})()

//触发
var oBtn = document.getElementById("btn");

oBtn.onmousemove = function () {
    
    LogPack({
        type: 'onmousemove',
        date: new Date().getTime()
    });
    
}
//            
oBtn.onmouseover = function () {
    
    LogPack({
        type: 'onmouseover',
        date: new Date().getTime()
    });
    
}
//发送结果
```


## MVC
### 后端的MVC概念
Model（模型）表示应用程序核心（比如数据库记录列表）。
是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。
View（视图）显示数据（数据库记录）。
是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的
Controller（控制器）处理输入（写入数据库记录）
是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。
### 前端中的实践
Model，页面数据的存储操作
View，渲染出可视（给人看）的页面操作
Controller，页面的交互对视图的更改以及对数据的更改



View层通过Model渲染数据，所以View层可以访问Model层
Controller层可以对Model写入读取数据，所以Controller可以访问Model层
Controller层可以对View层打开弹层，浮层等交互，所以Controller可以访问View层

在前端的框架中，很多框架是基于MVC模式实现，比如BackBone
它将MVC进行了一些改造，比如它允许Model层可以访问View层，实现数据层的更新通知View层视图的渲染

通常一个模块对应一个控制器，一个模型，一个视图，那么如果将页面所有模块中的视图，控制器模型放在一起，逻辑比较混乱，为了管理方便，将一个模块的控制器，模型，视图，放在一个文件内管理，根据他们的父模块不同，来进行分别建立。（父模块作为建立文件夹的标准）

```
var MVC = {}

//模型模块
// get 得到模型
// add 添加模型
MVC.Model = (function () { 
    
    //用来存储数据层面的数据
    var M = {};
    
    return {
         //读取数据的方法
         //@param {String} strName 读取数据的名称
         //eg: get('a.b.c') => M.a.b.c
        'get': function ( strName ) {
            
            var path = strName.split('.');
            
            var reslut = M;
            
            for ( var i=0; i<path.length; i++ ) {
                
                reslut = reslut[path[i]];
                
                //后置判定   undefined || 属性值不存在  
                if ( undefined === reslut ) {
                    
                    return null;
                    
                }
                
            }
            
            return reslut;    
                
        },
        
         //为模型添加数据
         //@param {String} strName  属性层级的字符串
         //@param {mixed} val 添加的数据值
         //eg:  add('a.b.c',111);
         //     add('a.b',{c: 222})     
        'add': function ( strName,val ) {
            
            var path = strName.split('.');
            
            var reslut = M;
            
            for ( var i=0; i<path.length-1; i++ ) {
                
                if ( reslut[path[i]] !== undefined && typeof reslut[path[i]] !== 'object' ) {
                    
                    throw new Error('非对象类不可以添加属性');
                    
                }
                
//                            reslut 没有属性, 就添加属性
                if ( reslut[path[i]] === undefined ) {
                    
                    reslut[path[i]] = {};
                    
                }
                
                //缓存
                reslut = reslut[path[i]];
                
            }
            
            //最后一步单独处理. 作为赋值。 是已知的值，所以要特殊处理。 前面for循环 遍历 length-1 项
            reslut[path[i]] = val;
            
            return this;
            
        }
        
    }
    
})()


//视图模块
//create 创建模型
//add 添加模型

MVC.View = (function () {
    
    var V = {}
    
    return {
         //创建视图
         //@param {String} id 表示视图的模块
        create: function ( id ) {
            
            var view = V[id];
            
            return view.call(MVC,MVC.Model,MVC.template);
            
        },
         //添加视图创建方法的
         //@param {String} id 创建方法的名称
         //@param {Function} method 创建方法的实现
        add: function ( id,method ) {
            
            V[id] = method;
            
            return this;
            
        }
        
    }
    
})()



//控制器模块
//init : 初始化所有控制器
//add : 添加控制器
MVC.Controller = (function () {
    
    var C = {};
    
    return {
        
        //初始化所有控制器
        init: function () {
            
            for ( var i in C ) {
                
                C[i].call(MVC,MVC.Model,MVC.View);
                
            }
            
        },
    
         //添加控制器操作 
        add: function ( id,method ) {
            
            C[id] = method;
            
            return this;
            
        }
        
    }
    
})()


//入口 函数
MVC.install = function () {
    
    window.onload = function () {
        
        MVC.Controller.init();
        
    }
    
}
 //模板字符串
MVC.template = function (tpl, data) {
    return tpl.replace(/\{#([\w\.]+)#\}/g, function (match, key) {
        var path = key.split('.');
        // 缓存参数中的data
        var result = data;
        for (var i = 0; i < path.length; i++) {
            // 缓存每一层级的数据
            result = result[path[i]];
            // 如果该数据不存在
            if (result === undefined) {
                return '';
            }
        }
        return result;
    })
}
```