[toc]

## 介绍

在javascript中, 数据类型主要分为原始类型和引用类型两种。而一切引用类型都来自于Object的拷贝。所有引用类型的原型链都可以追溯到 Object

## Object 构造函数属性

JavaScript 内置的一些构造函数有 Object, Function, Number, String, Boolean, Array, RegExp 等等, 它们主要有两个共有的属性。

- length 构造函数参数个数
- prototype 构造函数原型对象

## Object原型链

- Object.getPrototypeOf
- Object.isPrototypeOf
- Object.hasOwnProperty

一切引用对象的原型都来自 Object.prototype
测试各个数据类型的引用情况

```
var a = {},
    b = [],
    c = function () {},
    d = Function,
    e = String;
[a, b, c, d, e].forEach(function (val) {
    // all return true
    console.log(val instanceof Object);
});
```

每一个引用类型对象, 都含有一个原型属性`__proto__`, 它负责控制对象的原型属性和方法的查询使用

## 创建无Object.prototype的原型链对象

```
// method 1
var obj1 = Object.create(null);

// method 2
var obj2 = {};
Object.setPrototypeOf(obj2, null);

// method 3
var obj3 = {};
obj3.__proto__ = null;

[obj1, obj2, obj3].forEach(function (item) {
  console.log(item instanceof Object); // false
});
```

## __proto__ 与 prototype
`__proto__` 隐式原型， prototype 显示原型. 
实例对象通过隐式原型`__proto__` 匹配找到对应的函数和属性. 而prototype是每一个构造函数都存在的一个属性。其中prototype 包含 constructor属性

```
var o = {};
'__proto__' in o; // return true, 说明 字面量对象存在一个隐式属性

// 指定隐式属性
function Foo () {}
o.__proto__ = Foo.prototype;

o instanceof Foo; // return true
o instanceof Object; // return true
```

## 设置隐藏原型属性

```
var o = {};
'__proto__' in o;

function Foo () {}
function Bar () {}
Bar.prototype = new Foo();
Bar.prototype.constructor = Bar;

// 方法一, 直接设置 __proto__值
o.__proto__ = Foo.prototype;
console.log(o instanceof Foo); // return true;

// 方法二, 使用 setPrototypeOf
Object.setPrototypeOf(o, Bar.prototype); // 设置新原型
console.log(o instanceof Bar); // return true;

// 获取对象隐式属性
Object.getPrototypeOf(o) === Bar.prototype; // return true

// 检查原型 是否存在于对象属性的链中
Bar.prototype.isPrototypeOf(o); // true
Foo.prototype.isPrototypeOf(o); // true
Object.prototype.isPrototypeOf(o); // true
```

## 获取对象的实例属性

- Object.keys
- Object.getOwnPropertyNames

```
// return []
Object.keys(Object.prototype)

// return ["constructor", "__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "isPrototypeOf", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "toLocaleString"]
Object.getOwnPropertyNames(Object.prototype)
```

Object.keys 返回一个对象的实例可枚举属性, 如果使用了Object.defineProperty改变了对象的某个属性, 则无法通过Object.keys返回属性进行遍历属性, 也无法使用 for-in循环。

```
var obj = {
    foo: function () {}
};

// return ['foo']
Object.keys(obj);

Object.defineProperty(obj, 'foo', {
   enumerable: false
});

// return []
Object.keys(obj);

// empty loop
for (var name in obj) {
  console.log(name);
}

// return false
obj.hasOwnProperty('foo');
```

Object.getOwnPropertyNames() 返回自身实例属性名称, 无视enumerable: false

```
var obj = {
    foo: function () {},
    bar: 'hello world'
};

Object.defineProperty(obj, 'foo', {
    // foo 已被定义, 所以需要显示设置 false
    enumerable: false
});

Object.defineProperty(obj, 'foo2', {
    value: function () {},
    // foo2 未被定义, 默认enumerable为false
    enumerable: true
});

// 'bar', 'foo2'
Object.keys(obj);

// 'foo', 'bar', 'foo2'
Object.getOwnPropertyNames(obj);

'foo' in obj // return true
```

