## 1.forEach迭代器

forEach方法接收一个函数作为参数，对数组中每个元素使用这个函数，只调用这个函数，数组本身没有任何变化

```
//forEach迭代器
function square(num){
    document.write(num + ' ' + num*num + '<br>');
}

var nums = [1,2,3,4,5,6,7,8];
nums.forEach(square);
```
在浏览器中输出的结果是：
```
1  1
2  4
3  9
4  16
5  25
6  36
7  49
8  64
```

## 2.every迭代器

every方法接受一个返回值为布尔类型的函数，对数组中的每个元素使用这个函数，如果对于所有的元素，该函数均返回true，则该方法返回true，否则返回false
 
~~~
//every迭代器
function isEven(num){
    return num % 2 == 0;
}
var nums = [2,4,6,8];
document.write(nums.every(isEven));
~~~


## 3.some迭代器

some方法也是接受一个返回值为布尔类型的函数，只要有一个元素使得该函数返回true，该方法就返回true

~~~
//some迭代器
function isEven(num){
    return num % 2 == 0;
}
var nums = [1,3,5,7];
document.write(nums.some(isEven));
~~~

## 4.reduce迭代器


reduce方法接受一个函数，返回一个值，该方法从一个累加值开始，不断对累加值和数组中的后续元素调用该函数，知道数组中最后一个元素，最后得到返回的累加值

~~~
//reduce迭代器
function add(runningTotal, currentValue){
    return runningTotal + currentValue;
}
var nums = [1,2,3,4,5,6,7,8,9,10];
var sum = nums.reduce(add);
document.write(sum);
~~~

得到的结果是：55

reduce()函数和add()函数一起，从左到右，一次对数组中的元素求和，执行过程如下：
 
~~~
add(1,2) -> 3
add(3,3) -> 6
add(6,4) -> 10
add(10,5) -> 15
add(15,6) -> 21
add(21,7) -> 28
add(28,8) -> 36
add(36,9) -> 45
add(45,10) -> 55
~~~


reduce方法也可以用来将数组中的元素链接成一个长的字符串，代码如下

~~~
//使用reduce连接数组元素
function concat(accumulatedString, item){
    return accumulatedString + item;
}
var words = ['the ', 'quick ', 'brown ', 'fox'];
var sentence = words.reduce(concat);
document.write(sentence);
~~~

最后输出结果如下：

```
the quick brown fox
```

javascript还提供了reduceRight方法，和Reduce方法不同，它是从右到左执行，如下：

~~~
//使用reduce连接数组元素
function concat(accumulatedString, item){
    return accumulatedString + item;
}
var words = ['the ', 'quick ', 'brown ', 'fox '];
var sentence = words.reduceRight(concat);
document.write(sentence);
~~~

执行结果如下：

```
fox brown quick the
```


## 5.map迭代器


map迭代器和forEach有些类似，但是map会改变数组，生成新的数组，如下代码

~~~
//使用map迭代器生成新的数组
function curve(grade){
    return grade+5;
}
var grades = [77,65,81,92,83];
var newgrades = grades.map(curve);
document.write(newgrades);
~~~
 

输出结果：

```
82,70,86,97,88
```

## 6.fiter迭代器

和every迭代器类似，传入一个返回值为布尔类型的函数，和every方法不同的是，当数组中所有元素对应该函数返回的结果均为true时，该方法并不返回true，而是返回一个新的数组，该数组包含对应函数返回结果为true的元素，代码如下

~~~
function isEven(num){
    return num % 2 == 0;
}

function isOdd(num){
    return num % 2 != 0;
}

var nums = [];
for (var i=0; i<20; i++) {
    nums[i] = i+1;
}
var evens = nums.filter(isEven);
document.write(evens);
document.write('<br>');
var odds = nums.filter(isOdd);
document.write(odds);
~~~

输出结果如下：

```
2，4，6，8，10，12，14，16，18，20

1，3，5，7，9，11，13，15，17，19
```

## 总结

以上就是关于javascript中的六种迭代器的总结，希望本文的内容对大家学习工作能有所帮助。
