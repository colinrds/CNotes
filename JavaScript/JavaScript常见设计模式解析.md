[toc]

设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。
使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。
毫无疑问，设计模式于己于他人于系统都是多赢的；设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。

## 观察者模式 Observer Pattern

Observer模式也叫观察者模式、订阅/发布模式，是由GoF提出的23种软件设计模式的一种。
Observer模式是行为模式之一，它的作用是当一个对象的状态发生变化时，能够自动通知其他关联对象，自动刷新对象状态，或者说执行对应对象的方法。
这种设计模式可以大大降低程序模块之间的耦合度，便于更加灵活的扩展和维护。

观察者模式包含两种角色：

- 观察者（订阅者）
- 被观察者（发布者）

核心思想：观察者只要订阅了被观察者的事件，那么当被观察者的状态改变时，被观察者会主动去通知观察者，而无需关心观察者得到事件后要去做什么，实际程序中可能是执行订阅者的回调函数。

在各种框架中：vue中的$emit，Angular1.x.x中的$on、$emit、$broadcast，Angular2中的emit...都是最典型的例子。

简单的例子：
假设你是一个班长，要去通知班里的某些人一些事情，与其一个一个的手动调用触发的方法（私下里一个一个通知），不如维护一个列表（建一个群），这个列表存有你想要调用的对象方法（想要通知的人）；
之后每次通知事件的时候只要循环执行这个列表就好了（群发），而不用关心这个列表里有谁。

Javascript中实现一个例子：

```
// 我们向某dom文档订阅了点击事件，当点击发生时，他会执行我们传入的callback
element.addEventListener(‘click’, callback2, false)
element.addEventListener(‘click’, callback2, false)
```

我们用Javascript实现一个简单的播放器：

```
// 一个播放器类
class Player {

  constructor() {
    // 初始化观察者列表
    this.watchers = {}

    // 模拟2秒后发布一个'play'事件
    setTimeout(() => {
      this._publish('play', true)
    }, 2000)

    // 模拟4秒后发布一个'pause'事件
    setTimeout(() => {
      this._publish('pause', true)
    }, 4000)
  }

  // 发布事件
  _publish(event, data) {
    if (this.watchers[event] && this.watchers[event].length) {
      this.watchers[event].forEach(callback => callback.bind(this)(data))
    }
  }

  // 订阅事件
  subscribe(event, callback) {
    this.watchers[event] = this.watchers[event] || []
    this.watchers[event].push(callback)
  }

  // 退订事件
  unsubscribe(event = null, callback = null) {
    // 如果传入指定事件函数，则仅退订此事件函数
    if (callback) {
      if (this.watchers[event] && this.watchers[event].length) {
        this.watchers[event].splice(this.watchers[event].findIndex(cb => Object.is(cb, callback)), 1)
      }

    // 如果仅传入事件名称，则退订此事件对应的所有的事件函数
    } else if (event) {
      this.watchers[event] = []

    // 如果未传入任何参数，则退订所有事件
    } else {
      this.watchers = {}
    }
  }
}

// 实例化播放器
const player = new Player()
console.log(player)

// 播放事件回调函数1
const onPlayerPlay1 = function(data) {
  console.log('1: Player is play, the `this` context is current player', this, data)
}

// 播放事件回调函数2
const onPlayerPlay2 = data => {
  console.log('2: Player is play', data)
}

// 暂停事件回调函数
const onPlayerPause = data => {
  console.log('Player is pause', data)
}

// 加载事件回调函数
const onPlayerLoaded = data => {
  console.log('Player is loaded', data)
}

// 可订阅多个不同事件
player.subscribe('play', onPlayerPlay1)
player.subscribe('play', onPlayerPlay2)
player.subscribe('pause', onPlayerPause)
player.subscribe('loaded', onPlayerLoaded)

// 可以退订指定订阅事件
player.unsubscribe('play', onPlayerPlay2)
// 退订指定事件名称下的所有订阅事件
player.unsubscribe('play')
// 退订所有订阅事件
player.unsubscribe()

// 可以在外部手动发出事件（真实生产场景中，发布特性一般为类内部私有方法）
player._publish('loaded', true)
```

举个Vue中的例子吧：

