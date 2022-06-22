# webpack







## 1. webpack 一级配置属性有哪些？

**Entry**：入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。

**Output**：output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。

**Loader**：loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只能解析 JavaScript）。

**Plugins**：插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等。

**Mode**：模式，有生产模式production和开发模式development

#### 理解Loader

Webpack 本身只能加载JS/JSON模块，如果要加载其他类型的文件(模块)，就需要使用对应的loader 进行转换/加载

#### 理解Plugins

插件可以完成一些loader不能完成的功能。

插件的使用一般是在 webpack 的配置信息 plugins 选项中指定。

## 2. 除了 webpack ，其他的打包工具有了解过吗

> https://vue3js.cn/interview/webpack/Rollup_Parcel_snowpack_Vite.html#%E4%B8%80%E3%80%81%E6%A8%A1%E5%9D%97%E5%8C%96%E5%B7%A5%E5%85%B7

#### rollup

`Rollup` 是一款 `ES Modules` 打包器，从作用上来看，`Rollup` 与 `Webpack` 非常类似。不过相比于 `Webpack`，`Rollup`要小巧的多

现在很多我们熟知的库都都使用它进行打包，比如：`Vue`、`React`和`three.js`等

`Rollup`的优点：

- 代码效率更简洁、效率更高
- 默认支持 Tree-shaking

缺点：

- 加载其他类型的资源文件或者支持导入 `CommonJS` 模块，又或是编译 `ES` 新特性，这些额外的需求 `Rollup`需要使用插件去完成

#### Parcel

`Parcel`不仅打包了应用，同时也启动了一个开发服务器，跟`webpack Dev Server`一样

跟`webpack`类似，也支持模块热替换，但用法更简单

同时，`Parcel`有个十分好用的功能：支持自动安装依赖，像`webpack`开发阶段突然使用安装某个第三方依赖，必然会终止`dev server`然后安装再启动。而`Parcel`则免了这繁琐的工作流程

同时，`Parcel`能够零配置加载其他类型的资源文件，无须像`webpack`那样配置对应的`loader`

#### SnowPack

开发阶段，每次保存单个文件时，`Webpack`和`Parcel`都需要重新构建和重新打包应用程序的整个`bundle`。而`Snowpack`为你的应用程序每个文件构建一次，就可以永久缓存，文件更改时，`Snowpack`会重新构建该单个文件

#### Vite

它主要由两部分组成：

