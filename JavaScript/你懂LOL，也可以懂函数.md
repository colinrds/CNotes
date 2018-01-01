[toc]

## 技能介绍
先说说普通攻击，也就是函数最常见的调用方式，声明之后直接调用。不过要注意的是 非严格模式this指向window，严格模式为undefined
```
function fun(){//...}
fun();
```
增益技能——方法。obj.fun = function(){//技能效果...}，瞬间给obj加了一个fun的buff。不过这里已经将this改为了obj，同时这个buff可以叠加，叠加的方式很简单：函数执行后返回this实现链式调用。
召唤技能——构造函数。这个技能就牛X了，召唤很多个小弟，小弟还带有英雄的技能。召唤方法也很简单
```
function Fun(){}
var f = new Fun();
//或 var f = new Fun;
//this 将指向f
```
位移技能——apply()/call()。将英雄瞬间移动到其他位置。也就是改变作用域，让一些不具备该属性的对象也能调用该方法。
```
var obj = {
  id:1,
  show: function(){
    return this.id;
  }
};
var zdl = {
  id:2
};
obj.show.call(zdl);//2
```
终极技能——函数式编程。首先这个技能是可以叠加的，伤害值无上限。
```
//实现勾股定理
var square = function(x){return x*x;};
var add = function(x,y){return x+y;};
//将函数作为参数传入
var pyth = add(square(3),square(4));//
```
还可以从不同的方向施放。
```
//从右到左绑定参数，与bind正好相反
function partialRight(fun){
  var args = arguments;
  return function(){
    var a = array(arguments);
    a = a.concat(arguments);
    return fun.apply(this,a);
  };
}
```
对目标单位造成冰冻效果。
```
//一个记忆函数，可以缓存函数的执行结果，对于递归函数比较好用
function memorize(f){
  var cache = {};
  return functin(){
    //根据参数和参数长度来缓存。
    //为什么不直接用参数缓存？答案很简单，参数有可能为空~
    var key = arguments.length + Array.prototype.join.call(arguments, ",");
    if (key in cache) return cache[key];
    else return cache[key] = f.apply(this, arguments);
  }
}
```
## 英雄天赋
显示隐形单位——length。显示函数形参个数

```
function zdl(a,b){};
zdl.length;//2
```
增加三维——prototype。有了这个天赋，新创建的对象将会继承这个属性。

```
function Fun(){};
Fun.prototype.id = 1;
var f = new Fun();
f.id;//1
```
分身——bind()。创建一个带有装备的分身。

```
//加法函数
function add(x,y){
  return x + y;
}
//自增函数
var inc = add.bind(this, 1);
inc(2);//3
```
镜像——Function()，伪函数。为什么叫它们为镜像呢？因为它们和真正的函数有区别。

```
/*Function的区别
* 每次执行都将创建新的函数对象，在循环中使用会出现性能问题
* 不使用词法作用域，没有闭包特性，作用域为全局对象
*/
//用于动态创建函数
new Function("x", "y", "return x+y;");
//等价于
function(x,y){return x+y;}
//等价于
eval("function(x,y){return x+y;}");
```
```
/* 伪函数的区别
* 能被调用但不属于函数，如IE8这些浏览器不属于函数，如：
* window.alert()
* RegExp()
* 判断真伪函数的方法如下
*/
function (x){
  return Object.prototype.toString.call(x) === '[object Function]';
}
```
## 团队定位
肉——命名空间。当用作命名空间的时候就相当于是肉，抗住所有伤害

```
(function(){
  //...  
}());
```
ganker——闭包。ganker最喜欢的是什么？钻树林钻草丛。闭包也喜欢隐蔽，自带隐身技能，可以把变量隐藏起来私有化。闭包是指函数对象可以通过作用域链互相关联起来，函数体内部的变量都可以保存在函数作用域内。其原理就是当嵌套函数存储在对象或属性时，由于有外部引用指向它，所以不会被垃圾回收。

DPS——嵌套。如fun1(fun2())。值得注意的是函数和其它对象一样可以添加属性.
```
uniqueInteger.counter = 0;
function uniqueInteger(){
  return uniqueInteger.counter++;
}
```
## 推荐装备
第一件装备叫形参。

```
function fun(a,b,c){
  //a,b,c都是形参，c未传入为undefined
}
fun(1,2);
```
第二件是实参。arguments对象，伪数组，指向实参列表，通常用Array.prototype.slice(arguments)让其具有数组属性和方法。

两件装备？坑爹么？！淡定，这些只是出门装~

## 进阶攻略
可能有也可能没有，等我看完下一章“类与模块”再做定夺。

## 打怪升级
写出函数执行结果
```
var scope = 'global scope';
function checkscope(){
  var scope = 'local scope';
  function f() {
    return scope;
  }
  return f();
}
checkscope();
```
```
var scope = 'global scope';
function checkscope(){
  var scope = 'local scope';
  function f() {
    return scope;
  }
  return f;
}
checkscope()();
```
```
function zdl(){
  var funcs = [];
  for(var i=0;i<10;i++){
    funcs[i]  = function(){return i;}
  }
  return funcs;
}
var funcs = zdl();
funcs[5]();
```
利用闭包实现私有属性存取的方法