[toc]

## 介绍

使用过JavaScript框架（如AngularJS, Backbone 或者Ember）的人都很熟悉在UI（用户界面，前端）中mvc的工作机理。这些框架实现了MVC，使得在一个单页面中实现根据需要变化视图时更加轻松，而模型-视图-控制器（mvc）的核心概念就是：处理传入请求的控制器、显示信息的视图、表示业务规则和数据访问的模型。

因此，当需要创建这样一个需要在单个页面中实现切换出不同内容的应用时，我们通常选择使用上述框架之一。但是,如果我们仅仅需要一个在一个url中实现视图切换的框架,而不需要额外捆绑的功能的话，就不必使用象Angular和Ember等复杂的框架。本文就是尝试使用简单、有效方法来解决同样的问题。

## 这个概念

应用中的代码利用urls中的“#”实现MVC模式的导航。应用以一个缺省的url开始，基于哈希值的代码加载应用视图并且将对象-模型应用于视图模板。

URL的格式如下：
http：// 域名 /index.html#/ 路由名称

视图内容必须以{{Property-Name}}的方式绑定对象模型的值和属性。代码会查找这个专门的模板格式并且代替对象模型中的属性值。

以ajax的方式异步加载的视图会被放置于页面的占位符中。视图占位符可以是任何的元素（理想的情况是div），但是它必须有一个专门的属性，代码根据这个专门的属性来定位它，这样同样有助于代码的实现。当url改变时，会重复这个场景，另外一个视图被加载。听起来很简单吧！下面的流程图解释了在这个特定的实现中的消息跳转。


