[toc]

当谈到与处理异步开发在JavaScript中有很多工具可以使用。这篇文章解释了这些工具中的四个和它们的优点。这些是回调，监听器，控制流库和承诺。

## 示例场景

为了说明这四个工具的使用，让我们创建一个简单的示例场景。

让我们说，我们想要找到一些记录，然后处理它们，最后返回处理结果。两个操作（查找和处理）是异步的。

## 回调

让我们从回调模式开始，这是最基本的和最好的已知模式来处理异步编程。

回调看起来像这样：

```
finder([1, 2], function(results) {
  ..do something
});
```

在回调模式中，我们调用一个将执行异步操作的函数。我们传递的参数之一是当操作完成时将被调用的函数。

## 建立

为了说明它们如何工作，我们需要一些能找到并处理记录的函数。在现实世界中，这些函数会产生一个AJAX请求并返回结果，但是现在我们只使用超时。

```
function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(records);
    }, 1000);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(records);
    }, 1000);
}
```

## 使用回调

使用这些函数的代码如下所示：

```
finder([1, 2], function (records) {
    processor(records, function(records) {
      console.log(records);
    });
});
```

我们调用第一个函数，传递一个回调。在这个回调中，我们调用第二个函数传递另一个回调。

通过传递对另一个函数的引用，可以更清楚地写这些嵌套回调。

```
function onProcessorDone(records){
  console.log(records);
}

function onFinderDone(records) {
    processor(records, onProcessorDone);
}

finder([1, 2], onFinderDone);
```

在这两种情况下，控制台日志上面用log [1,2,3,4,5,6]

工作示例：
```
// setup

function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(records);
    }, 500);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(records);
    }, 500);
}

// using the callbacks
finder([1, 2], function (records) {
    processor(records, function(records) {
             console.log(records);       
    });
});

// or

function onProcessorDone(records){
    alert(records);   
}

function onFinderDone(records) {
    processor(records, onProcessorDone);
}

finder([1, 2], onFinderDone);
```
优点

他们是一个非常熟悉的模式，所以他们熟悉，因此容易理解。
很容易在你自己的库/函数中实现。

缺点

嵌套回调将形成如上所示的臭名昭着的末日金字塔，当你有多个嵌套层时，这很难阅读。但这是很容易修复通过拆分功能也如上所示。
您只能为给定事件传递一个回调，在许多情况下，这可能是一个很大的限制。


## 听众

监听器也是一个众所周知的模式，大多被jQuery和其他DOM库流行。侦听器可能如下所示：

```
finder.on('done', function (event, records) {
  ..do something
});
```

我们在添加监听器的对象上调用一个函数。在那个函数中，我们传递我们想要监听的事件的名字和一个回调函数。'on'是这个函数的常用名称之一，你会遇到的其他常用名称是'bind'，'listen'，'addEventListener'，'observe'。

## 建立

让我们为一个监听器演示做一些设置。不幸的是，所需的设置比在回调示例中多一点。

首先，我们需要一些对象来完成查找和处理记录的工作。

```
var finder = {
    run: function (records) {
        var self = this;
        setTimeout(function () {
            records.push(3, 4);
           self.trigger('done', [records]);
        }, 1000);
    }
}
var processor = {
    run: function (records) {
        var self = this;
        setTimeout(function () {
            records.push(5, 6);
            self.trigger('done', [records]);
        }, 1000);
    }
}
```
注意，当工作完成时，他们正在调用方法触发器，我将使用mix-in将这个方法添加到这些对象。再次“触发”是你会遇到的名称之一，其他常用的名字是'火'和'发布'。

我们需要一个具有侦听器行为的mix-in对象，在这种情况下，我将仅仅依靠jQuery：

```
var eventable = {
    on: function(event, cb) {
        $(this).on(event, cb);
    },
    trigger: function (event, args) {
        $(this).trigger(event, args);
    }
}
```

然后将行为应用于我们的finder和处理器对象：

```
$.extend(finder, eventable);
$.extend(processor, eventable);
```

优秀，现在我们的对象可以接听听者和触发事件。

## 使用侦听器

使用侦听器的代码很简单：

```
finder.on('done', function (event, records) {
  processor.run(records);
});
processor.on('done', function (event, records) {
    console.log(records);
});
finder.run([1,2]);
```

同样，控制台运行将输出[1,2,3,4,5,6]

工作示例：

```
// using listeners
var eventable = {
    on: function(event, cb) {
        $(this).on(event, cb);
    },
    trigger: function (event, args) {
        $(this).trigger(event, args);
    }
}
    
var finder = {
    run: function (records) {
            var self = this;
        setTimeout(function () {
            records.push(3, 4);
           self.trigger('done', [records]);            
        }, 500);
    }
}
var processor = {
    run: function (records) {
         var self = this;
        setTimeout(function () {
            records.push(5, 6);
            self.trigger('done', [records]);            
        }, 500);
    }
 }
 $.extend(finder, eventable);
 $.extend(processor, eventable);
    
finder.on('done', function (event, records) {
          processor.run(records);  
    });
processor.on('done', function (event, records) {
    alert(records);
});
finder.run([1,2]);
```

优点

这是另一个很好理解的模式。
最大的优点是你不限于每个对象一个监听器，你可以添加任意多的监听器。例如
```
finder
  .on('done', function (event, records) {
      .. do something
  })
  .on('done', function (event, records) {
      .. do something else
  });
```

缺点

比你自己的代码中的回调更难设置，你可能想使用一个库，例如jQuery，bean.js。


## 流控制库

流控制库也是一个非常好的方式来处理异步代码。我特别喜欢的是Async.js。

使用Async.js的代码看起来像这样：

