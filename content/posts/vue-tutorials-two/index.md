---
title: Vue介绍及教程二
categories:
- JavaScript
tags: 
- Vue
date: "2018-11-28"
ShowToc: true
TocOpen: false
---

# Vue 教程二

上篇讲了通过`script`方式引入vue来开发项目，这篇会结合`webpack`进行vue的模块化开发，读完本篇你会知道如何进行vue的模块化开发。

首先需要了解下vue的单文件组件`SFC`及webpack的vue插件`vue-loader`:

## 单文件组件（SFC）和Vue Loader

当使用基于HTML模板的vue组件时，会带来些困难：要么使用 JavaScript 的模板字符串，要么将模板和组件定义写到不同文件里或使用内联模板
***例1:***

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```

_模板写在组件定义里_

> 使用 JavaScript 的模板字符串时，它们在 IE 下并没有被支持，所以如果你需要在不 (经过 Babel 或 TypeScript 之类的工具) 编译的情况下支持 IE，要么使用折行转义字符取而代之

***例2:***

```html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

```js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

_如上是把模板定义在html里，然后js引用_

在这些情况下导致代码不好维护，Vue提供了单文件组件（Single FIle Components）来解决这个问题，将模板、组件定义、CSS写在一个`.vue`文件里

```html
//MyComponent.vue
<template>
  <div id="my-component">...</div>
</template>

<script>
  export default {
      ...
  }
</script>

<style>
  #my-component {
      ...
  }
</style>
```

webpack的loader插件`vue-loader`，会解析单文件组件，提取每个语言块，如有必要会通过其它 loader 处理，然后将他们组装成一个 CommonJS 模块，最后module.exports 出一个 Vue.js 组件对象。

[详解 SFC 与 vue-loader](https://segmentfault.com/a/1190000015708749)


## Learn Vue2: Step By Step 2

### Webpack &WDS & HMR

#### 安装

新建并初始化`testApp`项目：

```js
mkdir testApp && cd testApp && npm init -y
```

安装vue:

```js
npm i --save vue
```

安装webpack及loaders:

```js
npm i -D webpack webpack-cli
```

安装vue相关的loader：

```js
npm i -D vue-loader vue-template-compiler css-loader style-loader file-loader
```

安装babel以支持es2015语法：

```js
npm i -D babel-loader @babel/core@^7.0.0 @babel/preset-env
```

安装webpack插件：

```js
npm i -D html-webpack-plugin
```

安装webpack-dev-server:

```js
npm i -D webpack-dev-server
```

> 以上除了vue安装，其它可合并成一条命令执行：npm i -D webpack webpack-cli vue-loader vue-template-compiler css-loader style-loader file-loader babel-loader @babel/core@^7.0.0 @babel/preset-env html-webpack-plugin webpack-dev-server

#### 配置

新建webpack配置文件，项目代码目录src及入口文件main.js、vue单文件组件App.vue:

```js
touch webpack.config.js && mkdir -p src/components && touch src/main.js src/App.vue src/index.html src/components/Message.vue
```

```js
//webpack.config.js
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');

module.exports = {
    mode: 'development',
    entry: './src/main.js', // 使用的是相对路径
    output: {
        filename: '[name].bundle.js'
    },
    module: {
        rules: [
            {test: /\.vue$/, loader: 'vue-loader'},
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            },
            {
                test: /\.css$/,
                loader: ['style-loader', 'css-loader']
            },
            {
                test: /\.png|jpg|gif|svg$/,
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]?[hash]'
                }
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            hash: true,
            title: 'My Vue App'
        }),
        new webpack.HotModuleReplacementPlugin()
    ],
    devServer: {
        contentBase: './dist',
        port: 8888,
        hot: true
    }
}
```

```js
// src/main.js
import Vue from 'vue';
import App from './App.vue';


new Vue({
    el: '#app',
    render: h=>h(App)
})
```

```vue
// src/App.vue
<template>
    <div id="app">
        <message>Hello World!</message>
        <message>Hello World!</message>
    </div>
</template>

<script>
    import Message from './components/Message.vue';
    export default {
        components:{
            Message
        }
    }
</script>

<style>
    #app {
        color: blue;
    }
</style>
```

```vue
// src/components/Message.vue
<template>
    <div class="box">
        <p>
            <slot></slot>
        </p>
    </div>
