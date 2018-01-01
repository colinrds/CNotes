[toc]

## 总体概要

OO(面向对象)概念的提出是软件开发工程发展的一次革命，多年来我们借助它使得很多大型应用程序得以顺利实现。如果您还没有掌握并使用OO进行程序设计和开发，那么您无疑还停留在软件开发的石器时代。大多数编程语言，尤其是近年问世的一些语言，都很好的支持了面向对象，您可能对此了如执掌，但是一些语言在OO方面却无法与其它高级语言相比，在这些语言上进行面向对象程序设计和开发会有些困难，例如本文要讨论的JavaScript。JavaScript是一门古老的语言，但是随着近期Web2.0 技术的热捧，这门语言又重新焕发出青春的光辉，借助于JavaScript客户端技术，我们的Web体验变得丰富而又精彩，为了设计和开发更加完善、复杂的客户端应用，我们必须掌握JavaScript上的OO方法，这正是本文要讨论的。

当今 JavaScript 大行其道，各种应用对其依赖日深。web 程序员已逐渐习惯使用各种优秀的 JavaScript 框架快速开发 Web 应用，从而忽略了对原生 JavaScript 的学习和深入理解。所以，经常出现的情况是，很多做了多年 JS 开发的程序员对闭包、函数式编程、原型总是说不清道不明，即使使用了框架，其代码组织也非常糟糕。这都是对原生 JavaScript 语言特性理解不够的表现。要掌握好 JavaScript，首先一点是必须摒弃一些其他高级语言如 Java、C# 等类式面向对象思维的干扰，全面地从函数式语言的角度理解 JavaScript 原型式面向对象的特点。把握好这一点之后，才有可能进一步使用好这门语言。本文适合群体：使用过 JS 框架但对 JS 语言本质缺乏理解的程序员，具有 Java、C++ 等语言开发经验，准备学习并使用 JavaScript 的程序员，以及一直对 JavaScript 是否面向对象模棱两可，但希望知道真相的 JS 爱好者。

言归正传，我们切入主题------Javascript的面向对象编程。要谈Javascript的面向对象编程，我们第一步要做的事情就是忘记我们所学的面向对象编程。传统C++或Java的面向对象思维来学习Javascript的面向对象会给你带来不少困惑，让我们先忘记我们所学的，从新开始学习这门特殊的面向对象编程。既然是OO编程，要如何来理解OO编程呢，记得以前学Java，学了很久都不入门，后来有幸读了《Java编程思想》这本大作，顿时豁然开朗，因此本文也将以对象模型的方式来探讨的Javascript的OO编程。因为Javascript 对象模型的特殊性，所以使得Javascript的继承和传统的继承非常不一样，同时也因为Javascript里面没有类，这意味着Javascript里面没有extends,implements。那么Javascript到底是如何来实现OO编程的呢?好吧，让我们开始吧，一起在Javascript的OO世界里来一次漫游。

## 案例引入

1. 场景人物简介

- 主人公甲------大耳文老师 （软件行业领域的骨灰级程序猿）
- 主人公乙------大熊君 （初出茅庐的菜鸟程序员）
- 关系------ 工作上的上下级，私底下的师徒

2. 话题开展

有一天这师徒二人午饭后闲聊起来，突然聊起ATM取款机，大耳文老师说取款机对人们的日常生活帮助真是很大，同时也提高了银行的生产力，这项发明和我们的软件行业也是分不开的，大熊君说是啊老师，一些实用的操作提供了便捷的功能，这时大耳文老师微微一笑说道，那么你知道这些功能是如何实现的吗，满满自信的大熊君回答说：简单啊就是一些像存钱，取钱，转账的简单功能，老师我现在就写给你看。。。。未完待续

3. 实例讲解，循序渐进

十分钟过后大熊君把一份转账操作部分的代码给老师看，老师笑了（呵呵）

