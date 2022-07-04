


# 创建 Vite 项目
```
npm create vite@latest
```

# 配置
## 共享配置
#### root
* 类型：`string`
* 默认：`process.cwd()`

项目根目录（`index.html`文件所在的位置）。可以是一个绝对路径，或者一个相对于该配置文件本身的相对路径。

#### base
* 类型：`string`
* 默认：`/`

开发或生产环境服务的公共基础路径。合法的值包括以下几种：
* 绝对 URL 路径名，例如`/foo/`
* 完整的 URL，例如`https://foo.com/`
* 空字符串或`./`

#### publicDir
* 类型：`string | false`
* 默认：`"public"`

作为静态资源服务的文件夹。该目录中的文件在开发期间在`/`处提供，并在构建期间复制到`outDir`的根目录，并且始终按原样提供或复制而无需进行转换。该值可以是文件系统的绝对路径，也可以是相对于项目的根目录的相对路径。

将`publicDir`设定为`false`可以关闭此项功能。

## 服务器选项
### server.host
* 类型：`string | boolean`
* 默认：`localhost`

指定服务器应该监听哪个 IP 地址。如果将此设置为`0.0.0.0`或者`true`将监听所有地址，包括局域网和公网地址。
#### server.port
* 类型：`number`
* 默认值：5173

指定开发服务器端口。注意：如果端口已经被使用，Vite 会自动尝试下一个可用的端口，所以这可能不是开发服务器最终监听的实际端口。
#### server.strictPort
* 类型：`boolean`

设为`true`时若端口已被占用则会直接退出，而不是尝试下一个可用端口。
#### server.https
* 类型：`boolean | https.ServerOptions`

启用 TLS + HTTP/2。注意：当`server.proxy`选项 也被使用时，将会仅使用 TLS。

这个值也可以是一个传递给`https.createServer()`的选项对象。
#### server.open
* 类型：`boolean | string`

在开发服务器启动时自动在浏览器中打开应用程序。当此值为字符串时，会被用作 URL 的路径名。若你想指定喜欢的浏览器打开服务器，你可以设置环境变量`process.env.BROWSER`（例如：firefox）。
```js
export default defineConfig({
  server: {
    open: '/docs/index.html'
  }
})
```
#### server.proxy
* 类型： `Record<string, string | ProxyOptions>`

为开发服务器配置自定义代理规则。期望接收一个`{ key: options }`对象。如果`key`值以`^`开头，将会被解释为`RegExp`。`configure`可用于访问`proxy`实例。
```js
export default defineConfig({
  server: {
    proxy: {
      // 字符串简写写法
      '/foo': 'http://localhost:4567',
      // 选项写法
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      },
      // 正则表达式写法
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, '')
      },
      // 使用 proxy 实例
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        configure: (proxy, options) => {
          // proxy 是 'http-proxy' 的实例
        }
      },
      // Proxying websockets or socket.io
      '/socket.io': {
        target: 'ws://localhost:3000',
        ws: true
      }
    }
  }
})
```
#### server.cors
* 类型：`boolean | CorsOptions`

为开发服务器配置 CORS。默认启用并允许任何源，传递一个选项对象来调整行为或设为`false`表示禁用。
#### server.headers
* 类型：`OutgoingHttpHeaders`

指定服务器响应的`header`。
