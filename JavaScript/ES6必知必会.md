[toc]

## let 和 const

1. let声明的变量只有在当前代码块成效，而且不具备变量提升（即代码块中有let声明的变量提前使用会报错）;
```
{
   console.log(a)    // a is not defined
   console.log(b)   //  2
   let a = 1;
   var b = 2;
}
```
2. let同一作用域内不允许重复声明;
3. const声明一个只读常量，一旦声明，无法更改;
4. const声明一个变量的时候，必须初始化，而且该变量只能在当前作用域有效;
5. const声明一个符合类型的数据时（主要是对象和数组），保存的是变量的内存地址，只能保证这个地址固定，不能保证数据结构不变;

```
const o = {};
o.name = 'hello';    //可以给对象添加属性
console.log(o);      // { "name" : "hello" }

o = {};             //报错，因为o的内存地址不能改变
```

6. 可以使用 Object.freeze()方法来冻结一个对象或者对象的某个属性;
7. ES5声明变量的方式有两种 : var 和 function; ES6声明变量的方式有六种 : var let const function import class；

## 变量的解构赋值

1. ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值；
2. 如果变量解构不成功就会返回 undefined；
3. 只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值；
4. 解构赋值时可以指定默认值 （ let [ foo = true ] = [] ）；
5. 如果数组的某个成员不严格等于( === ) undefined ， 默认值就不会生效；

```
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', 'undefined'];  // x = 'a' , y = 'undefined'
```

6. 如果指定的默认值是一个表达式，那么该表达式是惰性求值（即只有在使用到的时候才去求值）；

7. 默认值可以引用解构赋值的其他变量，但该变量必须已经声明；

8. 对象的解构中，变量必须与属性同名，才能取到正确的值，如果变量没有对应的同名属性，则会导致取不到值，最后等于undefined；

9. 对象的解构赋值是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者；

```
let { foo: baz , bar } = { foo: "aaa", bar: "bbb" }; 
foo // foo is not defined
baz // 'aaa'
bar // 'bbb'
```

上述例子中 foo 是匹配的模式，baz才是变量。真正被赋值的是变量baz，而不是模式foo ， bar之所以能匹配，是因为对象的解构赋值是下面形式的简写

```
let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" }
```

10. 变量的解构赋值通常用于交换变量，函数参数定义，参数指定默认值等，虽然解构赋值虽然很方便，但是解析起来并不容易，如果使用不当就会产生与预期值不同的问题~

## 字符串的拓展

1. ES6为字符串添加了遍历器接口，因此可以使用for...of循环遍历字符串
2. 字符串新增的 includes()、startsWith()、endsWidth() 三个方法用于判断某一字符串是否包含于另一字符串

- includes()：返回布尔值，表示源字符串中是否包含参数字符串。
- startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部。
- endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部。

3. 新增 repeat() 方法， 该方法返回一个新字符串，不是对原字符串操作，表示将原字符串重复n次。

```
let str = 'abc';
str.repeat(3) //abcabcabc 
str //abc
```

ps:该方法参数为正整数，如果为负数会报错，小数向下取整；

4. 新增 padStart()，padEnd() 方法，用于补全字符串，该方法返回一个新字符串，不是对原字符串操作，传入两个参数，第一个参数用来指定字符串的最小长度，第二个参数是用来补全的字符串（缺省的话默认空格补全）。（ps：如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串）；

```
let str = 'abc';
str.padStart(2, 'abc') // 'abc'
str.padEnd(2, 'abc') // 'abc'
```

5. 模板字符串··（esc下面的那个按键），可以摆脱传统的字符串拼接的模式，直接将变量（表达式，对象的引用，函数等）写在模板字符串中输出

```
let str = 'world';
let html = `hello ${str}`;
html //hello wrold
```

ps：所有模板字符串的空格和换行，都是被保留的，可以使用trim方法消除换行。

6. 标签模板，即模板字符串紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串，这中方式被称为“标签模板”，“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数

```
console.log `123`
// 等同于
console.log (123)
```

7. 如果模板字符里面有变量，会将模板字符串先处理成多个参数，再调用函数;

```
var a = 5;
var b = 10;

function tag(s, v1, v2) {
  console.log(s)
  console.log(v1);
  console.log(v2);
}

tag`Hello ${ a + b } world ${ a * b }`;

//['Hello','world','']
//15
//50
```

可以看出，tag函数第一个参数是一个数组，数组的成员是模板字符串中那些没有变量替换的部分，其他参数，都是模板字符串各个变量被替换后的值；

