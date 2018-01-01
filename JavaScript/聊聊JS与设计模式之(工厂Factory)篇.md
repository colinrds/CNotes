[toc]

## 总体概要

1. 笔者浅谈

说起设计模式其实并不是什么很新奇的概念，它也不是基于特定语言所形成的产物，它是基于软件设计原则以及相关的方法论和经过特定时期衍生出的若干解决方案。本文会以一个实例带入大家学习设计模式以及与之对应的软件设计原则来从不同角度阐述和分析代码，并且会给大家提供一些第三方的的实现方式来供参考，还会拿出开源代码案例来更深入的理解如何使用设计模式。

2. 直入话题工厂模式是我们最常用的模式了,著名的前端框架Bootstrap以及服务端基于NodeJS的框架(ExpressJS) ,就使用了工厂模式，工厂模式在JS程序中可以说是随处可见。为什么工厂模式是如此常用？因为工厂模式就相当于创建实例对象的new，我们经常要根据类生成实例对象，如var button = new Button({}); 工厂模式也是用来创建实例对象的，所以以后当我们通过new创建对象时就可以思考一下是否可以用工厂模式来代替，可能多做一些工作，但会给你系统带来更大的可扩展性和尽量少的修改量。

简单工厂------简单工厂的特点就是参数化创建对象，简单工厂必须知道每一种产品以及何时提供给客户端。有人会说简单工厂还是换汤不换药，添加新类的时候还是需要修改这部分的代码！那么我们获得了什么好处呢？集中变化！ 这很好的符合了DRY原则，DRY———Don't Repeat Yourself Principle，直译为“不要重复自己”原则^_^ 创建逻辑存放在单一的位置，即使它变化，我们也只需要修改一处就可以了。DRY 很简单，但却是确保我们代码容易维护和复用的关键。DRY原则同时还提醒我们：对系统职能进行良好的分割！职责清晰的界限一定程度上保证了代码的单一性。这句话对我们后续的分析极具指导意义，毕竟简单工厂只是低层次上的代码复用，以下是一个简单的例子

```
var FullTime = function () {
    this.hourly = "$12";
};

var PartTime = function () {
    this.hourly = "$11";
};

var Temporary = function () {
    this.hourly = "$10";
};

var Contractor = function () {
    this.hourly = "$15";
};

function Factory() {
    this.createEmployee = function (type) {
        var employee;
        if (type === "fulltime") {
            employee = new FullTime();
        } else if (type === "parttime") {
            employee = new PartTime();
        } else if (type === "temporary") {
            employee = new Temporary();
        } else if (type === "contractor") {
            employee = new Contractor();
        }
        employee.type = type;
        employee.say = function () {
            console.log(this.type + ": rate " + this.hourly + "/hour");
        }
        return employee;
    };
}
var factory = new Factory();
factory.createEmployee("fulltime").say();
```

![image](http://images.cnitblog.com/blog/703832/201412/221052532182875.png)
 

抽象工厂------抽象工厂向客户端提供了一个接口，使得客户端在不指定具体产品类型的时候就可以创建产品中的产品对象。这就是抽象工厂的用意。抽象工厂面的问题是多个等级产品等级结构的系统设计。抽象工厂和工厂方法模式最大的区别就在于后者只是针对一个产品等级结构；而抽象工厂则是面对多个等级结构。同样出色的完成了把应用程序从特定的实现中解耦，工厂方法使用的方法是继承，而抽象工厂使用的对象组合。抽象工厂提供的是一个产品家族的抽象类型，这个类型的子类完成了产品的创建。以下是一个简单的例子

```
function Employee(name) {
    this.name = name;
    this.say = function () {
        console.log("I am employee " + name);
    };
}

function EmployeeFactory() {
    this.create = function (name) {
        return new Employee(name);
    };
}

function Vendor(name) {
    this.name = name;
    this.say = function () {
        console.log("I am vendor " + name);
    };
}

function VendorFactory() {
    this.create = function (name) {
        return new Vendor(name);
    };
}

var employeeFactory = new EmployeeFactory();
employeeFactory.create("BigBear");
```

![image](http://images.cnitblog.com/blog/703832/201412/221103011401452.png)

在实际的的使用中，抽闲产品和具体产品之间往往是多层次的产品结构，正如上图所示

题外话：简单工厂的确简单但是其背后的DRY原则在实践中让我们受益匪浅，去年我和我的搭档做站点的升级工作，写了很多重复的代码；代码重复，源代码组织混乱，没有做好规划和职责分析是原罪。今年新项目，DRY原则是我头顶的达摩克利斯之剑，不做重复的事情成为我进行项目计划组织管理的重要标准。

关于工厂模式的阶段总结：

- 识别变化隔离变化，简单工厂是一个显而易见的实现方式
- 简单工厂将创建知识集中在单一位置符合了DRY
- 客户端无须了解对象的创建过程，某种程度上支持了OCP
- 添加新的产品会造成创建代码的修改，这说明简单工厂模式对OCP支持不够
- 简单工厂类集中了所有的实例创建逻辑很容易违反高内聚的责任分配原则。

## 源码案例参考

1，ExpressJS

见下图

![image](http://images.cnitblog.com/blog/703832/201412/221114139684865.png)

这是esxpress.js文件中创建app的代码，不难看出这就是一个典型的工厂模式，简单工厂。

![image](http://images.cnitblog.com/blog/703832/201412/221115445933012.png)

这是application.js文件中的application的初始化代码。

这样就很容易看出来 TJ 大神（expressjs的作者）的用意了哈哈哈。好了我再来总结一下：

初始化工作如果是很长一段代码，说明要做的工作很多，将很多工作装入一个方法中，相当于将很多鸡蛋放在一个篮子里，是很危险的，这也是有背于面向对象的原则，面向对象的封装(Encapsulation)和分派(Delegation)告诉我们，尽量将长的代码分派"切割"成每段，将每段再"封装"起来(减少段和段之间偶合联系性)，这样，就会将风险分散，以后如果需要修改，只要更改每段，不会再发生牵一动百的事情，最重要的一句话是将创建与初始化的工作进行分离

## 案例引入------麦当劳的例子

![image](http://images.cnitblog.com/blog/703832/201412/221122199379432.png)

1. 创建麦香鸡类

```
function McChicken() {
    this.getFood = function () {
        console.log("我来一份麦香鸡!");
    };
};
``` 

2. 创建薯条类

```
function Potato() {
    this.getFood = function () {
        console.log("我来一份薯条!");
    };
};
```

3. 创建食物工厂类

```
function FoodFactory() {
    return {
        create: function (type) {
            var food;
            if (type === "McChicken") {
                food = new McChicken();
            } else if (type === "Potato") {
                food = new Potato();
            }
            return food;
        };
    };
};
```

4. 创建客户端测试

```
function MClient() {
    var food = FoodFactory().create("McChicken");
    food.getFood();
};
```

现在如果我要在吃食物时前洗手如何办，需要做洗手的工作

```
function createFood() {
    // doWash(); 洗手工作
    return FoodFactory.create("McChicken");
};
```

## 总结一下

1. 我们需要将创建实例的工作与使用实例的工作分开
2. 封装(Encapsulation)和分派(Delegation)告诉我们，尽量将长的代码分派"切割"成每段，将每段再"封装"起来(减少段和段之间偶合联系性)，这样，就会将风险分散，以后如果需要修改，只要更改每段，不会再发生牵一动百的事情。