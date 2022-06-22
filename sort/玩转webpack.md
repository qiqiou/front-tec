# 玩转webpack

## 1. 为什么选择webpack

- 社区⽣态丰富 

- 配置灵活和插件化扩展 

- 官⽅更新迭代速度快

## 2. 初识webpack

### 2.1 配置文件名称

webpack 默认配置⽂件：webpack.config.js

可以通过 webpack --config 指定配置⽂件

### 2.2 webpack 配置组成

```js
module.exports = {
  // 打包入口文件
  entry: './src/index.js',
  // 打包输出文件及名称
  output: './dist/main.js',
  // 打包环境
  mode: 'production',
  // loader 配置
  module: {
    rules: [
    	{ test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  // 插件配置
  plugins: [
    new HtmlwebpackPlugin({
    	template: './src/index.html’
    })
  ]
};
```

## 3. webpack基础用法

### 3.1 entry

Entry ⽤来指定 webpack 的打包⼊⼝

#### 单入口-单页面应用

```js
module.exports = {
	entry: './path/to/my/entry/file.js'
};
```

#### 多入口-多页面应用

```js
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js'
  }
};
```

### 3.2 output

Output ⽤来告诉 webpack 如何将编译后的⽂件输出到磁盘

#### 单入口

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
  output: {
    filename: 'bundle.js’,
    path: __dirname + '/dist'
  }
};
```

#### 多入口

```js
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    // 通过占位符确保⽂件名称的唯⼀
    filename: '[name].js',
    path: __dirname + '/dist'
  }
};
```

### 3.3 loaders

webpack 开箱即用只支持 JS 和 JSON 两种文件类型，

通过 Loaders 去支持其它文 件类型并且把它们转化成有效的模块，并且可以添加到依赖图中。

本身是一个函数，接受源文件作为参数，返回转换的结果。

#### 常见的loader

| 名称          | 描述                     |
| ------------- | ------------------------ |
| babel-loader  | 转换ES6、ES7等新特性语法 |
| css-loader    | 支持css文件的加载和解析  |
| less-loader   | 将less文件转为css        |
| ts-loader     | 将ts转为js               |
| file-loader   | 对图片和字体进行打包     |
| raw-loader    | 将文件以字符串的形式导入 |
| thread-loader | 多进程打包js和css        |

#### loaders的用法

```js
const path = require('path');
  module.exports = {
    output: {
    	filename: 'bundle.js'
    },
    module: {
    rules: [
      // test 指定匹配规则
      // use 指定使⽤的 loader 名称
    	{ test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```

### 3.4 plugins

插件⽤于 bundle ⽂件的优化，资源管理和环境变量注⼊ 

作⽤于整个构建过程

#### 常见的plugins

| 名称                     | 描述                                       |
| ------------------------ | ------------------------------------------ |
| CommonsChunkPlugin       | 将chunks相同的模块代码提取成公共的js       |
| CleanWebpackPlugin       | 清理构建目录                               |
| ExtractTextWebpackPlugin | 将CSS从bundle文件中提取成一个独立的CSS文件 |
| CopyWebpackPlugin        | 将文件或者文件夹拷贝到构建的输出目录       |
| HtmlWebpackPlugin        | 创建html文件去承载输出的bundle             |
| UglifyjsWebpackPlugin    | 压缩JS                                     |
| ZipWebpackPlugin         | 将打包出来的资源生成一个zip包              |

#### 用法

```js
const path = require('path');
  module.exports = {
  output: {
  	filename: 'bundle.js'
  },
  // 放到 plugins 数组⾥
  plugins: [
    new HtmlWebpackPlugin({template:
    './src/index.html'})
  ]
};
```

### 3.5 mode

Mode ⽤来指定当前的构建环境是：production、development 还是 none 

设置 mode 可以使⽤ webpack 内置的函数，默认值为 production

#### 用法

| 项          | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| development | 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 development. <br />为模块和 chunk 启用有效的名。 |
| production  | 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production。<br />为模块和 chunk 启用确定性的混淆名称，FlagDependencyUsagePlugin，FlagIncludedChunksPlugin，ModuleConcatenationPlugin，NoEmitOnErrorsPlugin 和 TerserPlugin 。 |
| none        | 不使用任何默认优化选项                                       |

### 3.6 资源解析

#### 3.6.1 解析 ES6

1. 使⽤ babel-loader 

   ```js
   const path = require('path');
   module.exports = {
     entry: './src/index.js',
     output: {
       filename: 'bundle.js',
       path: path.resolve(__dirname, 'dist')
     },
      module: {
       + rules: [
         + {
           + test: /\.js$/,
           + use: 'babel-loader'
         + }
       + ]
     + }
   };
   ```

2. babel的配置⽂件是：.babelrc

   ```js
   // 增加 ES6 的 babel preset 配置
   {
     "presets": [
     	+ "@babel/preset-env”
     ],
     "plugins": [
     	"@babel/proposal-class-properties"
     ]
   }
   ```

#### 3.6.2 解析 React JSX

```js
{
"presets": [
  "@babel/preset-env",
  // 增加 React 的 babel preset 配置
  	+ "@babel/preset-react"
  ],
  "plugins": [
  	"@babel/proposal-class-properties"
  ]
}
```

#### 3.6.3 解析 CSS

css-loader ⽤于加载 .css ⽂件，并且转换成 commonjs 对象 

style-loader 将样式通过\<style>标签插入到head中

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  + module: {
    + rules: [
      + {
        + test: /\.css$/,
        + use: [
          + 'style-loader',
          + 'css-loader'
        + ]
      + }
    + ]
  + }
};
```

#### 3.6.4 解析 Less 和 SaSS

less-loader ⽤于将 less 转换成 css

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  + module: {
    + rules: [
      + {
        + test: /\.less$/,
        + use: [
          + 'style-loader',
          + 'css-loader',
          + 'less-loader'
        + ]
      + }
    + ]
  + }
};
```

#### 3.6.5 解析图⽚和字体

file-loader ⽤于处理⽂件

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
    + {
      + test: /\.(png|svg|jpg|gif)$/,
      + use: [
      	+ 'file-loader'
      + ]
    + }
    ]
  }
};
```

#### 3.6.7 使⽤ url-loader

url-loader 也可以处理图⽚和字体 

可以设置较⼩资源⾃动 base64

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
  rules: [
    + {
      + test: /\.(png|svg|jpg|gif)$/,
      + use: [{
        + loader: 'url-loader’,
        + options: {
        	+ limit: 10240
        + }
      + }]
    + }
    ]
  }
};
```

### 3.7 ⽂件监听

⽂件监听是在发现源码发⽣变化时，⾃动重新构建出新的输出⽂件。

webpack 开启监听模式，有两种⽅式：

- 启动 webpack 命令时，带上 --watch 参数 
- 在配置 webpack.config.js 中设置 watch: true

**唯⼀缺陷：每次需要⼿动刷新浏览器**

```js
{
  "name": "hello-webpack",
  "version": "1.0.0",
  "description": "Hello webpack",
  "main": "index.js",
  "scripts": {
    "build": "webpack ",
    + "watch": "webpack --watch"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

#### △⽂件监听的原理分析

轮询判断⽂件的最后编辑时间是否变化 

某个⽂件发⽣了变化，并不会⽴刻告诉监听者，⽽是先缓存起来，等 aggregateTimeout

需要监听的文件列表是怎么确定的呢，默认情况下 Webpack 会从配置的 Entry 文件出发，递归解析出 Entry 文件所依赖的文件，把这些依赖的文件都加入到监听列表中去

```js
module.export = {
  //默认 false，也就是不开启
  watch: true,
  //只有开启监听模式时，watchOptions才有意义
  wathcOptions: {
    //默认为空，不监听的文件或者文件夹，支持正则匹配
    ignored: /node_modules/,
    //监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高
    aggregateTimeout: 300,
    //判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每隔1000毫秒询问一次
    poll: 1000
  }
}
```

### 3.8 热更新：webpack-dev-server

在使用 webpack-dev-server 模块去启动 webpack 模块时，webpack 模块的监听模式默认会被开启

控制浏览器刷新有三种方法：

1. 借助浏览器扩展去通过浏览器提供的接口刷新，WebStorm IDE 的 LiveEdit 功能就是这样实现的。
2. 往要开发的网页中注入代理客户端代码，通过代理客户端去刷新整个页面。
3. 把要开发的网页装进一个 iframe 中，通过刷新 iframe 去看到最新效果。

DevServer 支持第2、3种方法，第2种是 DevServer 默认采用的刷新方法。

在浏览器中打开网址后， 在浏览器的开发者工具中你会发现由代理客户端向 DevServer 发起的 WebSocket 连接



WDS 不刷新浏览器 

WDS 不输出⽂件，⽽是放在内存中 

使⽤ HotModuleReplacementPlugin插件

```js
{
  "name": "hello-webpack",
  "version": "1.0.0",
  "description": "Hello webpack",
  "main": "index.js",
  "scripts": {
  	"build": "webpack ",
  	+ ”dev": "webpack-dev-server --open"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

#### △热更新的原理分析

Webpack Compile: 将 JS 编译成 Bundle 

HMR Server: 将热更新的⽂件输出给 HMR Rumtime 

Bundle server: 提供⽂件在浏览器的访问 

HMR Rumtime: 会被注⼊到浏览器， 更新⽂件的变化 

bundle.js: 构建输出的⽂件

### △3.9 文件指纹

什么是文件指纹：打包后输出的⽂件名的后缀

文件指纹的好处：做项目的版本管理

#### ⽂件指纹如何⽣成

Hash：和整个项⽬的构建相关，只要项⽬⽂件有修改，整个项⽬构建的 hash 值就会更改 

Chunkhash：和 webpack 打包的 chunk 有关，不同的 entry 会⽣成不同的 chunkhash 值 

Contenthash：根据⽂件内容来定义 hash ，⽂件内容不变，则 contenthash 不变

#### JS 的⽂件指纹设置

设置 output 的 filename，使⽤ [chunkhash]

```js
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    + filename: '[name][chunkhash:8].js',
    path: __dirname + '/dist'
  }
};
```

#### CSS 的⽂件指纹设置

设置 MiniCssExtractPlugin 的 filename， 使⽤ [contenthash]

```js
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name][chunkhash:8].js',
    path: __dirname + '/dist'
  },
  plugins: [
    + new MiniCssExtractPlugin({
    	+ filename: `[name][contenthash:8].css
    + });
  ]
};
```

#### 图⽚的⽂件指纹设置

设置 file-loader 的 name，使⽤ [hash]

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
  rules: [
  {
    test: /\.(png|svg|jpg|gif)$/,
    use: [{
    loader: 'file-loader’,
    + options: {
    	+ name: 'img/[name][hash:8].[ext] '
    + }
  }]
}
```

### 3.10 代码压缩

- JS ⽂件的压缩：内置了 uglifyjs-webpack-plugin

- CSS ⽂件的压缩：使⽤ optimize-css-assets-webpack-plugin，同时使⽤ cssnano

  ```js
  module.exports = {
    entry: {
      app: './src/app.js',
      search: './src/search.js'
    },
    output: {
      filename: '[name][chunkhash:8].js',
      path: __dirname + '/dist'
    },
    plugins: [
      + new OptimizeCSSAssetsPlugin({
        + assetNameRegExp: /\.css$/g,
        + cssProcessor: require('cssnano’)
      + })
    ]
  };
  ```

- html ⽂件的压缩：修改 html-webpack-plugin，设置压缩参数

  ```js
  module.exports = {
    entry: {
      app: './src/app.js',
      search: './src/search.js'
    },
    output: {
      filename: '[name][chunkhash:8].js',
      path: __dirname + '/dist'
    },
    plugins: [
      + new HtmlWebpackPlugin({
        + template: path.join(__dirname, 'src/search.html’),
        + filename: 'search.html’,
        + chunks: ['search’],
        + inject: true,
        + minify: {
          + html5: true,
          + collapseWhitespace: true,
          + preserveLineBreaks: false,
          + minifyCSS: true,
          + minifyJS: true,
          + removeComments: false
        + }
      + })
    ]
  };
  ```

  

## 4. webpack进阶用法

