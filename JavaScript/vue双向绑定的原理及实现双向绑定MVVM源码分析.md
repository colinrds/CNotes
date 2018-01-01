[toc]

双向数据绑定的原理是：可以将对象的属性绑定到UI，具体的说，我们有一个对象，该对象有一个name属性，当我们给这个对象name属性赋新值的时候，新值在UI上也会得到更新。同样的道理，当我们有一个输入框或者textarea的时候，我们输入一个新值的时候，也会在该对象的name属性得到更新。

实现数据绑定的做法有如下几种：

1. 发布者-订阅模式（http://www.cnblogs.com/tugenhua0707/p/7471381.html）
2. 脏值检查(angular.js)
3. 数据劫持(vue.js)

- 脏值检查：是通过脏值检测的方式比对数据是否有变更，来决定是否更新视图，最简单的方式就是通过 setInterval()定时轮询检测数据的变动。
- 数据劫持：vue.js 则是采用数据劫持结合发布者-订阅者模式，通过Object.defineProperty()来劫持各个属性的setter，getter。在数据变动时发布消息给订阅者，触发响应的监听回调。

下面是一个通过 Object.defineProperty 来实现一个简单的数据双向绑定。通过该方法来监听属性的变化。
实现的效果简单如下：页面上有一个input输入框和div显示框，当在input输入框输入值的时候，div也会显示对应的值，当我打开控制台改变 obj.name="输入任意值"的时候，按回车键运行下，input输入框的值也会跟着变，可以简单的理解为 模型-> 视图的 改变，以及 视图 -> 模型的改变。如下代码：

```
<!DOCTYPE html>
 <html>
  <head>
    <meta charset="utf-8">
    <meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
    <meta content="yes" name="apple-mobile-web-app-capable">
    <meta content="black" name="apple-mobile-web-app-status-bar-style">
    <meta content="telephone=no" name="format-detection">
    <meta content="email=no" name="format-detection">
    <title>标题</title>
    <link rel="shortcut icon" href="/favicon.ico">
  </head>
  <body>
    <h3>使用Object.defineProperty实现简单的双向数据绑定</h3>
    <input type="text" id="input" />
    <div id="div"></div>
    <script>
        var obj = {};
        var inputVal = document.getElementById("input");
        var div = document.getElementById("div");

        Object.defineProperty(obj, "name", {
          set: function(newVal) {
            inputVal.value = newVal;
            div.innerHTML = newVal;
          }
        });
        inputVal.addEventListener('input', function(e){
          obj.name = e.target.value;
        });
    </script>
  </body>
</html>
```

vue是通过数据劫持的方式来做数据绑定的，最核心的方法是通过 Object.defineProperty()方法来实现对属性的劫持，达到能监听到数据的变动。要实现数据的双向绑定，需要实现如下几点：

1. 需要实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动拿到最新值并通知订阅者。
2. 需要实现一个指令解析器Compile, 对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
3. 需要实现一个Watcher，作为链接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应的回调函数，从而更新视图。

## 一： 实现Observer

我们可以使用Object.defineProperty()来监听属性的变动，我们需要对属性对象进行递归遍历，包括子属性对象的属性。再加上getter和setter方法，我们可以和上面的demo一样，监听input值的变化，即 视图 -> 模型。且当我们给某个对象的属性赋值的话，会自动调用setter方法，来动态修改数据，即 模型 -> 视图。
引入我们可以使用 object.defineProperty来实现监听属性的变动，如下代码：

```
<!DOCTYPE html>
 <html>
  <head>
    <meta charset="utf-8">
    <meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
    <meta content="yes" name="apple-mobile-web-app-capable">
    <meta content="black" name="apple-mobile-web-app-status-bar-style">
    <meta content="telephone=no" name="format-detection">
    <meta content="email=no" name="format-detection">
    <title>标题</title>
    <link rel="shortcut icon" href="/favicon.ico">
  </head>
  <body>
    <script>
      function Observer(data) {
        this.data = data;
        this.init();
      }
      Observer.prototype = {
        init: function() {
          var self = this;
          var data = self.data;
          // 遍历data对象
          Object.keys(data).forEach(function(key) {
            self.defineReactive(data, key, data[key]);
          });
        },
        defineReactive: function(data, key, value) {

          // 递归遍历子对象
          var childObj = observer(value);

          // 对对象的属性使用Object.defineProperty进行监听
          Object.defineProperty(data, key, {
            enumerable: true,  // 可枚举
            configurable: false, // 不能删除目标属性或不能修改目标属性
            get: function() {
              return value;
            },
            set: function(newVal) {
              if (newVal === value) {
                return;
              }
              console.log('已经监听到值的变化了', value, '==>', newVal);
              value = newVal;
            }
          });
        }
      }

      function observer(value) {
        if (!value || typeof value !== 'object') {
          return;
        }
        return new Observer(value);
      }

      // 测试demo
      var data = {name: 'kongzhi'};
      observer(data);
      data.name = 'kongzhi2';  // 控制台打印出  已经监听到值的变化了，kongzhi ==> kongzhi2
    </script>
  </body>
</html>
```

