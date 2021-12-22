## 安装与使用
```
# npm i -D 是 npm install --save-dev 的简写，是指安装模块并保存到 package.json 的 devDependencies
# 安装最新稳定版
npm i -D webpack

# 安装指定版本
npm i -D webpack@<version>

# 安装最新体验版本
npm i -D webpack@beta

npm i -g webpack
```
Webpack 在执行构建时默认会从项目根目录下的 webpack.config.js 文件读取配置，所以你还需要新建它，其内容如下：
```javascript
const path = require('path');

module.exports = {
  // JavaScript 执行入口文件
  entry: './main.js',
  output: {
    // 把所有依赖的模块合并输出到一个 bundle.js 文件
    filename: 'bundle.js',
    // 输出文件都放到 dist 目录下
    path: path.resolve(__dirname, './dist'),
  }
};
```
由于 Webpack 构建运行在 Node.js 环境下，所以该文件最后需要通过 CommonJS 规范导出一个描述如何构建的 Object 对象。
```
|-- index.html
|-- main.js
|-- show.js
|-- webpack.config.js
```
一切文件就绪，在项目根目录下执行 webpack 命令运行 Webpack 构建，你会发现目录下多出一个 dist目录，里面有个 bundle.js 文件， bundle.js 文件是一个可执行的 JavaScript 文件，它包含页面所依赖的两个模块 main.js 和 show.js 及内置的 webpackBootstrap 启动函数。 这时你用浏览器打开 index.html 网页将会看到 Hello,Webpack。

Webpack 是一个打包模块化 JavaScript 的工具，它会从 main.js 出发，识别出源码中的模块化导入语句， 递归的寻找出入口文件的所有依赖，把入口和其所有依赖打包到一个单独的文件中。 从 Webpack2 开始，已经内置了对 ES6、CommonJS、AMD 模块化语句的支持。


