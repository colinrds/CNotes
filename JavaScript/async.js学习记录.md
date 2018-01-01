[toc]

## 常用的场景

**1. 多个异步依次顺序执行。如果有异常抛出时就立即执行回调函数， 回调函数的err为抛出的异常；如果没有异常抛出，当所有异步函数完成后，执行成功回调函数，err值为null, result 为异步数组的结果数组**
```
async.series([
  function(callback) {
    doAsync1(arg, callback);
  },   
  function(callback) {
    doAsync2(arg, callback);
  },
  function(callback) {
    doAsync3(arg, callback);
  }
], function(err, result) {
  console.log(result);
});
```

在线demo: http://jsbin.com/sisasu/edit?js,console,output

**2.多个异步依次顺序执行，且后面异步函数的依赖前面异步函数的输出**

```
async.waterfall([
  function(callback) {
    doAsync1(2, callback);
  },
  function(arg, callback) {
    doAsync2(arg, callback);
  },
  function(arg, callback) {
    doAsync3(arg, callback);
  }
], function(err, result) {
  console.log(result);
});
```

在线demo: http://jsbin.com/yozuje/edit?js,console,output

**3.多个异步并行执行，当所有异步函数执行完成后执行回调函数，回到函数的参数为之前异步函数执行结果的数组，如果需要限制并行执行的数量可以使用parallelLimit**

```
async.parallel([
  function(callback) {
    doAsync1(arg, callback);
  },   
  function(callback) {
    doAsync2(arg, callback);
  },
  function(callback) {
    doAsync3(arg, callback);
  }
], function(err, result) {
  console.log(result);
});
```

在线demo: http://jsbin.com/mugixu/edit?js,console,output

## 数组相关

```
async.each(arr, doAsync, function(err, result){
  console.log(result);
});
// 等同于
async.each(arr, function(item, callback){
  doAsync(item, callback);
}, function(err, result){
  console.log(result);
});
```

在线demo: http://jsbin.com/supuko/edit?js,console,output

## 综合例子

```
var fs = require('fs');
var mysql = require('mysql');
var async = require('async');
 
var connection = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: 'root',
  database: 'rest'
});
 
connection.connect();
 
var ruleid = [277, 276, 278];
 
function exportData(num, cb) {
  async.waterfall([
    function(callback) {
      connection.query('SELECT * from user where name = ?', num, callback);
    },
    function(rows, fields, callback) {
      var result = [];
      rows.forEach(function(item) {
        result.push(data.name);
        result.push(arr.join("\t") + "\n");
      });
      var out = result.join('');
      fs.writeFile(num + '.txt', out, callback);
    }
  ], function(err, result) {
     console.log(num, 'is saved');
     cb(err, num);
  });
};
 
async.each(ruleid, function(item, callback) {
  exportData(item, callback);
}, function done() {
  console.log('ok');
  connection.end();
});
```