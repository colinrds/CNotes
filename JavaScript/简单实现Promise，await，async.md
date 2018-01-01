[toc]

我们处理异步的方式，从开始的回调，到Promise，再到现在的async，await，变得越来越方便，直观了。但是知其然要知其所以然，所以我们一步步来分析他们是如何实现的（需要知道Promise的使用方法，await async的使用方法，以及generator的功能）。

## Promise

```
function process() {
  setTimeout(() => {
    console.log("timeout")
  }, 1000)
}
```

如果我们想要执行上面的函数，并且在计时结束后的回调中执行某些动作，应该怎么办呢？
我们可以使用一个回调：

```
function process(callback) {
  setTimeout(() => {
    console.log("timeout")
     callback()
  }, 1000)
}

process(function doSomething() {
  console.log("do something")
})
```

这是我们正常的回调方式，但是这样的方式不够灵活，而且当需要嵌套回调的时候，往往就写成了callback hell。Promise其实就是对上面的process进行一层包装。
下面实现一个最简单的Promise

```
function Promise(fn) {
  const callbacks = [];

  this.then(calback) {
    callbacks.push(callback)
  }

  function resolve(value) {
    callbacks.forEach(callback => callback && callback(value))
  }

  fn(resolve)
  return this
} 
```

从代码可以看到，then函数其实就是把传入的函数放到一个回调数组里，这也让原本单个的回调变成支持许多回调。resolve函数就是传回我们将要执行的函数中的回调，他会调用每一个then里面添加的回调。最后，执行传入的函数，并且返回this

可以向正常的Promise一样使用：

```
var p = Promise(function process(resolve) {
  setTimeout(() => {
    console.log("timeout")
     resolve()
  }, 1000)
})

p.then(() => console.log("timeout!!"))
```

写到这里，也就明白resolve，reject是怎么实现的了。

## await async

**generator**

await 和 async，需要在generator的基础上来实现。generator可以实现控制函数间执行顺序的功能。可能说的有些抽象，但是理解了之后就感觉不难了。

```
function generateValue() {
  console.log("first")
  yield "hello"
  console.log("second")
  return "world"
}

var g = generateValue()
console.log(g.next())
console.log(g.next())

// 输出：
// first
// { value: 'hello', done: false }
// second
// { value: 'world', done: true }
```

这是一个生成器函数，在其他语言也有类似的语法。在JavaScript中需要在函数名称前带上*才能使用yield来使用生成器。

generator函数的特点是，每次执行到yield时，执行权交到调用它的地方，等待下一次调用next()之后才会继续执行。

## await async

那么generator与await和async是什么关系呢？

同样的，我们先看问题
(axios是一个强大的HTTP client，可以同时用于web和node)

```
axios.get("https://www.github.com")
.then(result => console.log(result))
```
代码中，如果我们想要正常的输出结果，就可以在一个then的调用中来执行log。但是怎么样不使用Promise，而用更加自然的方式实现呢

```
function syncDemo() {
  const result = yield axios.get("https://www.github.com")
  console.log('after get result', result)
}
我们利用yield，在上面的代码中，如果执行syncDemo()，函数执行到yield时会把代码的控制权转到调用函数的地方，于是参考generator的使用方法：

var g = syncDemo()
var p = g.next().value
p.then(result => g.next(result))
```

在代码中g.next(result)中的result会传回generator中赋给result，然后继续执行。

这就到了关键的地方，可以看到，利用generator的这个特性，我们已经实现了一个可以顺序执行的异步函数，但是在调用这个函数的时候，需要控制执行的过程。那么是不是可以写一个函数专门来自动执行这个步骤呢？

```
function runGenerator(fn) {
  var g = fn()
  g.next().value.then(result => {
    g.next(result)
  })
}
```

上面就是一个非常简单的执行函数。

调用：

```
runGenerator(syncDemo) 
```

syncDemo就自动执行了。

而JavaScript的async，await做的也是同样的事情。