```
function TransferTransaction(fromAccount, toAccount, balance) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
    this.balance = balance;
};
TransferTransaction.prototype = {
    transfer: function () {
        this.toAccount = this.fromAccount - balance;
    },
    getFromAccount: function () {
        return this.fromAccount;
    },
    getToAccount: function () {
        return this.toAccount;
    },
    getBalance: function () {
        return this.balance;
    }
};

var tt = new TransferTransaction(1000, 3000, 100);
tt.transfer();
tt.getToAccount();
```

老师说道大熊你这代码是有问题的，大熊没有想太多直接回答说，没有啊老师我这是面向对象写的啊，老师很有耐心的说道：先别着急听我给你讲。

首先先不说面向对象就功能而言也是有问题的，假设我的转出账户就是fromAccount要是余额为0那，不就会出现问题了吗，大熊，大熊看了一下很难为情地说奥 是啊  您稍等一下 大熊回去又作修改。。。。未完待续

改完了这下没问题了吧

```
function TransferTransaction(fromAccount, toAccount, balance) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
    this.balance = balance;
};
TransferTransaction.prototype = {
    transfer: function () {
        if (this.fromAccount < balance) {
            throw new Error("余额不足!");
        }
        this.toAccount = this.fromAccount - balance;
    },
    getFromAccount: function () {
        return this.fromAccount;
    },
    getToAccount: function () {
        return this.toAccount;
    },
    getBalance: function () {
        return this.balance;
    }
};

var tt = new TransferTransaction(1000, 3000, 100);
tt.transfer();
tt.getToAccount();
```

老师看了看说道基本功能是做到，但这样设计是很低效的，并且很多设计原则 也违反了。

大熊疑惑的问到为什么那，老师说如果现在新的需求来了比如根据账户的等级划分会有不同级别的转账金额你如何处理那？比如vip级别最多可以转账5000，Normal级别至多为1000你试试看，好的老师，您稍等。。。。未完待续

```
function TransferTransaction(fromAccount, toAccount, balance, rank) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
    this.balance = balance;
    this.rank = rank;
};
TransferTransaction.prototype = {
    transfer: function () {
        var transBalance = 1000;
        if ("vpi" == this.rank) {
            transBalance = 5000;
        }
        if (this.balance > transBalance) {
            throw new Error("您的转账金额超出了规定范围!");
        }
        if (this.fromAccount < this.balance) {
            throw new Error("余额不足!");
        }
        this.fromAccount = this.fromAccount - this.balance;
        this.toAccount = this.toAccount + this.balance;
    },
    getFromAccount: function () {
        return this.fromAccount;
    },
    getToAccount: function () {
        return this.toAccount;
    },
    getBalance: function () {
        return this.balance;
    }
};

var tt = new TransferTransaction(1000, 3000, 20000, "vip");
tt.transfer();
tt.getToAccount();
```
哈哈哈老师我完成了，很快就写出来了 嘻嘻……

大耳文老师看了下代码说道，恩改的不错，但依然还是有问题存在的，咱们回头说面向对象，你的代码书写方式确实符合了面向对象的特点，但是思想还停留在过程化的设计思想上。

这时大熊君过来很谦虚的说道请老师指点 。。。 未完待续

几分钟过后老师总结出一些例子中的问题

1. 职责多需要分解
2. 抽象实体模型
3. 依赖性强可扩展低

咱们来一步步分析和重构

首先你的对象完成了很多职责比如 级别过滤 金额比对 异常处理  这些都不属于转账的核心功能  应该划分出来

还有就是 在转账操作时 Account应该是一个实体数据模型  应该独立出来 不应该在具体业务方法中出现 这也违背了（迪米特法则）一种软件设计法则，稍后会提供扩展阅读链接(*^__^*)

重构分为几个步骤去设计：

1. 建立Account类

```
function Account(balance, rank) {
    this.balance = balance;
    this.rank = rank;
};
Account.prototype = {
    getBalance: function () {
        return this.balance;
    },
    getRank: function () {
        return this.balance;
    }
};
```

2. 重新设计TransferTransaction类

