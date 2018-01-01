[toc]

字符串作为基本的信息交流的桥梁，几乎被所有的编程语言所实现（然而c、c++没有提供）。多数开发者几乎每天都在和字符串打交道，语言内置的String模块，极大地提升了开发者的效率。JavaScript通过自动装箱字符串字面量为String对象，自然地继承了String.prototype的所有方法，更加简化了字符串的使用。

截止ES6，字符串共包含31个标准的API方法，其中有些方法出镜率较高，需要摸透原理；有些方法之间相似度较高，需要仔细分辨；甚至有些方法执行效率较低，应当尽量少的使用。下面将从String构造器方法说起，逐步帮助你掌握字符串。

## String构造器方法

### fromCharCode

fromCharCode() 方法返回使用指定的Unicode序列创建的字符串，也就是说传入Unicode序列，返回基于此创建的字符串。

语法：fromCharCode(num1, num2,…)，传入的参数均为数字。

如下这是一个简单的例子，将返回 ABC、abc、*、+、- 和 /：
```
String.fromCharCode(65, 66, 67); // "ABC"
String.fromCharCode(97, 98, 99); // "abc"
String.fromCharCode(42); // "*"
String.fromCharCode(43); // "+"
String.fromCharCode(45); // "-"
String.fromCharCode(47); // "/"
```

看起来fromCharCode像是满足了需求，但实际上由于js语言设计时的先天不足（只能处理UCS-2编码，即所有字符都是2个字节，无法处理4字节字符），通过该方法并不能返回一个4字节的字符，为了弥补这个缺陷，ES6新增了fromCodePoint方法，请往下看。

fromCodePoint(ES6)

fromCodePoint() 方法基于ECMAScript 2015（ES6）规范，作用和语法同fromCharCode方法，该方法主要扩展了对4字节字符的支持。

// "𝌆" 是一个4字节字符，我们先看下它的数字形式
"𝌆".codePointAt(); // 119558
//调用fromCharCode解析之，返回乱码
String.fromCharCode(119558); // "팆"
//调用fromCodePoint解析之，正常解析
String.fromCodePoint(119558); // "𝌆"
除了扩展对4字节的支持外，fromCodePoint还规范了错误处理，只要是无效的Unicode编码，就会抛出错误RangeError: Invalid code point...，这就意味着，只要不是符合Unicode字符范围的正整数（Unicode最多可容纳1114112个码位），均会抛出错误。

String.fromCodePoint('abc'); // RangeError: Invalid code point NaN
String.fromCodePoint(Infinity); // RangeError: Invalid code point Infinity
String.fromCodePoint(-1.23); // RangeError: Invalid code point -1.23
注：进一步了解Unicode编码，推荐这篇文章：浅谈文字编码和Unicode（下）。如需在老的浏览器中使用该方法，请参考Polyfill。

### raw(ES6)

raw() 方法基于ECMAScript 2015（ES6）规范，它是一个模板字符串的标签函数，作用类似于Python的r和C#的@字符串前缀，都是用来获取一个模板字符串的原始字面量。

语法： String.raw(callSite, ...substitutions)，callSite即模板字符串的『调用点对象』，...substitutions表示任意个内插表达式对应的值，这里理解起来相当拗口，下面我将通俗的讲解它。

如下是python的字符串前缀r：

```
# 防止特殊字符串被转义
print r'a\nb\tc' # 打印出来依然是 "a\nb\tc"
# python中常用于正则表达式
regExp = r'(?<=123)[a-z]+'
```

如下是String.raw作为前缀的用法：

```
// 防止特殊字符串被转义
String.raw`a\nb\tc`; // 输出为 "a\nb\tc"
// 支持内插表达式
let name = "louis";
String.raw`Hello \n ${name}`;  // "Hello \n louis"
// 内插表达式还可以运算
String.raw`1+2=${1+2},2*3=${2*3}`; // "1+2=3,2*3=6"
```

String.raw作为函数来调用的场景不太多，如下是用法：