8. 再来一个例子，看看标签模板的使用方法：

```
var total = 30;
var msg = passthru`The total is ${total} (${total*1.05} with tax)`;

function passthru(literals) {
  var result = '';
  var i = 0;

  while (i < literals.length) {
    result += literals[i++];
    if (i < arguments.length) {
      result += arguments[i];
    }
  }
  return result;
}
```

上述例子中，参数 literals 实际上是  `["The total is "," ("," with tax)"] `, 函数内部 arguments 的值是 `{ "0" : ["The total is "," ("," with tax)"] , "1" : 30 , "2" : 31.5 }`，通过以上梳理，可以将各个参数按照原来的位置拼合回去，最终得到输出为"The total is 30 (31.5 with tax)"

ps：在使用标签模板的时候，要注意参数的位置，可根据自己实际的需求进行修改，返回正确的结果；

## 函数的拓展

1. ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面，一目了然，十分实用

```
function say( x , y = 'World') {
    console.log( x , y);
}
say('Hello')  // Hello World
say('Hello','Kite')  //Hello Kite
```

2. 函数参数默认已经声明，不能再用 let 或者 const 声明，而且不能有同名参数

3. 一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失；

```
var x = 1;

function f(x, y = x) {
  console.log(y);
}
f(2) // 2
```

上面例子中，参数y的默认值等于变量x。调用函数f时，参数形成一个单独的作用域。在这个作用域里面，默认值变量x指向第一个参数x，而不是全局变量x，所以输出是2；

```
let x = 1;

function f( y = x ) {
  let x = 2;
  console.log(y);
}
f() // 1
```

上面例子中,函数f调用时，参数y=x形成一个单独的作用域。这个作用域里面，变量x本身没有定义，所以指向外层的全局变量x。函数调用时，函数体内部的局部变量x影响不到默认值变量x;

4. 函数声明时，可以将某个参数默认值设为undefined，表明这个参数是可以省略的；

5. rest 参数（形式为...变量名），用于获取函数的多余参数，该变量是一个数组，用于将多余的参数放入该数组中。（rest 参数之后不能再有其他参数，它只能是函数的最后一个参数，否则会报错）

```
function func(...params){
    console.log(params)
}
func(1,2,3) // [1，2，3]

function func( x , ...params){
    console.log(params)
}
func(1,2,3) // [2，3]
```

6. 箭头函数（=>），ES6 允许使用“箭头”（=>）定义函数，这种写法相比较 ES5 定义 function 看起来简洁得多；

```
var func = x => x 
//等同于
var func = function func(x) {
    return x;
};
```

7. 如果箭头函数没有参数或者有多个参数的话，则需要加上()来进行声明；

```
var func = () => 'Hello World';
//等同于
var func = function func() {
  return 'Hello World';
};

var func = ( x , y ) => x + y
//等同于
var func = function func(x, y) {
  return x + y;
};
```

8. 如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回;

```
var func = ( x , y ) => { return x + y; }
```

9. 如果箭头函数直接返回一个对象，必须在对象外面加上括号;

```
var func = ( x , y ) => ({ x : x , y : y })
```

10. 箭头函数使用时必须注意以下几个问题：

函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象；
```
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });  //42
```
上面代码中，setTimeout的参数是一个箭头函数，这个箭头函数的定义生效是在foo函数生成时，而它的真正执行要等到100毫秒后。如果是普通函数，执行时this应该指向全局对象window，这时应该输出21。但是，箭头函数导致this总是指向函数定义生效时所在的对象（本例是{id: 42}），所以输出的是42。

箭头函数不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误；

不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替

不可以使用yield命令，因此箭头函数不能用作 Generator 函数。


## 数组的扩展

1. 拓展运算符（'...'），它相当于rest参数的逆运算，用于将一个数组转换为用逗号分隔的参数序列；
```
console.log(...[1, 2, 3])
// 1 2 3
console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5
```

2. 如果扩展运算符后面是一个空数组，则不产生任何效果；

```
[...[], 1]
// [1]
```

3. 常见的拓展运算符的应用：

```
//合并数组

let arr1 = [1,2];
let arr2 = [3,4];
let arr3 = [5,6];

let newArr = [...arr1 , ...arr2 , ...arr3];  //等同于ES5 [].concat( arr1 , arr2 , arr3 )
// [1,2,3,4,5,6]


//与解构赋值结合（用于生成数组）

const [ val , ...rest] = [1, 2, 3, 4, 5];
val // 1
rest  // [2, 3, 4, 5]


//将字符串转为真正的数组

let str = 'mine';
[...str]  // ["m","i","n","e"]



//可以将类数组转化成正真的数组

 let arrayLike = {
    0 : 'div.class1' ,
    1 : 'div.class2' ,
    2 : 'div.class3' ,
    length : 3
}
console.log([...arrayLike])  // ["div.class1","div.class2","div.class3"]
```

