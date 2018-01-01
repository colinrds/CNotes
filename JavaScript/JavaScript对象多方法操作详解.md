## OOP（封装 + 继承 + 多态 + 抽象）OOP

一种程序设计范型，同时也是一种程序开发方法。对象指的是类的实例。它将对象作为程序的基本单元，将程序和数据封装其中，以提高软件的重用性，灵活性和扩展性。
```
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype.hi = function() {
    console.log('Hi Iam' + this.name + 'I am' + this.age + 'years old now')
}

Person.prototype.LEGS_NUM = 2;
Person.prototype.ARMS_NUM = 2;
Person.prototype.walk = function() {
    console.log(this.nama + 'is walking')
}
function Student(name, age, className {
    Person.call(this, name, age);
    this.className = className
}

Student.prototype = Object.create(Person.prototype); //
Student.prototype.constructor = Student; 
Student.prototype.hi = function() {
    console.log("Hi" + this.name + ',I am' + this.age + 'years old ,and from' + this.className + '.')
}
Student.prototype.learn = function(subject) {
    console.log('my major is' + subject)
}
//test
var tom = new Student('Tom', 22, 'Class 3'); 
tom.hi(); 
tom.LEGS_NUM; 
tom.walk(); 
tom.learn();

```

## prototype属性
```
Student.prototype.x = 101; tom.x; //101
Student.prototype = {
    y: 3
}; tom.y; //undefined
tom.x; //101
var jack = new Student('Jack', 22, 'Class 1'); 
jack.x; //undefined
jack.y; //2
```
Thus: 
- 动态修改prototype对象的属性, 会影响创建及已创建的实例
- 修改整个prototype对象，只影响后续创建的实例
- 内置构造器的prototype 

```
Object.prototype.x = 1;
var obj = {}; obj.x; //1
for (var key in obj) {
    console.log(key)
} //x
//ES5
Object.defineProperty(Object.prototype, {
    writable: true,
    value: 1
}) var obj = {}; obj.x; //1
for (var key in obj) {
    console.log(key)
} //nothing
```

## 实现继承的方式

```
function Person() {}
function Student() {}
Student.prototype = Person.prototype; //1 bad
Student.prototype = new Person; //2===4
Student.prototype = Object.create(Person.prototype); //3,ES5
//check
if (!Object.create) {
    Object.create = function(proto) {
        function F() {}
        F.prototype = proto;
        return new F;
    }
}
Student.prototype.constructor = Person; //4
```

## 模拟重载，链式调用，模块化

模拟重载
```
function Person() {
    var args = arguments;
    if (typeof args[0] === 'object' && args[0]) {
        if (args[0].name) {
            this.name = args[0].name
        }
        if (args[0].age) {
            this.age = args[0].age
        } else {
            if (args[0]) {
                this.name = args[0]
            }
            if (args[1]) {
                this.age = args[1]
            }
        }
    }
}
Person.prototype.toString = function() {
    return 'name=' + this.name + ',age' + this.age;
}
var tom = new Person('tom', 28);
var jack = new Person({
    name: 'Jack',
    age: 23
})
```

## 调用子类的方法

```
function Person(name) {
    this.name = name;
}
function Student(name, className) {
    this.className = className;
    Person.call(this, name);
}
var tom = new Student('tom', 'NT'); 
Person.prototype.init = function() {}；
Student.prototypt.init = function() {
    Person.prototype.init.apply(this, arguments)
};
```

## 链式调用
```
function ClassManager() {
    ClassManager.prototype.addClass = function(str) {
        console.log('Class' + str + 'added');
        return this; //实现链式调用
    }
}
var manager = new ClassManager(); manager.addClass('classA').addClass('classB').addClass('classC')
```

## 抽象类
```
function DetectorBase() {
    throw new Error('Abstract class can not be invoked directly!')
}
DetectorBase.detect = function() {
    console.log('Detector starting...')
}; 
DetectorBase.stop = function() {
    console.log('Detector stopped.')
}; 
DetectorBase.init = function() {
    throw new Error('Error')
};
function LinkDetector() {}
LinkDetector.prototype = Object.create(Detector.prototype); 
LinkDetector.prototype.constructor = LinkDetector;
//...add methods to LinkDetector...
```

## defineProperty(ES5) 

```
function Person(name) {
    Obect.defineProperty(this, 'name', {
        value: name,
        enumerable: true
    });
}
Object.defineProperty(Person, "ARMS_NUM", {
    value: 2,
    enumerable: true
}); 
Object.seal(Person.prototype); 
Object.seal(Person);
function Student(name, className) {
    this.className = className;
    Person.call(this, name)
}
Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;
```

## 模块化
```
var moduleA; 
moduleA = function() {
    var prop = 1;
    function func() {
        return {
            func: func,
            prop: prop
        }
    }
}
var moduleA; 
moduleA = new function() {
    var prop = 1;
    function func() {}
    this.func = func;
    this.prop = prop;
}
```