```
// 对象的raw属性值为字符串时，从第二个参数起，它们分别被插入到下标为0，1，2，...n的元素后面
String.raw({raw: 'abcd'}, 1, 2, 3); // "a1b2c3d"
// 对象的raw属性值为数组时，从第二个参数起，它们分别被插入到数组下标为0，1，2，...n的元素后面
String.raw({raw: ['a', 'b', 'c', 'd']}, 1, 2, 3); // "a1b2c3d"
```

那么怎么解释String.raw函数按照下标挨个去插入的特性呢？MDN上有段描述如下：

> In most cases, String.raw() is used with template strings. The first syntax mentioned above is only rarely used, because the JavaScript engine will call this with proper arguments for you, just like with other tag functions.

这意味着，String.raw作为函数调用时，基本与ES6的tag标签模板一样。如下：

```
// 如下是tag函数的实现
function tag(){
  const array = arguments[0];
  return array.reduce((p, v, i) => p + (arguments[i] || '') + v);
}
// 回顾一个simple的tag标签模板
tag`Hello ${ 2 + 3 } world ${ 2 * 3 }`; // "Hello 5 world 6"
// 其实就想当于如下调用
tag(['Hello ', ' world ', ''], 5, 6); // "Hello 5 world 6"
```

因此String.raw作为函数调用时，不论对象的raw属性值是字符串还是数组，插槽都是天生的，下标为0，1，2，...n的元素后面都是插槽（不包括最后一个元素）。实际上，它相当于是这样的tag函数：

```
function tag(){
  const array = arguments[0].raw;
  if(array === undefined || array === null){ // 这里可简写成 array == undefined
    throw new TypeError('Cannot convert undefined or null to object');
  }
  return array.reduce((p, v, i) => p + (arguments[i] || '') + v);
}
```

实际上，String.raw作为函数调用时，若第一个参数不是一个符合标准格式的对象，执行将抛出TypeError错误。

```
String.raw({123: 'abcd'}, 1, 2, 3); // TypeError: Cannot convert undefined or null to object
```

目前只有Chrome v41+和Firefox v34+版本浏览器实现了该方法。

### String.prototype

和其他所有对象一样，字符串实例的所有方法均来自String.prototype。以下是它的属性特性：

属性|参数
--|--
writable|false
enumerable|false
configurable|false

可见，字符串属性不可编辑，任何试图改变它属性的行为都将抛出错误。

属性

String.prototype共有两个属性，如下：

- String.prototype.constructor 指向构造器(String())
- String.prototype.length 表示字符串长度

方法

字符串原型方法分为两种，一种是html无关的方法，一种是html有关的方法。我们先看第一种。但是无论字符串方法如何厉害，都不至于强大到可以改变原字符串。

HTML无关的方法

常用的方法有，charAt、charCodeAt、concat、indexOf、lastIndexOf、localeCompare、match、replace、search、slice、split、substr、substring、toLocaleLowerCase、toLocaleUpperCase、toLowerCase、toString、toUpperCase、trim、valueof 等ES5支持的，以及 codePointAt、contains、endsWith、normalize、repeat、startsWith 等ES6支持的，还包括 quote、toSource、trimLeft、trimRight 等非标准的。

接下来我们将对各个方法分别举例阐述其用法。若没有特别说明，默认该方法兼容所有目前主流浏览器。

### charAt

charAt() 方法返回字符串中指定位置的字符。

语法：str.charAt(index)

index 为字符串索引（取值从0至length-1），如果超出该范围，则返回空串。

```
console.log("Hello, World".charAt(8)); // o, 返回下标为8的字符串o
```

### charCodeAt

charCodeAt() 返回指定索引处字符的 Unicode 数值。

语法：str.charCodeAt(index)

index 为一个从0至length-1的整数。如果不是一个数值，则默认为 0，如果小于0或者大于字符串长度，则返回 NaN。

Unicode 编码单元（code points）的范围从 0 到 1,114,111。开头的 128 个 Unicode 编码单元和 ASCII 字符编码一样.

