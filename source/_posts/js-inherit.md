---
title: JavaScript高级程序设计-继承
catalog: true
date: 2018-05-22 18:39:22
subtitle: 基础教程
header-img: "/img/article_header/article_header.png"
tags:
- Javascript
---

许多面向对象语言都支持两种继承方式：接口继承和实现继承。由于ES中没有接口的概念，所以在ES中只支持实现继承。而且其实现继承主要是依靠原型链来实现的。

## 原型链
原型链作为实现继承的主要方法，其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。简单回顾一下构造函数、原型和实例的关系：

**每个构造函数都有一个原型对象，原型对象包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。**

那么，加入我们让原型对象等于另一个类型的实例，结果会怎么样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓原型链的基本概念。

实现原型链有一种基本模式，其代码大致如下。

```
function SuperType(){
    this.property = true;
}

SuperType.prototype.getSuperValue = function(){
    return this.property;
}

function SubType(){
    this.subproperty = false;
}

//继承了 SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function (){
    return this.subproperty;
}

var instance = new SubType();
alert(instance.getSuperValue()); //true
```
实例以及构造函数和原型之间的关系如下图所示：

{% asset_img 6-4.png This is image %}

上面的代码中我们将 SuperType 的实例赋值给 SubType 的原型，所以 SubType 中就包含了 SuperType 的实例属性和原型共享的方法。


**原型链的问题**

通过上面的方式可以实现继承，但是还存在一定的问题，那就是如果 SuperType 中存在一个引用类型值得属性，那么在通过原型的方式继承给 SubType 后，该属性就变成了共享的原型属性了。所以 SubType 的实例修改其值会影响到其他实例。如下代码所示：

```
function SuperType(){
    this.color = ['red','green']
}

SuperType.prototype.getColor = function(){
    return this.color;
}

function SubType(){}

//继承了 SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();

instance1.color.push('blue');

var instance2 = new SubType();

alert(instance2.getColor()); //'red','green','blue'
```


## 借用构造函数
为了解决上述原型实现继承的方式中存在的问题，引入了借用构造函数（有时候也叫作伪造对象或经典继承）的技术。这种技术的基本思想是在子类型构造函数的内部调用超类型构造函数。函数只不过是在特定环境中执行代码的对象，因此通过使用apply()和call()方法也可以在新创建的对象上执行构造函数。示例代码如下：

```
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    alert(this.name);
}

function SubType(name,age){
    //继承了SuperType
    SuperType.call(this, name);
    this.age = age;
}

//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

这种方式虽然解决了原型继承引用类型引起的问题，但是它自身也同样存在一定问题。借用构造函数的本质是给每一个实例都生成了一份内容，所以从实质上来讲并没有达到数据共享的继承理念，除此之外 SuperType 的原型上的方法，SubType 是无法访问到的。

## 组合继承
组合继承，也成为伪经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的集成，而通过借用构造函数来实现对实例属性的继承。示例代码如下：

```
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    
    alert(this.name);
};

function SubType(name, age){
    //继承属性
    SuperType.call(this, name);
    this.age = age;
}

//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"

instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"

instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为JavaScript中最常用的继承模式。而且，instanceof和isPrototypeOf()也能够用于识别基于组合继承创建的对象。