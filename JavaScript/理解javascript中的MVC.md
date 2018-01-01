MVC模式是软件工程中一种软件架构模式，一般把软件模式分为三部分，模型(Model)+视图(View)+控制器(Controller);

模型：模型用于封装与应用程序的业务逻辑相关的数据以及对数据处理的方法。模型有对数据直接访问的权利。模型不依赖 "视图" 和 "控制器", 也就是说 模型它不关心页面如何显示及如何被操作.

视图：视图层最主要的是监听模型层上的数据改变，并且实时的更新html页面。当然也包括一些事件的注册或者ajax请求操作(发布事件),都是放在视图层来完成。

控制器：控制器接收用户的操作，最主要是订阅视图层的事件，然后调用模型或视图去完成用户的操作;比如：当页面上触发一个事件，控制器不输出任何东西及对页面做任何处理; 它只是接收请求并决定调用模型中的那个方法去处理请求, 然后再确定调用那个视图中的方法来显示返回的数据。

下面我们来实现一个简单的下拉框控件，我们可以对它进行增删操作；如下图所示：

![](image/57c529f302ed0.png)

代码如下：

~~~
/*
 模型用于封装与应用程序的业务逻辑相关的数据以及对数据处理的方法。模型有对数据直接访问的权利。
 模型不依赖 "视图" 和 "控制器", 也就是说 模型它不关心页面如何显示及如何被操作.
 */
function Mode(elems) {
    // 所有元素
    this._elems = elems;

    // 被选中元素的索引
    this._selectedIndex = -1;

    // 增加一项
    this.itemAdd = new Event(this);

    // 删除一项
    this.itemRemoved = new Event(this);

    this.selectedIndexChanged = new Event(this);
}

Mode.prototype = {

    constructor: 'Mode',

    // 获取所有的项
    getItems: function(){
        return [].concat(this._elems);
    },
    // 增加一项
    addItem: function(elem) {
        this._elems.push(elem);
        this.itemAdd.notify({elem:elem});
    },
    // 删除一项
    removeItem: function(index) {
        var item = this._elems[index];
        this._elems.splice(index,1);
        this.itemRemoved.notify({elem:item});

        if(index === this._selectedIndex) {
            this.setSelectedIndex(-1);
        }
    },
    getSelectedIndex: function(){
        return this._selectedIndex;
    },
    setSelectedIndex: function(index){
        var previousIndex = this._selectedIndex;
        this._selectedIndex = index;
        this.selectedIndexChanged.notify({previous : previousIndex});
    }
};
/*
 下面是观察者模式类,它又叫发布---订阅模式;它定义了对象间的一种一对多的关系，
 让多个观察者对象同时监听某一个主题对象，当一个对象发生改变时，所有依赖于它的对象都将得到通知。
 */
function Event(observer) {
    this._observer = observer;
    this._listeners = [];
}
Event.prototype = {
    constaructor: 'Event',
    attach : function(listeners) {
        this._listeners.push(listeners);
    },
    notify: function(objs){
        for(var i = 0,ilen = this._listeners.length; i < ilen; i+=1) {
            this._listeners[i](this._observer,objs);
        }
    }
};

/*
 * 视图显示模型数据，并触发UI事件。
 */
function View(model,elements){
    this._model = model;
    this._elements = elements;

    this.listModified = new Event(this);
    this.addButtonClicked = new Event(this);
    this.delButtonClicked = new Event(this);
    var that = this;

    // 绑定模型监听器
    this._model.itemAdd.attach(function(){
        that.rebuildList();
    });
    this._model.itemRemoved.attach(function(){
        that.rebuildList();
    });

    // 将监听器绑定到HTML控件上
    this._elements.list.change(function(e){
        that.listModified.notify({index: e.target.selectedIndex});
    });
    // 添加按钮绑定事件
    this._elements.addButton.click(function(e){
        that.addButtonClicked.notify();
    });
    // 删除按钮绑定事件
    this._elements.delButton.click(function(e){
        that.delButtonClicked.notify();
    });
}
View.prototype = {
    constructor:  'View',
    show:  function(){
        this.rebuildList();
    },
    rebuildList: function(){
        var list = this._elements.list,
            items,
            key;
        list.html("");
        items = this._model.getItems();
        for(key in items) {
            if(items.hasOwnProperty(key)) {
                list.append('<option value="'+items[key]+'">' +items[key]+ '</option>');
            }
        }
        this._model.setSelectedIndex(-1);
    }
};
/*
 控制器响应用户操作，调用模型上的变化函数
 负责转发请求，对请求进行处理
 */
