# 从 vue-cli2.X 到 vue-cli3.X

## 1.前言

### 1.新特新

1.功能丰富：对 babel、Typescript、ESLint...提供开箱即用的支持  
2.无需 Eject：vue cli 完全可配置的，无需再使用 webpack 配置  
3.一套完全图形化的创建和管理 Vue.js 项目的用户界面  
4.易于扩展：一个丰富的官方插件集合，集成了前端生态中最好的工具。  
5.面向未来：为现代浏览器轻松产出原生的 ES2015 代码，或将你的 vue 组件构建为原生的 Web Components 组件

### 2.对比

> 对比可以发现 vue-cli3 的项目结构更加简单，没有了相关配置文件（bulid、config），CLI 3 仅生成构建应用程序所需的文件，让使用者不用关心这些工具的具体配置，从而降低了工具的使用难度。与此同时，它也为每个工具提供了调整配置的灵活性，无需 eject。

vue-cli2 　　　　　　　　　　　　　　　　　　　　　　　　　 vue-cli3

![vue-cli2](./old-project.png 'Image Title') 　　　　　　　　　　　　![vue-cli3](./new-project.jpg 'Image Title')

## 2.开始

> Vue CLI 的包名称由 vue-cli 改成了 @vue/cli。 如果你已经全局安装了旧版本的 vue-cli (1.x 或 2.x)，你需要先通过 npm uninstall vue-cli -g 或 yarn global remove vue-cli 卸载它。

```
  npm install -g @vue/cli
  vue --version //查看版本
  vue create test
```

同时，也可以使用 vue ui 来管理和创建项目

```
  vue ui
```

详情可见[https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create](https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create)

## 3.项目结构

```
├── dist                     # 打包生成文件
├── public
│   ├── favicon.ico          # Favicon
│   └── index.html
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── lang                 # 语言翻译
│   ├── views                # 业务页面
│   ├── router               # 路由管理
│   ├── services             # 后台接口服务
│   ├── store                # vuex状态管理
│   ├── styles               # 样式库
│   ├── utils                # 工具库
│   ├── App.vue              # 全局入口视图页面
│   └── main.js              # 全局 JS
│
├── .env.development         #开发环境环境变量
├── .env.test                #测试环境环境变量
├── .env.production          #生产环境环境变量
├── vue.config.js
├── README.md
├── babel.config.js
└── .eslintrc.js
```

- vue-cli3 的 index.html 以及 favicon.ico 从根目录移到了 public 目录下面
- .env.\*自定义了不同环境的环境变量，定义了不同环境下 api 的地址

## 4.更改配置

### 1.修改配置

部分配置文件如下：

```
const CompressionWebpackPlugin = require('compression-webpack-plugin')
// const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  //放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录
  assetsDir: './static',

  //如果你不需要生产环境的 source map，可以将其设置为 false 以加速生产环境构建
  productionSourceMap: process.env.NODE_ENV === 'development',

  //设置为 true 时，eslint-loader 会将 lint 错误输出为编译警告。默认情况下，警告仅仅会被输出到命令行，且不会使得编译失败。这里关闭生产环境的eslint
  lintOnSave: process.env.NODE_ENV !== 'production',

  //开发环境配置
  devServer: {
    //跨域代理
    proxy: {
      '/api': {
        target: 'xxx',
        changeOrigin: true,
        //替换/api为 ''
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    host: 'xxx.xxx.com',
    port: 9999
  },

  //如果这个值是一个对象，则会通过 webpack-merge 合并到最终的配置中。
  //如果这个值是一个函数，则会接收被解析的配置作为参数。该函数及可以修改配置并不返回任何东西，也可以返回一个被克隆或合并过的配置版本。
  configureWebpack: config => {
    //配置gzip插件
    let plugins = [
      new CompressionWebpackPlugin({
        filename: '[path].gz[query]',
        algorithm: 'gzip',
        test: new RegExp('\\.(' + ['js', 'css'].join('|') + ')$'),
        threshold: 10240,
        minRatio: 0.8
      })
    ]
    if (process.env.NODE_ENV !== 'development') {
      config.plugins = [...config.plugins, ...plugins]
      config.performance = {
        hints: false
      }
    }
  },

  //是一个函数，会接收一个基于 webpack-chain 的 ChainableConfig 实例。允许对内部的 webpack 配置进行更细粒度的修改。
  chainWebpack: config => {
    // config.optimization.splitChunks({
    //   cacheGroups: {}
    // })
    config.plugins.delete('prefetch')
    config.plugins.delete('preload')
  },

  // 构建时开启多进程处理 babel 编译
  parallel: require('os').cpus().length > 1
}

```

