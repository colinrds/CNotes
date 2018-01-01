[toc]

## 简介

写出可维护的代码的一个最重要的方面就是在代码中能够注意到重复出现的主题并对其进行优化。设计模式的知识领域是无价的。

在本书的第一部分，我们将探索那些真正可以应用于任何编程语言的设计模式的历史和重要性。如果你已经熟悉这段历史，可以直接跳过"什么是模式？"这一章继续阅读。

设计模式可以追溯到早期的一名叫Christopher Alexander的建筑师。他经常会发表一些他在处理设计问题时的经验和如何与建筑和城镇相联系的。有一天，当Alexander使用了一次又一次后，他发现某些设计结构会导致做出的效果是最好的。

在Sara Ishikawa和Murray Silverstein的协作下，Alexander发明了一种可以帮助授权任何人去设计和构建希望的任何规模的模式语言。这在1977年的一篇名为"A Pattern Language"的论文中发表，在后来作为一本完整的精装书发表。

大约30年前，软件工程师开始将Alexander曾写过的原理并入第一版的设计模式，这是一个用来对那些想要改善他们编码技巧的新手开发者的一个指南。要注意，这时设计模式背后的概念实际上已经在编程行业成立以来就有了，虽然不是那么正式的形式。

第一个也是最标志性的关于软件工程的设计模式的正式作品是在1995年一本叫Design Patterns: Elements Of Reusable Object-Oriented Software的书中发表，这是Erich Gamma, Richard Helm, Ralph Johnson和 John Vlissides - 一群被称为Gang of Four(简称GoF)的人写的。

GoF的出版物被认为是非常有助于推动设计模式的概念在我们的领域发展的，因为它描述了大量的开发技术和缺陷，而且还有在今天的世界中大量使用的23个核心的面向对象的设计模式。我们将详细地在"设计模式分类"这一章中介绍这些模式。

在本书中，我们将看到一些流行的JavaScript设计模式，并探索为什么一些特定的模式比其他的更适合你的项目。但请记住模式不仅仅可以应用在单纯的JavaScript (例如：标准JavaScript代码)里，也可以在一些像jQuery或dojo的抽象库里使用。在我们开始之前，让我们看看模式在软件设计中的确切定义。

## 什么是设计模式

一个模式就是一个可重用的方案，可应用于在软件设计中的常见问题 - 在我们的例子里 - 就是编写JavaScript的web应用程序。模式的另一种解释就是一个我们如何解决问题的模板 - 那些可以在许多不同的情况里使用的模板。 那么理解和熟悉模式为什么是如此的重要？设计模式有以下三点好处：

- 模式是行之有效的解决方法：他们提供固定的解决方法来解决在软件开发中出现的问题，这些都是久经考验的反应了开发者的经验和见解的使用模式来定义的技术。
- 模式可以很容易地重用：一个模式通常反映了一个可以适应自己需要的开箱即用的解决方案。这个特性让它们很健壮。
- 模式善于表达：当我们看到一个提供某种解决方案的模式时，一般有一组结构和词汇可以非常优雅地帮助表达相当大的解决方案。

模式不是一个确切的解决方案。我们要记住模式的角色仅仅是给我们提供一个解决方案。模式不能解决所有的设计问题，也不能代替优秀的软件设计师。然而，它们在帮助我们。接下来我们将看看模式必须提供的其他的一些优势。

- 模式的重用可以帮助防止在应用程序开发过程中出现的一些可能导致重大问题的小问题。这意味着当代码是建立在行之有效的模式上时，我们可以花更少的时间去关心我们的代码结构，从而能花更多的时间关注我们的解决方案的整体质量。这是因为模式可以鼓励我们在更好的结构化和有组织的方式下编码，这将避免在未来由于清洁的目的而去重构它。
- 模式可以提供一个不需要绑定到一个特定问题的书面的概括性的解决方案。这个广义的方法意味着不用管我们正在处理的应用程序 (许多情况下的编程语言) 设计模式的应用可以提高我们的代码的结构。
- 某些模式可以通过避免重复来减小我们代码的文件大小。通过鼓励开发者更仔细地看待他们的解决方案来减少重复的地方，如通过将类似的执行流程作为一个一般性的函数来减少函数的数量，这样我们就可以减小代码库的总体大小，这也成为使代码更DRY。
- 模式增加了开发者的词汇，这使得交流更快速。
- 经常使用的模式可通过收集其他使用这些模式的开发人员贡献给设计模式社区的经验来改进。在某些情况下，这将导致全新模式的创建，同时也可以提供改进的指导大家如何使用特定的模式才是最好的。这可以确保基于模式的解决方案继续变得比特别的解决方案更健壮。

**我们已经每天都在使用模式**

为了了解模式有多有用，让我们看看jQuery提供给我们的一个很简单的元素选择问题。 假设我们有一个为页面上每一个class为"foo"的DOM元素添加一个计数器的脚本，什么才是查询这个元素的集合的最有效的方法呢？有几种不同的方法可以解决这个问题：

选择页面上所有的元素并存储它们的引用，然后使用正则表达式 (或其他方式) 来过滤这个集合中那些class为"foo"的元素的引用。

- 使用像asquerySelectorAll()的现代原生浏览器的特性，来选择所有的class为"foo"的元素。
- 使用像asgetElementsByClassName()的原生特性同样可以获取期望的集合。

那些，这些选择哪个是最快的呢？实际上第三个，比其他的替代选择快 8-10倍。但在实际的应用程序中，第三个选择无法在Internet Explorer 9以下的版本中使用，从而只能使用第一个，第二个和第三个都不支持。

使用jQuery的开发人员就不必担心这个问题，因为很幸运的是它使用Facade模式把这个问题抽象了出来。正如我们即将在后面更详细的介绍的那样，这种模式提供了一组简单的对更复杂的底层代码的抽象接口 (例如el.css(),el.animate()) 。正如我们所看到的，这意味着我们只会对实现级别的细节花费更少的时间。

在其后，库会根据我们当前浏览器的支持自动选择最优的方法来选择元素，我们只使用抽象层。

我们可能都熟悉jQuery的$("selector")，这是更容易使用的在一个页面选择HTML元素的方法，这样我们就不必手动来选择getElementById(),getElementsByClassName(),getElementByTagName()等方法。

虽然我们知道querySelectorAll()试图解决这个问题，但比比使用jQuery的Facade接口和自己来选择最优的方式时花费的精力，毫无疑问，使用模式可以提供真实世界的抽象价值。

我们将在本书的后面看到更多的设计模式。

**“模式特性”测试，模式原型和三条规则**

记住并不是每个算法、每个最佳实践和每个解决方案都可能被认为是一个完整的模式。这儿可能缺少了几个关键因素，而且模式社团除非经过严格的审查才谨慎地声明某东西为模式的。即使某东西对我们来说似乎满足了模式标准，它都不应该被当作模式，直到它由他人经过适当时间的周密调查和测试后才可能当作模式。

回头看看Alexander曾经做过的工作，他声明模式应当既是过程也是“事物”。这个定义故意不明确，因为他紧跟着说模式应该是创建“事物”的过程。这就是为什么模式通常集中定位在表面上可识别的结构的原因。例如，我们应当能够可视化地描绘（或者绘制）图片来展示把模式应用到实践中的结构。

在研究设计模式的时候，无意间碰到术语“模式原型”是很正常的。那么什么是模式原型呢？ 好，仍然没有通过"模式特性”测试的模式通常认为是模式原型。模式原型也许源自于某人已经确定的值得与社团共享的特定解决方案的工作，然而由于它提出时间短，所以可能仍然没有机会接受严格的审查。

另外，个人共享的模式也许没有时间或者没有兴趣通过“模式特性”测试这个过程，不过可能发布了这些模式原型的简短说明。这种类型模式的简要描述或者片段就是众所周知的小模式。

全面文档化具有资格的模式这样的工作是非常令人气馁的。回头看看设计模式领域最早期的某些工作，如果一个模式能做到以下事情，那么这个模式就可以认为是“好的”模式：

- 解决一类特定的问题：模式不能假设仅仅关注原理或者策略。它们需要关注解决方案。这是好的模式最重要的因素之一。
- 问题的解决方案不是表面上的：我们发现解决问题的技术常常首先试图源自于某个众所周知的原理。最佳的设计模式通常间接地提供问题的解决方案-认为模式是与设计相关的最具有挑战性的问题必然的解决方法。
- 所描述的想法一定得到了证明：设计模式需要提供所描述的它们运行的证据，如果没有这些证据，就不会认真的考虑这个设计。如果模式事实上是高度理论性的话，那么只有冒险者才可能试着用它。
- 它必须说明与代码之间关系：在某些情况下，模式似乎说明了一种类型的模块。虽然实现的可能就是这个方法，但是官方的模式说明一定要更深入的描述系统结构和机制，以解释它与代码之间的关系。

我们认为不满足准则的模式原型不值得学习，这可以得到谅解，然而，事实远不是这样的。许多模式原型确实非常的好。我不是说所有的模式原型都值得看，不过总有几个在自然环境下成长的有用的模式原型可以在未来的项目中帮到我们。从心底里使用上面列表来做最佳评判的话，你在选择哪个是模式的过程中将感觉非常愉快。

模式是否有效的附加要求之一是模式要展示某些重现现象。这个就是至少在三个关键方面 ，也就是三条规则验证是否取得资格经常要做的事情。为了展示使用这个规则后的重现，模式必须证明其：

- 适用性-模式怎样才能被认为是成功的。
- 有用性-为什么认为这个模式是成功的？
- 可用性 -因为设计得到广泛的应用，所以认为这个设计就是模式吗？如果是这样的话，那么需要说明。重新审核或者定义模式的时候，牢记以上规则非常重要。

## 设计模式的结构

你可能会对设计模式的作者如何接近勾勒出概念轮廓，实施和新模式的目的。模式是最初提出的一种在两者之间建立关系的规则：

- 上下文环境。
- 在这种环境下产生的系统的力量。
- 一类配置，考虑到允许这种力量在自己的上下文环境中解决这一点，现在让我们对一种设计模式的组件元素，一探究竟。一种设计模式应该具有。
- 模式名称和相应的描述。
- 上下文概述-在设计模式中的上下文对响应用户需求是很有效的。
- 问题声明-一类问题的声明，能让我们理解模式的意图。
- 解决方案-在可理解的列表和看法上，对用户的问题如何被解决的一种描述。
- 设计-模式设计，特别是与之交互的用户行为的描述。
- 实现-对模式如何被实现的一种指引。
- 例证-在模式中的一种类的虚拟化表示。
- 例子-模式实现的一种最下的形式。
- 共同条件-可能会有其他的什么模式会被用到，以对被描述的模式进行支持？
- 关系-与该模式相似的模式有哪些？是最相似的吗？
- 已知的使用-模式没有被正常使用？如果是，在哪，怎样做到的？
- 讨论-有激动人心的获利模式想法的团队或者是作者。