</template>

<script>
    export default {

    }
</script>

<style>
    .box {
        background: gray;
        padding: 10px;
        border: 1px solid green;
        margin-bottom: 1em;
    }
</style>
```

```html
// src/index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.7.2/css/bulma.min.css">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>

  <body>
    <div id="app"></div>
  </body>
</html>
```

#### 执行
1. 修改package.json的scripts为：

    ```json
    "scripts": {
        "dev": "webpack-dev-server --config webpack.config.js"
    },
    ```

2. 终端执行`npm run dev`

> webpack的配置可参见之前分享的`Webpack使用`教程

### Vue Ajax Requests With Axios

开始介绍`Axios`前，先使用`express`建一个server作为后端接口。

#### 配置testServer后端服务

初始化`testServer`后端服务:

```js
mkdir testServer && cd testServer && touch app.js && npm init -y && npm i -D express
```

```js
//app.js
const express = require('express');
const app = express();
const port = 3000

app.get('/', (req, res) => {
    res.send('Hello World!')
})

app.get('/skills', (req, res) => {
    res.send(['JavaScript', 'Vue', 'HTML5', 'CSS3'])
})

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

执行命令`node app.js`，即可访问后端服务接口地址:`http://localhost:3000`。

#### 安装axios并配置webpack代理

切回`testApp`前端项目，安装`axios`:

```js
npm install axios --save
```

由于前端地址和后端服务地址不在同一域名下，使用axios访问后端接口时，会报跨域错误，所以这里使用`webpack-dev-server`的代理:

```js
//webpack.config.js
...
    devServer: {
        contentBase: './dist',
        port: 8888,
        hot: true,
        proxy: {
            '/api': {
                target: 'http://localhost:3000',
                pathRewrite: {'^/api':''}
            }
        }
    }
...
```

使用`axios`：

```js
// main.js
import Vue from 'vue';
import App from './App.vue';
import axios from 'axios';

// 方便全局使用，而不需要每次都引入
window.axios = axios; // or Vue.prototype.$http = axios;

new Vue({
    el: '#app',
    render: h => h(App)
})
```

```vue
// components/Message.vue
<template>
    <div class="box">
        <p>
            <slot></slot>
        </p>

        <ul>
            <li v-for="skill in skills">{{ skill }}</li>
        </ul>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                skills: []
            }
        },
        mounted(){
            axios.get('/api/skills').then(res=>this.skills = res.data)
            // this.$http.get('/api/skills').then(res=>this.skills = res.data)
        }
    }
</script>

<style>
    .box {
        background: gray;
        padding: 10px;
        border: 1px solid green;
        margin-bottom: 1em;
    }
</style>
```

### Object-Oriented Forms: Part 1

#### 需求

1. 实现添加项目表单，项目字段：name和description
2. 表单提交，返回错误时，对应输入框下给错误提示
3. 纠正字段错误时，删除对应输入框下的错误提示
4. 表单提交报错时，disable按钮


#### 新增服务端接口

1. testServer新增 `body-parse`包，用来解析post数据:

    ```js
    npm i body-parse --save
    ```
2. 添加新增项目接口

    ```js
    // app.js
    ...
    const bodyParser = require('body-parser');
    app.use(bodyParser.json()); // for parsing application/json
    ...
    app.post('/projects', (req, res) => {
        let errors = {};
        if (!req.body.name){
            errors.name = ["The name field is required."];
        }
        if (!req.body.description){
            errors.description = ["The description field is required."];
        }

        if (Object.keys(errors).length>0){
            res.status(400).send(errors);
        }else{
            res.send({ message: 'Add success' });
        }
    })
    ...
    ```

#### 实现
新建MyForm组件:

