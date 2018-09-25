js对象
---

三类：内置对象、宿主对象、自定义对象

两类属性：继承属性、自有属性

##### 原型

js中的每个对象（null与Object.prototype除外）都和另一个对象关联，这个对象就是原型（prototype），在浏览器中可通过`__proto__`属性来查看原型（ie不支持），每一个对象可从原型中继承属性。

> `__proto__`是浏览器对对象原型的对外暴露，用以直接查询/设置对象的原型，但IE和Opera还不支持

> Object.prototype不继承任何属性

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



> 虽然大部分浏览器对`__proto__`都支持，但不推荐使用这个属性，因为还是存在有浏览器不支持的情况。
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