4. 新增 Array.from 方法，可以将类似数组的对象（array-like object）和可遍历（iterable）的对象转化成真正的数组；该方法还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组；

```
let arr = [ 1 , 2 , 3];

arr.map( x => x * x);
// [ 1 , 4 , 9 ]

Array.from(arr, (x) => x * x)
// [ 1 , 4 , 9 ]
```

5. 新增 Array.of 方法 ，用于将一组值，转换为数组（该方法基本上可以用来替代Array()或new Array()，避免出现参数不同而导致的重载）；

```
//传统Array

Array() // []
Array(3) // [, , ,]
Array(1, 2, 3) // [1, 2, 3]

//Array.of

Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

6. 数组实例方法 find() 用于找出第一个符合条件的数组成员，该方法的参数是一个回调函数，该回调函数可以接收三个参数，依次是当前元素，当前元素索引，数组本身；如果查找成功，返回符合条件的第一个成员，如果没有符合条件的成员，则返回undefined；

```
var arr = [1, 2, 4, 5];
var r = arr.find(function( element , index , self ){
    return element % 2 == 0;
})
r  // 2
```

7. 数组实例方法 findIndex() , 该方法的参数与 find() 一样 ， 不同的是该方法 返回的是第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1;

```
var arr = [1, 2, 4, 5];
var r = arr.find(function( element , index , self ){
    return element % 2 == 0;
})
r  // 1
```

ps：find() 和 findIndex() 这两个方法都可以发现NaN，弥补了数组的IndexOf方法的不足。

8. 数组实例方法 includes() , 方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似;该方法接收两个参数，第一个参数是要查找的成员，第二个参数表示搜索的起始位置（如果为负数，则表示倒数的位置，如果大于数组长度，则会重置为从0开始）

```
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false

[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

## 对象的拓展

1. ES6 允许直接写入变量和函数，作为对象的属性和方法(在对象中，直接写变量时，属性名为变量名, 属性值为变量的值)

```
//属性简写
var foo = 'bar';
var obj = {foo};
obj // { foo : "bar" }

//变量简写
var mine = {
    foo , 
    method(){
        //to do
    }
}
```

2. ES6 允许字面量定义对象时，用表达式作为对象的属性名或者方法名，即把表达式放在方括号内;

```
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123,
  ['s' + 'ay'](){
    console.log('hello world')
  }
}
obj // {"foo":true,"abc":123}
obj.say()  // 'hello world'
```

3. 新增 Object.is() 方法，用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致，不同之处在于一是+0不等于-0，二是NaN等于自身。

```
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

4. 新增 Object.assign 方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target），第一个参数是目标对象，后面的参数都是源对象；

```
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

ps:如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

```
var target = { a: 1, b: 1 };

var source1 = { b: 2, c: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

该方法不能用于目标对象是 undefined 和 null 上， 会报错；

5. Object.assign 方法实行的是浅拷贝，而不是深拷贝。如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用，修改会对原对象造成影响；

```
var obj1 = {a: {b: 1}};
var obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b     // 2
```

6. Object.assign 方法常用于以下几个方面

为对象添加属性

```
var _this = {};
Object.assign( _this , { name : 'mine' } );
_this // { name : 'mine' }
```

为对象添加方法

```
var _this = {};
Object.assign( _this , { func(){ console.log('hello world') } } );
_this.func()  // hello world
```

克隆对象

```
var _this = { name : 'mine' };
Object.assign( {} , _this );
```

合并多个对象

```
var _this = {};
var source1 = { name : 'mine' };
var source2 = { mail : 'your' };
Object.assign( _this , source1 , source2 );
_this // {"name":"mine","mail":"your"}
```

为属性指定默认值

```
var default = {  name : 'mine' , mail : 'your' }
function processContent(options) {
    options = Object.assign({}, default , options);
    // to do
}
```

7. Object.setPrototypeOf方法的作用与`_proto_`相同，用来设置一个对象的prototype对象，返回参数对象本身。它是 ES6 正式推荐的设置原型对象的方法。
8. 
```
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);
proto.y = 20;
proto.z = 40;
obj.x // 10
obj.y // 20
obj.z // 40
```

8. Object.getPrototypeOf()方法，该方法与Object.setPrototypeOf方法配套，用于读取一个对象的原型对象。

9. Object.keys()，Object.values()，Object.entries() 除第一个外，后面两个是ES6新增的方法，用于遍历对象，返回都是数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键、值和键值对数组。

```
let obj = { a : 1 , b : 'hello' }

