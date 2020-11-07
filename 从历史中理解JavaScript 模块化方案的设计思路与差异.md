# 引言

2020 年 9 月中旬，我有一个朋友让我写一写关于模块化的知识。我第一时间想到的就是：

```JavaScript
import React from 'react';

const _ = require('loash');
```

对于以上两行代码，我产生了以下几个问题

1. 为什么有的用 import，有的用 require
2. 如果分属与不同的规范，好像有听说过不能同时使用的传闻
3. 为什么需要模块化？

# 解惑

### 1. `import` 与 `require`

#### ESM

2016 年 5 月，ECMAScript 6.0 成为了国际标准，在这一标准中，首次使用了`import`和`export`关键字用于模块化方案，被称之为 ES Module（简称 ESM）模块化解决方案，可以说是官方发（~~糖~~）话了。可是由于历史遗留原因，各种模块化解决方案已经占据了一定的市场，出现了并存的现象，以`require` 为代表的类`commonJs`解决方案就是其中一员。

#### CJS

为什么我说我们使用的`require`是类 `CommonJS`解决方案呢？因为真正的`CommonJS`是用于`nodeJs`模块化解决方案，该解决方案在浏览器端主要有以下两个弊端：

- 导出的变量如果没有 function 包裹，会暴露在全局变量中
- 服务端模块化可以使用同步加载依赖的方式，因为依赖保存在本地，读取本地文件的时间很短，但是浏览器端需要在网络中请求，时间较长阻塞浏览器加载，需要支持异步回调。

#### AMD

这两个重要缺陷，导致浏览器端不能直接使用`CommonJS`的规范解决模块化问题。这时候在当时的社区，出现了很多其他衍生的规范，其中，Async Module Definition（简称 AMD）便是其中最为成功的一种。AMD 规范在继承了部分`CommonJS`的规范以外，还包含了以下内容

1. 定义全局函数`define(id, dependencies, factory)`，用于定义模块。
2. `id`为模块唯一标示，`dependencies`为模块的前置依赖，`factory`是对象或者函数
3. `factory`如果是函数，模块可以通过以下三种方式对外暴漏 API：`return` 任意类型；`exports.XModule = XModule`、`module.exports = XModule`。
4. `factory`如果是对象，则该对象即为模块的导出值。

2009 年，遵循 AMD 规范的`RequireJS`发布后，迅速得到广大开发者的青睐。但是 AMD 规范的依赖是提前加载，而不管依赖是在函数中什么时候使用，造成了一定的性能消耗，而`CommonJS`规范是延迟加载、就近声明（就近依赖）的特性，不少开发者觉得应该使用这种加载依赖的方式。

#### CMD

2011 年，国内阿里巴巴集团前端开发工程师玉伯（王保平），在给`RequireJS`提出修改意见不断被拒绝后，自己写了`SeaJS`，并提出了 Common Module Definition（简称 CMD）规范，内容上与 AMD 规范相差无几，但是采取了延迟加载、就近声明的特性。`SeaJS`在国内得到了广泛的应用，但是在国外并没有得到大范围的推广。

这时如果同一套代码如果要同时运行在服务端和浏览器端，便会使用不同的模块化方案。

#### UMD

2014 年，美籍华裔 Homa Wong 提出了 Universal Module Definition（简称 UMD）解决方案，UMD 提出了以下内容

1. 优先判断是否存在 `exports` 方法，如果存在，则采用 `CommonJS` 方式加载模块；
2. 其次判断是否存在 `define` 方法，如果存在，则采用 `AMD` 方式加载模块；

UMD 解决方案使得经过 webpack 打包之后的代码具有跨平台运行的特性，成为更加通用的前端模块化解决方案。

### 2. 如何处理使用不同前端模块化方案的依赖

由于历史遗留原因，当我们使用 npm 导出的模块时，会发现模块可能时采用类`CommonJS`方案进行导出的。但自从 ESM 出现之后，很多开发者已经开始使用 ESM 的规范来导入模块。

如果要我们在使用模块之前，一个个查看模块使用不同的导入模块的方法，必定是非常麻烦的。这时候，常用于浏览器版本兼容的编译工具 babel 出场三下五除二解决了这个问题。

babel 对使用 ESM 规范的模块统一编译成 AMD 规范的模块，这样我们在项目中混用`import` 与 `require`，不用对应依赖模块的导出方法，也不会出现问题了。

babel 会给 ESM 规范导出的模块对象一个`__esModule`的标示，用以标示使用 ESM 规范导出的模块。导入时，会判断是否存在`__esModule`属性，以对 ESM 规范和 AMD 规范导出的模块分别做不同的处理。

