我们通过表单验证的功能，来逐步演进面向对象的方式.   对于刚刚接触javascript的朋友来说，如果要写一个验证用户名，密码，邮箱的功能， 一般可能会这么写：

```
//表单验证
var checkUserName = function () {
    console.log('全局checkUserName');
};
var checkUserEmail = function () {
    console.log('全局checkUserEmail');
};
var checkUserPwd = function () {
    console.log('全局checkUserPwd');
};
```

这种写法，从功能上来说 没有什么问题， 但是在团队协作的时候， 会造成覆盖全局变量的问题， 那要大大降低覆盖的可能性， 一般会在外面套一个对象

```
var Utils = {
    checkUserName: function () {
        console.log('Utils->checkUserName');
    },
    checkUserEmail: function () {
        console.log('Utils->checkUserEmail');
    },
    checkUserPwd: function () {
        console.log('Utils->checkUserPwd');
    }
}

checkUserEmail();
Utils.checkUserEmail();
```
上面这种方式，是字面量方式添加，在设计模式里面，也称为单例（单体）模式， 与之类似的可以通过在函数本身添加属性和方法，变成静态属性和方法，达到类似的效果:

```
var Utils = function () {

}
Utils.checkUserName = function () {
    console.log('Utils.checkUserName');
}
Utils.checkUserPwd = function () {
    console.log('Utils.checkUserPwd');
}
Utils.checkUserEmail = function () {
    console.log('Utils.checkUserEmail');
}

Utils.checkUserEmail();

for (var key in Utils) {
    (Utils.hasOwnProperty(key)) ? console.log(key): '';
}

//加在函数上面的属性和方法，无法通过对象使用
var oUtil = new Utils();
oUtil.checkUserEmail(); //错误
```

还可以通过函数调用方式，返回一个对象，把方法和属性写在对象中, 这种方式 跟面向对象没有什么关系，只是从函数的返回值角度来改造

```
//使用函数的方式 返回一个对象
var Util = function () {
    return {
        checkUserName: function () {
            console.log('userName...');
        },
        checkUserPwd: function () {
            console.log('userPwd...');
        },
        checkUserEmail: function () {
            console.log('userEmail...');
        }
    }
}
Util().checkUserEmail();
```

还可以通过类似传统面向对象语言，使用构造函数方式 为每个实例添加方法和属性, 这种方式，存在一个问题， 不能达到函数共用，每个实例都会复制到方法.

```
var Util = function () {
    this.checkUserName = function () {
        console.log('userName');
    };
    this.checkUserEmail = function () {
        console.log('userEmail');
    };
    this.checkUserPwd = function () {
        console.log('userPwd');
    };
}

var oUtil1 = new Util();
var oUtil2 = new Util();
console.log(oUtil1.checkUserEmail === oUtil2.checkUserEmail); //false
```

一般，我们可以通过原型属性（prototype）改造这种方式，达到不同实例共用同一个方法

```
var Util = function () {

};
Util.prototype.checkUserName = function () {
    console.log('userName');
};
Util.prototype.checkUserPwd = function () {
    console.log('userPwd');
};
Util.prototype.checkUserEmail = function () {
    console.log('userEmail');
};
var oUtil1 = new Util();
var oUtil2 = new Util();
console.log(oUtil1.checkUserEmail === oUtil2.checkUserEmail); //true
```

也可以把原型对象上的所有方法，使用字面量方式简写

```
var Util = function () {

};
Util.prototype = {
    checkUserEmail: function () {
        console.log('userEmail');
    },
    checkUserName: function () {
        console.log('userName');
    },
    checkUserPwd: function () {
        console.log('userPwd');
    }
};
var oUtil1 = new Util();
var oUtil2 = new Util();
console.log(oUtil1.checkUserEmail === oUtil2.checkUserEmail); //true
```

注意：  字面量方式和原型对象一个个添加   这两种不要混用， 否则会产生覆盖

如果我们想把面向对象的使用方式更加的优雅，比如链式调用， 我们应该在每个方法中返回对象本身，才能继续调用方法， 即返回this