Object.keys( obj );     // ["a","b"]
Object.values( obj );   // [1,"hello"]
Object.entries( obj );  // [["a",1],["b","hello"]]
```

## Symbol

1. Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。它是 JavaScript 语言的第七种数据类型，前六种分别是：undefined、null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）；
2. Symbol 值通过Symbol函数生成，可以作为对象的属性名使用，保证不会与其他属性名产生冲突；
```
let s = Symbol();
typeof s  // symbol
```

ps：上面代码表示创建一个Symbol变量，值得注意的是，Symbol函数前不能使用new命令，否则会报错，也就是说Symbol 是一个原始类型的值，不是对象，也不能添加属性；

3. Symbol函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要用于区分不同的 Symbol 变量；

```
let s1 = Symbol('a');
let s2 = Symbol('b');

s1.toString()  // 'Symbol(a)'
s2.toString()  // 'Symbol(b)'
```

ps：Symbol函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的

```
let s1 = Symbol('a');
let s2 = Symbol('a');

s1 === s2  //false
```

4. Symbol 值不能与其他类型的值进行运算，但可以转为布尔值，但是不能转为数值；

```
let s = Symbol();
s + '2'       // Cannot convert a Symbol value to a string
Boolean(s)    // true
!s            // false
```

5. 用于对象的属性名，可以保证不会出现同名的属性，对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖；值得注意的是，Symbol 值作为对象属性名时，不能用点运算符，因为点运算符后面是一个字符串；

```
let s = Symbol();

let obj = {};
obj[s] = 'hello world';

//或者

let obj  = {
    [s] : 'hello world'
}

obj.s   // undefined
obj[s]  // hello world
```

6. Symbol 作为属性名，不会被常规方法遍历得到，即该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回，但是，它并不是私有属性，可以使用 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名；

```
var obj = {};
var a = Symbol('a');
var b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';
obj.c  = 'Mine';

for( let key in obj ){
   console.log(key)         // c
}

var objectSymbols = Object.getOwnPropertySymbols(obj);

console.log(objectSymbols) // [Symbol(a), Symbol(b)]
```

7. Symbol.for方法接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值；它与Symbol()不同的是，Symbol.for()不会每次调用就返回一个新的 Symbol 类型的值，而是会先检查给定的key是否已经存在，如果不存在才会新建一个值，而 Symbol()每次都会返回3不同的Symbol值；

```
Symbol.for("name") === Symbol.for("name")
// true

Symbol("name") === Symbol("name")
// false
```
8. Symbol.keyFor方法返回一个已登记的 Symbol 类型值的key，而Symbol()写法是没有登记机制的；

```
var s1 = Symbol.for("name");
Symbol.keyFor(s1) // "name"

var s2 = Symbol("name");
Symbol.keyFor(s2) // undefined
```

ps：Symbol.for为Symbol值登记的名字，是全局环境的，可以在不同的 iframe 或 service worker 中取到同一个值

## Set 和 Map

1. ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值，它 本身是一个构造函数，用来生成 Set 数据结构。

```
let s = new Set([1,2,3,4,5,2,2,3,5]);
s // [1,2,3,4,5]
```
2. 可以使用add(key)方法可以添加元素到Set中，可以重复添加，但不会有效果，值得注意的是向Set加入值的时候，不会发生类型转换，即 5 和 "5" 是两个不同的值，但在 Set 内部，两个NaN是相等

```
let s = new Set([1,2,3]);
s.add(4)   //[1,2,3,4]
s.add(4)   //[1,2,3,4]
s.add(5)   //[1,2,3,4,5]
s.add('5') //[1,2,3,4,5,"5"]
s.add(NaN) //[1,2,3,4,5,"5",NaN]
s.add(NaN) //[1,2,3,4,5,"5",NaN]
```
3. 可以利用Set数据不重复的特性，提供一种新的数组去重方法

```
// 去除数组的重复成员
[...new Set(array)]