```vue
// components/MyForm.vue
<template>
    <div>

        <form method="POST" @submit.prevent="onSubmit" @keydown="errors.clear($event.target.name)">

            <div class="control">
                <label for="name" class="label">Project Name:</label>

                <input type="text" id="name" name="name" class="input" v-model="name">

                <span class="help is-danger" v-if="errors.has('name')" v-text="errors.get('name')"></span>
            </div>

            <div class="control">
                <label for="description" class="label">Project Description:</label>

                <input type="text" id="description" name="description" class="input" v-model="description">

                <span class="help is-danger" v-if="errors.has('description')" v-text="errors.get('description')"></span>
            </div>

            <div class="control">
                <button class="button is-primary" :disabled="errors.any()">Create</button>
            </div>

        </form>

    </div>

</template>

<script>
    class Errors{
        constructor(){
            this.errors = {};
        }

        get(field){
            if (this.errors[field]){
                return this.errors[field][0];
            }
        }

        record(errors){
            this.errors = errors;
        }

        clear(field){
            delete this.errors[field];
        }

        has(field){
            return this.errors.hasOwnProperty(field);
        }

        any(){
            return Object.keys(this.errors).length > 0;
        }
    }

    export default {
        data(){
            return {
                name: '',
                description: '',
                errors: new Errors()
            }
        },
        methods: {
            onSubmit(){
                axios.post('/api/projects', {name: this.name, description: this.description})
                    .then(res => this.onSuccess())
                    .catch(error => this.errors.record(error.response.data))
            },

            onSuccess(response){
                alert(response.data.message);
                this.name = '';
                this.description = '';
            }
        }
    }
</script>

<style>
    .control {
        margin-bottom: 10px;
    }
</style>
```

引入MyForm组件:

```vue
// App.vue
<template>
    <div class="app container">
        <message>Hello World!</message>
        <my-form></my-form>
    </div>
</template>

<script>
    import Message from './components/Message.vue';
    import MyForm from './components/MyForm.vue';
    export default {
        components:{
            Message,
            MyForm
        }
    }
</script>

<style>
    .app {
        color: blue;
    }
</style>
```

### Object-Oriented Forms: Part 2

接着上节，我们给MyForm组件实现一个Form对象：
​    1. 封装表单字段和操作
​    2. 把错误对象封装在Form对象中

```vue
// components/MyForm.vue
<template>
    <div>
        <form method="POST" @submit.prevent="onSubmit" @keydown="form.errors.clear($event.target.name)">

            <div class="control">
                <label for="name" class="label">Project Name:</label>

                <input type="text" id="name" name="name" class="input" v-model="form.name">

                <span class="help is-danger" v-if="form.errors.has('name')" v-text="form.errors.get('name')"></span>
            </div>

            <div class="control">
                <label for="description" class="label">Project Description:</label>

                <input type="text" id="description" name="description" class="input" v-model="form.description">

                <span class="help is-danger" v-if="form.errors.has('description')" v-text="form.errors.get('description')"></span>
            </div>

            <div class="control">
                <button class="button is-primary" :disabled="form.errors.any()">Create</button>
            </div>

        </form>

    </div>

</template>

<script>

    class Form {
        constructor(data){
            this.originalData = data;

            for(let field in data){
                this[field] = data[field];
            }

            this.errors = new Errors();
        }

        reset(){
            for(let field in this.originalData){
                this[field] = '';
            }
            this.errors.clear();
        }

        data(){
            let data = {};

            for(let property in this.originalData){
                data[property] = this[property]
            }
            return data;
        }

        submit(requestType, url){
            axios[requestType](url, this.data())
                    .then(this.onSuccess.bind(this))
                    .catch(this.onFail.bind(this))
        }

        onSuccess(response){
            alert(response.data.message);
            this.reset();
        }

        onFail(error){
            this.errors.record(error.response.data)
        }
    }

    class Errors{
        constructor(){
            this.errors = {};
        }

        get(field){
            if (this.errors[field]){
                return this.errors[field][0];
            }
        }

        record(errors){
            this.errors = errors;
        }

        clear(field){
            if(field){
                delete this.errors[field];
                return;
            }
            this.errors = {};
        }

        has(field){
            return this.errors.hasOwnProperty(field);
        }

        any(){
            return Object.keys(this.errors).length > 0;
        }
    }

    export default {
        data(){
            return {
                form: new Form({
                    name: '',
                    description: ''
                })
            }
        },
        methods: {
            onSubmit(){
                this.form.submit('post', '/api/projects')
            }
        }
    }
</script>

<style>
    .control {
        margin-bottom: 10px;
    }
</style>
```

### Object-Oriented Forms: Part 3

接着上节，我们需要:

1. 改造onSubmit方法，使用promise对象来支持链式调用
2. form对象添加post、put、delete方法
3. 把`Errors`和`Form`分别单独成`Errors.js`和`Form.js`