```
// 事件发布者使用'vm.$emit、vm.$dispatch(vue1.0)、vm.$broadcast(vue1.0)发布事件
// 接受方使用$on方法或组件监听器订阅事件，传递一个回调函数
vm.$emit(event, […args]) // publish
vm.$on(event, callback) // subscribe
vm.$off([event, callback]) // unsubscribe

// 或者组件中监听事件
<component @event="callback" />

// 在Vue中无论是$on方法还是组件监听事件最终都会转化为实例中的监听器
```

各框架中观察者模式的实现：
Angularjs（AngularJS 1.x.x）中的实现；
同样，Vue中使用Object.defineProperty()实现对数据的双向绑定，在数据变更时，使用notify广播事件，最终同样执行对应属性所维护的Watchers列表进行回调。

## 中介者模式 Mediator Pattern

中介者在程序设计中非常常见，和观察者模式实现的功能非常相似。

形式上：不像观察者模式那样通过调用pub/sub的形式来实现，而是通过一个中介者统一来管理。

实质上：观察者模式通过维护一堆列表来管理对象间的多对多关系，中介者模式通过统一接口来维护一对多关系，且通信者之间不需要知道彼此之间的关系，只需要约定好API即可。

简单说：就像一辆汽车的行驶系统，观察者模式中，你需要知道车内坐了几个人（维护观察者列表），当汽车发生到站、停车、开车...这些事件（被订阅者事件）时，你需要给这个列表中订阅对应事件的的每个人进行通知；
在中介者模式中，你只需要在车内发出广播（到站啦、停车啦、上车啦...请文明乘车尊老爱幼啦...），而不用关心谁在车上，谁要上车谁要下车，他们自己根据广播做自己要做的事，哪怕他不听广播，听了也不做自己要做的事都无所谓。

中介者模式包含两种角色：

- 中介者（事件发布者）
- 通信者

Javascript中实现一个例子：

```
// 汽车
class Bus {

  constructor() {

    // 初始化所有乘客
    this.passengers = {}
  }

  // 发布广播
  broadcast(passenger, message = passenger) {
    // 如果车上有乘客
    if (Object.keys(this.passengers).length) {

      // 如果是针对某个乘客发的，就单独给他听
      if (passenger.id && passenger.listen) {

        // 乘客他爱听不听
        if (this.passengers[passenger.id]) {
          this.passengers[passenger.id].listen(message)
        }

      // 不然就广播给所有乘客
      } else {
        Object.keys(this.passengers).forEach(passenger => {
          if (this.passengers[passenger].listen) {
            this.passengers[passenger].listen(message)
          }
        })
      }
    }
  }

  // 乘客上车
  aboard(passenger) {
    this.passengers[passenger.id] = passenger
  }

  // 乘客下车
  debus(passenger) {
    this.passengers[passenger.id] = null
    delete this.passengers[passenger.id]
    console.log(`乘客${passenger.id}下车`)
  }

  // 开车
  start() {
    this.broadcast({ type: 1, content: '前方无障碍，开车！Over'})
  }

  // 停车
  end() {
    this.broadcast({ type: 2, content: '老司机翻车，停车！Over'})
  }
}

// 乘客
class Passenger {

  constructor(id) {
    this.id = id
  }

  // 听广播
  listen(message) {
    console.log(`乘客${this.id}收到消息`, message)
    // 乘客发现停车了，于是自己下车
    if (Object.is(message.type, 2)) {
      this.debus()
    }
  }

  // 下车
  debus() {
    console.log(`我是乘客${this.id}，我现在要下车`, bus)
    bus.debus(this)
  }
}

// 创建一辆汽车
const bus = new Bus()

// 创建两个乘客
const passenger1 = new Passenger(1)
const passenger2 = new Passenger(2)

// 俩乘客分别上车
bus.aboard(passenger1)
bus.aboard(passenger2)

// 2秒后开车
setTimeout(bus.start.bind(bus), 2000)

// 3秒时司机发现2号乘客没买票，2号乘客被驱逐下车
setTimeout(() => {
  bus.broadcast(passenger2, { type: 3, content: '同志你好，你没买票，请下车!' })
  bus.debus(passenger2)
}, 3000)

// 4秒后到站停车
setTimeout(bus.end.bind(bus), 3600)

// 6秒后再开车，车上已经没乘客了
setTimeout(bus.start.bind(bus), 6666)
```