[...new Set([1,2,2,3,3,4,5,5])]  //[1,2,3,4,5]
```
4. Set常见的操作方法有：

add(value)：添加某个值，返回Set结构本身。
delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
has(value)：返回一个布尔值，表示该值是否为Set的成员。
clear()：清除所有成员，没有返回值。

```
s.add(1).add(2).add(2);
// 注意2被加入了两次
s.size // 2
s.has(1) // true
s.has(2) // true
s.has(3) // false
s.delete(2);
s.has(2) // 
```

5. Set 结构的实例有四个遍历方法，可以用于遍历成员。

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员

需要特别指出的是，Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用Set保存一个回调函数列表，调用时就能保证按照添加顺序调用。

```
let set = new Set(['red', 'green', 'blue']);
for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue
for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue
for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

6. ES6 提供了 Map 数据结构，它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适；

7. Map常见的操作方法有：

- set(key，val)：添加某个值，返回Map结构本身。
- get(key)： 读取某个键，如果该键未知，则返回undefined
- delete(key)： 删除某个键，返回一个布尔值，表示删除是否成功。
- has(key)： 返回一个布尔值，表示该值是否为Map的键。
- clear() : 清除所有成员，没有返回值。

```
const m = new Map();
const o = { str : 'Hello World'};
m.set(o, 'content')
m.get(o) // "content"
m.has(o) // true
m.delete(o) // true
m.has(o) // false
```
8. 只有对同一个对象的引用，Map 结构才将其视为同一个键

```
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map.set(k1, 111).set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```

上面例子表明，Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键，因为 k1 和 k2 是两个不同的对象，放在不同的内存地址中，所以Map视为不同的键

9. Map 结构原生提供三个遍历器生成函数和一个遍历方法。

- keys()：返回键名的遍历器。
- values()：返回键值的遍历器。
- entries()：返回所有成员的遍历器。
- forEach()：遍历 Map 的所有成员。

ps：Map 的遍历顺序就是插入顺序，这里就不示例了，大家自己动手实践一下。

10. 可以使用扩展运算符（...）将Map转换为数组，反过来，将数组传入 Map 构造函数，就可以转为 Map了

```
//Map转数组
const map = new Map();
map.set('name' , 'hello').set({},'world');

[...map] //[["name","hello"],[{},"world"]]

//数组转Map
const map = new Map([["name","hello"],[{},"world"]]);

map // {"name" => "hello", Object {} => "world"}
```

## Promise 对象

1. Promise对象是ES6对异步编程的一种解决方案，它有以下两个特点：

Promise对象代表一个异步操作，它只有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败），并且该状态不会受外界的影响
Promise对象的状态改变，只有两种可能：从 Pending 变为 Resolved 或者从 Pending 变为 Rejected，并且一旦状态改变，就不会再变，任何时候都可以得到这个结果

2. Promise对象的一些缺点：

一旦新建了一个Promise对象，就会立即执行，并且无法中途取消

```
let promise = new Promise(function(resolve, reject) {
    console.log('Promise');
    resolve();
});
// Promise
```

如果不设置Promise对象的回调函数，Promise会在内部抛出错误，不会反应到外部，也就是在外部拿不到错误提示
如果Promise对象处于Pending状态时，是无法得知捕获进展到哪一个阶段的（刚刚开始还是即将完成）

3. Promise对象是一个构造函数，用来生成Promise实例，下面是创造了一个Promise实例的示例

```
let promise = new Promise(function(resolve, reject) {
  // ... to do

  if ( success ){
    resolve(value);    //成功操作
  } else {
    reject(error);     //失败操作
  }
});
```

ps：Promise 构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 和 reject ，分别用来处理成功和失败的回调；

4. Promise实例生成以后，可以用 then 方法分别指定 Resolved 状态和 Reject 状态的回调函数；

```
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

ps：then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调函数是Promise对象的状态变为Rejected时调用，其中，第二个函数是可选的；

5. resolve函数的参数除了正常的值以外，还可能是另一个 Promise 实例，表示异步操作的结果有可能是一个值，也有可能是另一个异步操作；

```
let promise1 = new Promise(function (resolve, reject) {
  // ...
});

