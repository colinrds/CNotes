观察者是一种设计模式，其中对象（称为主题）根据（观察者）维护对象列表，自动通知他们对状态的任何更改。

当一个主体需要通知观察者有趣的事情时，它向观察者广播一个通知（可以包括与通知的主题有关的具体数据）。

当我们不再希望向特定观察员通知他们注册的主题的更改时，主题可以将其从观察者列表中删除。

参考已发表的设计模式的定义，通常是有用的，这些定义是语言无关的，以便随着时间的推移更广泛地了解其使用和优势。GoF书“ 设计模式：可重用面向对象软件的要素”中提供的观察者模式的定义是：

> 一个或多个观察者对一个主体的状态感兴趣，并通过附加自己来注册他们对该主题的兴趣，当观察者可能感兴趣的主题发生变化时，发送一个通知消息，其中每个调用更新方法观察者当观察者对主体的状态不再感兴趣时，可以简单地分离自己。

现在我们可以利用以下组件来扩展我们所学到的来实现Observer模式：

- 主题：维护观察员名单，便利添加或删除观察员
- 观察者：为需要通知主体状态更改的对象提供更新界面
- ConcreteSubject：向观察者广播关于状态变化的通知，存储ConcreteObservers的状态
- ConcreteObserver：存储对ConcreteSubject的引用，为Observer实现一个更新界面，以确保状态与Subject

首先，我们来模拟一个主题可能拥有的依赖观察者列表：

```
function ObserverList() {
    this.observerList = [];
}

ObserverList.prototype.add = function (obj) {
    return this.observerList.push(obj);
};

ObserverList.prototype.count = function () {
    return this.observerList.length;
};

ObserverList.prototype.get = function (index) {
    if (index > -1 && index < this.observerList.length) {
        return this.observerList[index];
    }
};

ObserverList.prototype.indexOf = function (obj, startIndex) {
    var i = startIndex;

    while (i < this.observerList.length) {
        if (this.observerList[i] === obj) {
            return i;
        }
        i++;
    }

    return -1;
};

ObserverList.prototype.removeAt = function (index) {
    this.observerList.splice(index, 1);
};
```

接下来，我们来模拟主题，并添加，删除或通知观察员列表上的观察者的能力。

```
function Subject() {
    this.observers = new ObserverList();
}

Subject.prototype.addObserver = function (observer) {
    this.observers.add(observer);
};

Subject.prototype.removeObserver = function (observer) {
    this.observers.removeAt(this.observers.indexOf(observer, 0));
};

Subject.prototype.notify = function (context) {
    var observerCount = this.observers.count();
    for (var i = 0; i < observerCount; i++) {
        this.observers.get(i).update(context);
    }
};
```

然后，我们定义一个用于创建新观察者的骨架。update这里的功能将会随着自定义行为而被覆盖。

```
// The Observer
function Observer() {
    this.update = function () {
        // ...
    };
}
```

在我们使用上述Observer组件的示例应用程序中，我们现在定义：

- 用于向页面添加新的可观察复选框的按钮
- 一个控制复选框将作为一个主题，通知其他复选框，他们应该被检查
- 要添加新复选框的容器

然后，我们定义ConcreteSubject和ConcreteObserver处理程序，用于向页面添加新的观察者并实现更新界面。有关这些组件在我们的示例的上下文中做的内联评论，请参见下文。

HTML：

```
<button id="addNewObserver">Add New Observer checkbox</button>
<input id="mainCheckbox" type="checkbox"/>
<div id="observersContainer"></div>
```

示例脚本：

```
// Extend an object with an extension
function extend(obj, extension) {
    for (var key in extension) {
        obj[key] = extension[key];
    }
}

// References to our DOM elements

var controlCheckbox = document.getElementById("mainCheckbox"),
    addBtn = document.getElementById("addNewObserver"),
    container = document.getElementById("observersContainer");


// Concrete Subject

// Extend the controlling checkbox with the Subject class
extend(controlCheckbox, new Subject());

// Clicking the checkbox will trigger notifications to its observers
controlCheckbox.onclick = function () {
    controlCheckbox.notify(controlCheckbox.checked);
};

addBtn.onclick = addNewObserver;

// Concrete Observer

function addNewObserver() {

    // Create a new checkbox to be added
    var check = document.createElement("input");
    check.type = "checkbox";

    // Extend the checkbox with the Observer class
    extend(check, new Observer());

    // Override with custom update behaviour
    check.update = function (value) {
        this.checked = value;
    };

    // Add the new observer to our list of observers
    // for our main subject
    controlCheckbox.addObserver(check);

    // Append the item to the container
    container.appendChild(check);
}
```

