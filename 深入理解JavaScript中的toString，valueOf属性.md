##### 前言
>**原始类型**：在javascript中有三种主要的原始类型，数值、字符串、布尔值，当使`typeof`操作符时分别返回`'number'`、`'string'`、`'boolean'`，还有另外两个是`'undefined'`和`'null'`,其余类型都是对象类型。
**包装对象**：可以把原始类型的值包装成对象，三种原始类型的包装对象分别为`Number`、`String`、`Boolean`，当使用`new Number()`创建一个数字时，`typeof`操作会返回`'object'`而非`number`，而通过调用数字对象的valueOf方法可以返回该数字对象的原始类型的值。由于包装对象的存在，原始类型也可以调用包装对象上的方法和参数，调用时JavaScrip引擎会自动将原始类型的值转为包装对象实例，调用结束立即销毁实例。
 ---
##### 一、概述
`javascript` 中几乎所有类型都具有`toString`和`valueOf`属性。几乎所有的类型对象比如`Number`,`String`,`Boolean`,`Array`,`Function`,`Object`,`Date`,`RegExp`的原型对象上都有各自的`toString`或`valueOf`方法的实现,故它们的实例化的对象自然就继承了这两个方法。下面看一下这些类型的原型对象上是否有这两个方法的实现：
```javascript
代码示例：
Number.prototype.hasOwnProperty('toString');    //输出true
Number.prototype.hasOwnProperty('valueOf');    //输出true
String.prototype.hasOwnProperty('toString');    //输出true
String.prototype.hasOwnProperty('valueOf');    //输出true
Boolean.prototype.hasOwnProperty('toString');    //输出true
Boolean.prototype.hasOwnProperty('valueOf');    //输出true
Array.prototype.hasOwnProperty('toString');     //输出true
Array.prototype.hasOwnProperty('valueOf');     //输出false 
Function.prototype.hasOwnProperty('toString');     //输出true
Function.prototype.hasOwnProperty('valueOf');     //输出false
Object.prototype.hasOwnProperty('toString');     //输出true
Object.prototype.hasOwnProperty('valueOf');     //输出true
Date.prototype.hasOwnProperty('toString');     //输出true
Date.prototype.hasOwnProperty('valueOf');     //输出true
RegExp.prototype.hasOwnProperty('toString');     //输出true
RegExp.prototype.hasOwnProperty('valueOf');     //输出false
说明：hasOwnProperty用于查看某个对象本身是否具有某属性，只在对象本身查找不在该对象的原型链上查找
```
上面代码中，只有`Array`,`Function`,`RegExp`的原型上没有`valueOf`属性,但是为什么其实例化对象能调用该方法呢？我们都知道上面所有列举的类型的原型`(prototype)`都是继承于`Object`的原型`(prototype)`的，当`Array`,`Function`,`RegExp`的实例化对象找不到某个属性时会沿着原型链往上找，直到找到或给出`undefined`。其实例对象调用的是`Object`原型上的`valueOf`

---
##### 二、toString的作用
1.将值转换为字符串形式并返回，不同类型的toString方法各有不同

|类型|toString()的作用|
|:------:|:----|
|Number|返回文本表示，可接收一个参数表示输出的进制数，默认为十进制，`注意：10..toString()会把第一个.当作小数点`|
|String|直接返回原字符串值|
|Boolean|返回文本表示'true'或'false'|
|Object|返回[object 类型名]，Object类型调用该方法时返回[object Object]|
|Array|将数组元素转换为字符串，用逗号拼接并返回|
|Function|直接返回函数的文本声明|
|Date|返回日期的文本表示, `eg:'Sat Apr 21 2018 16:07:37 GMT+0800 (中国标准时间)'`|
|RegExp|返回文本格式为'**/**`pattern`**/**`flag`',其中pattern是正则表达式,flag是匹配模式：g:全局匹配、i:不分大小写、m:多行匹配；`eg:'/\[bc\]at/g'`|