如上代码我们可以监听每个属性对象数据的变化了，那么监听到属性值变化后我们需要把消息通知到订阅者，因此我们需要实现一个消息订阅器，该订阅器的作用是收集所有的订阅者，当有属性值发生改变的时候，就把该消息通知给所有订阅者。因此我们可以实现如下代码：

```
// 对所有的属性数据进行监听
function Observer(data) {
  this.data = data;
  this.init();
}

Observer.prototype = {
  init: function() {
    var self = this;
    var data = self.data;
    // 遍历data对象
    Object.keys(data).forEach(function(key) {
      self.defineReactive(data, key, data[key]);
    });
  },
  defineReactive: function(data, key, value) {
    var dep = new Dep();
    // 递归遍历子对象
    var childObj = observer(value);

    // 对对象的属性使用Object.defineProperty进行监听
    Object.defineProperty(data, key, {
      enumerable: true,  // 可枚举
      configurable: false, // 不能删除目标属性或不能修改目标属性
      get: function() {
        if (Dep.target) {
          dep.depend();
        }
        return value;
      },
      set: function(newVal) {
        if (newVal === value) {
          return;
        }
        value = newVal;
        // 如果新值是对象的话，递归该对象 进行监听
        childObj = observer(newVal);
        // 通知订阅者 
        dep.notify();
      }
    });
  }
}

function observer(value) {
  if (!value || typeof value !== 'object') {
    return;
  }
  return new Observer(value);
}

function Dep() {
  this.subs = [];
}

Dep.prototype = {
  addSub: function(sub) {
    this.subs.push(sub);
  },
  depend: function() {
    Dep.target.addDep(this);
  },
  removeSub: function(sub) {
    var index = this.subs.indexOf(sub);
    if (index != -1) {
      this.subs.splice(index, 1);
    }
  },
  notify: function() {
    // 遍历所有的订阅者 通知所有的订阅者
    this.subs.forEach(function(sub) {
      sub.update();
    })
  }
};

Dep.target = null;
```

从上面的分析得到，需要实现一个Watcher，作为链接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，所以订阅者就是Watcher，var dep = new Dep();是在 defineReactive方法内部定义的，是想通过dep添加订阅者，通过Dep定义一个全局的target属性，暂存watcher，添加完后会移除，因此最后一句代码设置 Dep.target = null;
执行getter方法会返回值，执行setter方法会判断新旧值是否相等，如果不相等的话，再次递归遍历设置的对象的所有子对象，然后会通知所有的订阅者。

## 二： 实现 Compile

compile做的事情是解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数。

如下图所示：

