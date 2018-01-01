## 简介

职责链模式（Chain of responsibility）是使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理他为止。 也就是说，请求以后，从第一个对象开始，链中收到请求的对象要么亲自处理它，要么转发给链中的下一个候选者。提交请求的对象并不明确知道哪一个对象将会处理它——也就是该请求有一个隐式的接受者（implicit receiver）。根据运行时刻，任一候选者都可以响应相应的请求，候选者的数目是任意的，你可以在运行时刻决定哪些候选者参与到链中。

## 案例一

果伯伯在水果市场卖苹果，果果老板说：苹果分优质，中等，一般，给钱就卖四大类苹果。
优质苹果5元一斤，中等苹果4元一斤，一般苹果3元一斤，给钱就卖苹果1元一斤，一次最少卖一斤（这是例子，现实中不是这样的）

- 小明拿5元钱在这个摊上买1斤苹果，
- 小亮拿4元钱也在这个摊上买1斤苹果，
- 小帅拿3元钱也在这个摊上买1斤苹果，
- 小七拿1元钱也在这个摊上买1斤苹果，

分析：

- 小明、小亮、小帅、小七都是拿钱去买苹果的，他们就好比【一个个的请求】;
- 果伯伯的摊位就可以当做一个职责链，处理不同的买方请求。
- 苹果价位：price=5,4,3,1,
- 小明、小亮、小帅、小七的钱：people$=5,4,3,1

第一个版本：

```
/**
 * 买苹果
 * @name {string} 姓名
 * @price {Number} 价格
 * @weight {Number} 重量
 * 
 * */
function buyapple(name,price,weight){
    if(price==5){
        console.log(name+'买到了5元1斤的苹果');
    }else if(price==4){
        console.log(name+'买到了4元1斤的苹果');
    }else if(price==3){
        console.log(name+'买到了3元1斤的苹果');
    }else if(price==1){
        console.log(name+'买到了1元1斤的苹果');
    }
}
buyapple('小明',5,1);
buyapple('小亮',4,1);
buyapple('小帅',3,1);
buyapple('小七',1,1);
```

上述代码也许是最简单代码，最初级的程序员都会写，但是这样写的代码if嵌套比较多，更改维护比较费劲。假设又有2元一斤的苹果与用户，那就需要更改源码了。

第二个版本：

```
function buyapple5(name,price,weight){
    if(price==5){
       console.log(name+'买到了5元1斤的苹果');
    }else{
        return 'next';
    }
}
function buyapple4(name,price,weight){
    if(price==4){
       console.log(name+'买到了4元1斤的苹果');
    }else{
        return 'next';
    }
}
function buyapple3(name,price,weight){
    if(price==3){
       console.log(name+'买到了3元1斤的苹果');
    }else{
        return 'next';
    }
}
function buyapple1(name,price,weight){
    if(price==1){
       console.log(name+'买到了1元1斤的苹果');
    }else{
        return 'next';
    }
}

function Buyapple(fn){
    this.fn=fn;
    this.nextfn=null;//把下一个执行设置为null
}
Buyapple.prototype.setNextfn=function(nextfn){
    return this.nextfn=nextfn;//设置下一个执行的函数
}
Buyapple.prototype.Nextfn=function(){
   var res = this.fn.apply(this,arguments);//执行当前函数  
   if( res === "next" ){  
        //判断当前函数是否有下一个执行函数，有则执行
      return this.nextfn && this.nextfn.Nextfn.apply(this.nextfn,arguments);  
   }  
}

//现在我们吧3个订单函数分别包装成职责链的节点  
var chainOrder5 = new Buyapple(buyapple5);  
var chainOrder4 = new Buyapple(buyapple4); 
var chainOrder3 = new Buyapple(buyapple3)
var chainOrder1 = new Buyapple(buyapple1);  
//然后指定节点在指责链的顺序  
chainOrder5.setNextfn(chainOrder4);  
chainOrder4.setNextfn(chainOrder3);
chainOrder3.setNextfn(chainOrder1);
//最后传递请求  最上层，过滤数据执行
chainOrder5.Nextfn('小帅',3,1);
```

