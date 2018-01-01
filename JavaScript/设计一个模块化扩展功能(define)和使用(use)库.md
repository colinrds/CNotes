模块化的诞生标志着javascript开发进入工业时代，近几年随着es6, require js( sea js ), node js崛起，特别是es6和node js自带模块加载功能，给大型程序开发带来了极大的便利。这几个东西没有出来之前，最原始的开发全部是利用全局函数进行封装，如:

```
function checkEmail(){}
function checkName(){}
function checkPwd(){}
```

这种开发方式，非常容易造成一个问题，全局变量污染（覆盖）

如：有个页面需要引入3个js文件( a.js, b.js, c.js )， A程序员在a.js文件中定义了一个函数叫show, B程序员在b.js中也定义了一个函数叫show, 那么引入同一个页面之后，后面的函数会把前面的覆盖，那C程序员引入之后，本来想调用A程序员的show方法，结果被B程序员写的文件给覆盖了，这就很恐怖了，如果这是一个公共方法（核心方法），可能造成系统完全挂掉，那怎么解决呢？早期程序员一般用命名空间，如：

```
var ghostwu = ghostwu || {};
ghostwu.tools = {
    checkEmail: function () {},
    checkName: function () {},
    checkPwd: function () {}
}
var zhangsan = zhangsan || {};
zhangsan.tools = {
    checkEmail: function () {},
    checkName: function () {},
    checkPwd: function () {}
}
ghostwu.tools.checkPwd();
zhangsan.tools.checkPwd();
```

这样就有可能大大降低重名的可能性，但是还是会有小部分覆盖的可能性，当然可以在程序中判断，如果存在这个命名空间，就不让定义，但是这样子写，就没有封装性可言，只要引入了js文件，谁都能够操作这些命名空间，也很危险，对于公共接口（ 比如获取数据，可以暴露给外部使用），对于一些核心，比较重要的，就要封装起来，不要暴露给外部使用，所以可以通过闭包（ 立即表达式）把命名空间中的方法选择性的暴露给外部

```
(function ( window, undefined ) {
    var ghostwu = ghostwu || {};
    ghostwu.tools = {
        checkEmail: function () {
            console.log('ghostwu给你暴露一个公共的checkEmail方法');
        },
        checkName: function () {
            console.log('checkName');
        },
        checkPwd: function () {
            console.log('checkPwd');
        }
    },
    public = {
        checkEmail : ghostwu.tools.checkEmail,
    }
    window.G = public;
})( window );
G.checkEmail();
G.checkName(); //报错, checkName没有暴露出来
```

改进之后，封装性和团队协作开发又得到了进一步的提高，但是依然存在另一个问题，后来的开发者如果要为这个框架新增一个功能模块，而且也能选择性的暴露一些接口怎么办呢？

我们可以封装一个模块扩展功能和模块使用功能，以后要做扩展就非常方便了.

```
(function (window, undefined) {
    var G = G || {};
    G.version = 'v1.0.0';
    G.define = function (path, fn) {
        var args = path.split('.'),
            parent = old = this;
        if (args[0] === 'G') {
            args = args.slice(1);
        }
        if (args[0] === 'define' || args[0] === 'module') {
            return;
        }
        for (var i = 0, len = args.length; i < len; i++) {
            if (typeof parent[args[i]] === 'undefined') {
                parent[args[i]] = {};
            }
            old = parent;
            parent = parent[args[i]];
        }
        if (fn) old[args[--i]] = fn();
        return this;
    };
    window.G = G;
})(window);

G.define("ghostwu.string", function () {
    return {
        trimLeft: function (str) {
            return str.replace(/^\s+/, '');
        },
        trimRight: function () {
            return str.replace(/\s+$/, '');
        },
        trim: function () {
            return str.replace(/^\s+|\s+$/g, '');
        }
    }
});

var str = '  跟着ghostwu学习设计模式 ';
alert('(' + str + ')');
alert('(' + G.ghostwu.string.trimLeft(str) + ')');
alert('(' + G.ghostwu.string.trimRight(str) + ')');
alert('(' + G.ghostwu.string.trim(str) + ')');
```

我们封装了一个define函数，这个函数用来给G模块扩展功能，这里我扩展了一个字符串处理对象，包含了3个方法: trim, trimLeft, trimRight

![image](http://ow2n75eab.bkt.clouddn.com/253192-2011170905213333647-647826522.jpg)

使用的时候，按照命名空间的方式使用即可，我们也可以加一个使用模块的方法use

```
(function (window, undefined) {
    var G = G || {};
    G.version = 'v1.0.0';
    G.define = function (path, fn) {
        var args = path.split('.'),
            parent = old = this;
        if (args[0] === 'G') {
            args = args.slice(1);
        }
        if (args[0] === 'define' || args[0] === 'module') {
            return;
        }
        for (var i = 0, len = args.length; i < len; i++) {
            if (typeof parent[args[i]] === 'undefined') {
                parent[args[i]] = {};
            }
            old = parent;
            parent = parent[args[i]];
        }
        if (fn) old[args[--i]] = fn();
        return this;
    };
    G.use = function () {
        var args = Array.prototype.slice.call(arguments),
            fn = args.pop(),
            path = args[0] && args[0] instanceof Array ? args[0] : args,
            relys = [],
            mId = '',
            i = 0,
            len = path.length,
            parent, j, jLen;
        while (i < len) {
            if (typeof path[i] === 'string') {
                parent = this;
                mId = path[i].replace(/^G\./, '').split('.');
                for (var j = 0, jLen = mId.length; j < jLen; j++) {
                    parent = parent[mId[j]] || false;
                }
                relys.push(parent);
            } else {
                relys.push(path[i]);
            }
            i++;
        }
        fn.apply(null, relys);
    };
    window.G = G;
})(window);

G.define("ghostwu.string", function () {
    return {
        trimLeft: function (str) {
            return str.replace(/^\s+/, '');
        },
        trimRight: function () {
            return str.replace(/\s+$/, '');
        },
        trim: function () {
            return str.replace(/^\s+|\s+$/g, '');
        }
    }
});

var str = '跟着ghostwu学习设计模式';
// G.use( ['ghostwu.string'], function( s ){
// console.log( '(' + s.trim( str ) + ')' );
// } );
// G.use( ['ghostwu.string', document], function( s, doc ){
// console.log( doc );
// console.log( '(' + s.trim( str ) + ')' );
// } );
G.use('ghostwu.string', function (s) {
    console.log(s);
    alert('(' + s.trim(str) + ')');
});
```