charCodeAt() 总是返回一个小于 65,536 的值。因为高位编码单元需要由一对字符来表示，为了查看其编码的完成字符，需要查看 charCodeAt(i) 以及 charCodeAt(i+1) 的值。如需更多了解请参考 fixedCharCodeAt。

```
console.log("Hello, World".charCodeAt(8)); // 111
console.log("前端工程师".charCodeAt(2)); // 24037,
```

可见也可以查看中文Unicode编码

### concat

concat() 方法将一个或多个字符串拼接在一起，组成新的字符串并返回。

语法：str.concat(string2, string3, …)

console.log("早".concat("上","好")); // 早上好
但是 concat 的性能表现不佳，强烈推荐使用赋值操作符（+或+=）代替 concat。"+" 操作符大概快了 concat 几十倍。（数据参考 性能测试）。

### indexOf和lastIndexOf

indexOf() 方法用于查找子字符串在字符串中首次出现的位置，没有则返回 -1。该方法严格区分大小写，并且从左往右查找。而 lastIndexOf 则从右往左查找，其它与前者一致。

语法：`str.indexOf(searchValue [, fromIndex=0])，str.lastIndexOf(searchValue [, fromIndex=0])`

searchValue 表示被查找的字符串，fromIndex 表示开始查找的位置，默认为0，如果小于0，则查找整个字符串，若超过字符串长度，则该方法返回-1，除非被查找的是空字符串，此时返回字符串长度。

```
console.log("".indexOf("",100)); // 0
console.log("IT改变世界".indexOf("世界")); // 4
console.log("IT改变世界".lastIndexOf("世界")); // 4
```

### localeCompare

localeCompare() 方法用来比较字符串，如果指定字符串在原字符串的前面则返回负数，否则返回正数或0，其中0 表示两个字符串相同。该方法实现依赖具体的本地实现，不同的语言下可能有不同的返回。

语法：str.localeCompare(str2 [, locales [, options]])

```
var str = "apple";
var str2 = "orange";
console.log(str.localeCompare(str2)); // -1
console.log(str.localeCompare("123")); // 1
```

目前 Safari 浏览器暂不支持该方法，但Chrome v24+、Firefox v29+，IE11+ 和 Opera v15+ 都已实现了它。

### match

match() 方法用于测试字符串是否支持指定正则表达式的规则，即使传入的是非正则表达式对象，它也会隐式地使用new RegExp(obj)将其转换为正则表达式对象。

语法：str.match(regexp)

该方法返回包含匹配结果的数组，如果没有匹配项，则返回 null。

描述

若正则表达式没有 g 标志，则返回同 RegExp.exec(str) 相同的结果。而且返回的数组拥有一个额外的 input 属性，该属性包含原始字符串，另外该数组还拥有一个 index 属性，该属性表示匹配字符串在原字符串中索引（从0开始）。
若正则表达式包含 g 标志，则该方法返回一个包含所有匹配结果的数组，没有匹配到则返回 null。
相关 RegExp 方法

若需测试字符串是否匹配正则，请参考 RegExp.test(str)。
若只需第一个匹配结果，请参考 RegExp.exec(str)。

```
var str = "World Internet Conference";
console.log(str.match(/[a-d]/i)); // ["d", index: 4, input: "World Internet Conference"]
console.log(str.match(/[a-d]/gi)); // ["d", "C", "c"]
// RegExp 方法如下
console.log(/[a-d]/gi.test(str)); // true
console.log(/[a-d]/gi.exec(str)); // ["d", index: 4, input: "World Internet Conference"]
```

由上可知，RegExp.test(str) 方法只要匹配到了一个字符也返回true。而RegExp.exec(str) 方法无论正则中有没有包含 g 标志，RegExp.exec将直接返回第一个匹配结果，且该结果同 str.match(regexp) 方法不包含 g 标志时的返回一致。

### replace

该方法在之前已经讲过，详细请参考 String.prototype.replace高阶技能 。

search

search() 方法用于测试字符串对象是否包含某个正则匹配，相当于正则表达式的 test 方法，且该方法比 match() 方法更快。如果匹配成功，search() 返回正则表达式在字符串中首次匹配项的索引，否则返回-1。