```
var Util = function () {
    return {
        checkUserName: function () {
            console.log('userName...');
            return this;
        },
        checkUserPwd: function () {
            console.log('userPwd...');
            return this;
        },
        checkUserEmail: function () {
            console.log('userEmail...');
            return this;
        }
    }
}
// 方法中如果没有返回this,下面这种调用方式是错误的
Util().checkUserEmail().checkUserName();

// 方法中返回对象本身,可以链式调用
Util().checkUserEmail().checkUserName().checkUserPwd();
```

```
var Util = function () {
    this.checkUserName = function () {
        console.log('userName');
        return this;
    };
    this.checkUserEmail = function () {
        console.log('userEmail');
        return this;
    };
    this.checkUserPwd = function () {
        console.log('userPwd');
        return this;
    };
}

new Util().checkUserEmail().checkUserName().checkUserPwd();
```

```
var Util = function () {

};
Util.prototype = {
    checkUserEmail: function () {
        console.log('userEmail');
        return this;
    },
    checkUserName: function () {
        console.log('userName');
        return this;
    },
    checkUserPwd: function () {
        console.log('userPwd');
        return this;
    }
};

new Util().checkUserEmail().checkUserName().checkUserPwd();
```

```
var Util = function () {

};
Util.prototype.checkUserName = function () {
    console.log('userName');
    return this;
};
Util.prototype.checkUserPwd = function () {
    console.log('userPwd');
    return this;
};
Util.prototype.checkUserEmail = function () {
    console.log('userEmail');
    return this;
};

new Util().checkUserEmail().checkUserName().checkUserPwd();
```

在实际开发中，我们经常需要扩展一些功能和模块。扩展可以在本对象或者父类对象或者原型上

```
Function.prototype.checkUserName = function () {
    console.log('ghostwu');
};

var fn1 = function () {};
var fn2 = function () {};

console.log('checkUserName' in fn1); //true
console.log('checkUserName' in fn2); //true

fn1.checkUserName(); //ghostwu
fn2.checkUserName(); //ghostwu
```

如果我们使用上面这种方式扩展，从功能上来说，是没有问题的，但是确造成了全局污染：通俗点说，并不是说有的函数都需要checkUserName这个方法，而我们这样写，所有的函数在创建过程中都会从父类的原型链上继承checkUserName, 但是这个方法，我们根本不用， 所以浪费性能， 为了解决这个问题，我们应该要在需要使用这个方法的函数上添加，不是所有的都添加，另外关于in的用法，如果不熟悉,可以看下我的这篇文章:立即表达式的多种写法与注意点以及in操作符的作用

```
Function.prototype.addMethod = function (name, fn) {
    this[name] = fn;
}

var fn1 = function () {};
var fn2 = function () {};

fn1.addMethod('checkUserName', function () {
    console.log('ghostwu');
});

fn1.checkUserName(); //ghostwu
fn2.checkUserName(); //报错
```

通过上述的改造，成功解决了全局污染， fn2这个函数上面没有添加checkUserName这个方法，只在fn1上面添加

我们继续把上面的方式，改成链式调用：  这里需要改两个地方， 一种是添加方法addMethod可以链式添加， 一种是添加完了之后，可以链式调用

```
Function.prototype.addMethod = function (name, fn) {
    this[name] = fn;
    return this;
};

var fn1 = function () {};

fn1.addMethod('checkUserName', function () {
    console.log('userName:ghostwu');
    return this;
}).addMethod('checkUserEmail', function () {
    console.log('userEmail');
    return this;
}).addMethod('checkUserPwd', function () {
    console.log('userUserPwd');
    return this;
});
fn1.checkUserName().checkUserEmail().checkUserPwd();
```

上面是属于函数式 链式 调用，  我们可以改造addMethod方法， 在原型上添加函数，而不是实例上， 这样我们就可以达到类式的链式调用

```
Function.prototype.addMethod = function (name, fn) {
    this.prototype[name] = fn;
    return this;
};

var fn1 = function () {};

fn1.addMethod('checkUserName', function () {
    console.log('userName:ghostwu');
    return this;
}).addMethod('checkUserEmail', function () {
    console.log('userEmail');
    return this;
}).addMethod('checkUserPwd', function () {
    console.log('userUserPwd');
    return this;
});
new fn1().checkUserName().checkUserEmail().checkUserPwd();
```