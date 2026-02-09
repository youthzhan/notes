

<!-- toc -->

- [webpack 知识点](#webpack-知识点)
  - [一、loader](#一loader)
    - [1、常见的loader](#1常见的loader)
    - [2、为什么需要Loader？](#2为什么需要loader)
    - [3、代码例子🌰](#3代码例子)
  - [二、Plugin](#二plugin)
    - [1、配置方式](#1配置方式)
    - [2、特性](#2特性)
    - [3、常见的Plugin](#3常见的plugin)
  - [三、Loader和Plugin的区别](#三loader和plugin的区别)
    - [1、区别](#1区别)
    - [2、编写loader](#2编写loader)
    - [3、编写plugin](#3编写plugin)
  - [四、webpack的热更新](#四webpack的热更新)
    - [1、定义](#1定义)
    - [2、实现原理](#2实现原理)
    - [3、总结](#3总结)
  - [五、webpack proxy工作原理](#五webpack-proxy工作原理)
    - [1、定义](#1定义-1)
    - [2、工作原理](#2工作原理)
    - [3、跨域](#3跨域)
  - [六、webpack优化前端性能](#六webpack优化前端性能)
    - [1、优化手段](#1优化手段)
      - [1、JS代码压缩](#1js代码压缩)
      - [2、CSS代码压缩](#2css代码压缩)
      - [3、Html文件代码压缩](#3html文件代码压缩)
      - [4、文件大小压缩](#4文件大小压缩)
      - [5、图片压缩](#5图片压缩)
      - [6、Tree Shaking](#6tree-shaking)
      - [7、代码分离](#7代码分离)
      - [8、内联chunk](#8内联chunk)
    - [2、总结](#2总结)
  - [七、提高webpack的构建速度](#七提高webpack的构建速度)

<!-- tocstop -->

# webpack 知识点

## 一、loader
### 1、常见的loader
样式处理：

- **style-loader**: 将css添加到DOM的内联样式标签style里
- **css-loader** :允许将css文件通过require的方式引入，并返回css代码
- **less-loader**: 处理less
- **sass-loader**: 处理sass
- **postcss-loader**: 用postcss来处理CSS
- autoprefixer-loader: 处理CSS3属性前缀，已被弃用，建议直接使用postcss

文件处理：（处理图片、字体等）
- **file-loader**: 分发文件到output目录并返回相对路径
- **url-loader**: 和file-loader类似，但是当文件小于设定的limit时可以返回一个Data Url

JavaScript处理：
- **babel-loader**: 用babel来转换ES6文件到ES（简：ES6转ES5）
- **ts-loader**: TypeScript编译，把.ts/.tsx 文件编译成 JavaScript，让 Webpack 能打包

其他：
- **html-minify-loader**: 压缩HTML
- **vue-loader**：把 .vue 单文件组件（SFC）翻译成 Webpack 能够理解和打包的 JavaScript 模块
- **markdown-loader**：把 Markdown 源文件（.md）转换成可在 Webpack 构建流程中直接使用的字符串 / HTML / Vue 组件 / React 组件等，从而“像引用 JS 模块一样引用 Markdown”。

### 2、为什么需要Loader？
1. 模块化：让所有类型的资源都能像JavaScript模块一样被导入和使用
2. 预处理：在打包前对文件进行转换（如Sass编译、ES6转码）
3. 优化：可以对资源进行压缩、提取等优化操作
4. 灵活性：可以根据不同环境配置不同的loader处理策略

在你的配置中，loader的作用就是让webpack能够处理SCSS文件，并支持CSS Modules功能，最终将样式提取到独立的CSS文件中。


### 3、代码例子🌰
```js
 config.module.rules.push(
      {
        test: /\.module\.(sa|sc)ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader, // 将CSS提取到单独的文件中（而不是打包到JS中）
            options: {
              hmr: devMode,
            },
          },
          {
            loader: 'css-loader', // 解析CSS文件中的@import和url()，并将CSS转换为CommonJS模块
            options: {
              modules: true, // 启用CSS Module
            },
          },
          {
            loader: 'sass-loader', // 将Sass/Scss文件编译为CSS
            options: {
              sassOptions: {
                logger: {
                  warn: (message, { deprecation }) => {
                    // 屏蔽告警，需要注意迁移
                  },
                },
              },
            },
          },
        ],
      },
      // 处理非CSS Module的SCSS文件（文件名不包含.module.scss）
      {
        test: /\.(sa|sc)ss$/,
        exclude: /\.module\.(sa|sc)ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              hmr: devMode,
            },
          },
          {
            loader: 'css-loader',
            options: {
              modules: false, // 禁用CSS Module
            },
          },
          {
            loader: 'sass-loader',
            options: {
              sassOptions: {
                logger: {
                  warn: (message, { deprecation }) => {
                    // 屏蔽告警，需要注意迁移
                  },
                },
              },
            },
          },
        ],
      },
    );

    config.module.rules.forEach((rule, idx) => {
      if (rule.test instanceof RegExp && rule.test.test('.png')) {
        // eslint-disable-next-line no-param-reassign
        rule.use[0].options.limit = 1024 * 60;
      }

      // 删掉tea中默认svg处理配置，方便自定义配置
      if (rule.test instanceof RegExp && rule.test.test('.svg')) {
        config.module.rules.splice(idx, 1);
      }
    });

    /**
     * 处理 svg
     * 1. react组件内部可直接当成组件引入svg
     * 2. css|sass|scss|less中不引入为组件，引入为url
     */
    config.module.rules.push({
      test: /\.svg$/,
      oneOf: [
        // 如果从 JavaScript/TypeScript 文件中导入
        {
          issuer: /\.[jt]sx?$/,
          use: [
            {
              loader: '@svgr/webpack',
              options: {
                icon: true,
                prettier: false,
                dimensions: false,
              },
            },
          ],
        },
        // 如果从 CSS/SCSS 文件中导入
        {
          issuer: /\.(css|scss|sass|less)$/,
          use: [
            {
              loader: 'url-loader',
              options: {
                limit: 20480,
                name: 'mps.[hash].[ext]',
              },
            },
            // 可以「链式」修改 SVG，最终生成新的SVG文件或内联字符串，常用于「去色、改色、加 class、删属性」等批量处理。
            'svg-transform-loader',
            {
              loader: 'svgo-loader', // svgo-loader 是 Webpack 的 SVGO 压缩管道，把任何进来的 SVG 先瘦身（删冗余、合并路径、压缩属性）
              options: {
                plugins: [{ removeTitle: true }, { convertStyleToAttrs: true }],
              },
            },
          ],
        },
        // 特殊逻辑单独处理
        {
          include: path.resolve(__dirname, 'src/assets/images/workflow'),
          use: [
            {
              loader: 'url-loader',
              options: {
                limit: 20480,
                name: 'mps.[hash].[ext]',
              },
            },
            'svg-transform-loader',
            {
              loader: 'svgo-loader',
              options: {
                plugins: [{ removeTitle: true }, { convertStyleToAttrs: true }],
              },
            },
          ],
        },
      ],
    });
```
Loader的执行顺序是从右到左，从下到上的链式调用。在你的配置中：
> Sass文件 → sass-loader → css-loader → MiniCssExtractPlugin.loader 
1. sass-loader：先将.scss文件编译为CSS
2. css-loader：处理CSS中的依赖关系（如@import）
3. MiniCssExtractPlugin.loader：将CSS提取到独立文件

## 二、Plugin
**Plugin是一种遵循一定规范的应用程序接口编写出来的计算机应用程序**，只能运行在程序规定的系统下，因为其需要调用原纯净系统提供的函数库或者数据，它和主应用程序互相交互，以提供特定的功能

webpack中的plugin的功能有很多，例如**打包优化、资源管理、环境变量注入**等，它们会运行在 webpack 的不同阶段（钩子 / 生命周期），贯穿了webpack整个编译周期

### 1、配置方式
一般情况，通过配置文件导出对象中plugins属性传入new实例对象
```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 访问内置的插件
module.exports = {
  ...
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};
```
### 2、特性
本质是一个具有**apply**方法**javascript对象**
apply 方法会被 webpack compiler调用，并且在整个编译生命周期都可以访问 compiler对象
```js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';
class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    // compiler hook 的 tap方法的第一个参数，应是驼峰式命名的插件名称
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建过程开始！');
    });
  }
}
module.exports = ConsoleLogOnBuildWebpackPlugin;
```
整个编译生命周期钩子，有如下：
- entry-option ：初始化 option
- run
- compile： 真正开始的编译，在创建 compilation 对象之前
- compilation ：生成好了 compilation 对象
- make： 从 entry 开始递归分析依赖，准备对每个模块进行 build
- after-compile： 编译 build 过程结束
- emit ：在将内存中 assets 内容写到磁盘文件夹之前
- after-emit ：在将内存中 assets 内容写到磁盘文件夹之后
- done： 完成所有的编译过程
- failed： 编译失败的时候
### 3、常见的Plugin
1. **HtmlWebpackPlugin**：在打包结束后，⾃动生成⼀个 html 文件，并把打包生成的js 模块引⼊到该 html 中（在 html 模板中，可以通过 <%=htmlWebpackPlugin.options.XXX%> 的方式获取配置的值）
```sh
npm install --save-dev html-webpack-plugin
```
```js
// webpack.config.js
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
 ...
  plugins: [
     new HtmlWebpackPlugin({
       title: "My App",
       filename: "app.html",
       template: "./src/html/index.html"
     }) 
  ]
};
```
```html
<!--./src/html/index.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title><%=htmlWebpackPlugin.options.title%></title>
</head>
<body>
    <h1>html-webpack-plugin</h1>
</body>
</html>
```
在 html 模板中，可以通过 <%=htmlWebpackPlugin.options.XXX%> 的方式获取配置的值

2. **clean-webpack-plugin**：删除（清理）构建目录
```sh
npm install --save-dev clean-webpack-plugin
```
```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
module.exports = {
 ...
  plugins: [
    ...,
    new CleanWebpackPlugin(),
    ...
  ]
}
```

3. **mini-css-extract-plugin**: 提取 CSS 到一个单独的文件中
```sh
npm install --save-dev mini-css-extract-plugin
```
```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
 ...,
  module: {
    rules: [
    {
      test: /\.s[ac]ss$/,
      use: [
        {
          loader: MiniCssExtractPlugin.loader
        },
          'css-loader',
          'sass-loader'
        ]
    }
    ]
  },
  plugins: [
    ...,
    new MiniCssExtractPlugin({
     filename: '[name].css'
    }),
    ...
  ]
}
```
4. **DefinePlugin**：允许在编译时创建配置的全局对象，是一个webpack内置的插件，不需要安装
```js
const { DefinePlugun } = require('webpack')

module.exports = {
 ...
    plugins:[
        new DefinePlugin({
            BASE_URL:'"./"'
        })
    ]
}
```
编译template模块时，能通过下述形式获取全局对象
```html
<link rel="icon" href="<%= BASE_URL%>favicon.ico>"
```
5. **copy-webpack-plugin**：复制文件或目录到执行区域（如vue的打包中，将一些文件放到public的目录下，那么这个目录会被复制到dist文件夹中）
```sh
npm install copy-webpack-plugin -D
```
```js
new CopyWebpackPlugin({
    parrerns:[
        {
            from:"public",
            globOptions:{
                ignore:[
                    '**/index.html'
                ]
            }
        }
    ]
})

```
复制的规则在patterns属性中设置：
- from：设置从哪一个源中开始复制
- to：复制到的位置，可以省略，会默认复制到打包的目录下
- globOptions：设置一些额外的选项，其中可以编写需要忽略的文件
## 三、Loader和Plugin的区别
### 1、区别
概念的区别：
- loader 是**文件加载器**，能够加载资源文件，并对这些文件进行一些处理，诸如编译、压缩等，最终一起打包到指定的文件中
- plugin 赋予了 webpack 各种灵活的**功能**，例如打包优化、资源管理、环境变量注入等，目的是解决 loader 无法实现的其他事
运行时机的区别：
- loader 运行在**打包文件之前**
- plugins 在**整个编译周期**都起作用
在Webpack运行的生命周期中会广播出许多事件，Plugin可以监听这些事件，在合适的时机通过Webpack提供的API改变输出结果
对于loader，实质是一个转换器，将A文件进行编译形成B文件，操作的是文件，比如将A.scss或A.less转变为B.css，单纯的文件转换过程
### 2、编写loader
loader的本质是函数，函数中的 this 作为上下文会被 webpack填充，因此我们不能将loader设为一个箭头函数
函数接受一个参数，为 webpack 传递给 loader 的文件源内容
函数中 this 是由 webpack 提供的对象，能够获取当前 loader 所需要的各种信息
函数中有异步操作或同步操作，异步操作通过 **this.callback** 返回，返回值要求为 **string** 或者 **Buffer**

代码如下所示：
```js
// 导出一个函数，source为webpack传递给loader的文件源内容
module.exports = function(source) {
  const content = doSomeThing2JsString(source);
  
  // 如果 loader 配置了 options 对象，那么this.query将指向 options
  const options = this.query;
  
  // 可以用作解析其他模块路径的上下文
  console.log('this.context');
  
  /*
    * this.callback 参数：
    * error：Error | null，当 loader 出错时向外抛出一个 error
    * content：String | Buffer，经过 loader 编译后需要导出的内容
    * sourceMap：为方便调试生成的编译后内容的 source map
    * ast：本次编译生成的 AST 静态语法树，之后执行的 loader 可以直接使用这个 AST，进而省去重复生成 AST 的过程
    */
  this.callback(null, content); // 异步
  return content; // 同步
}
```
一般在编写loader的过程中，保持**功能单一**，避免做多种功能
如less文件转换成 css文件也不是一步到位，而是 less-loader、css-loader、style-loader几个loader的链式调用才能完成转换

### 3、编写plugin
webpack基于发布订阅模式，在运行的生命周期中会广播出许多事件，插件通过监听这些事件，就可以在特定的阶段执行自己的插件任务
webpack编译会创建两个核心对象：
- **compiler**：包含了webpack环境的所有的配置信息，包括 options，loader 和 plugin，和 webpack 整个生命周期相关的钩子
- **compilation**：作为plugin内置事件回调函数的参数，包含了当前的模块资源、编译生成资源、变化的文件以及被跟踪依赖的状态信息。当检测到一个文件变化，一次新的 Compilation 将被创建
如果自己要实现plugin，也需要遵循一定的规范：
- 插件必须是一个函数或者是一个包含 apply 方法的对象，这样才能访问compiler实例
- 传给每个插件的 compiler 和 compilation 对象都是同一个引用，因此不建议修改
- 异步的事件需要在插件处理完任务时调用回调函数通知 Webpack 进入下一个流程，不然会卡住
实现plugin的模板如下：
```js
class MyPlugin {
    // Webpack 会调用 MyPlugin 实例的 apply 方法给插件实例传入 compiler 对象
  apply (compiler) {
    // 找到合适的事件钩子，实现自己的插件功能
    compiler.hooks.emit.tap('MyPlugin', compilation => {
        // compilation: 当前打包构建流程的上下文
        console.log(compilation);
        // do something...
    })
  }
}
```
在emit事件发生时，代表源文件的转换和组装已经完成，可以读取到最终将输出的资源、代码块、模块及其依赖，并且可以修改输出资源的内容
## 四、webpack的热更新
### 1、定义
HMR全称 Hot Module Replacement，模块热替换，指在不刷新整个页面的情况下，把改动过的模块实时替换到运行中的应用，保持当前状态（如输入框内容、弹窗、滚动位置等）。

例如，我们在应用运行过程中修改了某个模块，通过自动刷新会导致整个应用的整体刷新，那页面中的状态信息都会丢失
如果使用的是 HMR，就可以实现只将修改的模块实时替换至应用中，不必完全刷新整个应用

在webpack中配置开启热模块也非常的简单，如下代码：
```js
const webpack = require('webpack')
module.exports = {
  // ...
  devServer: {
    // 开启 HMR 特性
    hot: true
    // hotOnly: true
  }
}
```
HMR需要去指定**哪些模块**发生更新时进行HRM，如下代码：
```js
if(module.hot){
    module.hot.accept('./util.js',()=>{
        console.log("util.js更新了")
    })
}
```
### 2、实现原理
- Webpack Compile：将 JS 源代码编译成 bundle.js
- HMR Server：用来将热更新的文件输出给 HMR Runtime
- Bundle Server：静态资源文件服务器，提供文件访问路径
- HMR Runtime：socket服务器，会被注入到浏览器，更新文件的变化
- bundle.js：构建输出的文件
- 在HMR Runtime 和 HMR Server之间建立 websocket，即图上4号线，用于实时更新文件变化
分成启动阶段和更新阶段
启动阶段：
在编写未经过webpack打包的源代码后，Webpack Compile 将源代码和 HMR Runtime 一起编译成 bundle文件，传输给Bundle Server 静态资源服务器
更新阶段:
1. 当某一个文件或者模块发生变化时，webpack监听到文件变化对文件重新编译打包，编译生成唯一的hash值，这个hash值用来作为下一次热更新的标识
2. 根据变化的内容生成两个补丁文件：manifest（包含了 hash 和 chundId，用来说明变化的内容）和chunk.js 模块
3. 由于socket服务器在HMR Runtime 和 HMR Server之间建立 websocket链接，当文件发生改动的时候，服务端会向浏览器推送一条消息，消息包含文件改动后生成的hash值，作为下一次热更新的标识
### 3、总结
- 通过webpack-dev-server创建两个服务器：提供静态资源的服务（express）和Socket服务
- express server 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
- socket server 是一个 websocket 的长连接，双方可以通信
- 当 socket server 监听到对应的模块发生变化时，会生成两个文件.json（manifest文件）和.js文件（update chunk）
- 通过长连接，socket server 可以直接将这两个文件主动发送给客户端（浏览器）
- 浏览器拿到两个新的文件后，通过HMR runtime机制，加载这两个文件，并且针对修改的模块进行更新
## 五、webpack proxy工作原理
### 1、定义
webpack proxy，即webpack提供的代理服务。基本行为就是接收客户端发送的请求后**转发**给其他服务器，其目的是为了便于开发者在开发模式下**解决跨域**问题（浏览器安全策略限制）

实现代理需要一个中间服务器，webpack中提供服务器的工具为**webpack-dev-server**

webpack-dev-server是 webpack 官方推出的一款开发工具，将自动编译和自动刷新浏览器等一系列对开发友好的功能全部集成在一起

目的是为了**提高开发者日常的开发效率**，只适用在开发阶段

关于配置方面，在webpack配置对象属性中通过**devServer**属性提供，如下：
```js
// ./webpack.config.js
const path = require('path')

module.exports = {
    // ...
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        compress: true,
        port: 9000,
        proxy: {
            '/api': {
                target: 'https://api.github.com'
            }
        }
        // ...
    }
}
```
**devServer的proxy**则是关于代理的配置，该属性为对象的形式，对象中每一个属性就是一个代理的规则匹配

属性的名称是需要被代理的请求路径前缀，一般为了辨别都会设置前缀为/api，值为对应的代理匹配规则，对应如下：
- **target**：表示的是代理到的目标地址
- **pathRewrite**：默认情况下，我们的 /api-hy 也会被写入到URL中，如果希望删除，可以使用pathRewrite
- **secure**：默认情况下不接收转发到https的服务器上，如果希望支持，可以设置为false
- **changeOrigin**：它表示是否更新代理后请求的 headers 中host地址
### 2、工作原理
proxy工作原理实质上是利用**http-proxy-middleware** 这个http代理中间件，实现请求转发给其他服务器

例子：

在开发阶段，本地地址为http://localhost:3000，该浏览器发送一个前缀带有/api标识的请求到服务端获取数据，但响应这个请求的服务器只是将请求转发到另一台服务器中
```js
const express = require('express');
const proxy = require('http-proxy-middleware');

const app = express();

app.use('/api', proxy({target: 'http://www.example.org', changeOrigin: true}));
app.listen(3000);

// http://localhost:3000/api/foo/bar -> http://www.example.org/api/foo/bar
```
### 3、跨域
在开发阶段， webpack-dev-server 会启动一个本地开发服务器，所以我们的应用在开发阶段是独立运行在 localhost的一个端口上，而后端服务又是运行在另外一个地址上

所以在开发阶段中，由于浏览器同源策略的原因，当本地访问后端就会出现跨域请求的问题

通过设置webpack proxy实现代理请求后，相当于浏览器与服务端中添加一个代理者

当本地发送请求的时候，代理服务器响应该请求，并将请求转发到目标服务器，目标服务器响应数据后再将数据返回给代理服务器，最终再由代理服务器将数据响应给本地

在代理服务器传递数据给本地浏览器的过程中，两者同源，并不存在跨域行为，这时候浏览器就能正常接收数据

`注意：服务器与服务器之间请求数据并不会存在跨域行为，跨域行为是浏览器安全策略限制`
## 六、webpack优化前端性能
### 1、优化手段
webpack优化前端的手段有：
- JS代码压缩
- CSS代码压缩
- Html文件代码压缩
- 文件大小压缩
- 图片压缩
- Tree Shaking
- 代码分离
- 内联 chunk
#### 1、JS代码压缩
**terser**是一个JavaScript的解释、绞肉机、压缩机的工具集，可以帮助我们压缩、丑化我们的代码，让bundle更小

丑化: 把原本可读性很好的变量名、函数名、属性名等替换成极短且无意义的字符（如 a、b、c、_0x1a2b），同时删除所有注释和多余空白，从而让代码“人眼看不懂”，但机器仍能正常执行。

在production模式下，webpack 默认使用 TerserPlugin 处理代码。如果想要自定义配置它，配置方法如下：
```js
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
    ...
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({
                parallel: true // 电脑cpu核数-1
            })
        ]
    }
}
```
属性如下：
- extractComments：默认值为true，表示会将注释抽取到一个单独的文件中，开发阶段，我们可设置为 false ，不保留注释
- parallel：使用多进程并发运行提高构建的速度，默认值是true，并发运行的默认数量： os.cpus().length - 1
- terserOptions：设置我们的terser相关的配置：
- compress：设置压缩相关的选项
- mangle：设置丑化相关的选项，可以直接设置为true
- toplevel：底层变量是否进行转换
- keep_classnames：保留类的名称
- keep_fnames：保留函数的名称
#### 2、CSS代码压缩
CSS压缩通常是**去除无用的空格**等，因为很难去修改选择器、属性的名称、值等

CSS压缩使用另外一个插件：**css-minimizer-webpack-plugin**

```sh
npm install css-minimizer-webpack-plugin -D
```

配置方法如下：

```js
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
module.exports = {
    // ...
    optimization: {
        minimize: true,
        minimizer: [
            new CssMinimizerPlugin({
                parallel: true
            })
        ]
    }
}
```
#### 3、Html文件代码压缩
使用`HtmlWebpackPlugin`插件来生成`HTML`的模板时候，通过配置属性`minify`进行`html`优化
```js
module.exports = {
    ...
    plugin:[
        new HtmlwebpackPlugin({
            ...
            minify:{
                minifyCSS:false, // 是否压缩css
                collapseWhitespace:false, // 是否折叠空格
                removeComments:true // 是否移除注释
            }
        })
    ]
}
```
设置了`minify`，实际会使用另一个插件`html-minifier-terser`
#### 4、文件大小压缩
对文件的大小进行压缩，减少`http`传输过程中宽带的损耗 用 `compression-webpack-plugin`
```js
npm install compression-webpack-plugin -D
```
```js
new ComepressionPlugin({
    test:/\.(css|js)$/,  // 哪些文件需要压缩
    threshold:500, // 设置文件多大开始压缩
    minRatio:0.7, // 至少压缩的比例
    algorithm:"gzip", // 采用的压缩算法
})
```
#### 5、图片压缩
一般来说在打包之后，一些图片文件的大小是远远要比 js 或者 css 文件要来的大，所以图片压缩较为重要
配置方法如下：
```js
module: {
  rules: [
    {
      test: /\.(png|jpg|gif)$/,
      use: [
        {
          loader: 'file-loader',
          options: {
            name: '[name]_[hash].[ext]',
            outputPath: 'images/',
          }
        },
        {
          loader: 'image-webpack-loader',
          options: {
            // 压缩 jpeg 的配置
            mozjpeg: {
              progressive: true,
              quality: 65
            },
            // 使用 imagemin**-optipng 压缩 png，enable: false 为关闭
            optipng: {
              enabled: false,
            },
            // 使用 imagemin-pngquant 压缩 png
            pngquant: {
              quality: '65-90',
              speed: 4
            },
            // 压缩 gif 的配置
            gifsicle: {
              interlaced: false,
            },
            // 开启 webp，会把 jpg 和 png 图片压缩为 webp 格式
            webp: {
              quality: 75
            }
          }
        }
      ]
    },
  ]
}
```
#### 6、Tree Shaking
Tree Shaking 在计算机中表示消除死代码，依赖于`ES Module`的静态语法分析（不执行任何的代码，可以明确知道模块的依赖关系）

在webpack实现Trss shaking有两种不同的方案：
- usedExports：通过标记某些函数是否被使用，之后通过Terser来进行优化的
- sideEffects：跳过整个模块/文件，直接查看该文件是否有副作用

两种不同的配置方案， 有不同的效果

`usedExports`: 将usedExports设为true
```js
module.exports = {
  ...
  optimization:{
    usedExports
  }
}
```
使用之后，没被用上的代码在`webpack`打包中会加入`unused harmony export mul`注释，用来告知 `Terser` 在优化时，可以删除掉这段代码


`sideEffects`: 用于告知`webpack compiler`哪些模块时有副作用，配置方法是在package.json中设置sideEffects属性

如果sideEffects设置为false，就是告知webpack可以安全的删除未用到的exports

如果有些文件需要保留，可以设置为数组的形式
```js
"sideEffects":[
    "./src/util/format.js",
    "*.css" // 所有的css文件
]
```
`css tree shaking`: `css`进行`tree shaking`优化可以安装`PurgeCss`插件(purgecss-plugin-webpack)
```
npm install purgecss-plugin-webpack -D
```
```js
const PurgeCssPlugin = require('purgecss-webpack-plugin')
module.exports = {
  ...
  plugins:[
    new PurgeCssPlugin({
      path:glob.sync(`${path.resolve('./src')}/**/*`), {nodir:true}// src里面的所有文件
      satelist:function(){
        return {
          standard:["html"]
        }
      }
    })
  ]
}
```
- paths：表示要检测哪些目录下的内容需要被分析，配合使用glob
- 默认情况下，Purgecss会将我们的html标签的样式移除掉，如果我们希望保留，可以添加一个safelist的属性
#### 7、代码分离
将代码分离到不同的bundle，可以按需加载或者并行加载文件

默认情况下，所有的JavaScript代码（业务代码、第三方依赖、暂时没有用到的模块）在首页**全部都加载，就会影响首页的加载速度**

代码分离可以分出出更小的bundle，以及**控制资源加载优先级，提供代码的加载性能**

这里通过`splitChunksPlugin`来实现，该插件webpack已经默认安装和集成，只需要配置即可

```js
module.exports = {
    ...
    optimization:{
        splitChunks:{
            chunks: "all"
        }
    }
}
```
`splitChunks`主要属性如下：
- **chunks**: 对同步代码还是异步代码进行处理  **initial同步，async异步，all 同步+异步**
- **minSize**：拆分包的大小, 至少为minSize，如果包的大小不超过minSize，这个包不会拆分
- **maxSize**：将大于maxSize的包，拆分为不小于minSize的包
- **minChunks**：被引入的次数，默认是1
#### 8、内联chunk
可以通过`InlineChunkHtmlPlugin`插件将一些`chunk`的模块内联到`html`，如runtime的代码（对模块进行解析、加载、模块信息相关的代码），代码量并不大，但是必须加载的
```js
const InlineChunkHtmlPlugin = require('react-dev-utils/InlineChunkHtmlPlugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  ...
  plugin:[
    new InlineChunkHtmlPlugin(HtmlWebpackPlugin,[/runtime.+\.js/])
  ]
}
```
### 2、总结
关于webpack对前端性能的优化，可以通过**文件体积大小**入手，其次还可通过**分包**的形式、**减少http请求次数**等方式，实现对前端性能的优化
## 七、提高webpack的构建速度
常见的提升构建速度的手段有如下：
- 优化 loader 配置
- 合理使用 resolve.extensions
- 优化 resolve.modules
- 优化 resolve.alias
- 使用 DLLPlugin 插件
- 使用 cache-loader
- terser 启动多线程
- 合理使用 sourceMap



