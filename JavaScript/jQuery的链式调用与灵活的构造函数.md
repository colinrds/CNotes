一、我们从一个简单的构造函数+原型程序开始

```
var G = function(){};
G.prototype = {
    length : 5,
    size : function(){
        return this.length;
    }
}
```

上例是个非常简单的程序，如果需要调用，我们可以用new的方式

```
var oG = new G();
console.log( oG.size() ); //5
```

- 常见的错误调用方式1

```
console.log( G.size() ); //报错
```

G.size这种调用，是把size当做静态方法调用，如果需要正常的调用， 应该把size方法加在函数本身，如:

```
G.size = function(){
    return 10;
}
```

- 常见的错误调用方式2

```
G().size() //报错
```

G()返回的是undefined, undefined.size() 肯定是报错， size是函数G原型对象上的方法，如果要确保这种方式调用正常，那么我们就要让G() 返回一个G函数的实例
所以，我们可以这样做：

```
var G = function () {
    if( this instanceof G ) {
        return this;
    }else {
        return new G();
    }
};
```

把G的构造函数改造一下，判断this是否是当前G函数的实例，如果是,直接返回，如果不是，返回new G()  这样根据原型对象的查找原则，就能确保调用到size方法

完整的代码:

```
var G = function () {
    if( this instanceof G ) {
        return this;
    }else {
        return new G();
    }
};
G.prototype = {
    length: 5,
    size: function () {
        return this.length;
    }
}
console.log( G().size() );
```

在jquery框架中，他是怎么做的？

```
var G = function () {
    return G.fn;
};
G.fn = G.prototype = {
    length: 5,
    size: function () {
        return this.length;
    }
}
console.log( G.prototype.size() ); //5
console.log( G().size() ); //5
```

在jquery中, 为函数G增加一个属性fn( 记住：在js中函数是对象，这其实就是给对象增加了一个属性 )，然后在G函数中返回 G.fn 就是就是返回G.prototype

那么用G().size 其实就是相当于调用 G.prototype.size()

二、在jquery中，这个构造函数一般是用来选择元素的。

G返回的是G的原型对象，我们要增加选择元素的功能，自然而然，就是往原型对象上添加：

```
var G = function ( id ) {
    return G.fn.init( id );
};
G.fn = G.prototype = {
    init : function( id ){
        return document.getElementById( id );
    },
    length: 5,
    size: function () {
        return this.length;
    }
}
```

向G的原型对象上添加一个init方法， 然后在构造函数中调用

```
window.onload = function(){
    console.log( G( 'box' ) );
    G('box').style.backgroundColor = 'red';
    // G('box').size(); //报错，无法链式调用
}

<div id="box">ghost wu tell you how to learn design pattern</div>
```

虽然通过init方法，能够选择到dom元素，但是不能实现链式调用, 因为G('box')返回的是一个dom对象， 而在dom对象上是没有size这个方法的，因为size是G.prototype上的

所以要实现链式调用，就要确保init方法返回的是G的实例或者G.prototype,  这个时候，this就可以派上用场了

```
<script>
var G = function (id) {
    return G.fn.init(id);
};
G.fn = G.prototype = {
    init: function (id) {
        this[0] = document.getElementById(id);
        this.length = 1;
        return this;
    },
    length: 0,
    size: function () {
        return this.length;
    }
}
window.onload = function () {
    console.log(G('box'));
    console.log(G('box2').size());
}
</script>

<div id="box">ghost wu tell you how to learn design pattern</div>
<div id="box2">id为box2的第二个div</div>
```

把选择到的元素放在this中， 这个时候的this指向的是G.fn,G.prototype?

因为在构造函数中，是这样调用的: G.fn.init( id ), 我把G.fn标成红色, 也就是相当于G.fn是一个对象，没错他确实就是一个对象G.prototype，所以在init中的this指向的就是

init方法前面的对象( G.fn, G.prototype ).

三、this覆盖

接下来，就会产生一个问题， this共用之后，元素选择就会产生覆盖

