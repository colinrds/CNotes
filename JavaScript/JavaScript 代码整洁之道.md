[toc]

## 概述

一张幽默的图片：软件质量通过你在阅读代码的时候有多少报怨来进行评估

![image](http://olphpb1zg.bkt.clouddn.com/20170109212946985923.JPG)

Robert C. Martin 在 《代码整洁之道》 中提到的软件工程原则，同样适用于 JavaScript。这不是一个风格参考。它指导如何用 JavaScript 编写可读、可复用、可重构的软件。

并不是每一个原则都必须严格遵循，甚至很少得到大家的认同。它们仅用于参考，不过要知道这些原则都是《代码整洁之道》的作者们累积多年的集体经验。

我们在软件工程方面的技术发展刚刚超过 50 年，我们仍然在学习很多东西。当软件架构和架构本身一样古老的时候，我们应该遵循更为严格规则。现在，对于你和你的团队编写的 JavaScript 代码，不妨依据这些准则来进行质量评估。

还有一件事：知道这些不会马上让你成为更好的软件开发者，在工作中常年使用这些准则不能让你避免错误。每一段代码都从最初的草图开始到最终成型，就像为湿粘土塑形一样。最后，当我们与同行一起审查的时候，再把不完美的地方消除掉。不要因为初稿需要改善而否定自己，需要要否定的只是那些代码！

## 变量

### 使用有准确意义的变更名

不好:
```
var yyyymmdstr = moment().format('YYYY/MM/DD');
```

好:

```
var yearMonthDay = moment().format('YYYY/MM/DD');
```

### 在变量的值不会改变时使用 ES6 的常量

在不好的示例中，变量可以被改变。如果你申明一个常量，它会在整个程序中始终保持不变。

不好:

```
var FIRST_US_PRESIDENT = "George Washington";
```

好:

```
const FIRST_US_PRESIDENT = "George Washington";
```

### 对同一类型的变量使用相同的词汇

不好:

```
getUserInfo();
getClientData();
getCustomerRecord();
```

好:

```
getUser();
```

### 使用可检索的名称

我们阅读的代码永远比写的折。写可读性强、易于检索的的代码非常重要。在程序中使用无明确意义的变量名会难以理解，对读者造成伤害。所以，把名称定义成可检索的。

不好:

```
// 见鬼，525600 是个啥？
for (var i = 0; i < 525600; i++) {
  runCronJob();
}
```

好:

```
// 用 `var` 申明为大写的全局变量
var MINUTES_IN_A_YEAR = 525600;
for (var i = 0; i < MINUTES_IN_A_YEAR; i++) {
  runCronJob();
}
```

### 使用解释性的变量

不好:

```
const cityStateRegex = /^(.+)[,\\s]+(.+?)\s*(\d{5})?$/;
saveCityState(cityStateRegex.match(cityStateRegex)[1], cityStateRegex.match(cityStateRegex)[2]);
```

好:

```
const cityStateRegex = /^(.+)[,\\s]+(.+?)\s*(\d{5})?$/;
const match = cityStateRegex.match(cityStateRegex)
const city = match[1];
const state = match[2];
saveCityState(city, state);
```

### 避免暗示

显式优于隐式。

不好:

```
var locations = ['Austin', 'New York', 'San Francisco'];
locations.forEach((l) => {
  doStuff();
  doSomeOtherStuff();
  ...
  ...
  ...
  // 等等，`l` 又是什么？
  dispatch(l);
});
```

好:

```
var locations = ['Austin', 'New York', 'San Francisco'];
locations.forEach((location) => {
  doStuff();
  doSomeOtherStuff();
  ...
  ...
  ...
  dispatch(location);
});
```

### 不要添加没必要的上下文

如果你的类名称/对象名称已经说明了它们是什么，不要在(属性)变量名里重复。

不好:

```
var Car = {
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue'
};

function paintCar(car) {
  car.carColor = 'Red';
}
```

好:

```
var Car = {
  make: 'Honda',
  model: 'Accord',
  color: 'Blue'
};

function paintCar(car) {
  car.color = 'Red';
}
```

### 短路语法比条件语句更清晰

不好:

```
function createMicrobrewery(name) {
  var breweryName;
  if (name) {
    breweryName = name;
  } else {
    breweryName = 'Hipster Brew Co.';
  }
}
```

好:

```
function createMicrobrewery(name) {
  var breweryName = name || 'Hipster Brew Co.'
}
```

## 函数

### 函数参数 (理论上少于等于2个)

限制函数参数的数量极为重要，它会让你更容易测试函数。超过3个参数会导致组合膨胀，以致于你必须根据不同的参数对大量不同的情况进行测试。

理想情况下是没有参数。有一个或者两个参数也还好，三个就应该避免了。多于那个数量就应该考虑合并。通常情况下，如果你有多于2个参数，你的函数会尝试做太多事情。如果不是这样，大多数时候可以使用一个高阶对象作为参数使用。

既然 JavaScript 允许我们在运行时随意创建对象，而不需要预先定义样板，那么你在需要很多参数的时候就可以使用一个对象来处理。

不好:

```
function createMenu(title, body, buttonText, cancellable) {
  ...
}
```

好:

```
var menuConfig = {
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
}

function createMenu(menuConfig) {
  ...
}
```

### 一个函数只做一件事

目前这是软件工程中最重要的原则。如果函数做了较多的事情，它就难以组合、测试和推测。当你让函数只做一件事情的时候，它们就很容易重构，而且代码读起来也会清晰得多。你只需要遵循本指南的这一条，就能领先于其他很多开发者。

不好:

```
function emailClients(clients) {
  clients.forEach(client => {
    let clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

好:

```
function emailClients(clients) {
  clients.forEach(client => {
    emailClientIfNeeded(client);
  });
}

function emailClientIfNeeded(client) {
  if (isClientActive(client)) {
    email(client);
  }
}

function isClientActive(client) {
  let clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

### 函数名称要说明它做的事

不好:

```
function dateAdd(date, month) {
  // ...
}

let date = new Date();

// 很难从函数名了解到加了什么
dateAdd(date, 1);
```

好:

```
function dateAddMonth(date, month) {
  // ...
}

let date = new Date();
dateAddMonth(date, 1);
```

### 函数应该只抽象一个层次

如果你有多个层次的抽象，那么你的函数通常做了太多事情，此时应该拆分函数使其易于复用和易于测试。

不好:

```
function parseBetterJSAlternative(code) {
  let REGEXES = [
    // ...
  ];

  let statements = code.split(' ');
  let tokens;
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    })
  });

  let ast;
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  })
}
```

好:

```
function tokenize(code) {
  let REGEXES = [
    // ...
  ];

  let statements = code.split(' ');
  let tokens;
  REGEXES.forEach((REGEX) => {
    statements.forEach((statement) => {
      // ...
    })
  });

  return tokens;
}

function lexer(tokens) {
  let ast;
  tokens.forEach((token) => {
    // lex...
  });

  return ast;
}

function parseBetterJSAlternative(code) {
  let tokens = tokenize(code);
  let ast = lexer(tokens);
  ast.forEach((node) => {
    // parse...
  })
}
```

### 删除重复代码

任何情况下，都不要有重复的代码。没有任何原因，它很可能是阻碍你成为专业开发者的最糟糕的一件事。重复代码意味着你要修改某些逻辑的时候要修改不止一个地方的代码。JavaScript 是弱类型语句，所以它很容易写通用性强的函数。记得利用这一点！

不好:

```
function showDeveloperList(developers) {
  developers.forEach(developers => {
    var expectedSalary = developer.calculateExpectedSalary();
    var experience = developer.getExperience();
    var githubLink = developer.getGithubLink();
    var data = {
      expectedSalary: expectedSalary,
      experience: experience,
      githubLink: githubLink
    };

    render(data);
  });
}

function showManagerList(managers) {
  managers.forEach(manager => {
    var expectedSalary = manager.calculateExpectedSalary();
    var experience = manager.getExperience();
    var portfolio = manager.getMBAProjects();
    var data = {
      expectedSalary: expectedSalary,
      experience: experience,
      portfolio: portfolio
    };

    render(data);
  });
}
```

好:

```
function showList(employees) {
  employees.forEach(employee => {
    var expectedSalary = employee.calculateExpectedSalary();
    var experience = employee.getExperience();
    var portfolio;

    if (employee.type === 'manager') {
      portfolio = employee.getMBAProjects();
    } else {
      portfolio = employee.getGithubLink();
    }

    var data = {
      expectedSalary: expectedSalary,
      experience: experience,
      portfolio: portfolio
    };

    render(data);
  });
}
```

### 使用默认参数代替短路表达式

不好:

```
function writeForumComment(subject, body) {
  subject = subject || 'No Subject';
  body = body || 'No text';
}
```

好:

```
function writeForumComment(subject = 'No subject', body = 'No text') {
  ...
}
```

### 用 Object.assign 设置默认对象

不好:

```
var menuConfig = {
  title: null,
  body: 'Bar',
  buttonText: null,
  cancellable: true
}

function createMenu(config) {
  config.title = config.title || 'Foo'
  config.body = config.body || 'Bar'
  config.buttonText = config.buttonText || 'Baz'
  config.cancellable = config.cancellable === undefined ? config.cancellable : true;

}

createMenu(menuConfig);
```

好:

```
var menuConfig = {
  title: 'Order',
  // User did not include 'body' key
  buttonText: 'Send',
  cancellable: true
}

function createMenu(config) {
  config = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config);

  // 现在 config 等于: {title: "Foo", body: "Bar", buttonText: "Baz", cancellable: true}
  // ...
}

createMenu(menuConfig);
```

不要把标记用作函数参数

标记告诉你的用户这个函数做的事情不止一件。但是函数应该只做一件事。如果你的函数中会根据某个布尔参数产生不同的分支，那就拆分这个函数。

不好:

```
function createFile(name, temp) {
  if (temp) {
    fs.create('./temp/' + name);
  } else {
    fs.create(name);
  }
}
```

好:

```
function createTempFile(name) {
  fs.create('./temp/' + name);
}

function createFile(name) {
  fs.create(name);
}
```

### 避免副作用

如果一个函数不是获取一个输入的值并返回其它值，它就有可能产生副作用。这些副作用可能是写入文件、修改一些全局变量，或者意外地把你所有钱转给一个陌生人。

现在你确实需要在程序中有副作用。像前面提到的那样，你可能需要写入文件。现在你需要做的事情是搞清楚在哪里集中完成这件事情。不要使用几个函数或类来完成写入某个特定文件的工作。采用一个，就一个服务来完成。

关键点是避免觉的陷阱，比如在没有结构的对象间共享状态，使用可以被任意修改的易变的数据类型，没有集中处理发生的副作用等。如果你能做到，你就能比其他大多数程序员更愉快。

不好:

```
// 下面的函数使用了全局变量。
// 如果有另一个函数在使用 name，现在可能会因为 name 变成了数组而不能正常运行。
var name = 'Ryan McDermott';

function splitIntoFirstAndLastName() {
  name = name.split(' ');
}

splitIntoFirstAndLastName();

console.log(name); // ['Ryan', 'McDermott'];
```

好:

```
function splitIntoFirstAndLastName(name) {
  return name.split(' ');
}

var name = 'Ryan McDermott'
var newName = splitIntoFirstAndLastName(name);

console.log(name); // 'Ryan McDermott';
console.log(newName); // ['Ryan', 'McDermott'];
```

### 不要写入全局函数

JavaScript 中全局污染是一件糟糕的事情，因为它可能和另外库发生冲突，然而使用你 API 的用户却不会知道——直到他们在生产中遇到一个异常。来思考一个例子：你想扩展 JavaScript 的原生 Array，使之拥有一个 diff 方法，用来展示两数据之前的区别，这时你会怎么做？你可以给 Array.prototype 添加一个新的函数，但它可能会与其它想做同样事情的库发生冲突。如果那个库实现的 diff 只是比如数组中第一个元素和最后一个元素的异同会发生什么事情呢？这就是为什么最好是使用 ES6 的类语法从全局的 Array 派生一个类来做这件事。

不好:

```
Array.prototype.diff = function(comparisonArray) {
  var values = [];
  var hash = {};

  for (var i of comparisonArray) {
    hash[i] = true;
  }

  for (var i of this) {
    if (!hash[i]) {
      values.push(i);
    }
  }

  return values;
}
```

好:

```
class SuperArray extends Array {
  constructor(...args) {
    super(...args);
  }

  diff(comparisonArray) {
    var values = [];
    var hash = {};

    for (var i of comparisonArray) {
      hash[i] = true;
    }

    for (var i of this) {
      if (!hash[i]) {
        values.push(i);
      }
    }

    return values;
  }
}
```

### 喜欢上命令式编程之上的函数式编程

如果 Haskell 是 IPA 那么 JavaScript 就是 O'Douls。就是说，与 Haskell 不同，JavaScript 不是函数式编程语言，不过它仍然有一点函数式的意味。函数式语言更整洁也更容易测试，所以你最好能喜欢上这种编程风格。

不好:

```
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

var totalOutput = 0;

for (var i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

好:

```
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

var totalOutput = programmerOutput
  .map((programmer) => programmer.linesOfCode)
  .reduce((acc, linesOfCode) => acc + linesOfCode, 0);
```

### 封装条件

不好:

```
if (fsm.state === 'fetching' && isEmpty(listNode)) {
  /// ...
}
```

好:

```
function shouldShowSpinner(fsm, listNode) {
  return fsm.state === 'fetching' && isEmpty(listNode);
}

if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```

### 避免否定条件

不好:

```
function isDOMNodeNotPresent(node) {
  // ...
}

if (!isDOMNodeNotPresent(node)) {
  // ...
}
```

好:

```
function isDOMNodePresent(node) {
  // ...
}

if (isDOMNodePresent(node)) {
  // ...
}
```

### 避免条件

这似乎是个不可能完成的任务。大多数人第一次听到这个的时候会说，“没有 if 语句我该怎么办？”回答是在多数情况下都可以使用多态来实现相同的任务。第二个问题通常是，“那太好了，不过我为什么要这么做呢？”答案在于我们之前了解过整洁的概念：一个函数应该只做一件事情。如果你的类和函数有 if 语句，就意味着你的函数做了更多的事。记住，只做一件事。

不好:

```
class Airplane {
  //...
  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return getMaxAltitude() - getPassengerCount();
      case 'Air Force One':
        return getMaxAltitude();
      case 'Cessna':
        return getMaxAltitude() - getFuelExpenditure();
    }
  }
}
```

好:

```
class Airplane {
  //...
}

