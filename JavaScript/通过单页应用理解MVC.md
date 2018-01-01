## 前言

之前我们为view引入了wrapperSet的概念，想以此解决view局部刷新问题，后来发现这个方案不太合理

view里面插入了业务相关的代码，事实上这个是应该剥离出去，业务的需求千奇百怪，我们不应该去处理
 
view现在只提供最基础的功能：

1. 定义各个状态的模板
3. 渲染模板

整个view的逻辑便该结束了，有一个比较特殊的情况是，当状态值不变的情况就应该是更新，这个可能会有不一样的逻辑也应该划出去
 
Adapter的意义在于存储view渲染过程中需要的data数据，从组成上分为

1. datamodel
2. viewmodel

datamodel用于具体操作，viewmodel被干掉了，提供一个getViewModel的方法替换，并且对外提供一个format方法用于用户继承

format的参数便是datamodel，这里通过处理返回的数据便是我们所谓的viewModel，他将会用于view生成对应html

然后datamodel的改变会引起对应view的变化，这个变化发起端与控制端皆在viewController，最后viewController会通知到view重新渲染
 
Controller依旧是交互的核心，他是连接Adapter以及view的桥梁

view与Adapter本身并没有关联，Controller将之联系到了一起：

1. 在实例化时候便会关联一个Adapter以及view的实例，这里Adapter不是必须的
2. viewController会保留一个view的根节点，view的根节点只会存在一个
3. viewController会在Adapter实例上监听自身，在adpter datamodel发生变化时候通知到自己，便会触发update事件
4. 传入初始状态的status以及Adapter的datamodel，调用view的render方法，会生成当前状态的view的html
5. 将之装入view的根节点，并且为viewController的this.$el赋值，create的逻辑结束
6. 触发viewController show事件，将events绑定到根节点，将$el append到container容器中并显示，初步逻辑结束
7. viewController有几个事件点用于用户注册，本身也具有很多一系列dom事件，可能导致datamodel的变化
8. 若是Adapter的datamodel发生变化便会触发dataAdpter改变的notify事件，这个时候viewController便会有所反应
9. datamodel的改变会触发viewController的update事件，默认会再次触发render事件重新新渲染结构
由于render会放给view自定义，所以其中需要执行的逻辑便不需要我们的关注了

## 实例

这个便是一个标准的MVC模型，借IOS MVC的模型来说