在一个组织或团队中，当在同一页面上创建和维护的解决方案时，对所有涉及到的开发者来说，设计模式能帮上大忙。如果考虑到你自己的工作模式，记住，虽然他们可能在制定计划和编写阶段，有一个较大的初期成本投入，但从投资方返回的值是值得的。然而，新的模式工作前，务必深入研究，你会发现它比起重新开始，更有利于使用或建立比现有的行之有效的模式之上。

## 编写设计模式

虽然本书的目标，针对的是新的设计模式，但对设计模式是怎样编写的有一个根本的理解后，会让我们受益匪浅。对于初学者来说，对于为什么需要一个模式背后的推理，我们可以得到更深的理解。我们同时也会学习到当我们在重视我们自己的需求的时候，如何区分一种模式（或原模式）。

要编写好的模式，是一种极具挑战性的任务。模式不仅仅需要对终端用户提供数量可观的材料，还要能够说明为什么需要这种模式。

在读过前续章节-什么是模式以后，我们可能会认为足够帮助我们去辨别我们在非标准条件下看到的模式。事实上这并非完全正确。这并不总是很清楚，如果我们正在寻找的一段代码，出现像它一样符合的一组模式，或只是偶然发生。

当我们在寻找认为可能使用某种设计模式的代码的时候，应该考虑写下的代码的一些方面，我们相信属于一个特定的现有格局或一组模式。

在很多模式分析的案例中，我们会发现，正巧看到了那些具有良好的原则和设计实践，而这些可能突然引起对模式的覆盖规则。记住-既不相互作用，也没有定义规则的解决方案模式。

如果敢于尝试编写自己的设计模式的道路，我推荐从其他那些已经过来之人学习，学习他们好的方面。花时间从大量不同的设计模式描述中吸取信息，并找到对你有意义的。

探索结构和语义-可以通过检查交互和你感兴趣的模式的上下文，因此你可以标示出运用有用的配置，将模式组织在一起的原则。

一旦我们暴露了自己丰富的模式文献资料，我们不妨使用现有的格式，开始写我们的模式，并看看我们是否能集思广益，打开新思路，对它进行改进或把我们的想法进行整合。

一个开发者的例子，该例子的作者是近几年的Christian Heilmann，他在对已存在的模式的基础上做了一些基本的改变，以此创建了暴露模块模式（该模式在本书后续部分会讲到）。

对于那些对创建新设计模式的人，我对他们有如下的建议：

- 模式是否实用？: 确保这个模式能够对一些常见的问题有明确的解决方案，而不是临时的解决方案。
- 保持最佳实践: 我们的设计需要以最佳实践中所获得的理解作为基础。
- 设计模式对用户来说应该是清晰的: 设计模式必须对任何形式的用户体验都是清晰的。 因为设计模式主要服务于开发者们，所以不能强迫他们去改变原来的行为，那样开发者们才会去使用这个模式。
- 独创力不是设计模式的关键： 当我们在设计一个模式的时候，我们既不需要是发明者，也不需要去担心是否是其他模式的子集。如果某个想法有很强的实用性，那么这就是一个创造新模式的机会。
- 需要有几个有说服力的例子： 一个好的设计模式需要有一个有说服力的例子来展示这个模式是成功的。为了广泛使用这个设计模式，这些例子需要展示良好的设计原则。

在创造一个新的设计模式的时候，在通用性，特殊性和可用性之间有一个微妙的平衡点。如果新的模式覆盖了应用中最多的可能情况，那么这个模式应该是良好的。我希望通过这段简介能够对下个章节内容的学习有所帮助。

## 反模式

如果我们认为模式代表一个最佳的实践，那么反模式将代表我们已经学到一个教训。受启发于Gof的《设计模式》，Andrew Koeing在1995年的11月的C++报告大会上首次提出反模式。在Koeing的报告中，反模式有着两种观念:

- 描述对于一个特殊的问题，提出了一个糟糕的解决方案，最终导致一个坏结果发生
- 描述如何摆脱上述解决方案并能提出一个好的解决方案

关于这个话题，Alexander写过要在好的设计结构和好的上下文中找到平衡是困难的:

> 这些笔记是关于设计的过程，这个过程发明显示一个新的物理顺序响应功能，组织形式，物质的东西......每一个设计问题开始于努力实现两个实体之间的形式：问题中的形式和它的上下文。此形式是解决问题的方法，而上下文定义了该问题。

虽然理解设计模式很重要，但对于理解反模式也是同等重要。我们有资格知道这背后的原因。当我们开发一个应用，这个工程的生命周期开始建设一直至项目完成，但一旦完成后，就进入维护阶段。判断一个解决方案的好坏要看这个团队在这个项目上投资的技术和花费的时间。这里被认为是好的和坏的情况下-如果应用在错误的情况下，一个“完美”的设计可能有资格作为一个反模式。

最大的挑战发生于应用进入生产和维护阶段。一个之前没有开发过这个应用的开发者来维护一个系统可能会引进糟糕的设计。如果说糟糕的设计是因为反模式，那么将允许开发者提前找到一种认识到时这样的手段，这样就能避免一些普通错误的发生，与此同时这也是设计模式给我们提供一种认识到普通技术也是有用的方式。

反模式是一个值得为此专门编写编写总结文档的糟糕设计。Javascript的反模式例子如下:

- 在全局上面文中定义大量污染全局命令空间的变量
- 在调用setTimeout和setInterval时传递字符串(会用eval来执行)而不是函数。
- 修改Object的原型 (这是最糟糕的反模式)
- 使用内联Javascript

在本应使用document.createElement的地方使用document.write。document.write被错误的用了相当多的年头，它有相当多的缺点，包括如果在页面加载后执行它可能会覆盖我们的页面。再有它不能工作在XHTML下，这也是另外一个我们使用像document.createElement这种对DOM友好方法的原因。

知道反模式对成功来说很关键。一旦我们能识别这些反模式，我们就能够重构我们的代码使项目的整体质量立马提升。

## 设计模式的分类

**设计模式的种类**

在众所周知的设计书《Domain-Driven Terms》中,它被描述为:

“设计模式是命名、抽象和识别对可重用的面向对象设计有用的的通用设计结构。设计模式确定类和他们的实体、他们的角色和协作、还有他们的责任分配。

每一个设计模式都聚焦于一个面向对象的设计难题或问题。它描述了在其它设计的约束下它能否使用，使用它后的后果和得失。因为我们必须最终实现我们的设计模式，所以每个设计模式都提供了例子..代码来对实现进行阐释。

虽然设计模式被描述为面向对象的设计，它们基于那些已经被主流面向对象语言实现过的解决方案...”

设计模式可以被分成几个不同的种类。在这个部分我们将复习三个分类，并且在我们进入特定的设计模式详情之前我们提到该分组下的模式的几个示例。

**创建型设计模式**

创建型设计模式关注于对象创建的机制方法，通过该方法,对象以适应工作环境的方式被创建。基本的对象创建方法可能会给项目增加额外的复杂性，而这些模式的目的就是为了通过控制创建过程解决这个问题。

属于这一类的一些模式是：构造器模式（Constructor）,工厂模式（Factory）,抽象工厂模式 （Abstract）,原型模式 （Prototype）,单例模式 （Singleton）以及 建造者模式（Builder）。

**结构设计模式**

结构模式关注于对象组成和通常识别的方式实现不同对象之间的关系。该模式有助于在系统的某一部分发生改变的时候，整个系统结构不需要改变。该模式同样有助于对系统中某部分没有达到某一目的的部分进行重组。

在该分类下的模式有：装饰模式，外观模式，享元模式，适配器模式和代理模式。

**行为设计模式**

行为模式关注改善或精简在系统中不同对象间通信。

行为模式包括：迭代模式，中介者模式，观察者模式和访问者模式。

**设计模式的分类**

在我早起学习设计模式的经验中，我个人发现，下面的表格是一个非常有用的提醒，大多数模式所提供-它覆盖了由GOF提出的23种模式。最早的表格由 Elyse Nielsen 在2004年汇总，我已经做了部分修改以适应我们的讨论。 我推荐使用该表格作为参考，但要记住大量额外的模式在这里么有提及，但在本书的后续的章节中会提到。

**关于类的简单说明**

要记住这张表中会有模式引用“类”的概念。JavaScript是一种弱类型语言，不过类可以通过函数模拟出来。 最常见的实现这一点的方法，是先定义一个JavaScript函数，然后再使用这个新的关键字创建一个对象。可以通过这种方法像下面这样给类定义新的属性与方法。

```
// A car "class"
function Car( model ) {

  this.model = model;
  this.color = "silver";
  this.year  = "2012";

  this.getInfo = function () {
    return this.model + " " + this.year;
  };

}
```

接着我们可以使用上面定义的Car构造函数实例化对象，就像这样：

```
var myCar = new Car("ford");

myCar.year = "2010";

console.log( myCar.getInfo() );
```

## 设计模式分类概览表


- Factory Method(工厂方法)	通过将数据和事件接口化来构建若干个子类。
- Abstract Factory(抽象工厂)	建立若干族类的一个实例，这个实例不需要具体类的细节信息。（抽象类）
- Builder (建造者)	将对象的构建方法和其表现形式分离开来，总是构建相同类型的对象。
- Prototype(原型)	一个完全初始化的实例，用于拷贝或者克隆。
- Singleton(单例)	一个类只有唯一的一个实例，这个实例在整个程序中有一个全局的访问点。
- Adapter(适配器)	将不同类的接口进行匹配，调整，这样尽管内部接口不兼容但是不同的类还是可以协同工作的。
- Bridge(桥接模式)	将对象的接口从其实现中分离出来，这样对象的实现和接口可以独立的变化。
- Composite(组合模式)	通过将简单可组合的对象组合起来，构成一个完整的对象，这个对象的能力将会超过这些组成部分的能力的总和，即会有新的能力产生。
- Decorator(装饰器)	动态给对象增加一些可替换的处理流程。
- Facada(外观模式)	一个类隐藏了内部子系统的复杂度，只暴露出一些简单的接口。
- Flyweight(享元模式)	一个细粒度对象，用于将包含在其它地方的信息 在不同对象之间高效地共享。
- Proxy(代理模式)	一个充当占位符的对象用来代表一个真实的对象。
- Interpreter(解释器)	将语言元素包含在一个应用中的一种方式，用于匹配目标语言的语法。
- Template Method(模板方法)	在一个方法中为某个算法建立一层外壳，将算法的具体步骤交付给子类去做。
- Chain of Responsibility(响应链)	一种将请求在一串对象中传递的方式，寻找可以处理这个请求的对象。
- Command(命令)	封装命令请求为一个对象，从而使记录日志，队列缓存请求，未处理请求进行错误处理 这些功能称为可能。
- Iterator(迭代器)	在不需要直到集合内部工作原理的情况下，顺序访问一个集合里面的元素。
- Mediator(中介者模式)	在类之间定义简化的通信方式，用于避免类之间显式的持有彼此的引用。
- Observer(观察者模式)	用于将变化通知给多个类的方式，可以保证类之间的一致性。
- State(状态)	当对象状态改变时，改变对象的行为。
- Strategy(策略)	将算法封装到类中，将选择和实现分离开来。
- Visitor(访问者)	为类增加新的操作而不改变类本身。


