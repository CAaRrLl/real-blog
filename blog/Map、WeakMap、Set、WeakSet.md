>  ES6引入了四种新的数据结构：
映射(Map)、集合(Set)、弱集合(WeakSet)和弱映射(WeakMap)

#### 一、Map 对比 Object

 > Object作为哈希表使用存在以下问题

1. ``Object``的``key``必须是``String``或者是``Symbol``，当``key``不为字符串时，会调用``toString()``进行强制转换，将转换后的字符串作为``key``

2. ``Object``含有内置属性，如``constructor``、``toString``、``valueOf``，与其同名的键值会产生冲突，可以使用``Object.create(null)``创建一个空对象继承自``null``来避免此问题

3. ``Object``其属性可能是不可遍历的、有些属性可能是在原型链上，所以``Object``长度的获取比较繁琐

4. ``Object``是不可迭代的，即不能使用``for...of``来遍历，`typeof obj[Symbol.iterator] === undefined`

5. ``Object``是无序的，其元素顺序与添加的顺序无关

> Object的使用

1. 创建
```javascript
var obj = new Object();            //创建一个空对象
var obj = new Object;              //创建一个空对象
var obj = Object.create(null);    //obj继承null
```

2. 添加
```javascript
obj['age'] = 11;
obj.age = 11; 
```

3. 获取
```javascript
obj.age;   //11
obj['age'];  //11
```

4. 判断某个key是否存在相应的value
```javascript
if(obj.age !== undefined) {}     //判断存在
if('age' in obj) {}                        //判断存在
```

5. 删除
```javascript
delete obj.age;      //删除彻底该属性
obj.age = undefined;      //仅仅是改变了值为undefined，该对象仍然保留有该属性
```

6. 获取大小
```javascript
Object.keys(obj).length;  //Object.keys只返回对象自身的可遍历属性的全部属性名，不包括原型链上的属性
```

7. 遍历
```javascript
// obj {age: 11, name: "jack"}
for (var key in obj){
   console.log(`key: ${key}, value: ${obj[key]}`);
   //key: age, value: 11
   //key: name, value: jack
}
Object.keys(obj).forEach((key)=> console.log(`key: ${key}, value: ${obj[key]}`));
//key: age, value: 11
//key: name, value: jack
```

> Map更适合用来做哈希表

1. 各种类型的值（包括``object``）都可以作为``key``

2.  `Map`支持迭代，直接使用`for...of`来遍历，而不需要像对象一样先获取`key`再遍历，`typeof obj[Symbol.iterator] === function`


3. `Map`在遇到频繁删除添加和键值对的场景下，有更好的性能表现

4. `Map`用迭代的方式遍历`key`时，得到的`key`的顺序与`key`添加到Map时的顺序相同

> Map的使用方式

1. 创建
```javascript
var map = new Map();   //空Map
var map = new Map([[1,2],[2,3]]);  // map = {1=>2, 2=>3}
```

2. 添加
```javascript
map.set(4,5);     //map = {4=>5}，但添加的key已存在相应的value，则覆盖旧value
```

3. 获取
```javascript
map.get(4);      //5，若相应的value不存在则返回undefined
```

4.  判断某个key是否存在相应的value
```javascript
map.has(4);        //true   通过key判断，返回一个boolean值
```

5. 删除
```javascript
map.delete(4);    //删除成功返回true，返回false表示该属性不存在
map.clear();        //删除所有元素
```

6. 获取大小
```javascript
map.size();
```

7. 遍历
```javascript
//map: { 2=>3, 4=>5}
for (const item of map){
    console.log(item); 
    //Array[2,3]
    //Array[4,5]
}
//或者
for (const [key,value] of map){
    console.log(`key: ${key}, value: ${value}`);
    //key: 2, value: 3
    //key: 4, value: 5
}
//或者
map.forEach((value, key) => console.log(`key: ${key}, value: ${value}`));
//key: 2, value: 3
//key: 4, value: 5
```

#### 二、Set
>  类似于数组，没有重复的元素，可迭代

代码示例
```javascript
//创建
var set = new Set();
var set = new Set([1,2,3,4]);    //出了可接收数组作为参数，也可接受具有iterable接口的其他数据结构

//添加
set.add(5);  

//删除
set.delete(5);    //删除某个值，返回一个布尔值，表示删除是否成功
set.clear();          //清空集合

//判断成员是否存在
set.has(5);      //返回布尔值，表示该值是否为成员

//获取大小
set.size;  

//遍历
for (var item of set) {
  console.log(item);  // 1 2 3 4
}
```

#### 三、weakMap和weakSet

1. weakMap

> `WeakMap`跟`Map`结构类似，也拥有`get`、`has`、`delete`等方法，使用法和使用途都一样。

不同之处：
1. `WeakMap`只接受对象作为键名，但`null`不能作为键名
2. `WeakMap`不支持`clear`方法，不支持遍历，也就没有了`keys`、`values`、`entries`、`forEach`这4个方法，也没有属性`size`
3. `WeakMap` 键名中的引用类型是弱引使用，假如这个引使用类型的值被垃圾机制回收了，`WeakMap`实例中的对应键值对也会消失。`WeakMap`中的`key`不计入垃圾回收，即若只有`WeakMap`中的`key`对某个对象有引用，那么此时执行垃圾回收时就会回收该对象，而`Map`中的`key`是计入垃圾回收

2. WeakSet

>  `WeakSet` 结构与 `Set` 类似，但只有有`add`、`delete`、`has`三个方法

不同之处：
1. `WeakSet`的成员只能是对象，并且`WeakSet`不支持`clear`方法，不支持遍历，也没有`forEach`这个方法，没有属性`size`

2. `WeakSet` 中的对象都是弱引用，即垃圾回收机制不考虑 `WeakSet` 对该对象的引用，如果只有`WeakSet`引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存
