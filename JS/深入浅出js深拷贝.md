文章参考：https://juejin.im/post/5bc1ae9be51d450e8b140b0c?utm_source=gold_browser_extension

 js中深拷贝几乎就是js面试题中必问之题，那就一起来对这个问题由浅入深的探讨

深拷贝与浅拷贝
---

我们都知道js数据类型分为基本类型与引用类型，而引用类型的变量存储的其实是存储数据的地址，所以引用类型变量的赋值只是地址的赋值，两个变量最终指向同一个数据对象，如果对其中一个变量操作都会影响到另一个变量，例如

```js
var a = {a:123}
var b = a
b.a = 333
console.log(a)//输出{a:333}
```

所以就引申出了深拷贝与浅拷贝

浅拷贝：只对引用地址进行拷贝，最终两个变量指向同一个数据

深拷贝：对引用类型进行深度复制，使两者的内存和以后的操作互不影响。

```js
//浅拷贝
function shallowClone(source) {
    var target = {};
    for(var i in source) {
        if (source.hasOwnProperty(i)) {
            target[i] = source[i];
        }
    }
    return target;
}
```



深拷贝的简单实现
---

通过浅拷贝+递归的方式来实现深拷贝

```js
function clone(source) {
    var target = {};
    for(var i in source) {
        //判断是否是自身属性
        if (source.hasOwnProperty(i)) {
            if (typeof source[i] === 'object') {
                target[i] = clone(source[i]); // 注意这里
            } else {
                target[i] = source[i];
            }
        }
    }

    return target;
}
```

但是这个方法还是存在一些问题

**爆栈**，当数据的层次很深时，就会产生栈溢出:`Maximum call stack size exceeded`

```js
//生成指定深度和每层广度的代码
function createData(deep, breadth) {
    var data = {};
    var temp = data;

    for (var i = 0; i < deep; i++) {
        temp = temp['data'] = {};
        for (var j = 0; j < breadth; j++) {
            temp[j] = j;
        }
    }

    return data;
}

clone(createData(10000)); // Maximum call stack size exceeded
clone(createData(10, 100000)); // ok 广度不会溢出

```

但大都数情况不会出现深度那么深的数据，但还有一种情况也会出现爆栈（**循环引用**）

```js
var a = {};
a.a = a;

clone(a) // Maximum call stack size exceeded 直接死循环
```

还有一种问题就是**引用丢失**：假如一个对象a，a下面的两个键值都引用同一个对象b，经过深拷贝后，a的两个键值会丢失引用关系，从而变成两个不同的对象，o(╯□╰)o

```js
var b = {};
var a = {a1: b, a2: b};

a.a1 === a.a2 // true

var c = clone(a);
c.a1 === c.a2 // false
```



一行代码的深拷贝
---

```js
function cloneJSON(source) {
    return JSON.parse(JSON.stringify(source));
}
```

通过系统自带的JSON来实现深拷贝，但是也会出现递归爆栈的问题

```js
var a = {};
a.a = a;

cloneJSON(a) // Uncaught TypeError: Converting circular structure to JSON	
```

因为`stringify`这个方法对循环引用做了校验，所以直接抛出异常

解决递归爆栈
---

既然递归会造成爆栈，那就不用递归，改用循环

假设有如下的数据结构

```
var a = {
    a1: 1,
    a2: {
        b1: 1,
        b2: {
            c1: 1
        }
    }
}
复制代码
```

这不就是一个树吗，其实只要把数据横过来看就非常明显了

```
    a
  /   \
 a1   a2        
 |    / \         
 1   b1 b2     
     |   |        
     1  c1
         |
         1  
```



用循环遍历一棵树，需要借助一个栈，当栈为空时就遍历完了，栈里面存储下一个需要拷贝的节点

首先我们往栈里放入种子数据，`key`用来存储放哪一个父元素的那一个子元素拷贝对象

然后遍历当前节点下的子元素，如果是对象就放到栈里，否则直接拷贝

```js
//循环深度拷贝方法
function cloneLoop(x) {
    //克隆对象
    const root = {};
    // 栈
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x,
        }
    ];
    //循环
    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const parent = node.parent;//父节点
        const key = node.key;//属性
        const data = node.data;//值

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }

        for(let k in data) {
            if (data.hasOwnProperty(k)) {
                if (typeof data[k] === 'object') {
                    // 下一次循环
                    loopList.push({
                        parent: res,
                        key: k,
                        data: data[k],
                    });
                } else {
                    res[k] = data[k];
                }
            }
        }
    }
    return root;
}

```

解决循环引用与引用丢失
---

解决原理就是，把每次拷贝一个新的对象时，把这个原对象保存下来，再之后拷贝前都检测一下是否拷贝过，如果拷贝过了，直接引用原来的，这样就能保存引用关系了。

```js
// 保持引用关系
function cloneForce(x) {
    // =============
    //引入uniqueList用来存储已经拷贝的数组
    const uniqueList = []; // 用来去重
    // =============

    let root = {};

    // 循环数组
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x,
        }
    ];

    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const parent = node.parent;
        const key = node.key;
        const data = node.data;

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }
        
        // =============
        // 数据已经存在
        let uniqueData = find(uniqueList, data);
        if (uniqueData) {
            parent[key] = uniqueData.target;
            continue; // 中断本次循环
        }

        // 数据不存在
        // 保存源数据，在拷贝数据中对应的引用
        uniqueList.push({
            source: data,
            target: res,
        });
        // =============
    
        for(let k in data) {
            if (data.hasOwnProperty(k)) {
                if (typeof data[k] === 'object') {
                    // 下一次循环
                    loopList.push({
                        parent: res,
                        key: k,
                        data: data[k],
                    });
                } else {
                    res[k] = data[k];
                }
            }
        }
    }

    return root;
}
//判断是否存在引用关系
function find(arr, item) {
    for(let i = 0; i < arr.length; i++) {
        if (arr[i].source === item) {
            return arr[i];
        }
    }

    return null;
}

```



下面对各种方法进行对比，希望给大家提供一些帮助

|          | clone        | cloneJSON    | cloneLoop | cloneForce   |
| -------- | ------------ | ------------ | --------- | ------------ |
| 难度     | ☆☆           | ☆            | ☆☆☆       | ☆☆☆☆         |
| 兼容性   | ie6          | ie8          | ie6       | ie6          |
| 循环引用 | 一层         | 不支持       | 一层      | 支持         |
| 栈溢出   | 会           | 会           | 不会      | 不会         |
| 保持引用 | 否           | 否           | 否        | 是           |
| 适合场景 | 一般数据拷贝 | 一般数据拷贝 | 层级很多  | 保持引用关系 |



对于其他特殊数据类型的处理
---

#### 1 Function

网上一直没找到这类对象的怎么处理（获取我找的方式不对），可能这类值一般不需要考虑到深拷贝的问题。

如果需要对function做处理，那应该怎么处理呢

```
//data:原对象，ta：克隆对象
if(typeof data == 'function'){
    data = 
}
```



#### 2 ArrayBuffer，TypedArray ，DataView类数组对象

`ArrayBuffer`对象、`TypedArray`视图和`DataView`视图是 JavaScript 操作二进制数据的一个接口。这些对象早就存在，属于独立的规格（2011 年 2 月发布），ES6 将它们纳入了 ECMAScript 规格，并且增加了新的方法。它们都是以数组的语法处理二进制数据，所以统称为二进制数组。

#### 3 ES6 的set, map, weakset, weakmap。。。