class Boeing777 extends Airplane {
  //...
  getCruisingAltitude() {
    return getMaxAltitude() - getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  //...
  getCruisingAltitude() {
    return getMaxAltitude();
  }
}

class Cessna extends Airplane {
  //...
  getCruisingAltitude() {
    return getMaxAltitude() - getFuelExpenditure();
  }
}
```

### 避免类型检查(第1部分)

JavaScript 是无类型的，也就是说函数可以获取任意类型的参数。有时候你会觉得这种自由是种折磨，因而会不由自主地在函数中使用类型检查。有很多种方法可以避免类型检查。首先要考虑的就是 API 的一致性。

不好:

```
function travelToTexas(vehicle) {
  if (vehicle instanceof Bicycle) {
    vehicle.peddle(this.currentLocation, new Location('texas'));
  } else if (vehicle instanceof Car) {
    vehicle.drive(this.currentLocation, new Location('texas'));
  }
}
```

好:

```
function travelToTexas(vehicle) {
  vehicle.move(this.currentLocation, new Location('texas'));
}
```

### 避免类型检查(第2部分)

如果你在处理基本类型的数据，比如字符串，整数和数组，又不能使用多态，这时你会觉得需要使用类型检查，那么可以考虑 TypeScript。这是普通 JavaScript 的完美替代品，它在标准的 JavaScript 语法之上提供了静态类型。普通 JavaScript 手工检查类型的问题在于这样会写很多废话，而人为的“类型安全”并不能弥补损失的可读性。让你的 JavaScript 保持整洁，写很好的测试，并保持良好的代码审查。否则让 TypeScript (我说过，这是很好的替代品)来做所有事情。

不好:

```
function combine(val1, val2) {
  if (typeof val1 == "number" && typeof val2 == "number" ||
      typeof val1 == "string" && typeof val2 == "string") {
    return val1 + val2;
  } else {
    throw new Error('Must be of type String or Number');
  }
}
```

好:

```
function combine(val1, val2) {
  return val1 + val2;
}
```

### 不要过度优化

现在浏览器在运行时悄悄地做了很多优化工作。很多时候你的优化都是在浪费时间。这里有很好的资源 可以看看哪些优化比较缺乏。把它们作为目标，直到他们能固定下来的时候。

不好:

```
// 在旧浏览器中，每次循环的成本都比较高，因为每次都会重算 `len`。
// 现在浏览器中，这已经被优化了。
for (var i = 0, len = list.length; i < len; i++) {
  // ...
}
```

好:

```
for (var i = 0; i < list.length; i++) {
  // ...
}
```

### 删除不用的代码

不用的代码和重复的代码一样糟糕。在代码库中保留无用的代码是毫无道理的事情。如果某段代码用不到，那就删掉它！如果你以后需要它，仍然可以从代码库的历史版本中找出来。

不好:

```
function oldRequestModule(url) {
  // ...
}

function newRequestModule(url) {
  // ...
}

var req = newRequestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

好:

```
function newRequestModule(url) {
  // ...
}

var req = newRequestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```

## 对象和数据结构

### 使用 getter 和 setter

JavaScript 没有接口或者类型，也没有像 public 和 private 这样的关键字，所以很难应用设计模式。实事上，在对象上使用 getter 和 setter 访问数据远好于直接查找对象属性。“为什么？”你可能会这样问。那好，下面列出了原因：

你想在获取对象属性的时候做更多的事，不必在代码中寻找所有访问的代码来逐个修改。

在进行 set 的时候可以进行额外的数据检验。

封装内部表现。

在获取或设置的时候易于添加日志和错误处理。

继承当前类，可以重写默认功能。

可以对对象属性进行懒加载，比如说从服务器获取属性的数据。

不好:

```
class BankAccount {
  constructor() {
       this.balance = 1000;
  }
}

let bankAccount = new BankAccount();

// 买鞋...
bankAccount.balance = bankAccount.balance - 100;
```

好:

```
class BankAccount {
  constructor() {
       this.balance = 1000;
  }

  // It doesn't have to be prefixed with `get` or `set` to be a getter/setter
  withdraw(amount) {
    if (verifyAmountCanBeDeducted(amount)) {
      this.balance -= amount;
    }
  }
}

let bankAccount = new BankAccount();

// 买鞋...
bankAccount.withdraw(100);
```

### 让对象拥有私有成员

这可以通过闭包实现(ES5以之前的版本)。

不好:

```
var Employee = function(name) {
  this.name = name;
}

Employee.prototype.getName = function() {
  return this.name;
}

var employee = new Employee('John Doe');
console.log('Employee name: ' + employee.getName()); // Employee name: John Doe
delete employee.name;
console.log('Employee name: ' + employee.getName()); // Employee name: undefined
```

好:

```
var Employee = (function() {
  function Employee(name) {
    this.getName = function() {
      return name;
    };
  }

  return Employee;
}());

var employee = new Employee('John Doe');
console.log('Employee name: ' + employee.getName()); // Employee name: John Doe
delete employee.name;
console.log('Employee name: ' + employee.getName()); // Employee name: John Doe
```

## 类

### 单一职责原则 (SRP)

正如《代码整洁之道》所说，“不应该有超过一个原因来改变类”。往一个类里塞进许多功能是件诱人的事情，就像在坐飞机的时候只带一个手提箱一样。这带来的问题是，你的类不会在概念上有凝聚力，会有很多因素造成对它的改变。让你的类需要改变的次数最少是件非常重要的事情。这是因为如果一个类里塞入了太多功能，你只修改它的一部分，可能会让人难以理解它为何会影响代码库中其它相关模块。

不好:

```
class UserSettings {
  constructor(user) {
    this.user = user;
  }

  changeSettings(settings) {
    if (this.verifyCredentials(user)) {
      // ...
    }
  }

  verifyCredentials(user) {
    // ...
  }
}
```

好:

```
class UserAuth {
  constructor(user) {
    this.user = user;
  }

  verifyCredentials() {
    // ...
  }
}

class UserSettings {
  constructor(user) {
    this.user = user;
    this.auth = new UserAuth(user)
  }

  changeSettings(settings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

### 开放封装原则(OCP)

正如 Bertrand Meyer 所说，“软件实体(类、模块、函数等)应该对扩展开放，对修改封闭。”这是什么意思呢？这个原则基本上规定了你应该允许用户扩展你的模块，但不需要打开 .js 源代码文件来进行编辑。

不好:

```
class AjaxRequester {
  constructor() {
    // 如果我们需要另一个 HTTP 方法，比如 DELETE，该怎么办？
    // 我们必须打开这个文件然后手工把它加进去
    this.HTTP_METHODS = ['POST', 'PUT', 'GET'];
  }