## JavaScript设计模式

### 构造器模式

在面向对象编程中，构造器是一个当新建对象的内存被分配后，用来初始化该对象的一个特殊函数。在JavaScript中几乎所有的东西都是对象，我们经常会对对象的构造器十分感兴趣。

对象构造器是被用来创建特殊类型的对象的，首先它要准备使用的对象，其次在对象初次被创建时，通过接收参数，构造器要用来对成员的属性和方法进行赋值。

对象创建

下面是我们创建对象的三种基本方式:

下面的每一种都会创建一个新的对象:

```
var newObject = {};

// or
var newObject = Object.create( null );

// or
var newObject = new Object();
```

最后一个例子中"Object"构造器创建了一个针对特殊值的对象包装，只不过这里没有传值给它，所以它将会返回一个空对象。

有四种方式可以将一个键值对赋给一个对象:

```
// ECMAScript 3 兼容形式

// 1. “点号”法

// 设置属性
newObject.someKey = "Hello World";

// 获取属性
var key = newObject.someKey;

// 2. “方括号”法

// 设置属性
newObject["someKey"] = "Hello World";

// 获取属性
var key = newObject["someKey"];

// ECMAScript 5 仅兼容性形式
// For more information see: http://kangax.github.com/es5-compat-table/

// 3. Object.defineProperty方式

// 设置属性
Object.defineProperty(newObject, "someKey", {
    value: "for more control of the property's behavior",
    writable: true,
    enumerable: true,
    configurable: true
});

// 如果上面的方式你感到难以阅读，可以简短的写成下面这样：

var defineProp = function (obj, key, value) {
    config.value = value;
    Object.defineProperty(obj, key, config);
};

// 为了使用它，我们要创建一个“person”对象
var person = Object.create(null);

// 用属性构造对象
defineProp(person, "car", "Delorean");
defineProp(person, "dateOfBirth", "1981");
defineProp(person, "hasBeard", false);

// 4. Object.defineProperties方式

// 设置属性
Object.defineProperties(newObject, {

    "someKey": {
        value: "Hello World",
        writable: true
    },

    "anotherKey": {
        value: "Foo bar",
        writable: false
    }

});

// 3和4中的读取属行可用1和2中的任意一种
```

在这本书的后面一点，这些方法会被用于继承，如下：

```
// 使用:

// 创建一个继承与Person的赛车司机
var driver = Object.create(person);

// 设置司机的属性
defineProp(driver, "topSpeed", "100mph");

// 获取继承的属性 (1981)
console.log(driver.dateOfBirth);

// 获取我们设置的属性 (100mph)
console.log(driver.topSpeed);
```

基础构造器

正如我们先前所看到的，Javascript不支持类的概念，但它有一种与对象一起工作的构造器函数。使用new关键字来调用该函数，我们可以告诉Javascript把这个函数当做一个构造器来用,它可以用自己所定义的成员来初始化一个对象。

在这个构造器内部，关键字this引用到刚被创建的对象。回到对象创建，一个基本的构造函数看起来像这样:

```
function Car(model, year, miles) {
    this.model = model;
    this.year = year;
    this.miles = miles;

    this.toString = function () {
        return this.model + " has done " + this.miles + " miles";
    };
}

// 使用:

// 我们可以示例化一个Car
var civic = new Car("Honda Civic", 2009, 20000);
var mondeo = new Car("Ford Mondeo", 2010, 5000);

// 打开浏览器控制台查看这些对象toString()方法的输出值
// output of the toString() method being called on
// these objects
console.log(civic.toString());
console.log(mondeo.toString());
```
上面这是个简单版本的构造器模式，但它还是有些问题。一个是难以继承，另一个是每个Car构造函数创建的对象中，toString()之类的函数都被重新定义。这不是非常好，理想的情况是所有Car类型的对象都应该引用同一个函数。 这要谢谢 ECMAScript3和ECMAScript5-兼容版，对于构造对象他们提供了另外一些选择，解决限制小菜一碟。

使用“原型”的构造器

在Javascript中函数有一个prototype的属性。当我们调用Javascript的构造器创建一个对象时，构造函数prototype上的属性对于所创建的对象来说都看见。照这样，就可以创建多个访问相同prototype的Car对象了。下面，我们来扩展一下原来的例子：

```
function Car(model, year, miles) {

    this.model = model;
    this.year = year;
    this.miles = miles;

}

// 注意这里我们使用Note here that we are using Object.prototype.newMethod 而不是
// Object.prototype ，以避免我们重新定义原型对象
Car.prototype.toString = function () {
    return this.model + " has done " + this.miles + " miles";
};

// 使用:

var civic = new Car("Honda Civic", 2009, 20000);
var mondeo = new Car("Ford Mondeo", 2010, 5000);

console.log(civic.toString());
console.log(mondeo.toString());
```

通过上面代码，单个toString()实例被所有的Car对象所共享了。

### 模块化模式

模块
模块是任何健壮的应用程序体系结构不可或缺的一部分，特点是有助于保持应用项目的代码单元既能清晰地分离又有组织。

在JavaScript中，实现模块有几个选项，他们包括：

- 模块化模式
- 对象表示法
- AMD模块
- CommonJS 模块
- ECMAScript Harmony 模块

我们在书中后面的现代模块化JavaScript设计模式章节中将探讨这些选项中的最后三个。

模块化模式是基于对象的文字部分，所以首先对于更新我们对它们的知识是很有意义的。

**对象字面值**

在对象字面值的标记里，一个对象被描述为一组以逗号分隔的名称/值对括在大括号（{}）的集合。对象内部的名称可以是字符串或是标记符后跟着一个冒号":"。在对象里最后一个名称/值对不应该以","为结束符，因为这样会导致错误。

```
var myObjectLiteral = {

    variableKey: variableValue,

    functionKey: function () {
      // ...
    }
};
```

对象字面值不要求使用NEW操作实例，但是不能够在结构体开始使用，因为打开"{"可能被解释为一个块的开始。在对象外NEW成员会被加载，使用分配如下：smyModule.property = "someValue"; 下面我们可以看到一个更完整的使用对象字面值定义一个模块的例子：

```
var myModule = {

    myProperty: "someValue",

    // 对象字面值包含了属性和方法（properties and methods）.
    // 例如，我们可以定义一个模块配置进对象：
    myConfig: {
        useCaching: true,
        language: "en"
    },

    // 非常基本的方法
    myMethod: function () {
        console.log("Where in the world is Paul Irish today?");
    },

    // 输出基于当前配置（<span>configuration</span>）的一个值
    myMethod2: function () {
        console.log("Caching is:" + (this.myConfig.useCaching) ? "enabled" : "disabled");
    },

    // 重写当前的配置（configuration）
    myMethod3: function (newConfig) {

        if (typeof newConfig === "object") {
            this.myConfig = newConfig;
            console.log(this.myConfig.language);
        }
    }
};

// 输出: Where in the world is Paul Irish today?
myModule.myMethod();

// 输出: enabled
myModule.myMethod2();

// 输出: fr
myModule.myMethod3({
    language: "fr",
    useCaching: false
});
```
使用对象字面值可以协助封装和组织你的代码。如果你想近一步了解对象字面值可以阅读 Rebecca Murphey 写过的关于此类话题的更深入的文章(depth)。

也就是说，如果我们选择了这种技术，我们可能对模块模式有同样的兴趣。即使使用对象字面值，但也只有一个函数的返回值。

**模块化模式**

模块化模式最初被定义为一种对传统软件工程中的类提供私有和公共封装的方法。

在JavaScript中，模块化模式用来进一步模拟类的概念，通过这样一种方式：我们可以在一个单一的对象中包含公共/私有的方法和变量，从而从全局范围中屏蔽特定的部分。这个结果是可以减少我们的函数名称与在页面中其他脚本区域定义的函数名称冲突的可能性。

**私有信息**

模块模式使用闭包的方式来将"私有信息",状态和组织结构封装起来。提供了一种将公有和私有方法，变量封装混合在一起的方式，这种方式防止内部信息泄露到全局中，从而避免了和其它开发者接口发生冲图的可能性。在这种模式下只有公有的API 会返回，其它将全部保留在闭包的私有空间中。

这种方法提供了一个比较清晰的解决方案，在只暴露一个接口供其它部分使用的情况下，将执行繁重任务的逻辑保护起来。这个模式非常类似于立即调用函数式表达式(IIFE-查看命名空间相关章节获取更多信息)，但是这种模式返回的是对象，而立即调用函数表达式返回的是一个函数。

需要注意的是，在javascript事实上没有一个显式的真正意义上的"私有性"概念，因为与传统语言不同，javascript没有访问修饰符。从技术上讲，变量不能被声明为公有的或者私有的，因此我们使用函数域的方式去模拟这个概念。在模块模式中，因为闭包的缘故，声明的变量或者方法只在模块内部有效。在返回对象中定义的变量或者方法可以供任何人使用。

历史

从历史角度来看，模块模式最初是在2003年由一群人共同发展出来的，这其中包括Richard Cornford。后来通过Douglas Crockford的演讲，逐渐变得流行起来。另外一件事情是，如果你曾经用过雅虎的YUI库，你会看到其中的一些特性和模块模式非常类似，而这种情况的原因是在创建YUI框架的时候，模块模式极大的影响了YUI的设计。

例子

下面这个例子通过创建一个自包含的模块实现了模块模式。

```
var testModule = (function () {

    var counter = 0;
    
    return {
        incrementCounter: function () {
            return counter++;
        },
        resetCounter: function () {
            console.log("counter value prior to reset: " + counter);
            counter = 0;
        }
    };
})();

// Usage:

// Increment our counter
testModule.incrementCounter();

// Check the counter value and reset
// Outputs: 1
testModule.resetCounter();
```

在这里我们看到，其它部分的代码不能直接访问我们的incrementCounter() 或者 resetCounter()的值。counter变量被完全从全局域中隔离起来了，因此其表现的就像一个私有变量一样，它的存在只局限于模块的闭包内部，因此只有两个函数可以访问counter。我们的方法是有名字空间限制的，因此在我们代码的测试部分，我们需要给所有函数调用前面加上模块的名字(例如"testModule")。

当使用模块模式时，我们会发现通过使用简单的模板，对于开始使用模块模式非常有用。下面是一个模板包含了命名空间，公共变量和私有变量。

```
var myNamespace = (function () {

    var myPrivateVar, myPrivateMethod;

    // A private counter variable
    myPrivateVar = 0;

    // A private function which logs any arguments
    myPrivateMethod = function (foo) {
        console.log(foo);
    };

    return {

        // A public variable
        myPublicVar: "foo",

        // A public function utilizing privates
        myPublicFunction: function (bar) {

            // Increment our private counter
            myPrivateVar++;

            // Call our private method using bar
            myPrivateMethod(bar);

        }
    };

})();
```

