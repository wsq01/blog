---
title: webpack配置项
date: 2020-04-04 20:27:31
tags: webpack
---

webpack 是需要传入一个配置对象。

整个配置中我们使用 Node 内置的`path`模块，并在它前面加上`__dirname`这个全局变量。可以防止不同操作系统之间的文件路径问题，并且可以使相对路径按照预期工作。
# 模式(mode)
`mode`配置选项用来指定当前的构建环境。
设置`mode`可以使用 webpack 内置的函数，默认值为`production`
```js
module.exports = {
  mode: "production" // production | development | none
}
```

| 选项 | 描述 |
| :--: | :-- |
| `development` | 会将`process.env.NODE_ENV`的值设为`development`。启用`NamedChunksPlugin`和`NamedModulesPlugin`。 |
| `production` | 会将`process.env.NODE_ENV`的值设为`production`。启用`FlagDependencyUsagePlugin`，`FlagIncludedChunksPlugin`，`ModuleConcatenationPlugin`，`NoEmitOnErrorsPlugin`，`OccurrenceOrderPlugin`，`SideEffectsFlagPlugin`和`UglifyJsPlugin`。 |
| `none` | 不开启任何优化选项 |

# 入口(entry)
`entry`用来指定 webpack 的打包入口。
简单规则：每个 HTML 页面都有一个入口起点。单页应用(SPA)：一个入口起点，多页应用(MPA)：多个入口起点。
```js
module.exports = {
  entry: "./app/entry", // string | object | array
  // 如果传递一个数组，那么数组的每一项都会执行
  entry: ["./app/entry1", "./app/entry2"],
  entry: {
    a: "./app/entry-a",
    b: ["./app/entry-b1", "./app/entry-b2"]
  }
}
```
如果传入一个字符串或字符串数组，`chunk`会被命名为`main`。如果传入一个对象，则每个键(`key`)会是`chunk`的名称，该值描述了`chunk`的入口起点。
# 输出(output)
`output`用来告诉 webpack 如何将编译后的文件输出到磁盘。
```js
module.exports = {
  output: {
    // webpack 如何输出结果的相关选项
    path: path.resolve(__dirname, "dist"), // string
    // 所有输出文件的目标路径
    // 必须是绝对路径（使用 Node.js 的 path 模块）

    filename: "bundle.js", // string
    filename: "[name].js", // 用于多个入口点
    filename: "[chunkhash].js", // 用于长效缓存
    // 「入口分块(entry chunk)」的文件名模板

    publicPath: "/assets/", // string
    publicPath: "",
    publicPath: "https://cdn.example.com/",
    // 输出解析文件的目录，url 相对于 HTML 页面

    library: "MyLibrary", // string, 导出库(exported library)的名称

    libraryTarget: "umd", // 通用模块定义
        libraryTarget: "umd2", // 通用模块定义
        libraryTarget: "commonjs2", // exported with module.exports
        libraryTarget: "commonjs-module", // 使用 module.exports 导出
        libraryTarget: "commonjs", // 作为 exports 的属性导出
        libraryTarget: "amd", // 使用 AMD 定义方法来定义
        libraryTarget: "this", // 在 this 上设置属性
        libraryTarget: "var", // 变量定义于根作用域下
        libraryTarget: "assign", // 盲分配(blind assignment)
        libraryTarget: "window", // 在 window 对象上设置属性
        libraryTarget: "global", // property set to global object
        libraryTarget: "jsonp", // jsonp wrapper
    // 导出库(exported library)的类型
  }
}
```
## output.path
`output`目录对应一个绝对路径。
```js
path: path.resolve(__dirname, 'dist/assets')
```
## output.filename
此选项决定了每个输出`bundle`的名称。这些`bundle`将写入到`output.path`选项指定的目录下。

对于单个入口起点，`filename`会是一个静态名称。
```js
filename: "bundle.js"
```
然而，当通过多个入口起点、代码拆分或各种插件创建多个`bundle`，应该使用以下一种替换方式，来赋予每个`bundle`一个唯一的名称。

```js
// 使用入口名称：
filename: "[name].bundle.js"
// 使用内部chunk id
filename: "[id].bundle.js"
// 使用每次构建过程中，唯一的 hash 生成
filename: "[name].[hash].bundle.js"
// 使用基于每个 chunk 内容的 hash：
filename: "[chunkhash].bundle.js"
```
可以使用以下替换模板字符串：