in 操作符, 检查属性是否在能够获取, 无视属性

## 对象定义属性

- Object.defineProperty
- Object.getOwnPropertyDescriptor

属性描述一般是由enumrable, value, get, set, configurable, writeable, value 组成， 其中 get, set 与 value 为互斥关系

## 定义一个对象的描述

```
var obj = Object.create(null);

Object.defineProperty(obj, 'foo', {
    value: 'foo',
    configurable: false,
    writeable: false,
    enumerable: false
});

obj.foo = 'change foo';
console.log(obj.foo); // still foo

Object.defineProperty(obj, 'foo', {
    writeable: true,
    configurable: true
});

obj.foo = 'change foo 2';
console.log(obj.foo); // still foo
'foo' in obj; // return true
```

一旦被定义了configureable: false, 那么后续再次 defineProperty时不会生效。
writeable: false 的情况下, 无法修改属性的 value 值

## 属性描述中的 value 与 get, set

```
var obj = Object.create(null);

// throw Error 
// Uncaught TypeError: Invalid property descriptor. 
// Cannot both specify accessors and a value or writable attribute
Object.defineProperty(obj, 'foo', {
    value: 'foo',
    set: function (val) {
        // this 指向 obj
        this._foo = val;
    },
    get: function () {
        return this._foo;
    }
});
```

因为 value 与 get, set 为互斥关系, 所以无法同时进行定义。

## writeable 与 get, set

```
var obj = Object.create(null);

Object.defineProperty(obj, 'foo', {
    // 失效
    writeable: false,
    set: function (val) {
        // this 指向 obj
        this._foo = val;
    },
    get: function () {
        return this._foo;
    }
});

obj.foo = 'abc'; // set()
obj._foo === obj.foo  // return true
console.log(obj.foo); // return 'abc'
'foo' in obj // return true
```

writeable 失效, 无法对属性做任何限制

## 重复定义对象的属性

```
var obj = Object.create(null);

Object.defineProperty(obj, 'foo', {
    configurable: false,
    value: 'foo'
});

// Uncaught TypeError: Cannot redefine property: foo
Object.defineProperty(obj, 'foo', {
    value: 'foo2'
});
```

一旦定义了configurable: false以后, 不允许再次定义 descriptor

## 获取对象的属性描述

```
var obj = Object.create(null);

Object.defineProperty(obj, 'foo', {
    configurable: true,
    value: 'foo'
});
var descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');

console.log(descrptor); // {value: "foo", writable: false, enumerable: false, configurable: false}

// {foo: {value: "foo", writable: false, enumerable: false, configurable: false}}
Object.getOwnPropertyDescriptors(obj);
```

## 对象拷贝

### 浅拷贝

- Object.assign

将候选的对象里的可枚举属性进行引用赋值, 支持多个候选的对象传递

```
var o = {
  foo: 'foo',
  bar: 'bar'
};

Object.defineProperty(o, 'foo', {
  enumerable: false
});

var obj = Object.assign(Object.create(null), o, {
  o: o
});

/*
obj = {
  bar: 'bar',
  o: {
    'foo': 'foo',
    'bar': 'bar'
  }
}
*/
obj.o === o; // return true;
```

由于Object.assign 处于浅拷贝的关系, 所以返回的key 都为简单的引用方式.

### 深度拷贝

#### 使用 对象系列化方式copy数据格式

```
// 该方法只能拷贝基本数据类型
var obj = {a: 1, b: 2, c: function () { console.log('hello world');}, d: {e: 3, f: 4}};

JSON.parse(JSON.stringify(obj));
```

#### 自行实现深度拷贝

