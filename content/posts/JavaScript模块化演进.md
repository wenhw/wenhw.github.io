---
title: JavaScript模块化演进
categories:
- JavaScript
tags: 
- JavaScript
- 模块化
date: "2018-10-24"
ShowToc: true
TocOpen: true
---

# JavaScript模块化演进

## 什么是模块化

就是把原来聚合在一起的各类代码，按功能、职责（逻辑层、数据层、测试）等划分，分散到不同的模块文件中，分离关注点

## 使用模块的好处

模块化可以降低代码（功能）之间耦合，同时具有以下好处：

1. 命名空间：可以避免命名（变量、函数、类名等重名）污染，减少不必要的全局变量
2. 代码易维护和扩展：设计良好的模块，彼此间会尽量减少依赖，方便对其独立改造或扩展，比起一坨代码，维护独立的模块肯定轻松的多
3. 代码复用：跟复制粘贴说拜拜，因为你有个可重复使用的模块

<!--more-->

## 演进

使用一个简单的 web 应用程序来演示模块的概念，应用程序在浏览器中显示数组的和，程序由4个函数和一个index.html文件组成

函数依赖关系如下：

`main -> sum -> add和reduce`

main函数对数组求和，并把结果显示在html的span标签上；sum函数依赖add和reduce函数，看下面代码

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is<span id="answer"></span>
    </h1>
</body>
</html>
```

```javascript
// main
var values = [1, 2, 4, 5, 6, 7, 8, 9];
var answer = sum(values)
document.getElementById("answer").innerHTML = answer;
```

```javascript
// sum
function sum(arr) {
    return reduce(arr, add);
}
```

```javascript
// add
function add(a, b) {
    return a + b;
}
```

```javascript
// reduce
function reduce(arr, add) {
    var i = 0,
        memo = 0,
        length = arr.length;
    for (i; i < length; i++) {
        memo = add(memo, arr[i])
    }
    return memo;
}
```
我们来看下如何把这些代码整合到一起，来构建应用程序

### 内嵌脚本

内嵌脚本就是在 <script></script> 标签之间添加 JavaScript 代码。这是我们开始学 JavaScript 时的做法

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is <span id="answer"></span>
    </h1>
</body>
<script>

    function add(a, b) {
        return a + b;
    }

    function reduce(arr, add) {
        var i = 0,
            memo = 0,
            length = arr.length;
        for (i; i < length; i++) {
            memo = add(memo, arr[i])
        }
        return memo;
    }

    function sum(arr) {
        return reduce(arr, add);
    }

    // main
    var values = [1, 2, 4, 5, 6, 7, 8, 9];
    var answer = sum(values)
    document.getElementById("answer").innerHTML = answer;

</script>
</html>
```
这是一个很好的入门办法。没有外部文件或依赖关系需要担心。但是这也导致了不可维护的代码，因为：

* **代码缺乏可重用性**：如果另一个页面需要这些函数，我们就不得不复制粘贴代码。
* **缺乏依赖解析**：你必须保证 main 函数之前已经添加了 add、reduce 和 sum 函数。
* **全局命名空间污染**：所有的函数和变量将都将驻留在全局作用域中。

### script标签引入js文件

将大段的 JavaScript 分成更小的代码片段，并用  `<script src="">` 标签加载它们。

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is: <span id="answer"></span>
    </h1>
    <script type="text/javascript" src="./add.js"></script>
    <script type="text/javascript" src="./reduce.js"></script>
    <script type="text/javascript" src="./sum.js"></script>
    <script type="text/javascript" src="./main.js"></script>