![image](http://olphpb1zg.bkt.clouddn.com/03071637_4voN.png)

## 写代码

我们以基本的模块设计模式开始，并且最终用门面设计模式的方式将我们的libs曝光于全局范围内。

```
; (function (w, d, undefined) { //剩下的代码 })(window, document);
```

我们需要将视图元素存储到一个变量中，这样就可以多次使用。

```
var _viewElement =  null ; //将用于渲染视图的元素
```

我们需要一个缺省的路由来应对url中没有路由信息的情况，这样就缺省的视图就可以被加载而不是展示空白页面。

```
var _defaultRoute =  null ;
```

现在我们来创建我们的主要MVC对象的构造方法。我们会把路由信息存储在“_routeMap”中

```
var jsMvc = function () {
    //mapping object for the routes
    this._routeMap = {};
}
```

是时候创建路由对象了，我们会将路由、模板、控制器的信息存储在这个对象中。

```
var routeObj = function (c, r, t) {
    this.controller = c;
    this.route = r;
    this.template = t;
}
```

每一个url会有一个专门的路由对象routeObj.所有的这些对象都会被添加到_routeMap对象中，这样我们后续就可以通过key-value的方式获取它们。

为了添加路由信息到MVC libs中，我们需要曝光libs中的一个方法。所以让我们创建一个方法，这个方法可以被各自的控制器用来添加新路由。

```
jsMvc.prototype.AddRoute = function (controller, route, template) {
    this._routeMap[route] = new routeObj(controller, route, template);
}
```

方法AddRoute接收3个参数：控制器，路由和模板(contoller, route and template)。他们分别是：

- controller:控制器的作用就是访问特定的路线。
- route:路由的路线。这个就是url中#后面的部分。
- template:这是外部的html文件，它作为这个路由的视图被加载。现在我们的libs需要一个切入点来解析url，并且为相关联的html模板页面提供服务。为了完成这个，我们需要一个方法。


现在，我们需要一个库的入口点来开始解析URL，并将相关联的html模板提供给页面。为了做到这一点，我们需要一个功能。

Initialize方法做如下的事情：

1. 获取视图相关的元素的初始化。代码需要一个具有view属性的元素，这样可以被用来在HTML页面中查找：
2. 设置缺省的路由
3. 验证视图元素是否合理
4. 绑定窗口哈希变更事件，当url不同哈希值发生变更时视图可以被及时更新
5. 最后，启动mvc

```
//初始化Mvc管理器对象以开始运行
jsMvc.prototype.Initialize = function () {
    var startMvcDelegate = startMvc.bind(this);

    //获取将用于渲染视图的html元素  
    _viewElement = d.querySelector(&apos;[view]&apos;);        
    if (!_viewElement) return; //如果没有找到view元素，则不做任何操作

    //设置默认路由 
    _defaultRoute = this._routeMap[Object.getOwnPropertyNames(this._routeMap)[0]];

    //启动Mvc经理
    w.onhashchange = startMvcDelegate;
    startMvcDelegate();
}
```

在上面的代码中，我们从startMvc方法中创建了一个代理方法startMvcDelegate。当哈希值变化时，这个代理都会被调用。下面就是当哈希值变化时我们做的操作的先后顺序：

1. 获取哈希值
2. 从哈希中获取路由值
3. 从路由map对象_routeMap中获取路由对象routeObj
4. 如果url中没有路由信息，需要获取缺省的路由对象
5. 最后，调用跟这个路由有关的控制器并且为这个视图元素的视图提供服务

上面的所有步骤都被下面的startMvc方法所实现

```
//函数启动mvc支持
function startMvc() {
    var pageHash = w.location.hash.replace(&apos;#&apos;, &apos;&apos;),
        routeName = null,
        routeObj = null;                
        
    routeName = pageHash.replace(&apos;/&apos;, &apos;&apos;); //从散列中获取路由的名称         
    routeObj = this._routeMap[routeName]; //获取路由对象

    //设置为默认路由对象，如果没有找到路由
    if (!routeObj)
        routeObj = _defaultRoute;
    
    loadTemplate(routeObj, _viewElement, pageHash); //抓取并设置路由的视图 
}
```

下一步，我们需要使用XML HTTP请求异步加载合适的视图。为此，我们会传递路由对象的值和视图元素给方法loadTemplate。

```
//加载外部html数据
function loadTemplate(routeObject, view) {
    var xmlhttp;
    if (window.XMLHttpRequest) {
         //代码为IE7 +，Firefox，Chrome，Opera，Safari 
        xmlhttp = new XMLHttpRequest();
    } else {
         // IE6代码，IE5 
        xmlhttp = new ActiveXObject(&apos;Microsoft.XMLHTTP&apos;);
    }
    xmlhttp.onreadystatechange = function () {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            loadView(routeObject, view, xmlhttp.responseText);
        }
    }
    xmlhttp.open(&apos;GET&apos;, routeObject.template, true);
    xmlhttp.send();
}
```

当前只剩加载视图和将对象模型与视图模板绑定了。我们会创建一个空的模型对象，然后传递与方法相关的模型来唤醒路由控制器。更新后的模型对象会与先前已经加载的XHR调用中的HTML模板绑定。

loadView方法被用于调用控制器方法，以及准备模型对象。

replaceToken方法被用于与HTML模板一起绑定模型

```
//使用模板加载视图的函数
function loadView(routeObject, viewElement, viewHtml) {
    var model = {}; 

    //从当前路线的控制器获取生成的模型
    routeObject.controller(model); 

    //将模型与视图绑定  
    viewHtml = replaceToken(viewHtml, model); 
    
    //将视图加载到视图元素中
    viewElement.innerHTML = viewHtml; 
}

function replaceToken(viewHtml, model) {
    var modelProps = Object.getOwnPropertyNames(model),
        
    modelProps.forEach(function (element, index, array) {
        viewHtml = viewHtml.replace(&apos;{{&apos; + element + &apos;}}&apos;, model[element]);
    });
    return viewHtml;
}
```

最后，我们将插件曝光于js全局范围外
```
//将mvc对象附加到窗口 
w[&apos;jsMvc&apos;] = new jsMvc();
```

现在，是时候在我们单页应用中使用这个MVC插件。在下一个代码段中，下面这些会实现:

1. 在web页面中引入这个代码
2. 用控制器添加路由信息和视图模板信息
3. 创建控制器功能
4. 最后，初始化lib。


除了上面我们需要的链接让我们导航到不同的路径外，一个容器元素的视图属性包含着视图模板html。

```
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript Mvc</title>
    <script src="jsMvc.js"></script>
    <!--[if lt IE 9]> <script src="jsMvc-ie8.js"></script> <![endif]-->
    
    <style type="text/css">
        .NavLinkContainer {
            padding: 5px;
            background-color: lightyellow;
        }

        .NavLink {
            background-color:black;
            color: white;
            font-weight:800;
            text-decoration:none;
            padding:5px;
            border-radius:4px;
        }
            .NavLink:hover {
                background-color:gray;
            }
    </style>
</head>
<body>
    <h3>Navigation Links</h3>
    <div class="NavLinkContainer">
        <a class="NavLink" href="index.html#/home">Home</a> 
   
        <a class="NavLink" href="index.html#/contact">Contact</a> 

        <a class="NavLink" href="index.html#/admin">Admin</a> 
       
    </div>
    <br />
    <br />
    <h3>View</h3>
    <div view></div>
    <script>
        jsMvc.AddRoute(HomeController, &apos;home&apos;, &apos;Views/home.html&apos;);
        jsMvc.AddRoute(ContactController, &apos;contact&apos;, &apos;Views/contact.html&apos;);
        jsMvc.AddRoute(AdminController, &apos;admin&apos;, &apos;Views/admin.html&apos;);
        jsMvc.Initialize();

        function HomeController(model) {
            model.Message = &apos;Hello World&apos;;
        }

        function ContactController(model) {
            model.FirstName = "John";
            model.LastName = "Doe";
            model.Phone = &apos;555-123456&apos;;
        }

        function AdminController(model) {
            model.UserName = "John";
            model.Password = "MyPassword";
        }
    </script>
</body>
</html>
```

上面的代码有一段包含一个为IE的条件注释。

```
<!--[if lt IE 9]> <script src="jsMvc-ie8.js"></script> <![endif]-->
```

如果IE的版本低于9，那么function.bind，Object.getOwnPropertyNames和Array.forEach属性将不会被支持。因此我们要通过判断浏览器是否低于IE9来反馈代码是否支持。

其中的内容有home.html， contact.html 和 admin.html 请看下面：

home.html：

```
{{Message}}
```

![image](http://olphpb1zg.bkt.clouddn.com/03071638_FetI111111111.png)

contact.html：

```
{{FirstName}} {{LastName}}
<br />
{{Phone}}
```

![image](http://olphpb1zg.bkt.clouddn.com/03071638_0Jlw2222222222222.png)

admin.html：

```
<div style="padding:2px;margin:2px;text-align:left;">
    <label for="txtUserName">User Name</label>
    <input type="text" id="txtUserName" value="{{UserName}}" />
</div>
<div style="padding:2px;margin:2px;text-align:left;">
    <label for="txtPassword">Password</label>
    <input type="password" id="txtPassword" value="{{Password}}" />
</div>
```

![image](http://olphpb1zg.bkt.clouddn.com/03071638_1M6H333333.png)