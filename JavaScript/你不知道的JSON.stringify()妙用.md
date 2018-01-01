[toc]

## 语法

```
JSON.stringify(value[, replacer[, space]])
```

一般用法：

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user);
"{"name":"andy","isDead":false,"age":11,"addr":"shanghai"}"
```

## 扩展用法

### replacer

replacer可以是函数或者是数组。

- 功能1: 改变属性值

将isDead属性的值翻译成0或1，0对应false,1对应true

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user, function(key, value){
    if(key === 'isDead'){
        return value === true ? 1 : 0;
    }
    return value;
});
"{"name":"andy","isDead":0,"age":11,"addr":"shanghai"}"
```

- 功能2：删除某个属性

将isDead属性删除，如果replacer的返回值是undefined,那么该属性会被删除。

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user, function(key, value){
    if(key === 'isDead'){
        return undefined;
    }
    return value;
});
"{"name":"andy","age":11,"addr":"shanghai"}"
```

功能3: 通过数组过滤某些属性
只需要name属性和addr属性，其他不要。

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user, ['name', 'addr']);
"{"name":"andy","addr":"shanghai"}"
```

### space
space可以是数字或者是字符串, 如果是数字则表示属性名前加上空格符号的数量，如果是字符串，则直接在属性名前加上该字符串。

- 功能1: 给输出属性前加上n个空格

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user, null, 4);
"{
    "name": "andy",
    "isDead": false,
    "age": 11,
    "addr": "shanghai"
}"
```

- 功能2: tab格式化输出

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
JSON.stringify(user, null, '\t');
"{
    "name": "andy",
    "isDead": false,
    "age": 11,
    "addr": "shanghai"
}"
```

- 功能3： 搞笑

```
JSON.stringify(user, null, 'good');
"{
good"name": "andy",
good"isDead": false,
good"age": 11,
good"addr": "shanghai"
}"
```

###  深拷贝

```
var user = {name: 'andy', isDead: false, age: 11, addr: 'shanghai'};
var temp = JSON.stringify(user);
var user2 = JSON.parse(temp);
```

## 其他
JSON.parse() 其实也是支持第二个参数的。功能类似于JSON.stringify的第二个参数的功能。