</body>
</html>
```
```javascript
// add.js
function add(a, b) {
    return a + b;
}
```
```javascript
// reduce.js
function reduce(arr, add) {
    var i = 0,
        memo = 0,
        length = arr.length;
    for (i; i < length; i++) {
        memo = add(memo, arr[i])
    }
    return memo;
}
```
```javascript
// sum.js
function sum(arr) {
    return reduce(arr, add);
}
```
```javascript
// main.js
var values = [1, 2, 4, 5, 6, 7, 8, 9];
var answer = sum(values)
document.getElementById("answer").innerHTML = answer;
```
通过将文件分成多个 js 文件，我们可以重用这些代码。我们不再需要在不同的 html 页面之间复制和粘贴代码。我们只需要将该文件用 script 标签加载就可以了。尽管这是更好的方法，但仍然有以下问题：

* **缺乏依赖解析**：文件的顺序很重要。你必须保证在加载 main.js 文件之前已经加载了  add.js、reduce.js 和 sum.js 文件。
* **全局命令空间污染**：所有的函数和变量依然在全局作用域中。

### 模块对象 和 模块模式(IIFE)

通过使用模块对象和 立即调用的函数表达式(IIFE) ，我们可以减少对全局作用域的污染。在这种方法中，我们只向全局作用域公开一个对象。该对象包含了我们在应用程序中需要的所有方法和值。在本例中，我们只向全局作用域公开了 myApp 对象。所有的函数都将被保存在 myApp 对象中。

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is: <span id="answer"></span>
    </h1>
    <script type="text/javascript" src="./my-app.js"></script>
    <script type="text/javascript" src="./add.js"></script>
    <script type="text/javascript" src="./reduce.js"></script>
    <script type="text/javascript" src="./sum.js"></script>
    <script type="text/javascript" src="./main.js"></script>
</body>
</html>
```
```javascript
// my-app.js
var myApp = {};
```
```javascript
// add.js
(function() {
    myApp.add = function(a, b) {
        return a + b;
    }
})();
```
```javascript
// reduce.js
(function() {
    myApp.reduce = function(arr, add) {
        var i = 0,
            memo = 0,
            length = arr.length;
        for (i; i < length; i++) {
            memo = add(memo, arr[i])
        }
        return memo;
    }
})();
```
```javascript
// sum.js
(function() {
    myApp.sum = function(arr) {
        return myApp.reduce(arr, myApp.add);
    }
})();
```
```javascript
// main.js
(function(app){
    var values = [1, 2, 4, 5, 6, 7, 8, 9];
    var answer = app.sum(values)
    document.getElementById("answer").innerHTML = answer;
})(myApp);
```
通过将每个文件封装到 IIFE 中，所有的本地变量都保留在函数作用域内。因此，函数中的所有变量都将保持在函数作用域内，而不会污染全局作用域。

我们通过将 add、reduce 和 sum 函数附加在 myApp 对象上，从而对外公开它们。通过引用myApp对象来访问这些函数

与前面的例子相比，IIFE 是一个巨大的改进。大多数流行的 js 库，如 jQuery ，都使用这种模式。它公开了一个全局对象 $，所有的函数都在 $ 对象中。

然而，这并不能算是一个完美的解决方案。这种方法仍然面临相同的问题。

* **缺乏依赖解析**：文件的顺序依然重要，myApp.js 必须出现在所有其它文件之前加载，main.js 必须处在所有其它库文件之后。
* **全局命令空间污染**：现在全局变量的数量变成了 1，但是还不是 0 。

### CommonJS

2009年，美国程序员Ryan Dahl创造了node.js项目，将javascript语言用于服务器端编程。因此 CommonJS 诞生了

CommonJS 不是一个 JavaScript 库。它是一个标准化组织。它就像 ECMA 或 W3C 一样。ECMA 定义了 JavaScript 的语言规范。W3C定义了 JavaScript web API ，比如 DOM 或 DOM 事件。 **CommonJS 的目标是为 js在包括web 服务器、桌面和命令行应用程序定义一套通用的 API 。**

> 注：CommonJS制定了一些规范，而Node.js实现了这些规范。**Node.js自身实现了require方法作为其引入模块的方法，同时NPM也基于CommonJS定义的包规范**，实现了依赖管理和模块自动安装等功能。

CommonJS 还定义了模块 API 。因为在服务器应用程序中没有 HTML 页面和 `<script><\script>` 标签，所以为模块提供一些清晰的 API 是很有意义的。模块需要被公开(`export`)以供其它模块使用，并且可以访问(`import`)。它的导出模块语法如下：

```javascript
// add.js
module.exports = function add (a, b) {
    return a + b;
}
```
上述代码定义和输出了一个模块。要使用或导入 add 模块，您需要 require 函数，使用文件名或模块名作为参数。下面的语法描述了如何将一个模块导入到代码中：
```javascript
var add = require('./add');
```