看一下另外一个例子，下面我们看到一个使用这种模式实现的购物车。这个模块完全自包含在一个叫做basketModule 全局变量中。模块中的购物车数组是私有的，应用的其它部分不能直接读取。只存在与模块的闭包中，因此只有可以访问其域的方法可以访问这个变量。

```
var basketModule = (function () {

    // privates

    var basket = [];

    function doSomethingPrivate() {
        //...
    }

    function doSomethingElsePrivate() {
        //...
    }

    // Return an object exposed to the public
    return {

        // Add items to our basket
        addItem: function (values) {
            basket.push(values);
        },

        // Get the count of items in the basket
        getItemCount: function () {
            return basket.length;
        },

        // Public alias to a  private function
        doSomething: doSomethingPrivate,

        // Get the total value of items in the basket
        getTotal: function () {

            var q = this.getItemCount(),
                p = 0;

            while (q--) {
                p += basket[q].price;
            }

            return p;
        }
    };
}());
```

在模块内部，你可能注意到我们返回了应外一个对象。这个自动赋值给了basketModule 因此我们可以这样和这个对象交互。

```
// basketModule returns an object with a public API we can use

basketModule.addItem({
    item: "bread",
    price: 0.5
});

basketModule.addItem({
    item: "butter",
    price: 0.3
});

// Outputs: 2
console.log(basketModule.getItemCount());

// Outputs: 0.8
console.log(basketModule.getTotal());

// However, the following will not work:

// Outputs: undefined
// This is because the basket itself is not exposed as a part of our
// the public API
console.log(basketModule.basket);

// This also won't work as it only exists within the scope of our
// basketModule closure, but not the returned public object
console.log(basket);
```

上面的方法都处于basketModule 的名字空间中。

请注意在上面的basket模块中 域函数是如何在我们所有的函数中被封装起来的，以及我们如何立即调用这个域函数，并且将返回值保存下来。这种方式有以下的优势：

可以创建只能被我们模块访问的私有函数。这些函数没有暴露出来（只有一些API是暴露出来的），它们被认为是完全私有的。
当我们在一个调试器中，需要发现哪个函数抛出异常的时候，可以很容易的看到调用栈，因为这些函数是正常声明的并且是命名的函数。
正如过去 T.J Crowder 指出的，这种模式同样可以让我们在不同的情况下返回不同的函数。我见过有开发者使用这种技巧用于执行UA（尿检，抽样检查）测试，目的是为了在他们的模块里面针对IE专门提供一条代码路径，但是现在我们也可以简单的使用特征检测达到相同的目的。
模块模式的变体

**Import mixins(导入混合)**

这个变体展示了如何将全局（例如 jQuery, Underscore）作为一个参数传入模块的匿名函数。这种方式允许我们导入全局，并且按照我们的想法在本地为这些全局起一个别名。

```
// Global module
var myModule = (function (jQ, _) {

    function privateMethod1() {
        jQ(".container").html("test");
    }

    function privateMethod2() {
        console.log(_.min([10, 5, 100, 2, 1000]));
    }

    return {
        publicMethod: function () {
            privateMethod1();
        }
    };

    // Pull in jQuery and Underscore
}(jQuery, _));

myModule.publicMethod();
```

**Exports（导出）**

这个变体允许我们声明全局对象而不用使用它们，同样也支持在下一个例子中我们将会看到的全局导入的概念。

```
// Global module
var myModule = (function () {

    // Module object
    var module = {},
        privateVariable = "Hello World";

    function privateMethod() {
        // ...
    }

    module.publicProperty = "Foobar";
    module.publicMethod = function () {
        console.log(privateVariable);
    };

    return module;

}());
```

工具箱和框架特定的模块模式实现。

**Dojo**

Dojo提供了一个方便的方法 dojo.setObject() 来设置对象。这需要将以"."符号为第一个参数的分隔符，如：myObj.parent.child 是指定义在"myOjb"内部的一个对象“parent”，它的一个属性为"child"。使用setObject()方法允许我们设置children 的值，可以创建路径传递过程中的任何对象即使这些它们根本不存在。

例如，如果我们声明商店命名空间的对象basket.coreas，可以实现使用传统的方式如下：

```
var store = window.store || {};

if (!store["basket"]) {
    store.basket = {};
}

if (!store.basket["core"]) {
    store.basket.core = {};
}

store.basket.core = {
    // ...rest of our logic
};
```

或使用Dojo1.7（AMD兼容的版本）及以上如下：

```
require(["dojo/_base/customStore"], function (store) {

    // using dojo.setObject()
    store.setObject("basket.core", (function () {

        var basket = [];

        function privateMethod() {
            console.log(basket);
        }

        return {
            publicMethod: function () {
                privateMethod();
            }
        };

    }()));

});
```

欲了解更多关于dojo.setObject（）方法的信息，请参阅官方文档 documentation

**ExtJS**

对于这些使用Sencha的ExtJS的人们，你们很幸运，因为官方文档包含一些例子，用于展示如何正确地在框架里面使用模块模式。

下面我们可以看到一个例子关于如何定义一个名字空间，然后填入一个包含有私有和公有API的模块。除了一些语义上的不同之外，这个例子和使用vanilla javascript 实现的模块模式非常相似。

```
// create namespace
Ext.namespace("myNameSpace");

// create application
myNameSpace.app = function () {

    // do NOT access DOM from here; elements don't exist yet
    // private variables

    var btn1,
        privVar1 = 11;

    // private functions
    var btn1Handler = function (button, event) {
        console.log("privVar1=" + privVar1);
        console.log("this.btn1Text=" + this.btn1Text);
    };

    // public space
    return {
        // public properties, e.g. strings to translate
        btn1Text: "Button 1",

        // public methods
        init: function () {

            if (Ext.Ext2) {

                btn1 = new Ext.Button({
                    renderTo: "btn1-ct",
                    text: this.btn1Text,
                    handler: btn1Handler
                });

            } else {

                btn1 = new Ext.Button("btn1-ct", {
                    text: this.btn1Text,
                    handler: btn1Handler
                });

            }
        }
    };
}();
```

**YUI**

类似地，我们也可以使用YUI3来实现模块模式。下面的例子很大程度上是基于原始由Eric Miraglia实现的YUI本身的模块模式，但是和vanillla Javascript 实现的版本比较起来差异不是很大。

```
Y.namespace("store.basket") = (function () {

    var myPrivateVar, myPrivateMethod;

    // private variables:
    myPrivateVar = "I can be accessed only within Y.store.basket.";

    // private method:
    myPrivateMethod = function () {
        Y.log("I can be accessed only from within YAHOO.store.basket");
    }

    return {
        myPublicProperty: "I'm a public property.",

        myPublicMethod: function () {
            Y.log("I'm a public method.");

            // Within basket, I can access "private" vars and methods:
            Y.log(myPrivateVar);
            Y.log(myPrivateMethod());

            // The native scope of myPublicMethod is store so we can
            // access public members using "this":
            Y.log(this.myPublicProperty);
        }
    };

})();
```

**jQuery**

因为jQuery编码规范没有规定插件如何实现模块模式，因此有很多种方式可以实现模块模式。Ben Cherry 之间提供一种方案，因为模块之间可能存在大量的共性，因此通过使用函数包装器封装模块的定义。

在下面的例子中，定义了一个library 函数，这个函数声明了一个新的库，并且在新的库（例如 模块）创建的时候，自动将初始化函数绑定到document的ready上。

```
function library(module) {

    $(function () {
        if (module.init) {
            module.init();
        }
    });

    return module;
}

var myLibrary = library(function () {

    return {
        init: function () {
            // module implementation
        }
    };
}());
```

- 优势

既然我们已经看到单例模式很有用，为什么还是使用模块模式呢？首先，对于有面向对象背景的开发者来讲，至少从javascript语言上来讲，模块模式相对于真正的封装概念更清晰。

其次，模块模式支持私有数据-因此，在模块模式中，公共部分代码可以访问私有数据，但是在模块外部，不能访问类的私有部分（没开玩笑！感谢David Engfer 的玩笑）。

- 缺点

模块模式的缺点是因为我们采用不同的方式访问公有和私有成员，因此当我们想要改变这些成员的可见性的时候，我们不得不在所有使用这些成员的地方修改代码。

我们也不能在对象之后添加的方法里面访问这些私有变量。也就是说，很多情况下，模块模式很有用，并且当使用正确的时候，潜在地可以改善我们代码的结构。

其它缺点包括不能为私有成员创建自动化的单元测试，以及在紧急修复bug时所带来的额外的复杂性。根本没有可能可以对私有成员打补丁。相反地，我们必须覆盖所有的使用存在bug私有成员的公共方法。开发者不能简单的扩展私有成员，因此我们需要记得，私有成员并非它们表面上看上去那么具有扩展性。


### 暴露模块模式

既然我们对模块模式已经有一些了解了，让我们看一下改进版本 - Christian Heilmann 的启发式模块模式。 启发式模块模式来自于，当Heilmann对这样一个现状的不满，即当我们想要在一个公有方法中调用另外一个公有方法，或者访问公有变量的时候，我们不得不重复主对象的名称。他也不喜欢模块模式中，当想要将某个成员变成公共成员时，修改文字标记的做法。

因此他工作的结果就是一个更新的模式，在这个模式中，我们可以简单地在私有域中定义我们所有的函数和变量，并且返回一个匿名对象，这个对象包含有一些指针，这些指针指向我们想要暴露出来的私有成员，使这些私有成员公有化。

下面给出一个如何使用暴露式模块模式的例子:

```
var myRevealingModule = function () {

    var privateVar = "Ben Cherry",
        publicVar  = "Hey there!";

    function privateFunction() {
        console.log( "Name:" + privateVar );
    }

    function publicSetName( strName ) {
        privateVar = strName;
    }

    function publicGetName() {
        privateFunction();
    }

    // Reveal public pointers to 
    // private functions and properties

    return {
        setName: publicSetName,
        greeting: publicVar,
        getName: publicGetName
    };

}();

myRevealingModule.setName( "Paul Kinlan" );
```

这个模式可以用于将私有函数和属性以更加规范的命名方式展现出来。


```
var myRevealingModule = function () {

    var privateCounter = 0;

    function privateFunction() {
        privateCounter++;
    }

    function publicFunction() {
        publicIncrement();
    }

    function publicIncrement() {
        privateFunction();
    }

    function publicGetCount(){
      return privateCounter;
    }

    // Reveal public pointers to
    // private functions and properties       

   return {
        start: publicFunction,
        increment: publicIncrement,
        count: publicGetCount
    };

}();

myRevealingModule.start();
```

- 优势

这个模式是我们脚本的语法更加一致。同样在模块的最后关于那些函数和变量可以被公共访问也变得更加清晰，增强了可读性。

- 缺点

