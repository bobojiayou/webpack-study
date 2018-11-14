#   第一步,从bound.js开始
 先通过bound.js猜想webpack做了哪些事情,
###  源码
add.js
```js
module.exports = function(a, b){
    return a + b;
}
```
index.js
```js
const add = require('./src/add');
console.log(add(10, 9));
```
###  配置
```js
const path = require('path');
module.exports = {
    entry: path.resolve(__dirname, 'index.js'),
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'bound.js'
    },
    mode: "development"
}
```
### 打包文件
 打包出来的bound.js对于现有的打包场景, 真正有用的就是以下这些内容
```js
(function (modules) { // webpackBootstrap
  // The require function
  function __webpack_require__(moduleId) {
    // Create a new module (and put it into the cache)
    var module = {
      i: moduleId,
      l: false,
      exports: {}
    };
    // Execute the module function
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
  }
  // Load entry module and return exports
  return __webpack_require__(__webpack_require__.s = "./index.js");
})

({
  "./index.js": (function (module, exports, __webpack_require__) {
    eval("const add = __webpack_require__(/*! ./src/add */ \"./src/add.js\");\nconsole.log(add(10, 9));\n\n\n//# sourceURL=webpack:///./index.js?");
  }),
  "./src/add.js": (function (module, exports) {
    eval("module.exports = function(a, b){\n    return a + b;\n}\n\n//# sourceURL=webpack:///./src/add.js?");
  })
});
```
如上所示,webpack打包出来的代码是一个自执行函数（可以称之为启动函数）,这个函数接受的参数 即为modules对象,key为moduleId,value为一个函数。这个函数中包含处理过的源码,源码各模块引入实现都被替换成了传入的__webpack_require__函数；启动函数接受所有的模块,并挨个执行传入的模块对象的第二个参数(fn),并向其传入__webpack_require__方法

### 总结 
通过bound.js文件,大概可以推理出
1. webpack针对各种打包形式应该有多种模板去生成打包文件,打包文件是一个自执行函数,该函数实现了__webpack_require__去处理模块依赖
2. webpack通过配置的入口文件,进行依赖模块收集,并将这些模块处理成{moduleId: fn}的形式,fn中包含处理过的源码,目前可见的就是将require替换成了已经实现并传入的__webpack_require__,然后从入口文件开始,依次执行各模块代码



