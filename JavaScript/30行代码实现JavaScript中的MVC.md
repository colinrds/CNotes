从09年左右开始，MVC逐渐在前端领域大放异彩，并终于在刚刚过去的2015年随着React Native的推出而迎来大爆发：AngularJS、EmberJS、Backbone、ReactJS、RiotJS、VueJS…… 一连串的名字走马观花式的出现和更迭，它们中一些已经渐渐淡出了大家的视野，一些还在迅速茁壮成长，一些则已经在特定的生态环境中独当一面舍我其谁。但不论如何，MVC已经并将持续深刻地影响前端工程师们的思维方式和工作方法。

很多讲解MVC的例子都从一个具体的框架的某个概念入手，比如Backbone的collection或AngularJS中model，这当然不失为一个好办法。但框架之所以是框架，而不是类库（jQuery）或者工具集（Underscore），就是因为它们的背后有着众多优秀的设计理念和最佳实践，这些设计精髓相辅相成，环环相扣，缺一不可，要想在短时间内透过复杂的框架而看到某一种设计模式的本质并非是一件容易的事。

这便是这篇随笔的由来——为了帮助大家理解概念而生的原型代码，应该越简单越好，简单到刚刚足以大家理解这个概念就够了。

### 1. MVC的基础是观察者模式，这是实现model和view同步的关键

为了简单起见，每个model实例中只包含一个primitive value值。

~~~
function Model(value) {
    this._value = typeof value === 'undefined' ? '' : value;
    this._listeners = [];
}
Model.prototype.set = function (value) {
    var self = this;
    self._value = value;
    // model中的值改变时，应通知注册过的回调函数
    // 按照Javascript事件处理的一般机制，我们异步地调用回调函数
    // 如果觉得setTimeout影响性能，也可以采用requestAnimationFrame
    setTimeout(function () {
        self._listeners.forEach(function (listener) {
            listener.call(self, value);
        });
    });
};
Model.prototype.watch = function (listener) {
    // 注册监听的回调函数
    this._listeners.push(listener);
};
// html代码：
<div id="div1"></div>
// 逻辑代码：
(function () {
    var model = new Model();
    var div1 = document.getElementById('div1');
    model.watch(function (value) {
        div1.innerHTML = value;
    });
    model.set('hello, this is a div');
})();
~~~
借助观察者模式，我们已经实现了在调用model的set方法改变其值的时候，模板也同步更新，但这样的实现却很别扭，因为我们需要手动监听model值的改变（通过watch方法）并传入一个回调函数，有没有办法让view（一个或多个dom node）和model更简单的绑定呢？

### 2. 实现bind方法，绑定model和view

~~~
Model.prototype.bind = function (node) {
    // 将watch的逻辑和通用的回调函数放到这里
    this.watch(function (value) {
        node.innerHTML = value;
    });
};
// html代码：
<div id="div1"></div>
<div id="div2"></div>
// 逻辑代码：
(function () {
    var model = new Model();
    model.bind(document.getElementById('div1'));
    model.bind(document.getElementById('div2'));
    model.set('this is a div');
})();
~~~
通过一个简单的封装，view和model之间的绑定已经初见雏形，即使需要绑定多个view，实现起来也很轻松。注意bind是Function类prototype上的一个原生方法，不过它和MVC的关系并不紧密，笔者又实在太喜欢bind这个单词，一语中的，言简意赅，所以索性在这里把原生方法覆盖了，大家可以忽略。言归正传，虽然绑定的复杂度降低了，这一步依然要依赖我们手动完成，有没有可能把绑定的逻辑从业务代码中彻底解耦呢？

### 3. 实现controller，将绑定从逻辑代码中解耦

细心的朋友可能已经注意到，虽然讲的是MVC，但是上文中却只出现了Model类，View类不出现可以理解，毕竟HTML就是现成的View（事实上本文中从始至终也只是利用HTML作为View，javascript代码中并没有出现过View类），那Controller类为何也隐身了呢？别急，其实所谓的”逻辑代码”就是一个框架逻辑（姑且将本文的原型玩具称之为框架）和业务逻辑耦合度很高的代码段，现在我们就来将它分解一下。