  get(url) {
    // ...
  }

}
```

好:

```
class AjaxRequester {
  constructor() {
    this.HTTP_METHODS = ['POST', 'PUT', 'GET'];
  }

  get(url) {
    // ...
  }

  addHTTPMethod(method) {
    this.HTTP_METHODS.push(method);
  }
}
```

### 里氏替换原则(LSP)

这是一个吓人的术语，但描述的却是个简单的概念。它的正式定义为“如果 S 是 T 的子类，那所有 T 类型的对象都可以替换为 S 类型的对象(即 S 类型的对象可以替代 T 类型的对象)，这个替换不会改变程序的任何性质(正确性、任务执行等)。”这确实是个吓人的定义。

对此最好的解释是，如果你有父类和子类，那么父类和子类可以交替使用而不会造成不正确的结果。这可能仍然让人感到疑惑，那么让我们看看经典的正方形和矩形的例子。在数学上，正方形也是矩形，但是如果你在模型中通过继承使用 “is-a” 关系，你很快就会陷入困境。

不好:

```
class Rectangle {
  constructor() {
    this.width = 0;
    this.height = 0;
  }

  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  constructor() {
    super();
  }

  setWidth(width) {
    this.width = width;
    this.height = width;
  }

  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

function renderLargeRectangles(rectangles) {
  rectangles.forEach((rectangle) => {
    rectangle.setWidth(4);
    rectangle.setHeight(5);
    let area = rectangle.getArea(); // 不好：这里对正方形会返回 25，但应该是 20.
    rectangle.render(area);
  })
}

let rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles(rectangles);
```

好:

```
class Shape {
  constructor() {}

