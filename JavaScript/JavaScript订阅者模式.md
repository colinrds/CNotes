我们来尝试一个自定义的发布订阅模式，那么如何实现发布订阅呢

- 指定一个发布者
- 给发布者添加一个缓冲列表，用于存放回调函数以用于通知订阅者
- 发布消息时，发布者遍历缓存列表，依次触发每个订阅者的回调函数

一个简单的天气状态订阅
```
var Weather = {
    list: [], // 缓存列表
    listen: function(fn) { // 增加订阅者  
        this.list.push(fn)
    },
    publish: function() { // 发布消息
        for(var i=0,fn; fn=this.list[i++];) {
            fn.apply(this,arguments);
        }
    }
};

// 订阅消息
Weather.listen(function(weather, wind){
    console.log('天气：' + weather, '风力：'+ wind);
})

// 发布消息
Weather.publish("晴天","微风"); // 天气：晴天 风力：微风
Weather.publish("雷阵雨","5级风"); // 天气：雷阵雨 风力：5级风
```

以上，已经实现了一个最简单的发布—订阅模式，还可以为订阅者增加自选功能，订阅自己想要的消息，也可以增加取消订阅的事件。

```
var PubSub = {
    list: [],
    listen: function(key, fn){
        if(!this.list[key]) {
            this.list[key]=[];
        }
        this.list[key].push(fn);
    },
    publish: function(){
        var key = Array.prototype.shift.call(arguments),
            fns = this.list[key];
        if(!fns || fns.length === 0) {
            return false;
        }
        for(var i = 0, fn; fn = fns[i++];){
            fn.apply(this, arguments);
        }
    }
}

//
var installEvent = function(obj) {
    for (var i in PubSub) {
        obj[i] = PubSub[i];
    }
};

var day = {}
installEvent(day);

day.listen('天气', function(wind) {
    console.log('风力：'+ wind);
});

day.publish('天气', "8级风");
```