![image](http://ow2n75eab.bkt.clouddn.com/561794-20170925003548306-218462638.png)


```
// 具体看如下代码，代码中有对应的注释

/*
 * @param {el} 元素节点容器 如：el: '#mvvm-app'
 * @param {vm} Object 对象传递数据 
*/
function Compile(el, vm) {
  this.$vm = vm;
  // 判断是元素节点 还是 选择器
  this.$el = this.isElementNode(el) ? el : document.querySelector(el);
  if (this.$el) {
    /*
     因为遍历解析过程有多次操作dom节点，为了提高性能和效率，会先将根节点el转换成文档碎片fragment进行解析编译操作，
     解析完成后，再将fragment添加回原来真实的dom节点中。有关 DocumentFragment 请看 http://www.cnblogs.com/tugenhua0707/p/7465915.html
     */
    this.$fragment = this.node2Fragment(this.$el);
    this.init();
    this.$el.appendChild(this.$fragment);
  }
}

Compile.prototype = {
  node2Fragment: function(el) {
    var fragment = document.createDocumentFragment(),
      child;
    // 将原生节点拷贝到fragment中
    while (child = el.firstChild) {
      fragment.appendChild(child);
    }
    return fragment;
  },
  init: function() {
    // 进行解析编译操作
    this.compileElement(this.$fragment);
  },
  compileElement: function(el) {
    var childNodes = el.childNodes;
    // 遍历所有的子节点 判断子节点是 元素节点还是文本节点 分别进行编译解析操作
    [].slice.call(childNodes).forEach(function(node) {

      // 获取节点的文本内容和它的所有后代
      var text = node.textContent;

      // 正则匹配 {{xx}} 这样的xx文本值
      var reg = /\{\{(.*)\}\}/;
      // 判断是否是元素节点，然后进行编译
      if (this.isElementNode(node)) {
        this.compile(node);

      } else if(this.isTextNode(node) && reg.test(text)) {
        // 判断node是否是文本节点 且 符合正则匹配的 {{xx}} 那么就会进行编译解析
        this.compileText(node, RegExp.$1);
      }

      // 如果该节点还有子节点的话，那么递归进行判断编译
      if (node.childNodes && node.childNodes.length) {
        this.compileElement(node);
      }
    });
  },
  // 元素节点编译
  compile: function(node) {
    // 获取节点的所有属性
    var nodeAttrs = node.attributes;
    var self = this;
    /*
     遍历该节点的所有属性，判断该属性是事件指令还是普通指令
     */
    [].slice.call(nodeAttrs).forEach(function(attr) {
      // 获取属性名
      var attrName = attr.name;
      // 先判断属性名是否是 以 v- 开头的
      if (self.isDirective(attrName)) {
        var attrValue = attr.value;
        // 获取 v-xx 中从xx开始的所有字符串
        var dir = attrName.substring(2);
        // 判断是否是事件指令
        if (self.isEventDirective(dir)) {
          compileUtil.eventHandler(node, self.$vm, attrValue, dir);
        } else {
          // 普通指令
          compileUtil[dir] && compileUtil[dir](node, self.$vm, attrValue);
        }
        // 循环完成一次后 删除该属性
        node.removeAttribute(attrName);
      }
    });
  },
  // 编译文本
  compileText: function(node, exp) {
    compileUtil.text(node, this.$vm, exp);
  },
  // 是否是v- 开始的指令
  isDirective: function(attrName) {
    return attrName.indexOf('v-') === 0;
  },
  /*
   * 是否是事件指令 事件指令以 v-on开头的
   */
   isEventDirective: function(dir) {
     return dir.indexOf('on') === 0;
   },
   // 是否是元素节点
   isElementNode: function(node) {
     return node.nodeType === 1;
   },
   // 是否是文本节点
   isTextNode: function(node) {
     return node.nodeType === 3;
   }
};

// 指令处理
var compileUtil = {
  text: function(node, vm, exp) {
    this.bind(node, vm, exp, 'text');
  },
  html: function(node, vm, exp) {
    this.bind(node, vm, exp, 'html');
  },
  /*
   * 普通指令 v-model 开头的，调用model方法
   * @param {node} 容器节点
   * @param {vm} 数据对象
   * @param {exp} 普通指令的值 比如 v-model="xx" 那么exp就等于xx
   */
  model: function(node, vm, exp) {
    this.bind(node, vm, exp, 'model');
    var self = this;
    var val = this.getVMVal(vm, exp);
    // 监听input的事件
    node.addEventListener('input', function(e) {
      var newValue = e.target.value;
      // 比较新旧值是否相同
      if (val === newValue) {
        return;
      }
      // 重新设置新值
      self.setVMVal(vm, exp, newValue);
      val = newValue;
    });
  },
  /*
   *  返回 v-mdoel = "xx" 中的xx的值， 比如data对象会定义如下：
    data: {
      "xx" : "111"
    }
    * @param {vm} 数据对象
    * @param {exp} 普通指令的值 比如 v-model="xx" 那么exp就等于xx
   */
  getVMVal: function(vm, exp) {
    var val = vm;
    exp = exp.split('.');
    exp.forEach(function(k) {
      val = val[k];
    });
    return val;
  },
  /*
    设置普通指令的值
    @param {vm} 数据对象
    @param {exp} 普通指令的值 比如 v-model="xx" 那么exp就等于xx
    @param {value} 新值
   */
  setVMVal: function(vm, exp, value) {
    var val = vm;
    exp = exp.split('.');
    exp.forEach(function(key, index) {
      // 如果不是最后一个元素的话，更新值
      /*
       数据对象 data 如下数据
       data: {
        child: {
          someStr: 'World !'
        }
      },
       如果 v-model="child.someStr" 那么 exp = ["child", "someStr"], 遍历该数组，
       val = val["child"]; val 先储存该对象，然后再继续遍历 someStr，会执行else语句，因此val['someStr'] = value, 就会更新到对象的值了。
       */
      if (i < exp.length - 1) {
        val = val[key];
      } else {
        val[key] = value;
      }
    });
  },
  class: function(node, vm, exp) {
    this.bind(node, vm, exp, 'class');
  },
  /* 
   事件处理
   @param {node} 元素节点
   @param {vm} 数据对象
   @param {attrValue} attrValue 属性值
   @param {dir} 事件指令的值 比如 v-on:click="xx" 那么dir就等 on:click
   */
  eventHandler: function(node, vm, attrValue, dir) {
    // 获取事件类型 比如dir=on:click 因此 eventType="click" 
    var eventType = dir.split(':')[1];
    /*
     * 获取事件的函数 比如 v-on:click="clickBtn" 那么就会对应vm里面的clickBtn 函数。
     * 比如 methods: {
        clickBtn: function(e) {
          console.log(11)
        }
      }
     */
    var fn = vm.$options.methods && vm.$options.methods[attrValue];
    if (eventType && fn) {
      // 如果有事件类型和函数的话，就绑定该事件
      node.addEventListener(eventType, fn.bind(vm), false);
    }
  },
  /*
   @param {node} 节点
   @param {vm} 数据对象
   @param {exp} 正则匹配的值
   @param {dir} 字符串类型 比如 'text', 'html' 'model'
   */
  bind: function(node, vm, exp, dir) {
    // 获取updater 对象内的对应的函数
    var updaterFn = updater[dir + 'Updater'];

    // 如有该函数的话就执行该函数 参数node为节点，第二个参数为 指令的值。 比如 v-model = 'xx' 那么返回的就是xx的值
    updaterFn && updaterFn(node, this.getVMVal(vm, exp));

    // 调用订阅者watcher
    new Watcher(vm, exp, function(newValue, oldValue) {
      updaterFn && updaterFn(node, newValue, oldValue);
    });
  }
};

var updater = {
  textUpdater: function(node, value) {
    node.textContent = typeof value === 'undefined' ? '' : value;
  },
  htmlUpdater: function(node, value) {
    node.innerHTML = typeof value === 'undefined' ? '' : value;
  },
  classUpdater: function(node, newValue, oldValue) {
    var className = node.className;
    className = className.replace(oldValue, '').replace(/\s$/, '');
    var space = className && String(newValue) ? ' ' : '';
    node.className = className + space + newValue;
  },
  modelUpdater: function(node, newValue) {
    node.value = typeof newValue === 'undefined' ? '' : newValue;
  }
};
```

## 三： 实现Watcher

Watcher是订阅者是作为 Observer和Compile之间通信的桥梁。
相关代码如下：

```
/*
 * @param {vm} 数据对象
 * @param {expOrFn} 属性值 比如 v-model='xx' v-on:click='yy' v-text="tt" 中的 xx, yy, tt 
 * @param {cb}  回调函数
 */
function Watcher(vm, expOrFn, cb) {
  this.vm = vm;
  this.expOrFn = expOrFn;
  this.cb = cb;
  this.depIds = {};

  // expOrFn 是事件函数的话
  if (typeof expOrFn === 'function') {
    this.getter = expOrFn;
  } else {
    this.getter = this.parseGetter(expOrFn);
  }
  // 为了触发属性getter，从而在dep添加自己作为订阅者
  this.value = this.get();
}

Watcher.prototype = {
  update: function() {
    this.run();    // observer 中的属性值发生变化 收到通知
  },
  run: function() {
    var value = this.get();
    var oldValue = this.value;
    // 新旧值不相等的话
    if (value !== oldValue) {
      // 把当前的值赋值给 this.value 更新this.value的值
      this.value = value;
      this.cb.call(this.vm, value, oldValue);  // 执行Compile中绑定的回调 更新视图
    }
  },
  get: function() {
    Dep.target = this; // 将当前订阅者指向自己
    var value = this.getter.call(this.vm, this.vm); // 触发getter，添加自己到属性订阅器中
    Dep.target = null;  // 添加完成后 清空数据
    return value;
  },
  parseGetter: function(exp) {
    var reg = /[^\w.$]/;
    if (reg.test(exp)) {
      return;
    }
    var exps = exp.split('.');
    return function(obj) {
      for(var i = 0, len = exps.length; i < len; i++) {
        if (!obj) {
          return;
        }
        obj = obj[exps[i]];
      }
      return obj;
    }
  }
}
```

## 四： 实现MVVM

MVVM是数据绑定的入口，整合 Observer， Compile， 和 Watcher，通过Observer来监听model数据变化，通过Compile来解析编译模板指令，最后使用watcher搭起Observer和Compile之间的通信桥梁。 达到数据变化 --》视图更新，视图变化 --》 数据model更新的 双向绑定效果。
代码如下：

```
function MVVM(options) {
  this.$options = options || {};
  var data = this._data = this.$options.data;
  var self = this;

  /*
   数据代理，实现 vm.xxx 
   上面的代码看出 监听数据的对象是 options.data, 因此每次更新视图的时候；如：
   var vm = new MVVM({
     data: {name: 'kongzhi'}
   });
   那么更新数据就变成 vm._data.name = 'kongzhi2'; 但是我们想实现这样更改 vm.name = "kongzhi2";
   因此这边需要使用属性代理方式，利用Object.defineProperty()方法来劫持vm实列对象属性的读写权，使读写vm实列的属性转成vm.name的属性值。
   */
   Object.keys(data).forEach(function(key) {
     self._proxyData(key);
   });

   this._initComputed();

   // 初始化Observer
   observer(data, this);

   // 初始化 Compile
   this.$compile = new Compile(options.el || document.body, this);
}

MVVM.prototype = {
  $watch: function(key, cb, options) {
    new Watcher(this, key, cb);
  },

  _proxyData: function(key) {
    var self = this;
    Object.defineProperty(self, key, {
      configurable: false,  // 是否可以删除或修改目标属性
      enumerable: true,   // 是否可枚举
      get: function proxyGetter() {
        return self._data[key];
      },
      set: function proxySetter(newVal) {
        self._data[key] = newVal;
      }
    })
  },

  _initComputed: function() {
    var self = this;
    var computed = this.$options.computed;
    if (typeof computed === 'object') {
      Object.keys(computed).forEach(function(key) {
        Object.defineProperty(self, key, {
          get: typeof computed[key] === 'function' ? computed[key] : computed[key].get,
          set: function() {}
        })
      })
    }
  }
}
```

html代码初始化如下：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>MVVM</title>
</head>
<body>

<div id="mvvm-app">
  <input type="text" v-model="someStr">
  <input type="text" v-model="child.someStr">
  <p>{{getHelloWord}}</p>
  <p v-html="child.htmlStr"></p>
  <button v-on:click="clickBtn">change model</button>
</div>
<script src="./observer.js"></script>
<script src="./watcher.js"></script>
<script src="./compile.js"></script>
<script src="./mvvm.js"></script>
<script>
    var vm = new MVVM({
      el: '#mvvm-app',
      data: {
        someStr: 'hello ',
        className: 'btn',
        htmlStr: '<span style="color: #f00;">red</span>',
        child: {
          someStr: 'World !'
        }
      },
      computed: {
        getHelloWord: function() {
          return this.someStr + this.child.someStr;
        }
      },
      methods: {
        clickBtn: function(e) {
          var randomStrArr = ['childOne', 'childTwo', 'childThree'];
          this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
        }
      }
    });
    vm.$watch('child.someStr', function() {
        console.log(arguments);
    });
</script>
</body>
</html>
```

代码分析：

1. 代码初始化执行

首先html代码如下：

```
<div id="mvvm-app">
  <input type="text" v-model="someStr">
  <input type="text" v-model="child.someStr">
  <p>{{getHelloWord}}</p>
  <p v-html="child.htmlStr"></p>
  <button v-on:click="clickBtn">change model</button>
</div>
复制代码
数据调用如下：

复制代码
var vm = new MVVM({
  el: '#mvvm-app',
  data: {
    someStr: 'hello ',
    className: 'btn',
    htmlStr: '<span style="color: #f00;">red</span>',
    child: {
      someStr: 'World !'
    }
  },
  computed: {
    getHelloWord: function() {
      return this.someStr + this.child.someStr;
    }
  },
  methods: {
    clickBtn: function(e) {
      var randomStrArr = ['childOne', 'childTwo', 'childThree'];
      this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
    }
  }
});
vm.$watch('child.someStr', function() {
  console.log(arguments);
});
复制代码
执行 new MVVM 后 mvvm.js 实列化，mvvm.js 代码如下：

复制代码
function MVVM(options) {
  this.$options = options || {};
  var data = this._data = this.$options.data;
  var self = this;
  /*
   数据代理，实现 vm.xxx 
   上面的代码看出 监听数据的对象是 options.data, 因此每次更新视图的时候；如：
   var vm = new MVVM({
     data: {name: 'kongzhi'}
   });
   那么更新数据就变成 vm._data.name = 'kongzhi2'; 但是我们想实现这样更改 vm.name = "kongzhi2";
   因此这边需要使用属性代理方式，利用Object.defineProperty()方法来劫持vm实列对象属性的读写权，使读写vm实列的属性转成vm.name的属性值。
   */
   Object.keys(data).forEach(function(key) {
     self._proxyData(key);
   });

   this._initComputed();

   // 初始化Observer
   observer(data, this);

   // 初始化 Compile
   this.$compile = new Compile(options.el || document.body, this);
}
```

因此 参数options值为对象(Object)：

```
this.$options = {
  el: '#mvvm-app',
  data: {
    someStr: 'hello ',
    className: 'btn',
    htmlStr: '<span style="color: #f00;">red</span>',
    child: {
      someStr: 'World !'
    }
  },
  computed: {
    getHelloWord: function() {
      return this.someStr + this.child.someStr;
    }
  },
  methods: {
    clickBtn: function(e) {
      var randomStrArr = ['childOne', 'childTwo', 'childThree'];
      this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
    }
  }
}
```

```
var data = this._data = this.$options.data = {
  someStr: 'hello ',
  className: 'btn',
  htmlStr: '<span style="color: #f00;">red</span>',
  child: {
    someStr: 'World !'
  }
}
```

然后 遍历data对象，如下：

```
Object.keys(data).forEach(function(key) {
  self._proxyData(key);
});
```

_proxyData 方法代码如下：

```
_proxyData: function(key) {
  var self = this;
  Object.defineProperty(self, key, {
    configurable: false,  // 是否可以删除或修改目标属性
    enumerable: true,   // 是否可枚举
    get: function proxyGetter() {
      return self._data[key];
    },
    set: function proxySetter(newVal) {
      self._data[key] = newVal;
    }
  })
}
```

使用Object.defineProperty来监听对象属性的变化，使vm实列的属性转变成vm._data的属性值。
接着初始化 this._initComputed();方法；代码如下：

```
_initComputed: function() {
  var self = this;
  var computed = this.$options.computed;
  if (typeof computed === 'object') {
    Object.keys(computed).forEach(function(key) {
      Object.defineProperty(self, key, {
        get: typeof computed[key] === 'function' ? computed[key] : computed[key].get,
        set: function() {}
      })
    })
  }
}
```

首先判断 computed 是否为对象，然后遍历computed，判断如果computed[key] 键是否是一个函数，如果是一个函数的话，就执行该函数，否则的话，就执行该get调用。所以我们看到使用vue的
时候会看到 computed方法初始时调用。

然后 初始化Observer

```
observer(data, this);
```

因此会实例化 Observer；代码如下：

```
function Observer(data) {
  this.data = data;
  this.init();
}
```

因此 this.data值为：

```
this.data = {
  someStr: 'hello ',
  className: 'btn',
  htmlStr: '<span style="color: #f00;">red</span>',
  child: {
    someStr: 'World !'
  }
}
```

然后调用init方法，如下代码：

```
init: function() {
  var self = this;
  var data = self.data;
  // 遍历data对象
  Object.keys(data).forEach(function(key) {
    self.defineReactive(data, key, data[key]);
  });
},
```

遍历data，获取每一个key，也就是 key 可以为 someStr, className, htmlStr, child等。
然后执行 self.defineReactive 方法，参数如下三个

```
data = {
  someStr: 'hello ',
  className: 'btn',
  htmlStr: '<span style="color: #f00;">red</span>',
  child: {
    someStr: 'World !'
  }
}
```

key 分别为 someStr, className, htmlStr, child，
data[key]值分别为：

```
'hello ', 'btn', '<span style="color: #f00;">red</span>', 和 
{
  someStr: 'World !'
}
```

调用 defineReactive  代码如下：

```
defineReactive: function(data, key, value) {
  var dep = new Dep();
  // 递归遍历子对象
  var childObj = observer(value);

  // 对对象的属性使用Object.defineProperty进行监听
  Object.defineProperty(data, key, {
    enumerable: true,  // 可枚举
    configurable: false, // 不能删除目标属性或不能修改目标属性
    get: function() {
      if (Dep.target) {
        dep.depend();
      }
      return value;
    },
    set: function(newVal) {
      if (newVal === value) {
        return;
      }
      value = newVal;
      // 如果新值是对象的话，递归该对象 进行监听
      childObj = observer(newVal);
      // 通知订阅者 
      dep.notify();
    }
  });
}
```

同理 使用 Object.defineProperty 监听对象属性值的变化。执行完成后，下一步代码执行如下：

```
// 初始化 Compile
this.$compile = new Compile(options.el || document.body, this);
```

options.el参数就是传进去的Id为 #mvvm-app, 如果没有的话，就是 document.body;
compile实列化代码如下：

```
/*
 * @param {el} 元素节点容器 如：el: '#mvvm-app'
 * @param {vm} Object 对象传递数据 
*/
function Compile(el, vm) {
  this.$vm = vm;
  // 判断是元素节点 还是 选择器
  this.$el = this.isElementNode(el) ? el : document.querySelector(el);
  if (this.$el) {
    /*
     因为遍历解析过程有多次操作dom节点，为了提高性能和效率，会先将根节点el转换成文档碎片fragment进行解析编译操作，
     解析完成后，再将fragment添加回原来真实的dom节点中。有关 DocumentFragment 请看 http://www.cnblogs.com/tugenhua0707/p/7465915.html
     */
    this.$fragment = this.node2Fragment(this.$el);
    this.init();
    this.$el.appendChild(this.$fragment);
  }
}
```

调用init方法会进行compileElement 执行解析编译操作，找到id为#mvvm-app的childNodes, 遍历子节点首先会判断是元素节点还是文本节点，如果是元素节点会调用 该如下方法：
this.compile(node); 进行元素节点编译和解析操作， 如果是文本节点的话，就会 调用 this.compileText(node, RegExp.$1); 方法编译文本节点。

理清思路：

视图 --> 模型的改变：

在输入框输入新值的时，会在 compile.js里面会监听input事件，如下代码：

```
node.addEventListener('input', function(e) {
    var newValue = e.target.value;
    if (val === newValue) {
      return;
    }
    self.setVMVal(vm, attrValue, newValue);
    val = newValue;
 });
```

首先会获取当前的值，然后执行 setVMVal该方法，如下代码：

```
/*
    设置普通指令的值
    @param {vm} 数据对象
    @param {exp} 普通指令的值 比如 v-model="xx" 那么exp就等于xx
    @param {value} 新值
   */
setVMVal: function(vm, exp, value) {
    var val = vm;
    exp = exp.split('.');
    exp.forEach(function (key, index) {
        // 如果不是最后一个元素的话，更新值
        /*
         数据对象 data 如下数据
         data: {
          child: {
            someStr: 'World !'
          }
        },
         如果 v-model="child.someStr" 那么 exp = ["child", "someStr"], 遍历该数组，
         val = val["child"]; val 先储存该对象，然后再继续遍历 someStr，会执行else语句，因此val['someStr'] = value, 就会更新到对象的值了。
         */
        if (index < exp.length - 1) {
            val = val[key];
        } else {
            val[key] = value;
        }
    });
}
```
执行完setVMVal方法后，会拿到最新值，然后会调用 Object.defineProperty 里面的setter方法，如下代码：

```
set: function(newVal) {
    if (newVal === value) {
        return;
    }
    value = newVal;
    // 发布消息
    dep.public();
}
```
从而发布消息, 执行方法 dep.public(); 因此会调用Dep里面的public方法，代码如下：

```
public: function() {
    this.subs.forEach(function (sub) {
        sub.update();
    });
}
```

从而调用 watcher.js 里面的update方法，代码如下：

```
update: function() {
  this.run();
},
```

接着会调用run方法，代码如下：

```
run: function() {
    var value = this.get();
    var oldVal = this.value;
    if (value !== oldVal) {
        this.value = value;
        this.cb.call(this.vm, value, oldVal);
    }
},
```

先调用get方法拿到当前的值，代码如下：

```
get: function() {
    Dep.target = this;
    var value = this.getter.call(this.vm, this.vm);
    Dep.target = null;
    return value;
},
```

如上代码，会调用 this.getter方法，this.getter方法初始化代码在watcher如下：

```
// expOrFn 是事件函数的话
if (typeof expOrFn === 'function') {
    this.getter = expOrFn;
} else {
    this.getter = this.parseGetter(expOrFn);
}
```

rseGetter 方法代码：

```
parseGetter: function(exp) {
    if (/[^\w.$]/.test(exp)) return; 
    var exps = exp.split('.');
    return function(obj) {
      for (var i = 0, len = exps.length; i < len; i++) {
        if (!obj) return;
        obj = obj[exps[i]];
      }
      return obj;
    }
}
```

用完成后，会执行到get方法里面代码 var value = this.getter.call(this.vm, this.vm); 因此会继续调用到 parseGetter 返回函数的代码，所以最后返回新值。
返回新值后，在执行函数run方法后，判断新旧值是否相等，不相等的话，就会执行 compile.js里面bind的方法的回调，如下代码：

```
new Watcher(vm, attrValue, function(value, oldValue) {
    updaterFn && updaterFn(node, value, oldValue);
});
```

compile.js里面的updater对象如下：

```
var updater = {
    textUpdater: function (node, value) {
        node.textContent = typeof value === 'undefined' ? '' : value;
    },
    htmlUpdater: function (node, value) {
        node.innerHTML = typeof value === 'undefined' ? '' : value;
    },
    classUpdater: function (node, newValue, oldValue) {
        var className = node.className;
        className = className.replace(oldValue, '').replace(/\s$/, '');
        var space = className && String(newValue) ? ' ' : '';
        node.className = className + space + newValue;
    },
    modelUpdater: function (node, newValue) {
        node.value = typeof newValue === 'undefined' ? '' : newValue;
    }
};
```

从而更新视图。

模型 -> 视图的改变

当页面初始化时，会判断是普通指令 v-model, v-text, or v-html，还是事件指令 v-on:click; compile.js相对应的代码如下：

```
// 判断是否是事件指令
if (self.isEventDirective(dir)) {
  // 事件指令 比如 v-on:click 这样的
  compileUtil.eventHandler(node, self.$vm, attrValue, dir);
}
```

所以会对事件进行处理： 如下函数：

```
// 事件处理
eventHandler: function(node, vm, attrValue, dir) {
  var eventType = dir.split(':')[1],
    fn = vm.$options.methods && vm.$options.methods[attrValue];
    if (eventType && fn) {
      node.addEventListener(eventType, fn.bind(vm), false);
    }
}
```

因此会对该节点进行绑定 'click'事件，代码 node.addEventListener(eventType, fn.bind(vm), false); 就是对绑定的事件 进行监听，demo里面是使用 v-on:click, 因此eventType就是click事件，因此会对 点击事件 进行监听。然后执行相对应的回调函数。该demo中的回调函数为 clickBtn(函数名)。
clickBtn函数代码如下：

```
clickBtn: function(e) {
  var randomStrArr = ['childOne', 'childTwo', 'childThree'];
  this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
}
```

因此 this.child.someStr 会重新赋值一个随机数，也就是说值会得到更新，因此首先会调用 mvvm.js的_proxyData中的get方法；代码如下：

```
_proxyData: function(key, setter, getter) {
  var me = this;
  setter = setter || 
  Object.defineProperty(me, key, {
      configurable: false,
      enumerable: true,
      get: function proxyGetter() {
          return me._data[key];
      },
      set: function proxySetter(newVal) {
          me._data[key] = newVal;
      }
  });
}
```

页面初始化的时候，vm.data.child.someStr 会被mvvm.js 里面的 代理方法转换成 vm.child.someStr, 因此给 vm.child.someStr 设置新值的时候，会调用 Object.defineProperty方法来监听属性值的改变，因此需要返回一个新值，所以先调用mvvm.js中的get方法，之后会调用 Observer.js代码 的Object.defineProperty(obj, key, {}）中的get方法，返回页面初始化的值，然后会调用Observer.js该对应的set方法，获取新值，然后判断新旧值是否相等，如果不相等的话，就会把消息发布出去该订阅者，订阅者会接收该消息。从而和上面 视图 -》模型 步骤一样 渲染视图页面。