## 一、题目介绍

以下是我copy自网上的面试题原文：

实现一个LazyMan，可以按照以下方式调用:

```
LazyMan("Hank")输出:
Hi! This is Hank!

LazyMan("Hank").sleep(10).eat("dinner")输出
Hi! This is Hank!
//等待10秒..
Wake up after 10
Eat dinner~

LazyMan("Hank").eat("dinner").eat("supper")输出
Hi This is Hank!
Eat dinner~
Eat supper~

LazyMan("Hank").sleepFirst(5).eat("supper")输出
//等待5秒
Wake up after 5
Hi This is Hank!
Eat supper
```
以此类推。

## 二、题目考察的点

先声明：我不是微信员工，考察的点是我推测的，可能不是，哈哈！

1. 方法链式调用
2. 类的使用和面向对象编程的思路
3. 设计模式的应用
4. 代码的解耦
5. 最少知识原则，也即 迪米特法则（Law of Demeter）
6. 代码的书写结构和命名

## 三、题目思路解析

1. 看题目输出示例，可以确定这是拟人化的输出，也就是说：应该编写一个类来定义一类人，叫做LazyMan。可以输出名字、吃饭、睡觉等行为。
2. 从输出的句子可以看出，sleepFrist的优先级是最高的，其他行为的优先级一致。
3. 从三个例子来看，都得先调用LazyMan来初始化一个人，才能继续后续行为，所以LazyMan是一个接口。
4. 句子是按调用方法的次序进行顺序执行的，是一个队列。

## 四、采用观察者模式实现代码

### 4.1 采用模块模式来编写代码

```
(function(window, undefined){

)(window);
```

### 4.2 声明一个变量taskList，用来存储需要队列信息

```
(function(window, undefined){
    var taskList = [];
)(window);
```

队列中，单个项的存储设计为一个json，存储需要触发的消息，以及方法执行时需要的参数列表。比如LazyMan('Hank')，需要的存储信息如下。

```
{
    'msg':'LazyMan',
    'args':'Hank'
}
```

当执行LazyMan方法的时候，调用订阅方法，将需要执行的信息存入taskList中，缓存起来。
存储的信息，会先保留着，等发布方法进行提取，执行和输出。

### 4.3 订阅方法

订阅方法的调用方式设计：subscribe("lazyMan", "Hank")

```
(function(window, undefined){
    var taskList = [];

    // 订阅
    function subscribe(){
        var param = {},
            args = Array.prototype.slice.call(arguments);

        if(args.length < 1){
            throw new Error("subscribe 参数不能为空!");
        }

        param.msg = args[0]; // 消息名
        param.args = args.slice(1); // 参数列表

        if(param.msg == "sleepFirst"){
            taskList.unshift(param);
        }else{
            taskList.push(param);
        }
    }
)(window);
```

用一个param变量来组织好需要存储的信息，然后push进taskList中，缓存起来。
特别的，如果是sleepFirst，则放置在队列头部。

### 4.4 发布方法

```
(function(window, undefined){
    var taskList = [];

    // 发布
    function publish(){
        if(taskList.length > 0){
            run(taskList.shift());
        }
    }
)(window);
```

将队列中的存储信息读取出来，交给run方法(暂定，后续实现)去执行。这里限定每次发布只执行一个，以维持队列里面的方法可以挨个执行。
另外，这里使用shift()方法的原因是，取出一个，就在队列中删除这一个，避免重复执行。

### 4.5 实现LazyMan类

```
// 类
function LazyMan(){};

LazyMan.prototype.eat = function(str){
    subscribe("eat", str);
    return this;
};

LazyMan.prototype.sleep = function(num){
    subscribe("sleep", num);
    return this;
};

LazyMan.prototype.sleepFirst = function(num){
    subscribe("sleepFirst", num);
    return this;
};
```

将LazyMan类实现，具有eat、sleep、sleepFrist等行为。
触发一次行为，就在taskList中记录一次，并返回当前对象，以支持链式调用。

### 4.6 实现输出console.log的包装方法

```
// 输出文字
function lazyManLog(str){
    console.log(str);
}
```

为什么还要为console.log包装一层，是因为在实战项目中，产经经常会修改输出提示的UI。如果每一处都用console.log直接调用，那改起来就麻烦很多。
另外，如果要兼容IE等低级版本浏览器，也可以很方便的修改。
也就是DRY原则（Don't Repeat Youself）。

### 4.7 实现具体执行的方法

