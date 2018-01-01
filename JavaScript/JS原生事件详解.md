[toc]

对于事件来讲，首先，我们需要了解这样几个概念：事件；事件处理程序；事件类型；事件流；事件冒泡；事件捕获；事件对象；事件模拟，事件方面的性能优化（事件委托、移除事件处理程序）； 

## 事件的概念

事件：指的是文档或者浏览器窗口中发生的一些特定交互瞬间。我们可以通过监听器（或者处理程序）来预定事件，以便事件发生的时候执行相应的代码。

事件处理程序：我们用户在页面中进行的点击这个动作，鼠标移动的动作，网页页面加载完成的动作等，都可以称之为事件名称，即：click、mousemove、load等都是事件的名称。响应某个事件的函数则称为事件处理程序，或者叫做事件监听器。 
实例：具体解释见代码注释：

~~~
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>事件</title>
</head>

<body>
<div id="tg">
    理解事件的基本概念
</div>
</body>
<script>

    var tg=document.getElementById('tg');//根据指定的 id 属性值得到对象
    //此处click 点击 是一种事件名称 是浏览器窗口中发生点击的瞬间 on这个单词，就是响应click这个事件 所以onclick就是事件处理程序 又叫事件侦听器 作用是为tg这个元素预定了点击 在点击发生时 执行函数中的代码
    tg.onclick=function(){
        alert('点了我');

    }
</script>
</html>
~~~

关于事件处理程序，从最初的发展到现在，经历了几次的变化。

事件处理程序的名字是以“on”开头，因此click事件的处理程序就是onclick。在最初，是使用HTML事件处理程序的，也就是说，某个元素（如div），支持的每一种事件，都可以使用一个与相应事件处理程序同名的HTML特性来制定（也就是标签的一个属性），这个特性的值就是能够执行的JavaScript代码。例如：

~~~
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>HTML事件</title>
</head>

<body>
<p onclick="alert('click')">HTML事件处理程序</p>

</body>
</html>
~~~

当然，我们也可以再onclick=””当中进行函数的调用。（不建议使用这种HTML事件）

在DOM0级事件处理程序推出之后，广为各个用户的使用，但是，却出现了这样一个问题，当我希望为同一个元素/标签绑定多个同类型事件的时候（如，为上面的这个p标签绑定3个点击事件），是不被允许的。那么，此时，出现了另一种事件处理程序，就是DOM2级的事件处理程序，在DOM2级当中，定义了两个基本方法，用于处理指定（即绑定）和删除事件处理程序的操作，分别是addEventListener()和removeEventListener 
具体实例代码如下：

~~~
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>绑定多个同类型事件</title>
</head>

<body>
<p id='btn'>DOM0级事件处理程序</p>
</body>
<script>
    var btn = document.getElementById('btn');
    btn.addEventListener("click", test, false);
    function test(){
        alert(this.innerHTML);
    }
</script>

</html>
~~~

addEventListener()和removeEventListener()中的第三个参数，表示的是在哪个事件阶段进行事件处理，如果是false，则指的是冒泡阶段；如果是true，则指的是捕获阶段。

## 事件类型

### 单击事件onClick

　　当用户单击鼠标按钮时，产生onClick事件。同时onClick指定的事件处理程序或代码将被调用执行。通常在下列基本对象中产生：

~~~
button（按钮对象） 
checkbox（复选框）或（检查列表框） 
radio （单选钮） 
reset buttons（重要按钮） 
submit buttons（提交按钮）
~~~

在onClick等号后，可以使用自己编写的函数作为事件处理程序，也可以使用JavaScript中内部的函数。还可以直接使用JavaScript的代码等。例： 

### onChange改变事件

　　当利用text或texturea元素输入字符值改变时发该事件，同时当在select表格项中一个选项状态改变后也会引发该事件。

### 选中事件onSelect

　　当Text或Textarea对象中的文字被加亮后，引发该事件。

### 获得焦点事件onFocus

　　当用户单击Text或textarea以及select对象时，产生该事件。此时该对象成为前台对象。

