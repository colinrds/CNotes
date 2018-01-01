一直以来，很多新手都会经常问，我学完了基础知识，如何做项目？平时在公司工作都是做些什么？其实我想说，只要你找对方法，随便打开一个网站，都能是你的项目。

这里指的面向对象不单单适用于javascript，也适用其他语言。

万物皆对象，所以，任何事物都是有特征(属性)和动作(方法)的，一般拿到一份需求分档，或者你浏览一个网页看到一个画面的时候，脑子里就要有提炼出来的属性和方法的能力，那你才是合格的。

例如一个购物车例子

估计很多人都做过购物车，我就不卖关子， 做任何东西，先宏观思考*
，然后再去处理细节，然后组装起来，就好像组装汽车的道理一样。例如上图，红色的就是属性，黄色的就是方法，抽象出属性和方法，其他都是死的。

假如是刚学前端的同学，可能就会用这种全局化的变量，也叫面向函数编程，缺点就是很乱，代码冗余

```
//商品属性
var name = 'macbook pro'
var description = ''。'
var price = 0;
//商品方法
addOne:funcion(){alert('增加一件商品')},
reduceOne:function(){alert('减少一件商品')},

//购物车属性
var card = ['macbook pro' ,'dell']
var sum = 2,
var allPrice = 22000,
//购物车方法
function addToCart:function(){
    alert('添加到购物车')
}


addToCart()
```
假如是单例模式的思想，可能会这样做,但这样还是不太好。对象太多，可能造成变量重复，项目小还可以接受

```
var product={
    name:'macbook pro',
    description:'',
    price:6660,
    addOne:funcion(){},
    reduceOne:function(){},
    addToCart:function(){
        alert('添加到购物车')
    }
}

/*购物车*/
var cart={
    name:'购物车',
    products:[],
    allPrice:5000,
    sum:0
}
```
假如是有一定经验的人，可能会这样子做。

```
function Product(name,price,des) {
    /*属性 行为 可以为空或者给默认值*/
    this.name = name;
    this.price = price;
    this.description = des;
}
Product.prototype={
    addToCart:function(){
        alert('添加到购物车')
    }
    addOne:funcion(){},
    reduceOne:function(){},
     /*绑定元素*/
    bindDom:function(){
        //在这里进行字符串拼接，
        //例如
        var str = ''
        str +='<div>价格是:'+this.privce+'</div>'
        return str
    },
}

function Card(products,allPrice,sum) {
    /*属性 行为 可以为空或者给默认值*/
    this.products = products;
    this.allPrice = allPrice;
    this.sum = sum
}
Product.prototype={
    getAllPrice:function(){
        alert('计算购物车内商品总价')
    }
}
```

通过创建各种对象例如macbook

```
//后台给的数据
var products= [
    {name:'macbook',price:21888},
    {name:'dell',price:63999}
]

var str = ''
for(var i = 0,len=products.length;i<len;i++) {
    var curName = products[i].name
    var curName = new Product()
    curName.name=products[i].name;
    curName.price=products[i].price;
    str+= curName.bindDom()
}
```
上面这种方式，就降低了耦合性，不管你用什么语言，还是任何javascript框架(模板引擎，jquery,react等)，都是脱离不开上面那段代码的思想，

再来说说，现在mvvm的模式，例如vue，他们不需要获取dom,那么渲染的时候，定义好一个一个的组件就行了。属性全部用{{}}定义好，剩下的就是替换模板，就解决了。

```
data:{
    name ='',
    price='',
    description = ''
},
methods:{
    addToCart:function(){
        alert('添加到购物车')
    }
    addOne:funcion(){},
    reduceOne:function(){},  
}
```

然后page级组件引入这个产品组件，然后循环这个产品组件就好了。

组件化的好处

- 将代码分类管理
- 代码清晰
- 容易维护
- 容易发现问题
- 代码可读性好
- 易于团队化协作

当然这篇文章是为了锻炼抽象化思维的能力，虽然跟javascript模块化的历程也有点搭边，我还希望大家在浏览任何网页的时候，去分析一下，这个模块你来设计，你会怎么设计，能做到解耦和,版本可迭代可维护，利于团队开发吗？