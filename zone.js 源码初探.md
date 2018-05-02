### 概述
>zone是异步任务中持续存在的执行上下文
zone.js提供了一种机制来拦截异步任务以及追踪异步任务
zone.js的代码库使用monkey patch的方式，在运行时动态地给浏览器的异步api进行一层包装，并让其在zone的上下文执行。通过指定拦截规则，能够让我们对异步操作的调用和调度进行拦截，还可以在异步任务之前或之后添加代码。一个系统中能够存在多个zone实例，但是任意时刻只能有一个处于激活状态，通过Zone.current可以获取当前激活的zone实例。

**zone.js所做的事情有如下几点**：

    1.拦截异步任务的调度
    2.在异步操作中封装回调函数，以此来进行错误处理和zone追踪
    3.提供一种方法添加数据到zones中去
    4.提供最后一帧的错误处理的具体上下文
    5.拦截阻塞的方法(alert/confirm/prompt/sync ajax)
-----------------
##### 一、封装回调函数
zones需要在异步操作中持续存在，所以每次的异步任务建立时都需要捕获当前的zone并将其封装到回调函数中，在执行异步任务时，将当前的Zone.current恢复为之前捕获的zone。所以如果一个异步操作链是一个执行线程，那么Zone.current将充当为线程的局部变量。

-----------------------------

##### 二、异步操作的调度
存在三种可以调度的异步任务

    1.MicroTask：在当前task结束之后和下一个task开始之前执行的，不可取消，如`Promise`，`MutationObserver`、`process.nextTick`
    2.MacroTask：一段时间后才执行的task，可以取消，如`setTimeout`, `setInterval`, `setImmediate`, `I/O`, `UI rendering`
    3.EventTask：监听未来的事件，可能执行0次或多次，执行时间是不确定的

zone.js对上述的api都进行了monkey patch，对这些api都进行了重谢并替换了全局对象中的默认方法

--------------

##### 三、可组合性
zones之间可以同过Zone.fork()组合在一起，一个子zone可以创建自己规则，可以：

    1.将拦截委派给父zone，有选择地在封装回调之前或之后添加钩子，
    2.或者不用代理处理请求

组合性允许zones之间彼此互补干扰，比如顶层的zone可以选择捕获错误，子zone可以选择追踪用户的行为

--------

##### 四、根zone
浏览器在开始运行时会创建一个特殊的根zone，其余所有的zone都是根zone的子zone

---------

##### 五、分析
官方例子：profiling.html 计算异步任务的耗时
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
   ...
</head>
<body>
  <h1>Profiling with Zones</h1>
  <button id="b1">Start Profiling</button>
  <script>
  function sortAndPrintArray (unsortedArray) {
    profilingZoneSpec.reset();
    //执行排序
    asyncBogosort(unsortedArray, function (sortedArray) {
      console.log(sortedArray);
      //排序结束输出耗时
      console.log('sorting took ' + profilingZoneSpec.time() + ' of CPU time');
    });
  }
  function asyncBogosort (arr, cb) {
    //异步任务建立
    setTimeout(function () {
      if (isSorted(arr)) {
        cb(arr);
      } else {
        var newArr = arr.slice(0);
        newArr.sort(function () {
          return Math.random() - 0.5;
        });
        asyncBogosort(newArr, cb);
      }
    }, 0);
  }
  function isSorted (things) {
    for (var i = 1; i < things.length; i += 1) {
      if (things[i] < things[i - 1]) {
        return false;
      }
    }
    return true;
  }
  //主函数
  function main () {
    var unsortedArray = [3,4,1,2,7];
    //异步任务
    b1.addEventListener('click', function () {
      sortAndPrintArray(unsortedArray);
    });
  }
  //返回zone的规则，并使用闭包维护了一个变量time来记录时长
  var profilingZoneSpec = (function () {
    var time = 0,
        //当存在performance时使用performane的method，否则调用Date的method
        //主要用于获取当前时间
        timer = performance ?
                    performance.now.bind(performance) :
                    Date.now.bind(Date);
    //返回zone的规则集
    return {
      //delegate类型是ZoneDelegate，每个zone都会有一个ZoneDelegate对象，主要    
      //为zone调用传入的回调函数，建立、调用回调函数中的异步任务，捕捉异步任    
      //务的错误，这里传入的delegate为父zone的代理对象。
      //异步任务被调用前会执行该函数
      onInvokeTask: function (delegate, current, target, task, applyThis, applyArgs) {
        this.start = timer();    //获得开始时间
        //可以让父代理执行异步回调也可自己执行不使用代理
        delegate.invokeTask(target, task, applyThis, applyArgs);
        //异步回调执行完毕计算耗时
        time += timer() - this.start; 
      },
      //获取当前耗时
      time: function () {
        return Math.floor(time*100) / 100 + 'ms';
      },
      //将当前耗时置为零
      reset: function () {
        time = 0;
      }
    };
  }());
  //根zone创建了一个子zone，子zone执行函数main
  Zone.current.fork(profilingZoneSpec).run(main);
  </script>