### 3. 为什么需要前端模块化

JavaScript 早期作为一门轻量级脚本语言，并没有依赖管理的概念，当时只用于 Web 上与用户的少量交互使用。随着 Web 的发展，JavaScript 书写的代码越来越复杂，全局变量冲突，自身没有依赖管理的问题大大困扰着开发者们。

在官方提供 ESM 规范之前，开发者们推出了自己的模块化规范解决方案，这些规范提供了 ESM 实现思路，推动了 JavaScript 语言的发展，也造成了多种模块化并存的情况。对于现在的开发者来说，由于 babel 这类的编译工具的存在，可以不用关心模块化的兼容问题。

# 模块化方案使用指南

我们知道 AMD 和 CMD 设计的规范有一点不一样，也知道 ESM 作为官方的模块化解决方案应该是面向未来的，那它们的在使用上到底有什么不一样呢？

## CommonJs

前面说到`CommonJs`是用于`nodeJs`模块化解决方案，这里就不自己写使用了，通过webpack可以对代码进行打包，打包时可以选择打包出来的库模块化方案。简单的`npm -init` 和安装`webpack webpack-cli`依赖包之后，在`webpack.config.js`文件里进行 webpack 配置，配置如下
```JavaScript
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "./index.js",
    libraryTarget: "commonjs2",
    library: "add",
  },
};
```

这里定义了入口文件为src目录下的index文件，然后创建该文件并如下书写
```JavaScript
// ./src/index.js
let a = require("./a-module");
module.exports = a(1, 2);
```

index文件引入了当前文件夹的a-module文件，我们创建并如下书写
```JavaScript
// ./src/a-module.js
module.exports = function add(a, b) {
  return a + b;
};

```

以上，我们完成了webpack打包配置，一个输出的a模块文件，一个引入了a模块的文件，并再次导出为模块的index文件。

接下来，运行`npx webpack`进行打包，打包出来的文件便是使用`commonJs`进行模块化的文件,对打包文件进行格式化后如下：

```JavaScript
// ./dist/index.js
module.exports.add = (() => {
  var r = {
      300: (r) => {
        r.exports = function (r, t) {
          return r + t;
        };
      },
      138: (r, t, e) => {
        let o = e(300);
        r.exports = o(1, 2);
      },
    },
    t = {};
  return (function e(o) {
    if (t[o]) return t[o].exports;
    var n = (t[o] = { exports: {} });
    return r[o](n, n.exports, e), n.exports;
  })(138);
})();

```

可以看到模块导出标志为`module.export`，导入这里看不到，在`commonJs`中，导入是使用`require`关键字进行导入

## RequireJs/AMD

AMD 是 RequireJS 实践中总结的规范，RequireJS 的使用

- 定义模块：`define(id?, dependencies?, factory) `
- 引入模块：`require([module], factory)`

`define`定义模块时提供了很多方式

1. `id`作为第一个参数，代表模块名，可以不填写，默认使用文件名
2. `dependencies`作为第二个参数，代表依赖模块，如果没有依赖模块，可以不填，默认依赖模块有`[require,exports,module]`
3. `factory`作为一个函数，其参数顺序和值与`dependencies`中的依赖模块导出的值一一对应

根据上面的特性，可以书写以下导出模块的例子

```JavaScript
// 模块a.js，没有依赖模块，导出字符串'a'
define(function(require, exports, module) {
    // 可以直接return, 也可以使用exports/module.exports导出对象
    return 'a';
})

// 模块b.js，没有依赖模块，导出对象{ name: 'b' }
define(function(require, exports, module) {
    // 可以直接return, 也可以使用exports/module.exports导出对象
    // exports.name = 'b';
    // module.exports = { name: 'b' };
    return { name: 'b' };
})

// 模块c.js，有依赖模块a，导出对象{ name: 'c' }
define(['./a.js'], function(a) {
    // 函数参数与依赖模块相关
    console.log(a);
    return { name: 'c' };
})
// 模块c.js的另一种写法，使用require引入
define(function(require, exports, module) {
    const a = require('./a.js');
    console.log(a);
    module.exports = { name: 'c' };
})

// 模块d.js，有依赖模块b，导出对象{ name: 'd' }
define(['./b.js'], function(b) {
    // 函数参数与依赖模块相关
    console.log(b.name);
    return { name: 'd' };
})

```

导入模块如下

```JavaScript
// index.js
require(['./a.js', './b.js'], function(a, b) {
    console.log(a, b.name);
})
```

## SeaJS/CMD