```
<script>
var G = function (id) {
    return G.fn.init(id);
};
G.fn = G.prototype = {
    init: function (id) {
        this[0] = document.getElementById(id);
        this.length = 1;
        console.log( this === G.fn, this === G.prototype, this );
        return this;
    },
    length: 0,
    size: function () {
        return this.length;
    }
}

window.onload = function(){
    console.log( G( 'box' ) );
    console.log( G( 'box2' ) );
}
</script>

<div id="box">ghost wu tell you how to learn design pattern</div>
<div id="box2">id为box2的第二个div</div>
```

调用两次构造函数G 去获取元素的时候， this[0] 现在都指向了 id为box2的元素， 把第一次G('box')选择到的id为box的元素覆盖了，产生覆盖的原因是this共用，那么我们

可以通过什么方法把this分开呢？不同的this指向不同的实例？ 用什么？ 恩，对了， 用new，每次new一个构造函数就会生成新的实例

四、解决this覆盖与链式调用

```
<script>
    var G = function (id) {
        return new G.fn.init(id);
    };
    G.fn = G.prototype = {
        init: function (id) {
            this[0] = document.getElementById(id);
            this.length = 1;
            return this;
        },
        length: 0,
        size: function () {
            return this.length;
        }
    }
    window.onload = function () {
        console.log(G('box'));
        console.log(G('box2'));
    }
</script>
<div id="box">ghost wu tell you how to learn design pattern</div>
<div id="box2">id为box2的第二个div</div>
```

通过构造函数中new G.fn.init( id ) 的方式，每次生成一个新的实例，但是产生了一个新的问题，不能链式调用， 因为init中的this发生了改变，不再指向( G.fn, G.prototype ).

```
G('box2').size();//报错,
```

为了能够调用到size(), 所以在执行完构造函数之后，我们要确保this指向G的实例，或者G的原型对象，
只需要把init函数的原型对象指向G.fn就可以了

```
var G = function (id) {
    return new G.fn.init(id);
};
G.fn = G.prototype = {
    init: function (id) {
        this[0] = document.getElementById(id);
        this.length = 1;
        return this;
    },
    length: 0,
    size: function () {
        return this.length;
    }
}
G.fn.init.prototype = G.fn;
```

加上G.fn.init.prototype = G.fn;我们就修正了this的覆盖与链式调用问题

 五、扩展选择器

上面支持id选择器，我们只需要在init函数加上其他类型的扩展就可以了，比如，我这里增加了一个标签选择器

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        div,
        p {
            border: 1px solid red;
            margin: 10px;
            padding: 10px;
        }
    </style>
    <script>
        var G = function (selector, context) {
            return new G.fn.init(selector, context);
        };
        G.fn = G.prototype = {
            constructor: G,
            init: function (selector, context) {
                this.length = 0;
                context = context || document;
                if (selector.indexOf('#') == 0) {
                    this[0] = document.getElementById(selector.substring(1));
                    this.length = 1;
                } else {
                    var aNode = context.getElementsByTagName(selector);
                    for (var i = 0, len = aNode.length; i < len; i++) {
                        this[i] = aNode[i];
                    }
                    this.length = len;
                }
                this.selector = selector;
                this.context = context;
                return this;
            },
            length: 0,
            size: function () {
                return this.length;
            }
        }
        G.fn.init.prototype = G.fn;

        window.onload = function () {
            console.log(G('#box')[0]);
            var aP = G('p', G('#box')[0]);
            // var aP = G('p');
            // var aP = G('#p1');
            for (var i = 0, len = aP.size(); i < len; i++) {
                aP[i].style.backgroundColor = 'blue';
            }
        }
    </script>
</head>

<body>
    <div id="box">
        <p>跟着ghostwu学习设计模式</p>
        <p>跟着ghostwu学习设计模式</p>
        <p>跟着ghostwu学习设计模式</p>
        <p>跟着ghostwu学习设计模式</p>
    </div>
    <p id="p1">跟着ghostwu学习设计模式</p>
    <p>跟着ghostwu学习设计模式</p>
</body>

</html>
```
