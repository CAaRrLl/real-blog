#### 写在前面的话

> 代码分割是webpack最强大的功能之一，它允许将代码拆分成多个捆绑的包(bundle)，然后对这些捆绑的包进行按需加载或者并行加载，使用得当可以达到首次加载速度的提升。

下面是有三种实现代码分割的通用方法，三种也可以搭配使用

```javascript
1.入口文件：通过手动配置webpack多个为入口
2.避免重复：使用SplitChunks插件删除并提取出公共代码块作为单独的一个chunk
3.动态导入：调用模块内的内联函数如import()或者require.ensure()异步加载代码块
```

##### 一、入口文件
> 最简单、最直观的代码分割方式，同时也存在的一些缺陷需要我们注意

a.js：
```javascript
import _ from 'lodash';
console.log(
    _.join(['A', 'Module', 'Loaded!'], ' ')
);
```
index.js：
```javascript
console.log('Index Module Loaded!');
```
webpack.config.js：
```javascript
var path = require('path');
var root_path = path.resolve(__dirname);
module.exports = {
    mode: 'production',
    entry: {  //配置多个入口文件打包成多个代码块
        index: path.resolve(root_path, 'index.js'), 
        a: path.resolve(root_path, 'a.js')
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(root_path, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: path.resolve(root_path, 'node_modules'),
                include: root_path
            }
        ]
    }
}
```
打包结果：
```javascript
Hash: 69ecfe6ff873d88b9d3a
Version: webpack 4.15.1
Time: 663ms
Built at: 2018-07-10 12:00:55
          Asset       Size  Chunks             Chunk Names
    a.bundle.js   70.5 KiB       0  [emitted]  a
index.bundle.js  982 bytes       1  [emitted]  index
[0] (webpack)/buildin/module.js 497 bytes {0} [built]
[1] (webpack)/buildin/global.js 489 bytes {0} [built]
[3] ./a.js 263 bytes {0} [built]
[4] ./index.js 51 bytes {1} [built]
    + 1 hidden module
```
这样使得无关系的两个js模块可以并行加载，这种方式相比较于加载一个等价大小的js模块有速度上的提升。

但是也存在以下问题
1.如果`index.js`也引入了`lodash`，那么`lodash`将会同时打包到两个js模块中
2.不灵活，不能根据核心应用程序的逻辑来动态分割代码

##### 二、避免重复

> [SplitChunks](https://webpack.js.org/plugins/split-chunks-plugin/)插件(```webpack 4.x以前使用CommonsChunkPlugin```)允许我们将公共依赖项提取到现有的`entry chunk`或全新的代码块中

index.js 也引入`lodash`：
```javascript
import _ from 'lodash';
console.log(
    _.join(['Index', 'Module', 'Loaded!'], ' ')
);
```
webpack.config.js 使用`SplitChunks`插件：

```javascript
optimization: {
     splitChunks: {
         chunks: 'all'
     }
 }
```

打包结果：
```javascript
Hash: bdaac52a2513ada9bfa7
Version: webpack 4.15.1
Time: 3540ms
Built at: 2018-07-10 13:11:59
                    Asset      Size  Chunks             Chunk Names
vendors~a~index.bundle.js  69.5 KiB       0  [emitted]  vendors~a~index
              a.bundle.js  1.64 KiB       1  [emitted]  a
          index.bundle.js   1.8 KiB    2, 1  [emitted]  index
[0] ./a.js 378 bytes {1} {2} [built]
[2] (webpack)/buildin/module.js 497 bytes {0} [built]
[3] (webpack)/buildin/global.js 489 bytes {0} [built]
[4] ./index.js 296 bytes {2} [built]
```

`lodash`被单独打包成`venders~a~index.bundle.js`,`SplitChunks插件还可以依照自己的需求进行配置，默认配置情况如下

 ```javascript
1.模块被重复引用或者来自node_modules中的模块
2.在压缩前最小为30kb
3.在按需加载时，请求数量小于等于5
4.在初始化加载时，请求数量小于等于3
```

满足以上条件的模块都会被单独拆分到一个代码块里面

社区还为代码分割提供了很多有用的plugin和loader

*  [`mini-css-extract-plugin`](https://webpack.js.org/plugins/mini-css-extract-plugin): 用于分割css代码
*   [`bundle-loader`](https://webpack.js.org/loaders/bundle-loader): 用于代码分割和懒加载分割后的代码块
*   [`promise-loader`](https://github.com/gaearon/promise-loader): 类似于`bundle-loader`，但是使用promise实现

##### 三、动态导入

有两种方式供我们实现动态导入
```javascript
1.import()   //ES的提案，返回以一个promise，导入的模块在then中拿到
2.require.ensure()  //webpack在编译时会静态地解析代码中的require.ensure()，将里面require的模块添加到一个分开的chunk中。这个新chunk会被webpack通过jsonp来按需加载。
```
以上两种方式都依赖于promise，如果在旧的浏览器中使用 `require.ensure` 请记得 shim `Promise`， 相应的库有 [es6-promise polyfill](https://github.com/stefanpenner/es6-promise).

下面主要使用import()实现动态导入，因为import()目前为止还是提案阶段，需要安装插件`npm install babel-plugin-syntax-dynamic-import --save-dev`，`.babelrc`配置如下
```javascript
{
    "presets": [
        ...
    ],
    "plugins": ["syntax-dynamic-import"]
}
```

下面看核心代码
  webpack.config.js：
```javascript
var path = require('path');
var root_path = path.resolve(__dirname);
module.exports = {
    mode: 'production',
    entry: {
        index: path.resolve(root_path, 'index.js'),
    },
    output: {
        filename: '[name].bundle.js',
        chunkFilename: '[name].demand.js',
        path: path.resolve(root_path, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: path.resolve(root_path, 'node_modules'),
                include: root_path
            }
        ]
    }
}
chunkFilename：指定非entry chunk的文件名，比如import()引入的模块不打包进entry中，而会作为单独的chunk打包，其文件名就由该属性决定
```

index.js：点击按钮才异步加载lodash
```javascript
let btn = document.getElementById('btn');

btn.addEventListener('click', () => {
    import(/* webpackChunkName: "lodash" */ 'lodash').then(
        _ => {
            var app = document.getElementById('app');
            app.textContent = _.join(['Index', 'Module', 'Loaded!'], ' ');
        }
    ).catch(
        err => {
            console.log('loading module error occur', err);
        }
    )
});
webpackChunkName：用于指定动态加载的组件的名称
```

#### 参考资料

[Webpack CodeSpliting](https://webpack.js.org/guides/code-splitting/)
