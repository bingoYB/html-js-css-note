for in 和for of的区别
===

#### 1 遍历数组通常用for循环

ES5的话也可以使用forEach，ES5具有遍历数组功能的还有map、filter、some、every、reduce、reduceRight等，只不过他们的返回结果不一样。但是使用foreach遍历数组的话，使用break不能中断循环，使用return也不能返回到外层函数。

```
Array.prototype.method=function(){
　　console.log(this.length);
}
var myArray=[1,2,4,5,6,7]
myArray.name="数组"
for (var index in myArray) {
  console.log(myArray[index]);
}
```

#### 2 for in遍历数组的毛病

1.index索引为字符串型数字，不能直接进行几何运算
 2.遍历顺序有可能不是按照实际数组的内部顺序
 3.使用for in会遍历数组所有的可枚举属性，包括原型。例如上栗的原型方法method和name属性
 所以for in更适合遍历对象，不要使用for in遍历数组。

那么除了使用for循环，如何更简单的正确的遍历数组达到我们的期望呢（即不遍历method和name），ES6中的for of更胜一筹.

```
Array.prototype.method=function(){
　　console.log(this.length);
}
var myArray=[1,2,4,5,6,7]
myArray.name="数组";
for (var value of myArray) {
  console.log(value);
}
```

*记住，for in遍历的是数组的索引（即键名），而for of遍历的是数组元素值。*

for of遍历的只是数组内的元素，而不包括数组的原型属性method和索引name

### 3 遍历对象

遍历对象 通常用for in来遍历对象的键名

```
Object.prototype.method=function(){
　　console.log(this);
}
var myObject={
　　a:1,
　　b:2,
　　c:3
}
for (var key in myObject) {
  console.log(key);
}
```

for in 可以遍历到myObject的原型方法method,如果不想遍历原型方法和属性的话，可以在循环内部判断一下,hasOwnPropery方法可以判断某属性是否是该对象的实例属性

```
for (var key in myObject) {
　　if（myObject.hasOwnProperty(key)){
　　　　console.log(key);
　　}
}
```

同样可以通过ES5的Object.keys(myObject)获取对象的实例属性组成的数组，不包括原型方法和属性

```
Object.prototype.method=function(){
　　console.log(this);
}
var myObject={
　　a:1,
　　b:2,
　　c:3
}
```

### 总结

- for..of适用遍历数/数组对象/字符串/map/set等拥有迭代器对象的集合.但是不能遍历对象,因为没有迭代器对象.与forEach()不同的是，它可以正确响应break、continue和return语句
- for-of循环不支持普通对象，但如果你想迭代一个对象的属性，你可以用for-in循环（这也是它的本职工作）或内建的Object.keys()方法：

```
for (var key of Object.keys(someObject)) {
  console.log(key + ": " + someObject[key]);
}
```

- 遍历map对象时适合用解构,例如;

```
for (var [key, value] of phoneBookMap) {
   console.log(key + "'s phone number is: " + value);
}
```

-  **当你为对象添加myObject.toString()方法后，就可以将对象转化为字符串，同样地，当你向任意对象添加myObjectSymbol.iterator方法，就可以遍历这个对象了。**
   举个例子，假设你正在使用jQuery，尽管你非常钟情于里面的.each()方法，但你还是想让jQuery对象也支持for-of循环，你可以这样做：

```
jQuery.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
```

所有拥有[Symbol.iterator]()的对象被称为可迭代的。在接下来的文章中你会发现，可迭代对象的概念几乎贯穿于整门语言之中，不仅是for-of循环，还有Map和Set构造函数、解构赋值，以及新的展开操作符。

- for...of的步骤
   or-of循环首先调用集合的[Symbol.iterator]()方法，紧接着返回一个新的迭代器对象。迭代器对象可以是任意具有.next()方法的对象；for-of循环将重复调用这个方法，每次循环调用一次。举个例子，这段代码是我能想出来的最简单的迭代器：

```
var zeroesForeverIterator = {
 [Symbol.iterator]: function () {
   return this;
  },
  next: function () {
  return {done: false, value: 0};
 }
};
```

作者：Haiya_32ef

链接：https://www.jianshu.com/p/c43f418d6bf0

來源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。