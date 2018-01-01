[toc]
     
### 2.1作用域

#### 2.1.1全局变量和局部变量

``` javascript
 <script type="text/javascript">
      function add(num1,num2){
          //使用var定义的为局部变量
          var sum= num1+num2;
      }
      var result(1,2);
      //alert(sum);//出错
       //1.全局变量
      function add2(num1,num2){
          //使用var定义的为局部变量
           sum= num1+num2;
      }
      var result2(1,2);
      alert(sum);//3
 </script>
```

#### 2.1.2没有块级作用域

``` javascript

 <script type="text/javascript">
       if(true){
           var a="abc";
       }
       alert(a);//abc
       
       for(var i=0;i<10;i++){
           
       }
       alert(i);//10
 </script>
 
```
### 2.2引用类型

#### 2.2.1 Object类型

创建Object实例的方式有两种
``` javascript
 <script type="text/javascript">
        /**
         * 创建Object对象有两种方式
         */
        //方式1
       var person1=new Object();
       person1.name="张三";
       person1.age=29;
       //方式2
       var person2={
           name: "李四",
           age:18,
           //属性名也可以使用字符串
           "nickname":"小四"
       };
       var person3={};
       person3.name="王五";
       person3.age=18;
       //访问属性
       //1.点语法
       alert(person3.name);
       //2.用方括号进行访问
       /**
        * 两种方式没有区别，方括号可以通过变量进行访问
        */
       alert(person3["name"]);
       var name="name";
       alert(person3[name]);
 </script>
```
#### 2.2.2 Array类型

与其他语言不同的是，JavaScript数组的每一项都可以保存任何类型的数据。

##### 2.2.2.1 创建数组
创建数组的基本方式有两种:

* 使用Array构造函数 
  + 如果预先知道数组要保存的项目数量，也可以给该构造函数传递该数量
  + 也可以想Array构造函数传递数组中应该包含的项；如果传递的是数值，会按照给定的数值创建包含给定项数的数组；其他值则会创建包含那个值的只有一项的数组。
  + 用Array构造函数时也可以省略new操作符
  
``` javascript

 <script type="text/javascript">
     //使用构造函数创建数组
        var colors1=new Array();
        //如果预先知道数组要保存的项目数量，也可以给该构造函数传递该数量
        var colors2=new Array(10);
        //也可以想Array构造函数传递数组中应该包含的项 
        // 如果传递的是数值，会按照给定的该数值创建包含给定项数的数组；其他值则会创建
        //包含那个值的只有一项的数组。
        var colors3=new Array("blue","red","green");
        //使用Array构造函数时也可以省略new操作符
        var colors4=Array(3);
 </script>
```

* 数字字面量表示法。


``` javascript
 <script type="text/javascript">
   //创建一个包含3个字符串的数组
        var colors5=["red","green","blue"];
        //创建了一个空数组
        var colors6=[];
        //在IE8以及以前IE版本中，values是一个包含3个项目每一项的值分别为1、2和undefined的数组
        //在其他浏览器和IE9+，value会成为一个包含2项且值分别为1和2的数组。
        var values=[1,2,];//会创建一个包含2或3项的数组
        var values2=[,,,,,];//会创建一个包含5或6项的数组 每一项都为undefined
  </script>
        
```
##### 2.2.2.2 数组项和长度

``` javascript
 <script type="text/javascript">
        //创建一个包含3个字符串的数组
        var colors5=["red","green","blue"];
        color5[2]="white";//修改第三项
        colors5[20]="black";
        alert(colors5.length);//21 //数组自动增加到该索引值加1的长度
        //数组的length属性不是只读的。因此，通过设置这个属性，可以从数组的末尾移除项或者向
        //数组中添加新项
        colors5.length=2;
        alert(colors5[2]);//undefined
  </script>
       
```
##### 2.2.2.3 检测数组

``` javascript
 <script type="text/javascript">
      /**
         * 两种方式区别没有搞明白
         */
        var colors5=["red","green","blue"];
        //方式1
        if(colors5 instanceof Array){
            alert("是数组");
        }
        //方式2
        if(Array.isArray(colors5)){
             alert("是数组");
        }
       
  </script>
       
```
##### 2.2.2.4 转换方法

