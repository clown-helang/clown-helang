---
title: JavaScript高级程序设计--理解与创建对象
catalog: true
date: 2018-5-21 13:36:54
subtitle: 基础教程
header-img: "/img/article_header/article_header.png"
tags:
- Javascript
---

## 理解对象

创建对象最简单的方式就是创建一个 Object 的实例，然后再为它添加属性和方法，如下所示：

```
var person = new Object();
person.name = "HeLang";
person.age = 24;
person.job = "Software Engineer";

person.sayName = function(){
    alert(this.name);
}
```
早期的JavaScript开发人员经常使用这个模式创建新对象。几年后，**对象字面量**成为创建这种对象的首选模式，如下所示：

```
var person = {
    name: 'HeLang',
    age: 24,
    job: "Software Engineer",
    
    sayName: function(){
        alert(this.name);
    }
}
```


### 属性类型
ECMAScript 中有两种属性：数据属性和访问器属性。

#### 1.数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有4个描述其行为的特性。
-  [[Configurable]]：表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为 true。
    
-  [[Enumerable]]：表示能否通过 for-in 循环返回属性。像前面例子中那样直接在对象上定义的属性，他们的这个特性默认值为 true。
    
-  [[Writable]]：表示能否修改属性的值。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为true。
    
-  [[Value]]：包含这个属性的数据值。读取属性值得时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。这个特性的默认值为 undefined。

要修改属性默认的特性，必须使用 ECMAScript 5 的 Object.defineProperty() 方法。这个方法接收三个参数：属性所在的对象、属性的名字和描述符对象。其中，描述符对象的属性必须是：configurable、enumerable、writable 和 value。设置其中的一或多个值，可以修改对应的特性值。例如：

```
var person = {};
Object.defineProperty(person,name,{
    writable: false,
    value: "HeLang"
});

alert(person.name);       //HeLang
person.name = "Nicholas";
alert(person.name);       //Helang
```
这个例子创建了一个名为 name 的属性，他的值 "HeLang" 是只读的。这个属性的值是不可修改的，如果尝试为它指定新值，则在非严格模式下，赋值操作将被忽略；在严格模式下，赋值操作将会导致抛出错误。

#### 2. 访问器属性

访问器属性不包含数据值；他们包含一对儿 getter 和 setter 函数（不过，这两个函数都不是必须的）。在读取访问器属性时，会调用 getter 函数，这个函数负责返回有效的值；在写入访问器属性时，会调用 setter 函数并传入新值，这个函数负责决定如何处理数据。访问器属性有如下4个特性。
-  [[Configurable]]：表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。对于直接在对象上定义的属性，它们的这个特性默认值为 true。
    
-  [[Enumerable]]：表示能否通过 for-in 循环返回属性。对于直接在对象上定义的属性，他们的这个特性默认值为 true。

-  [[Get]]：在读取属性是调用的函数。默认值为 undefined。

-  [[Set]]：在写入属性时调用的函数。默认值为 undefined。

访问器属性不能直接定义，必须使用Object.defineProperty()来定义。如下：

```
var book = {
    _year: 2018,
    edition: 1
};

Object.defineProperty(book, "year", {
    get: function(){
        return this._year;
    },
    set: function(newValue){
        if(newValue > 2018) {
            this._year = newValue;
            this.edition += newValue - 2018;
        }
    }
});

book.year = 2019;
alert(book.edition);   //2
```

不一定非要同时指定 getter 和 setter。只指定 getter 意味着属性是不能写，尝试写入属性会被忽略。在严格模式下，尝试写入只指定了 getter 函数的属性会抛出错误。类似的，只指定 setter 函数的属性也不能读，否则在非严格模式下会返回 undefined，而在严格模式下会抛出错误。

### 定义多个属性

由于为对象定义多个属性的可能性很大，ECMAScript 5 又定义了一个 Object.defineProperties()方法。利用这个方法可以通过描述符一次定义多个属性。这个方法接收两个对象参数：第一个对象是要添加和修改其属性的对象，第二个对象的属性与第一个对象中要添加或修改的属性一一对应。例如：