这个模式的一个缺点是如果私有函数需要使用公有函数，那么这个公有函数在需要打补丁的时候就不能被重载。因为私有函数仍然使用的是私有的实现，并且这个模式不能用于公有成员，只用于函数。

公有成员使用私有成员也遵循上面不能打补丁的规则。

因为上面的原因，使用暴露式模块模式创建的模块相对于原始的模块模式更容易出问题，因此在使用的时候需要小心。

### 单例模式

单例模式之所以这么叫，是因为它限制一个类只能有一个实例化对象。经典的实现方式是，创建一个类，这个类包含一个方法，这个方法在没有对象存在的情况下，将会创建一个新的实例对象。如果对象存在，这个方法只是返回这个对象的引用。

单例和静态类不同，因为我们可以退出单例的初始化时间。通常这样做是因为，在初始化的时候需要一些额外的信息，而这些信息在声明的时候无法得知。对于并不知晓对单例模式引用的代码来讲，单例模式没有为它们提供一种方式可以简单的获取单例模式。这是因为，单例模式既不返回对象也不返回类，它只返回一种结构。可以类比闭包中的变量不是闭包-提供闭包的函数域是闭包（绕进去了）。

在JavaScript语言中, 单例服务作为一个从全局空间的代码实现中隔离出来共享的资源空间是为了提供一个单独的函数访问指针。

我们能像这样实现一个单例:

```
var mySingleton = (function () {

  // Instance stores a reference to the Singleton
  var instance;

  function init() {

    // 单例

    // 私有方法和变量
    function privateMethod(){
        console.log( "I am private" );
    }

    var privateVariable = "Im also private";

    var privateRandomNumber = Math.random();

    return {

      // 共有方法和变量
      publicMethod: function () {
        console.log( "The public can see me!" );
      },

      publicProperty: "I am also public",

      getRandomNumber: function() {
        return privateRandomNumber;
      }

    };

  };

  return {

    // 如果存在获取此单例实例，如果不存在创建一个单例实例
    getInstance: function () {

      if ( !instance ) {
        instance = init();
      }

      return instance;
    }

  };

})();

var myBadSingleton = (function () {

  // 存储单例实例的引用
  var instance;

  function init() {

    // 单例

    var privateRandomNumber = Math.random();

    return {

      getRandomNumber: function() {
        return privateRandomNumber;
      }

    };

  };

  return {

    // 总是创建一个新的实例
    getInstance: function () {

      instance = init();

      return instance;
    }

  };

})();

// 使用:

var singleA = mySingleton.getInstance();
var singleB = mySingleton.getInstance();
console.log( singleA.getRandomNumber() === singleB.getRandomNumber() ); // true

var badSingleA = myBadSingleton.getInstance();
var badSingleB = myBadSingleton.getInstance();
console.log( badSingleA.getRandomNumber() !== badSingleB.getRandomNumber() ); // true
```

创建一个全局访问的单例实例 (通常通过 MySingleton.getInstance()) 因为我们不能(至少在静态语言中) 直接调用 new MySingleton() 创建实例. 这在JavaScript语言中是不可能的。

在四人帮(GoF)的书里面，单例模式的应用描述如下：

每个类只有一个实例，这个实例必须通过一个广为人知的接口，来被客户访问。
子类如果要扩展这个唯一的实例，客户可以不用修改代码就能使用这个扩展后的实例。
关于第二点，可以参考如下的实例，我们需要这样编码:

```
mySingleton.getInstance = function(){
  if ( this._instance == null ) {
    if ( isFoo() ) {
       this._instance = new FooSingleton();
    } else {
       this._instance = new BasicSingleton();
    }
  }
  return this._instance;
};
```

在这里，getInstance 有点类似于工厂方法，我们不需要去更新每个访问单例的代码。FooSingleton可以是BasicSinglton的子类，并且实现了相同的接口。

为什么对于单例模式来讲，延迟执行执行这么重要？

在c++代码中，单例模式将不可预知的动态初始化顺序问题隔离掉，将控制权返回给程序员。

区分类的静态实例和单例模式很重要：尽管单例模式可以被实现成一个静态实例，但是单例可以懒构造，在真正用到之前，单例模式不需要分配资源或者内存。

如果我们有个静态对象可以被直接初始化，我们需要保证代码总是以同样的顺序执行（例如 汽车需要轮胎先初始化）当你有很多源文件的时候，这种方式没有可扩展性。

单例模式和静态对象都很有用，但是不能滥用-同样的我们也不能滥用其它模式。

在实践中，当一个对象需要和另外的对象进行跨系统协作的时候，单例模式很有用。下面是一个单例模式在这种情况下使用的例子：

```
var SingletonTester = (function () {

  // options: an object containing configuration options for the singleton
  // e.g var options = { name: "test", pointX: 5}; 
  function Singleton( options )  {

    // set options to the options supplied
    // or an empty object if none are provided
    options = options || {};

    // set some properties for our singleton
    this.name = "SingletonTester";

    this.pointX = options.pointX || 6;

    this.pointY = options.pointY || 10; 

  }

  // our instance holder 
  var instance;

  // an emulation of static variables and methods
  var _static  = {  

    name:  "SingletonTester",

    // Method for getting an instance. It returns
    // a singleton instance of a singleton object
    getInstance:  function( options ) {   
      if( instance  ===  undefined )  {    
        instance = new Singleton( options );   
      }   

      return  instance; 

    } 
  }; 

  return  _static;

})();

var singletonTest  =  SingletonTester.getInstance({
  pointX:  5
});

// Log the output of pointX just to verify it is correct
// Outputs: 5
console.log( singletonTest.pointX );
```

尽管单例模式有着合理的使用需求，但是通常当我们发现自己需要在javascript使用它的时候，这是一种信号，表明我们可能需要去重新评估自己的设计。

这通常表明系统中的模块要么紧耦合要么逻辑过于分散在代码库的多个部分。单例模式更难测试，因为可能有多种多样的问题出现，例如隐藏的依赖关系，很难去创建多个实例，很难清理依赖关系，等等。

要想进一步了解关于单例的信息，可以读读 Miller Medeiros 推荐的这篇非常棒的关于单例模式以及单例模式各种各样问题的文章，也可以看看这篇文章的评论，这些评论讨论了单例模式是怎样增加了模块间的紧耦合。


### 观察者模式

观察者模式是这样一种设计模式。一个被称作被观察者的对象，维护一组被称为观察者的对象，这些对象依赖于被观察者，被观察者自动将自身的状态的任何变化通知给它们。

当一个被观察者需要将一些变化通知给观察者的时候，它将采用广播的方式，这条广播可能包含特定于这条通知的一些数据。

当特定的观察者不再需要接受来自于它所注册的被观察者的通知的时候，被观察者可以将其从所维护的组中删除。 在这里提及一下设计模式现有的定义很有必要。这个定义是与所使用的语言无关的。通过这个定义，最终我们可以更深层次地了解到设计模式如何使用以及其优势。在四人帮的《设计模式:可重用的面向对象软件的元素》这本书中，是这样定义观察者模式的:

一个或者更多的观察者对一个被观察者的状态感兴趣，将自身的这种兴趣通过附着自身的方式注册在被观察者身上。当被观察者发生变化，而这种便可也是观察者所关心的，就会产生一个通知，这个通知将会被送出去，最后将会调用每个观察者的更新方法。当观察者不在对被观察者的状态感兴趣的时候，它们只需要简单的将自身剥离即可。

我们现在可以通过实现一个观察者模式来进一步扩展我们刚才所学到的东西。这个实现包含一下组件：

- 被观察者：维护一组观察者， 提供用于增加和移除观察者的方法。
- 观察者：提供一个更新接口，用于当被观察者状态变化时，得到通知。
- 具体的被观察者：状态变化时广播通知给观察者，保持具体的观察者的信息。
- 具体的观察者：保持一个指向具体被观察者的引用，实现一个更新接口，用于观察，以便保证自身状态总是和被观察者状态一致的。

首先，让我们对被观察者可能有的一组依赖其的观察者进行建模：

```
function ObserverList(){
  this.observerList = [];
}

ObserverList.prototype.Add = function( obj ){
  return this.observerList.push( obj );
};

ObserverList.prototype.Empty = function(){
  this.observerList = [];
};

ObserverList.prototype.Count = function(){
  return this.observerList.length;
};

ObserverList.prototype.Get = function( index ){
  if( index > -1 && index < this.observerList.length ){
    return this.observerList[ index ];
  }
};

ObserverList.prototype.Insert = function( obj, index ){
  var pointer = -1;

  if( index === 0 ){
    this.observerList.unshift( obj );
    pointer = index;
  }else if( index === this.observerList.length ){
    this.observerList.push( obj );
    pointer = index;
  }

  return pointer;
};

ObserverList.prototype.IndexOf = function( obj, startIndex ){
  var i = startIndex, pointer = -1;

  while( i < this.observerList.length ){
    if( this.observerList[i] === obj ){
      pointer = i;
    }
    i++;
  }

  return pointer;
};

ObserverList.prototype.RemoveAt = function( index ){
  if( index === 0 ){
    this.observerList.shift();
  }else if( index === this.observerList.length -1 ){
    this.observerList.pop();
  }
};

// Extend an object with an extension
function extend( extension, obj ){
  for ( var key in extension ){
    obj[key] = extension[key];
  }
}
```

接着，我们对被观察者以及其增加，删除，通知在观察者列表中的观察者的能力进行建模：

```
function Subject(){
  this.observers = new ObserverList();
}

Subject.prototype.AddObserver = function( observer ){
  this.observers.Add( observer );
}; 

Subject.prototype.RemoveObserver = function( observer ){
  this.observers.RemoveAt( this.observers.IndexOf( observer, 0 ) );
}; 

Subject.prototype.Notify = function( context ){
  var observerCount = this.observers.Count();
  for(var i=0; i < observerCount; i++){
    this.observers.Get(i).Update( context );
  }
};
```

我们接着定义建立新的观察者的一个框架。这里的update 函数之后会被具体的行为覆盖。

```
// The Observer
function Observer(){
  this.Update = function(){
    // ...
  };
}
```

在我们的样例应用里面，我们使用上面的观察者组件，现在我们定义：

- 一个按钮，这个按钮用于增加新的充当观察者的选择框到页面上
- 一个控制用的选择框 , 充当一个被观察者，通知其它选择框是否应该被选中
- 一个容器，用于放置新的选择框

我们接着定义具体被观察者和具体观察者，用于给页面增加新的观察者，以及实现更新接口。通过查看下面的内联的注释，搞清楚在我们样例中的这些组件是如何工作的。

HTML

```
<button id="addNewObserver">Add New Observer checkbox</button>
<input id="mainCheckbox" type="checkbox"/>
<div id="observersContainer"></div>
```

Sample script