``` javascript
 <script type="text/javascript">
         var colors=["red","green","blue"];
        alert(colors.toString());//red,green,blue
        alert(colors.valueOf());//red,green,blue
        alert(colors);//red,green,blue 
        //默认用逗号分割可以，可以用join设置不同的分隔符
        alert(colors.join("-"));//red-green-blu
  </script>
       
```
##### 2.2.2.5 栈方法

JavaScript提供了push()和pop()方法让数组表现得像栈（先进先出）一样。

``` javascript
        <script type="text/javascript">
        var colors=["red","green","blue"];
        //push()方法可以接受任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度
        var count=colors.push("white","black");//5
        alert(count);
        //pop()方法则从数组末尾移除最后一项，减少数组的length值得，然后返回移除的项
        alert(colors.pop());//black
        alert(colors.length);//4
        </script>
```
##### 2.2.2.6 队列方法

结合使用shift()和push()方法，可以像使用队列（先进后出）一样使用数组。

``` javascript
        <script type="text/javascript">
        var colors=["red","green","blue"];
        //push()方法可以接受任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度
        var count=colors.push("white","black");//5
        alert(count);
        //
        alert(colors.shift());//red
        alert(colors.length);//4
        //unshift()方法可以从相反的方向来模拟队列，即在数组的前端添加项，从数组末端移除项
        colors.unshift("gray","yellow");
        alert(colors.pop());//black
        </script>
```
##### 2.2.2.7 重排序方法

数组中已经存在两个可以直接用reverse()和sort()方法。

``` javascript

        <script type="text/javascript">
        var colors=["red","green","blue"];
        colors.reverse();
        alert(colors);//blue,green,red
        //sort()方法按照升序排列数组项，sort()方法会调用每个数组项目的toString()转型方法，然后比较
        //得到的字符串，已确定如何排序。即使数组中的每一项都是数值，sort()方法比较的也是字符串
        var values=[0,1,5,10,15];
        values.sort();
        alert(values);//0,1,10,15,5
        //sort()方法可以接收一个比较函数作为参数，以便我们指定那个值位于那个值的前面
        
        //比较函数接收两个参数，如果第一个参数位于第二个之前则返回一个负数
        //如果两个参数相等则返回0，如果第一个参数应该位于第二个之后则返回一个正数。
        function compare(value1,value2){
            if(value1<value2){
                return 1;
            }else if(value1>value2){
                return -1;
            }else{
                return 0;
            }
        }
        values.sort(compare);
        alert(values);//15,10,5,1,0
        </script>
```

##### 2.2.2.8 操作方法

* concat()：这个方法会想创建当前数组的一个副本，然后将接受到的参数添加到这个副本的末尾，最后返回新构建的数组。
* slice()：可以接收一或两个参数，即要返回项的起始和结束位置。slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。slice()方法不会影响原始数组。

``` javascript
        <script type="text/javascript">
        var colors=["red","green","blue"];
        var colors2=colors.concat("yellow","black");
        alert(colors2);//red,green,blue,yellow,black
        var colors3=colors2.slice(1);
        alert(colors3);//green,blue,yellow,black
        var colors4=colors2.slice(1,2);
        alert(colors4);//green
        </script>
```        
* splice方法恐怕要算是最强大的数组方法了，它有很多种用法。splice()的主要用途是向数组的中不插入项，但是这种方法的方式则有如下3种。
  + 删除：可以删除任意数量的项，只需要指定2个参数：要删除第一项的位置和要删除的项数。
  + 删除并且插入
 
 ``` javascript
 
       <script type="text/javascript">
        var colors=["red","green","blue","yellow","black"];
        //删除 
        var remove=colors.splice(1,1);
        alert(colors.length);//4
        alert(remove);//green
        //先移除再插入
        remove=colors.splice(1,1,"green","white","orange");
        alert(colors);//red,green,white,orange,yellow,black
        alert(remove);//blue
        //移除一项并插入一项 实现替换
        remove=colors.splice(1,1,"pink");
        </script>
```        
##### 2.2.2.9 位置方法

* indexOf()： 开头开始查找
* lastIndexOf()：末尾开始查找，找不到返回-1.

 ``` javascript

        <script type="text/javascript">
        var colors=["red","green","blue","yellow","black"];
        alert(colors.indexOf("blue"));//2
        </script>
```               
##### 2.2.2.10 迭代方法
JavaScript为数组定义了5个迭代方法。

