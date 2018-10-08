JS原型以及继承
---

##### 原型

js中的每个对象（null与Object.prototype除外）都和另一个对象关联，这个对象就是原型（prototype），js中的每个对象（null与Object.prototype除外）都从原型中继承属性。

原型属性是不可配置的，不可枚举的，不可写的，但是在大多数浏览器中实现了通过`__proto__`属性来查看原型（ie不支持），并可通过这个属性进行设置。

> `__proto__`是浏览器对对象原型的对外暴露，用以直接查询/设置对象的原型，但IE和Opera还不支持
>
> 通过`__proto__`修改原型非常影响性能，推荐使用 `Object.create()`来实现原型继承

> Object.prototype不继承任何属性

##### 原型链

除了Object.prototype，其他原型都是普通对象，普通对象都有原型，最终原型指向Object.prototype，这样的关联关系为原型链。

例如：

```js
var obj = new Object();
obj.__proto__ === Object.prototype //返回true
//obj的原型直接指向Object.prototype

var obj2 = new String("test"); 
obj2.__proto__ === String.prototype //返回true
String.prototype.__proto__ === Object.prototype //返回true
//obj2的原型指向String.prototype 然后String.prototype的原型指向Object.prototype
```



> 实例为了方便用了`__proto__`，虽然大部分浏览器对`__proto__`都支持，但不推荐使用这个属性，因为还是存在有浏览器不支持的情况。
>
> 获得对象的原型可通过Object.getPrototypeOf()方法获得。
>
> 例如上述例子可写为
>
> ```
> var obj = new Object();
> Object.getPrototypeOf(obj)===Object.prototype //返回true
> ```



通过原型继承的属性一般是可枚举的（可以被for/in遍历到），但是某些特殊属性是不可枚举的（例如toString）

用for..in遍历对象的时候，`__proto__`属性也会被遍历,但是function的prototype属性不会被遍历。



##### function的prototype

每个function对象都有prototype属性，通过`new`操作符可实现继承

```js
//新建一个构造函数
function Animail(name){
   //初始化代码
   this.name = name
}
//设置prototype的属性
Animail.prototype.getName = function(){
    return this.name;
}
//通过new实例化一个对象
var cat = new Animail("TOM") //新建一个名字叫TOM的猫
//调用继承的属性方法
cat.getName()//返回'TOM'
```

其中`new`操作符相当于进行了以下操作

```js
var cat= {};

obj.__proto__ = ClassA.prototype;

ClassA.call(obj);
```



##### js继承的方式：

原型链继承

```js
//创建一个Person类
function Person(first, last, age, gender, interests) {
  this.name = {
    first,
    last
  };
  this.age = age;
  this.gender = gender;
  this.interests = interests;
};

Person.prototype.greeting = function() {
  alert('Hi! I\'m ' + this.name.first + '.');
};

//设置 Teacher() 的原型和构造器引用
//创建一个Teacher类，继承Person的所有成员
function Teacher(first, last, age, gender, interests, subject) {
  Person.call(this, first, last, age, gender, interests);

  this.subject = subject;
}
Teacher.prototype = Object.create(Person.prototype);

//现在Teacher()的prototype的constructor属性指向的是Person(), 
//所以需要下一行来指定constructor 为 Teacher
Teacher.prototype.constructor = Teacher;

//向 Teacher() 添加一个新的greeting()函数
Teacher.prototype.greeting = function() {
  var prefix;

  if(this.gender === 'male' || this.gender === 'Male' || this.gender === 'm' || this.gender === 'M') {
    prefix = 'Mr.';
  } else if(this.gender === 'female' || this.gender === 'Female' || this.gender === 'f' || this.gender === 'F') {
    prefix = 'Mrs.';
  } else {
    prefix = 'Mx.';
  }

  alert('Hello. My name is ' + prefix + ' ' + this.name.last + ', and I teach ' + this.subject + '.');
};
```

实例继承

```js
function Teacher(first, last, age, gender, interests,subject){
  var instance = new Person(first, last, age, gender, interests);
  instance.subject = subject;
  return instance;
}

/**缺点**/
/**
1 实例是父类的实例，不是子类的实例
2 不支持多继承
**/
```