> 注意：search方法与indexOf方法作用基本一致，都是查询到了就返回子串第一次出现的下标，否则返回-1，唯一的区别就在于search默认会将子串转化为正则表达式形式，而indexOf不做此处理，也不能处理正则。

语法：str.search(regexp)

```
var str = "abcdefg";
console.log(str.search(/[d-g]/)); // 3,
```

匹配到子串"defg",而d在原字符串中的索引为3

search() 方法不支持全局匹配（正则中包含g参数），如下：

```
console.log(str.search(/[d-g]/g)); // 3, 与无g参数时,返回相同
```

### slice

slice() 方法提取字符串的一部分，并返回新的字符串。该方法有些类似 Array.prototype.slice 方法。

语法：str.slice(start, end)

首先 end 参数可选，start可取正值，也可取负值。

取正值时表示从索引为start的位置截取到end的位置（不包括end所在位置的字符，如果end省略则截取到字符串末尾）。

取负值时表示从索引为 length+start 位置截取到end所在位置的字符。

```
var str = "It is our choices that show what we truly are, far more than our abilities.";
console.log(str.slice(0,-30)); // It is our choices that show what we truly are
console.log(str.slice(-30)); // , far more than our abilities.
```

### split

split() 方法把原字符串分割成子字符串组成数组，并返回该数组。

语法：str.split(separator, limit)

两个参数均是可选的，其中 separator 表示分隔符，它可以是字符串也可以是正则表达式。如果忽略 separator，则返回的数组包含一个由原字符串组成的元素。如果 separator 是一个空串，则 str 将会被分割成一个由原字符串中字符组成的数组。limit 表示从返回的数组中截取前 limit 个元素，从而限定返回的数组长度。

```
var str = "today is a sunny day";
console.log(str.split()); // ["today is a sunny day"]
console.log(str.split("")); // ["t", "o", "d", "a", "y", " ", "i", "s", " ", "a", " ", "s", "u", "n", "n", "y", " ", "d", "a", "y"]
console.log(str.split(" ")); // ["today", "is", "a", "sunny", "day"]
```

使用limit限定返回的数组大小，如下：

```
console.log(str.split(" ")); // ["today"]
```

使用正则分隔符（RegExp separator）， 如下：

```
console.log(str.split(/\s*is\s*/)); // ["today", "a sunny day"]
```

若正则分隔符里包含捕获括号，则括号匹配的结果将会包含在返回的数组中。

```
console.log(str.split(/(\s*is\s*)/)); // ["today", " is ", "a sunny day"]
```

### substr

substr() 方法返回字符串指定位置开始的指定数量的字符。

语法：str.substr(start[, length])

start 表示开始截取字符的位置，可取正值或负值。取正值时表示start位置的索引，取负值时表示 length+start位置的索引。

length 表示截取的字符长度。

```
var str = "Yesterday is history. Tomorrow is mystery. But today is a gift.";
console.log(str.substr(47)); // today is a gift.
console.log(str.substr(-16)); // today is a gift.
```

目前 Microsoft's JScript 不支持 start 参数取负的索引，如需在 IE 下支持，请参考 Polyfill。

### substring

substring() 方法返回字符串两个索引之间的子串。

语法：str.substring(indexA[, indexB])

indexA、indexB 表示字符串索引，其中 indexB 可选，如果省略，则表示返回从 indexA 到字符串末尾的子串。

描述

substring 要截取的是从 indexA 到 indexB（不包含）之间的字符，符合以下规律：

- 若 indexA == indexB，则返回一个空字符串；
- 若 省略 indexB，则提取字符一直到字符串末尾；
- 若 任一参数小于 0 或 NaN，则被当作 0；
- 若 任一参数大于 length，则被当作 length。

而 如果 indexA > indexB，则 substring 的执行效果就像是两个参数调换一般。比如：str.substring(0, 1) == str.substring(1, 0)

```
var str = "Get outside every day. Miracles are waiting everywhere.";
console.log(str.substring(1,1)); // ""
console.log(str.substring(0)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(-1)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(0,100)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(22,NaN)); // Get outside every day.
```