* every()：对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。
* some()：对数组中的每一项运行给定函数，如果该函数对任何一项返回true，则返回true。
* filter()：对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
* forEach()：对数组中的每一项目运行给定函数，这个方法没有返回值。
* map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。

 ``` javascript
 
        <script type="text/javascript">
        var numbers=[1,2,3,4,5,4,3,2,1];
        var everyResult=numbers.every(function(item,index,array){
            return (item>2);
        });
        alert(everyResult);//false
       var someResult=numbers.some(function(item,index,array){
           return (item>2)
       });
       alert(someResult);//true
       
       var filterResult=numbers.filter(function(item,index,array){
           return (item>2)
       });
       alert(filterResult);//3,4,5,4,3
       
       var mapResult=numbers.map(function(item,index,array){
           return item*2;
       });
       alert(mapResult);//2,4,6,8,10,8,6,4,2
       numbers.forEach(function(item,index,array){
          //操作
          alert(item);
       });
        </script>
```    
##### 2.2.2.11 缩小方法
JavaScript提供了两个缩小数组的方法：
* reduce()
* reduceRight()

这两个方法都接受两个参数：一个在每一项上调用的函数和作为缩小基础的初始值。传入的函数接收4个参数：前一个值、当前值、项的索引和数组对象。

 ``` javascript
        <script type="text/javascript">
       var numbers=[1,2,3,4,5];
       var sum=numbers.reduce(function(prev,cur,index,array){
           return prev+cur;
       });
       //第一次 perv是1 cur是2 第二次 prev是3（1加2的结果）cur 是3
       alert(sum);//15
       
        sum=numbers.reduceRight(function(prev,cur,index,array){
           return prev+cur;
       });
       //第一次 perv是5 cur是4第二次 prev是9（5加4的结果）cur是3
       alert(sum);//15
        </script>
       
``` 
#### 2.2.3 Date类型

##### 2.2.3.1 创建对象

 ``` javascript
 
        <script type="text/javascript">
        //创建时间对象
        var date=new Date();
        
        alert(date);//Thu Dec 26 2013 11:29:39 GMT+0800 (CST) 打印当前时间
        //如果想根据特定的日期和时间创建日期对象，必须传入表示该日期的毫秒数(1970
           // 年1月1日午夜起至该日期经过的毫秒数)。JavaScript为简化操作，提供了两个方法
        /**
         * 1.Date.parse()：这个方法因地区而已，将地区设置为美国浏览器通常接受下列日期格式
         *  年/日/月 如：6/13/2004
         *  英文月名 日,年 如：January 12,2004
         *  英文星期几 英文月名 日 年 时:分:秒 时区 如: Tue May 25 2004 00:00:00 GMT-0700
         *  ISO 8601扩展格式 YYYY-MM-DDTHH:mm:ss:sssZ  如：2004-05-25T00:00:00
         *
         * 如果不能表示日期 则返回NaN
         * 
         */
       date=new Date(Date.parse("12/21/2013"));
       alert(date);//Sat Dec 21 2013 00:00:00 GMT+0800 (CST)
       //实际上，如果直接将表示日期的字符串传递给Date构造函数，
       //也会在后台调用Date.parse()方法 所以下面的代码和上面的等价
       date=new Date("11/11/2013");
       alert(date);//Mon Nov 11 2013 00:00:00 GMT+0800 (CST)
       /**
        *  2.Date.UTC()
         * 与 Date.parse()在构建值时使用不同的信息。
         * Date.UTC()的参数分别是年份、月份(0~11)、日（1~31）小时（0～23）
         * 分钟、秒
         * 只有年和月是必须的，如果没有提供天数，则默认是1，如果省略其他参数，则统统是0
        */
       date=new Date(Date.UTC(2013,10,17,55,55));
       alert(date);//Tue Nov 19 2013 15:55:00 GMT+0800 (CST)
       //类似Date.parse 
       date=new Date(2013,8,17,55,55);
       alert(date);//Thu Sep 19 2013 07:55:00 GMT+0800 (CST)
       
       //Date.now()方法：获取当前毫秒数
       alert(Date.now());//1388033120172
        </script>
``` 
##### 2.2.3.2 常用方法

 ``` javascript
 
        <script type="text/javascript">
        //创建时间对象
        var date=new Date();
        alert("toLocaleString()--"+date.toLocaleString());//Thu Dec 26 12:58:33 2013
        alert("toString()--"+date.toString());//Thu Dec 26 2013 12:58:33 GMT+0800 (CST)
        alert("valueOf()--"+date.valueOf());//1388033913966
        alert("toDateString()--"+date.toDateString());//Thu Dec 26 2013        
        alert("toTimeString()--"+date.toTimeString());//12:58:33 GMT+0800 (CST)
        alert("toLocaleDateString()--"+date.toLocaleDateString());//12/26/2013
        alert("toLocaleTimeString()--"+date.toLocaleTimeString());//12:58:33
        alert("toUTCString()--"+date.toUTCString());//Thu, 26 Dec 2013 04:58:33 GMT
        </script>
``` 
#### 2.2.4 RegExp