```
var book = {};

Object.defineProperties(book, {
    _year: {
        value: 2018
    },
    
    edition:{
        value: 1
    },
    
    year:{
        get: function(){
            return this._year;
        },
        set: function(newValue){
            if(newValue > 2018){
                this._year = newValue;
                this.edition += newValue - 2018;
            }
        }
    }
});

```

### 读取属性的特性

使用ECMAScript 5的 Object.getOwnPropertyDescriptor()方法，可以取得给定属性的描述符。这个方法接收两个参数：属性所在的对象和要读取其描述符的属性名称。返回值是一个对象，如果是访问器属性，这个对象的属性有 configurable、enumberable、get 和 set；如果是数据属性，这个对象的属性有 configuration、enumerable、writable 和 value。例如：(示例中book对象为上文定义)

```
var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
alert(descriptor.value);                //2018
alert(descriptor.configurable);         //false
alert(typeof descriptor.get);           //undefined

var descriptor = Object.getOwnPropertyDescriptor(book, "year");
alert(descriptor.value);                //undefined
alert(descriptor.enumerable);           //false
alert(typeof descriptor.get);           //"function"

```

## 创建对象

虽然 Object 构造函数或对象字面量都可以用来创建单个对象，但这些方式有个明显的缺点：使用同一个接口创建很多个对象，会产生大量的重复代码。为了解决这个问题，人们开始使用工厂模式的一种变体。

### 工厂模式

工厂模式是软件工程领域一种广为人知的设计模式，这种模式抽象了创建具体对象的过程。考虑到 ES 中无法创建类，开发人员就发明了一种函数，用来封装以特定接口创建对象的细节。如下例子所示：

```
function createPerson(name, age, job){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        alert(this.name);
    };
    return o;
}

var person1 = createPerson("HeLang", 24, "Software Engineer");
var person2 = createPerson("ZhangSan", 29, "Doctor");
```

工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎样知道一个对象的类型）。随着JavaScript的发展，又一个新模式出现了。

### 构造函数模式

ES中的构造函数可用来创建特定类型的对象。像 Object 和 Array 这样的原生构造函数，在运行时会自动出现在执行环境中。此外，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。例如，可以使用构造函数模式将前面的例子重写如下。

```
function createPerson(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        alert(this.name);
    }
}

var person1 = createPerson("HeLang", 24, "Software Engineer");
var person2 = createPerson("ZhangSan", 29, "Doctor");
```

按照惯例，构造函数始终都应该以一个大写字母开头，而非构造函数则以小写字母开头。主要是为了区别于ES中的其他函数，因为构造函数本身也是函数，只不过可以用来创建对象而已。

要创建 Person 的新实例，必须使用 new 操作符。以这种方式调用构造函数实际上会经理一下4个步骤：
1. 创建一个新对象；
2. 将构造函数的作用域给新对象（因此this就指向了这个新对象）；
3. 执行构造函数中的代码（为这个新对象添加属性）；
4. 返回新对象。

在前面例子的最后，person1 和 person2 分别保存着 Person 的一个不同的实例。这两个对象都有一个 constructor(构造函数)属性，该属性指向Person，如下所示：

```
alert(person1.constructor == Person); //true
alert(person2.constructor == Person); //true
```
对象的constructor 属性最初是用来标识对象类型的。但是，提到检测对象类型，还是instanceof操作符要更可靠一些。我们在这个例子中创建的所有对象既是 Object 的实例，同时也是 Person 的实例，这一点通过 instanceof 操作符可以得到验证。

```
alert(person1 instanceof Object);   //true
alert(person1 instanceof Person);   //true
alert(person2 instanceof Object);   //true
alert(person2 instanceof Person);   //true
```
创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型；而这正是构造函数模式胜过工厂模式的地方。在这个例子中，person1和 person2之所以同时是Object的实例，是因为所有对象均继承自object。

**1. 将构造函数当做函数**