  setColor(color) {
    // ...
  }

  render(area) {
    // ...
  }
}

class Rectangle extends Shape {
  constructor() {
    super();
    this.width = 0;
    this.height = 0;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor() {
    super();
    this.length = 0;
  }

  setLength(length) {
    this.length = length;
  }

  getArea() {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes) {
  shapes.forEach((shape) => {
    switch (shape.constructor.name) {
      case 'Square':
        shape.setLength(5);
      case 'Rectangle':
        shape.setWidth(4);
        shape.setHeight(5);
    }

    let area = shape.getArea();
    shape.render(area);
  })
}

let shapes = [new Rectangle(), new Rectangle(), new Square()];
renderLargeShapes(shapes);
```

### 接口隔离原则(ISP)

JavaScript 中没有接口，所以实行这个原则不能像其它语言那样严格。然而即使对 JavaScript 的弱类型系统来说，它仍然是重要的相关。

ISP 指出，“客户不应该依赖于那些他们不使用的接口。” 由于 Duck Typing 理论，接口在 JavaScript 中是个隐性契约。

在 JavaScript 中有一个很好的例子来演示这个原则，即一个拥有巨大设置对象的类。比较好的做法是不要求客户设置大量的选项，因为多数时候他们不需要所有设置。让这些选项成为可选的有助于防止“胖接口”。

不好:

```
class DOMTraverser {
  constructor(settings) {
    this.settings = settings;
    this.setup();
  }

  setup() {
    this.rootNode = this.settings.rootNode;
    this.animationModule.setup();
  }

  traverse() {
    // ...
  }
}

let $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  animationModule: function() {} // 多数时候我们不需要动画
  // ...
});
```

好:

```
class DOMTraverser {
  constructor(settings) {
    this.settings = settings;
    this.options = settings.options;
    this.setup();
  }