| 模板 | 描述 |
| :--: | :--: |
| `[hash]` | 模块标识符的`hash` |
| `[chunkhash]` | `chunk`内容的`hash` |
| `[name]` | 模块名称 |
| `[id]` | 模块标识符 |
| `[query]` | 模块的`query`，例如，文件名`?`后面的字符串 |

`[hash]`和`[chunkhash]`的长度可以使用`[hash:16]`（默认为20）来指定。
## output.publicPath
对于按需加载或加载外部资源（如图片、文件等）来说，`output.publicPath`是很重要的选项。如果指定了一个错误的值，则在加载这些资源时会收到 404 错误。

此选项指定在浏览器中所引用的「此输出目录对应的公开 URL」。相对 URL会被相对于 HTML 页面（或`<base>`标签）解析。相对于服务的 URL，相对于协议的 URL或绝对 URL 也可是可能用到的，或者有时必须用到，例如：当将资源托管到 CDN 时。

该选项的值是以`runtime`(运行时) 或`loader`(载入时) 所创建的每个 URL 为前缀。因此，在多数情况下，此选项的值都会以`/`结束。

默认值是一个空字符串`""`。

简单规则如下：`output.path`中的 URL 以 HTML 页面为基准。
```js
path: path.resolve(__dirname, "public/assets"),
publicPath: "https://cdn.example.com/assets/"
```
对于这个配置：
```js
publicPath: "/assets/",
chunkFilename: "[id].chunk.js"
```
对于一个`chunk`请求，看起来像这样`/assets/4.chunk.js`。

对于一个输出 HTML 的`loader`可能会像这样输出：
```html
<link href="/assets/spinner.gif" />
```
或者在加载 CSS 的一个图片时：
```
background-image: url(/assets/spinner.gif);
```
`webpack-dev-server`也会默认从`publicPath`为基准，使用它来决定在哪个目录下启用服务，来访问 webpack 输出的文件。

注意，参数中的`[hash]`将会被替换为编译过程的`hash`。

示例：
```js
publicPath: "https://cdn.example.com/assets/", // CDN（总是 HTTPS 协议）
publicPath: "//cdn.example.com/assets/", // CDN (协议相同)
publicPath: "/assets/", // 相对于服务
publicPath: "assets/", // 相对于 HTML 页面
publicPath: "../assets/", // 相对于 HTML 页面
publicPath: "", // 相对于 HTML 页面（目录相同）
```
# 模块(module)
这些选项决定了如何处理项目中的不同类型的模块。
```js
module.exports = {
  module: {
    // 关于模块配置
    rules: [
      // 模块规则（配置 loader、解析器等选项）
      {
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],
        // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
        // test 和 include 具有相同的作用，都是必须匹配选项
        // exclude 是必不匹配选项（优先于 test 和 include）
        // 最佳实践：
        // - 只在 test 和 文件名匹配 中使用正则表达式
        // - 在 include 和 exclude 中使用绝对路径数组
        // - 尽量避免 exclude，更倾向于使用 include

        loader: "babel-loader",
        // 应该应用的 loader，它相对上下文解析

        options: {
          presets: ["es2015"]
        },
        // loader 的可选项
      },

      {
        test: /\.html$/,
        test: "\.html$"

        use: [
          // 应用多个 loader 和选项
          "htmllint-loader",
          {
            loader: "html-loader",
            options: {
              /* ... */
            }
          }
        ]
      }
    ],

    noParse: [
      /special-library\.js$/
    ],
    // 不解析这里的模块
  }
}
```
## module.rules
创建模块时，匹配请求的规则数组。这些规则能够修改模块的创建方式。这些规则能够对模块应用`loader`，或者修改解析器。
## module.noParse
防止 webpack 解析那些任何与给定正则表达式相匹配的文件。忽略的文件中不应该含有`import, require, define`的调用，或任何其他导入机制。忽略大型的`library`可以提高构建性能。
```js
noParse: /jquery|lodash/

// 从 webpack 3.0.0 开始
noParse: function(content) {
  return /jquery|lodash/.test(content);
}
```
# 解析(resolve)
这些选项能设置模块如何被解析。例如，当调用`import "lodash"`，`resolve`选项能够对 webpack 查找`lodash`的方式去做修改。
```js
module.exports = {
  resolve: {
    // 解析模块请求的选项
    // （不适用于对 loader 解析）

    modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    // 用于查找模块的目录

    extensions: [".js", ".json", ".jsx", ".css"],
    // 使用的扩展名

    alias: {
      // 模块别名列表

      "module": "new-module",
      // 起别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"

      "only-module$": "new-module",
      // 起别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"

      "module": path.resolve(__dirname, "app/third/module.js"),
      // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
      // 模块别名相对于当前上下文导入
    }
  },
}
```
## resolve.alias
创建`import`或`require`的别名，来确保模块引入变得更简单。例如，一些位于`src/`文件夹下的常用模块：
```js
alias: {
  Utilities: path.resolve(__dirname, 'src/utilities/'),
  Templates: path.resolve(__dirname, 'src/templates/')
}
```
现在，替换「在导入时使用相对路径」这种方式，就像这样：
```js
import Utility from '../../utilities/utility';
```
你可以这样使用别名：
```js
import Utility from 'Utilities/utility';
```
也可以在给定对象的键后的末尾添加`$`，以表示精准匹配：
```js
alias: {
  xyz$: path.resolve(__dirname, 'path/to/file.js')
}
```
这将产生以下结果：
```js
import Test1 from 'xyz'; // 精确匹配，所以 path/to/file.js 被解析和导入
import Test2 from 'xyz/file.js'; // 非精确匹配，触发普通解析
```
## resolve.extensions

