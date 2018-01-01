[toc]

## 前言

又到了扯淡时间了，我最近在思考javascript事件机制底层的实现，但是暂时没有勇气去看chrome源码，所以今天我来猜测一把

我们今天来猜一猜，探讨探讨，javascript底层事件机制是如何实现的

博客里面关于事件绑定与执行顺序一块理解有误，请看最新博客

## 基础知识

### 事件捕获/冒泡

我们点击一个span，我可能就想点击一个span，事实上他是先点击document，然后点击事件传递到span的，而且并不会在span停下，span有子元素就会继续往下，最后会依次回传至document，我们这里偷一张图：

![image](http://images.cnitblog.com/blog/294743/201312/07132637-3ecf3bb32e3b45968f27d21bf1fe3aa5.png)

我们这里偷了一张图，这张图很好的说明了事件的传播方式

事件冒泡即由最具体的元素（文档嵌套最深节点）接收，然后逐步上传至document

事件捕获会由最先接收到事件的元素然后传向最里边（我们可以将元素想象成一个盒子装一个盒子，而不是一个积木堆积）
这里我们进入dom事件流，这里我们详细看看javascript事件的传递方式

**DOM事件流**

DOM2级事件规定事件包括三个阶段：

1. 事件捕获阶段
2. 处于目标阶段
3. 事件冒泡阶段

### 事件对象

所谓事件对象，是与特定对象相关，并且包含该事件详细信息的对象。

事件对象作为参数传递给事件处理程序（IE8之前通过window.event获得），所有事件对象都有事件类型type与事件目标target（IE8之前的srcElement我们不关注了）

各个事件的事件参数不一样，比如鼠标事件就会有相关坐标，包含和创建他的特定事件有关的属性和方法，触发的事件不一样，参数也不一样（比如鼠标事件就会有坐标信息），我们这里题几个较重要的

> 以下的兄弟全部是只读的，所以不要妄想去随意更改，IE之前的问题我们就不关注了

#### bubbles

表明事件是否冒泡

#### cancelable

表明是否可以取消事件的默认行为

#### currentTarget

某事件处理程序当前正在处理的那个元素

#### defaultPrevented

为true表明已经调用了preventDefault（DOM3新增）

#### eventPhase

调用事件处理程序的阶段：1 捕获；2 处于阶段；3 冒泡阶段

这个属性的变化需要在断点中查看，不然你看到的总是0

#### target

事件目标（绑定事件那个dom）

#### trusted

true表明是系统的，false为开发人员自定义的（DOM3新增）

#### type

事件类型

#### view

与事件关联的抽象视图，发生事件的window对象

#### preventDefault

取消事件默认行为，cancelable是true时可以使用

#### stopPropagation

取消事件捕获/冒泡，bubbles为true才能使用

#### stopImmediatePropagation

取消事件进一步冒泡，并且组织任何事件处理程序被调用（DOM3新增）

在我们的事件处理内部，this与currentTarget相同

## 模拟javascript事件机制

在此之前，我们来说几个基础知识点

### dom唯一标识

在页面上的dom，每个dom都应该有其唯一标识——`_zid`（我们这里统一为`_zid`）/sourceIndex，但是多数浏览器可能认为，这个接口并不需要告诉用户所以我们都不能获得

但是IE将这个接口放出来了——sourceIndex

我们这里以百度首页为例：

```
var doms = document.getElementsByTagName('*');
var str = '';
for (var i = 0, len = doms.length; i < len; i++) {
    str += doms[i].tagName + ': ' + doms[i].sourceIndex + '\n';
}
```

![image](http://images.cnitblog.com/blog/294743/201312/17000708-942f9893639949cd989bda8a9f12ae73.png)

![image](http://images.cnitblog.com/blog/294743/201312/17000716-272150f4c36d429aa896c0dd1333a9f5.png)

可以看到，越是上层的_zid越小

其实，dom _zid生成规则应该是以树的正序而来（好像是吧.....），反正是从上到下，从左到右

有了这个后，我们来看看我们如何获得一个dom的注册事件集合

获取dom注册事件集合

比如我们为一个dom同时绑定了2个click事件，又给他绑定一个keydown事件，那么对于这个dom来说他就具有3个事件了

我们有什么办法可以获得一个dom注册的事件呢？？？

答案很遗憾，浏览器都没有放出api，所以我们暂时不能知道一个dom到底被注册了多少事件......

PS：如果您知道这个问题的答案，请留言

有了以上两个知识点，我们就可以开始今天的扯淡了

> 注意：下文进入猜想时间

**补充点**

这里通过园友 JexCheng 的提示，其实一些浏览器是提供了获取dom事件节点的方法的

DOM API是没有。不过浏览器提供了一个调试用的接口。
Chrome在console下可以运行下面这个方法：
getEventListeners(node)，
获得对象上绑定的所有事件监听函数。

> 注意，是在console里面执行getEventListeners方法

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title></title>
</head>
<body>
<div id="d">ddssdsd</div>
  <script type="text/javascript">
    var node = document.getElementsByTagName('*');
    var d = document.getElementById('d');
    d.addEventListener('click', function () {
      alert();
    }, false);
    d.addEventListener('click', function () {
      alert('我是第二次');
    }, false);
    d.onclick = function () {
      alert('不规范的绑定');
    }
    d.addEventListener('click', function () {
      alert();
    }, true);

    d.addEventListener('mousedown', function () {
      console.log('mousedown');
    }, true);
    var evets = typeof getEventListeners == 'function' && getEventListeners(d)
  </script>
</body>
</html>
```

以上代码在chrome中的console结果为：

![image](http://images.cnitblog.com/blog/294743/201312/17124749-231ce4a078904dcf97a12c0a866ca561.png)

可以看到，无论何种绑定，这里都是可以获取的，而且获取的对象与我们模拟的对象比较接近

### 事件注册发生的事

首先，我们为dom注册事件的语法是：

```
dom.addEventListener('click', function () {
    alert('ddd');
})
```

以上述代码来说，我作为浏览器，以这个代码来说，在注册阶段我便可以保存以下信息：

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <style type="text/css">
         #p { width: 300px; height: 300px; padding: 10px;  border: 1px solid black; }
         #c { width: 100px; height: 100px; border: 1px solid red; }
    </style>
</head>
<body>
    <div id="p">
        parent
        <div id="c">
            child
        </div>
    </div>
    <script type="text/javascript">
        var p = document.getElementById('p'),
        c = document.getElementById('c');
        c.addEventListener('click', function () {
            alert('子节点捕获')
        }, true);

        c.addEventListener('click', function () {
            alert('子节点冒泡')
        }, false);

        p.addEventListener('click', function () {
            alert('父节点捕获')
        }, true);

        p.addEventListener('click', function () {
            alert('父节点冒泡')
        }, false);
    </script>
</body>
</html>
```

![image](http://images.cnitblog.com/blog/294743/201312/17001454-21a84820dead4b20add9052ef3cb2f14.png)

这里，我们为parent和child绑定了click事件，所以浏览器可以获得如下队列结构：

```
/****** 第一步-注册事件 ******/
//页面事件存储在一个队列里
//以_zid排序
var eventQueue = [
  {
    _zid: 'parent',
    handlers: {
      click: {
        captrue: [fn, fn],
        bubble: [fn, fn]
      }
    }
  },
  {
    _zid:'child',
    handlers:{
      click: {
        captrue: [],
        bubble: []
      }
    }
  },
  {
    _zid: '_zid',
    handlers: {
    //……
    }
  }
];
```

就那parent这个div来说，我们为他绑定了两个click事件（我们其实可以绑定3个4个或者更多，所以事件集合是一个数组，执行具有先后顺序）

其中注册事件时候，又会分冒泡和捕获，而且这里以_zid排序（比如：document->body->div#p->div#c）

然后第一个阶段就结束了

PS：我想底层c++语言一定有类似的这个队列，而且可以释放接口，让我们获取一个dom所注册的所有事件

注意，此处队列是这样，但是我们真正点击一个元素，可能就只抽取其中一部分关联的对象组成一个新的队列，供下面使用

### 初始化事件参数

第二步就是初始化事件参数，我们可以通过addEventListener，创建事件参数，但是我们这里简单模拟即可：

> 注意，为了方便理解，我们这里暂不考虑mousedown

```
/****** 第二步-初始化事件参数 ******/
var Event = {};
Event.type = 'click';
Event.target = el;//当前手指点击最深dom元素
//初始化信息
//......
//鼠标位置信息等
```

在这里比较关键的就是我们一定要好好定义我们的target！！！

于是可以进入我们的关键步骤了，触发事件

### 触发事件

事件触发分三步走，首先是捕获然后是处于阶段最后是冒泡阶段：

```
/****** 第三步-触发事件 ******/
var isTarget = false;
Event.eventPhase = 1;
//首先是捕获阶段，事件执行至event.target为止，我们这里只关注click
for (var index = 0, length = eventQueue.lenth; index < length; index++) {
  //获取捕获时期该元素的click事件集合
  var clickHandlers = eventQueue[index].handlers.click.captrue;
  for (var i = 0, len = clickHandlers.length; i < len; i++) {
    Event.currentTarget = clickHandlers[i]; //事件处理程序当前正在处理的那个元素
    //执行至target便跳出循环，不再执行下面的操作
    if (Event.target._zid == eventQueue[index]._zid) {
      Event.eventPhase = 2;//当前阶段
      isTarget = true;
    }
    //执行绑定事件
    clickHandlers[i](Event);
    //如果阻止冒泡，跳出所有循环，不执行后面的事件
    if (Event.bubbles) {
      return;
    }
  }
  //若是当前已经是target便不再向下捕获
  if(isTarget) break;
}
Event.eventPhase = 3;
//冒泡阶段
for(var index = eventQueue.lenth; index !=0; index--) {
  //如果zid小于等于当前元素，说明不需要处理
  if(eventQueue[index]._zid <= Event.target._zid) continue;
  //需要处理的部分了
  var clickHandlers = eventQueue[index].handlers.click.bubble;
 
  //此段代码可以重构，暂时不管
  for (var i = 0, len = clickHandlers.length; i < len; i++) {
    Event.currentTarget = clickHandlers[i]; //事件处理程序当前正在处理的那个元素
    //执行绑定事件
    clickHandlers[i](Event);
    //如果阻止冒泡，跳出所有循环，不执行后面的事件
    if (Event.bubbles) {
      return;
    }
  }
}
```

这个注释写的很清楚了应该能表达清楚我的意思，于是我们这里就简单的模拟了事件机制的底层原理了：）

PS：如果您觉得不对，请留言

## 验证猜想

现在，基础理论提出来了，我们需要验证下这个想法是否站得住脚，所以这里提了几个例子，首先我们回到上面的问题吧

### 验证一：点击问题

http://sandbox.runjs.cn/show/pesvelp1

首先我们来看这个问题，我们分别为parent与child注册了两个click事件，一次冒泡一次捕获

当我们点击父元素时，我们按照理论的执行逻辑如下：

开始遍历事件队列（由document开始）

当遍历对象如果注册了click事件就会触发，如果阻止了冒泡，执行后便跳出循环不再执行

因为之前并没有注册事件，所以直接到了parent，这里发现parent的_zid与target的_zid相等

于是便将状态置为处于目标阶段，并打上标记跳出捕获循环，不再执行后面的事件句柄

```
Event.eventPhase = 2;//当前阶段
isTarget = true;
```

捕获结束后，开始执行冒泡的事件，循环由后向前，开始是child的click事件，但是此时child的`_zid`大于target的`_zid`所以继续循环

最后会执行parent以上的dom注册的click事件，没有就算了

至于点击child的逻辑我们这里就不分析了

### 验证二：突然移除dom

我们这里对上题做一个变形，我们在parent点击时候（捕获阶段）将child div给删除，看看有什么情况

http://sandbox.runjs.cn/show/f1ke5vp8

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <style type="text/css">
         #p { width: 300px; height: 300px; padding: 10px;  border: 1px solid black; }
         #c { width: 100px; height: 100px; border: 1px solid red; }
    </style>
</head>
<body>
    <div id="p">
        parent
        <div id="c">
            child
        </div>
    </div>
    <script type="text/javascript">
        var p = document.getElementById('p'),
        c = document.getElementById('c');
        c.addEventListener('click', function () {
            alert('子节点捕获')
        }, true);

        c.addEventListener('click', function () {
            alert('子节点冒泡')
        }, false);

        p.addEventListener('click', function () {
            alert('父节点捕获')
            p.removeChild(c);
        }, true);

        p.addEventListener('click', function () {
            alert('父节点冒泡')
        }, false);
    </script>
</body>
</html>
```

其实这里还有一个优化点，相信大家都知道：

移除dom并不会移除事件句柄，这个必须手动释放
就是因为这个原因，我们的整个逻辑仍然会执行，各位自己可以试试

### 验证三：child阻止冒泡

我们这里再将上题稍加变形，在child 冒泡阶段组织冒泡，其实这个不用说，parent的click不会执行

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <style type="text/css">
         #p { width: 300px; height: 300px; padding: 10px;  border: 1px solid black; }
         #c { width: 100px; height: 100px; border: 1px solid red; }
    </style>
</head>
<body>
    <div id="p">
        parent
        <div id="c">
            child
        </div>
    </div>
    <script type="text/javascript">
        var p = document.getElementById('p'),
        c = document.getElementById('c');
        c.addEventListener('click', function () {
            alert('子节点捕获')
        }, true);

        c.addEventListener('click', function (e) {
            alert('子节点冒泡')
            e.stopPropagation();
        }, false);

        p.addEventListener('click', function () {
            alert('父节点捕获')
        }, true);

        p.addEventListener('click', function () {
            alert('父节点冒泡')
        }, false);
    </script>
</body>
</html>
```

### 验证四：模拟click事件

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <style type="text/css">
         #p { width: 300px; height: 300px; padding: 10px;  border: 1px solid black; }
         #c { width: 100px; height: 100px; border: 1px solid red; }
    </style>
</head>
<body>
    <div id="p">
        parent
        <div id="c">
            child
        </div>
    </div>
    <script type="text/javascript">
        alert = function (msg) {
            console.log(msg);
        }

        var p = document.getElementById('p'),
        c = document.getElementById('c');
        c.addEventListener('click', function (e) {
            console.log(e);
            alert('子节点捕获')
        }, true);
        c.addEventListener('click', function (e) {
            console.log(e);
            alert('子节点冒泡')
        }, false);

        p.addEventListener('click', function (e) {
            console.log(e);
            alert('父节点捕获')
        }, true);

        p.addEventListener('click', function (e) {
            console.log(e);
            alert('父节点冒泡')
        }, false);

        document.addEventListener('keydown', function (e) {
            if (e.keyCode == '32') {
                var type = 'click'; //要触发的事件类型
                var bubbles = true; //事件是否可以冒泡
                var cancelable = true; //事件是否可以阻止浏览器默认事件
                var view = document.defaultView; //与事件关联的视图，该属性默认即可，不管
                var detail = 0;
                var screenX = 0;
                var screenY = 0;
                var clientX = 0;
                var clientY = 0;
                var ctrlKey = false; //是否按下ctrl
                var altKey = false; //是否按下alt
                var shiftKey = false;
                var metaKey = false;
                var button = 0; //表示按下哪一个鼠标键
                var relatedTarget = 0; //模拟mousemove或者out时候用到，与事件相关的对象
                var event = document.createEvent('Events');
                event.myFlag = '叶小钗';
                event.initEvent(type, bubbles, cancelable, view, detail, screenX, screenY, clientX, clientY,
ctrlKey, altKey, shiftKey, metaKey, button, relatedTarget);
                
                console.log(event);
                c.dispatchEvent(event);
            }
        }, false);
    </script>
</body>
</html>
```

http://sandbox.runjs.cn/show/pesvelp1

我们最后模拟一下click事件，这里按空格便会触发child的click事件，这里依然走我们上述逻辑

所以，我们今天到此为止

## 结语

今天，我们一起模拟猜测了javascript事件机制的底层实现，这里只做了最简单最单纯的模拟

比如两个平级dom（div）点击时候这里的算法就有一点问题，但是无伤大雅，探讨嘛，至于事情的真相如何，这里就只能抛砖引玉了。