  setup() {
    this.rootNode = this.settings.rootNode;
    this.setupOptions();
  }

  setupOptions() {
    if (this.options.animationModule) {
      // ...
    }
  }

  traverse() {
    // ...
  }
}

let $ = new DOMTraverser({
  rootNode: document.getElementsByTagName('body'),
  options: {
    animationModule: function() {}
  }
});
```

### 依赖倒置原则(DIP)

这个原则说明了两个基本问题：

1. 上层模块不应该依赖下层模块，两者都应该依赖抽象。

2. 抽象不应该依赖于具体实现，具体实现应该依赖于抽象。

这一开始可能很难理解，但是如果你使用 Angular.js，你已经看到了对这个原则的一种实现形式：依赖注入(DI)。虽然它们不是完全相同的概念，DIP 阻止上层模块去了解下层模块的细节并设置它们。它可以通过 DI 来实现。这带来的巨大好处降低了模块间的耦合。耦合是种非常不好的开发模式，因为它让代码难以重构。

前提已经提到，JavaScript 没有接口，因此抽象依赖于隐性契约。也就是说，一个对象/类会把方法和属性暴露给另一个对象/类。在下面的例子中，隐性契约是任何用于 InventoryTracker 的 Request 模块都应该拥有 requestItems 方法。

不好:

```
class InventoryTracker {
  constructor(items) {
    this.items = items;

    // 不好：我们创建了一个依赖于特定请求的实现。
    // 我们应该只依赖请求方法：`request` 的 requestItems
    this.requester = new InventoryRequester();
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item);
    });
  }
}