![image](http://images.cnitblog.com/i/294743/201405/241345585123980.png)

![image](http://images.cnitblog.com/i/294743/201405/241513110439845.png)

核心操控点在于Controller，Controller会组织model以及view

由于Controller上注册的各系列事件，会引起model的变化，

每次model产生变化后会重新通知，Controller 通知view update操作

这个时候view会获取viewController的状态值以及model的数据渲染出新的view
 
view只负责html渲染

model只负责操作数据，并且通知观察者改变事件

Controller将view以及model联系起来，所有的事件全部注册至Controller

> 传统的View会包含事件交互，这里放到了Controller上
 
模型会把datamodel的改变通知到控制器，控制器会更新视图信息，控制器根据view组成dom结构，并且注册各种UI事件，又会触发datamodel的各种改变

这就达到了理想情况的view与model的分离，一个model（adpter可用于多个viewController），一个dataAdpter的改变会影响两个视图的改变
 
这个MVC可以完全解耦view以及model，view的变化相当频繁，若是model控制view渲染便会降低model的重用
 
这里首先举一个例子做简单说明：
 
```
<!doctype html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>ToDoList</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" type="text/css" href="http://designmodo.github.io/Flat-UI/bootstrap/css/bootstrap.css">
  <link rel="stylesheet" type="text/css" href="http://designmodo.github.io/Flat-UI/css/flat-ui.css">
  <link href="../style/main.css" rel="stylesheet" type="text/css" />
  <style type="text/css">
    .cui-alert {
      width: auto;
      position: static;
    }

    .txt {
      border: #cfcfcf 1px solid;
      margin: 10px 0;
      width: 80%;
    }

    ul,
    li {
      padding: 0;
      margin: 0;
    }

    .cui_calendar,
    .cui_week {
      list-style: none;
    }

    .cui_calendar li,
    .cui_week li {
      float: left;
      width: 14%;
      overflow: hidden;
      padding: 4px 0;
      text-align: center;
    }
  </style>
</head>

<body>
  <article id="container">
  </article>
  <script type="text/underscore-template" id="template-ajax-init">
    <div class="cui-alert">
      <div class="cui-pop-box">
        <div class="cui-hd">
          <%=title%>
        </div>
        <div class="cui-bd">
          <div class="cui-error-tips">
          </div>
          <div class="cui-roller-btns" style="padding: 4px; ">
            <input type="text" placeholder="设置最低价 {day: '', price: ''}" style="margin: 2px; width: 100%; " id="ajax_data" class="txt" value="{day: , price: }">
          </div>
          <div class="cui-roller-btns">
            <div class="cui-flexbd cui-btns-sure">
              <%=confirm%>
            </div>
          </div>
        </div>
      </div>
    </div>
  </script>
  <script type="text/underscore-template" id="template-ajax-suc">
    <ul>
      <li>最低价：本月
        <%=ajaxData.day %>号，价格：
          <%=ajaxData.price %> 元</li>
    </ul>
  </script>

  <script type="text/underscore-template" id="template-ajax-loading">
    <span>loading....</span>
  </script>

  <script src="../../vendor/underscore-min.js" type="text/javascript"></script>
  <script src="../../vendor/zepto.min.js" type="text/javascript"></script>
  <script src="../../src/underscore-extend.js" type="text/javascript"></script>
  <script src="../../src/util.js" type="text/javascript"></script>
  <script src="../../src/mvc.js" type="text/javascript"></script>
  <script type="text/javascript">
    //模拟Ajax请求
    function getAjaxData(callback, data) {
      setTimeout(function () {
        if (!data) {
          data = {
            day: 3,
            price: 20
          };
        }
        callback(data);
      }, 1000);
    }

    var AjaxView = _.inherit(Dalmatian.View, {
      _initialize: function ($super) {
        //设置默认属性
        $super();

        this.templateSet = {
          init: $('#template-ajax-init').html(),
          loading: $('#template-ajax-loading').html(),
          ajaxSuc: $('#template-ajax-suc').html()
        };

      }
    });

    var AjaxAdapter = _.inherit(Dalmatian.Adapter, {
      _initialize: function ($super) {
        $super();
        this.datamodel = {
          title: '标题',
          confirm: '刷新数据'
        };
        this.datamodel.ajaxData = {};
      },

      format: function (datamodel) {
        //处理datamodel生成viewModel的逻辑
        return datamodel;
      },

      ajaxLoading: function () {
        this.notifyDataChanged();
      },

      ajaxSuc: function (data) {
        this.datamodel.ajaxData = data;
        this.notifyDataChanged();
      }
    });

    var AjaxViewController = _.inherit(Dalmatian.ViewController, {
      _initialize: function ($super) {
        $super();
        //设置基本的属性
        this.view = new AjaxView();
        this.adapter = new AjaxAdapter();
        this.viewstatus = 'init';
        this.container = '#container';
      },

      //处理datamodel变化引起的dom改变
      render: function (data) {
        //这里用户明确知道自己有没有viewdata
        var viewdata = this.adapter.getViewModel();
        var wrapperSet = {
          loading: '.cui-error-tips',
          ajaxSuc: '.cui-error-tips'
        };
        //view具有唯一包裹器
        var root = this.view.root;
        var selector = wrapperSet[this.viewstatus];

        if (selector) {
          root = root.find(selector);
        }

        this.view.render(this.viewstatus, this.adapter && this.adapter.getViewModel());

        root.html(this.view.html);

      },

      //显示后Ajax请求数据
      onViewAfterShow: function () {
        this._handleAjax();
      },

      _handleAjax: function (data) {
        this.setViewStatus('loading');
        this.adapter.ajaxLoading();
        getAjaxData($.proxy(function (data) {
          this.setViewStatus('ajaxSuc');
          this.adapter.ajaxSuc(data);
        }, this), data);
      },

      events: {
        'click .cui-btns-sure': function () {
          var data = this.$el.find('#ajax_data').val();
          data = eval('(' + data + ')');
          this._handleAjax(data);
        }
      }
    });

    var a = new AjaxViewController();
    a.show();
  </script>
</body>

</html>
```

```
(function () {
    var _ = window._;
    if (typeof require === 'function' && !_) {
        _ = require('underscore');
    };

    // @description 全局可能用到的变量
    var arr = [];
    var slice = arr.slice;

    var method = method || {};


    /**
     * @description inherit方法，js的继承，默认为两个参数
     * @param {function} supClass 可选，要继承的类
     * @param {object} subProperty 被创建类的成员
     * @return {function} 被创建的类
     */
    method.inherit = function () {

        // @description 参数检测，该继承方法，只支持一个参数创建类，或者两个参数继承类
        if (arguments.length === 0 || arguments.length > 2) throw '参数错误';

        var parent = null;

        // @description 将参数转换为数组
        var properties = slice.call(arguments);

        // @description 如果第一个参数为类（function），那么就将之取出
        if (typeof properties[0] === 'function')
            parent = properties.shift();
        properties = properties[0];

        // @description 创建新类用于返回
        function klass() {
            if (_.isFunction(this.initialize))
                this.initialize.apply(this, arguments);
        }

        klass.superclass = parent;
        // parent.subclasses = [];

        if (parent) {
            // @description 中间过渡类，防止parent的构造函数被执行
            var subclass = function () {};
            subclass.prototype = parent.prototype;
            klass.prototype = new subclass();
            // parent.subclasses.push(klass);
        }

        var ancestor = klass.superclass && klass.superclass.prototype;
        for (var k in properties) {
            var value = properties[k];

            //满足条件就重写
            if (ancestor && typeof value == 'function') {
                var argslist = /^\s*function\s*\(([^\(\)]*?)\)\s*?\{/i.exec(value.toString())[1].replace(/\s/i, '').split(',');
                //只有在第一个参数为$super情况下才需要处理（是否具有重复方法需要用户自己决定）
                if (argslist[0] === '$super' && ancestor[k]) {
                    value = (function (methodName, fn) {
                        return function () {
                            var scope = this;
                            var args = [function () {
                                return ancestor[methodName].apply(scope, arguments);
                            }];
                            return fn.apply(this, args.concat(slice.call(arguments)));
                        };
                    })(k, value);
                }
            }

            //此处对对象进行扩展，当前原型链已经存在该对象，便进行扩展
            if (_.isObject(klass.prototype[k]) && _.isObject(value) && (typeof klass.prototype[k] != 'function' && typeof value != 'fuction')) {
                //原型链是共享的，这里不好办
                var temp = {};
                _.extend(temp, klass.prototype[k]);
                _.extend(temp, value);
                klass.prototype[k] = temp;
            } else {
                klass.prototype[k] = value;
            }

        }

        if (!klass.prototype.initialize)
            klass.prototype.initialize = function () {};

        klass.prototype.constructor = klass;

        return klass;
    };

    // @description 返回需要的函数
    method.getNeedFn = function (key, scope) {
        scope = scope || window;
        if (_.isFunction(key)) return key;
        if (_.isFunction(scope[key])) return scope[key];
        return function () {};
    };

    method.callmethod = function (method, scope, params) {
        scope = scope || this;
        if (_.isFunction(method)) {
            return _.isArray(params) ? method.apply(scope, params) : method.call(scope, params);
        }

        return false;
    };

    //获取url参数
    method.getUrlParam = function (url, name) {
        var i, arrQuery, _tmp, query = {};
        var index = url.lastIndexOf('//');
        var http = url.substr(index, url.length);

        url = url.substr(0, index);
        arrQuery = url.split('&');

        for (i = 0, len = arrQuery.length; i < len; i++) {
            _tmp = arrQuery[i].split('=');
            if (i == len - 1) _tmp[1] += http;
            query[_tmp[0]] = _tmp[1];
        }

        return name ? query[name] : query;
    };


    /**
     * @description 在fn方法的前后通过键值设置两个传入的回调
     * @param fn {function} 调用的方法
     * @param beforeFnKey {string} 从context对象中获得的函数指针的键值，该函数在fn前执行
     * @param afterFnKey {string} 从context对象中获得的函数指针的键值，该函数在fn后执行
     * @param context {object} 执行环节的上下文
     * @return {function}
     */
    method.wrapmethod = method.insert = function (fn, beforeFnKey, afterFnKey, context) {

        var scope = context || this;
        var action = _.wrap(fn, function (func) {

            _.callmethod(_.getNeedFn(beforeFnKey, scope), scope);

            func.call(scope);

            _.callmethod(_.getNeedFn(afterFnKey, scope), scope);
        });

        return _.callmethod(action, scope);
    };

    method.Hash = method.inherit({
        inisilize: function (opts) {
            this.keys = [];
            this.values = [];
        },

        length: function () {
            return this.keys.length;
        },

        //传入order，若是数组中存在的话会将之放到最后，保证数组的唯一性，因为这个是hash，不能存在重复的键
        push: function (key, value, order) {
            if (_.isObject(key)) {
                for (var i in key) {
                    if (key.hasOwnProperty(i)) this.push(i, key[i], order);
                }
                return;
            }

            var index = _.indexOf(key, this.keys);

            if (index != -1 && !order) {
                this.values[index] = value;
            } else {
                if (order) this.remove(key);
                this.keys.push(key);
                this.vaules.push(value);
            }

        },

        remove: function (key) {
            return this.removeByIndex(_.indexOf(key, this.keys));
        },

        removeByIndex: function (index) {
            if (index == -1) return this;

            this.keys.splice(index, 1);
            this.values.splice(index, 1);

            return this;
        },

        pop: function () {
            if (!this.length()) return;

            this.keys.pop();
            return this.values.pop();
        },

        //根据索引返回对应键值
        indexOf: function (value) {
            var index = _.indexOf(value, this.vaules);
            if (index != -1) return this.keys[index];
            return -1;
        },

        //移出栈底值
        shift: function () {
            if (!this.length()) return;

            this.keys.shift();
            return this.values.shift();
        },

        //往栈顶压入值
        unShift: function (key, vaule, order) {
            if (_.isObject(key)) {
                for (var i in key) {
                    if (key.hasOwnProperty(i)) this.unShift(i, key[i], order);
                }
                return;
            }
            if (order) this.remove(key);
            this.keys.unshift(key);
            this.vaules.unshift(value);
        },

        //返回hash表的一段数据
        //
        slice: function (start, end) {
            var keys = this.keys.slice(start, end || null);
            var values = this.values.slice(start, end || null);
            var hash = new _.Hash();

            for (var i = 0; i < keys.length; i++) {
                hash.push(keys[i], values[i]);
            }

            return obj;
        },

        //由start开始，移除元素
        splice: function (start, count) {
            var keys = this.keys.splice(start, end || null);
            var values = this.values.splice(start, end || null);
            var hash = new _.Hash();

            for (var i = 0; i < keys.length; i++) {
                hash.push(keys[i], values[i]);
            }

            return obj;
        },

        exist: function (key, value) {
            var b = true;

            if (_.indexOf(key, this.keys) == -1) b = false;

            if (!_.isUndefined(value) && _.indexOf(value, this.values) == -1) b = false;

            return b;
        },


        filter: function () {

        }

    });

    _.extend(_, method);


    //  if (module && module.exports)
    //    module.exports = _;

}).call(this);

underscore-extend
```

```
/**
 * @description 静态日期操作类，封装系列日期操作方法
 * @description 输入时候月份自动减一，输出时候自动加一
 * @return {object} 返回操作方法
 */
var dateUtil = {
    /**
     * @description 数字操作，
     * @return {string} 返回处理后的数字
     */
    formatNum: function (n) {
        if (n < 10) return '0' + n;
        return n;
    },
    /**
     * @description 将字符串转换为日期，支持格式y-m-d ymd (y m r)以及标准的
     * @return {Date} 返回日期对象
     */
    parse: function (dateStr, formatStr) {
        if (typeof dateStr === 'undefined') return null;
        if (typeof formatStr === 'string') {
            var _d = new Date(formatStr);
            //首先取得顺序相关字符串
            var arrStr = formatStr.replace(/[^ymd]/g, '').split('');
            if (!arrStr && arrStr.length != 3) return null;

            var formatStr = formatStr.replace(/y|m|d/g, function (k) {
                switch (k) {
                    case 'y':
                        return '(\\d{4})';
                    case 'm':
                        ;
                    case 'd':
                        return '(\\d{1,2})';
                }
            });

            var reg = new RegExp(formatStr, 'g');
            var arr = reg.exec(dateStr)

            var dateObj = {};
            for (var i = 0, len = arrStr.length; i < len; i++) {
                dateObj[arrStr[i]] = arr[i + 1];
            }
            return new Date(dateObj['y'], dateObj['m'] - 1, dateObj['d']);
        }
        return null;
    },
    /**
     * @description将日期格式化为字符串
     * @return {string} 常用格式化字符串
     */
    format: function (date, format) {
        if (arguments.length < 2 && !date.getTime) {
            format = date;
            date = new Date();
        }
        typeof format != 'string' && (format = 'Y年M月D日 H时F分S秒');
        return format.replace(/Y|y|M|m|D|d|H|h|F|f|S|s/g, function (a) {
            switch (a) {
                case "y":
                    return (date.getFullYear() + "").slice(2);
                case "Y":
                    return date.getFullYear();
                case "m":
                    return date.getMonth() + 1;
                case "M":
                    return dateUtil.formatNum(date.getMonth() + 1);
                case "d":
                    return date.getDate();
                case "D":
                    return dateUtil.formatNum(date.getDate());
                case "h":
                    return date.getHours();
                case "H":
                    return dateUtil.formatNum(date.getHours());
                case "f":
                    return date.getMinutes();
                case "F":
                    return dateUtil.formatNum(date.getMinutes());
                case "s":
                    return date.getSeconds();
                case "S":
                    return dateUtil.formatNum(date.getSeconds());
            }
        });
    },
    // @description 是否为为日期对象，该方法可能有坑，使用需要慎重
    // @param year {num} 日期对象
    // @return {boolean} 返回值
    isDate: function (d) {
        if ((typeof d == 'object') && (d instanceof Date)) return true;
        return false;
    },
    // @description 是否为闰年
    // @param year {num} 可能是年份或者为一个date时间
    // @return {boolean} 返回值
    isLeapYear: function (year) {
        //传入为时间格式需要处理
        if (dateUtil.isDate(year)) year = year.getFullYear()
        if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)) return true;
        else return false;
    },

    // @description 获取一个月份的天数
    // @param year {num} 可能是年份或者为一个date时间
    // @param year {num} 月份
    // @return {num} 返回天数
    getDaysOfMonth: function (year, month) {
        //自动减一以便操作
        month--;
        if (dateUtil.isDate(year)) {
            month = year.getMonth(); //注意此处月份要加1，所以我们要减一
            year = year.getFullYear();
        }
        return [31, dateUtil.isLeapYear(year) ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31][month];
    },

    // @description 获取一个月份1号是星期几，注意此时的月份传入时需要自主减一
    // @param year {num} 可能是年份或者为一个date时间
    // @param year {num} 月份
    // @return {num} 当月一号为星期几0-6
    getBeginDayOfMouth: function (year, month) {
        //自动减一以便操作
        month--;
        if ((typeof year == 'object') && (year instanceof Date)) {
            month = year.getMonth();
            year = year.getFullYear();
        }
        var d = new Date(year, month, 1);
        return d.getDay();
    }
};

Util
```

```
"use strict";

// ------------------华丽的分割线--------------------- //

// @description 正式的声明Dalmatian框架的命名空间
var Dalmatian = Dalmatian || {};

// @description 定义默认的template方法来自于underscore
Dalmatian.template = _.template;
Dalmatian.View = _.inherit({
    // @description 构造函数入口
    initialize: function (options) {
        this._initialize();
        this.handleOptions(options);
        this._initRoot();

    },

    _initRoot: function () {
        //根据html生成的dom包装对象
        //有一种场景是用户的view本身就是一个只有一个包裹器的结构，他不想要多余的包裹器
        this.root = $(this.defaultContainerTemplate);
        this.root.attr('id', this.viewid);
    },

    // @description 设置默认属性
    _initialize: function () {

        var DEFAULT_CONTAINER_TEMPLATE = '<section class="view" id="<%=viewid%>"><%=html%></section>';

        // @description view状态机
        // this.statusSet = {};

        this.defaultContainerTemplate = DEFAULT_CONTAINER_TEMPLATE;

        // @override
        // @description template集合，根据status做template的map
        // @example
        //    { 0: '<ul><%_.each(list, function(item){%><li><%=item.name%></li><%});%></ul>' }
        // this.templateSet = {};

        this.viewid = _.uniqueId('dalmatian-view-');

    },

    // @description 操作构造函数传入操作
    handleOptions: function (options) {
        // @description 从形参中获取key和value绑定在this上
        if (_.isObject(options)) _.extend(this, options);

    },

    // @description 通过模板和数据渲染具体的View
    // @param status {enum} View的状态参数
    // @param data {object} 匹配View的数据格式的具体数据
    // @param callback {functiion} 执行完成之后的回调
    render: function (status, data, callback) {

        var templateSelected = this.templateSet[status];
        if (templateSelected) {

            // @description 渲染view
            var templateFn = Dalmatian.template(templateSelected);
            this.html = templateFn(data);

            //这里减少一次js编译
            //      this.root.html('');
            //      this.root.append(this.html);

            this.currentStatus = status;

            _.callmethod(callback, this);

            return this.html;

        }
    },

    // @override
    // @description 可以被复写，当status和data分别发生变化时候
    // @param status {enum} view的状态值
    // @param data {object} viewmodel的数据
    update: function (status, data) {

        if (!this.currentStatus || this.currentStatus !== status) {
            return this.render(status, data);
        }

        // @override
        // @description 可复写部分，当数据发生变化但是状态没有发生变化时，页面仅仅变化的可以是局部显示
        //              可以通过获取this.html进行修改
        _.callmethod(this.onUpdate, this, data);
    }
});

Dalmatian.Adapter = _.inherit({

    // @description 构造函数入口
    initialize: function (options) {
        this._initialize();
        this.handleOptions(options);

    },

    // @description 设置默认属性
    _initialize: function () {
        this.observers = [];
        //    this.viewmodel = {};
        this.datamodel = {};
    },

    // @description 操作构造函数传入操作
    handleOptions: function (options) {
        // @description 从形参中获取key和value绑定在this上
        if (_.isObject(options)) _.extend(this, options);
    },

    // @override
    // @description 设置
    format: function (datamodel) {
        return datamodel;
    },

    getViewModel: function () {
        return this.format(this.datamodel);
    },

    registerObserver: function (viewcontroller) {
        // @description 检查队列中如果没有viewcontroller，从队列尾部推入
        if (!_.contains(this.observers, viewcontroller)) {
            this.observers.push(viewcontroller);
        }
    },

    unregisterObserver: function (viewcontroller) {
        // @description 从observers的队列中剔除viewcontroller
        this.observers = _.without(this.observers, viewcontroller);
    },

    //统一设置所有观察者的状态，因为对应观察者也许根本不具备相关状态，所以这里需要处理
    //  setStatus: function (status) {
    //    _.each(this.observers, function (viewcontroller) {
    //      if (_.isObject(viewcontroller))
    //        viewcontroller.setViewStatus(status);
    //    });
    //  },

    notifyDataChanged: function () {
        // @description 通知所有注册的观察者被观察者的数据发生变化
        var data = this.getViewModel();
        _.each(this.observers, function (viewcontroller) {
            if (_.isObject(viewcontroller))
                _.callmethod(viewcontroller.update, viewcontroller, [data]);
        });
    }
});

Dalmatian.ViewController = _.inherit({

    _initialize: function () {

        //用户设置的容器选择器，或者dom结构
        this.container;
        //根元素
        this.$el;

        //一定会出现
        this.view;
        //可能会出现
        this.adapter;
        //初始化的时候便需要设置view的状态，否则会渲染失败，这里给一个默认值
        this.viewstatus = 'init';

    },

    // @description 构造函数入口
    initialize: function (options) {
        this._initialize();
        this.handleOptions(options);
        this._handleAdapter();
        this.create();
    },

    //处理dataAdpter中的datamodel，为其注入view的默认容器数据
    _handleAdapter: function () {
        //不存在就不予理睬
        if (!this.adapter) return;
        this.adapter.registerObserver(this);
    },

    // @description 操作构造函数传入操作
    handleOptions: function (options) {
        if (!options) return;

        this._verify(options);

        // @description 从形参中获取key和value绑定在this上
        if (_.isObject(options)) _.extend(this, options);
    },

    setViewStatus: function (status) {
        this.viewstatus = status;
    },

    // @description 验证参数
    _verify: function (options) {
        //这个underscore方法新框架在报错
        //    if (!_.property('view')(options) && (!this.view)) throw Error('view必须在实例化的时候传入ViewController');
        if (options.view && (!this.view)) throw Error('view必须在实例化的时候传入ViewController');
    },

    // @description 当数据发生变化时调用onViewUpdate，如果onViewUpdate方法不存在的话，直接调用render方法重绘
    update: function (data) {

        //    _.callmethod(this.hide, this);

        if (this.onViewUpdate) {
            _.callmethod(this.onViewUpdate, this, [data]);
            return;
        }
        this.render();

        //    _.callmethod(this.show, this);
    },

    /**
     * @override
     */
    render: function () {
        // @notation  这个方法需要被复写
        this.view.render(this.viewstatus, this.adapter && this.adapter.getViewModel());
        this.view.root.html(this.view.html);
    },

    _create: function () {
        this.render();

        //render 结束后构建好根元素dom结构
        this.view.root.html(this.view.html);
        this.$el = this.view.root;
    },

    create: function () {

        //l_wang这块不是很明白
        //是否检查映射关系，不存在则recreate，但是在这里dom结构未必在document上
        //    if (!$('#' + this.view.viewid)[0]) {
        //      return _.callmethod(this.recreate, this);
        //    }

        // @notation 在create方法调用前后设置onViewBeforeCreate和onViewAfterCreate两个回调
        _.wrapmethod(this._create, 'onViewBeforeCreate', 'onViewAfterCreate', this);

    },

    /**
     * @description 如果进入create判断是否需要update一下页面，sync view和viewcontroller的数据
     */
    _recreate: function () {
        this.update();
    },

    recreate: function () {
        _.wrapmethod(this._recreate, 'onViewBeforeRecreate', 'onViewAfterRecreate', this);
    },

    //事件注册点
    bindEvents: function (events) {
        if (!(events || (events = _.result(this, 'events')))) return this;
        this.unBindEvents();

        // @description 解析event参数的正则
        var delegateEventSplitter = /^(\S+)\s*(.*)$/;
        var key, method, match, eventName, selector;

        //注意，此处做简单的字符串数据解析即可，不做实际业务
        for (key in events) {
            method = events[key];
            if (!_.isFunction(method)) method = this[events[key]];
            if (!method) continue;

            match = key.match(delegateEventSplitter);
            eventName = match[1], selector = match[2];
            method = _.bind(method, this);
            eventName += '.delegateEvents' + this.view.viewid;

            if (selector === '') {
                this.$el.on(eventName, method);
            } else {
                this.$el.on(eventName, selector, method);
            }
        }

        return this;
    },

    //取消所有事件
    unBindEvents: function () {
        this.$el.off('.delegateEvents' + this.view.viewid);
        return this;
    },

    _show: function () {
        this.bindEvents();
        $(this.container).append(this.$el);
        this.$el.show();
    },

    show: function () {
        _.wrapmethod(this._show, 'onViewBeforeShow', 'onViewAfterShow', this);
    },

    _hide: function () {
        this.forze();
        this.$el.hide();
    },

    hide: function () {
        _.wrapmethod(this._hide, 'onViewBeforeHide', 'onViewAfterHide', this);
    },

    _forze: function () {
        this.unBindEvents();
    },

    forze: function () {
        _.wrapmethod(this._forze, 'onViewBeforeForzen', 'onViewAfterForzen', this);
    },

    _destory: function () {
        this.unBindEvents();
        this.$el.remove();
        //    delete this;
    },

    destory: function () {
        _.wrapmethod(this._destory, 'onViewBeforeDestory', 'onViewAfterDestory', this);
    }
});
```

完整MVC想法

代码效果如下：

![image](http://images.cnitblog.com/i/294743/201405/241440095744761.png)

![image](http://images.cnitblog.com/i/294743/201405/241440129962416.png)

每次点击刷新数据时候会模拟一次Ajax操作，将datamodel中的数据改变，然后会触发视图改变

```
events: {
    'click .cui-btns-sure': function () {
        var data = this.$el.find('#ajax_data').val();
        data = eval('(' + data + ')');
        this._handleAjax(data);
    }
}
```

```
_handleAjax: function (data) {
    this.setViewStatus('loading');
    this.adapter.ajaxLoading();
    getAjaxData($.proxy(function (data) {
        this.setViewStatus('ajaxSuc');
        this.adapter.ajaxSuc(data);
    }, this), data);
},
```

```
ajaxSuc: function (data) {
    this.datamodel.ajaxData = data;
    this.notifyDataChanged();
}
```

中间又一次状态变化，将视图变为loading状态，一次数据请求成功，我们要做的是，重写viewController的render方法

```
render: function (data) {
    //这里用户明确知道自己有没有viewdata
    //var viewdata = this.adapter.getViewModel();
    var wrapperSet = {
        loading: '.cui-error-tips',
        ajaxSuc: '.cui-error-tips'
    };
    //view具有唯一包裹器
    var root = this.view.root;
    var selector = wrapperSet[this.viewstatus];

    if (selector) {
        root = root.find(selector);
    }

    this.view.render(this.viewstatus, this.adapter && this.adapter.getViewModel());

    root.html(this.view.html);

},
```

这块逻辑需要被用户重写，因为具体每次渲染后，形成的html装载在什么位置，我们并不能确定

这里我们再写一个例子，看一看共享一个Adapter的效果

```
<!doctype html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>ToDoList</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" type="text/css" href="http://designmodo.github.io/Flat-UI/bootstrap/css/bootstrap.css">
  <link rel="stylesheet" type="text/css" href="http://designmodo.github.io/Flat-UI/css/flat-ui.css">
</head>

<body>
  <article class="container">
  </article>

  <div style=" border: 1px solid black; margin: 10px; padding: 10px; ">上下组件共享Adapter</div>

  <article class="list">
  </article>

  <script type="text/underscore-template" id="template-todolist">
    <section class="row">
      <div class="col-xs-9">
        <form action="">
          <legend>To Do List -- Input</legend>
          <input type="text" placeholer="ToDoList" id="todoinput">
          <button class="btn btn-primary" data-action="add">添加</button>
        </form>
        <ul id="todolist">
          <%_.each(list, function(item){%>
            <li>
              <%=item.content %>
            </li>
            <%})%>
        </ul>
      </div>
    </section>
  </script>

  <script type="text/underscore-template" id="template-list">
    <ul>
      <%for(var i =0, len = list.length; i < len; i++) {%>
        <li>
          <%=list[i].content %> -
            <span index="<%=i %>">删除</span>
        </li>
        <%}%>
    </ul>
  </script>

  <script type="text/javascript" src="../../vendor/underscore.js"></script>
  <script type="text/javascript" src="../../vendor/zepto.js"></script>
  <script src="../../src/underscore-extend.js" type="text/javascript"></script>
  <script src="../../src/mvc.js" type="text/javascript"></script>
  <script type="text/javascript">
    var View = _.inherit(Dalmatian.View, {
      _initialize: function ($super) {
        //设置默认属性
        $super();
        this.templateSet = {
          init: $('#template-todolist').html()
        };
      }
    });

    var Adapter = _.inherit(Dalmatian.Adapter, {
      _initialize: function ($super) {
        $super();
        this.datamodel = {
          list: [{
            content: '测试数据01'
          }, {
            content: '测试数据02'
          }]
        };
      },

      addItem: function (item) {
        this.datamodel.list.push(item);
        this.notifyDataChanged();
      },

      removeItem: function (index) {
        this.datamodel.list.splice(index, 1);
        this.notifyDataChanged();
      }

    });

    var adapter = new Adapter();

    var Controller = _.inherit(Dalmatian.ViewController, {
      _initialize: function ($super) {
        $super();
        this.view = new View();
        this.adapter = adapter;
        this.container = '.container';
      },

      render: function () {
        this.view.render(this.viewstatus, this.adapter && this.adapter.getViewModel());
        this.view.root.html(this.view.html);
      },

      events: {
        'click button': 'action'
      },
      action: function (e) {
        e.preventDefault();
        var target = $(e.currentTarget).attr('data-action');
        var strategy = {
          'add': function (e) {
            var value = $('#todoinput').val();
            this.adapter.addItem({
              content: value
            });
            var s = '';
          }
        }
        strategy[target].apply(this, [e]);
      }
    });

    var controller = new Controller();
    controller.show();


    var LView = _.inherit(Dalmatian.View, {
      _initialize: function ($super) {
        //设置默认属性
        $super();
        this.templateSet = {
          init: $('#template-list').html()
        };
      }
    });

    var LController = _.inherit(Dalmatian.ViewController, {
      _initialize: function ($super) {
        $super();
        this.view = new LView();
        this.adapter = adapter;
        this.container = '.list';
      },

      render: function () {
        this.view.render(this.viewstatus, this.adapter && this.adapter.getViewModel());
        this.view.root.html(this.view.html);
      },

      events: {
        'click span': 'action'
      },
      action: function (e) {
        var index = $(e.currentTarget).attr('index');
        this.adapter.removeItem(index);

      }
    });

    var lcontroller = new LController();
    lcontroller.show();
  </script>
</body>

</html>
```

![image](http://images.cnitblog.com/i/294743/201405/241450320276598.png)

![image](http://images.cnitblog.com/i/294743/201405/241450358569896.png)

可以看到，上下的变化根源是数据操作，每次数据的变化是共享的

## 结语

今天更正了上一次留下来的wrapperSet思维，这里对自己所知的MVC做了一次梳理

然后框架在一次次修改中逐步成型了，是个好现象，慢慢来吧