在这个例子中，我们研究了如何实现和使用Observer模式，涵盖了Subject，Observer，ConcreteSubject和ConcreteObserver的概念。

**观察者与发布/订阅模式之间的差异**

虽然Observer模式在JavaScript世界中很常见，但我们会发现它通常使用被称为Publish / Subscribe模式的变体来实现。虽然非常相似，但这些模式之间存在差异，值得注意。

观察者模式要求希望接收主题通知的观察者（或对象）必须向对象触发事件（主题）订阅此兴趣。

然而，“发布/订阅”模式使用位于希望接收通知（订阅者）的对象和触发事件的对象（发布者）之间的主题/事件通道。该事件系统允许代码定义应用程序特定的事件，该事件可以传递包含订户所需的值的自定义参数。这里的想法是避免用户和发布者之间的依赖关系。

这与观察者模式不同，因为它允许任何用户实现适当的事件处理程序来注册和接收发布者广播的主题通知。

以下是一个示例，说明如果提供功能实现和幕后功能publish()，可以使用“发布/订阅”subscribe()unsubscribe()

```
// A very simple new mail handler

// A count of the number of messages received
var mailCounter = 0;

// Initialize subscribers that will listen out for a topic
// with the name "inbox/newMessage".

// Render a preview of new messages
var subscriber1 = subscribe("inbox/newMessage", function (topic, data) {

    // Log the topic for debugging purposes
    console.log("A new message was received: ", topic);

    // Use the data that was passed from our subject
    // to display a message preview to the user
    $(".messageSender").html(data.sender);
    $(".messagePreview").html(data.body);

});

// Here's another subscriber using the same data to perform
// a different task.

// Update the counter displaying the number of new
// messages received via the publisher

var subscriber2 = subscribe("inbox/newMessage", function (topic, data) {

    $('.newMessageCounter').html(++mailCounter);

});

publish("inbox/newMessage", [{
    sender: "hello@google.com",
    body: "Hey there! How are you doing today?"
}]);

// We could then at a later point unsubscribe our subscribers
// from receiving any new topic notifications as follows:
// unsubscribe( subscriber1 );
// unsubscribe( subscriber2 );
```

这里的一般想法是促进松耦合。而不是单个对象直接调用其他对象的方法，而是订阅另一个对象的特定任务或活动，并在发生时通知它们。

- 优点

观察者和发布/订阅模式鼓励我们对我们应用程序的不同部分之间的关​​系进行思考。他们还帮助我们确定包含直接关系的层次，而不是用一组主题和观察者来代替。这有效地用于将应用程序分解成更小，更松散的块，以改进代码管理和重新使用的潜力。

使用Observer模式背后的进一步动机是我们需要保持相关对象之间的一致性，而不会使类紧密耦合。例如，当对象需要能够通知其他对象而不对这些对象做出假设时。

使用任一模式时，观察者和受试者之间可能存在动态关系。这提供了大量的灵活性，当我们的应用程序的不同部分紧密耦合时，可能并不容易实现。

尽管它可能并不总是每个问题的最佳解决方案，但这些模式仍然是设计解耦系统的最佳工具之一，应被视为任何JavaScript开发人员实用带中的重要工具。

- 缺点

因此，这些模式的一些问题实际上源于它们的主要优点。在发布/订阅中，通过将发布者与订阅者脱钩，有时可能难以获得我们应用程序的特定部分正常运行的保证。

例如，发布商可以假定一个或多个订阅者正在收听。假设我们使用这样的假设来记录或输出有关某些应用程序过程的错误。如果执行日志的用户崩溃（或由于某种原因无法运行），则由于系统的脱钩性质，发布商将无法看到这种情况。

这种模式的另一个缺点是用户对彼此的存在相当无知，并且忽视了转换发布商的成本。由于订阅者和发布者之间的动态关系，更新依赖性可能难以追踪。


**发布/订阅实施**

发布/订阅在JavaScript生态系统中非常适用，主要是因为在核心上，ECMAScript实现是事件驱动的。在浏览器环境中尤其如此，因为DOM使用事件作为脚本的主要交互API。

也就是说，ECMAScript和DOM都不提供用于在实现代码中创建自定义事件系统的核心对象或方法（除了可能与DOM绑定的DOM3 CustomEvent，因此不是一般有用的）。

幸运的是，流行的JavaScript库（如dojo，jQuery（自定义事件））和YUI已经具有可以帮助轻松实现“发布/订阅”系统的实用程序。下面我们可以看到一些例子：