class InventoryRequester {
  constructor() {
    this.REQ_METHODS = ['HTTP'];
  }

  requestItem(item) {
    // ...
  }
}

let inventoryTracker = new InventoryTracker(['apples', 'bananas']);
inventoryTracker.requestItems();
```

好:

```
class InventoryTracker {
  constructor(items, requester) {
    this.items = items;
    this.requester = requester;
  }

  requestItems() {
    this.items.forEach((item) => {
      this.requester.requestItem(item);
    });
  }
}

class InventoryRequesterV1 {
  constructor() {
    this.REQ_METHODS = ['HTTP'];
  }

  requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 {
  constructor() {
    this.REQ_METHODS = ['WS'];
  }

  requestItem(item) {
    // ...
  }
}

// 通过构建外部依赖并注入它们，我们很容易把请求模块替换成
// 一个使用 WebSocket 的新模块。
let inventoryTracker = new InventoryTracker(['apples', 'bananas'], new InventoryRequesterV2());
inventoryTracker.requestItems();
```

### 多用 ES6 类语法，少用 ES5 构造函数语法

在经典的 ES5 的类定义中，很难找到易读的继承、构造、方法定义等。如果你需要继承(你会发现做不到)，那就应该使用类语法。不过，应该尽可能使用小函数而不是类，直到你需要更大更复杂的对象。

不好:

```
var Animal = function(age) {
    if (!(this instanceof Animal)) {
        throw new Error("Instantiate Animal with `new`");
    }

    this.age = age;
};

Animal.prototype.move = function() {};

var Mammal = function(age, furColor) {
    if (!(this instanceof Mammal)) {
        throw new Error("Instantiate Mammal with `new`");
    }

    Animal.call(this, age);
    this.furColor = furColor;
};

Mammal.prototype = Object.create(Animal.prototype);
Mammal.prototype.constructor = Mammal;
Mammal.prototype.liveBirth = function() {};

var Human = function(age, furColor, languageSpoken) {
    if (!(this instanceof Human)) {
        throw new Error("Instantiate Human with `new`");
    }

    Mammal.call(this, age, furColor);
    this.languageSpoken = languageSpoken;
};

Human.prototype = Object.create(Mammal.prototype);
Human.prototype.constructor = Human;
Human.prototype.speak = function() {};
```

好:

```
class Animal {
    constructor(age) {
        this.age = age;
    }

    move() {}
}

class Mammal extends Animal {
    constructor(age, furColor) {
        super(age);
        this.furColor = furColor;
    }

    liveBirth() {}
}

class Human extends Mammal {
    constructor(age, furColor, languageSpoken) {
        super(age, furColor);
        this.languageSpoken = languageSpoken;
    }

    speak() {}
}
```

### 使用方法链

在这里我的意见与《代码整洁之道》的观点不同。有人认为方法链不整洁，而且违反了得墨忒耳定律。也许他们是对的，但这个模式在 JavaScript 中非常有用，你可以很多库中看到，比如 jQuery 和 Lodash。它让代码变得既简洁又有表现力。在类中，只需要在每个函数结束前返回 this，就实现了链式调用的类方法。

不好:

```
class Car {
  constructor() {
    this.make = 'Honda';
    this.model = 'Accord';
    this.color = 'white';
  }

  setMake(make) {
    this.name = name;
  }

  setModel(model) {
    this.model = model;
  }

  setColor(color) {
    this.color = color;
  }

  save() {
    console.log(this.make, this.model, this.color);
  }
}

let car = new Car();
car.setColor('pink');
car.setMake('Ford');
car.setModel('F-150')
car.save();
```

好:

```
class Car {
  constructor() {
    this.make = 'Honda';
    this.model = 'Accord';
    this.color = 'white';
  }

  setMake(make) {
    this.name = name;
    // NOTE: 返回 this 以实现链式调用
    return this;
  }

  setModel(model) {
    this.model = model;
    // NOTE: 返回 this 以实现链式调用
    return this;
  }

  setColor(color) {
    this.color = color;
    // NOTE: 返回 this 以实现链式调用
    return this;
  }

  save() {
    console.log(this.make, this.model, this.color);
  }
}

let car = new Car()
  .setColor('pink')
  .setMake('Ford')
  .setModel('F-150')
  .save();
```

### 多用组合，少用继承

大家都知道 GoF 的设计模式，其中提到应该多用组合而不是继承。对于继承和组合，都有大量的理由在支撑，但这个准则的要点在于，你的想法本能地会想到继承，但这时候不防多思考一下用组合是否能更好的处理问题——某些时候，的确能。

你可能会考虑：“我什么时候该用继承？”这取决于你遇到的问题。这里有一个不错的清单说明了什么时候用继承比用组合更合适：

你的继承是一个“is-a”关系，而不是“has-a”关系(Animal->Human 对比 User->UserDetails)。

可以从基础复用代码 (人可以像所有动物一样移动)。

你想通过修改基础来实现对所有子类的全局性更改。(改变动物移动时的热量消耗)。

不好:

```
class Employee {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  // ...
}

// 这样不好，因为 Employees "拥有" 税务数据。EmployeeTaxData 不是属于 Employee 的一个类型
class EmployeeTaxData extends Employee {
  constructor(ssn, salary) {
    super();
    this.ssn = ssn;
    this.salary = salary;
  }