```
function deepClone (obj) {
  var newObj;
  var isPlainObject = function (o) {
    return Object.prototype.toString.call(o) === '[object Object]';
  };
  var isArray = function (o) {
    return Object.prototype.toString.call(o) === '[object Array]';
  };
  if (isArray(obj)) {
    newObj = [];
    for (var i = 0, len = obj.length; i < len; i++) {
      newObj.push(deepClone(obj[i]));
    }
  } else if (isPlainObject(obj)) {
    newObj = {};
    for (var key in obj) {
      newObj[key] = deepClone(obj[key]);
    }
  } else {
    newObj = obj;
  }
  return newObj;
}

var o = {a: 1, b: [{c: 2, d: 3}]};
var o2 = deepClone(o);
o.b[0] !== o2.b[0]; // return true
```

深度拷贝对象函数, 内置的对象和数组都被完整的拷贝出来。

#### 循环引用拷贝问题

```
var o = {a: 1, b: 2};

// 循环引用问题
o.foo = o;

// Uncaught TypeError: Converting circular structure to JSON
JSON.stringify(o);

// Uncaught RangeError: Maximum call stack size exceeded
deepClone(o);

// 将循环引用的key设置为不可枚举型
Object.defineProperty(o, 'foo', {enumerable: false});
// OK {"a":1,"b":2}
JSON.stringify(o);
```

#### 避免重复循环引用的深度clone

```
function deepClone (obj) {

  var objStack = [];

  var isPlainObject = function (o) {
    return Object.prototype.toString.call(o) === '[object Object]';
  };

  var isArray = function (o) {
    return Object.prototype.toString.call(o) === '[object Array]';
  };
  
  var isError = function (o) {
    return o instanceof Error;
  };

  function _deepClone (obj) {
    var newObj, cloneObj;
    if (isArray(obj) || isPlainObject(obj)) {
      // 对象重复引用
      if (objStack.indexOf(obj) === -1) {
        objStack.push(obj);
      } else {
        return new Error('parameter Error. it is exits loop reference');
      }
    }
    if (isArray(obj)) {
      newObj = [];
      for (var i = 0, len = obj.length; i < len; i++) {
        cloneObj = _deepClone(obj[i]);
        if (!isError(cloneObj)) {
          newObj.push(cloneObj);
        }
      }
    } else if (isPlainObject(obj)) {
      newObj = {};
      for (var key in obj) {
        cloneObj = _deepClone(obj[key]);
        if (!isError(cloneObj)) {
          newObj[key] = cloneObj;
        }
      }
    } else {
      newObj = obj;
    }
    return newObj;
  }
  return _deepClone(obj);
}
```

## Object.create 创建对象实例

一般创建对象有三种方式

- 字面量方式 var o = {};
- 构造函数方式 var o = new Object();
- Object.create();

Object.create 与 字面量创建的区别

Object.create 可以指定创建的对象的隐式原型链 `__proto__`, 也可以创建空原型链的的对象

```
var prototype = {
  foo: 'foo',
  name: 'prototoype'
};
var o = Object.create(prototype);
var o2 = Object.create(null);
o.__proto__ === prototype; // true

'__protot__' in o2; // false
```

Object.create 与构造函数的区别

Object.create 直接进行开辟对象空间, 绑定隐式原型链属性。而构造函数创建对象除了上面的操作以外, 还会运行构造函数。

利用 Object.create 与构造函数, 实现继承对象关系

```
// Shape - superclass
function Shape() {
  this.x = 0;
  this.y = 0;
}

// superclass method
Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};

// Rectangle - subclass
function Rectangle() {
  Shape.call(this); // call super constructor.
}

// subclass extends superclass
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();

console.log('Is rect an instance of Rectangle?',
  rect instanceof Rectangle); // true
console.log('Is rect an instance of Shape?',
  rect instanceof Shape); // true
rect.move(1, 1); // Outputs, 'Shape moved.'
```