- 一个开发服务器，它基于 原生 ES 模块 提供了丰富的内建功能，如速度快到惊人的 [模块热更新HMR
- 一套构建指令，它使用 Rollup打包你的代码，并且它是预配置的，可以输出用于生产环境的优化过的静态资源

其作用类似`webpack`+ `webpack-dev-server`，其特点如下：

- 快速的冷启动
- 即时的模块热更新
- 真正的按需编译

`vite`会直接启动开发服务器，不需要进行打包操作，也就意味着不需要分析模块的依赖、不需要编译，因此启动速度非常快

## 3. webpack5 了解过哪些更新吗？

> https://blog.csdn.net/qq_17175013/article/details/119769033

## 4. 写 webpack loader or plugin 时，有遇到过什么棘手的问题？

## 5. webpack HMR 原理是什么？

> https://zhuanlan.zhihu.com/p/30669007
>
> https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/118

- 启动`webpack-dev-server`服务器时，创建一个webpack实例，

- 然后创建Server服务器，使用sockjs在浏览器端和服务端之间建立一个 websocket 长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端
  - 创建Server服务器时，会绑定webpack的编译完成事件回调
    - Server 会监听这些配置文件夹中静态文件的变化，变化后会通知浏览器端进行更新
    - 通过_sendStatus 方法将编译打包后的新模块 hash 值发送到浏览器端
    - hash发送成功后紧接着发送ok到浏览器端，客户端收到ok的消息后会执行reloadApp方法进行更新
  - 添加`webpack-dev-middleware`中间件，负责返回生成的文件
    - 利用`memory-fs`设置文件系统为内存文件系统
    - 调用 webpack 暴露的 API对代码变化进行监控，监听到文件变化，根据配置文件对模块重新编译打包，并将打包后的代码保存在内存中。
  - 创建express应用app，创建http服务器并启动服务
- reloadApp方法：判断是刷新浏览器还是进行模块热更新，如果是模块热更新，执行webpackHotUpdate事件，否则直接刷新浏览器
- HotModuleReplacement.runtime 接收到传递给他的新模块的 hash 值
  - 向 server 端发送 Ajax 请求，服务端返回一个 json，该 json 包含了所有要更新的模块的 hash 值，
  - 获取到更新列表后，该模块再次通过 jsonp 请求，获取到最新的模块代码
    - HotModulePlugin 将会对新旧模块进行对比，决定是否更新模块
    - 在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用。



#### **如何保存在内存中的？**

memory-fs 是 webpack-dev-middleware 的一个依赖库，

webpack-dev-middleware 将 webpack 原本的 outputFileSystem 替换成了MemoryFileSystem 实例，

这样 bundle.js 文件代码就作为一个简单 javascript 对象保存在了内存中，

当浏览器请求 bundle.js 文件时，devServer就直接去内存中找到上面保存的 javascript 对象返回给浏览器端。



#### **bundle.js 中接收 websocket 消息的代码从哪来的呢**

webpack-dev-server 修改了webpack 配置中的 entry 属性，在里面添加了 webpack-dev-client 的代码，

这样在最后的 bundle.js 文件中就会有接收 websocket 消息的代码了。

## 6. 请介绍一下webpack.config.js中的mode字段，webpack 的 dev 和 prod 的区别

**开发环境**中，我们需要：

- 强大的 source map 
- 有着 live reloading(实时重新加载) 或 hot module replacement(热模块替换) 能力的 localhost server。

**生产环境**目标则转移至其他方面，通过这些优化方式改善加载时间。关注点在于：

- 压缩 bundle
- 更轻量的 source map
- 资源优化等

## 7. webpack 的 splitChunk 使用方式

## 9. 对webpack有了解吗？chunk、bundle和module有什么区别

> https://github.com/pwstrick/daily/issues/815
> https://www.cnblogs.com/skychx/p/webpack-module-chunk-bundle.html

**Module**：对于 webpack 来说，项目源码中所有资源（包括 JS、CSS、Image、Font 等等）都属于 module 模块。可以配置指定的 Loader 去处理这些文件。

**Chunk**：当使用 webpack 将我们编写的源代码进行打包时，webpack 会根据文件引用关系生成 chunk 文件，webpack 会对这些 chunk 文件进行一些操作。

**Bundle**：webpack 处理完 chunk 文件之后，最终会输出 bundle 文件，这个 bundle 文件包含了经过加载和编译的最终产物。



通俗一点就是处理之前的文件是叫 module，处理中的文件叫 chunk，处理完后输出的文件就叫 bundle

## 10. webpack 打包 vue 速度太慢怎么办？

## 11. webpack的加载器有哪些用途？

| 名称          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| babel-loader  | 转换ES6、ES7等新特性语法，把 ES6 转换成 ES5                  |
| css-loader    | 支持css文件的加载和解析                                      |
| style-loader  | 把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS。   |
| less-loader   | 将less文件转为css                                            |
| ts-loader     | 将ts转为js                                                   |
| file-loader   | 对图片和字体进行打包，把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件 |
| url-loader    | 和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去 |
| raw-loader    | 将文件以字符串的形式导入                                     |
| thread-loader | 多进程打包js和css                                            |
| image-loader  | 加载并且压缩图片文件                                         |
| eslint-loader | 通过 ESLint 检查 JavaScript 代码                             |

## 12. 请简单描述一下webpack的插件

| 名称                     | 描述                                       |
| ------------------------ | ------------------------------------------ |
| CommonsChunkPlugin       | 将chunks相同的模块代码提取成公共的js       |
| CleanWebpackPlugin       | 清理构建目录                               |
| ExtractTextWebpackPlugin | 将CSS从bundle文件中提取成一个独立的CSS文件 |
| CopyWebpackPlugin        | 将文件或者文件夹拷贝到构建的输出目录       |
| HtmlWebpackPlugin        | 创建html文件去承载输出的bundle             |
| UglifyjsWebpackPlugin    | 压缩JS                                     |
| ZipWebpackPlugin         | 将打包出来的资源生成一个zip包              |

## 13. webpack中的Source Map有什么功能？

## 14. 如何理解webpack中的Tree Shaking？

## 15. 如何清理webpack输出目录中的文件？

## 16. 说说webpack中的hash、chunkhash和contenthash的区别？

## 17. 简要介绍一下WebPack的底层实现原理？

## 18. 你有没有对webpack进行过优化

## 19. webpack的加载器和插件有什么不同

#### loader

Loader直译为"加载器"。

Webpack将一切文件视为模块，但是webpack原生是只能解析js文件，如果想将其他文件也打包的话，就会用到loader。 

所以Loader的作用是让webpack拥有了加载和解析非JavaScript文件的能力。

#### plugin

Plugin直译为"插件"。

Plugin可以扩展webpack的功能，让webpack具有更多的灵活性。 

在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

#### 区别

- Loader在module.rules中配置，也就是说他作为模块的解析规则而存在。 类型为数组，每一项都是一个Object，里面描述了对于什么类型的文件（test），使用什么加载(loader)和使用的参数（options）
- Plugin在plugins中单独配置。 类型为数组，每一项是一个plugin的实例，参数都通过构造函数传入。

## 20. webpack的构建流程是怎么样的？

> https://github.com/pwstrick/daily/issues/950

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
- 确定入口：根据配置中的 entry 找出所有的入口文件；
- 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
- 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

### 21. 如何利用webpack来优化前端性能？

- 压缩代码。删除多余的代码、注释、简化代码的写法等等方式。可以利用webpack的UglifyJsPlugin和ParallelUglifyPlugin来压缩JS文件， 利用cssnano（css-loader?minimize）来压缩css
- 利用CDN加速。在构建过程中，将引用的静态资源路径修改为CDN上对应的路径。可以利用webpack对于output参数和各loader的publicPath参数来修改资源路径
- 删除死代码（Tree Shaking）。将代码中永远不会走到的片段删除掉。可以通过在启动webpack时追加参数--optimize-minimize来实现
- 提取公共代码。

## 22. Import 和 CommonJS 在 webpack 打包过程中有什么不同

## 23. 介绍下 Webpack 的整个生命周期

- entry-option ：初始化 option
- run
- compile： 真正开始的编译，在创建 compilation 对象之前
- compilation ：生成好了 compilation 对象
- make 从 entry 开始递归分析依赖，准备对每个模块进行 build
- after-compile： 编译 build 过程结束
- emit ：在将内存中 assets 内容写到磁盘文件夹之前
- after-emit ：在将内存中 assets 内容写到磁盘文件夹之后
- done： 完成所有的编译过程
- failed： 编译失败的时候

## 24. 说一下 webpack 中 css-loader 和 style-loader 的区别，file-loader 和 url-loader 的区别

## 25. 如何实现 webpack 持久化缓存

## 26. Webpack 打包时 Hash 码是怎么生成的？随机值存在一样的情况，如何避免？

## 27. Webpack 打包出来的体积太大，如何优化体积？

## 28. Webpack 为什么慢，如何进行优化

## 29. 如何提高webpack的打包速度?

> https://github.com/febobo/web-interview/issues/132

- happypack: 利用进程并行编译loader,利用缓存来使得 rebuild 更快,遗憾的是作者表示已经不会继续开发此项目,类似的替代者是thread-loader
- 外部扩展(externals): 将不怎么需要更新的第三方库脱离webpack打包，不被打入bundle中，从而减少打包时间,比如jQuery用script标签引入。
- dll: 采用webpack的 DllPlugin 和 DllReferencePlugin 引入dll，让一些基本不会改动的代码先打包成静态资源,避免反复编译浪费时间
- 利用缓存: webpack.cache、babel-loader.cacheDirectory、HappyPack.cache都可以利用缓存提高rebuild效率
- 缩小文件搜索范围: 比如babel-loader插件,如果你的文件仅存在于src中,那么可以include: path.resolve(__dirname, 'src'),当然绝大多数情况下这种操作的提升有限,除非不小心build了node_modules文件



1. 多入口情况下，使用CommonsChunkPlugin来提取公共代码
2. 通过externals配置来提取常用库
3. 利用DllPlugin和DllReferencePlugin预编译资源模块 通过DllPlugin来对那些我们引用但是绝对不会修改的npm包来进行预编译，再通过4. DllReferencePlugin将预编译的模块加载进来。
4. 使用Happypack 实现多线程加速编译
5. 使用webpack-uglify-parallel来提升uglifyPlugin的压缩速度。 原理上webpack-uglify-parallel采用了多核并行压缩来提升压缩速度
6. 使用Tree-shaking和Scope Hoisting来剔除多余代码

## 30. 使用 import 时，webpack 对 node_modules 里的依赖会做什么

## 31. package.json的两个字段dependencies和devDependencies有什么作用？

## 32. 什么是长缓存？在webpack中如何做到长缓存优化？

## 33. 说说webpack proxy工作原理？为什么能解决跨域? 

> https://github.com/febobo/web-interview/issues/130

## 34. 如何用localStoragewebpack 离线缓存静态资源？

## 35. webpack 如何做异步加载

## 36. webpack 把vue转为js的原理

> http://soiiy.com/Vue-js/15262.html