```vue
// components/MyForm.vue
<template>
    <div>
        <form method="POST" @submit.prevent="onSubmit" @keydown="form.errors.clear($event.target.name)">

            <div class="control">
                <label for="name" class="label">Project Name:</label>

                <input type="text" id="name" name="name" class="input" v-model="form.name">

                <span class="help is-danger" v-if="form.errors.has('name')" v-text="form.errors.get('name')"></span>
            </div>

            <div class="control">
                <label for="description" class="label">Project Description:</label>

                <input type="text" id="description" name="description" class="input" v-model="form.description">

                <span class="help is-danger" v-if="form.errors.has('description')" v-text="form.errors.get('description')"></span>
            </div>

            <div class="control">
                <button class="button is-primary" :disabled="form.errors.any()">Create</button>
            </div>

        </form>

    </div>

</template>

<script>
    import Form from '../core/Form';

    export default {
        data(){
            return {
                form: new Form({
                    name: '',
                    description: ''
                })
            }
        },
        methods: {
            onSubmit(){
                this.form.post('/api/projects')
                        .then(data => console.log(data))
                        .catch(error => console.log(error))
            }
        }
    }
</script>

<style>
    .control {
        margin-bottom: 10px;
    }
</style>
```

```js
// core/Form.js
import Errors from './Errors';

class Form {
    constructor(data) {
        this.originalData = data;

        for (let field in data) {
            this[field] = data[field];
        }

        this.errors = new Errors();
    }

    reset() {
        for (let field in this.originalData) {
            this[field] = '';
        }
        this.errors.clear();
    }

    data() {
        let data = {};

        for (let property in this.originalData) {
            data[property] = this[property]
        }

        return data;
    }

    post(url) {
        return this.submit('post', url);
    }

    put(url) {
        return this.submit('put', url);
    }

    delete(url) {
        return this.submit('delete', url);
    }

    submit(requestType, url) {

        return new Promise((resolve, reject) => {
            axios[requestType](url, this.data())
                .then(response => {
                    this.onSuccess(response.data);

                    resolve(response.data);
                })
                .catch(error => {
                    this.onFail(error.response.data);

                    reject(error.response.data);
                })
        });
    }

    onSuccess(data) {
        alert(data.message);
        this.reset();
    }

    onFail(errors) {
        this.errors.record(errors)
    }
}

export default Form;
```

```js
// core/Errors.js
class Errors {
    constructor() {
        this.errors = {};
    }

    get(field) {
        if (this.errors[field]) {
            return this.errors[field][0];
        }
    }

    record(errors) {
        this.errors = errors;
    }

    clear(field) {
        if (field) {
            delete this.errors[field];
            return;
        }
        this.errors = {};
    }

    has(field) {
        return this.errors.hasOwnProperty(field);
    }

    any() {
        return Object.keys(this.errors).length > 0;
    }
}

export default Errors;
```

### Custom Input Components

新建并实现一个输入框组件，能在组件上使用`v-model`指令

```vue
// components/CouponInput.vue

<template>
    <input type="text" :value="value" @input="updateCoupon($event.target.value)" ref="input">
</template>

<script>
export default {
    props: ['value'],
    methods: {
        updateCoupon(data){

            if(data == 'ALLFREE'){
                alert('This coupon is no longer valid. Sorry!');
                this.$refs.input.value = data = '';
            }

            this.$emit('input', data);
        }
    }
}
</script>

<style>

</style>
```

这里要注意`ref`的使用。

`App`组件添加对`CouponInput`组件的使用:

```vue
// App.vue
<template>
    <div class="app container">
        <message>Hello World!</message>
        <my-form></my-form>

        <coupon-input v-model="coupon"></coupon-input>

    </div>
</template>

<script>
    import Message from './components/Message.vue';
    import MyForm from './components/MyForm.vue';
    import CouponInput from './components/Couponinput.vue';

    export default {
        data(){
            return {
                coupon: 'FREEBIE'
            }
        },
        components:{
            Message,
            MyForm,
            CouponInput
        }
    }
</script>

<style>
    .app {
        color: blue;
    }
</style>
```

### Vue SPA Essentials: Routing

1. 安装vue-router: `npm install -D vue-router`
2. 新增`routes.js`文件，用来配置路由
3. 新建views文件夹，并在文件里添加`Home.vue`和`About.vue`组件