let promise2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```

上面代码表示一个异步操作的结果是返回另一个异步操作，promise1 的状态就会传递给 promise2 , 如果 promise1 的状态是Pending，那么 promise2 的回调函数就会等待promise1的状态改变；如果promise1的状态已经是Resolved或者Rejected，那么promise2的回调函数将会立刻执行;

6. Promise实例方法then返回的是一个新的Promise实例,因此可以采用链式写法，即then方法后面再调用另一个then方法

```
let promise = new Promise(function (resolve, reject) {
  // ...
})
promise.then(function(res) {
   return res.post;
}).then(function(post) {
   // ...
});
```

ps：上例中依次指定了两个回调函数，第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数，如果返回的是 Promise 对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用

```
let promise = new Promise(function (resolve, reject) {
  // ...
})
promise.then(function(res) {
   return new Promise(/.../);
}).then(function(res) {
   // Resolved
},function(error){
   // Rejected
});
```

7. Promise.prototype.catch 方法用于指定发生错误时的回调函数，不仅异步操作抛出错误（即状态就会变为Rejected），而且 then 方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获

```
let promise = new Promise(function(resolve, reject) {
    throw new Error('test');
}).catch(function(error) {
  console.log(error);
});

// Error: test
```

8. 如果Promise状态已经变成Resolved，再抛出错误是无效的

```
let promise = new Promise(function(resolve, reject) {
  resolve('ok');
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });

  //ok
```

ps： 出现上述结果是由于 之前提到的 Promise 的状态一旦改变，就永久保持该状态，不会再变了，因此在 resolve 语句后面，再抛出错误，是不会被捕获的

9. Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止，因此建议总是使用catch方法，而不使用then方法的第二个参数，因为使用catch方法可以捕获前面then方法执行中的错误

```
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

10. Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例，该方法接收一个 promise对象的数组作为参数，当这个数组里的所有promise对象全部变为resolve或reject状态的时候，它才会去调用 .then 方法。

```
var p1 = new Promise(function (resolve) {
    setTimeout(function () {
        resolve("Hello");
    }, 3000);
});

var p2 = new Promise(function (resolve) {
    setTimeout(function () {
        resolve("World");
    }, 1000);
});

Promise.all([p1, p2]).then(function (result) {
    console.log(result);    // ["Hello", "World"]
});
```

上面的例子模拟了传输两个数据需要不同的时长，虽然 p2 的速度比 p1 要快，但是 Promise.all 方法会按照数组里面的顺序将结果返回，但 promise 本身并不是一个个的顺序执行的，而是同时开始、并行执行的，可以利用这个特点处理需要多个回调返回后才能进行的操作

11. Promise.race方法和Promise.all方法类似，也接收一个promise对象数组为参数，不同的是只要该数组中的 Promise 对象的状态发生变化（无论是 resolve 还是 reject）该方法都会返回。

```
var p1 = new Promise(function (resolve) {
    setTimeout(function () {
        resolve("Hello");
    }, 3000);
});
var p2 = new Promise(function (resolve) {
    setTimeout(function () {
        resolve("World");
    }, 1000);
});
Promise.race([p1, p2]).then(function (result) {
    console.log(result);    // Wrold
});
```

12. 一般情况下我们都会使用 new Promise() 来创建promise对象，除此之外，可以使用 Promise.resolve 和 Promise.reject这两个方法；

静态方法Promise.resolve(value) 可以认为是 new Promise() 方法的快捷方式

```
let promise = Promise.resolve('resolved');
//等价于
let promise = new Promise(function(resolve){
    resolve('resolved');
});
```

上述的promise对象立即进入确定（即resolved）状态，并将 'resolved' 传递给后面then里所指定的 onFulfilled 函数。
```
Promise.resolve('resolved').then(function(value){
    console.log(value);
});
// resolved
```

Promise.reject(error)是和 Promise.resolve(value) 类似的静态方法，是 new Promise() 方法的快捷方式。

```
let promise = Promise.reject(new Error("出错了"));
//等价于
let promise = new Promise(function(resolve,reject){
    reject(new Error("出错了"));
});
```

上述 promise 对象通过then指定的 onRejected 函数，并将错误（Error）对象传递给这个 onRejected 函数
```
Promise.reject(new Error("fail!")).catch(function(error){
    console.error(error);
});
// Error : fail!
```

13. 我们可以利用 Promise 应用到我们实际开发中，下面举几个栗子

```
//图片加载
const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    var image = new Image();
    image.onload  = resolve(image);
    image.onerror = function() {
        reject(new Error('Could not load image at ' + path));
    };
    image.src = path;
  });
}

//文件读取
function reader (file) {
  return new Promise(function (resolve, reject) {
    let reader = new FileReader();

    reader.onload = function () {
      resolve(reader);
    };
    reader.onerror = function() {
        reject(new Error('Could not open the file ' + file));
    };

    if (!file.type || /^text\//i.test(file.type)) {
      reader.readAsText(file);
    } else {
      reader.readAsDataURL(file);
    }
  })
}
```