```
// Publish
 
// jQuery: $(obj).trigger("channel", [arg1, arg2, arg3]);
$( el ).trigger( "/login", [{username:"test", userData:"test"}] );
 
// Dojo: dojo.publish("channel", [arg1, arg2, arg3] );
dojo.publish( "/login", [{username:"test", userData:"test"}] );
 
// YUI: el.publish("channel", [arg1, arg2, arg3]);
el.publish( "/login", {username:"test", userData:"test"} );
 
 
// Subscribe
 
// jQuery: $(obj).on( "channel", [data], fn );
$( el ).on( "/login", function( event ){...} );
 
// Dojo: dojo.subscribe( "channel", fn);
var handle = dojo.subscribe( "/login", function(data){..} );
 
// YUI: el.on("channel", handler);
el.on( "/login", function( data ){...} );
 
 
// Unsubscribe
 
// jQuery: $(obj).off( "channel" );
$( el ).off( "/login" );
 
// Dojo: dojo.unsubscribe( handle );
dojo.unsubscribe( handle );
 
// YUI: el.detach("channel");
el.detach( "/login" );
```

对于那些希望使用香草JavaScript（或另一个库）发布/订阅模式的人，AmplifyJS包含一个可以与任何库或工具包一起使用的干净的，与库无关的实现。


特别是jQuery开发人员还有很多其他的选择，可以选择使用Peter Higgins的jQuery插件和Ben Alman（优化）Pub / Sub jQuery gist在GitHub上的众多开发完善的实现之一。只能链接到其中的几个，可以在下面找到。

