---
title: "react进阶之flux"
date: 2016-11-10T18:39:53+08:00
---

> 去年翻译的flux官方文档对flux架构的描述，觉得最近很多朋友开始react编程了，所以我觉得有必要拿出来这篇水水的译文了，如果你还是喜欢看英文原版，那么请点[这里](http://facebook.github.io/flux/docs/overview.html)

------------------------------------------------------------------------------

- [Overview](#overview)
- [Structure and Data Flow](#structure-and-data-flow)
- [A Single Dispatcher](#a-single-dispatcher)
- [Stores](#stores)
- [Views and Controller-Views](#views-and-controller-views)
- [Actions](#actions)
- [What About that Dispatcher?](#what-about-that-dispatcher)

### Overview

> 为了方便理解，主要的英文名词都没有翻译。比如：dispatcher（调度者）、 store（仓库）、view（视图）

Flux是Facebook用来构建用户端的web应用的应用程序体系架构。它通过利用数据的单向流动为React的可复用的视图组件提供了补充。相比于形式化的框架它更像是一个架构思想，不需要太多新的代码你就可以马上使用Flux构建你的应用。

Flux应用主要包括三部分：dispatcher、store和views（React components），千万不要和MVC(model-View-Controller)搞混。Controller在Flux应用中也确实存在，但是是以controller-view的形式。view通常处于应用的顶层，它从stores中获取数据，同时将这些数据传递给它的后代节点。另外，action creators - dispatcher辅助方法 - 一个被用来提供描述应用所有可能存在的改变的语义化的API。把它理解为Flux更新闭环的第四个组成部分可以帮助你更好的理解它。

Flux使用单向的数据流动来避开MVC. 当用户与React视图交互的时候，视图会通过中枢dispatcher产生一个action。然后大量的保存着应用数据和业务逻辑的视图接收到冒泡的action，更新所有受影响的view。这种方式很适合React这种声明式的编程方式，因为它的store更新，并不需要特别指定如何在view和state中过渡。

我们独创性的解决了数据的获取：举个栗子，比如我们需要展示一个会话列表，高亮其中未读的会话，同时展示未读会话的数量。如果用MVC架构的话将很难处理这种情况，因为更新一个对话为已读的时候会更新对话model，然后同样也需要更新未读对话数量model（数量-1）。这样的依赖和瀑布式的更新在大型的应用中非常常见，导致错综复杂的数据流动和不可预测的结果。（这其实是Facebook之前的一个线上bug，有时候用户看到提示说有一条未读信息，但是点进去却发现没有）。

反过来让 Store 来控制：store接受更新，并在合适的时机处理这些更新。而不是采用一贯依赖外部的方式来更新数据。在store外部，并没办法看到store内部是如何处理它内部的数据的，这样的方式保证了一个清晰的关注点分离。Store并没有类似setAsRead()这样直接的setter方法，但是在其自成一体的世界中拥有唯一个获取新数据的方法 - store通过dispatcher注册的回调函数。 

### Structure and Data Flow 

在Flux应用中数据是单向流动的：
<figure>
  <img width="650" src="http://facebook.github.io/flux/img/flux-simple-f8-diagram-1300w.png" alt="unidirectional data flow in Flux">
</figure>

单向的数据流是Flux应用的核心特性，上图应该成为Flux程序员的主要心智模型。Dispatcher，stores和views是拥有清晰的输入输出的独立节点。而actions是包含了新的数据和身份属性的简单对象。

用户的交互可能会使views产生新的action，这个action可以向整个系统中传播：

<figure>
  <img width="650" src="http://facebook.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png" alt="unidirectional data flow in Flux">
</figure>

所有的数据的流动都通过中枢dispatcher。Action可以通过_action creator_产生并被提供给dispatcher，但多数情况下action是通过用户与views的交互产生。dispaer接收到action并执行那些已经注册的回调，向所有stores分发action。通过注册的回调，store响应那些与他们所保存的状态有关的action。然后store会触发一个 _change_ 事件，来提醒controller-views数据已经发生了改变。Controller-views监听这些事件并重新从store中获取数据。这些controller-views调用他们自己的 `setState()` 方法，重新渲染自身以及组件树上的所有后代组件。

<figure>
  <img width="650" src="http://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png" alt="unidirectional data flow in Flux">
</figure>

这种的响应式编程，或者更准确的说数据流编程亦或基于数据流的编程，可以使我们很容易去推断我们的应用是如何工作的。因为我们的应用中数据是单项流动的，不存在双向绑定。应用的状态只保存在store中，这就允许应用中不同部分保持高度的低耦合。虽然依赖在store中也确实存在，但他们之间保持着严格的等级制度，并通过dispacher来管理同步更新。

我们发现双向绑定会导致瀑布式的更新，一个对象发生变化会引起另一个对象的改变，并可能导致更多地更新。随着应用的增大，这些瀑布流式的更新方式会使我们很难预测用户交互可能会导致的改变。当更新只能以单一回合进行的时候，系统的可预测性也就会变得更高。

让我们来看看Flux的各个部分。从diapatcher先开始会比较好

### A Single Dispatcher 

dispatcher 就像是一个中央的集线器，管理著所有的数据流。本质上它就是 store callback 的注册表，本身并没有实际的高度功能。它就是一个用来向stores分发actions的机器。 每一个 store 各自注册自己的 callback 以提供对应的处理动作。当 dispatcher 发出一个 action 时，应用中所有的store都会通过注册的callback收到这个action。

随着应用的增长，dispacher会变得更加必不可少，因为它能够指定注册的callback的执行顺序来管理store之间的依赖。store可以被声明等待其他store完成更新之后，再执行更新。

Facebook目前在生产环境中使用的flux可以分别在npm, Bower，or Gihub中获取。

### Stores 

Stores 包含了应用的状态和逻辑，它有点儿像传统MVC中的model层，但是却管理这多个对象的状态 - 他们不像传统的ORM model 只管理单个的数据记录，和backbone中得collection也不一样。

举个栗子，Facebook的回看视频编辑器使用TimeStore来保存播放时间和播放状态。另外，应用中的ImageStore保存着图片的集合。再比如说我们的TodoMVC示例中，TodoStore也类似地管理着to-do items的集合。store典型的特征就是既是models的集合，又是所属业务域下的model实例。

就像上面所说的，store在dispacher中注册，并提供相应地回调。回调会接收action并把它当成自己的一个参数。当action被触发，回调函数会使用swich语句来解析action中的type参数，并在合适的type下提供钩子来执行内部方法。这就允许action通过dispacher来响应store中的state更新。store更新完成之后，会向应用中广播一个change事件，views可以选择响应事件来重新获取新的数据并更新。

### Views and Controller-Views 

React提供了一种可组合式的view让我们可以自由组合展示层。在接近顶层的地方，有些view需要监听所依赖的store的广播事件。我们称之为controller-view，因为他们提供了胶水代码来从store中获取数据，并向下层层传递这些数据。我们会利用这些controller-views来处理页面上某些重要部分。

当它接收到store的广播事件后，它首先会通过store的公共getter方法来获取所需的数据，然后调用自身的setState() 或 forceUpdate()方法来促使自身和后代的重新渲染。

在单一实例中，我们通常会向后代view传递全部数据，而让他们自己从中提取所需数据。此外我们在结构的顶部也维持着类似controller的行为，并且让后代的view保持的function特性。通过向后代传递所有的数据，也有助于减少我们需要管理的props的数量。

偶尔，我们需要在系统的更深层的地方加入controller-views来保持我们的组件的简单。这有助于封装一个特定的数据域下的相关部分。需要注意的是，系统深层的controller-views可能会影响数据的单向流动，因为他们可能会引入一些新的，潜在的存在冲突的数据流入口。在决定是否增加深层的controller-views时，我们需要多方面权衡简单的组件和复杂多样的数据更新流这两点。这些多样的数据更新可能会导致一些古怪的副作用，伴随着不同的controller-views的render调用，潜在的增加了Debug的难度。

### Actions 

dispatcher提供了一个可以允许我们向store中触发分发的方法，我们称之为action。它包含了一个数据的payload。action生成被包含进一个语义化的辅助方法中，来发送action到dispatcher。比如，我们想更新todo应用中一个todo-item的文本内容。我们会在TodoActions模块中生成一个类似updateText(todoId, newText) 这样的函数，这个函数可以被视图事件处理调用执行，因此我们可以通过调用它来响应用户交互。action生成函数同样会增加一个type参数，根据type的不同，store可以做出合适的响应。在我们的例子中，这个type可以叫做TODO_UPDATE_TEXT。

一个payload大概是这个样子的：

```js
{  
  source: "SERVER_ACTION",  
  action: {  
    type: "RECEIVE_RAW_NODES",  
    addition: "some data",  
    rawNodes: rawNodes  
  }  
} 
```

Actions也可能来自其他地方，比如服务器端。这种情况可能会在数据初始化的时候出现，也可能是当服务器视图更新的时候返回了错误的时候出现。

### What About that Dispatcher? 

就像之前提到的那样，dispatcher也可以用来管理store之间的依赖。我们可以通过dispacher的waitFor()方法来实现。在类似TodoMVC这样简单地应用中我们可能用不到这个方法，但是在更大型，更复杂的应用的它会变得不可或缺。

在执行TodoStore的注册回调时，我们可以明确地等待任何依赖先更新，然后再进行后续的处理：

```js
case 'TODO_CREATE':
  Dispatcher.waitFor([
    PrependedTextStore.dispatchToken,
    YetAnotherStore.dispatchToken
  ]);

  TodoStore.create(PrependedTextStore.getText() + ' ' + action.text);
  break;
```

waitFor()的参数是一个包含了dispatcher注册索引的数组，这个索引通常被称之为dispatch tokens。因此store可以使用waitFor()来依赖其他的state，以此来确定如何更新它自己的state。

使用register()方法注册回调的时候会返回一个id，这个id可以用作dispatch token

```js
PrependedTextStore.dispatchToken = Dispatcher.register(function (payload) {
  // ...
});
```

想了解更多关于waitFor(), actions, action creators 以及 dispatcher相关的知识，请参看[Flux: Actions 和 Dispatcher](http://facebook.github.io/flux/docs/actions-and-the-dispatcher.html#content)