### 失去焦点onBlur

　　当text对象或textarea对象以及select对象不再拥有焦点、而退到后台时，引发该文件，他与onFocas事件是一个对应的关系。

### 载入文件onLoad

　　当文档载入时，产生该事件。onLoad一个作用就是在首次载入一个文档时检测cookie的值，并用一个变量为其赋值，使它可以被源代码使用。

### 卸载文件onUnload

　　当Web页面退出时引发onUnload事件，并可更新Cookie的状态。

![](http://olphpb1zg.bkt.clouddn.com/57ce817fd4e53.png)

## 事件流

### 事件流：描述的是从页面中接收事件的顺序。 
1. 事件捕获阶段
2. 处于目标阶段
3. 事件冒泡阶段

IE与原来的NetScape(网景)，对于事件流提出的是完全不同的顺序。IE团队提出的是事件冒泡流；NetScape的事件流是事件捕获流。 

### 事件冒泡

事件冒泡：表示的是，事件开始的时候由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。 

###事件捕获

事件捕获：表示的是，事件开始的时候由最不具体的节点接收，然后逐级向下传播到最具体的节点。 
![](http://olphpb1zg.bkt.clouddn.com/57ce817796bbc.png)

来看一个实例：

~~~
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <title>tg</title>
</head>
<body>
    <div id="tg">
        <p>理解事件的基本概念</p>
    </div>
</body>
</html>
~~~

如果单击了p标签，那么，如果是事件冒泡流的事件流机制，则click事件将按照如下顺序进行执行：p —— div —— body —— html —— document。

如果采用捕获流的事件流机制，则click事件的执行顺序为：document —— html —— body —— div —— p 
实例：

~~~
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>事件流</title>

</head>

<body>
<div id="p">
    parent
    <div id="c">
        child
    </div>
</div>
<script type="text/javascript">
    var p = document.getElementById('p');
    var  c = document.getElementById('c');
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
~~~

1. 点击parent，事件首先在document上然后parent捕获到事件，处于目标阶段然后event.target也等于parent，所以触发捕获事件由于target与currentTarget相等了，所以认为到底了，开始冒泡，执行冒泡事件
2. 点击child情况有所不同，事件由document传向parent执行事件，然后传向child最后开始冒泡，所以执行顺序各位一定要清晰 

### 事件对象

事件对象：在触发DOM上的某个事件的时候，会产生一个事件对象event，而在这个对象当中会包含着所有与事件有关的信息。

所谓事件对象，是与特定对象相关，并且包含该事件详细信息的对象。

事件对象作为参数传递给事件处理程序（IE8之前通过window.event获得），所有事件对象都有事件类型type与事件目标target（IE8之前的srcElement我们不关注了）

各个事件的事件参数不一样，比如鼠标事件就会有相关坐标，包含和创建他的特定事件有关的属性和方法，触发的事件不一样，参数也不一样（比如鼠标事件就会有坐标信息），我们这里题几个较重要的

实例1

~~~
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <title>事件</title>
</head>
<body>
<div id="main">理解事件的基本概念</div>
</body>
<script>
    var main = document.getElementById('main');
    main.onclick = function(event){
        console.log(event);
    }
</script>
</html>
~~~


**bubbles**

表明事件是否冒泡

**cancelable**

表明是否可以取消事件的默认行为

**currentTarget**

某事件处理程序当前正在处理的那个元素

**defaultPrevented**

为true表明已经调用了preventDefault（DOM3新增）

**eventPhase**

调用事件处理程序的阶段：1 捕获；2 处于阶段；3 冒泡阶段

这个属性的变化需要在断点中查看，不然你看到的总是0

**target**

事件目标（绑定事件那个dom）

**trusted**

true表明是系统的，false为开发人员自定义的（DOM3新增）

**type**

事件类型

**view**

与事件关联的抽象视图，发生事件的window对象

**preventDefault**

取消事件默认行为，cancelable是true时可以使用

**stopPropagation**

取消事件捕获/冒泡，bubbles为true才能使用

**stopImmediatePropagation**

取消事件进一步冒泡，并且组织任何事件处理程序被调用（DOM3新增）

在我们的事件处理内部，this与currentTarget相同

**createEvent**

可以在document对象上使用createEvent创建一个event对象

DOM3新增以下事件： 

1. UIEvents 
2. MouseEvents 
3. MutationEvents，一般化dom变动 
4. HTMLEvents一般dom事件

使用console.log打印出的结果。 

实例某段代码

~~~
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
var button = 0;//表示按下哪一个鼠标键
var relatedTarget = 0; //模拟mousemove或者out时候用到，与事件相关的对象

var event = document.createEvent('MouseEvents');
event.initMouseEvent(type, bubbles, cancelable, view, detail, screenX, screenY, clientX, clientY,
    ctrlKey, altKey, shiftKey, metaKey, button, relatedTarget);
~~~

我们就自己创建了一个event对象，然后可以传给我们自己创建的事件。

其中有两个信息，我们最为常用，分别是type和target。

type表示的是被触发事件的类型；

target表示的是事件的目标。

其他信息，如：

bubbles：表示事件是否冒泡
cancelable：表示是否可以取消事件的默认行为
currentTarget：表示事件处理程序当前正在处理事件的那个元素
defaultPrevented：表示是否调用了preventDefault()
detail：表示的是与事件相关的细节信息
eventPhase：调用事件处理处理程序的阶段：1表示捕获阶段、2表示处于目标、3表示冒泡阶段

### 事件模拟

事件模拟是javascript事件机制中相当有用的功能，理解事件模拟与善用事件模拟是判别一个前端的重要依据，事件一般是由用户操作触发，其实javascript也是可以触发的，比较重要的是，javascript的触发事件还会冒泡。意思就是，javascript触发的事件与浏览器本身触发其实是一样的（并不完全一致）

如此，我们这里来通过键盘事件触发刚刚的点击事件吧，我们这里点击键盘便触发child的点击，看看他的表现如何

由于是键盘触发，便不具有相关参数了，我们可以捕捉event参数，这对我们队事件传输的理解有莫大的帮助：

我们这里先创建事件参数，然后给键盘注册事件，在点击键盘时候便触发child的点击事件 
实例

~~~
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>无标题文档</title>
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
            event.myFlag = '汤小高';
            event.initEvent(type, bubbles, cancelable, view, detail, screenX, screenY, clientX, clientY,
                    ctrlKey, altKey, shiftKey, metaKey, button, relatedTarget);
            console.log(event);
            c.dispatchEvent(event);
        }
    }, false);