上面例子中（当然，稍微扩展了点哈），Bus即为中介者对象，乘客为通信者，乘客具有一些统一的方法API，Bus只管开车停车发广播，执行自己的事物，乘客在不断地接受广播，根据广播信息的类型和内容作出自己的判断，执行事务。

## 代理模式 Proxy Pattern

简单说就是：为对象提供一种代理以控制对这个对象的访问。

代理模式使得代理对象控制具体对象的引用。代理几乎可以是任何对象：文件，资源，内存中的对象，或者是一些难以复制的东西。

举个例子: 一个工厂制造商品（目标对象），你可以给这个工厂设置一个业务代理（代理对象），提供流水线管理，订单，运货，淘宝网店等多种行为能力（扩展属性）。
当然，里面还有最关键的一点就是，这个代理能把一些骗纸和忽悠都过滤掉，将最真实最直接的订单给工厂，让工厂能够专注于生产（控制访问）。

上面工厂的例子：

```
// 真实工厂
class Factory {

  constructor(count) {
    // 工厂默认有1000件产品
    this.productions = count || 1000
  }

  // 生产商品
  produce(count) {
    // 原则上低于5个工厂是不接单的
    this.productions += count
  }

  // 向外批发
  wholesale(count) {
    // 原则上低于10个工厂是不批发的
    this.productions -= count
  }
}

// 代理工厂
class ProxyFactory extends Factory {

  // 代理工厂默认第一次合作就从工厂拿100件库存
  constructor(count = 100) {
    super(count)
  }

  // 代理工厂向真实工厂下订单之前会做一些过滤
  produce(count) {
    if (count > 5) {
      super.produce(count)
    } else {
      console.log('低于5件不接单')
    }
  }

  wholesale(count) {
    if (count > 10) {
      super.wholesale(count)
    } else {
      console.log('低于10件不批发')
    }
  }

  taobao(count) {
      // ...
  }

  logistics() {
      // ...
  }
}

// 创建一个代理工厂
const proxyFactory = new ProxyFactory()

// 通过代理工厂生产4件商品，被拒绝
proxyFactory.produce(4)

// 通过代理工厂批发20件商品
proxyFactory.wholesale(20)

// 代理工厂的剩余商品 80
console.log(proxyFactory.productions)
```

ES6中的Proxy对象：

ES6中Proxy对象可以理解为：在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为"代理器"。

基本形式：

```
// 参数分别为目标对象和代理解析器
var proxy = new Proxy(target, handler)
```

无操作转发代理：

```
const target = {}
const p = new Proxy(target, {})
p.a = 3  // 被转发到代理的操作
console.log(target.a) // 3 操作已经被正确地转发至目标对象
```

使用错误拦截属性读取操作：

```
const handler = {
    get(target, property) {
        if (property in target) {
            return target[property]
        } else {
            throw new ReferenceError("Property \"" + property + "\" does not exist.")
        }
    }
}

const p = new Proxy({}, handler)
p.a = 1
p.b = undefined

console.log(p.a, p.b) // 1, undefined
console.log('c' in p, p.c) // Uncaught ReferenceError: Property "c" does not exist.
```

实现一个service客户端：

```
function createWebService(baseUrl) {
  return new Proxy({}, {
    get(target, propKey, receiver) {
      return () => httpGet(baseUrl+'/' + propKey)
    }
  })
}

const serviceA = createWebService('http://example.com/data-a')
const serviceB = createWebService('http://example.com/data-b')
const serviceC = createWebService('http://example.com/data-c')

serviceA.employees().then(json => {
  const employees = JSON.parse(json)
  // ···
})

serviceB...
```

## 单例模式 Singleton Pattern

简单说：保证一个类只有一个实例，并提供一个访问它的全局访问点（调用一个类，任何时候返回的都是同一个实例）。

实现方法：使用一个变量来标志当前是否已经为某个类创建过对象，如果创建了，则在下一次获取该类的实例时，直接返回之前创建的对象，否则就创建一个对象。

类/构造函数实例：