function Controller(model,view) {
    this._model = model;
    this._view = view;
    var that = this;

    this._view.listModified.attach(function(sender,args){
        that.updateSelected(args.index);
    });
    this._view.addButtonClicked.attach(function(){
        that.addItem();
    });
    this._view.delButtonClicked.attach(function(){
        that.delItem();
    });
}
Controller.prototype = {
    constructor: 'Controller',

    addItem: function(){
        var item = window.prompt('Add item:', '');
        if (item) {
            this._model.addItem(item);
        }
    },

    delItem: function(){
        var index = this._model.getSelectedIndex();
        if(index !== -1) {
            this._model.removeItem(index);
        }
    },

    updateSelected: function(index){
        this._model.setSelectedIndex(index);
    }
};
~~~


HTML代码如下：

~~~
<select id="list" size="10" style="width: 10rem"></select><br/>
<button id="plusBtn">  +  </button>
<button id="minusBtn">  -  </button>
~~~

页面初始化代码如下：

~~~
$(function () {
    var model = new Mode(['PHP', 'JavaScript']),
      view = new View(model, {
        'list' : $('#list'), 
        'addButton' : $('#plusBtn'), 
        'delButton' : $('#minusBtn')
       }),
       controller = new Controller(model, view);        
        view.show();
}); 
~~~

代码分析如下：

先分下下 我们是要实现什么样的功能; 

基本功能有：一个下拉框，通过用户输入的操作来实现用户增加一项及用户选中一项后删除一项的功能;

当然也添加了用户切换到那一项的事件;

比如我们现在来增加一条数据的时候，在视图层上添加监听事件，如下代码：
~~~
// 添加按钮绑定事件
this._elements.addButton.click(function(e){
    that.addButtonClicked.notify();
});
~~~
然后调用观察者类Event中的方法notify(发布一个事件) that.addButtonClicked.notify();大家都知道，观察者模式又叫发布-订阅模式,
让多个观察者对象同时监听某一个主题对象，当某一个主题对象发生改变的时候，所有依赖它的对象都会得到通知;
因此在控制层(Controller)我们可以使用如下代码对发布者进行监听操作：
~~~
this._view.addButtonClicked.attach(function(){
    that.addItem();
});
~~~
之后调用自身的方法addItem();代码如下：
~~~
addItem: function(){
    var item = window.prompt('Add item:', '');
    if (item) {
        this._model.addItem(item);
    }
}
~~~
调用模型层(model)的方法addItem();把一条数据插入到select框里面去;model(模型层)的addItem()方法代码如下：
~~~
// 增加一项
addItem: function(elem) {
    this._elems.push(elem);
    this.itemAdd.notify({elem:elem});
},
~~~
如上代码 增加一项后，通过 this.itemAdd 发布一个消息，然后在视图层(View)上通过如下代码来监听这个消息;代码如下：
~~~
// 绑定模型监听器
this._model.itemAdd.attach(function(){
      that.rebuildList();
});
~~~
最后监听到模型上(Model)的数据发生改变后，及时调用自身的方法rebuildList()去更新页面上的数据;

模型层(Model)最主要做业务数据封装操作。视图层(View)主要发布事件操作及监听模型层上的数据，如果模型层上有数据改变的时候，及时更新页面操作，
最后显示给页面上来，控制层(Controller)主要监听视图层(View)的事件，调用模型层(Model)的方法来更新模型上的数据，模型层数据更新后，会发布
一条消息出去，最后视图层(View)通过监听模型层(Model)的数据变化，来更新页面的显示; 如上是MVC的基本流程。

**MVC的优点：**

1. 耦合性低：视图层和业务层分离了，如果页面上显示改变的话，直接在视图层更改即可，不用动模型层和控制层上的代码;也就是视图层 与 模型层和控制层
已经分离了;所以很容易改变应用层的数据层和业务规则。
2. 可维护性：分离视图层和业务逻辑层也使得WEB应用更易于维护和修改。
MVC的缺点：
个人觉得适合于大型项目，对于中小型项目并不适合，因为要实现一个简单的增删改操作，只需要一点点JS代码，但是MVC模式代码量明显增加了。
对于学习成本也就提高了，当然如果使用一些封装好的MVC库或者框架就好了。