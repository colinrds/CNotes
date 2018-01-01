[toc]

在写javascript的时候，调试错误必不可少，除了能够在浏览器中打印出来错误外，常常还需要知道错误的类型是什么，以便对症下药的纠错；也有时候，在自己封装的工具函数中，不传参或传入了错误类型的参数，也要适当的抛出一些错误以示警告；使用框架不正常情况下也会抛出错误，如果对错误一无所知，便无从下手调试。综合上述，了解错误的处理机制是多么必要，以下是笔者归纳总结，如有误之处，欢迎指出。

## 错误构造函数

javascript规范中总共有8中错误类型构造函数

- Error -- 错误对象
- SyntaxError --解析过程语法错误
- TypeError -- 不属于有效类型
- ReferenceError -- 无效引用
- RangeError -- 数值超出有效范围
- URIError -- 解析URI编码出错
- EvalError -- 调用eval函数错误
- InternalError -- Javascript引擎内部错误的异常抛出， "递归太多"

其中两种做个特殊说明：

> EvalError调用eval函数错误，已经弃用，为了向后兼容，低版本还可以使用。
> InternalError 递归过深 抛出错误，多数浏览器未实现，属于非标准方法，生产环境禁用

## 继承关系

Error是错误的基类，其他类型都继承Error这个类，可以使用ES6中提供的Object.getPrototypeOf()来判断，一个类是否继承了另一个类。

```
console.log(Object.getPrototypeOf(SyntaxError) === Error);    // true
console.log(Object.getPrototypeOf(TypeError) === Error);   // true
console.log(Object.getPrototypeOf(ReferenceError) === Error);   // true
console.log(Object.getPrototypeOf(RangeError) === Error);   // true
console.log(Object.getPrototypeOf(URIError) === Error);   // true
console.log(Object.getPrototypeOf(EvalError) === Error);   // true
```

来聊一聊每一种错误类型的使用和出错的场景。

## Error

通过Error的构造器可以创建一个错误对象。当运行时错误产生时，Error的实例对象会被抛出。

语法：new Error([message])

参数：
- message 可选，错误描述信息。

抛出错误

使用throw语句来抛出异常

```
throw new Error('这里抛出的是错误信息')
```

运行后，会在控制台打印输出：

```
Uncaught Error: 这里抛出的是错误信息
```

> 注意： 使用throw抛出异常后，之后的代码不再执行。

捕获错误

可以通过try{}catch(){}语句来捕获到这个错误

```
try{
    throw new Error('这里抛出的是错误信息')
}catch(err){
    alert(err.name + ' '+ err.message)
}
```

属性说明：

当使用new Error创建错误实例后，会有两个属性：

```
let e = new Error('这里抛出的是错误信息');
```

- name属性，为错误的类型，此时为Error
- message属性，为错误的信息，此时为'这里抛出的是错误信息'

## SyntaxError

解析过程语法错误,这种类型抛出的错误有很多，往往是书写时候造成的语法错误，例如：

```
let n = 1\1;   // Uncaught SyntaxError: Invalid or unexpected token
let str = "hel"lo" // Uncaught SyntaxError: Unexpected identifier
let 123Var = 'hi' // Uncaught SyntaxError: Invalid or unexpected token
```
语法错误有很多就不一一列举了，当在浏览器运行时，控制台会抛错，并且告知第几行，所以调试器来比较方便。但要读懂错误的类型为SyntaxError，以及后面的错误信息，这样方便改错。

## TypeError

不属于有效类型。这种错误就是在给的不是需要的类型而导致无法操作，会抛出类型错误。

变量或参数不是预期类型，
例如**new**运算符后必须是函数，而给定的不是函数，则会抛出类型错误

```
let fn = 'hello';
new fn;
```

抛出错误：

```
Uncaught TypeError: fn is not a constructor
```

调用对象不存在的方法

```
let obj = {};
obj.fn()
```

抛出错误：

```
Uncaught TypeError: obj.fn is not a function
```

当然你也可以在封装函数时候，强制传入的参数为指定类型，否则抛出类型错误。

```
function flatten(arr){
    if( !Array.isArray(arr) ){
        throw new TypeError('传入参数不是数组')
    }    
}
flatten('test');
```

传入的参数不为数组时，抛出自定义的类型错误：

Uncaught TypeError: 传入参数不是数组

## ReferenceError

无效引用。

引用了一个不存在的变量

```
console.log(a);
```

抛出错误

```
Uncaught ReferenceError: a is not defined
```

将变量赋值给一个无法被赋值的数据
这个错误常常犯的地方实在调用一个方法后在if语句中做判断，将比较运算符==写成了赋值运算符=，例如判断一个字符串第一个字符是不是指定的字符：

```
let str = 'hello';
if( str.charAt(0) = 'h' ){
    console.log('第一个字符为h');
}
```

抛出错误：

```
Uncaught ReferenceError: Invalid left-hand side in assignment
```

## RangeError

数值超出有效范围。在一些方法中，传入的数值必须在一定的范围内，否则会抛出超出范围的错误。

创建数组传入的长度小于了0

```
let arr = new Array(-1)
```

抛出错误：

```
Uncaught RangeError: Invalid array length
```

repeat方法重复指定的字符串重复次数小于0

```
let str = 'hello';
str.repeat(-1)
```

抛出错误：

```
Uncaught RangeError: Invalid count value
```

## URIError

处理URI编码出错。函数参数不正确，主要是encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数。

例如：

```
decodeURIComponent('%');
decodeURI('%2')
```

抛出错误：

```
Uncaught URIError: URI malformed
```

## 自定义错误类型

有时候希望自定义错误类型，需要自定义一个构造函数，然后让原型继承继承Error.prototype即可。

```
function MyErrorType(message){
    this.message = message || '错误';
    this.name = 'MyErrorType';
    this.stack = (new Error()).stack;  // 错误位置和调用栈
}

MyErrorType.prototype = Object.create(Error.prototype);
MyErrorType.prototype.constructor = MyErrorType;

throw new MyErrorType('自定义错误类型抛出错误')
```

## 关于调用的错误栈信息

提供的错误的跟踪功能，以什么样的调用顺序，在哪个文件的哪一行捕获到这个错误。

例如以下调用：

```
function trace() {
  try {
    throw new Error('myError');
  }
  catch(e) {
    console.log(e.stack);
  }
}
function b() {
  trace();
}
function a() {
  b(3, 4, '\n\n', undefined, {});
}
a('first call, firstarg');
```

错误信息为：

```
Error: myError
    at trace (<Error.html>:3:14)
    at b (<Error.html>:10:6)
    at a (<Error.html>:13:6)
    at <Error.html>:15:4
```

以上为抛错的构造函数的总结，如有误之处欢迎扶正。
以上每一种错误场景并没有列出太多，如果你有新的错误信息发现，欢迎留言讨论。