```
class Singleton {

  constructor(name) {
    this.name = name
    this.instance = null
  }

  getName() {
    alert(this.name)
  }

  static getInstance(name) {
    if (!this.instance) {
      this.instance = new Singleton(name)
    }
    return this.instance
  }
}

const instanceA = Singleton.getInstance('seven1')
const instanceB = Singleton.getInstance('seven2')

console.log(instanceA, instanceB)
```

闭包包装实例：

```
const SingletonP = (function() {
  let instance
  return class Singleton {

    constructor(name) {
      if (instance) {
        return instance
      } else {
        this.init(name)
        instance = this
        return this
      }
    }

    init(name) {
      this.name = name
      console.log('已初始化')
    }
  }
})()

const instanceA = new SingletonP('seven1')
const instanceB = new SingletonP('seven2')

console.log(instanceA, instanceB)
```

惰性包装实例：

```
const getSingle = function (fn) {
    let result
    return function() {
        return result || (result = fn.apply(this, arguments))
    }
}
```

## 工厂模式 Factory Pattern

与创建型模式类似，工厂模式创建对象（视为工厂里的产品）时无需指定创建对象的具体类。
工厂模式定义一个用于创建对象的接口，这个接口由子类决定实例化哪一个类。该模式使一个类的实例化延迟到了子类。而子类可以重写接口方法以便创建的时候指定自己的对象类型。

简单说：假如我们想在网页面里插入一些元素，而这些元素类型不固定，可能是图片、链接、文本，根据工厂模式的定义，在工厂模式下，工厂函数只需接受我们要创建的元素的类型，其他的工厂函数帮我们处理。

上代码：

```
// 文本工厂
class Text {
    constructor(text) {
        this.text = text
    }
    insert(where) {
        const txt = document.createTextNode(this.text)
        where.appendChild(txt)
    }
}

// 链接工厂
class Link {
    constructor(url) {
        this.url = url
    }
    insert(where) {
        const link = document.createElement('a')
        link.href = this.url
        link.appendChild(document.createTextNode(this.url))
        where.appendChild(link)
    }
}

// 图片工厂
class Image {
    constructor(url) {
        this.url = url
    }
    insert(where) {
        const img = document.createElement('img')
        img.src = this.url
        where.appendChild(img)
    }
}

// DOM工厂
class DomFactory {

  constructor(type) {
    return new (this[type]())
  }

  // 各流水线
  link() { return Link }
  text() { return Text }
  image() { return Image }
}

// 创建工厂
const linkFactory = new DomFactory('link')
const textFactory = new DomFactory('text')

linkFactory.url = 'https://surmon.me'
linkFactory.insert(document.body)

textFactory.text = 'HI! I am surmon.'
textFactory.insert(document.body)
```

## 装饰者模式 Decorative Pattern

装饰者(decorator)模式能够在不改变对象自身的基础上，在程序运行期间给对像动态的添加职责（方法或属性）。与继承相比，装饰者是一种更轻便灵活的做法。

简单说：可以动态的给某个对象添加额外的职责，而不会影响从这个类中派生的其它对象。

实例：假设同事A在window.onload中指定了一些任务，这个函数由同事A维护，如何在对window.onload函数不进行任何修改的基础上，在window.onload函数执行最后执行自己的任务？

Show me the code:

```
// 同事A的任务
window.onload = () => {
    console.log('window loaded!')
}

// 装饰者
let _onload= window.onload || function () {}
window.onload = () => {
    _onload()
    console.log('自己的处理函数')
};
```

如何在所有函数执行前后分别执行指定函数：

```
// 新添加的函数在旧函数之前执行
Function.prototype.before = function (beforefn) {
    let _this = this
    return function () {
        beforefn.apply(this, arguments)
        return _this.apply(this, arguments)
    }
}

// 新添加的函数在旧函数之后执行
Function.prototype.after = function(afterfn) {
    let _this = this
    return function () {
        let ret = _this.apply(this, arguments)
        afterfn.apply(this, arguments)
        return ret
    }
}

// 使用
var func = function(param) {
    console.log(param)
}

func = func.before(function(param) {
    param.name = 'beforename'
})

func({ name: 'func' }) // { name: 'beforename' }
```