  // ...
}
```

好:

```
class Employee {
  constructor(name, email) {
    this.name = name;
    this.email = email;

  }

  setTaxData(ssn, salary) {
    this.taxData = new EmployeeTaxData(ssn, salary);
  }
  // ...
}

class EmployeeTaxData {
  constructor(ssn, salary) {
    this.ssn = ssn;
    this.salary = salary;
  }

  // ...
}
```

## 测试

测试比生产更重要。如果你不进行测试，或者测试的量不够，那你就不能肯定你写的代码不会造成破坏。测试数量依靠你的开发团队来决定，但 100% 覆盖率(所有语句和分支)能让你拥有巨大的信心，也能使程序员们安心。也就是说，你需要一个不错的测试框架，还需要一个好的覆盖检查工具.

没有什么理由可以让你不写测试。这里有 大量不错的 JS 测试框架，可以去找个你们团队喜欢的来用。如果你找一个适合在你的团队中使用的工作，就把为每个新产生的特性/方法添加测试作为目标。如果你喜欢测试驱动开发(TDD)的方法，非常好，但要注意在让你的测试覆盖所有特性，或者重构过的代码。

### 每次测试一个概念

不好:

```
const assert = require('assert');

describe('MakeMomentJSGreatAgain', function() {
  it('handles date boundaries', function() {
    let date;

    date = new MakeMomentJSGreatAgain('1/1/2015');
    date.addDays(30);
    date.shouldEqual('1/31/2015');

    date = new MakeMomentJSGreatAgain('2/1/2016');
    date.addDays(28);
    assert.equal('02/29/2016', date);

    date = new MakeMomentJSGreatAgain('2/1/2015');
    date.addDays(28);
    assert.equal('03/01/2015', date);
  });
});
```

好:

```
const assert = require('assert');

describe('MakeMomentJSGreatAgain', function() {
  it('handles 30-day months', function() {
    let date = new MakeMomentJSGreatAgain('1/1/2015');
    date.addDays(30);
    date.shouldEqual('1/31/2015');
  });

  it('handles leap year', function() {
    let date = new MakeMomentJSGreatAgain('2/1/2016');
    date.addDays(28);
    assert.equal('02/29/2016', date);
  });

  it('handles non-leap year', function() {
    let date = new MakeMomentJSGreatAgain('2/1/2015');
    date.addDays(28);
    assert.equal('03/01/2015', date);
  });
});
```

## 并发

### 使用 Promise 而不是回调

回调并不整洁，它会导致过多的嵌套。ES6 的 Promise 是个内置的全局类型。使用它！

不好:

```
require('request').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', function(err, response) {
  if (err) {
    console.error(err);
  }
  else {
    require('fs').writeFile('article.html', response.body, function(err) {
      if (err) {
        console.error(err);
      } else {
        console.log('File written');
      }
    })
  }
})
```

好:

```
require('request-promise').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then(function(response) {
    return require('fs-promise').writeFile('article.html', response);
  })
  .then(function() {
    console.log('File written');
  })
  .catch(function(err) {
    console.error(err);
  })
```

### async/await 比 Promise 还整洁

与回调相当，Promise 已经相当整洁了，但 ES7 带来了更整洁的解决方案 —— async 和 await。你要做的事情就是在一个函数前加上 async 关键字，然后写下命令形式的逻辑，而不再需要 then 链。现在可以使用这个 ES7 特性带来的便利！

不好:

```
require('request-promise').get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin')
  .then(function(response) {
    return require('fs-promise').writeFile('article.html', response);
  })
  .then(function() {
    console.log('File written');
  })
  .catch(function(err) {
    console.error(err);
  })
```

好:

```
async function getCleanCodeArticle() {
  try {
    var request = await require('request-promise')
    var response = await request.get('https://en.wikipedia.org/wiki/Robert_Cecil_Martin');
    var fileHandle = await require('fs-promise');

    await fileHandle.writeFile('article.html', response);
    console.log('File written');
  } catch(err) {
      console.log(err);
    }
  }
