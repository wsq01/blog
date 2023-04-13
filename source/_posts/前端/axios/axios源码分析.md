---
title: axios源码分析
date: 2020-03-19 09:50:05
tags: axios
categories: 
 - 前端
 - axios
---
看源码第一步，先看`package.json`。一般都会申明`main`主入口文件。
```js
// package.json
{
  "name": "axios",
  "main": "index.js",
  // ...
}
```
主入口文件
```js
// index.js
module.exports = require("./lib/axios");
```
# lib/axios.js主文件
axios.js文件代码可分为三部分。
1. 第一部分：引入一些工具函数`utils、Axios`构造函数、默认配置`defaults`等。
2. 第二部分：是生成实例对象`axios、axios.Axios、axios.create`等。
3. 第三部分：取消相关 API 实现，还有`all、spread`、导出等实现。

## 第一部分
引入一些工具函数utils、Axios构造函数、默认配置defaults等。
```js
// 引入 utils 对象，有很多工具方法。
var utils = require("./utils");
// 引入 bind 方法
var bind = require("./helpers/bind");
// 核心构造函数 Axios
var Axios = require("./core/Axios");
// 合并配置方法
var mergeConfig = require("./core/mergeConfig");
// 引入默认配置
var defaults = require("./defaults");
```
## 第二部分
生成实例对象`axios、axios.Axios、axios.create`等。
```js
function createInstance(defaultConfig) {
  // new 一个 Axios 生成实例对象
  var context = new Axios(defaultConfig);
   // bind 返回一个新的 wrap 函数，
   // 也就是为什么调用 axios 是调用 Axios.prototype.request 函数的原因
  var instance = bind(Axios.prototype.request, context);
  // 将Axios构造函数中的原型方法绑定到instance上并且指定this作用域为context上下文环境
  // 也就是为什么 有 axios.get 等别名方法，
  // 且调用的是 Axios.prototype.get 等别名方法。
  utils.extend(instance, Axios.prototype, context);
  // 复制 context 到 intance 实例
  // 也就是为什么默认配置 axios.defaults 和拦截器  axios.interceptors 可以使用的原因
  // 其实是new Axios().defaults 和 new Axios().interceptors
  utils.extend(instance, context);
  // 最后返回实例对象
  return instance;
}

// 导出 创建默认实例
var axios = createInstance(defaults);
// 暴露 Axios class 允许 class 继承 也就是可以 new axios.Axios()
// 但  axios 文档中 并没有提到这个，我们平时也用得少。
axios.Axios = Axios;
// 工厂函数 根据配置创建新的实例
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};
```
### 工具方法之 bind
```js
// axios/lib/helpers/bind.js
// 给某个函数指定上下文，也就是this指向
module.exports = function bind(fn, thisArg) {
  //以thisArg为上下文，执行fn函数
  return function wrap() {
    var args = new Array(arguments.length);
    //将arguments按照顺序依次保存到args里面
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i];
    }
    //执行fn函数，参数为thisArg为上下文，args为参数
    return fn.apply(thisArg, args);
  };
};
```
### 工具方法之 utils.extend
```js
// axios/lib/utils.js
// 将一个对象的方法和属性扩展到另外一个对象上，并指定上下文
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```
其实就是遍历参数 b 对象，复制到 a 对象上，如果是函数则用 bind 调用。
### 工具方法之 utils.forEach
遍历数组和对象。设计模式称之为迭代器模式。
```js
// axios/lib/utils.js
function forEach(obj, fn) {
  // 判断 null 和 undefined 直接返回
  if (obj === null || typeof obj === 'undefined') {
    return;
  }
  // 如果不是对象，放在数组里
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }
  if (isArray(obj)) {
    // 是数组 则用for 循环，调用 fn 函数。参数类似 Array.prototype.forEach 的前三个参数。
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // 用 for in 遍历对象，但 for in 会遍历原型链上可遍历的属性。
    // 所以用 hasOwnProperty 来过滤自身属性了。
    // 其实也可以用Object.keys来遍历，它不遍历原型链上可遍历的属性。
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}
```
## 第三部分
取消相关 API 实现，还有`all、spread`、导出等实现。
```js
// 导出 Cancel 和 CancelToken
axios.Cancel = require("./cancel/Cancel");
axios.CancelToken = require("./cancel/CancelToken");
axios.isCancel = require("./cancel/isCancel");
// 导出 all 和 spread API
axios.all = function all(promises) {
  returnPromise.all(promises);
};
axios.spread = require("./helpers/spread");
module.exports = axios;
// Allow use of default import syntax in TypeScript
// 也就是可以以下方式引入
// import axios from "axios";
module.exports.default = axios;
```
# 核心构造函数 Axios
```js
// axios/lib/core/Axios.js
function Axios(instanceConfig) {
  // 默认参数
  this.defaults = instanceConfig;
  // 拦截器 请求和响应拦截器
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

Axios.prototype.request = function(config){
  // 省略，这个是核心方法，后文结合例子详细描述
}
// 这是获取 Uri 的函数，这里省略
Axios.prototype.getUri = function(){}
// 提供一些请求方法的别名
// 遍历执行
// 也就是为啥我们可以 axios.get 等别名的方式调用，而且调用的是 Axios.prototype.request 方法
utils.forEach(["delete", "get", "head", "options"], function forEachMethodNoData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, config) {
    returnthis.request(utils.merge(config || {}, {
      method: method,
      url: url
    }));
  };
});

utils.forEach(["post", "put", "patch"], function forEachMethodWithData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, data, config) {
    returnthis.request(utils.merge(config || {}, {
      method: method,
      url: url,
      data: data
    }));
  };
});

module.exports = Axios;
```
# 拦截器管理构造函数 InterceptorManager
请求前拦截，和请求后拦截。