JavaScript通过RegExp类型来支持正则表达式。

##### 2.2.4.1 创建对象

创建一个正则表达式有两种方式

* var expression= /pattern /flags;
  * pattern部分可以是任何简单或者复杂的正则表达式
  * flags 用以标明正则表达式的行为。正则表达式的匹配模式支持下列3个标志。
     + g：全局模式
     + i：不区分大小写
     + m：表示多行模式
* 通过构造函数

 ``` javascript
 
        <script type="text/javascript">
        //创建一个正则表达式
        var pattern= /[bc]at/i;
        alert(pattern.test("cat"));//true
        //使用构造函数创建一个正则表达
        var pattern2=new RegExp("[bc]at","i");
        alert(pattern2.test("Bat"));//true
        </script>
``` 
##### 2.2.4.2 实例属性

 ``` javascript
        <script type="text/javascript">
        //创建一个正则表达式
        var pattern= /[bc]at/i;
        alert(pattern.global);//false 表示是否设置了g标志
        alert(pattern.ignoreCase);//true 表示是否设置了i标志
        alert(pattern.mutiline);//false 表示是否设置了m标志
        alert(pattern.lastIndex);//0 整数，表示开始搜索下一个匹配项的自负位置，从0开始
        alert(pattern.source);///[bc]at/i 正则表达式的字符串表示
        </script>
``` 
##### 2.2.4.3 实例方法

 ``` javascript
        <script type="text/javascript">
        //创建一个正则表达式
        var text="mom and dad and baby";
        var pattern= /mom( and dad( and baby)?)?/gi;
        //exec（）专门为捕获数组而设计的。
        var matches=pattern.exec(text);
        alert(matches.index);//0
        alert(matches.input);//mom and dad and baby
        alert(matches[0]);//mom and dad and baby
        alert(matches[1]);// and dad and baby
        alert(matches[2]);// and baby
        //test()方法
        </script>
``` 
##### 2.2.4.4 静态属性

#### 2.2.5 Function类型

JavaScript中，每个函数都是Function类型的实例。由于函数是对象，因此函数名实际上是一个指向函数对象的指针，不会与某个函数绑定。

##### 2.2.5.1 创建函数

 ``` javascript

        <script type="text/javascript">
        //
        function sum1(num1,num2){
            return num1+num2;
        }
        alert(sum1(3,8));
        //使用函数表达式定义函数时，没有必要使用函数名，通过变量sum即可引用函数。
        var sum2=function(num1,num2){
            return num1+num2;
        };
     
        alert(sum2(4,9));
        //使用Function构造函数定义方法
        var sum3=new Function("num1","num2","return num1+num2");
        alert(sum3(8,7));//15
        </script>
``` 
##### 2.2.5.2 函数的声明和函数表达式

##### 2.2.5.3 作为值的函数

函数也可以作为值来使用，也就是说，不仅可以像传递参数一样把一个函数传递给另一个函数，而且可以将一个函数作为另一个函数的结果返回。

 ``` javascript
 
        <script type="text/javascript">
        function add10(num){
            return num+10;
        }
        function callSomeFunction(someFunction,someArgument){
            return someFunction(someArgument);
        }
        var result=callSomeFunction(add10,20);//30
        alert(result);
        </script>
        
``` 