构造函数与其他函数的唯一区别，就在于调用他们的方式不同。不过，构造函数毕竟也是函数，不逊在定义构造函数的特殊语法。任何函数，只要通过new操作符调用，那他就可以作为构造函数；而任何函数，如果不通过new操作符来调用，那它跟普通函数也不会有什么两样。例如，前面例子中定义的Person()函数可以用过下列任何一种方式来调用。

```
//当做构造函数使用
var person = new Person("HeLang", 24, "Software Engineer");
person.sayName();   //"HeLang"

//作为普通函数调用
Person("ZhangSan", 27, "Doctor");  //添加到window
window.sayName();    //"ZhangSan"

//在另一个对象的作用域中调用
var o = new Object();
Person.call(o,"LiSi",25,"Nurse");
o.sayName();     //"LiSi"
```

**2. 构造函数的问题**

构造函数模式虽然好用，但也并非没有缺点。使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍。在前面的例子中，person1和person2都有一个名为sayName()的方法，但那两个方法不是用一个Function的实例。不要忘了——ES中的函数都是对象，因此每定义一个函数，也就是实例化了一个对象。从逻辑角度讲，此时的构造函数也可以这样定义。

```
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = new Function("alert(this.name)");   //与声明函数在逻辑上是等价的
}
```

从这个角度上来看构造函数，更容易明白每个Person实例都包含一个不同的Function实例（以显示 name 属性）的本质。说明白些，以这种方式创建函数，会导致不同的作用域链和标识符解析，但创建Function新实例的机制仍然是相同的。因此，不同实例上的同名函数是不相等的，以下代码可以证明这一点。

```
alert(person1.sayName == person2.sayName);  //false
```
然而，创建两个完成同样任务的Function 实例的确没有必要；况且有this 对象在，根本不用在
执行代码前就把函数绑定到特定对象上面。因此，大可像下面这样，通过把函数定义转移到构造函数外
部来解决这个问题。


```
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}

function sayName(){
    alert(this.name);
}

var person1 = new Person("HeLang", 29, "Software Engineer");
var person2 = new Person("ZhangSan", 27, "Doctor");
```
在这个例子中，我们把sayName()函数的定义转移到了构造函数外部。而在构造函数内部，我们将sayName 属性设置成等于全局的sayName函数。这样一来，由于sayName 包含的是一个指向函数的指针，因此person1 和person2对象就共享了在全局作用域中定义的同一个sayName()函数。这样做确实解决了两个函数做同一件事的问题，可是新问题又来了：在全局作用域中定义的函数实际上只能被某个对象调用，这让全局作用域有点名不副实。而更让人无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了。好在，这些问题可以通过使用原型模式来解决。

### 原型模式

我们创建的每个函数都有一个 prototype (原型) 属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype 就是通过调用构造函数而创建的那个对象实例的原型对象。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。换句话说，不必再构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面的例子所示。

```
function Person(){}

Person.prototype.name = "HeLang";
Person.prototype.age = 24;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
person1.sayName(); //"HeLang"

var person2 = new Person();
person2.sayName(); //"HeLang"
alert(person1.sayName == person2.sayName); //true

```
在此，我们将sayName()方法和所有属性直接添加到了Person 的prototype 属性中，构造函数变成了空函数。即使如此，也仍然可以通过调用构造函数来创建新对象，而且新对象还会具有相同的属性和方法。但与构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的。换句话说，person1和person2访问的都是同一组属性和同一个sayName()函数。要理解原型模式的工作原理，必须先理解ECMAScript 中原型对象的性质。

**1.理解原型对象**

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含一个指向prototype属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor指向Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

创建了自定义的构造函数之后，其原型对象默认只会取得constructor属性；至于其他方法，则都是从Object继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），**指向构造函数的原型对象**。ECMA-262第5版中管这个指针叫[[Prototype]]。重点注意，这个指针指向的是原型对象而不是构造函数。