构造函数，`handles`用于存储拦截器函数。
```js
function InterceptorManager() {
  this.handlers = [];
}
```
接下来声明了三个方法：使用、移除、遍历。
## InterceptorManager.prototype.use 使用
传递两个函数作为参数，数组中的一项存储的是`{fulfilled: function(){}, rejected: function(){}}`。返回数字 ID，用于移除拦截器。
```js
/**
 * Add a new interceptor to the stack
 *
 * @param {Function} fulfilled The function to handle `then` for a `Promise`
 * @param {Function} rejected The function to handle `reject` for a `Promise`
 *
 * @return {Number} 返回ID 是为了用 eject 移除
 */
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};
```
## InterceptorManager.prototype.eject 移除
根据`use`返回的 ID 移除拦截器。
```js
InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};
```
## InterceptorManager.prototype.forEach 遍历
遍历执行所有拦截器，传递一个回调函数（每一个拦截器函数作为参数）调用，被移除的一项是null，所以不会执行，也就达到了移除的效果。
```js
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};
```
# Axios.prototype.request 请求核心方法
这个函数是核心函数。主要做了这几件事：
1. 判断第一个参数是字符串，则设置`url`,也就是支持`axios("example/url", [, config])`，也支持`axios({})`。
2. 合并默认参数和用户传递的参数
3. 设置请求的方法，默认是是`get`方法
4. 将用户设置的请求和响应拦截器、发送请求的`dispatchRequest`组成`Promise`链，最后返回还是`Promise`实例。

```js
Axios.prototype.request = function request(config) {
  /*eslint no-param-reassign:0*/
  //判断参数类型，支持axios('url',{})以及axios(config)两种形式
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }
  // 合并默认参数和用户传递的参数
  config = mergeConfig(this.defaults, config);

  // 设置 请求方法，默认 get
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }
```
## 组成Promise链，返回Promise实例
这部分：用户设置的请求和响应拦截器、发送请求的`dispatchRequest`组成`Promise`链。也就是保证了请求前拦截器先执行，然后发送请求，再响应拦截器执行这样的顺序。
```js
  // 组成`Promise`链
  // 把 xhr 请求的 dispatchRequest 和 undefined 放在一个数组里
  var chain = [dispatchRequest, undefined];
  // 创建 Promise 实例
  var promise = Promise.resolve(config);
  // 遍历用户设置的请求拦截器 放到数组的 chain 前面
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });
  // 遍历用户设置的响应拦截器 放到数组的 chain 后面
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });
  // 遍历 chain 数组，直到遍历 chain.length 为 0
  while (chain.length) {
    // 两两对应移出来 放到 then 的两个参数里。
    promise = promise.then(chain.shift(), chain.shift());
  }
  //返回最终promise对象
  return promise;
};
```
# dispatchRequest 最终派发请求
这个函数主要做了如下几件事情：
1. 如果已经取消，则`throw`原因报错，使`Promise`走向`rejected`。
2. 确保`config.header`存在。
3. 利用用户设置的和默认的请求转换器转换数据。
4. 拍平`config.header`。
5. 删除一些`config.header`。
6. 返回适配器`adapter`（`Promise`实例）执行后 then执行后的`Promise`实例。返回结果传递给响应拦截器处理。