##### 2.2.5.4 函数内部属性

在函数内部有两个特殊的对象

* arguments
  * arguments对象有一个名为callee的属性，该属性是一个指针，只想拥有这个arguments对象的函数。
  
 ``` javascript
 
        <script type="text/javascript">
			//计算阶乘
			function factorial1(num) {
				if (num <= 1) {
					return 1;
				} else {
					return num * factorial1(num - 1);
				}
			}

			/**
			 * 这个函数的执行和函数名factorial紧紧耦合在了一起，为了消除这种耦合现象可以
			 * 像下面这样使用arguments.callee
			 */
			function factorial2(num) {
				if (num <= 1) {
					return 1;
				} else {
					return num * arguments.callee(num - 1);
				}
			}

			//这样无论引用函数时使用的是什么名字，都可以保证正常完成递归调用

			var factorial3 = factorial2;
			//factorial3实际上是在另外一个位置上保存了一个函数的指针。
			//所以能得到阶乘 
			factorial2 = function() {
				return 0;
			};
			alert(factorial1(10));
			//3628800
			alert(factorial2(10));//0

			alert(factorial3(10));//3628800
        </script>
        
  ``` 
  
* this：this引用的是函据以执行的环境对象。

  
 ``` javascript
 
        <script type="text/javascript">
		//this关键字
			window.color = "red";
			var o = {
				color : "blue"
			};
			function sayColor() {
				alert(this.color);
			}

			sayColor();
			//red
			//对象定义方法
			o.sayColor = sayColor;
			o.sayColor();
			//blue
        </script>
        
  ``` 

* 对象的属性：caller，这个属性中保存着调用当前函数的函数的引用。如果在全局中调用函数，它的值为null；

 ``` javascript
 
        <script type="text/javascript">
		function outer() {
				inner();
			}

			function inner() {
				alert(inner.caller);
			}
			

			outer();//显示outer()函数的源代码
			inner();// ""
        </script>
        
  ``` 
  
##### 2.2.5.5 函数属性和方法
每个函数都包含两个静态属性：
* length：表示函数希望接受的命名参数的个数。
* prototype：

每个函数都包含两个非继承而来的方法，apply()和call()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值。
* apply():接受两个参数：一个是在其中运行的作用域，另外一个是参数数组。其中，第二个参数可以是Array的实例，也可以是arguments对象。
* call():与apply()方法的作用相同，区别仅在于接受参数的方式不同。
* bind():这个方法会创建一个函数的实例。


 ``` javascript
 
      <script type="text/javascript">
			function sayName(name) {

			}

			function sum(num1, num2) {
				return num1 + num2;
			}

			function sayHi() {
			}

			alert(sayName.length);
			//1
			alert(sum.length);
			//2
			alert(sayHi.length);
			//0
			//apply()方法
			function callSum1(num1, num2) {
				return sum.apply(this, arguments);
			}

			function callSum2(num1, num2) {
				return sum.apply(this, [num1, num2]);
			}

			//call()方法
			function callSum3(num1, num2) {
				return sum.call(this, num1, num2);
			}

			alert(callSum1(10, 10));
			//20
			alert(callSum2(10, 10));
			//20
			alert(callSum3(10, 10));
			//20

			//它们真正强大的地方是能够扩充函数赖以运行的作用域
			//对象不需要与方法有任何耦合关系
			window.color = "red";
			var o = {
				color : "blue"
			};
			function sayColor() {
				alert(this.color);
			}

			sayColor();//red
			sayColor.call(this);//red
			sayColor.call(window);//red
			sayColor.call(o);//blue
			
			//bind() 该方法会创建一个对象的实例
			var objbectSayColor=sayColor.bind(o);
			
			objbectSayColor();//blue
        </script>
        
  ``` 
#### 2.2.6 基本包装类型