</body>
</html>
```
`Zone.current.fork(profilingZoneSpec).run(main)`是这份代码最重要的一句，为了弄懂这句话究竟做了什么，先来看下zone.js的源码实现
```javascript
Class Zone：
...
private _zoneDelegate: ZoneDelegate;
static get current(): AmbientZone {
      return _currentZoneFrame.zone;
}
public fork(zoneSpec: ZoneSpec): AmbientZone {
      if (!zoneSpec) throw new Error('ZoneSpec required!');
      return this._zoneDelegate.fork(this, zoneSpec);
}
public run<T>(callback: (...args: any[]) => T, applyThis: any = undefined, applyArgs: any[] = null
        ,source: string = null): T {
      _currentZoneFrame = {parent: _currentZoneFrame, zone: this};
      try {
        return this._zoneDelegate.invoke(this, callback, applyThis, applyArgs, source);
      } finally {
        _currentZoneFrame = _currentZoneFrame.parent;
      }
    }
...

Class ZoneDelegate:
...
fork(targetZone: Zone, zoneSpec: ZoneSpec): AmbientZone {
      return this._forkZS ? this._forkZS.onFork(this._forkDlgt, this.zone, targetZone, zoneSpec) :
                            new Zone(targetZone, zoneSpec);
    }
 invoke(targetZone: Zone, callback: Function, applyThis: any, applyArgs: any[], source: string):
        any {
      return this._invokeZS ?
          this._invokeZS.onInvoke(
              this._invokeDlgt, this._invokeCurrZone, targetZone, callback, applyThis, applyArgs,
              source) :
          callback.apply(applyThis, applyArgs);
    }
...
```
如代码所示，`current`方法是Zone类的一个静态方法，返回`_currentZoneFrame.zone`，`_currentZoneFrame`是一个全局对象，保存了当前系统中的zone帧链，它有两个属性，`parent`指向了父zoneFrame，`zone`指向了当前激活的zone对象。所以`_currentZoneFrame`并不是固定不变的。

-----------

```javascript
let _currentZoneFrame: _ZoneFrame = {parent: null, zone: new Zone(null, null)};
```
系统初始化时，实例化zone时，需往构造函数传入一个父zone对象和一个zone规则对象，当zone规则对象为null时，构造函数将认为该zone是根zone。这也说明了为什么浏览器在开始运行时会创建一个特殊的根zone，因为在声明`_currentZoneFrame`时就创建了根zone。
所以 `Zone.current.fork(profilingZoneSpec).run(main)` 的意思就是，使用根zone创建新的未命名的子zone，然后让子zone去运行main()

-------------

```javascript
public fork(zoneSpec: ZoneSpec): AmbientZone {
      if (!zoneSpec) throw new Error('ZoneSpec required!');
      return this._zoneDelegate.fork(this, zoneSpec);    //体交由zone的代理对象来实现
}
```
从上面代码中可以知道，zone实例的fork方法中会交代给代理去创建新的子zone。因为zone.js允许我们在新建子zone前添加hook，代理对象的fork方法会判断是否有onFork的hook，若有则先执行onFork，如下所示
```
fork(targetZone: Zone, zoneSpec: ZoneSpec): AmbientZone {
      return this._forkZS ?   //fork规则是否存在
      this._forkZS.onFork(this._forkDlgt, this.zone, targetZone, zoneSpec) :
      new Zone(targetZone, zoneSpec);
    }
```
当发现规则中有onFork的要求时，则先执行该hook。除此onFork之外，在拦截规则中我们同样可以设置onInvoke、onHandleError、onInvokeTask、onCancelTask等hook，原理同上，zone都会将其交由代理来处理。例如onInvoke，拿`Zone.current.fork(profilingZoneSpec).run(main)`举例，run会调用Invoke方法，下面是run方法的具体实现
```javascript
public run<T>(callback: (...args: any[]) => T, applyThis: any = undefined, applyArgs: any[] = null
        ,source: string = null): T {
      _currentZoneFrame = {parent: _currentZoneFrame, zone: this};
      try {
        return this._zoneDelegate.invoke(this, callback, applyThis, applyArgs, source);
      } finally {
        _currentZoneFrame = _currentZoneFrame.parent;
      }
    }
```
可以看到，执行`run(main)`之后，zone将main的调用交给了代理对象，代理对象的invoke方法实现如下
```javascript，
invoke(targetZone: Zone, callback: Function, applyThis: any, applyArgs: any[], source: string):
        any {
      return this._invokeZS ?
          this._invokeZS.onInvoke(
              this._invokeDlgt, this._invokeCurrZone, targetZone, callback, applyThis, applyArgs,
              source) :
          callback.apply(applyThis, applyArgs);    //直接执行传入的回调
    }
