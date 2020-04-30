---
title: "深入理解javascript原型继承"
date: 2020-04-30T18:39:53+08:00
---

作为一门被长期误解的编程语言, javascript一直被人所诟病. 但是如果你真正的了解它之后, 你会深深的爱上它. 

首先, javascript是一个面向对象的编程语言, 而且是一个纯粹的面向对象. 虽然很多人不能理解, 因为在他们眼中, 只有像java, c++这样的编程语言才能称之为面向对象. 但是, 我只想说, 你误解我, 是因为你不懂我. 

> [JavaScrip秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/#intro)在JavaScrip中, 一切变量皆对象, 除了两个特殊值undefined 和 null. 

**什么是面向对象？**

* 一切事物皆对象
* 对象具有封装和继承特性
* 对象与对象之间使用消息通信，各自存在信息隐藏

  

以这三点做为依据，C++ 是半面向对象半面向过程语言，因为，虽然他实现了类的封装、继承和多态，但存在非对象性质的全局函数和变量。Java、C# 是完全的面向对象语言，它们通过类的形式组织函数和变量，使之不能脱离对象存在。但这里函数本身是一个过程，只是依附在某个类上。

然而，面向对象仅仅是一个概念或者编程思想而已，它不应该依赖于某个语言存在。比如 Java 采用面向对象思想构造其语言，它实现了类、继承、派生、多态、接口等机制。但是这些机制，只是实现面向对象编程的一种手段，而非必须。换言之，一门语言可以根据其自身特性选择合适的方式来实现面向对象。所以，由于大多数程序员首先学习或者使用的是类似 Java、C++ 等高级编译型语言（Java 虽然是半编译半解释，但一般做为编译型来讲解），因而先入为主地接受了“类”这个面向对象实现方式，从而在学习脚本语言的时候，习惯性地用类式面向对象语言中的概念来判断该语言是否是面向对象语言，或者是否具备面向对象特性。这也是阻碍程序员深入学习并掌握 JavaScript 的重要原因之一。

实际上，JavaScript语言是通过一种叫做原型（prototype）的方式来实现面向对象编程的。它和其他的面向对象类编程语言一样，只是它的实现方式不同而已，或者说他们采用了不同的面向对象设计哲学。

在基于类的面向对象方式中，对象（object）依靠类（class）来产生。而在基于原型的面向对象方式中，对象（object）则是依靠 构造器（constructor）利用 原型（prototype）构造出来的。

举个客观世界的例子来说明二种方式认知的差异。例如工厂造一辆车，一方面，工人必须参照一张工程图纸，设计规定这辆车应该如何制造。这里的工程图纸就好比是语言中的 类 (class)，而车就是按照这个 类（class）制造出来的；另一方面，工人和机器 ( 相当于 constructor)利用各种零部件如发动机，轮胎，方向盘 ( 相当于 prototype 的各个属性 ) 将汽车构造出来。

在这里我不想讨论太多这两种面向对象究竟孰优孰劣的问题, 这个讨论来讨论去也不会有定论. 但是以为个人观点我更倾向于基于原型的面向对象. 因为一切事物皆对象. 现实世界中的所有对象也都是由其他对象构造出来的, 而紧紧依靠图纸是没法产生一个现实中的汽车的. 这个设计更符合现实世界的客观规律. 

JavaScript的面向对象来源于‘self’这个牛逼但短命的编程语言。

> Self语言把概念上的精简作为设计原则。它取消了类的概念，只有对象的概念，同时把消息作为最基本的操作。把对象的属性理解为获取或更改属性这两种方法，从而把属性的概念简化为方法；取消了变量和赋值，并以通过消息来读槽和写槽的方式代之。

> 在 JavaScript 中，prototype 是函数的一个属性，同时也是由构造函数创建的对象的一个属性。 函数的原型为对象。 它主要在函数用作构造函数时使用。

在编程语言中, 目前存在两种继承方式: 类继承和原型继承. 

* 类继承
* 原型继承

而由于java等主流编程语言的支持使得类继承被大众所普遍接受

## 那么, JavaScript中是如何实现的基于原型的面向对象?

要理解原型继承, 首先得先熟悉几个概念, 咱们一步步说起:

### 如何生成对象?

-1. 声明对象直接量: JSON  

``` js
var obj = {
    name: "jack",
    eat: "bread"
}
console.log(typeof obj);
```

-2. 使用构造函数生成一个新的对象

``` js
    //构造函数
    var Foo = function(name) {
        this.name = name; //私有属性    
    }

    //原型方法和属性,被继承时候才会调用
    Foo.prototype.run = function() {
        alert("I'm running so fast that can't stop at all!");
    }

    var kick = new Foo("kick");
    console.log(typeof kick);
    console.log(kick.name);
    kick.run();
```

-3. 使用使用Object. create创建对象
ECMAScript 5中引入了一个新方法: Object. create. 可以调用这个方法来创建一个新对象. 新对象的原型就是调用create方法时传入的第一个参数:

> 先来看一下create方法是如何实现的, 该方法来源于Douglas Crockford, 现在已被ECMAScript 5引入:

``` js
    Object.create = function(parent) {
        function F() {}
        F.prototype = parent;
        return new F();
    };
```

这个看起来很简洁, 而且能够完全代替new的用法, 毕竟new关键字并不真正的属于JavaScrip的原型模式. 它先是声明了一个构造器, 然后将其原型设置为你想要的值, 最后返回生成的新对象. 其实就是封装了new. 

下面这段代码就是真正的原型继承了. look:

``` js
    var Point = {
        x: 0,
        y: 0,
        print: function() {
            console.log(this.x, this.y);
        }
    };
    var p = Object.create(Point); //new一个对象
    p.x = 10;
    p.y = 20;
    p.print(); // 10 20
```

code:

``` js
    function Plant(name, year) {
        this.name = name;
        this.year = year || 0;
    }
    var tree.prototype = new Plant('tree');
    tree.prototype.grow = function() {
        this.year++;
    }
    tree.prototype.old = functiono() {
        console.log(this.year);
    }
```

上面这段代码使用原型实现了一个简单的对象继承. 下面来分析下上面这段代码
首先是声明了一个构造函数, 构造函数和普通函数有什么区别? 构造函数可以使用new调用, 生成一个新的对象. 
如果想要在对象上添加方法, 可以将方法写在对象的原型上. 
子类继承父类, 只需要把父对象复制给自对象的原型上即可. 

## JavaScrip原型链(prototype chain)

**下面这段是ECMAScript关于原型的解释**

> ECMAScript does not contain proper classes such as those in C++, Smalltalk, or Java, but rather, supports constructors which create objects by executing code that allocates storage for the objects and initialises all or part of them by assigning initial values to their properties. All constructors are objects, but not all objects are constructors. Each constructor has a Prototype property that is used to implement prototype-based inheritance and shared properties. Objects are created by using constructors in new expressions; for example, new String("A String") creates a new String object. Invoking a constructor without using new has consequences that depend on the constructor. For example, String("A String") produces a primitive string, not an object. 

> ECMAScript supports prototype-based inheritance. Every constructor has an associated prototype, and every object created by that constructor has an implicit reference to the prototype (called the object's prototype) associated with its constructor. Furthermore, a prototype may have a non-null implicit reference to its prototype, and so on; this is called the prototype chain. When a reference is made to a property in an object, that reference is to the property of that name in the first object in the prototype chain that contains a property of that name. In other words, first the object mentioned directly is examined for such a property; if that object contains the named property, that is the property to which the reference refers; if that object does not contain the named property, the prototype for that object is examined next; and so on. 

依据我的理解就是说:  
JavaScrip可以采用构造器(constructor)生成一个新的对象, 每个构造器都拥有一个prototype属性, 而每个通过此构造器生成的对象都有一个指向该构造器原型(prototype)的内部私有的链接(__proto__), 而这个prototype因为是个对象, 它也拥有自己的原型, 这么一级一级指导原型为null, 这就构成了原型链. 

**这里我们涉及到了一个隐匿属性__proto__, 那么__proto__和prototype究竟有什么区别嘞?**
**注:** __proto__ 是一个不应在你代码中出现的非正规的用法，这里仅仅用它来解释JavaScript原型继承的工作原理。

知道了JavaScrip原型链的存在之后, 让我们来看下它的实现, 下面这段代码展示了原型链是如何工作的. 

``` js
    function getProperty(obj, prop) {
        if (obj.hasOwnProperty(prop)) //首先查找自身属性,如果有则直接返回
            return obj[prop]
        else if (obj.__proto__ !== null)
            return getProperty(obj.__proto__, prop) //如何不是私有属性,就在原型链上一步步向上查找,直到找到,如果找不到就返回undefind
        else
            return undefined
    }
```

So, 如果__proto__可以使用的话, 我们可以通过下面这种方式实现继承:

``` js
    var person = {
        city: "Beijing",
        hate: function() {
            alert("I really hate the PM2.5 and the foggy wether!");
        }
    }
    var lee = {
        name: "lee",
        age: "18",
        __proto__: person
    }
    console.log(lee);
    lee.hate();
```

这都什么玩意儿, 不是要用new吗. 事实上, 事情不是这么简单滴, 为了和主流的类继承扯上那么一点儿关系, JavaScrip引入了'new'关键字, 引入了构造函数. 所以通常我们看到的是下面这样的:

``` js
    var Person = function(name, age) {
        this.name = name;
        this.age = age;
    };
    Person.prototype = {
        city: "Beijing",
        hate: function() {
            alert("I really hate the PM2.5 and the foggy wether!");
        }
    }
    var lee = new Person('lee', 18);
    console.log(lee.name);
    lee.hate();
```

我们需要一个像类一样的东西, 于是有了构造函数, 我们得有一个通过类生成实例的过程, 于是又出现了new. 这么一来JavaScrip的原型继承似乎就变得不伦不类了. 虽然JavaScrip的原型继承来源于'self', 但是却追随了类继承的形式. 罪过, 不过话说回来, 也许就是因为这种妥协才让JavaScrip能够流行起来, 并成为了现在最流行的原型继承语言, 而self, 说实话, 它独特写法确实挺难让人接受的. 

``` js
    var Foo = function() {
        this.name = "foo";
    }
    Foo.prototype.say = function() {
        alert("Hello World!");
    }
    var foo = new Foo();
    console.log(foo.__proto__); //私有链接,指向构造函数的原型
    console.log(Foo.prototype);
    console.log(foo.__proto__ === Foo.prototype); //true
    console.log(foo.__proto__.constructor === Foo); //true

    // 声明 Animal 对象构造器
    function Animal(name) {
        this.name = name;
    }
    // 将 Animal 的 prototype 属性指向一个对象，
    // 亦可直接理解为指定 Animal 对象的原型
    Animal.prototype = {
        weight: 0,
        eat: function() {
            alert("Animal is eating!");
        }
    }
    // 声明 Mammal 对象构造器
    function Mammal() {
        this.name = "mammal";
    }
    // 指定 Mammal 对象的原型为一个 Animal 对象。
    // 实际上此处便是在创建 Mammal 对象和 Animal 对象之间的原型链
    Mammal.prototype = new Animal("animal");
    // 声明 Horse 对象构造器
    function Horse(height, weight) {
        this.name = "horse";
        this.height = height;
        this.weight = weight;
    }
    // 将 Horse 对象的原型指定为一个 Mamal 对象，继续构建 Horse 与 Mammal 之间的原型链
    Horse.prototype = new Mammal();
    // 重新指定 eat 方法 , 此方法将覆盖从 Animal 原型继承过来的 eat 方法
    Horse.prototype.eat = function() {
        alert("Horse is eating grass!");
    }
    // 验证并理解原型链
    var horse = new Horse(100, 300);
    console.log(horse.__proto__ === Horse.prototype);
    console.log(Horse.prototype.__proto__ === Mammal.prototype);
    console.log(Mammal.prototype.__proto__ === Animal.prototype);
    //原型链
    Horse-- > Mammal的实例
    Mammal-- > Animal的实例
    Animal-- > Object.prototype
```

在 ECMAScript 中，每个由构造器创建的对象拥有一个指向构造器 prototype 属性值的 隐式引用（implicit reference），这个引用称之为 原型（prototype）。进一步，每个原型可以拥有指向自己原型的 隐式引用（即该原型的原型），如此下去，这就是所谓的 原型链（prototype chain） [参考资源](http://bclary.com/2004/11/07/#a-4.3.5)。在具体的语言实现中，每个对象都有一个 __proto__ 属性来实现对原型的 隐式引用。

我们已经了解了JS原型继承是什么，以及JS如何用特定的方式来实现之。然而使用真正的原型继承（如 Object. create 以及 __proto__）还是存在以下缺点：

* 标准性差：__proto__ 不是一个标准用法，甚至是一个不赞成使用的用法。同时原生态的 Object. create 和道爷写的原版也不尽相同。
* 优化性差： 不论是原生的还是自定义的 Object. create ，其性能都远没有 new 的优化程度高，前者要比后者慢高达10倍。

到了这里我们基本对JavaScrip的原型继承有了一个更深层的认识了. 通过历史回溯我们也了解了为什么JavaScrip会变成现在这个不伦不类的样子. 
JavaScrip是一个完全的面向对象函数式编程语言, 采用原型继承, 虽然写法类似类继承. 但是我们不能因此就认为它不是面向对象的编程语言. 而且nodejs的出现, 又让JavaScrip在编程语言界火了一把. 所以是时候拥抱JavaScrip了. 