### toLocaleLowerCase

toLocaleLowerCase() 方法返回调用该方法的字符串被转换成小写的值，转换规则根据本地化的大小写映射。而toLocaleUpperCase() 方法则是转换成大写的值。

语法：str.toLocaleLowerCase(), str.toLocaleUpperCase()

```
console.log('ABCDEFG'.toLocaleLowerCase()); // abcdefg
console.log('abcdefg'.toLocaleUpperCase()); // ABCDEFG
```

### toUpperCase

这两个方法分别表示将字符串转换为相应的小写，大写形式，并返回。如下：

```
console.log('ABCDEFG'.toLowerCase()); // abcdefg
console.log('abcdefg'.toUpperCase()); // ABCDEFG
```

### toString和valueOf

这两个方法都是返回字符串本身。

语法：str.toString(), str.valueOf()

```
var str = "abc";
console.log(str.toString()); // abc
console.log(str.toString()==str.valueOf()); // true
```

对于对象而言，toString和valueOf也是非常的相似，它们之间有着细微的差别，请尝试运行以下一段代码：

```
var x = {
    toString: function () { return "test"; },
    valueOf: function () { return 123; }
};

console.log(x); // test
console.log("x=" + x); // "x=123"
console.log(x + "=x"); // "123=x"
console.log(x + "1"); // 1231
console.log(x + 1); // 124
console.log(["x=", x].join("")); // "x=test"
```

当 "+" 操作符一边为数字时，对象x趋向于转换为数字，表达式会优先调用 valueOf 方法，如果调用数组的 join 方法，对象x趋向于转换为字符串，表达式会优先调用 toString 方法。

### trim

trim() 方法清除字符串首尾的空白并返回。

语法：str.trim()

```
console.log(" a b c ".trim()); // "a b c"
```

trim() 方法是 ECMAScript 5.1 标准加入的，它并不支持IE9以下的低版本IE浏览器，如需支持，请参考以下兼容写法：

```
if(!String.prototype.trim) {
  String.prototype.trim = function () {
    return this.replace(/^\s+|\s+$/g,'');
  };
}
```

### codePointAt(ES6)

codePointAt() 方法基于ECMAScript 2015（ES6）规范，返回使用UTF-16编码的给定位置的值的非负整数。

语法：str.codePointAt(position)

```
console.log("a".codePointAt(0)); // 97
console.log("\u4f60\u597d".codePointAt(0)); // 20320
```

如需在老的浏览器中使用该方法，请参考 Polyfill。

### includes(ES6)

includes() 方法基于ECMAScript 2015（ES6）规范，它用来判断一个字符串是否属于另一个字符。如果是，则返回true，否则返回false。

语法：str.includes(subString [, position])

subString 表示要搜索的字符串，position 表示从当前字符串的哪个位置开始搜索字符串，默认值为0。

```
var str = "Practice makes perfect.";
console.log(str.includes("perfect")); // true
console.log(str.includes("perfect",100)); // false
```

实际上，Firefox 18~39中该方法的名称为contains，由于bug 1102219的存在，它被重命名为includes() 。目前只有Chrome v41+和Firefox v40+版本浏览器实现了它，如需在其它版本浏览器中使用该方法，请参考 Polyfill。

### endsWith(ES6)

endsWith() 方法基于ECMAScript 2015（ES6）规范，它基本与 contains() 功能相同，不同的是，它用来判断一个字符串是否是原字符串的结尾。若是则返回true，否则返回false。

语法：str.endsWith(substring [, position])

与contains 方法不同，position 参数的默认值为字符串长度。

```
var str = "Learn and live.";
console.log(str.endsWith("live.")); // true
console.log(str.endsWith("Learn",5)); // true
```

同样目前只有 Firefox v17+版本实现了该方法。其它浏览器请参考 Polyfill。

### normalize(ES6)

normalize() 方法基于ECMAScript 2015（ES6）规范，它会按照指定的 Unicode 正规形式将原字符串正规化。

语法：str.normalize([form])