</script>

</body>
</html>
~~~

## 事件方面性能优化

谈一谈事件方面如何优化性能——事件委托和事件处理程序的移除

在JavaScript代码当中，添加到页面中的事件越多，页面的性能也就越差。导致这一问题的原因主要有：

每个函数都是对象，都会占用内存。内存中对象越多，性能也就越差。
必须事先指定所有事件处理程序而导致的DOM访问次数，会延迟整个页面的交互就绪时间。
为了进行页面的性能优化，因此我们会采用两种方法，就是上面提到的——事件委托和事件处理程序的移除。 
事件委托

什么时候使用事件委托，其实，简单来说，当时一个页面事件处理程序比较多的时候，我们通常情况下会使用它。

事件委托主要利用了事件冒泡，只指定一个事件处理程序，就可以管理一个类型的所有事件。例如：我们为整个一个页面制定一个onclick事件处理程序，此时我们不必为页面中每个可点击的元素单独设置事件处理程序（onclick）。还是，看一个例子。 
效果：点击不同的元素执行不同的操作。 
不使用事件委托：

~~~
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <title>事件</title>
    <style>
        *{margin: 0;padding: 0;}
        .control span{
            float: left;
            width: 50px;
            height: 50px;
            line-height: 50px;
            font-size: 32px;
            text-align: center;
            border: 1px solid #f00;
        }
    </style>
</head>
<body>
<div class='control'>
    <span id='left'>左</span>
    <span id='first'>1</span>
    <span id='second'>2</span>
    <span id='third'>3</span>
    <span id='right'>右</span>