```js
var utils = require('./../utils');
// 转换数据
var transformData = require('./transformData');
// 取消状态
var isCancel = require('../cancel/isCancel');
// 默认参数
var defaults = require('../defaults');

/**
 * 抛出 错误原因，使`Promise`走向`rejected`
 */
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    config.cancelToken.throwIfRequested();
  }
}

/**
 * Dispatch a request to the server using the configured adapter.
 *
 * @param {object} config The config that is to be used for the request
 * @returns {Promise} The Promise to be fulfilled
 */
module.exports = function dispatchRequest(config) {
  // 取消相关
  throwIfCancellationRequested(config);
  // 确保 headers 存在
  config.headers = config.headers || {};
  // 转换请求的数据
  config.data = transformData(
    config.data,
    config.headers,
    config.transformRequest
  );
  // Flatten headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );
  // 以下这些方法删除 headers
  utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );
```
## dispatchRequest 之 transformData 转换数据
transformData 其实就是遍历传递的函数数组 对数据操作，最后返回数据。

axios.defaults.transformResponse 数组中默认就有一个函数，所以使用concat链接自定义的函数。
```js
module.exports = function transformData(data, headers, fns) {
  /*eslint no-param-reassign:0*/
  utils.forEach(fns, function transform(fn) {
    data = fn(data, headers);
  });

  return data;
};
```
## dispatchRequest 之 adapter 适配器执行部分
```js
  var adapter = config.adapter || defaults.adapter;

  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // 转换响应的数据
    response.data = transformData(
      response.data,
      response.headers,
      config.transformResponse
    );

    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // 转换响应的数据
      if (reason && reason.response) {
        reason.response.data = transformData(
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }

    return Promise.reject(reason);
  });
};
```
## adapter 适配器 真正发送请求
根据当前环境引入，如果是浏览器环境引入xhr，是node环境则引入http。
```js
// axios/lib/defaults.js
function getDefaultAdapter() {
  var adapter;
  // 根据 XMLHttpRequest 判断
  if (typeof XMLHttpRequest !== "undefined") {
    // For browsers use XHR adapter
    adapter = require("./adapters/xhr");
    // 根据 process 判断
  } elseif (typeof process !== "undefined" && Object.prototype.toString.call(process) === "[object process]") {
    // For node use HTTP adapter
    adapter = require("./adapters/http");
  }
  return adapter;
}
var defaults = {
  adapter: getDefaultAdapter(),
  // ...
};
```
## xhr
```js
module.exports = function xhrAdapter(config) {
  returnnewPromise(function dispatchXhrRequest(resolve, reject) {
    // 这块代码有删减
    var request = new XMLHttpRequest();
    request.open()
    request.timeout = config.timeout;
    // 监听 state 改变
    request.onreadystatechange = function handleLoad() {
      if (!request || request.readyState !== 4) {
        return;
      }
      // ...
    }
    // 取消
    request.onabort = function(){};
    // 错误
    request.onerror = function(){};
    // 超时
    request.ontimeout = function(){};
    // cookies 跨域携带 cookies 面试官常喜欢考这个
    // 一个布尔值，用来指定跨域 Access-Control 请求是否应带有授权信息，如 cookie 或授权 header 头。
    // Add withCredentials to request if needed
    if (!utils.isUndefined(config.withCredentials)) {
      request.withCredentials = !!config.withCredentials;
    }

    // 上传下载进度相关
    // Handle progress if needed
    if (typeof config.onDownloadProgress === "function") {
      request.addEventListener("progress", config.onDownloadProgress);
    }

    // Not all browsers support upload events
    if (typeof config.onUploadProgress === "function" && request.upload) {
      request.upload.addEventListener("progress", config.onUploadProgress);
    }

    // Send the request
    // 发送请求
    request.send(requestData);
  });
}
```
# dispatchRequest 之 取消模块
可以使用cancel token取消请求。