为了便于操作基本类型值，JavaScript还提供了3个特殊的应用类型
* Boolean
* Number
* String

 ``` javascript
 
 <script type="text/javascript">
        //每当读取一个基本类型值的时候后台就会创建一个对应的基本包装类型对象，从而让我们能够调用
        //一些方法来操作这些数据
        var s1="some text";
        var s2=s1.substring(2);//me text
        alert(s2);
        //使用new 操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中
        //而自动创建的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。
        s1.color="red";
        alert(s1.color);//undefined
        //Object构造函数也会根据传入值的类型返回相应基本包装类型的实例。
        var obj=new Object("some text");
        alert(obj instanceof String);//true
        /**
         * 注意：使用new调用基本包装类型的构造函数，与直接调用同名的转型函数是不一样的。
         */
        var value="24";
        var number=Number(value);
        alert(typeof number);//number
        var obj2=new Number(value);//构造函数
        alert(typeof obj2);//object
        </script>
        
  ``` 
##### 2.2.6.1 Boolean


 ``` javascript
 
        <script type="text/javascript">
        //调用构造函数创建Boolean对象
        var falseObject=new Boolean(false);//
       alert(falseObject.valueOf());//false
       /**
        * 常见错误 布尔表达式中使用Boolean对象
        */
       //布尔表达式中的所有对象都会被转换为true
       var result1=booleanObject && true;
       alert(result1);//true
       var falseValue=false;
       result2=falseValue&&true;
       alert(result2);//false
       alert(typeof falseObject);//object
       alert(typeof falseValue);//boolean
       alert(falseObject instanceof Boolean);//true
       alert(falseValue instanceof Boolean);//false
        </script>
  ``` 
##### 2.2.6.2 Number

 ``` javascript

        <script type="text/javascript">
			//Number提供了一些用于将数值格式化为字符串的方法。

			//toFixed()方法会按照指定的小数位返回数值的字符串表示
			//参数：指定显示几位小数
			var num1 = 10;
			alert(num1.toFixed(3));
			//10.000
			//如果数值本身包含的小数位比指定的还多，那么接近指定的最大小数位
			//的值就会四舍五入
			var num2 = 10.003;
			alert(num2.toFixed(2));
			//10.00
			var num3 = 10.005;
			alert(num3.toFixed(2));
			//10.01

			//toExponential()该方法返回指数表示法
			//参数：指定输出结果中的小数位数
			var num4 = 10;
			alert(num4.toExponential(2));
			//1.00e+1

			//toPrecision()方法可能会返回固定大小格式，也可能返回指定指数格式；
			//具体规则看那种格式最合适。这个方法就接受一个参数：即表示数值的所有数字的位数。
		  var num5=99;
		  /**
		   * 结果是1e+2，因为一位数无法精确地表示99，因此四舍五入为100，这样就可以使用1位数来表示了
		   */
		  alert(num5.toPrecision(1));//1e+2 
		  
		  alert(num5.toPrecision(2));//99
		  alert(num5.toPrecision(3));//99.0
        </script>
  ``` 
##### 2.2.6.3 String类型

 ``` javascript
 
      <script type="text/javascript">
        //使用构造函数来创建
        var stringObject=new String("hello world");
        //length属性
        alert(stringObject.length);
        //常用方法
        
        /**
         * 字符方法
         * 1.charAt():返回字符
         * 2.charCodeAt():返回字符编码
         * 3.使用方括号加数字索引来访问字符串中的特定字符
         */
       alert(stringObject.charAt(1));// e
       alert(stringObject.charCodeAt(1));//101 也就是小写字母e的字符编码
       alert(stringObject[1]);//e
       
       /**
        * 字符串操作方法
        * 1.concat()：拼接字符串，可以接收任意多个参数
        * concat()可以用来拼接字符串，但是使用+拼接的字符串更方便
        * 3个创建新字符串的方法
        * 2.slice()
        * 3.substr()
        * 4.substring()
        */
       var result1=stringObject.concat(" hi!");//
       alert(result1);//hello world hi!
       alert(stringObject);//hello world 
       alert(stringObject.slice(3));//lo world 
        
       alert(stringObject.substring(3));//lo world 
       alert(stringObject.substr(3));//lo world
       //在传入负数情况下，他们的行为就不尽相同了不知道为什么报错
       alert(stringObject.slice(-3));//rld
       alert(stringObject.substring(-3));//hello world
       alert(stringObject.substr(-3));//rld
       
       /**
        * 位置方法
        * 
        * indexOf()
        * lastIndexOf()
        */
       alert(stringObject.indexOf("o"));//4
       alert(stringObject.lastIndexOf("o"));//7
       //从指定位置开始搜索
       alert(stringObject.indexOf("o",6));//7
       alert(stringObject.lastIndexOf("o",6));//4
       
       
       /**
        * trim()方法 去除空格
        */
      var stringValue1=" hello world ";
      alert(stringValue1.trim());
      
      /**
       * 字符串大小写转换
       */
      
      var stringValue1="hello world";
      //转换为大写
      alert(stringValue1.toLocaleUpperCase());//
     alert(stringValue1.toUpperCase());
     //转换为小写
      alert(stringValue1.toLowerCase());
      alert(stringValue1.toLocaleLowerCase());
      
      /**
       * 字符串的模式匹配方法
       */
      var text="cat,bat,sat,fat";
      var pattern=/.at/;
      var matches=text.match(pattern);
      alert(matches.index);//0
      alert(matches[0]);//cat
      alert(pattern.lastIndex);//0
      
      /**
       * localeCompare（）方法
       */
      
      /**
       * fromCharCode()方法
       */
      
      /**
       * HTML方法
       */
        </script>
   
  ``` 