```js
// routes.js
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);

let routes = [
    {
        path: '/',
        component: require('./views/Home.vue').default
    },
    {
        path: '/about',
        component: require('./views/About.vue').default
    }
]

export default new VueRouter({
    routes,
    linkActiveClass: 'is-active',
    linkExactActiveClass: 'is-exact-active'

})
```

Home和About组件模板使用了[bulma section](https://bulma.io/documentation/layout/section/)

```vue
// views/Home.vue
<template>
    <section class="section">
        <div class="container">
            <h1 class="title">Home</h1>
            <h2 class="subtitle">
                A simple container to divide your page into <strong>sections</strong>, like the one you're currently reading
            </h2>
        </div>
    </section>
</template>

<script>
export default {

}
</script>
```

```vue
// views/About.vue
<template>
    <section class="section">
        <div class="container">
            <h1 class="title">About</h1>
            <h2 class="subtitle">
                A simple container to divide your page into <strong>sections</strong>, like the one you're currently reading
            </h2>
        </div>
    </section>
</template>

<script>
export default {

}
</script>
```

修改`App.vue`，添加路由组件; 这里用了[bulma navbar](https://bulma.io/documentation/components/navbar/)

```vue
<template>
    <div class="app container">
        <message>Hello World!</message>
        <my-form></my-form>
        <coupon-input v-model="coupon"></coupon-input>


        <nav class="navbar is-primary" role="navigation" aria-label="main navigation">
            <div class="navbar-brand">
                <a class="navbar-item" href="https://bulma.io">
                    <img src="https://bulma.io/images/bulma-logo.png" width="112" height="28">
                </a>

                <a role="button" class="navbar-burger burger" aria-label="menu" aria-expanded="false" data-target="navbarBasicExample">
                    <span aria-hidden="true"></span>
                    <span aria-hidden="true"></span>
                    <span aria-hidden="true"></span>
                </a>
            </div>

            <div id="navbarBasicExample" class="navbar-menu">
                <div class="navbar-start">
                    <router-link class="navbar-item" to="/" exact>Home</router-link>
                    <router-link class="navbar-item" to="/about">About</router-link>
                </div>

            </div>
        </nav>

        <router-view></router-view>

    </div>

</template>

<script>

    import Message from './components/Message.vue';
    import MyForm from './components/MyForm.vue';
    import CouponInput from './components/Couponinput.vue';

    export default {
        data(){
            return {
                coupon: 'FREEBIE'
            }
        },
        components:{
            Message,
            MyForm,
            CouponInput
        }
    }
</script>

<style>
    .app {
        color: blue;
    }
    nav {
        margin-top: 20px;
    }
</style>
```

### SPAs and the Backend

这节介绍如何组织接口调用。

在`testServer`中新增如下接口：

```js
// app.js
...
app.get('/statuses', (req, res) => {
    res.send([
        {
            name: 'Lee Hui',
            body: 'It\'s just a test'
        },
        {
            name: 'Kevin Wen',
            body: 'It\'s just a test'
        }
    ])
})
...
```

在`testApp`中修改`Home.vue`组件

```vue
// Home.vue
<template>
    <section class="section">

        <article class="message" v-for="(status, index) in statuses" :key="index">
            <div class="message-header">
                <p>{{ status.name }}</p>
                <button class="delete" aria-label="delete"></button>
            </div>
            <div class="message-body">
                {{ status.body }}
            </div>
        </article>

    </section>
</template>

<script>
import Status from '../models/Status.js';

export default {
    data(){
        return {
            statuses: []
        }
    },

    mounted(){
        Status.all(statuses => this.statuses = statuses)
    }
}
</script>
```

新增models文件夹，并添加`Status.js`模块

```js
// Stauts.js
class Status {

    static all(then){
        return axios.get('/api/statuses')
                    .then(({data}) => then(data))
    }
}

export default Status
```

### Vue Filters

### Dedicated Form Components

### Testing Vue: Part 1

### Vue Component Responsibility

### Vue Subclassing

### Scoped Slots

### [翻译](https://laracasts.com/series/learn-vue-2-step-by-step)

[learn-vue-2-step-by-step](https://laracasts.com/series/learn-vue-2-step-by-step)