不污染Function原型的做法：

```
// 装饰器
const before = function(fn, before) {
    return function() {
        before.apply(this, arguments)
        return fn.apply(this, arguments)
    }
}

// 普通函数
function a() { console.log('a') }
function b() { console.log('b') }

// 使用装饰器执行函数
const c = before(a, b)

c() // b a
模拟传统语言的装饰者：

// 飞机
class Plane {

  constructor(name) {
    this.name = name
  }

  // 发射子弹
  fire() {
    console.log('发射普通子弹')
  }
}

// 武器加强版（装饰类）
class MissileDecorator {

  constructor(plane) {
    this.plane = plane
    this.plane.name = '高级飞机'
  }

  fire() {
    this.plane.fire()
    console.log('发射导弹')
  }
}

let plane = new Plane('普通飞机')
plane = new MissileDecorator(plane)
plane.fire()

// 发射普通子弹
// 发射导弹
```

使用ES7中的装饰器：

首先需要搞清楚ES6中Class语法糖的背后工作原理：

```
class Cat {
    say() {
        console.log("meow ~")
    }
}

// 实际上当我们给一个类添加一个属性的时候，会调用到 Object.defineProperty 这个方法，它会接受三个参数：target 、name 和 descriptor ，上面的Class本质等同于：
function Cat() {}
Object.defineProperty(Cat.prototype, 'say', {
    value: function() { console.log("meow ~"); },
    enumerable: false,
    configurable: true,
    writable: true
})
```

ES7装饰器基本示例：

```
function isAnimal(target) {
    target.isAnimal = true
    return target
}

// 装饰器
@isAnimal
class Cat {
    // ...
}
console.log(Cat.isAnimal)    // true

// 上面装饰器代码基本等同于
Cat = isAnimal(function Cat() { ... })
```

作用于类属性的装饰器：

```
function readonly(target, name, descriptor) {
    discriptor.writable = false
    return discriptor
}

class Cat {
    @readonly
    say() {
        console.log("meow ~")
    }
}

var kitty = new Cat()
kitty.say = function() {
    console.log("woof !")
}
kitty.say()    // meow ~
```

在类的属性中定义装饰器的时候，参数有三个：target、name、descriptor，上面说了，因为装饰器在作用于属性的时候，实际上是通过Object.defineProperty来进行扩展和封装的。

所以在上面的这段代码中，装饰器实际的作用形式是这样的：

```
let descriptor = {
    value: function() {
        console.log("meow ~")
    },
    enumerable: false,
    configurable: true,
    writable: true
}
descriptor = readonly(Cat.prototype, 'say', descriptor) || descriptor
Object.defineProperty(Cat.prototype, 'say', descriptor)
```

这里也是 JS 里装饰器作用于类和作用于类的属性的不同的地方。
当装饰器作用于类本身的时候，我们操作的对象也是这个类本身，而当装饰器作用于类的某个具体的属性的时候，我们操作的对象既不是类本身，也不是类的属性，而是它的描述符（descriptor），
而描述符里记录着我们对这个属性的全部信息，所以，我们可以对它自由的进行扩展和封装，最后达到的目的呢，就和之前说过的装饰器的作用是一样的。

也可以直接在 target 上进行扩展和封装，比如：

```
function fast(target, name, descriptor) {
    target.speed = 20
    let run = descriptor.value
    descriptor.value = function() {
        run()
        console.log(`speed ${this.speed}`)
    }
    return descriptor;
}

class Rabbit {
    @fast
    run() {
        console.log("running~")
    }
}

var bunny = new Rabbit()
bunny.run()
// running~
// speed 20
console.log(bunny.speed)   // 20
```

总结：装饰器允许你在类和方法定义的时候去注释或者修改它。装饰器是一个作用于函数的表达式，它接收三个参数target、name和descriptor，然后可选性的返回被装饰之后的descriptor对象。

装饰者模式和代理模式的区别：

- 代理模式的目的是，当直接访问本体不方便或者不符合需要时，为这个本体提供一个代替者。本体定义了关键功能，而代理提供了或者拒绝对他的访问，或者是在访问本体之前做一些额外的事情。
- 装饰者模式的作用就是为对象动态的加入某些行为。