```
当代理对象发现规则中有onInvoke的hook时，则先执行该hook。但是在该样例中并没有设置onInvoke，所以代理对象直接执行了main。

---------------

到这里或许会很疑惑，zone执行了main就结束了吗？它是如何追踪异步任务的，答案是monkey patch，通过对异步任务的patch，在任务创建前和执行前都进行了一层封装，下面来看zone.js是如何patch setTimeout的
```javascript
//browser.ts
Zone.__load_patch('timers', (global: any) => {
  const set = 'set';
  const clear = 'clear';
  patchTimer(global, set, clear, 'Timeout');
  patchTimer(global, set, clear, 'Interval');
  patchTimer(global, set, clear, 'Immediate');
});
```
patchTimer是实现patch的主要方法，参数global是window对象，set、clear是异步任务的前缀，最后一个参数是异步任务的后缀。接着看下patchTimer的部分实现
```javascript
patchTimer(window: any, setName: string, cancelName: string, nameSuffix: string) {
  let setNative: Function = null;    //原生的setTimeout
  let clearNative: Function = null;    //原生的cleanTimeout
  setName += nameSuffix;        //获得'setTimeout'
  cancelName += nameSuffix;    //获得'cleanTimeout'

  //该方法会在建立异步任务时被调用，具体可看官方源码
  function scheduleTask(task: Task) {
    const data = <TimerOptions>task.data;
    //要执行的异步任务
    function timer() {
      try {
        task.invoke.apply(this, arguments);
      } finally {...}
      }
     }
   }
    data.args[0] = timer;
    //调用原生setTimeout，将data.args[0] 即callback和data.args[1]即delay作为参数  
    data.handleId = setNative.apply(window, data.args);
    return task;
}
  setNative =
      patchMethod(window, setName, (delegate: Function) => function(self: any, args: any[]) {}
  clearNative =
      patchMethod(window, cancelName, (delegate: Function) => function(self: any, args: any[]) {}
}
```
`patchMethod`返回的是原生的`setTimeout`，同时`patchMethod`会将`window.setTimeout`进行patch，下面是patch后的`window.setTimeout`的部分代码
```javascript
window.setTimeout = function() {
  return patchDelegate(this, arguments as any);
}
patchDelegate(self: any, args: any[]) {
    if (typeof args[0] === 'function') {
      const options: TimerOptions = {
      handleId: null,
      isPeriodic: nameSuffix === 'Interval',
      delay: (nameSuffix === 'Timeout' || nameSuffix === 'Interval') ? args[1] || 0 : null,
      args: args
    };
      //新建异步任务对象，同时在返回task前，会调用传入的scheduleTask方法，
      //scheduleTask方法会调用原生的setTimeout对象
      const task =
        scheduleMacroTaskWithCurrentZone(setName, args[0], options, scheduleTask, clearTask);
        ...
}
```
所以，当我们调用全局的setTimeout时，就会将传入的回调函数和延迟时间包装为一个Task对象，然后zone代理对象执行Task的scheduleTask方法，scheduleTask方法又调用了原生setTimeout方法，然后setTimeout在一段时间后执行Task的invoke方法，invoke方法里包装了真正的回调函数。

--------------

最后说说官方例子中``onInvokeTask``是什么时候执行的
```javascript
//代理对象的invokeTask方法
invokeTask(targetZone: Zone, task: Task, applyThis: any, applyArgs: any): any {
      return this._invokeTaskZS ?
          this._invokeTaskZS.onInvokeTask(
              this._invokeTaskDlgt, this._invokeTaskCurrZone, targetZone, task, applyThis,
              applyArgs) :
          task.callback.apply(applyThis, applyArgs);
}
```
异步任务执行时最后会经过层层传递，最后交由代理对象来执行，代理对象会先判断是否有设置`onInvokeTask`的hook，有则执行`onInvokeTask`，不执行异步任务的回调函数。
```javascript
onInvokeTask: function (parentdelegate, current, target, task, applyThis, applyArgs) {
      this.start = timer();       
      //交由父代理来执行异步任务
      parentdelegate.invokeTask(target, task, applyThis, applyArgs);
      time += timer() - this.start; 
 },
```
执行`onInvokeTask`时，会交由父代理来执行异步任务，因为可能存在父zone中也设置了`onInvokeTask`的情况，直到有某个父zone没有设置`onInvokeTask`时，才真正执行异步任务的回调函数。

##### 参考:
[Brian Ford Zone](https://www.youtube.com/watch?v=3IqtmUscE_U)

[zone.js](https://github.com/angular/zone.js)

[How the hell does zone.js really work?](https://blog.strongbrew.io/how-the-hell-do-zones-really-work/)