如果要将绑定的逻辑交给框架完成，那么就需要告诉框架如何来完成绑定。由于JS中较难完成annotation（注解），我们可以在view中做这层标记——使用html的标签属性就是一个简单有效的办法。

~~~
function Controller(callback) {
    var models = {};
    // 找到所有有bind属性的元素
    var views = document.querySelectorAll('[bind]');
    // 将views处理为普通数组
    views = Array.prototype.slice.call(views, 0);
    views.forEach(function (view) {
        var modelName = view.getAttribute('bind');
        // 取出或新建该元素所绑定的model
        models[modelName] = models[modelName] || new Model();
        // 完成该元素和指定model的绑定
        models[modelName].bind(view);
    });
    // 调用controller的具体逻辑，将models传入，方便业务处理
    callback.call(this, models);
}
// html:
<div id="div1" bind="model1"></div>
<div id="div2" bind="model1"></div>
// 逻辑代码：
new Controller(function (models) {
    var model1 = models.model1;
    model1.set('this is a div');
});
~~~

就这么简单吗？就这么简单。MVC的本质就是在controller中完成业务逻辑，并对model进行修改，同时model的改变引起view的自动更新，这些逻辑在上面的代码中都有所体现，并且支持多个view、多个model。虽然不足以用于生产项目，但是希望对大家的MVC学习多少有些帮助。

整理后去掉注释的”框架”代码：

~~~
function Model(value) {
    this._value = typeof value === 'undefined' ? '' : value;
    this._listeners = [];
}
Model.prototype.set = function (value) {
    var self = this;
    self._value = value;
    setTimeout(function () {
        self._listeners.forEach(function (listener) {
            listener.call(self, value);
        });
    });
};
Model.prototype.watch = function (listener) {
    this._listeners.push(listener);
};
Model.prototype.bind = function (node) {
    this.watch(function (value) {
        node.innerHTML = value;
    });
};
function Controller(callback) {
    var models = {};
    var views = Array.prototype.slice.call(document.querySelectorAll('[bind]'), 0);
    views.forEach(function (view) {
        var modelName = view.getAttribute('bind');
        models[modelName] = models[modelName] || new Model();
        models[modelName].bind(view);
    });
    callback.call(this, models);
}
~~~

### 后记：

笔者在学习Flux和redux的过程中，虽然掌握了工具的使用方法，但只是知其然而不知其所以然，对ReactJS官方文档中一直强调的 “Flux eschews MVC in favor of a unidirectional data flow” 不甚理解，始终觉得单向数据流和MVC并不冲突，不明白为什么在ReactJS的文档中这二者会被对立起来，有他无我，有我无他（eschew，避开）。终于下定决心，回到MVC的定义上重新研究，虽然平日工作里大大咧咧复制粘贴，但是咱们偶尔也得任性一把，咬文嚼字一番，对吧？这样的方式也的确帮助了我对于这句话的理解，这里可以把自己的思考分享给大家：之所以觉得MVC和flux中的单向数据流相似，可能是因为没有区分清楚MVC和观察者模式的关系造成的——MVC是基于观察者模式的，flux也是，因此这种相似性的由来是观察者模式，而不是MVC和flux本身。这样的理解也在四人组的设计模式原著中得到了印证：”The first and perhaps best-known example of the Observer pattern appears in Smalltalk Model/View/Controller (MVC), the user interface framework in the Smalltalk environment [KP88]. MVC’s Model class plays the role of Subject, while View is the base class for observers. ”。

如果读者有兴趣在这样一个原型玩具的基础上继续拓展，可以参考下面的一些方向：

1. 实现对input类标签的双向绑定
2. 实现对controller所控制的scope的精准控制，这里一个controller就控制了整个dom树
3. 实现view层有关dom node隐藏/显示、创建/销毁的逻辑
4. 集成virtual dom，增加dom diff的功能，提高渲染效率
5. 提供依赖注入功能，实现控制反转
6. 对innerHTML的赋值内容进行安全检查，防止恶意注入
7. 实现model collection的逻辑，这里每个model只有一个值
8. 利用es5中的setter改变set方法的实现，使得对model的修改更加简单
9. 在view层中增加对属性和css的控制
10.支持类似AngularJS中双大括号的语法，只绑定部分html