```
// 我们DOM 元素的引用

var controlCheckbox = document.getElementById( "mainCheckbox" ),
  addBtn = document.getElementById( "addNewObserver" ),
  container = document.getElementById( "observersContainer" );

// 具体的被观察者

//Subject 类扩展controlCheckbox 类
extend( new Subject(), controlCheckbox );

//点击checkbox 将会触发对观察者的通知
controlCheckbox["onclick"] = new Function( "controlCheckbox.Notify(controlCheckbox.checked)" );

addBtn["onclick"] = AddNewObserver;

// 具体的观察者

function AddNewObserver(){

  //建立一个新的用于增加的checkbox
  var check  = document.createElement( "input" );
  check.type = "checkbox";

  // 使用Observer 类扩展checkbox
  extend( new Observer(), check );

  // 使用定制的Update函数重载
  check.Update = function( value ){
    this.checked = value;
  };

  // 增加新的观察者到我们主要的被观察者的观察者列表中
  controlCheckbox.AddObserver( check );

  // 将元素添加到容器的最后
  container.appendChild( check );
}
```

在这个例子里面，我们看到了如何实现和配置观察者模式，了解了被观察者，观察者，具体被观察者，具体观察者的概念。

**观察者模式和发布/订阅模式的不同**

观察者模式确实很有用，但是在javascript时间里面，通常我们使用一种叫做发布/订阅模式的变体来实现观察者模式。这两种模式很相似，但是也有一些值得注意的不同。

观察者模式要求想要接受相关通知的观察者必须到发起这个事件的被观察者上注册这个事件。

发布/订阅模式使用一个主题/事件频道，这个频道处于想要获取通知的订阅者和发起事件的发布者之间。这个事件系统允许代码定义应用相关的事件，这个事件可以传递特殊的参数，参数中包含有订阅者所需要的值。这种想法是为了避免订阅者和发布者之间的依赖性。

这种和观察者模式之间的不同，使订阅者可以实现一个合适的事件处理函数，用于注册和接受由发布者广播的相关通知。

这里给出一个关于如何使用发布者/订阅者模式的例子，这个例子中完整地实现了功能强大的publish(), subscribe() 和 unsubscribe()。

```
// 一个非常简单的邮件处理器

// 接受的消息的计数器
var mailCounter = 0;

// 初始化一个订阅者，这个订阅者监听名叫"inbox/newMessage" 的频道

// 渲染新消息的粗略信息
var subscriber1 = subscribe( "inbox/newMessage", function( topic, data ) {

  // 日志记录主题，用于调试
  console.log( "A new message was received: ", topic );

  // 使用来自于被观察者的数据，用于给用户展示一个消息的粗略信息
  $( ".messageSender" ).html( data.sender );
  $( ".messagePreview" ).html( data.body );

});

// 这是另外一个订阅者，使用相同的数据执行不同的任务

// 更细计数器，显示当前来自于发布者的新信息的数量
var subscriber2 = subscribe( "inbox/newMessage", function( topic, data ) {

  $('.newMessageCounter').html( mailCounter++ );

});

publish( "inbox/newMessage", [{
  sender:"hello@google.com",
  body: "Hey there! How are you doing today?"
}]);

// 在之后，我们可以让我们的订阅者通过下面的方式取消订阅来自于新主题的通知
// unsubscribe( subscriber1,  );
// unsubscribe( subscriber2 );
```

这个例子的更广的意义是对松耦合的原则的一种推崇。不是一个对象直接调用另外一个对象的方法，而是通过订阅另外一个对象的一个特定的任务或者活动，从而在这个任务或者活动出现的时候的得到通知。

- 优势

观察者和发布/订阅模式鼓励人们认真考虑应用不同部分之间的关系，同时帮助我们找出这样的层，该层中包含有直接的关系，这些关系可以通过一些列的观察者和被观察者来替换掉。这中方式可以有效地将一个应用程序切割成小块，这些小块耦合度低，从而改善代码的管理，以及用于潜在的代码复用。

使用观察者模式更深层次的动机是，当我们需要维护相关对象的一致性的时候，我们可以避免对象之间的紧密耦合。例如，一个对象可以通知另外一个对象，而不需要知道这个对象的信息。

两种模式下，观察者和被观察者之间都可以存在动态关系。这提供很好的灵活性，而当我们的应用中不同的部分之间紧密耦合的时候，是很难实现这种灵活性的。

尽管这些模式并不是万能的灵丹妙药，这些模式仍然是作为最好的设计松耦合系统的工具之一，因此在任何的JavaScript 开发者的工具箱里面，都应该有这样一个重要的工具。

- 缺点

事实上，这些模式的一些问题实际上正是来自于它们所带来的一些好处。在发布/订阅模式中，将发布者共订阅者上解耦，将会在一些情况下，导致很难确保我们应用中的特定部分按照我们预期的那样正常工作。

例如，发布者可以假设有一个或者多个订阅者正在监听它们。比如我们基于这样的假设，在某些应用处理过程中来记录或者输出错误日志。如果订阅者执行日志功能崩溃了（或者因为某些原因不能正常工作），因为系统本身的解耦本质，发布者没有办法感知到这些事情。

另外一个这种模式的缺点是，订阅者对彼此之间存在没有感知，对切换发布者的代价无从得知。因为订阅者和发布者之间的动态关系，更新依赖也很能去追踪。

- 发布/订阅实现

发布/订阅在JavaScript的生态系统中非常合适，主要是因为作为核心的ECMAScript 实现是事件驱动的。尤其是在浏览器环境下更是如此，因为DOM使用事件作为其主要的用于脚本的交互API。

也就是说，无论是ECMAScript 还是DOM都没有在实现代码中提供核心对象或者方法用于创建定制的事件系统（DOM3 的CustomEvent是一个例外，这个事件绑定在DOM上，因此通常用处不大）。

幸运的是，流行的JavaScript库例如dojo, jQuery(定制事件)以及YUI已经有相关的工具，可以帮助我们方便的实现一个发布/订阅者系统。下面我们看一些例子。

```
// 发布

// jQuery: $(obj).trigger("channel", [arg1, arg2, arg3]);
$( el ).trigger( "/login", [{username:"test", userData:"test"}] );

// Dojo: dojo.publish("channel", [arg1, arg2, arg3] );
dojo.publish( "/login", [{username:"test", userData:"test"}] );

// YUI: el.publish("channel", [arg1, arg2, arg3]);
el.publish( "/login", {username:"test", userData:"test"} );

// 订阅

// jQuery: $(obj).on( "channel", [data], fn );
$( el ).on( "/login", function( event ){...} );

// Dojo: dojo.subscribe( "channel", fn);
var handle = dojo.subscribe( "/login", function(data){..} );

// YUI: el.on("channel", handler);
el.on( "/login", function( data ){...} );

// 取消订阅

// jQuery: $(obj).off( "channel" );
$( el ).off( "/login" );

// Dojo: dojo.unsubscribe( handle );
dojo.unsubscribe( handle );

// YUI: el.detach("channel");
el.detach( "/login" );
```

对于想要在vanilla Javascript(或者其它库)中使用发布/订阅模式的人来讲， AmplifyJS 包含了一个干净的，库无关的实现，可以和任何库或者工具箱一起使用。Radio.js, PubSubJS 或者 Pure JS PubSub 来自于 Peter Higgins 都有类似的替代品值得研究。

尤其对于jQuery 开发者来讲，他们拥有很多其它的选择，可以选择大量的良好实现的代码，从Peter Higgins 的jQuery插件到Ben Alman 在GitHub 上的（优化的）发布/订阅 jQuery gist。下面给出了这些代码的链接。