[1]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#21%E4%BD%9C%E7%94%A8%E5%9F%9F
[2]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#211%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E5%92%8C%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F
[3]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#212%E6%B2%A1%E6%9C%89%E5%9D%97%E7%BA%A7%E4%BD%9C%E7%94%A8%E5%9F%9F
[4]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#22%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B
[5]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#221-object%E7%B1%BB%E5%9E%8B
[6]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#222-array%E7%B1%BB%E5%9E%8B
[7]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2221-%E5%88%9B%E5%BB%BA%E6%95%B0%E7%BB%84
[8]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2222-%E6%95%B0%E7%BB%84%E9%A1%B9%E5%92%8C%E9%95%BF%E5%BA%A6
[9]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2223-%E6%A3%80%E6%B5%8B%E6%95%B0%E7%BB%84
[10]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2224-%E8%BD%AC%E6%8D%A2%E6%96%B9%E6%B3%95
[11]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2225-%E6%A0%88%E6%96%B9%E6%B3%95
[12]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2226-%E9%98%9F%E5%88%97%E6%96%B9%E6%B3%95
[13]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2227-%E9%87%8D%E6%8E%92%E5%BA%8F%E6%96%B9%E6%B3%95
[14]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2228-%E6%93%8D%E4%BD%9C%E6%96%B9%E6%B3%95
[15]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2229-%E4%BD%8D%E7%BD%AE%E6%96%B9%E6%B3%95
[16]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#22210-%E8%BF%AD%E4%BB%A3%E6%96%B9%E6%B3%95
[17]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#22211-%E7%BC%A9%E5%B0%8F%E6%96%B9%E6%B3%95
[18]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#223-date%E7%B1%BB%E5%9E%8B
[19]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2231-%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1
[20]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2232-%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95
[21]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#224-regexp
[22]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2241-%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1
[23]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2242-%E5%AE%9E%E4%BE%8B%E5%B1%9E%E6%80%A7
[24]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2243-%E5%AE%9E%E4%BE%8B%E6%96%B9%E6%B3%95
[25]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2244-%E9%9D%99%E6%80%81%E5%B1%9E%E6%80%A7
[26]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#225-function%E7%B1%BB%E5%9E%8B
[27]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2251-%E5%88%9B%E5%BB%BA%E5%87%BD%E6%95%B0
[28]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2252-%E5%87%BD%E6%95%B0%E7%9A%84%E5%A3%B0%E6%98%8E%E5%92%8C%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F
[29]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2253-%E4%BD%9C%E4%B8%BA%E5%80%BC%E7%9A%84%E5%87%BD%E6%95%B0
[30]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2254-%E5%87%BD%E6%95%B0%E5%86%85%E9%83%A8%E5%B1%9E%E6%80%A7
[31]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2255-%E5%87%BD%E6%95%B0%E5%B1%9E%E6%80%A7%E5%92%8C%E6%96%B9%E6%B3%95
[32]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#226-%E5%9F%BA%E6%9C%AC%E5%8C%85%E8%A3%85%E7%B1%BB%E5%9E%8B
[33]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2261-boolean
[34]: https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2262-number
[35]:　https://github.com/malinkang/JavaScript/blob/master/docs/Chapter2.md#2263-string%E7%B1%BB%E5%9E%8B