目前的职责链能够保证按照顺序执行下去。可以随意组合。但现在的代码只能保证是同步执行的。如果有异步中间执行代码则无法执行。

第三个版本：

```
function buyapple5(name,price,weight){
    if(price==5){
       console.log(name+'买到了5元1斤的苹果');
    }else{
        return 'next';
    }
}
function buyapple4(name,price,weight){
    var that=this;
    if(price==4){
       console.log(name+'买到了4元1斤的苹果');
    }
    setTimeout(function(){
        console.log('等待两秒后执行');
        that.next(name,price,weight);
    },2000);

}
function buyapple3(name,price,weight){
    if(price==3){
       console.log(name+'买到了3元1斤的苹果');
    }else{
        return 'next';
    }
}
function buyapple1(name,price,weight){
    if(price==1){
       console.log(name+'买到了1元1斤的苹果');
    }else{
        return 'next';
    }
}

function Buyapple(fn){
    this.fn=fn;
    this.nextfn=null;//把下一个执行设置为null
}
Buyapple.prototype.setNextfn=function(nextfn){
    return this.nextfn=nextfn;//设置下一个执行的函数
}
Buyapple.prototype.Nextfn=function(){
   var res = this.fn.apply(this,arguments);//执行当前函数  
   if( res === "next" ){  
        //判断当前函数是否有下一个执行函数，有则执行
      return this.nextfn && this.nextfn.Nextfn.apply(this.nextfn,arguments);  
   }  
}
Buyapple.prototype.next=function(){
        //异步函数是无法返回next，只有借助next函数触发执行
      return this.nextfn && this.nextfn.Nextfn.apply(this.nextfn,arguments);  
}
//现在我们吧3个订单函数分别包装成职责链的节点  
var chainOrder5 = new Buyapple(buyapple5);  
var chainOrder4 = new Buyapple(buyapple4); 
var chainOrder3 = new Buyapple(buyapple3)
var chainOrder1 = new Buyapple(buyapple1);  
//然后指定节点在指责链的顺序  
chainOrder5.setNextfn(chainOrder4);  
chainOrder4.setNextfn(chainOrder3);
chainOrder3.setNextfn(chainOrder1);
//最后传递请求  最上层，过滤数据执行
chainOrder5.Nextfn('小帅',3,1);
```

![image](http://ow2n75eab.bkt.clouddn.com/7186abfd89941dcdccc7082ee5977fce.png)

## connect框架next函数


```
var i=0,//索引
_arrycallback=[];//存储函数
//执行下一个函数
function next(){
    i++;
    _arrycallback[i] && _arrycallback[i]();
}
//存储next函数
function push_next(fn){
    _arrycallback.push(fn);
};

push_next(function(){
    console.log("1");
    next();
});
push_next(function(){
    setTimeout(function(){
        console.log("2","延迟500毫秒输出");
        next();
    },3000)

});
push_next(function(){
    console.log("3");
    next();
});
push_next(function(){
    console.log("4");
    next();
});
//执行
_arrycallback[0]();
```

![image](http://ow2n75eab.bkt.clouddn.com/5331fbce52f28211fd76ac670961e551.png)

Node.js 中大名鼎鼎的connect框架正是这样实现中间件队列的。有兴趣可以去看看它的源码或者这篇解读《何为 connect 中间件》。

## 总结

1. 职责链模式的最大优点就是解耦了请求必送者和N个接收者之间的复杂关系，由于不知道链中的如个节点可以处理你发现的请求，所以你只需要请求传递给第一个节点即可。
2. 使用了职责链模式之后，链中的节点对象可以灵活地拆分重组。增加或者删除一个节点，或者改变节点在链中的位置都是轻而易举的事情。
3. 在职责模式还有一个优点，那就是可以手动指定起始节点，请求并不是非得从链中的第一个节点开始传递。
4. 职责链模式使得程序中多了一些节点对象，可能在某一次的请求过程中，大部分节点并没有起到实质性的作用，它们的作用仅仅是让请求传递下去，从性能方面考虑，我们要避免过长的职责链带来性能损耗。
5. 如果一个节点出错，就会导致整个链条断开。