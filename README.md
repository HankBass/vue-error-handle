# vue-error-handle

## Vue 官方提供的异常handle

### silent

  * 类型：`boolean`

  * 默认值：`false`

  * 用法：

  ```
    Vue.config.silent = true
  ```
  取消 Vue 所有的日志与警告。

### devtools 

  * 类型：`boolean`

  * 默认值：`true` (生产版为 `false`)

  * 用法：
  ```
    // 务必在加载 Vue 之后，立即同步设置以下内容
    Vue.config.devtools = true
  ```

  配置是否允许 [vue-devtools](https://github.com/vuejs/vue-devtools) 检查代码。开发版本默认为 `true`，生产版本默认为 `false`。生产版本设为 `true` 可以启用检查。

### errorHandler 

  * 类型：`Function`

  * 默认值：`undefined`

  * 用法：
  ```
    Vue.config.errorHandler = function (err, vm, info) {
      // handle error
      // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
      // 只在 2.2.0+ 可用
    }
  ```
  指定组件的渲染和观察期间未捕获错误的处理函数。这个处理函数被调用时，可获取错误信息和 Vue 实例。

  > 从 2.2.0 起，这个钩子也会捕获组件生命周期钩子里的错误。同样的，当这个钩子是 `undefined` 时，被捕获的错误会通过 `console.error` 输出而避免应用崩溃。

  > 从 2.4.0 起这个钩子也会捕获 Vue 自定义事件处理函数内部的错误了。

  > 错误追踪服务 [Sentry](https://sentry.io/) 和 [Bugsnag](https://docs.bugsnag.com/platforms/browsers/vue/) 都通过此选项提供了官方支持。

### warnHandler 

  > 2.4.0 新增

  * 类型：`Function`

  * 默认值：`undefined`

  * 用法：
  ```
    Vue.config.warnHandler = function (msg, vm, trace) {
      // `trace` 是组件的继承关系追踪
    }
  ```

  为 Vue 的运行时警告赋予一个自定义处理函数。注意这只会在开发者环境下生效，在生产环境下它会被忽略。

### 生命周期 errorCaptured

  > 2.5.0+ 新增

  * 类型：`(err: Error, vm: Component, info: string) => ?boolean`

  * 详细：

    当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 false 以阻止该错误继续向上传播。

    > 你可以在此钩子中修改组件的状态。因此在模板或渲染函数中设置其它内容的短路条件非常重要，它可以防止当一个错误被捕获时该组件进入一个无限的渲染循环。

    错误传播规则

      * 默认情况下，如果全局的 config.errorHandler 被定义，所有的错误仍会发送它，因此这些错误仍然会向单一的分析服务的地方进行汇报。

      * 如果一个组件的继承或父级从属链路中存在多个 errorCaptured 钩子，则它们将会被相同的错误逐个唤起。

      * 如果此 errorCaptured 钩子自身抛出了一个错误，则这个新错误和原本被捕获的错误都会发送给全局的 config.errorHandler。

      * 大家都在发大家都在发一个 errorCaptured 钩子能够返回 false 以阻止错误继续向上传播。本质上是说“这个错误已经被搞定了且应该被忽略”。它会阻止其它任何会被这个错误唤起的 errorCaptured 钩子和全局的 config.errorHandler。

## 常见的异常及解决方案

  * 安装超时(install timeout)

    常用的解决方案大概有：
    
    1. cnpm : 国内对npm的镜像版本
   
  ```
    npm install -g cnpm --registry=https://registry.npm.taobao.org
  ```

    2. yarn 和 npm 改源

       * 使用 nrm 模块 : www.npmjs.com/package/nrm
       * npm config : npm config set registry https://registry.npm.taobao.org
       * yarn config : yarn config set registry https://registry.npm.taobao.org

  * 安装一些需要编译的包:提示没有安装python、build失败等

    某些 npm 的包安装需要编译的环境,mac 和 linux 都还好,大多都齐全
window 用户依赖 visual studio 的一些库和python 2+,
    * [windows-build-tools](https://github.com/felixrieseberg/windows-build-tools)
    * [python 2.x](https://www.python.org/downloads/)


  * `can't not find 'xxModule'` - 找不到某些依赖或者模块

    一般情况下，看是那个包报错，卸载再重新安装即可

  * `data functions should return an object`

    这个问题是 Vue 实例内,单组件的data必须返回一个对象;如下

    ```
      export default {
        name: 'test',
            data () {
            return {
              obj:{name:"小明"}
            }
          }
      }
    ```

    为什么要 return 一个数据对象呢?

    > 当一个组件被定义，data 必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。如果 data 仍然是一个纯粹的对象，则所有的实例将共享引用同一个数据对象！通过提供 data 函数，每次创建一个新实例后，我们能够调用 data 函数，从而返回初始数据的一个全新副本数据对象。

  * 添加事件不生效
  
    ```
      <!--比如用了第三方框架,或者一些封装的内置组件; 然后想绑定事件-->

      <!-- 错误例子1-->
      <el-input placeholder="请输入内容 " @mouseover="test()"></el-input>

      <!-- 错误例子2-->
      <router-link :to="北京" @click="goBeiJing=''">
        <span>我要去北京</span>
      </router-link>

      <!--原因,少了一个修饰符 .native-->

      <router-link :to="北京" @click.native="goBeiJing=''">
        <span>我要去北京</span>
      </router-link>
      
    ```
    [官方解释](https://cn.vuejs.org/v2/guide/components-custom-events.html#%E5%B0%86%E5%8E%9F%E7%94%9F%E4%BA%8B%E4%BB%B6%E7%BB%91%E5%AE%9A%E5%88%B0%E7%BB%84%E4%BB%B6)

  * 我用了 `axios` , 为什么 IE 浏览器不识别(IE9+)

    因为 IE 不支持 promise, 解决方案:

    ```
      npm install es6-promise
      require("es6-promise").polyfill();
    ```

  * 使用了`this.xxx = `,抛出`Cannot set property 'xxx' of undefined` 的错误;

    this的指向变了

    解决方案:

      1. 暂存法: 函数内先缓存 this , let that = this;
      2. 箭头函数: 会关联当前运行区域为 this 的上下文;

  * `Component template shold contain exactly one root element.If you are useing v-if on multiple elements , xxxxx`

    每个组件必须只有一个根元素。你可以将模板的内容包裹在一个父元素内，来修复这个问题，例如：

    ```
      <div class="blog-post">
        <h3>{{ title }}</h3>
        <div v-html="content"></div>
      </div>
    ```

    [官方解释](https://cn.vuejs.org/v2/guide/components.html#%E5%8D%95%E4%B8%AA%E6%A0%B9%E5%85%83%E7%B4%A0)


  * `Invalid prop: type check failed for prop "xxx". Expected Boolean, got String.`

    一般情况下就是组件内的 props 类型已经设置了接受的范围类型,
而你传递的值却又不是它需要的类型。


  * `No 'Access-Control-Allow-Origin' header is present on the requested resource.`

    跨域问题：

    1. 开发环境下，vue-cli提供了`proxyTable`用于解决跨域问题

    ```
      // 在 config 目录下的index.js
        proxyTable: {
          "/api": {
            target: "",// api 的代理的实际路径
            changeOrigin: true, // 是否跨域
            // pathRewrite: { // 路径重定向
            //   "^/": "/"
            // }
          }
        }
    ```
    2. CORS , 前后端都要对应去配置,IE10+
    3. nginx 反向代理。
   
    关于跨域的处理方案，还有很多，这里就不详述了。

  * 对数组进行操作，数据更新了，但视图不更新

    由于 JavaScript 的限制，Vue 不能检测以下变动的数组,[官方文档](https://cn.vuejs.org/v2/guide/list.html#%E6%95%B0%E7%BB%84%E6%9B%B4%E6%96%B0%E6%A3%80%E6%B5%8B)有详细的描述。

    一般我们更常用的手段是使用[this.$set](https://cn.vuejs.org/v2/api/#Vue-set)或者是使用Vue官方提供的数组[变异方法](https://cn.vuejs.org/v2/guide/list.html#%E5%8F%98%E5%BC%82%E6%96%B9%E6%B3%95);

  * 组件间的样式不能继承或者覆写？

    单组件开发模式下,请确认是否开启了 CSS模块化功能!!也就是`scoped`

    ```
      <style lang="scss" scoped></style>
    ```

    因为在`scoped`下，每个类或者 id 乃至标签都会自动在css后面添加hash值，以保证该css的样式是私有的!

   

    比如

    ```
      // 写的时候是这个
      .try{}

      // 编译过后,加上了 hash
      .try[data-v-1ec35ffc]{}
    ```