以前面使用Person 构造函数和Person.prototype 创建实例的代码为例，图6-1 展示了各个对象之间的关系。
{% asset_img 6-1.png This is a object relative image %}

可以通过isPrototypeOf()方法来确定对象之间是否存在这种关系。从本质上讲，如果[[Prototype]]指向调用isPrototypeOf()方法的对象（Person.prototype），那么这个方法就返回true，如下所示：

```
alert(Person.prototype.isPrototypeOf(person1)); //true
alert(Person.prototype.isPrototypeOf(person2)); //true
```
ECMAScript 5 增加了一个新方法，叫Object.getPrototypeOf()，在所有支持的实现中，这个方法返回[[Prototype]]的值。例如：

```
alert(Object.getPrototypeOf(person1) == Person.prototype); //true
alert(Object.getPrototypeOf(person1).name); //"HeLang"
```
每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。搜索首先从对象实例本身开始。如果在实例中找到了具有给定名字的属性，则返回该属性的值；如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性。如果在原型对象中找到了这个属性，则返回该属性的值。
也就是说，在我们调用person1.sayName()的时候，会先后执行两次搜索。首先，解析器会问：“实例person1有sayName属性吗？”答：“没有。”然后，它继续搜索，再问：“person1的原型有sayName属性吗？”答：“有。”于是，它就读取那个保存在原型对象中的函数。当我们调用person2.sayName()时，将会重现相同的搜索过程，得到相同的结果。而这正是多个对象实例共享原型所保存的属性和方法的基本原理。
虽然可以通过对象实例访问保存在原型中的值，但却不能通过对象实例重写原型中的值。如果我们在实例中添加了一个属性，而该属性与实例原型中的一个属性同名，那我们就在实例中创建该属性，该属性将会屏蔽原型中的那个属性。来看下面的例子。

```
function Person(){}

Person.prototype.name = "HeLang";
Person.prototype.age = 24;
Person.prototype.job = "Software Engineer";

Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

person1.name = "ZhangSan";

alert(person1.name); //"ZhangSan"——来自实例
alert(person2.name); //"HeLang"——来自原型
```
当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性；换句话说，添加这个属性只会阻止我们访问原型中的那个属性，但不会修改那个属性。即使将这个属性设置为null，也只会在实例中设置这个属性，而不会恢复其指向原型的连接。不过，使用delete 操作符则可以完全删除实例属性，从而让我们能够重新访问原型中的属性，如下所示。

```
function Person(){}

Person.prototype.name = "HeLang";
Person.prototype.age = 24;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

person1.name = "ZhangSan";
alert(person1.name); //"ZhangSan"——来自实例
alert(person2.name); //"HeLang"——来自原型

delete person1.name;
alert(person1.name); //"HeLang"——来自原型
```
使用hasOwnProperty()方法可以检测一个属性是存在于实例中，还是存在于原型中。这个方法（不要忘了它是从Object继承来的）只在给定属性存在于对象实例中时，才会返回true。来看下面这个例子。

```
var person1 = new Person();
var person2 = new Person();

alert(person1.hasOwnProperty("name")); //false

person1.name = "ZhangSan";
alert(person1.name); //"ZhangSan"——来自实例
alert(person1.hasOwnProperty("name")); //true

alert(person2.name); //"HeLang"——来自原型
alert(person2.hasOwnProperty("name")); //false
delete person1.name;

alert(person1.name); //"HeLang"——来自原型
alert(person1.hasOwnProperty("name")); //false

```

**2. 原型与in操作符**

有两种方式使用in 操作符：单独使用和在for-in循环中使用。在单独使用时，in操作符会在通过对象能够访问给定属性时返回true，无论该属性存在于实例中还是原型中。在使用for-in 循环时，返回的是所有能够通过对象访问的、可枚举的（enumerated）属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。屏蔽了原型中不可枚举属性（即将[[Enumerable]]标记为false的属性）的实例属性也会在for-in循环中返回，因为根据规定，所有开发人员定义的属性都是可枚举的——只有在IE8 及更早版本中例外。

**3. 更简单的原型语法**