- 定义模块：`define(factory)`
- factory 有三个函数参数：`require`, `exports`, `module`
- 暴露 api 方法可以使用与 RequireJs 一致，若是未暴露，则返回`{}`，RequireJs 返回`undefined`
- 会调用`factory`的`toString`方法对其进行正则匹配以此分析依赖，所以注释中的`require`也会被误解为需要引入的依赖
- 依赖会预先下载但延迟执行

下面看看例子

```JavaScript
// 模块e.js，依赖a.js和b.js，我们假设a、b模块都已经使用CMD重写
define(function(require, exports, module) {
    var a = require('./a.js');
    console.log(a);
    var b = require('./b.js');
    console.log(b.name);
})
```

> 这样看来，模块 e 的写法在 RequireJs 中也可以写成一摸一样的代码，是否意味着 RequireJs 是可以延迟加载、就近声明（就近依赖）的呢？答案是不是的，还记得 var 的变量提升吗？RequireJs 中使用`require`导入的模块也会进行提升，最后执行结果是提前加载。

## UMD

使用webpck对上文commonjs使用指南重新进行打包，打包方式为umd，打包后的文件为
```JavaScript
!(function (e, t) {
  // 判断是否能使用commonjs模块化方案
  "object" == typeof exports && "object" == typeof module
    // 使用commonjs模块化方案，供nodejs使用
    ? (module.exports = t())
    // 判断是否能使用amd规范
    : "function" == typeof define && define.amd
    // 使用requireJs模块化解决方案
    ? define([], t)
    // 其他，导出形式符合commonjs规范
    : "object" == typeof exports
    // 符合commonjs规范
    ? (exports.add = t())
    // 挂在self上
    : (e.add = t());
})(self, function () {
  return (
    (e = {
      300: (e) => {
        e.exports = function (e, t) {
          return e + t;
        };
      },
      138: (e, t, o) => {
        let r = o(300);
        e.exports = r(1, 2);
      },
    }),
    (t = {}),
    (function o(r) {
      if (t[r]) return t[r].exports;
      var n = (t[r] = { exports: {} });
      return e[r](n, n.exports, o), n.exports;
    })(138)
  );
  var e, t;
});
```


## ESM

ESM 相信大家都用过，这里就不赘述了，但是前文中说过

> babel 对使用 ESM 规范的模块统一编译成 AMD 规范的模块

但是随着时间的推移，无论是 AMD 还是 UMD 终将成为历史，而 ESM 才是面向未来的。Vue 的作者尤大大正在鼓捣的工具`vite`,是一个基于浏览器原生 ESM 的开发服务器。有兴趣的可以在下方的参考链接中查看。

# 结束语

差异总结

- AMD 与 CMD：

  - AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
  - CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。
  - CMD 推崇依赖就近，AMD 推崇依赖前置。

- ES Module 与 CommonJS:

  - CommonJS 模块是对象，是运行时加载，运行时才把模块挂载在 exports 之上（加载整个模块的所有），加载模块其实就是查找对象属性。
  - ES Module 不是对象，是使用 export 显示指定输出，再通过 import 输入。此法为编译时加载，编译时遇到 import 就会生成一个只读引用。等到运行时就会根据此引用去被加载的模块取值。所以不会加载模块所有方法，仅取所需。
  - CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
  - CommonJS 模块是运行时加载，ES6 模块是编译时输出接口

- CommonJS 与 AMD/CMD:

  - AMD/CMD 是 CommonJS 在浏览器端的解决方案。
  - CommonJS 是同步加载（代码在本地，加载时间基本等于硬盘读取时间）。
  - AMD/CMD 是异步加载（浏览器必须这么做，代码在服务端）

- UMD 与 AMD/CMD
  - UMD（Universal Module Definition）是 AMD 和 CommonJS 的糅合，跨平台的解决方案。
  - AMD 模块以浏览器第一的原则发展，异步加载模块。
  - CommonJS 模块以服务器第一原则发展，选择同步加载，它的模块无需包装(unwrapped modules)。
  - UMD 先判断是否支持 Node.js 的模块（exports）是否存在，存在则使用 Node.js 模块模式。再判断是否支持 AMD（define 是否存在），存在则使用 AMD 方式加载模块。

---

_参考文献_

- _[AMD , CMD, CommonJS，ES Module，UMD](https://juejin.im/post/6844903663404580878)_
- _[《编程时间简史系列》JavaScript 模块化的历史进程](https://segmentfault.com/a/1190000023017398)_
- _[是什么让尤大选择放弃 webpack?](https://mp.weixin.qq.com/s/mb20GHGeDJF3bGuIhJaK1Q)_