2.判断对象的类型
```javascript
代码示例：
var a = new Object();
a.toString();      //"[object Object]"
a.toString.call(a);  //"[object Object]"
Object.prototype.toString.call(a);  //"[object Object]"
Object.prototype.toString.call(Object); //"[object Function]"
Object.prototype.toString.call(Object.prototype);   //"[object Object]"
```
上面提到Object.prototype.toString()可以返回"[object 调用该方法的对象类型]",所以说是不是可以通过这个方法来判断对象的类型。只要让对象直接调用该方法即可，这需要借助Function.prototype.call方法。
```javascript
Object.prototype.toString.call(1);   //"[object Number]"
Object.prototype.toString.call('2');    //"[object String]"
Object.prototype.toString.call(true);    //"[object Boolean]"
Object.prototype.toString.call([]);      //"[object Array]"
Object.prototype.toString.call(function(){});    //"[object Function]"
Object.prototype.toString.call(new Date());      //"[object Date]"
Object.prototype.toString.call(/^hello world$/);  //"[object RegExp]"
```
---
##### 三、valueOf的作用
|类型|valueOf()的作用|
|:------:|:----|
|Number|返回原始类型的数字值|
|String|返回原始类型的字符串值|
|Boolean|返回原始类型的Boolean|
|Object|返回对象本身|
|Array|方法继承于Object.prototype,返回原数组|
|Function|方法继承于Object.prototype,返回函数本身|
|Date|方法等同于getTime，返回时间戳|
|RegExp|方法继承于Object.prototype,返回值本身|

---
##### 四、toString和valueOf的关系
两者在类型转换中扮演着重要的角色，两者关系与`javascript`的类型转换息息相关，下面说下javascript的类型转换及其原则

1.强制转换: Number()、String()、Boolean()
```javascript
示例：//原始类型的强制转换
Number(123) // 123
Number('123') // 123
Number('a123b') // NaN
Number('') // 0
Number(true) // 1
Number(false) // 0
Number(undefined) // NaN
Number(null) // 0
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
//下面5种情况为false，其余情况为true
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
```
以上是`Number`、`String`、`Boolean`、`null`、`undefined`对于原始类型的强制转换的规则，但是当`Number()`、`String()`遇到对`对象`的强制转换时情况就不同了，这个时候就会用到`toString()`,`valueOf()`方法了。
```javascript
Number(对象)
调用对象的valueOf方法，若返回原始类型的值，则遵照上面的"非对象强制转换规则",若还是返
回对象，则调用toString方法，若返回原始类型的值，则遵照上面的"非对象强制转换规则",
若返回对象则报错
String(对象)
调用对象的toString方法，若返回原始类型的值，则遵照上面的"非对象强制转换规则",若还是返
回对象，则调用valueOf方法，若返回原始类型的值，则遵照上面的"非对象强制转换规则",
若返回对象则报错
```
2.自动类型转换

`情况一：两个不同类型的值进行数值运算`
`情况二：对非Boolean类型进行Boolean运算`
`情况三：对非数值类型使用一元运算符（+ 、-）`
```
规则：预期什么类型的值，就调用该类型的转换函数。
若预期为String类型的值，那么就用String()来进行强转。
如果该位置即可以是String，也可能是Number，那么默认为Number。
一般Number的优先级高于String
```
```javascript
//1.二元运算符+若有一个参与运算的数值为String则预期为String：
'3'+1;  //31
'3' + true // "3true"
'3' + {} // "3[object Object]"
'3' + [] // "3"
'3' + function (){} // "3function (){}"
'3' + undefined // "3undefined"
'3' + null // "3null"
//2.二元运算符 - 、* 、/ 预期一般为Number:
'7' - '2' // 5
'7' * '2' // 14
true - 2  // -1
false - 1 // -1
'3' * []    // 0
false / '3' // 0
'abcd' - 2   // NaN
null + 2 // 2
undefined + 1 // NaN
//3.一元运算符预期为Number:
+'abcde' // NaN
-'abcde' // NaN
+true // 1
-false // 0
```

##### 参考文章：
[JavaScript 标准参考教程（alpha）](http://javascript.ruanyifeng.com/)