自动解析确定的扩展。默认值为：
```js
extensions: [".js", ".json"]
```
能够使用户在引入模块时不带扩展：
```js
import File from '../path/to/file'
```
使用此选项，会覆盖默认数组，这就意味着 webpack 将不再尝试使用默认扩展来解析模块。对于使用其扩展导入的模块，例如，`import SomeFile from "./somefile.ext"`，要想正确的解析，一个包含“*”的字符串必须包含在数组中。
# devtool
此选项控制是否生成，以及如何生成`source map`。

使用`SourceMapDevToolPlugin`进行更细粒度的配置。
```js
module.exports = {
  devtool: "source-map", // enum
  devtool: "inline-source-map", // 嵌入到源文件中
  devtool: "eval-source-map", // 将 SourceMap 嵌入到每个模块中
  devtool: "hidden-source-map", // SourceMap 不在源文件中引用
  devtool: "cheap-source-map", // 没有模块映射(module mappings)的 SourceMap 低级变体
  devtool: "cheap-module-source-map", // 有模块映射的 SourceMap 低级变体
  devtool: "eval", // 没有模块映射，而是命名模块。以牺牲细节达到最快。
  // 通过在浏览器调试工具(browser devtools)中添加元信息增强调试
  // 牺牲了构建速度的 `source-map' 是最详细的。
}
```
# context
基础目录，绝对路径，用于从配置中解析入口起点和`loader`。
```js
context: path.resolve(__dirname, "app")
```
默认使用当前目录，但是推荐在配置中传递一个值。
```js
module.exports = {
   context: __dirname, // string（绝对路径！）
  // webpack 的主目录
  // entry 和 module.rules.loader 选项
  // 相对于此目录解析
}
```
# 开发中 Server(devServer)
```js
module.exports = {
   devServer: {
    proxy: { // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    contentBase: path.join(__dirname, 'public'), // boolean | string | array, static file location
    compress: true, // enable gzip compression
    historyApiFallback: true, // true for index.html upon 404, object for multiple paths
    hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
    https: false, // true for self-signed, object for cert authority
    noInfo: true, // only errors & warns on hot reload
    // ...
  },
}
```
# 插件(plugins)
`plugins`选项用于以各种方式自定义 webpack 构建过程。webpack 附带了各种内置插件，可以通过`webpack.[plugin-name]`访问这些插件。
```javascript
module.exports = {
  plugins: [
    // 例如，当多个 bundle 共享一些相同的依赖，
    // CommonsChunkPlugin 有助于提取这些依赖到共享的 bundle 中，来避免重复打包。
    new webpack.optimize.CommonsChunkPlugin({
      ...
    })
  ]
}
```
