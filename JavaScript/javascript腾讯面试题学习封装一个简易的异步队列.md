这道js的面试题，是这样的，页面上有一个按钮，一个ul，点击按钮的时候，每隔1秒钟向ul的后面追加一个li, 一共追加10个，li的内容从0开始计数( 0, 1, 2, ....9 )，首先我们用闭包封装一个创建li元素的函数.

```
var create = (function(){
    var count = 0;
    return function(){
        var oLi = document.createElement( "li" );
        oLi.innerHTML = count++;
        return oLi;
    }
})();
```

页面上的2个元素:

```
<input type="button" value="点我">
<ul id="box"></ul>
```

js代码：
```
var oBtn = document.querySelector( "input" );
var oBox = document.querySelector( "#box" );

var create = (function(){
    var count = 0;
    return function(){
        var oLi = document.createElement( "li" );
        oLi.innerHTML = count++;
        return oLi;
    }
})();

oBtn.onclick = function(){
    setTimeout(function(){
        oBox.appendChild( create() );
        setTimeout( function(){
            oBox.appendChild( create() );
            setTimeout( function(){
                oBox.appendChild( create() );
            }, 1000 );
        }, 1000 );
    }, 1000 );
}
```

点击按钮的时候，用回调函数嵌套方式，这里我加入3个li，就已经快受不了了，这就是javascript著名的回调地狱，那么在这里，我用循环简化一下：

```
var oBtn = document.querySelector("input");
var oBox = document.querySelector("#box");
var timer = oNode =  null;
var create = (function () {
    var count = 0;
    return function () {
        var oLi = document.createElement("li");
        oLi.innerHTML = count++;
        return oLi;
    }
})();
function add(){
    oNode = oBox.appendChild( create() );
    if ( oNode.innerHTML < 9 ) {
        timer = setTimeout( add, 1000 );
    }else {
        clearTimeout( timer );
    }
}
oBtn.onclick = function () {
    add();
}
```

恩，确实简化了，但是这种面向过程的方式，耦合性太强，下面呢，我就把这个封装成一个通用队列

第一步：封装一个队列，包含（ 入列，出列），队列的特点（先进先出，如果你不懂这个，需要去补下基本的数据结构与算法内容）

```
var Queue = function () {
    this.list = []
}
Queue.prototype = {
    constructor: Queue,
    enQueue: function ( fn ) {
        this.list.push( fn );
        return this;
    },
    deQueue: function () {
        var fn = this.list.shift() || function () {};
        fn.apply( this, arguments );
    }
}
```

我们来使用它：

```
var oQ = new Queue();
oQ.enQueue( function(){
    console.log( 'ghostwu1' );
}).enQueue( function(){
    console.log( 'ghostwu2' );
}).enQueue( function(){
    console.log( 'ghostwu3' );
}).deQueue();
while( oQ.list.length ){
    oQ.deQueue();
}
```

第二步、虽然我们现在实现了一个队列，但是，这玩意是同步的，接下来继续改造成异步的：

```
var oQ = new Queue();
oQ.enQueue( function(){
    var _this = this;
    console.log( 'ghostwu1' );
    setTimeout( function(){ _this.deQueue(); }, 1000 );
}).enQueue( function(){
    var _this = this;
    console.log( 'ghostwu2' );
    setTimeout( function(){ _this.deQueue(); }, 1000 );
}).enQueue( function(){
    var _this = this;
    console.log( 'ghostwu3' );
    setTimeout( function(){ _this.deQueue(); }, 1000 );
}).deQueue();
```

第三步、这样就实现了一个异步队列， 这里有个小东西要注意下，把this保存下来，因为定时器的this指向的是window.另外在封装deQueue(出列)函数时，一定要给个空函数，否则出列完了之后，会报错，但是这玩意还是有耦合性，继续改造：

```
<input type="button" value="点我">
<ul id="box"></ul>
<script>
var Utils = {
    isFunction: function (a) {
        return Object.prototype.toString.call(a) === '[object Function]';
    },
    isNumber: function (a) {
        return typeof a === 'number';
    }
};
var Queue = function () {
    this.list = []
}
Queue.prototype = {
    constructor: Queue,
    enQueue: function (fn) {
        this.list.push(fn);
        return this;
    },
    delay: function (time) {
        this.list.push(time);
        return this;
    },
    deQueue: function () {
        var _this = this;
        var cur = this.list.shift() || function () { };
        if (Utils.isFunction(cur)) {
            cur.apply(_this, arguments);
            if (_this.list.length) _this.deQueue();
        } else if (Utils.isNumber(cur)) {
            setTimeout(function () {
                _this.deQueue();
            }, cur);
        }
    }
}

var oBtn = document.querySelector("input");
var oBox = document.querySelector("#box");
var create = (function () {
    var count = 0;
    return function () {
        var oLi = document.createElement("li");
        oLi.innerHTML = count++;
        return oLi;
    }
})();
oBtn.onclick = function () {
    var oQ = new Queue();
    function add() {
        for (var i = 0; i < 10; i++) {
            oQ.enQueue(function () {
                oBox.appendChild(create());
            }).delay(1000);
        }
    }
    add();
    oQ.deQueue();
}
</script>
```

这样封装之后，我们的异步队列就变得通用一点了，把延时和业务逻辑分开处理