```

## 错误处理

抛出错误是件好事！这表示运行时已经成功检测到程序出错了，它停止当前调用框上的函数执行，并中止进程(在 Node 中)，最后在控制台通知你，并输出栈跟踪信息。

### 不要忽略捕捉到的错误

捕捉到错误却什么也不错，你就失去了纠正错误的机会。多数情况下把错误记录到控制台(console.log)也不比忽略它好多少，因为在少量的控制台信息中很难发现这一条。如果尝试在 try/catch 中封装代码，就意味着你知道这里可能发生错，你应该在错误发生的时候有应对的计划、或者处理办法。

不好:

```
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}
```

好:

```
try {
  functionThatMightThrow();
} catch (error) {
  // 选择之一(比 console.log 更闹心)：
  console.error(error);
  // 另一个选择：
  notifyUserOfError(error);
  // 另一个选择：
  reportErrorToService(error);
  // 或者所有上述三种选择！
}
```

### 不要忽视被拒绝的Promise

这一条与不要忽略从 try/catch 捕捉到的错误有相同的原因。

不好:

```
getdata()
.then(data => {
  functionThatMightThrow(data);
})
.catch(error => {
  console.log(error);
});
```

好:

```
getdata()
.then(data => {
  functionThatMightThrow(data);
})
.catch(error => {
  // 选择之一(比 console.log 更闹心)：
  console.error(error);
  // 另一个选择：
  notifyUserOfError(error);
  // 另一个选择：
  reportErrorToService(error);
  // 或者所有上述三种选择！
});
```

## 格式

格式是个很主观的东西，像这里提到的许多规则一，你不必完全遵循。要点不在于争论格式。大量工具 可以自动处理优化格式。用一个！让工程师争论格式问题简直就是在浪费时间和金钱。

对于那些不能自动处理的格式(可以自动处理的包括缩进、Tab或空格、双引号或单引用等)，就看看这里的指导。

### 使用一致的大小写

JavaScript 是无类型的，所以大小写可以帮助你了解变量、函数等。这些规则具有较强的主观性，所以你的团队应该选择需要的。重点不在于你选择了什么，而在于要始终保持一致。

不好:

```
var DAYS_IN_WEEK = 7;
var daysInMonth = 30;

var songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
var Artists = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restore_database() {}

class animal {}
class Alpaca {}
```

好:

```
var DAYS_IN_WEEK = 7;
var DAYS_IN_MONTH = 30;

var songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];
var artists = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}
function restoreDatabase() {}

class Animal {}
class Alpaca {}
```

### 函数调用者和被调用者应该尽可能放在一起

如果一个函数调用另一个函数，那应该让他们在源文件中的位置非常接近。理想情况下应该把调用者放在被调用者的正上方，这会让你的代码更易读，因为我们都习惯从上往下读代码，就像读报纸那样。

不好:

```
class PerformanceReview {
  constructor(employee) {
    this.employee = employee;
  }

  lookupPeers() {
    return db.lookup(this.employee, 'peers');
  }

  lookupMananger() {
    return db.lookup(this.employee, 'manager');
  }

  getPeerReviews() {
    let peers = this.lookupPeers();
    // ...
  }

  perfReview() {
      getPeerReviews();
      getManagerReview();
      getSelfReview();
  }

  getManagerReview() {
    let manager = this.lookupManager();
  }

  getSelfReview() {
    // ...
  }
}

let review = new PerformanceReview(user);
review.perfReview();
```

好:

```
class PerformanceReview {
  constructor(employee) {
    this.employee = employee;
  }

  perfReview() {
      getPeerReviews();
      getManagerReview();
      getSelfReview();
  }

  getPeerReviews() {
    let peers = this.lookupPeers();
    // ...
  }

  lookupPeers() {
    return db.lookup(this.employee, 'peers');
  }

  getManagerReview() {
    let manager = this.lookupManager();
  }

  lookupMananger() {
    return db.lookup(this.employee, 'manager');
  }

  getSelfReview() {
    // ...
  }
}

let review = new PerformanceReview(employee);
review.perfReview();
```

## 注释

### 只注释业务逻辑复杂的内容

注释是用来解释代码的，而不是必须的。好的代码应该 自注释。

不好:

```
function hashIt(data) {
  // Hash 码
  var hash = 0;

  // 字符串长度
  var length = data.length;

  // 遍历数据中所有字符
  for (var i = 0; i < length; i++) {
    // 获取字符编码
    var char = data.charCodeAt(i);
    // 生成 Hash
    hash = ((hash << 5) - hash) + char;
    // 转换为32位整数
    hash = hash & hash;
  }
}
```

好:

```
function hashIt(data) {
  var hash = 0;
  var length = data.length;

  for (var i = 0; i < length; i++) {
    var char = data.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;

    // 转换为32位整数
    hash = hash & hash;
  }
}
```

### 不要把注释掉的代码留在代码库中

版本控制存在的原因就是保存你的历史代码。

不好:

```
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

好:

```
doStuff();
```

### 不需要日志式的注释

记住，使用版本控制！没用的代码、注释掉的代码，尤其是日志式的注释。用 git log 来获取历史信息！

不好:

```
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
function combine(a, b) {
  return a + b;
}
```

好:

```
function combine(a, b) {
  return a + b;
}
```

### 避免位置标记

位置标记通常只会添加垃圾信息。通过对函数或变量名以及适当的缩进就能为代码带来良好的可视化结构。

不好:

```
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
let $scope.model = {
  menu: 'foo',
  nav: 'bar'
};

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
let actions = function() {
  // ...
}
```

好:

```
let $scope.model = {
  menu: 'foo',
  nav: 'bar'
};

let actions = function() {
  // ...
}
```

### 避免在源文件中添加版权注释

这是代码文件树顶层的 LICENSE 文件应该干的事情。

不好:

```
/*
The MIT License (MIT)

Copyright (c) 2016 Ryan McDermott

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE
*/

function calculateBill() {
  // ...
}
```

好:

```
function calculateBill() {
  // ...
}
```