```
async.series([
    function(){ ... },
    function(){ ... }
]);
```

设置（示例1）

再次，我们需要一些功能来完成这项工作，因为在其他示例中，这些功能在现实世界中可能会创建一个Ajax请求并返回结果。现在让我们使用超时。

```
function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(null, records);
    }, 1000);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(null, records);
    }, 1000);
}
```

节点继续传递风格

注意在上面的函数里面的回调中使用的样式。

1
cb(null, records);
如果没有发生错误，回调中的第一个参数为null; 或错误（如果发生）。这是Node.js库中的常见模式，Async.js使用此模式。通过使用这种风格，Async.js和回调之间的流变得超级简单。

## 使用异步

将消耗这些函数的代码如下所示：

```
async.waterfall([
    function(cb){
        finder([1, 2], cb);
    },
    processor,
    function(records, cb) {
        alert(records);
    }
]);
```

Async.js负责在前一个函数完成后按顺序调用每个函数。注意我们如何才能传递“处理器”函数，这是因为我们使用的是Node继承风格。正如你可以看到，这个代码是很小，很容易理解。

工作示例：

```
// setup
function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(null, records);
    }, 500);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(null, records);
    }, 500);
}

async.waterfall([
    function(cb){
        finder([1, 2], cb);
    },
    processor
    ,
    function(records, cb) {
        alert(records);
    }
]);
```

## 另一种设置（实施例2）

现在，当进行前端开发时，不太可能有一个跟随回调（null，results）签名的库。所以更现实的例子看起来像这样：

```
function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(records);
    }, 500);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(records);
    }, 500);
}

// using the finder and the processor
async.waterfall([
    function(cb){
        finder([1, 2], function(records) {
            cb(null, records)
        });
    },
    function(records, cb){
        processor(records, function(records) {
            cb(null, records);
        });
    },
    function(records, cb) {
        alert(records);
    }
]);
```

它变得更加复杂，但至少你可以看到流从上到下。

工作示例：

```
// setup

function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(records);
    }, 500);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(records);
    }, 500);
}

async.waterfall([
    function(cb){
        finder([1, 2], function(records) {
            cb(null, records)
        });
    },
    function(records, cb){
        processor(records, function(records) {
            cb(null, records);
        });
    },
    function(records, cb) {
        alert(records);
    }
]);
```

优点

通常使用控制流库的代码更容易理解，因为它遵循自然顺序（从上到下）。这不是回调和侦听器。
缺点

如果函数的签名不匹配，如第二个例子，那么你可以认为流控制库提供很少的可读性。


## 承诺

最后，我们到达我们的最终目的地。承诺是一个非常强大的工具，但他们是最不了解的。

使用promise的代码可能如下所示：

```
finder([1,2])
    .then(function(records) {
      .. do something
    });
```
这将有很大的不同取决于你使用的承诺库，在这种情况下，我使用when.js。

## 建立

out finder和处理器功能如下所示：

```
function finder(records){
    var deferred = when.defer();
    setTimeout(function () {
        records.push(3, 4);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}
function processor(records) {
     var deferred = when.defer();
    setTimeout(function () {
        records.push(5, 6);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}
```
每个函数创建一个延迟对象并返回一个promise。然后当结果到达时解决延迟。

## 使用promises

使用这些函数的代码如下所示：

```
finder([1,2])
    .then(processor)
    .then(function(records) {
            alert(records);
    });
```
正如你可以看到，它是相当最小，很容易理解。当这样使用时，promise为你的代码带来了很多清晰，因为他们遵循自然的流程。注意在第一个回调中我们可以简单的传递'处理器'函数。这是因为这个函数返回一个promise本身，所以一切都只是流畅地。

工作示例：

```
// using promises
function finder(records){
    var deferred = when.defer();
    setTimeout(function () {
        records.push(3, 4);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}
function processor(records) {
     var deferred = when.defer();
    setTimeout(function () {
        records.push(5, 6);
        deferred.resolve(records);
    }, 500);
    return deferred.promise;
}

finder([1,2])
    .then(processor)
    .then(function(records) {
            alert(records);
    });
```

有很多承诺：

它们可以作为常规对象传递
聚合成更大的承诺
你可以为失败的promise添加处理程序

## 承诺的大好处

现在如果你认为这是所有的承诺你失去了我认为最大的优势。Promises有一个巧妙的技巧，回调，侦听器或控制流都不能做。你可以添加一个监听器来承诺，即使它已经解决，在这种情况下，监听器将立即触发，这意味着你不必担心，如果事件已经发生，当你添加监听器。这对于聚合promises工作相同。让我告诉你一个例子：

```
function log(msg) {
    document.write(msg + '<br />');
}

// using promises
function finder(records){
    var deferred = when.defer();
    setTimeout(function () {
        records.push(3, 4);
        log('records found - resolving promise');
        deferred.resolve(records);
    }, 100);
    return deferred.promise;
}

var promise = finder([1,2]);

// wait 
setTimeout(function () {
    // when this is called the finder promise has already been resolved
    promise.then(function (records) {
        log('records received');        
    });
}, 1500);
```

这是处理浏览器中的用户交互的巨大功能。在复杂应用程序中，您现在可能不会执行用户将采取的操作顺序，因此您可以使用promise来跟踪使用交互。如果有兴趣，看看这个其他职位。

优点

真的很强大，你可以聚合承诺，传递它们，或者在已经解决时添加监听器。
缺点

最不了解的所有这些工具。
他们可能很难跟踪，当你有很多聚合承诺与一路上添加的听众。

## 结论

而已！在我看来，这是处理异步代码的四个主要工具。希望我帮助你更好地了解他们，并为您提供更多的选择异步需求。