vue-cli3 的打包配置文件也可以在\_@vue_cli-service@3.5.1@@vue/cli-service/lib/config 找到  
其他基础配置请见
详情可见[https://cli.vuejs.org/zh/config/#vue-config-js](https://cli.vuejs.org/zh/config/#vue-config-js)

### 2.分环境打包

package.js

```
  {
    "script": {
      "serve": "vue-cli-service serve",  //开发
      "test": "vue-cli-service build --mode test", //测试打包
      "build": "vue-cli-service build" //生产环境打包
    }
  }
```

配置环境变量 .env.development

```
NODE_ENV='development'
VUE_APP_CURRENTMODE='development'
VUE_APP_BASEURL='/api'
```

配置环境变量 .env.development

```
NODE_ENV='development'
VUE_APP_CURRENTMODE='development'
VUE_APP_BASEURL='/api'
```

配置环境变量 .env.test

```
NODE_ENV='production'
VUE_APP_CURRENTMODE='test'
VUE_APP_BASEURL='https://api-test.xxx.com'
```

配置环境变量 .env.production

```
NODE_ENV='production'
VUE_APP_CURRENTMODE='production'
VUE_APP_BASEURL='https://api.xxx.com'
```

默认的 vue-cli3 提供了 dev 和 prod 的 webpack 打包，环境变量主要区分了开发模式和生产模式，测试模式的打包方式和生产模式一样，只是 api 接口地址不一样。

### 3.审查配置

> 因为 @vue/cli-service 对 webpack 配置进行了抽象，所以理解配置中包含的东西会比较困难，尤其是当你打算自行对其调整的时候。
> vue-cli-service 暴露了 inspect 命令用于审查解析好的 webpack 配置

你可以将其输出重定向到一个文件以便进行查阅：

```
vue inspect > output.js //默认导出develop环境的配置，你可以加上--mode 导出不同环境配置
```

或者指向一个规则或插件的名字：

```
vue inspect --rule vue
vue inspect --plugin html
```

最后，你可以列出所有规则和插件的名字：

```
vue inspect --rules
vue inspect --plugins
```

## 5.把使用 vue-cli2.x 的项目迁移到新项目

steps:

- 使用 vue-cli3 脚手架新建一个项目
- 替换生成项目中的 index.html 文件和 favicon.ico
- 在 package.json 中添加需要用到的 npm 包，并且安装
- 新建 vue.config.js 文件修改开发环境的配置，以及进行一些特定的配置

problems:

1..vue 文件中的 style 标签 lang=""，未指定语言，报错

![lang-empty](./lang-empty.jpg 'Image Title')  
解决方法，删除 lang=""，或者指定语言

2.warning:  
![autofixer](./autofixer.jpg 'Image Title')  
解决方法

```
/*! autoprefixer: off */
-webkit-box-orient: vertical;
/* autoprefixer: on */

//css预处理语句以上写法应该是控制一个块，而不是一行，因此改为

/*! autoprefixer: ignore next */
-webkit-box-orient: vertical;
```

3.vue-template-compiler 版本问题  
![vue-template-compiler](./autofixer.jpg 'Image Title')  
解决方法：

```
//package.json

"vue-template-compiler": "^2.5.21"

//明确指定版本，去掉版本号前方的^
"vue-template-compiler": "2.5.21"
```

4.项目运行成功浏览器报错

![runtime](./runtime.jpg 'Image Title')  
原因：因为 vue-cli3 配置中引用的 vue 是 vue/dist/vue.runtime.esm.js，不包含变异器，要么改写 main.js 中的 Vue 初始化的写法，要么配置 alias 中的 vue 为带编译器的版本。  
解决方法：

```
let main = new Vue({
  el: '#app',
  router,
  i18n,
  store,
  components: {
    App
  },
  template: '<App/>'
})

//修改为

let main = new Vue({
  router,
  i18n,
  store,
  render: h => h(App)
}).$mount('#app')
```

5.可能会报缺少相应文件 loader 的错误，需要自己在 vue.config.js 中配置  
6.eslint 报出的很多语法错误，--fix 修复不了的需要自己手动修改