form 参数可省略，目前有四种 Unicode 正规形式，即 "NFC"、"NFD"、"NFKC" 以及 "NFKD"，form的默认值为 "NFC"。如果form 传入了非法的参数值，则会抛出 RangeError 错误。

```
var str = "\u4f60\u597d";
console.log(str.normalize()); // 你好
console.log(str.normalize("NFC")); // 你好
console.log(str.normalize("NFD")); // 你好
console.log(str.normalize("NFKC")); // 你好
console.log(str.normalize("NFKD")); // 你好
```

目前只有Chrome v34+和Firefox v31+实现了它。

### repeat(ES6)

repeat() 方法基于ECMAScript 2015（ES6）规范，它返回重复原字符串多次的新字符串。

语法：str.repeat(count)

count 参数只能取大于等于0 的数字。若该数字不为整数，将自动转换为整数形式，若为负数或者其他值将报错。

```
var str = "A still tongue makes a wise head.";
console.log(str.repeat(0)); // ""
console.log(str.repeat(1)); // A still tongue makes a wise head.
console.log(str.repeat(1.5)); // A still tongue makes a wise head.
console.log(str.repeat(-1)); // RangeError:Invalid count value
```

目前只有 Chrome v41+、Firefox v24+和Safari v9+版本浏览器实现了该方法。其他浏览器请参考 Polyfill。

### startsWith(ES6)

startsWith() 方法基于ECMAScript 2015（ES6）规范，它用来判断当前字符串是否是以给定字符串开始的，若是则返回true，否则返回false。

语法：str.startsWith(subString [, position])

```
var str = "Where there is a will, there is a way.";
console.log(str.startsWith("Where")); // true
console.log(str.startsWith("there",6)); // true
```

目前以下版本浏览器实现了该方法，其他浏览器请参考 Polyfill。

Chrome|Firefox|Edge|Opera|Safari
--|--|--|--|--
41+|17+|✔️|28+|9+

其它非标准的方法暂时不作介绍，如需了解请参考 String.prototype 中标注为感叹号的方法。

HTML有关的方法

常用的方法有 anchor，link 其它方法如 big、blink、bold、fixed、fontcolor、fontsize、italics、small、strike、sub、sup均已废除。

接下来我们将介绍 anchor 和 link 两个方法，其他废除方法不作介绍。

### anchor

anchor() 方法创建一个锚标签。

语法：str.anchor(name)

name 指定被创建的a标签的name属性，使用该方法创建的锚点，将会成为 document.anchors 数组的元素。

```
var str = "this is a anchor tag";
document.body.innerHTML = document.body.innerHTML + str.anchor("anchor1"); // body末尾将会追加这些内容 <a name="anchor1">this is a anchor tag</a>
```

### link

link() 方法同样创建一个a标签。

语法：str.link(url)

url 指定被创建的a标签的href属性，如果url中包含特殊字符，将自动进行编码。例如 " 会被转义为 &\quot。 使用该方法创建的a标签，将会成为 document.links 数组中的元素。

var str = "百度";
document.write(str.link("https://www.baidu.com")); // <a href="https://www.baidu.com">百度</a>
小结

部分字符串方法之间存在很大的相似性，要注意区分他们的功能和使用场景。如：

- substr 和 substring，都是两个参数，作用基本相同，两者第一个参数含义相同，但用法不同，前者可为负数，后者值为负数或者非整数时将隐式转换为0。前者第二个参数表示截取字符串的长度，后者第二个参数表示截取字符串的下标；同时substring第一个参数大于第二个参数时，执行结果同位置调换后的结果。
- search方法与indexOf方法作用基本一致，都是查询到了就返回子串第一次出现的下标，否则返回-1，唯一的区别就在于search默认会将子串转化为正则表达式形式，而indexOf不做此处理，也不能处理正则。
另外，还记得吗？concat方法由于效率问题，不推荐使用。

通常，字符串中，常用的方法就charAt、indexOf、lastIndexOf、match、replace、search、slice、split、substr、substring、toLowerCase、toUpperCase、trim、valueof 等这些。熟悉它们的语法规则就能熟练地驾驭字符串。