- [Ben Alman的发布/订阅 gist(推荐)](https://gist.github.com/661855)
- [Rick Waldron 在上面基础上修改的 jQuery-core 风格的实现](https://gist.github.com/705311)
- [Peter Higgins 的插件](http://github.com/phiggins42/bloody-jquery-plugins/blob/master/pubsub.js)
- [AppendTo 在AmplifyJS中的 发布/订阅实现](http://amplifyjs.com/)
- [Ben Truyman的 gist](https://gist.github.com/826794)

从上面我们可以看到在javascript中有这么多种观察者模式的实现，让我们看一下最小的一个版本的发布/订阅模式实现，这个实现我放在github 上，叫做pubsubz。这个实现展示了发布，订阅的核心概念，以及如何取消订阅。

我之所以选择这个代码作为我们例子的基础，是因为这个代码紧密贴合了方法签名和实现方式，这种实现方式正是我想看到的javascript版本的经典的观察者模式所应该有的样子。

**发布/订阅实例**

```
var pubsub = {};

(function(q) {

    var topics = {},
        subUid = -1;

    // Publish or broadcast events of interest
    // with a specific topic name and arguments
    // such as the data to pass along
    q.publish = function( topic, args ) {

        if ( !topics[topic] ) {
            return false;
        }

        var subscribers = topics[topic],
            len = subscribers ? subscribers.length : 0;

        while (len--) {
            subscribers[len].func( topic, args );
        }

        return this;
    };

    // Subscribe to events of interest
    // with a specific topic name and a
    // callback function, to be executed
    // when the topic/event is observed
    q.subscribe = function( topic, func ) {

        if (!topics[topic]) {
            topics[topic] = [];
        }

        var token = ( ++subUid ).toString();
        topics[topic].push({
            token: token,
            func: func
        });
        return token;
    };

    // Unsubscribe from a specific
    // topic, based on a tokenized reference
    // to the subscription
    q.unsubscribe = function( token ) {
        for ( var m in topics ) {
            if ( topics[m] ) {
                for ( var i = 0, j = topics[m].length; i < j; i++ ) {
                    if ( topics[m][i].token === token) {
                        topics[m].splice( i, 1 );
                        return token;
                    }
                }
            }
        }
        return this;
    };
}( pubsub ));
```

示例:使用我们的实现

我们现在可以使用发布实例和订阅感兴趣的事件，例如:

```
// Another simple message handler

// A simple message logger that logs any topics and data received through our
// subscriber
var messageLogger = function ( topics, data ) {
    console.log( "Logging: " + topics + ": " + data );
};

// Subscribers listen for topics they have subscribed to and
// invoke a callback function (e.g messageLogger) once a new
// notification is broadcast on that topic
var subscription = pubsub.subscribe( "inbox/newMessage", messageLogger );

// Publishers are in charge of publishing topics or notifications of
// interest to the application. e.g:

pubsub.publish( "inbox/newMessage", "hello world!" );

// or
pubsub.publish( "inbox/newMessage", ["test", "a", "b", "c"] );

// or
pubsub.publish( "inbox/newMessage", {
  sender: "hello@google.com",
  body: "Hey again!"
});

// We cab also unsubscribe if we no longer wish for our subscribers
// to be notified
// pubsub.unsubscribe( subscription );

// Once unsubscribed, this for example won't result in our
// messageLogger being executed as the subscriber is
// no longer listening
pubsub.publish( "inbox/newMessage", "Hello! are you still there?" );
```

例如：用户界面通知

接下来，让我们想象一下，我们有一个Web应用程序，负责显示实时股票信息。

应用程序可能有一个表格显示股票统计数据和一个计数器显示的最后更新点。当数据模型发生变化时，应用程序将需要更新表格和计数器。在这种情况下，我们的主题（这将发布主题/通知）是数据模型以及我们的订阅者是表格和计数器。

当我们的订阅者收到通知：该模型本身已经改变，他们自己可以进行相应的更新。

在我们的实现中，如果发现新的股票信息是可用的，我们的订阅者将收听到的主题“新数据可用”。如果一个新的通知发布到该主题，那将触发表格去添加一个包含此信息的新行。它也将更新最后更新计数器，记录最后一次添加的数据

```
// Return the current local time to be used in our UI later
getCurrentTime = function (){

   var date = new Date(),
         m = date.getMonth() + 1,
         d = date.getDate(),
         y = date.getFullYear(),
         t = date.toLocaleTimeString().toLowerCase();

        return (m + "/" + d + "/" + y + " " + t);
};

// Add a new row of data to our fictional grid component
function addGridRow( data ) {

   // ui.grid.addRow( data );
   console.log( "updated grid component with:" + data );

}

// Update our fictional grid to show the time it was last
// updated
function updateCounter( data ) {

   // ui.grid.updateLastChanged( getCurrentTime() );  
   console.log( "data last updated at: " + getCurrentTime() + " with " + data);

}

// Update the grid using the data passed to our subscribers
gridUpdate = function( topic, data ){

  if ( data !== "undefined" ) {
     addGridRow( data );
     updateCounter( data );
   }

};

// Create a subscription to the newDataAvailable topic
var subscriber = pubsub.subscribe( "newDataAvailable", gridUpdate );

// The following represents updates to our data layer. This could be
// powered by ajax requests which broadcast that new data is available
// to the rest of the application.

// Publish changes to the gridUpdated topic representing new entries
pubsub.publish( "newDataAvailable", {
  summary: "Apple made $5 billion",
  identifier: "APPL",
  stockPrice: 570.91
});

pubsub.publish( "newDataAvailable", {
  summary: "Microsoft made $20 million",
  identifier: "MSFT",
  stockPrice: 30.85
});
```

样例：在下面这个电影评分的例子里面，我们使用Ben Alman的发布/订阅实现来解耦应用程序。我们使用Ben Alman的jQuery实现，来展示如何解耦用户界面。请注意，我们如何做到提交一个评分，来产生一个发布信息，这个信息表明了当前新的用户和评分数据可用。

剩余的工作留给订阅者，由订阅者来代理这些主题中的数据发生的变化。在我们的例子中，我们将新的数据压入到现存的数组中，接着使用Underscore库的template()方法来渲染模板。

HTML/模板

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

JavaScript


```
;(function( $ ) {

  // Pre-compile templates and "cache" them using closure
  var
    userTemplate = _.template($( "#userTemplate" ).html()),
    ratingsTemplate = _.template($( "#ratingsTemplate" ).html());

  // Subscribe to the new user topic, which adds a user
  // to a list of users who have submitted reviews
  $.subscribe( "/new/user", function( e, data ){

    if( data ){

      $('#users').append( userTemplate( data ));

    }

  });

  // Subscribe to the new rating topic. This is composed of a title and
  // rating. New ratings are appended to a running list of added user
  // ratings.
  $.subscribe( "/new/rating", function( e, data ){

    var compiledTemplate;

    if( data ){

      $( "#ratings" ).append( ratingsTemplate( data );

    }

  });

  // Handler for adding a new user
  $("#add").on("click", function( e ) {

    e.preventDefault();

    var strUser = $("#twitter_handle").val(),
       strMovie = $("#movie_seen").val(),
       strRating = $("#movie_rating").val();

    // Inform the application a new user is available
    $.publish( "/new/user",  { name: strUser } );

    // Inform the app a new rating is available
    $.publish( "/new/rating",  { title: strMovie, rating: strRating} );

    });

})( jQuery );
```

样例：解耦一个基于Ajax的jQuery应用。

在我们最后的例子中，我们将从实用的角度来看一下如何在开发早起使用发布/订阅模式来解耦代码，这样可以帮助我们避免之后痛苦的重构过程。

在Ajax重度依赖的应用里面，我们常会见到这种情况，当我们收到一个请求的响应之后，我们希望能够完成不仅仅一个特定的操作。我们可以简单的将所有请求后的逻辑加入到成功的回调函数里面，但是这样做有一些问题。

高度耦合的应用优势会增加重用功能的代价，因为高度耦合增加了内部函数/代码的依赖性。这意味着如果我们只是希望获取一次性获取结果集，可以将请求后 的逻辑代码 硬编码在回调函数里面，这种方式可以正常工作，但是当我们想要对相同的数据源(不同的最终行为)做更多的Ajax调用的时候，这种方式就不适合了，我们必须要多次重写部分代码。与其回溯调用相同数据源的每一层，然后在将它们泛化，不如一开始就使用发布/订阅模式来节约时间。

使用观察者，我们可以简单的将整个应用范围的通知进行隔离，针对不同的事件，我们可以把这种隔离做到我们想要的粒度上，如果使用其它模式，则可能不会有这么优雅的实现。

注意我们下面的例子中，当用户表明他们想要做一次搜索查询的时候，一个话题通知就会生成，而当请求返回，并且实际的数据可用的时候，又会生成另外一个通知。而如何使用这些事件（或者返回的数据），都是由订阅者自己决定的。这样做的好处是，如果我们想要，我们可以有10个不同的订阅者，以不同的方式使用返回的数据，而对于Ajax层来讲，它不会关心你如何处理数据。它唯一的责任就是请求和返回数据，接着将数据发送给所有想要使用数据的地方。这种相关性上的隔离可以是我们整个代码设计更为清晰。

HTML/Templates


```
<form id="flickrSearch">

   <input type="text" name="tag" id="query"/>

   <input type="submit" name="submit" value="submit"/>

</form>

<div id="lastQuery"></div>

<div id="searchResults"></div>

<script id="resultTemplate" type="text/html">
    <% _.each(items, function( item ){  %>
            <li><p><img src="<%= item.media.m %>"/></p></li>
    <% });%>
</script>
```

JavaScript

```
;(function( $ ) {

   // Pre-compile template and "cache" it using closure
   var resultTemplate = _.template($( "#resultTemplate" ).html());

   // Subscribe to the new search tags topic
   $.subscribe( "/search/tags" , function( tags ) {
       $( "#searchResults" )
                .html("
<p>
    Searched for:<strong>" + tags + "</strong>
</p>
");
   });

   // Subscribe to the new results topic
   $.subscribe( "/search/resultSet" , function( results ){

       $( "#searchResults" ).append(resultTemplate( results ));

   });

   // Submit a search query and publish tags on the /search/tags topic
   $( "#flickrSearch" ).submit( function( e ) {

       e.preventDefault();
       var tags = $(this).find( "#query").val();

       if ( !tags ){
        return;
       }

       $.publish( "/search/tags" , [ $.trim(tags) ]);

   });

   // Subscribe to new tags being published and perform
   // a search query using them. Once data has returned
   // publish this data for the rest of the application
   // to consume

   $.subscribe("/search/tags", function( tags ) {

       $.getJSON( "http://api.flickr.com/services/feeds/photos_public.gne?jsoncallback=?" ,{
              tags: tags,
              tagmode: "any",
              format: "json"
            },

          function( data ){

              if( !data.items.length ) {
                return;
              }

              $.publish( "/search/resultSet" , data.items  );
       });

   });

})();
```

观察者模式在应用设计中，解耦一系列不同的场景上非常有用，如果你没有用过它，我推荐你尝试一下今天提到的之前写到的某个实现。这个模式是一个易于学习的模式，同时也是一个威力巨大的模式。


### 中介者模式

字典中中介者的定义是，一个中立方，在谈判和冲突解决过程中起辅助作用。在我们的世界，一个中介者是一个行为设计模式，使我们可以导出统一的接口，这样系统不同部分就可以彼此通信。

如果系统组件之间存在大量的直接关系，就可能是时候，使用一个中心的控制点，来让不同的组件通过它来通信。中介者通过将组件之间显式的直接的引用替换成通过中心点来交互的方式，来做到松耦合。这样可以帮助我们解耦，和改善组件的重用性。

在现实世界中，类似的系统就是，飞行控制系统。一个航站塔（中介者）处理哪个飞机可以起飞，哪个可以着陆，因为所有的通信（监听的通知或者广播的通知）都是飞机和控制塔之间进行的，而不是飞机和飞机之间进行的。一个中央集权的控制中心是这个系统成功的关键，也正是中介者在软件设计领域中所扮演的角色。

从实现角度来讲，中介者模式是观察者模式中的共享被观察者对象。在这个系统中的对象之间直接的发布/订阅关系被牺牲掉了，取而代之的是维护一个通信的中心节点。

也可以认为是一种补充-用于应用级别的通知，例如不同子系统之间的通信，子系统本身很复杂，可能需要使用发布/订阅模式来做内部组件之间的解耦。

另外一个类似的例子是DOM的事件冒泡机制，以及事件代理机制。如果系统中所有的订阅者都是对文档订阅，而不是对独立的节点订阅，那么文档就充当一个中介者的角色。DOM的这种做法，不是将事件绑定到独立节点上，而是用一个更高级别的对象负责通知订阅者关于交互事件的信息。

**基础的实现**

中间人模式的一种简单的实现可以在下面找到,publish()和subscribe()方法都被暴露出来使用:

```
var mediator = (function(){

    // Storage for topics that can be broadcast or listened to
    var topics = {};

    // Subscribe to a topic, supply a callback to be executed
    // when that topic is broadcast to
    var subscribe = function( topic, fn ){

        if ( !topics[topic] ){
          topics[topic] = [];
        }

        topics[topic].push( { context: this, callback: fn } );

        return this;
    };

    // Publish/broadcast an event to the rest of the application
    var publish = function( topic ){

        var args;

        if ( !topics[topic] ){
          return false;
        }

        args = Array.prototype.slice.call( arguments, 1 );
        for ( var i = 0, l = topics[topic].length; i < l; i++ ) {

            var subscription = topics[topic][i];
            subscription.callback.apply( subscription.context, args );
        }
        return this;
    };

    return {
        publish: publish,
        subscribe: subscribe,
        installTo: function( obj ){
            obj.subscribe = subscribe;
            obj.publish = publish;
        }
    };

}());
```

**高级的实现**

对于那些对更加高级实现感兴趣的人,以走读的方式看一看以下我对Jack Lawson优秀的Mediator.js重写的一个缩略版本.在其它方面的改进当中,为我们的中间人支持主题命名空间,用户拆卸和一个更加稳定的发布/订阅系统。但是如果你想跳过这个走读，你可以直接进入到下一个例子继续阅读。

得感谢Jack优秀的代码注释对这部分内容的协助。

首先，让我们实现认购的概念，我们可以考虑一个中间人主题的注册。

通过生成对象实体,我们稍后能够简单的更新认购,而不需要去取消注册然后重新注册它们.认购可以写成一个使用被称作一个选项对象或者一个上下文环境的函数

```
// Pass in a context to attach our Mediator to.
// By default this will be the window object
(function( root ){

  function guidGenerator() { /*..*/}

  // Our Subscriber constructor
  function Subscriber( fn, options, context ){

    if ( !(this instanceof Subscriber) ) {

      return new Subscriber( fn, context, options );

    }else{

      // guidGenerator() is a function that generates
      // GUIDs for instances of our Mediators Subscribers so
      // we can easily reference them later on. We're going
      // to skip its implementation for brevity

      this.id = guidGenerator();
      this.fn = fn;
      this.options = options;
      this.context = context;
      this.topic = null;

    }
  }
})();
```

在我们的中间人主题中包涵了一长串的回调和子主题,当中间人发布在我们中间人实体上被调用的时候被启动.它也包含操作数据列表的方法

```
// Let's model the Topic.
// JavaScript lets us use a Function object as a
// conjunction of a prototype for use with the new
// object and a constructor function to be invoked.
function Topic(namespace) {

    if (!(this instanceof Topic)) {
        return new Topic(namespace);
    } else {

        this.namespace = namespace || "";
        this._callbacks = [];
        this._topics = [];
        this.stopped = false;

    }
}

// Define the prototype for our topic, including ways to
// add new subscribers or retrieve existing ones.
Topic.prototype = {

    // Add a new subscriber
    AddSubscriber: function (fn, options, context) {

        var callback = new Subscriber(fn, options, context);

        this._callbacks.push(callback);

        callback.topic = this;

        return callback;
    },
    //我们的主题实体被当做中间人调用的一个参数被传递.使用一个方便实用的calledStopPropagation()方法,回调就可以进一步被传播开来:

    StopPropagation: function () {
        this.stopped = true;
    },
    //我们也能够使得当提供一个GUID的标识符的时候检索订购用户更加容易:

    GetSubscriber: function (identifier) {

        for (var x = 0, y = this._callbacks.length; x < y; x++) {
            if (this._callbacks[x].id == identifier || this._callbacks[x].fn == identifier) {
                return this._callbacks[x];
            }
        }

        for (var z in this._topics) {
            if (this._topics.hasOwnProperty(z)) {
                var sub = this._topics[z].GetSubscriber(identifier);
                if (sub !== undefined) {
                    return sub;
                }
            }
        }

    },
    //接着,在我们需要它们的情况下,我们也能够提供添加新主题,检查现有的主题或者检索主题的简单方法:

    AddTopic: function (topic) {
        this._topics[topic] = new Topic((this.namespace ? this.namespace + ":" : "") + topic);
    },

    HasTopic: function (topic) {
        return this._topics.hasOwnProperty(topic);
    },

    ReturnTopic: function (topic) {
        return this._topics[topic];
    },
    //如果我们觉得不再需要它们了,我们也可以明确的删除这些订购用户.下面就是通过它的其子主题递归删除订购用户的代码:

    RemoveSubscriber: function (identifier) {

        if (!identifier) {
            this._callbacks = [];

            for (var z in this._topics) {
                if (this._topics.hasOwnProperty(z)) {
                    this._topics[z].RemoveSubscriber(identifier);
                }
            }
        }

        for (var y = 0, x = this._callbacks.length; y < x; y++) {
            if (this._callbacks[y].fn == identifier || this._callbacks[y].id == identifier) {
                this._callbacks[y].topic = null;
                this._callbacks.splice(y, 1);
                x--; y--;
            }
        }

    },
    //接着我们通过递归子主题将发布任意参数的能够包含到订购服务对象中:

    Publish: function (data) {

        for (var y = 0, x = this._callbacks.length; y < x; y++) {

            var callback = this._callbacks[y], l;
            callback.fn.apply(callback.context, data);

            l = this._callbacks.length;

            if (l < x) {
                y--;
                x = l;
            }
        }

        for (var x in this._topics) {
            if (!this.stopped) {
                if (this._topics.hasOwnProperty(x)) {
                    this._topics[x].Publish(data);
                }
            }
        }

        this.stopped = false;
    }
};
//接着我们暴露我们将主要交互的调节实体.这里它是通过注册的并且从主题中删除的事件来实现的

function Mediator() {

    if (!(this instanceof Mediator)) {
        return new Mediator();
    } else {
        this._topics = new Topic("");
    }

};
//想要更多先进的用例,我们可以看看调解支持的主题命名空间,下面这样的asinbox:messages:new:read.GetTopic 返回基于一个命名空间的主题实体。

Mediator.prototype = {

    GetTopic: function (namespace) {
        var topic = this._topics,
            namespaceHierarchy = namespace.split(":");

        if (namespace === "") {
            return topic;
        }

        if (namespaceHierarchy.length > 0) {
            for (var i = 0, j = namespaceHierarchy.length; i < j; i++) {

                if (!topic.HasTopic(namespaceHierarchy[i])) {
                    topic.AddTopic(namespaceHierarchy[i]);
                }

                topic = topic.ReturnTopic(namespaceHierarchy[i]);
            }
        }

        return topic;
    },
    //这一节我们定义了一个Mediator.Subscribe方法，它接受一个主题命名空间,一个将要被执行的函数,选项和又一个在订阅中调用函数的上下文环境.这样就创建了一个主题,如果这样的一个主题存在的话

    Subscribe: function (topiclName, fn, options, context) {
        var options = options || {},
            context = context || {},
            topic = this.GetTopic(topicName),
            sub = topic.AddSubscriber(fn, options, context);

        return sub;
    },
    //根据这一点,我们可以进一步定义能够访问特定订阅用户,或者将他们从主题中递归删除的工具

    // Returns a subscriber for a given subscriber id / named function and topic namespace

    GetSubscriber: function (identifier, topic) {
        return this.GetTopic(topic || "").GetSubscriber(identifier);
    },

    // Remove a subscriber from a given topic namespace recursively based on
    // a provided subscriber id or named function.

    Remove: function (topicName, identifier) {
        this.GetTopic(topicName).RemoveSubscriber(identifier);
    },
    //我们主要的发布方式可以让我们随意发布数据到选定的主题命名空间，这可以在下面的代码中看到。

    //主题可以被向下递归.例如,一条对inbox:message的post将发送到inbox:message:new和inbox:message:new:read.它将像接下来这样被使用:Mediator.Publish( "inbox:messages:new", [args] );

    Publish: function (topicName) {
        var args = Array.prototype.slice.call(arguments, 1),
            topic = this.GetTopic(topicName);

        args.push(topic);

        this.GetTopic(topicName).Publish(args);
    }
};
//最后，我们可以很容易的暴露我们的中间人，将它附着在传递到根中的对象上：

root.Mediator = Mediator;
Mediator.Topic = Topic;
Mediator.Subscriber = Subscriber;

// Remember we can pass anything in here. I've passed inwindowto
// attach the Mediator to, but we can just as easily attach it to another
// object if desired.
```

示例

无论是使用来自上面的实现（简单的选项和更加先进的选项都是），我们能够像下面这样将一个简单的聊天记录系统整到一起:

HTML

```
<h1>Chat</h1>
<form id="chatForm">
    <label for="fromBox">Your Name:</label>
    <input id="fromBox" type="text"/>
    <br />
    <label for="toBox">Send to:</label>
    <input id="toBox" type="text"/>
    <br />
    <label for="chatBox">Message:</label>
    <input id="chatBox" type="text"/>
    <button type="submit">Chat</button>
</form>

<div id="chatResult"></div>
```

Javascript

```
$( "#chatForm" ).on( "submit", function(e) {
    e.preventDefault();

    // Collect the details of the chat from our UI
    var text = $( "#chatBox" ).val(),
        from = $( "#fromBox" ).val(),
        to = $( "#toBox" ).val();

    // Publish data from the chat to the newMessage topic
    mediator.publish( "newMessage" , { message: text, from: from, to: to } );
});

// Append new messages as they come through
function displayChat( data ) {
    var date = new Date(),
        msg = data.from + " said \"" + data.message + "\" to " + data.to;

    $( "#chatResult" )
        .prepend("
<p>
    " + msg + " (" + date.toLocaleTimeString() + ")
</p>
");
}

// Log messages
function logChat( data ) {
    if ( window.console ) {
        console.log( data );
    }
}

// Subscribe to new chat messages being submitted
// via the mediator
mediator.subscribe( "newMessage", displayChat );
mediator.subscribe( "newMessage", logChat );

// The following will however only work with the more advanced implementation:

function amITalkingToMyself( data ) {
    return data.from === data.to;
}

function iAmClearlyCrazy( data ) {
    $( "#chatResult" ).prepend("
<p>
    " + data.from + " is talking to himself.
</p>
");
}

mediator.Subscribe( amITalkingToMyself, iAmClearlyCrazy );
```

**优点&缺点**

中间人模式最大的好处就是，它节约了对象或者组件之间的通信信道，这些对象或者组件存在于从多对多到多对一的系统之中。由于解耦合水平的因素，添加新的发布或者订阅者是相对容易的。

也许使用这个模式最大的缺点是它可以引入一个单点故障。在模块之间放置一个中间人也可能会造成性能损失，因为它们经常是间接地的进行通信的。由于松耦合的特性，仅仅盯着广播很难去确认系统是如何做出反应的。

这就是说，提醒我们自己解耦合的系统拥有许多其它的好处，是很有用的——如果我们的模块互相之间直接的进行通信，对于模块的改变（例如：另一个模块抛出了异常）可以很容易的对我们系统的其它部分产生多米诺连锁效应。这个问题在解耦合的系统中很少需要被考虑到。

在一天结束的时候，紧耦合会导致各种头痛，这仅仅只是另外一种可选的解决方案，但是如果得到正确实现的话也能够工作得很好。

**中间人VS观察者**

开发人员往往不知道中间人模式和观察者模式之间的区别。不可否认，这两种模式之间有一点点重叠，但让我们回过头来重新寻求GoF的一种解释：

“在观察者模式中，没有封装约束的单一对象”。取而代之，观察者和主题必须合作来维护约束。通信的模式决定于观察者和主题相互关联的方式：一个单独的主题经常有许多的观察者，而有时候一个主题的观察者是另外一个观察者的主题。“

中间人和观察者都提倡松耦合，然而，中间人默认使用让对象严格通过中间人进行通信的方式实现松耦合。观察者模式则创建了观察者对象，这些观察者对象会发布触发对象认购的感兴趣的事件。

**中间人VS门面**

不久我们的描述就将涵盖门面模式，但作为参考之用，一些开发者也想知道中间人和门面模式之间有哪些相似之处。它们都对模块的功能进行抽象，但有一些细微的差别。

中间人模式让模块之间集中进行通信，它会被这些模块明确的引用。门面模式却只是为模块或者系统定义一个更加简单的接口，但不添加任何额外的功能。系统中其他的模块并不直接意识到门面的概念，而可以被认为是单向的。