> 进一步了解CommonJS可查看[CommonJS规范](http://javascript.ruanyifeng.com/nodejs/module.html#toc0)

### 异步模块定义(AMD)

CommonJs 规范的问题在于它是同步的。当你调用` var add = require('./add');` 时，系统将暂停，直到模块准备(ready) 完成。这意味着当所有的模块都加载时，这一行代码将冻结浏览器(意思为除了加载该文件，浏览器什么事情也不做)。所以CommonJS规范不适用于浏览器环境

于是有了异步模块定义(AMD)，AMD具有以下格式：
```javascript
// sum.js
define(['add', 'reduce'], function(add, reduce){
	return function(){...};
});
```
define 函数(或关键字)将依赖项列表和回调函数作为参数。回调函数的参数与数组中的依赖是相同的顺序。相当于导入模块。并且回调函数返回一个值，即是你导出的值。

CommonJS 和 AMD 解决了模块模式中剩下的两个问题：**依赖解析** 和 **全局作用域污染** 。我们只需要处理每个模块或每个文件的依赖关系就可以了。并且不再有全局作用域污染。

> 在浏览器可以通过RequireJS(模块加载器)来使用，[参考](http://javascript.ruanyifeng.com/nodejs/module.html#toc4)

### Browserify

RequireJS 和 AMD 解决了我们以前所遇到的所有问题。然而，它也带来了一些不那么严重的问题。

* AMD 的语法过于冗余。
* 数组中的依赖列表必须与函数的参数列表匹配。如果存在许多依赖项，则很难维护依赖项的顺序。如果您的模块中有几十个依赖项，并且如果你不得不在中间删除某个依赖，那么就很难找到匹配的模块和参数。
* 在当前浏览器下（HTTP 1.1），加载很多小文件会降低性能。

由于上述这些原因，有些人想要使用 CommonJS 语法来替换。但 CommonJS 语法是用于服务端，并且是同步的，这时 Browserify 就来解救我们了！通过 Browserify ，你可以在浏览器应用程序中使用 CommonJS 模块。Browserify 是一个 **模块打包器(module bundler)** 。Browserify 遍历代码的依赖树，并将依赖树中的所有模块打包成一个文件。

不同于 RequireJS ，但是 Browserify 是一个命令行工具，需要 NodeJS 和 NPM 来安装它。如果系统中安装了 NodeJS ，就可以用如下命令来安装 Browserify：
```javascript
npm install -g browserify
```

我们用CommonJS语法编写示例应用程序:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is: <span id="answer"></span>
    </h1>
    <script type="text/javascript" src="bundle.js"></script>
</body>
</html>
```
```javascript
// main.js
var sum = require('./sum');
var values = [1, 2, 4, 5, 6, 7, 8, 9];
var answer = sum(values)
document.getElementById("answer").innerHTML = answer;
```
```javascript
// sum.js
var reduce = require('./reduce');
var add = require("./add");

module.exports = function(arr) {
    return reduce(arr, add);
}
```
```javascript
// add.js
module.exports = function add (a, b) {
    return a + b;
}
```
```javascript
// reduce.js
module.exports = function reduce(arr, add) {
    var i = 0,
        memo = 0,
        length = arr.length;
    for (i; i < length; i++) {
        memo = add(memo, arr[i])
    }
    return memo;
}
```
接着执行：
```javascript
browserify main.js -o bundle.js
```
执行上述命令，Browserify会 解析 main.js 中的 require() 函数调用，并遍历项目中的依赖树。然后将依赖树打包到一个bundle.js文件中。

### UMD

在一些同时需要AMD和CommonJS功能的项目中，你需要使用另一种规范：Universal Module Definition（通用模块定义规范）。

UMD创造了一种同时使用两种规范的方法，并且也支持全局变量定义。所以UMD的模块可以同时在客户端和服务端使用。

> [参考](https://github.com/umdjs/umd)

### ES6(es2015) 模块语法

JavaScript 语言中并没有内置模块系统。这就是为什么我们有这么多不同的导入和导出模块(全局模块对象、CommonJS、AMD 和 UMD)的原因。但这种情况最近发生了变化。 ES6 语言规范中，模块是 JavaScript 的一部分。如果想让项目兼容未来，我们需要使用 ES6 模块语法。

```javascript
// main.js
import sum from "./sum";
var values = [ 1, 2, 4, 5, 6, 7, 8, 9 ];
var answer = sum(values);
document.getElementById("answer").innerHTML = answer;
```
```javascript
// sum.js
import add from './add';
import reduce from './reduce';
export default function sum(arr) {
    return reduce(arr, add);
}
```
```javascript
// add.js
export default function add(a, b) {
    return a + b;
}
```
```javascript
//reduce.js
export default function reduce(arr, add) {
    var i = 0,
        memo = 0,
        length = arr.length;
    for (i; i < length; i++) {
        memo = add(memo, arr[i])
    }
    return memo;
}
```

ES6 模块有个不幸的问题。浏览器还没有为完全支持。目前，只有 Chrome 浏览器支持 import 语句。即使大多数浏览器支持 import 和 export ，如果您的应用程序必须支持较老的浏览器，那么您可能会遇到问题。

不过，现在已经有很多工具可以用了，这些工具让我们现在就可以用 ES6 模块语法。

### Webpack

Webpack 是一个 模块打包器(module bundler) 。就像 Browserify 一样，它会遍历依赖树，然后将其打包到一到多个文件。那么问题来了，如果它和 Browserify 一样，为什么我们需要另一个模块打包器呢？Webpack 可以处理 CommonJS 、 AMD 和 ES6 模块。并且 Webpack 还有更多的灵活性和一些很酷的功能特性，比如：

* **代码分离**：当有多个应用程序共享相同的模块时。Webpack 可以将代码打包到两个或更多的文件中。例如，如果您有两个应用程序 app1 和 app2 ，并且都共享许多模块。 使用 Browserify ，你会有 app1.js 和 app2.js ，每个文件都包含所有依赖关系模块。但是使用 Webpack ，您可以创建 app1.js ，app2.js 和 shared-lib.js。是的，您必须从 html 页面加载 2 个文件。但是使用哈希文件名，浏览器缓存和 CDN ，可以减少初始加载时间。
* **加载器**：用自定义加载器，可以加载任何文件到源文件中。用 require() 语法，不仅仅可以加载 JavaScript 文件，还可以加载 CSS、CoffeeScript、Sass、Less、HTML模板、图像，等等。
* **插件**：Webpack 插件可以在打包写入到打包文件之前对其进行操作。有很多社区创建的插件。例如，给打包代码添加注释，添加 Source map，将**打包文件分离成块**等等

Webpack DevServer 是一个开发服务器，它可以检测源代码改变并自动打包源代码，并刷新浏览器。它通过提供代码的即时反馈，从而加快开发过程。

我们用 Webpack 来构建示例应用程序。因为 Webpack 是 JavaScript 命令行工具，所以需要先安装上 NodeJS 和 NPM

```javascript
mkdir -p project/app project/dist && cd project
npm init -y
npm install -D webpack webpack-dev-server webpack-cli
touch webpack.config.js
```
在webpack.config.js中添加如下内容:
```javascript
// webpack.config.js
module.exports = {
    entry: './app/main.js',
    output: {
        filename: './dist/bundle.js'
    }
}
```

打开 package.json 文件，在 script 字段后添加如下行
```javascript
"scripts": {
	"start": "webpack-dev-server --progress --colors",
	"build": "webpack"
},
```
在 project 目录下添加 index.html，在 project/app 目录下添加所有 JavaScript 模块

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JS Modules</title>
</head>
<body>
    <h1>
        The Answer is: <span id="answer"></span>
    </h1>
    <script type="text/javascript" src="./dist/bundle.js"></script>
</body>
</html>
```

```javascript
// app/add.js

module.exports = function add(a, b) {
    return a + b;
}
```
```javascript
// app/reduce.js

module.exports = function reduce(arr, add) {
    var i = 0,
        memo = 0,
        length = arr.length;
    for (i; i < length; i++) {
        memo = add(memo, arr[i])
    }
    return memo;
}
```
```javascript
// app/sum.js

define(['./reduce', './add'], function(reduce, add) {
    var sum = function(arr) {
        return reduce(arr, add);
    };

    return sum;
});
```
```javascript
// app/main.js

var sum = require('./sum');
var values = [1, 2, 4, 5, 6, 7, 8, 9];
var answer = sum(values)
document.getElementById("answer").innerHTML = answer;
```
>注:  add.js 、 reduce.j和main.js 是用 CommonJS 风格写的，而 sum.js 是用 AMD 风格写的。 Webpack 默认是可以处理 CommonJS 和 AMD。如果你用的是 ES6 模块，那就需要安装和配置  babel loader。

运行程序
```javascript
npm start
```

打开浏览器，访问http://localhost:8081/ (端口会变)，可以看到应用；此时编辑模块文件时，浏览器会自动刷新，显示修改后的结果

另外，你可能会注意到，在dist目录里找不到 bundle.js 文件。这是因为 Webpack DevServer 会创建打包文件，但是不会写入到文件系统中，而是放在内存中。

如果要部署，就得创建打包文件。可以通过键入如下命令创建 bundle.js 文件：
```javascript
npm run build
```

## 结语

通过使用构建工具，极大改变了前端项目开发模式，模块化（模块打包器）模糊了nodejs和前端开发边界，使得前端可以用npm的库，促进前端生态发展

## 参考链接
* [JavaScript 模块](http://www.css88.com/archives/7628)
* [JavaScript 模块化入门Ⅰ：理解模块](https://zhuanlan.zhihu.com/p/22890374)
* [JavaScript 模块化入门Ⅱ：模块打包构建](https://zhuanlan.zhihu.com/p/22945985)
* [js模块](https://colobu.com/2014/09/23/JavaScript-Module-Pattern-In-Depth/)
* [es6模块](http://es6.ruanyifeng.com/#docs/module)
* [webpack使用1](https://llp0574.github.io/2016/11/29/getting-started-with-webpack2/)
* [webpack使用](https://github.com/ruanyf/webpack-demos)

## 附

### 模块对象 和 模块模式(IIFE)

使用模块时，我们往往会用到类的概念（但原生JavaScript并不支持类，虽然最新的ES6里引入了Class不过还不普及）这样我们就能把公有和私有方法和变量存储在一个对象中——这就和我们在Java或Python里使用类的一样。这样我们就能在公开调用API的同时，仍然在一个闭包范围内封装私有变量和方法；js的模块模式往往要求整个模块必须在一个文件里，即一个文件就是一个模块

实现模块模式方法有多种（可进行试验）：

1. 匿名闭包函数(也叫立即执行函数（IIFE）)

即创建匿名函数，并立即执行。所有函数内的代码都在闭包内

```javascript
(function(){
    // 1. 这里定义的变量和函数都只在该作用域有效，变量和函数私有
    // 2. 外部访问不到作用域里的内容（变量和函数）
    // 3. 该作用域可访问全局变量和函数
}());
```
通过这种方式，匿名函数有了自己的作用域，这允许我们从父(全局)命名空间隐藏变量，不会意外覆盖全局变量

2. 全局引入

JavaScript有个特性，称为隐性全局。使用变量时，解释器会从作用域（由内而外），一层层找变量声明，直到全局变量。这意味着在匿名函数里使用全局变量很简单，但这也会导致代码难以管理，文件中不容易区分（对人而言）哪个变量是全局的。

我们可以把全局变量作为参数传递给匿名函数，将它们引入我们的代码，让代码清晰可读，同时比隐性全局快

```javascript
(function($, ko){
    // 当前域可访问全局jQuery和ko
}(jQuery, ko))
```

3. 模块出口

把匿名函数的返回值，赋值给全局模块变量，从而输出公开的属性和方法

```javascript
var MODULE = (function(){
    var my = {},
    	privateVariable = 1;

    function privateMethod(){
        // ...
    }

    my.moduleProperty = 1;
    my.moduleMethod = function(){
        // ...
    };

    return my;
}())
```

以上代码声明了一个全局模块MODULE，有两个公开属性和方法，MODULE.moduleProperty、MODULE.moduleMethod。同时，匿名函数的闭包还维持了私有内部状态。通过全局引入，我们很容易引入需要的全局变量，而模块出口，则让我们可以输出属性或方法到全局变量，供外部访问。

### 高级模式

通过以上三种模式的结合，我们可以创造出强大的，可扩展的结构

1. 扩充模块

模块模式的一个限制是整个模块必须在一个文件里。任何人都了解长代码分割到不同文件的必要。还好，我们有很好的办法扩充模块。（在扩充文件）首先我们引入模块（从全局），给他添加属性，再输出他。

```javascript
`m1.js`

var MODULE = (function(){
    var my = {},
    	privateVariable = 1;

    function privateMethod(){
        // ...
    }

    my.moduleProperty = 1;
    my.moduleMethod = function(){
        // ...
    };

    return my;
}())

`m2.js`

var MODULE = (function(m){
    m.anotherMethod = function(){
        // ...
    }
}(MODULE))
```
以上代码执行后，MODULE获得一个新公开方法MODULE.anotherMethod。扩充没有影响原模块的私有内部状态。

2. 松耦合扩充

上面的例子需要我们首先创建模块，然后扩充它，这并不是必要的，可以用如下结构避免；另外使用松耦合扩充，我们可以异步加载脚本，**不需要关心加载顺序**。不过每个文件都要有如下的结构

```javascript
var MODULE = (function(my){
    // 扩充...

    return my;
}(MODULE || {}))
```

3. 紧耦合扩充

虽然松耦合很不错，但模块上也有些限制。最重要的，你不能安全的覆写模块属性（因为没有加载顺序）。初始化时也无法使用其他文件定义的模块属性（但你可以在初始化后运行）。紧耦合扩充意味着**一组加载顺序**，但是允许覆写。下面是一个例子（扩充最初定义的MODULE）：

```javascript
var MODULE = (function(my){
    var old_moduleMethod = my.moduleMethod;

    my.moduleMethod = function(){
        // ...
    }

    return my;
}(MODULE))
```
这里覆写了MODULE.moduleMethod方法

4. 子模块

创建子模块

```javascript
MODULE.sub = (function(){
    var my = {};
    // 多了一级命名空间

    return my;
}())
```
这样子模块有正常的模块功能，包括扩充和私有状态