```
function TransferTransaction(fromAccount, toAccoun) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
};
TransferTransaction.prototype = {
    transfer: function (balance) {
        var transBalance = 1000;
        if ("vpi" == this.this.fromAccount.getRank()) {
            transBalance = 5000;
        }
        if (this.fromAccount.getBalance() > transBalance) {
            throw new Error("您的转账金额超出了规定范围!");
        }
        if (this.fromAccount.getBalance() < balance) {
            throw new Error("余额不足!");
        }
        this.fromAccount.setBalance(this.fromAccount.getBalance() - this.balance);
        this.toAccount.setBalance(this.toAccount.getBalance() + this.balance);
    },
    getFromAccount: function () {
        return this.fromAccount;
    },
    getToAccount: function () {
        return this.toAccount;
    }
};
```

3. 增加新的转账管理类TransTManager类(主要负责周边功能任务)

```
function TransTManager(fromAccount, toAccoun) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
    this.transferTransaction = null;
};
TransTManager.prototype = {
    transfer: function (balance) {
        var transBalance = 1000;
        if ("vpi" == this.this.fromAccount.getRank()) {
            transBalance = 5000;
        }
        if (this.fromAccount.getBalance() > transBalance) {
            throw new Error("您的转账金额超出了规定范围!");
        }
        if (this.fromAccount.getBalance() < balance) {
            throw new Error("余额不足!");
        }
        this.transferTransaction = new TransferTransaction(this.fromAccount, this.toAccount);
        this.transferTransaction.transfer(balance);
    }
};
```

4. 接下来设计会员的级别类每个等级作为一个类出现（引用策略模式）

```
function NormalDiscount() {

};
NormalDiscount.prototype = {
    getDiscount: function () {
        return 3000;
    }
};

function VIPDiscount() {

};
VIPDiscount.prototype = {
    getDiscount: function () {
        return 5000;
    }
};
```

5. 经过重构后思维会清晰很多而且更加面向对象了，其实思维往往比语言本身重要得多

以下是完整的案例代码

```
function Account(balance, rank) {
    this.balance = balance;
    this.rank = rank;
};
Account.prototype = {
    getBalance: function () {
        return this.balance;
    }
    getRank: function () {
        return this.balance;
    }
};

function TransferTransaction(fromAccount, toAccoun) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
};
TransferTransaction.prototype = {
    transfer: function (balance) {
        this.fromAccount.setBalance(this.fromAccount.getBalance() - balance);
        this.toAccount.setBalance(this.toAccount.getBalance() + balance);
    },
    getFromAccount: function () {
        return this.fromAccount;
    },
    getToAccount: function () {
        return this.toAccount;
    }
};

function TransTManager(fromAccount, toAccoun) {
    this.fromAccount = fromAccount;
    this.toAccount = toAccount;
    this.transferTransaction = null;
    this.rankVendor = {
        "normal": {
            get: function () {
                return 1000;
            }
        },
        "vip": {
            get: function () {
                return 5000;
            }
        }
    };
};
TransTManager.prototype = {
    transfer: function (balance) {
        var transBalance = this.rankVendor[this.fromAccount.getRank()]["get"]();
        if (this.fromAccount.getBalance() > transBalance) {
            throw new Error("您的转账金额超出了规定范围!");
        }
        if (this.fromAccount.getBalance() < balance) {
            throw new Error("余额不足!");
        }
        this.transferTransaction = new TransferTransaction(this.fromAccount, this.toAccount);
        this.transferTransaction.transfer(balance);
    }
};

var tt = new TransTManager(new Account(5000, "vip"), new Account(3000, "Normal"));
tt.transfer(3000);
```

## 名词解释

1. 迪米特法则：迪米特法则（Law of Demeter）又叫作最少知识原则（Least Knowledge Principle 简写LKP），就是说一个对象应当对其他对象有尽可能少的了解,不和陌生人说话。英文简写为: LoD.
2. 实体模型：实体类一般对应物理意义上的实体，业务类则是对一些业务规则(算法)的封装，比如User，是一个实体类，它有属性，Login这里就算一个业务

## 总结一下

1. 类的职责不能多，职责多需要分解
2. 抽象实体模型
3. 理解oo的特质封装，继承，多态