- [Ben Alman的发布/订阅 gist(推荐)](https://gist.github.com/661855)
- [Rick Waldron 在上面基础上修改的 jQuery-core 风格的实现](https://gist.github.com/705311)
- [Peter Higgins 的插件](http://github.com/phiggins42/bloody-jquery-plugins/blob/master/pubsub.js)
- [AppendTo 在AmplifyJS中的 发布/订阅实现](http://amplifyjs.com/)
- [Ben Truyman的 gist](https://gist.github.com/826794)

所以我们可以感受到Observer模式的多少个vanilla JavaScript实现可能起作用，我们来看看在GitHub上发布的一个名为pubsubz的项目的发布/订阅的简约版本。这表明了订阅，发布以及取消订阅的概念的核心概念。

我已经选择基于这个代码，因为它密切关注方法签名和实现方法，我希望在JavaScript版本的经典Observer模式中看到。


**发布/订阅实施**

```
var pubsub = {};

(function (myObject) {

    // Storage for topics that can be broadcast
    // or listened to
    var topics = {};

    // A topic identifier
    var subUid = -1;

    // Publish or broadcast events of interest
    // with a specific topic name and arguments
    // such as the data to pass along
    myObject.publish = function (topic, args) {

        if (!topics[topic]) {
            return false;
        }

        var subscribers = topics[topic],
            len = subscribers ? subscribers.length : 0;

        while (len--) {
            subscribers[len].func(topic, args);
        }

        return this;
    };

    // Subscribe to events of interest
    // with a specific topic name and a
    // callback function, to be executed
    // when the topic/event is observed
    myObject.subscribe = function (topic, func) {

        if (!topics[topic]) {
            topics[topic] = [];
        }

        var token = (++subUid).toString();
        topics[topic].push({
            token: token,
            func: func
        });
        return token;
    };

    // Unsubscribe from a specific
    // topic, based on a tokenized reference
    // to the subscription
    myObject.unsubscribe = function (token) {
        for (var m in topics) {
            if (topics[m]) {
                for (var i = 0, j = topics[m].length; i < j; i++) {
                    if (topics[m][i].token === token) {
                        topics[m].splice(i, 1);
                        return token;
                    }
                }
            }
        }
        return this;
    };
}(pubsub));
```

示例：使用我们的实现

我们现在可以使用实现来发布和订阅感兴趣的事件如下：

```
// Another simple message handler

// A simple message logger that logs any topics and data received through our
// subscriber
var messageLogger = function (topics, data) {
    console.log("Logging: " + topics + ": " + data);
};

// Subscribers listen for topics they have subscribed to and
// invoke a callback function (e.g messageLogger) once a new
// notification is broadcast on that topic
var subscription = pubsub.subscribe("inbox/newMessage", messageLogger);

// Publishers are in charge of publishing topics or notifications of
// interest to the application. e.g:

pubsub.publish("inbox/newMessage", "hello world!");

// or
pubsub.publish("inbox/newMessage", ["test", "a", "b", "c"]);

// or
pubsub.publish("inbox/newMessage", {
    sender: "hello@google.com",
    body: "Hey again!"
});

// We can also unsubscribe if we no longer wish for our subscribers
// to be notified
pubsub.unsubscribe(subscription);

// Once unsubscribed, this for example won't result in our
// messageLogger being executed as the subscriber is
// no longer listening
pubsub.publish("inbox/newMessage", "Hello! are you still there?");
```

**示例：用户界面通知**

接下来，我们假设我们有一个负责显示实时股票信息的Web应用程序。

应用程序可能具有用于显示库存统计信息的网格和用于显示最后更新点的计数器。当数据模型发生变化时，应用程序将需要更新网格和计数器。在这种情况下，我们的主题（将发布主题/通知）是数据模型，我们的用户是网格和计数器。

当我们的用户收到模型本身已经改变的通知时，他们可以相应地更新自己。

在我们的实现中，我们的用户将会听取“newDataAvailable”主题，以了解是否有新的股票信息可用。如果对此主题发布了新的通知，则会触发gridUpdate将新行添加到包含此信息的网格中。它还将更新上一次更新的计数器，以记录上次添加数据的时间

```
// Return the current local time to be used in our UI later
getCurrentTime = function () {

    var date = new Date(),
        m = date.getMonth() + 1,
        d = date.getDate(),
        y = date.getFullYear(),
        t = date.toLocaleTimeString().toLowerCase();

    return (m + "/" + d + "/" + y + " " + t);
};

// Add a new row of data to our fictional grid component
function addGridRow(data) {

    // ui.grid.addRow( data );
    console.log("updated grid component with:" + data);

}

// Update our fictional grid to show the time it was last
// updated
function updateCounter(data) {

    // ui.grid.updateLastChanged( getCurrentTime() );
    console.log("data last updated at: " + getCurrentTime() + " with " + data);

}

// Update the grid using the data passed to our subscribers
gridUpdate = function (topic, data) {

    if (data !== undefined) {
        addGridRow(data);
        updateCounter(data);
    }

};

// Create a subscription to the newDataAvailable topic
var subscriber = pubsub.subscribe("newDataAvailable", gridUpdate);

// The following represents updates to our data layer. This could be
// powered by ajax requests which broadcast that new data is available
// to the rest of the application.

// Publish changes to the gridUpdated topic representing new entries
pubsub.publish("newDataAvailable", {
    summary: "Apple made $5 billion",
    identifier: "APPL",
    stockPrice: 570.91
});

pubsub.publish("newDataAvailable", {
    summary: "Microsoft made $20 million",
    identifier: "MSFT",
    stockPrice: 30.85
});
```

**示例：使用Ben Alman的Pub / Sub实现解耦应用程序**

在以下电影评分示例中，我们将使用Ben Alman的“发布/订阅”的jQuery实现来演示我们如何解耦用户界面。请注意，如何提交评分仅具有发布新用户和评级数据可用的事实的效果。

这些主题的订阅者将被放置，然后委托该数据发生什么。在我们的例子中，我们将新数据推送到现有的数组中，然后使用Underscore库的.template()模板方法渲染它们。

HTML /模板

```
<script id="userTemplate" type="text/html">
    <li><%= name %></li>
</script>


<script id="ratingsTemplate" type="text/html">
    <li><strong><%= title %></strong> was rated <%= rating %>/5</li>
</script>


<div id="container">

    <div class="sampleForm">
        <p>
            <label for="twitter_handle">Twitter handle:</label>
            <input type="text" id="twitter_handle" />
        </p>
        <p>
            <label for="movie_seen">Name a movie you've seen this year:</label>
            <input type="text" id="movie_seen" />
        </p>
        <p>

            <label for="movie_rating">Rate the movie you saw:</label>
            <select id="movie_rating">
                <option value="1">1</option>
                <option value="2">2</option>
                <option value="3">3</option>
                <option value="4">4</option>
                <option value="5" selected>5</option>

            </select>
        </p>
        <p>

            <button id="add">Submit rating</button>
        </p>
    </div>



    <div class="summaryTable">
        <div id="users"><h3>Recent users</h3></div>
        <div id="ratings"><h3>Recent movies rated</h3></div>
    </div>

</div>
```

JavaScript的

```
; (function ($) {

    // Pre-compile templates and "cache" them using closure
    var userTemplate = _.template($("#userTemplate").html()),
        ratingsTemplate = _.template($("#ratingsTemplate").html());

    // Subscribe to the new user topic, which adds a user
    // to a list of users who have submitted reviews
    $.subscribe("/new/user", function (e, data) {

        if (data) {

            $('#users').append(userTemplate(data));

        }

    });

    // Subscribe to the new rating topic. This is composed of a title and
    // rating. New ratings are appended to a running list of added user
    // ratings.
    $.subscribe("/new/rating", function (e, data) {

        if (data) {

            $("#ratings").append(ratingsTemplate(data));

        }

    });

    // Handler for adding a new user
    $("#add").on("click", function (e) {

        e.preventDefault();

        var strUser = $("#twitter_handle").val(),
            strMovie = $("#movie_seen").val(),
            strRating = $("#movie_rating").val();

        // Inform the application a new user is available
        $.publish("/new/user", { name: strUser });

        // Inform the app a new rating is available
        $.publish("/new/rating", { title: strMovie, rating: strRating });

    });

})(jQuery);
```

**示例：去除基于Ajax的jQuery应用程序**

在我们的最后一个例子中，我们将进一步研究如何在开发过程中早期使用Pub / Sub脱钩我们的代码可以在稍后的时间里为我们节省一些潜在的痛苦的重构。

在Ajax很重的应用程序中，经常遇到一个请求的响应，我们希望实现的不仅仅是一个独特的操作。可以简单地将所有后请求逻辑添加到成功回调中，但是这种方法存在缺点。

由于增加的功能间/代码依赖性，高耦合应用程序有时会增加重用功能所需的工作量。这意味着，尽管如果我们只是尝试抓取一个结果集一次，保持我们在回调中硬编码的后请求逻辑可能会很好，当我们想对同一数据源进行进一步的Ajax调用时，这并不合适（和不同的结束行为），而不是重写部分代码多次。我们可以从开始使用pub / sub并节省时间，而不必回头调用相同数据源的每个层，并在之后对其进行泛化。

使用观察者，我们还可以轻松地将关于不同事件的应用程序范围的通知分解为我们所熟悉的任何粒度级别 - 可以使用其他模式进行优化。

请注意，在下面的示例中，当用户指示要进行搜索查询时发出一个主题通知，并且在请求返回并且实际数据可用于消费时进行另一个主题通知。然后由用户决定如何使用这些事件的知识（或返回的数据）。这样做的好处是，如果我们想要，我们可以有10个不同的用户利用不同方式返回的数据，但就Ajax层而言，它不在乎。其唯一的职责是要求并返回数据，然后将其传递给想要使用它的人。这种关切的分离可以使我们的代码的整体设计更清洁一些。

HTML /模板：

```
<form id="flickrSearch">
    <input type="text" name="tag" id="query" />
    <input type="submit" name="submit" value="submit" />
</form>

<div id="lastQuery"></div>
<ol id="searchResults"></ol>



<script id="resultTemplate" type="text/html">
    <% _.each(items, function( item ){ %>
        <li><img src="<%= item.media.m %>" /></li>
    <% });%>
</script>
```

JavaScript：

```
;(function ($) {

    // Pre-compile template and "cache" it using closure
    var resultTemplate = _.template($("#resultTemplate").html());

    // Subscribe to the new search tags topic
    $.subscribe("/search/tags", function (e, tags) {
        $("#lastQuery").html("<p>Searched for:<strong>" + tags + "</strong></p>");
    });

    // Subscribe to the new results topic
    $.subscribe("/search/resultSet", function (e, results) {

        $("#searchResults").empty().append(resultTemplate(results));

    });

    // Submit a search query and publish tags on the /search/tags topic
    $("#flickrSearch").submit(function (e) {

        e.preventDefault();
        var tags = $(this).find("#query").val();

        if (!tags) {
            return;
        }

        $.publish("/search/tags", [$.trim(tags)]);

    });


    // Subscribe to new tags being published and perform
    // a search query using them. Once data has returned
    // publish this data for the rest of the application
    // to consume

    $.subscribe("/search/tags", function (e, tags) {

        $.getJSON("http://api.flickr.com/services/feeds/photos_public.gne?jsoncallback=?", {
            tags: tags,
            tagmode: "any",
            format: "json"
        };

        function (data) {

            if (!data.items.length) {
                return;
            }

            $.publish("/search/resultSet", { items: data.items });
        });

    });


})(jQuery);
```

观察者模式对于在应用程序设计中解耦多个不同的场景是有用的，如果您没有使用它，我建议您选择今天提到的一个预先编写的实现，并尝试一下。它是开始使用的更简单的设计模式之一，也是最强大的设计模式之一。