前面例子中每添加一个属性和方法就要敲一遍Person.prototype。为减少不必要的输入，也为了从视觉上更好地封装原型的功能，更常见的做法是用一个包含所有属性和方法的对象字面量来重写整个原型对象，如下面的例子所示。

```
function Person(){}

Person.prototype = {
    name : "Nicholas",
    age : 29,
    job: "Software Engineer",
    sayName : function () {
        alert(this.name);
    }
};
```
在上面的代码中，我们将Person.prototype设置为等于一个以对象字面量形式创建的新对象。最终结果相同，但有一个例外：constructor属性不再指向Person 了。前面曾经介绍过，每创建一个函数，就会同时创建它的prototype对象，这个对象也会自动获得constructor属性。而我们在这里使用的语法，本质上完全重写了默认的prototype 对象，因此constructor属性也就变成了新对象的constructor属性（指向Object构造函数），不再指向Person函数。此时，尽管instanceof操作符还能返回正确的结果，但通过constructor 已经无法确定对象的类型了，如下所示。

```
var friend = new Person();
alert(friend instanceof Object); //true
alert(friend instanceof Person); //true
alert(friend.constructor == Person); //false
alert(friend.constructor == Object); //true
```
在此，用instanceof 操作符测试Object和Person仍然返回true，但constructor属性则等于Object而不等于Person了。如果constructor的值真的很重要，可以像下面这样特意将它设置回适当的值。

```
function Person(){}

Person.prototype = {
    constructor : Person,
    
    name : "Nicholas",
    age : 29,
    job: "Software Engineer",

    sayName : function () {
        alert(this.name);
    }
};
```

以上代码特意包含了一个constructor 属性，并将它的值设置为Person，从而确保了通过该属性能够访问到适当的值。

**原型对象的问题**

原型模式也不是没有缺点。首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。虽然这会在某种程度上带来一些不方便，但还不是原型的最大问题。原型模式的最大问题是由其共享的本性所导致的。
原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。对于那些包含基本值的属性倒也说得过去，毕竟（如前面的例子所示），通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。然而，对于包含引用类型值的属性来说，问题就比较突出了。来看下面的例子。

```
function Person(){}

Person.prototype = {
    constructor: Person,
    
    name : "Nicholas",
    age : 29,
    job : "Software Engineer",
    friends : ["Shelby", "Court"],
    sayName : function () {
        alert(this.name);
    }
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("Van");

alert(person1.friends); //"Shelby,Court,Van"
alert(person2.friends); //"Shelby,Court,Van"
alert(person1.friends === person2.friends); //true
```
在此，Person.prototype 对象有一个名为friends的属性，该属性包含一个字符串数组。然后，创建了Person的两个实例。接着，修改了person1.friends 引用的数组，向数组中添加了一个字符串。由于friends数组存在于Person.prototype而非person1中，所以刚刚提到的修改也会通过person2.friends（与person1.friends 指向同一个数组）反映出来。假如我们的初衷就是像这样在所有实例中共享一个数组，那么对这个结果我没有话可说。可是，实例一般都是要有属于自己的全部属性的。而这个问题正是我们很少看到有人单独使用原型模式的原因所在。

### 组合使用构造函数模式和原型模式
创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。下面的代码重写了前面的例子。

```
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor : Person,
    sayName : function(){
        alert(this.name);
    }
}

var person1 = new Person("HeLang", 24, "Software Engineer");
var person2 = new Person("ZhangSan", 27, "Doctor");
person1.friends.push("Van");

alert(person1.friends); //"Shelby,Count,Van"
alert(person2.friends); //"Shelby,Count"

alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true
```
在这个例子中，实例属性都是在构造函数中定义的，而由所有实例共享的属性constructor和方法sayName()则是在原型中定义的。而修改了person1.friends（向其中添加一个新字符串），并不会影响到person2.friends，因为它们分别引用了不同的数组。这种构造函数与原型混成的模式，是目前在ECMAScript 中使用最广泛、认同度最高的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。

