### 前言

> 之前对闭包的理解一直不是很清晰，总觉得自己已经理解了，面试的时候也说得头头是道，但是看了看了一篇有关javascript闭包内存泄漏的文章后发现自己对闭包的理解完全不够

### 一、什么是闭包
> 闭包是捆绑在一起（封闭）的函数与对其周围状态（词法作用域环境下）的引用的组合。 换句话说，闭包使你能从内部函数访问外部函数的作用域。 在JavaScript中，每次创建函数时，都会在函数创建时创建闭包。

```javascript
//创建闭包的一般示例
function a() {
  var b = 1;
  function c() {
    console.log(b);
  }
}
function a() {
  var b = 1;
  return function c() {
    console.log(b);
  }
}
function a(b) {
  function c() {
    console.log(b);
  }
}
//直接在全局环境创建函数实际上也是一个闭包,因为满足闭包的定义
var a = 1;
function b() {
  console.log(a);
}
```

### 二、如何使用闭包

> 要使用闭包，只需在另一个函数内定义一个函数并公开它。 公开函数的方式有两种，一种是直接return嵌套函数，另一种是将嵌套函数作为参数传递给另外一个函数

```javascript
//直接return嵌套函数的方式一
var accept;
function a() {
  var b = 1;
  return function c() {
    console.log(b);
  }
}
accept = a();
//直接return嵌套函数的方式二
var accept;
function a() {
  var b = 1;
  function c() {
    console.log(b);
  }
  accept = c;
}
//作为参数传递的方式
function accept(func) {
  ...
}
function a() {
  var b = 1;
  function c() {
    console.log(b);
  }
  accept(c);
}
```
> 即便我们暴露出了内部函数`(如上文中的函数c)`，并且外部函数`(如上文中的a函数)`已经执行结束并返回，内部函数仍然可以访问外部函数的作用域

还有一种比较特殊的闭包，立即执行函数，通过构建函数作用域的形式将变量与全局命名空间隔离， 并通过闭包的形式让它们存在于整个运行时，一般用于保护全局命名空间免受变量污染
```javascript
(function(window){
    var foo, bar;
    function private(){
        // do something
    }
    window.Module = {
        public: function(){
            // do something 
        }
    };
})(this);
```

### 三、闭包的作用

```javascript
//在函数a执行完后，i不会被回收，它会存在于b的作用域链上
function a() {
    var i = 0;
    function b() {
        console.log(++i);
    }
    return b;
}
var c = a();
c();    //1
c();    //2
```

其实可以理解为b对a的context有访问权限，换句话说，就是b在自身的活动对象找不到的情况下，会在作用域链上去找

### 四、闭包的规则

> 一个函数`a`里的所有闭包函数(内嵌函数)都共享同一个`context`，这个`context`来源于函数`a`。

当函数`a`执行完毕后，所有`没有被闭包函数引用的变量都会被垃圾回收`，而所有`被闭包函数引用的变量都会作为context`。`context`被所有闭包函数共享(即可以通过作用域链访问到)。若存在一个闭包函数被函数a暴露给外部引用，那么函数`a`执行完后，`context`中的变量不会被垃圾回收

```javascript
function PING() {
    this.a = new Array(1000000);
}
function a() {
    var ping = new PING();
    var obj = {name: 'carl', arr: new Array(1000000)};
    var obj2 = {name: 'jack', arr: new Array(1000000)};
    function f1() {
        if(obj) {}
    }
    function f2() {
        if(ping) {}
    }
    return function b() {}
}
var c = a();
```
![执行结果.png](https://upload-images.jianshu.io/upload_images/8568352-fa3fd8acbc36dc94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果函数`a`里使用了`eval`且把`a`中的内嵌函数`b`暴露给外部引用的话，那么`a`中所有的b除外的变量都不会被回收

```javascript
function PING() {
    this.a = new Array(1000000);
}
function a() {
    var ping = new PING();
    var obj = {name: 'carl', arr: new Array(1000000)};
    var obj2 = {name: 'jack', arr: new Array(1000000)};
    function f() { eval(); }
    return function b() {}
}
var c = a();
```

![执行结果.png](https://upload-images.jianshu.io/upload_images/8568352-b47410bb8c2c0afa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 五、看一下有关闭包的内存泄漏的代码

```javascript
var theThing = null;
var replaceThing = function () {
    var originalThing = theThing;   
    var unused = function () {
        if(originalThing) {}
    };
    theThing = {
        longStr: Date.now() + Array(1000000).join('*'),
        someMethod: function () {}
    };
};
setInterval(replaceThing, 1000);
```
分析
第一秒：`someMethod` 被暴露到外部，`unused`引用了`originalThing`。`someMethod`的context一直包含着`originalThing`，故`originalThing`没被回收，`originalThing`指向null

第二秒：`someMethod` 被暴露到外部，`unused`引用了`originalThing`，`someMethod`的context一直包含着`originalThing`，故`originalThing`没被回收，但是`originalThing`指向了第一秒的`theThing`所指向的内存空间(包含第一秒的`someMethod`，所以第一秒的`originalThing`到了第二秒还是没有被回收，但是`originalThing`指向null，所以问题不大)

第三秒：`someMethod` 被暴露到外部，`unused`引用了`originalThing`，`someMethod`的context一直包含着`originalThing`，故`originalThing`没被回收，但是`originalThing`指向了第二秒的`theThing`所指向的内存空间(包含第二秒的`someMethod`，所以第二秒的`originalThing`到了第三秒还是没有被回收，但是`originalThing`指向第二秒的`theThing`所指向的内存空间，这个空间很大，问题也很大)

第四秒：...

综上所述，第三秒没有回收第二秒的`theThing`指向的内存空间，第四秒也没回收第二秒的`theThing`指向的内存空间和第三秒的`theThing`指向的内存空间...

最后，上面这种闭包的写法是不合理的