拷贝继承

```js
function Teacher(first, last, age, gender, interests,subject){
  var ps = new Person(first, last, age, gender, interests);
  for(var p in ps){
    Teacher.prototype[p] = ps[p];
  }
  Teacher.prototype.subject = subject;
}

/**缺点**/
/**
1 效率较低，内存占用高
2 无法获取父类不可枚举的方法
**/
```



##### Object.prototype.constructor

一般指向实例对象的构造函数

所有对象都会从它的原型上继承一个 `constructor` 属性

```js
var o = {};
o.constructor === Object; // true

var o = new Object;
o.constructor === Object; // true

var a = [];
a.constructor === Array; // true

var a = new Array;
a.constructor === Array // true

var n = new Number(3);
n.constructor === Number; // true
```



js对象扩展
---

三类：内置对象（Math、String等）、宿主对象（window）、自定义对象（var obj = {}）

两类属性：继承属性（通过原型继承的属性）、自有属性（自身定义的属性）



#### 属性：名字，值，特性

##### 普通属性

##### 存储器属性

getter，setter  (IE之外浏览器都支持，属于ES3语法)

格式

```js
var obj = {
	//普通属性值
	dataprop:value,
	//存储器属性
    get propName(){/**函数体，一般用于返回属性值**/}，
    set propName(){/**函数体，一般用于设置属性值**/}
}
```

实例

```js
var circle = {
    radius:12,
    //可读写存储器属性area
    get area(){
        var r = this.radius;
        return Math.PI*r*r;
    },
    set area(radius){
        this.radius = radius;
    }
}

//使用读写功能
circle.area  //读，返回452.389...
circle.area = 6 //写，执行完后circle的r属性一被改为6
```



##### 属性的特性：attribute

数据属性的特性：值（value）、可写性（writable），可枚举性（enumerable），可配置性（configurable）

存储器属性的特性：读入（get）、写入（set），可枚举性（enumerable），可配置性（configurable）

> 一旦可配置性修改为false,则不可再次修改属性的特性了

方法：Object.getOwnPropertyDescriptor(object,prop)【此方法IE8以上支持】 获取对象属性的特性 

```js
Object.getgetOwnPropertyDescriptor({x:123},'x');
//返回：{value：123,writable:true,enumerable:true,configurable:true}
```

Object.defineProperty(object,prop,attribute)、Object.defineProperties(object,props)：定义属性并设置特性

```js
Object.defineProperty({},'x',
{value：123,writable:true,enumerable:true,configurable:true})

Object.defineProperties({},{
    x:{value：123,writable:true,enumerable:true,configurable:true},
    y:{value：123,writable:true,enumerable:true,configurable:true}
})
```

> IE8不完全支持上述两方法



#### 对象的三个属性：

原型，类属性，可扩展性

##### 原型

用来继承属性的。在实例对象创建之初就设置好了。

`Object.getPrototypeOf(object)`   获取object的原型

 `object.__proto__`    获取object的原型，并可对其修改 【修改操作非常影响性能，推荐使用 `Object.create()`】

> 确保Web浏览器的兼容性。为了更好的支持，建议只使用 [`Object.getPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf)。

`object.isPrototypeOf(p)`  判断p是否是object的原型



##### 类属性

用以表示对象的类型信息，一般为一个字符串。ES3与ES5都未提供方法获取或设置这个属性。

间接方法获取

```js
function classof(obj){
    if(obj===null) return "null";
    if(obj===undefine) return "undefine";
	return {}.toString.call(obj) .slicee(8,-1)
}
```

```js
classof("")//Sring
classof(1)//Number
classof({})//Object
classof(Window)//Window
```

> 但此方法不支持通过自定义的类，没办法通过类属性来区分

##### 可扩展性

用以表示能否给对象添加新的属性

Object.isExtensible(object) 判断object的可扩展性

Object.preventExtensions(object) 转换object为不可扩展【此过程不可逆，不可再转回来】