```
// 具体方法
function lazyMan(str){
    lazyManLog("Hi!This is "+ str +"!");

    publish();
}

function eat(str){
    lazyManLog("Eat "+ str +"~");
    publish();
}

function sleep(num){
    setTimeout(function(){
        lazyManLog("Wake up after "+ num);

        publish();
    }, num);

}

function sleepFirst(num){
    setTimeout(function(){
        lazyManLog("Wake up after "+ num);

        publish();
    }, num);
}
```

这里的重点是解决setTimeout执行时会延迟调用，也即线程异步执行的问题。只有该方法执行成功后，再发布一次消息publish()，提示可以执行下一个队列信息。否则，就会一直等待。

### 4.8 实现run方法，用于识别要调用哪个具体方法，是一个总的控制台

```
// 鸭子叫
function run(option){
    var msg = option.msg,
        args = option.args;

    switch(msg){
        case "lazyMan": lazyMan.apply(null, args);break;
        case "eat": eat.apply(null, args);break;
        case "sleep": sleep.apply(null,args);break;
        case "sleepFirst": sleepFirst.apply(null,args);break;
        default:;
    }
}
```

这个方法有点像鸭式辨型接口，所以注释叫鸭子叫。
run方法接收队列中的单个消息，然后读取出来，看消息是什么类型的，然后执行对应的方法。

### 4.9 暴露接口LazyMan，让外部可以调用

```
(function(window, undefined){
    // 暴露接口
    window.LazyMan = function(str){
        subscribe("lazyMan", str);

        setTimeout(function(){
            publish();
        }, 0);

        return new LazyMan();
    };
)(window);
```

接口LazyMan里面的publish方法必须使用setTimeout进行调用。这样能让publish()执行的线程延后，挂起。等链式方法都执行完毕后，线程空闲下来，再执行该publish()。
另外，这是一个对外接口，所以调用的时候，同时也会new 一个新的LazyMan，并返回，以供调用。

## 五、总结

1. 好处

使用观察者模式，让代码可以解耦到合理的程度，使后期维护更加方便。
比如我想修改eat方法，我只需要关注eat()和LazyMan.prototype.eat的实现。其他地方，我都可以不用关注。这就符合了最少知识原则。

2. 不足
LazyMan.prototype.eat这种方法的参数，其实可以用arguments代替，我没写出来，怕弄得太复杂，就留个优化点吧。
使用了unshift和shift方法，没有考虑到低版本IE浏览器的兼容。

## 六、完整源码和线上demo

完整源码已经放在我的gitHub上

github地址：https://github.com/wall-wxk/blogDemo/blob/master/2017/01/22/lazyMan.html

demo访问地址：https://wall-wxk.github.io/blogDemo/2017/01/22/lazyMan.html

demo需要打开控制台，在控制台中调试代码。

七、番外

网上有人也实现了lazyMan，但是实现的方式我不是很喜欢和认同，但是也是一种思路，这里顺便贴出来给大伙看看。

```
function _LazyMan(name) {
    this.tasks = [];   
    var self = this;
    var fn =(function(n){
        var name = n;
        return function(){
            console.log("Hi! This is " + name + "!");
            self.next();
        }
    })(name);
    this.tasks.push(fn);
    setTimeout(function(){
        self.next();
    }, 0); // 在下一个事件循环启动任务
}
/* 事件调度函数 */
_LazyMan.prototype.next = function() { 
    var fn = this.tasks.shift();
    fn && fn();
}
_LazyMan.prototype.eat = function(name) {
    var self = this;
    var fn =(function(name){
        return function(){
            console.log("Eat " + name + "~");
            self.next()
        }
    })(name);
    this.tasks.push(fn);
    return this; // 实现链式调用
}
_LazyMan.prototype.sleep = function(time) {
    var self = this;
    var fn = (function(time){
        return function() {
            setTimeout(function(){
                console.log("Wake up after " + time + "s!");
                self.next();
            }, time * 1000);
        }
    })(time);
    this.tasks.push(fn);
   return this;
}
_LazyMan.prototype.sleepFirst = function(time) {
    var self = this;
    var fn = (function(time) {
        return function() {
            setTimeout(function() {
                console.log("Wake up after " + time + "s!");
                self.next();
            }, time * 1000);
        }
    })(time);
    this.tasks.unshift(fn);
    return this;
}
/* 封装 */
function LazyMan(name){
    return new _LazyMan(name);
}
```