</div>
</body>
<script>
    var left = document.getElementById('left');
    var first = document.getElementById('first');
    var second = document.getElementById('second');
    var third = document.getElementById('third');
    var right = document.getElementById('right');
    left.addEventListener("click", function(){
        alert('点击的是左这个字，执行相关操作');
    }, false);
    first.addEventListener("click", function(){
        alert('要执行第一个序号对应的相关操作');
    }, false);
    second.addEventListener("click", function(){
        alert('要执行第二个序号对应的相关操作');
    }, false);
    third.addEventListener("click", function(){
        alert('要执行第三个序号对应的相关操作');
    }, false);
    right.addEventListener("click", function(){
        alert('点击的是右这个字，执行相关操作');
    }, false);
</script>
</html>
~~~

不难看出，我们使用了5个事件侦听器，每设置一个就需要绑定一个。 
使用事件委托：

~~~
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <title>事件-独行冰海</title>
    <style>
        *{margin: 0;padding: 0;}
        .control span{
            float: left;
            width: 50px;
            height: 50px;
            line-height: 50px;
            font-size: 32px;
            text-align: center;
            border: 1px solid #f00;
        }
    </style>
</head>
<body>
<div class='control' id='control'>
    <span id='left'>左</span>
    <span id='first'>1</span>
    <span id='second'>2</span>
    <span id='third'>3</span>
    <span id='right'>右</span>
</div>
</body>
<script>
    var control = document.getElementById('control');
    control.addEventListener("click", function(e){
        // 注意，此处没有考虑IE8-的浏览器，如果想书写兼容全部浏览器的事件处理程序，请查看下一篇博文
        var target = e.target;
        // 在此使用的是switch语句，也可以采用if进行判断
        switch(target.id){
            case "left" : {
                alert('点击的是左这个字，执行相关操作');
                break;
            }
            case "first" : {
                alert('要执行第一个序号对应的相关操作');
                break;
            }
            case "second" : {
                alert('要执行第二个序号对应的相关操作');
                break;
            }
            case "third" : {
                alert('要执行第三个序号对应的相关操作');
                break;
            }

            case "right" : {
                alert('点击的是右这个字，执行相关操作');
                break;
            }
        }
    }, false);
</script>
</html>
~~~

简要的总结一下所谓的事件委托：给元素的父级或者祖级，甚至页面绑定事件，然后利用事件冒泡的基本原理，通过事件目标对象进行检测，然后执行相关操作。其优势在于：

大大减少了事件处理程序的数量，在页面中设置事件处理程序的时间就更少了（DOM引用减少——也就是上面我们通过id去获取标签，所需要的查找操作以及DOM引用也就更少了）。
document（注：上面的例子没有绑定在document上，而是绑定到了父级的div上，最为推荐的是绑定在document上）对象可以很快的访问到，而且可以在页面生命周期的任何时点上为它添加事件处理程序，并不需要等待DOMContentLoaded或者load事件。换句话说，只要可单击的元素在页面中呈现出来了，那么它就立刻具备了相应的功能。
整个页面占用的内存空间会更少，从而提升了整体的性能。
移除事件处理程序

每当将一个事件处理程序指定给一个元素时，在运行中的浏览器代码与支持页面交互的JavaScript代码之间就会建立一个连接。连接数量也直接影响着页面的执行速度。所以，当内存中存在着过时的“空事件处理程序”的时候，就会造成Web应用程序的内存和性能问题。 
那么什么时候会造成“空事件处理程序”的出现呢？

文档中元素存在事件，通过一些DOM节点操作（removeChild、replaceChild等方法），移除了这个元素，但是DOM节点的事件没有被移除。
innerHTML去替换页面中的某一部分，页面中原来的部分存在事件，没有移除。
页面卸载引起的事件处理程序在内存中的滞留。

解决方法：

合理利用事件委托；
在执行相关操作的时候，先移除掉事件，再移除DOM节点；
在页面卸载之前，先通过onunload事件移除掉所有事件处理程序。