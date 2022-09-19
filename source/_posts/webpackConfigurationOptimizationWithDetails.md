---
title: Webpack配置优化与详情
excerpt: 本文的内容是是webpack优化与详解
categories:
- 技术文章
tags:
- webpack
---

## 开发环境性能优化
### HMR（模块热替换）
HMR: hot module replacement 热模块替换 / 模块热替换
作用：一个模块发生变化，只会重新打包构建这一个模块（而不是打包所有模块） ，极大提升构建速度
代码：只需要在 devServer 中设置 hot 为 true，就会自动开启HMR功能（只能在开发模式下使用）
```javascript
devServer: {
  contentBase: resolve(__dirname, 'build'),
  compress: true,
  port: 3000,
  open: true,
  // 开启HMR功能
  // 当修改了webpack配置，新配置要想生效，必须重启webpack服务
  hot: true
}
```

每种文件实现热模块替换的情况：
- 样式文件：可以使用HMR功能，因为开发环境下使用的 style-loader 内部默认实现了热模块替换功能

- js 文件：默认不能使用HMR功能（修改一个 js 模块所有 js 模块都会刷新）
--> 实现 HMR 需要修改 js 代码（添加支持 HMR 功能的代码）
```javascript
// 绑定
if (module.hot) {
  // 一旦 module.hot 为true，说明开启了HMR功能。 --> 让HMR功能代码生效
  module.hot.accept('./print.js', function() {
    // 方法会监听 print.js 文件的变化，一旦发生变化，只有这个模块会重新打包构建，其他模块不会。
    // 会执行后面的回调函数
    print();
  });
}
```
注意：HMR 功能对 js 的处理，只能处理非入口 js 文件的其他文件。

- html 文件: 默认不能使用 HMR 功能（html 不用做 HMR 功能，因为只有一个 html 文件，不需要再优化）
使用 HMR 会导致问题：html 文件不能热更新了（不会自动打包构建）
解决：修改 entry 入口，将 html 文件引入（这样 html 修改整体刷新）
```javascript
entry: ['./src/js/index.js', './src/index.html']
```

### source-map
source-map：一种提供源代码到构建后代码的映射的技术 （如果构建后代码出错了，通过映射可以追踪源代码错误）
参数：[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map
code:
```javascript
devtool: 'eval-source-map'
```
可选方案：[生成source-map的位置|给出的错误代码信息]
- source-map：外部，错误代码准确信息 和 源代码的错误位置
- inline-source-map：内联，只生成一个内联 source-map，错误代码准确信息 和 源代码的错误位置
- hidden-source-map：外部，错误代码错误原因，但是没有错误位置（为了隐藏源代码），不能追踪源代码错误，只能提示到构建后代码的错误位置
- eval-source-map：内联，每一个文件都生成对应的 source-map，都在 eval 中，错误代码准确信息 和 源代码的错误位
- nosources-source-map：外部，错误代码准确信息，但是没有任何源代码信息（为了隐藏源代码）
- cheap-source-map：外部，错误代码准确信息 和 源代码的错误位置，只能把错误精确到整行，忽略列
- cheap-module-source-map：外部，错误代码准确信息 和 源代码的错误位置，module 会加入 loader 的 source-map

内联和外部的区别：1. 外部生成了文件，内联没有 2. 内联构建速度更快
开发/生产环境可做的选择：
**开发环境：**需要考虑速度快，调试更友好
- 速度快( eval > inline > cheap >... )
eval-cheap-souce-map
eval-source-map
- 调试更友好
souce-map
cheap-module-souce-map
cheap-souce-map

最终得出最好的两种方案 --> eval-source-map（完整度高，内联速度快） / eval-cheap-module-souce-map（错误提示忽略列但是包含其他信息，内联速度快）

**生产环境：**需要考虑源代码要不要隐藏，调试要不要更友好
- 内联会让代码体积变大，所以在生产环境不用内联
- 隐藏源代码
nosources-source-map 全部隐藏
hidden-source-map 只隐藏源代码，会提示构建后代码错误信息

最终得出最好的两种方案 --> source-map（最完整） / cheap-module-souce-map（错误提示一整行忽略列）

## 生产环境性能优化
### 优化打包构建速度
#### oneOf
oneOf：匹配到 loader 后就不再向后进行匹配，优化生产环境的打包构建速度
code:
```javascript
module: {
    rules: [{
            // js 语法检查
            test: /\.js$/,
            exclude: /node_modules/,
            // 优先执行
            enforce: 'pre',
            loader: 'eslint-loader',
            options: {
                fix: true
            }
        },
        {
            // oneOf 优化生产环境的打包构建速度
            // 以下loader只会匹配一个（匹配到了后就不会再往下匹配了）
            // 注意：不能有两个配置处理同一种类型文件（所以把eslint-loader提取出去放外面）
            oneOf: [{
                    test: /\.css$/,
                    use: [...commonCssLoader]
                },
                {
                    test: /\.less$/,
                    use: [...commonCssLoader, 'less-loader']
                },
                {
                    // js 兼容性处理
                    test: /\.js$/,
                    exclude: /node_modules/,
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            [
                                '@babel/preset-env',
                                {
                                    useBuiltIns: 'usage',
                                    corejs: { version: 3 },
                                    targets: {
                                        chrome: '60',
                                        firefox: '50'
                                    }
                                }
                            ]
                        ]
                    }
                },
                {
                    test: /\.(jpg|png|gif)/,
                    loader: 'url-loader',
                    options: {
                        limit: 8 * 1024,
                        name: '[hash:10].[ext]',
                        outputPath: 'imgs',
                        esModule: false
                    }
                },
                {
                    test: /\.html$/,
                    loader: 'html-loader'
                },
                {
                    exclude: /\.(js|css|less|html|jpg|png|gif)/,
                    loader: 'file-loader',
                    options: {
                        outputPath: 'media'
                    }
                }
            ]
        }
    ]
},
```

####  babel 缓存
babel 缓存：类似 HMR，将 babel 处理后的资源缓存起来（哪里的 js 改变就更新哪里，其他 js 还是用之前缓存的资源），让第二次打包构建速度更快
code:
```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    loader: 'babel-loader',
    options: {
        presets: [
            [
                '@babel/preset-env',
                {
                    useBuiltIns: 'usage',
                    corejs: { version: 3 },
                    targets: {
                        chrome: '60',
                        firefox: '50'
                    }
                }
            ]
        ],
        // 开启babel缓存
        // 第二次构建时，会读取之前的缓存
        cacheDirectory: true
    }
},
```

**文件资源缓存**
文件名不变，就不会重新请求，而是再次用之前缓存的资源
1. hash: 每次 wepack 打包时会生成一个唯一的 hash 值。
​ 问题：重新打包，所有文件的 hsah 值都改变，会导致所有缓存失效。（可能只改动了一个文件）

2. chunkhash：根据 chunk 生成的 hash 值。来源于同一个 chunk的 hash 值一样
​ 问题：js 和 css 来自同一个chunk，hash 值是一样的（因为 css-loader 会将 css 文件加载到 js 中，所以同属于一个chunk）

3. contenthash: 根据文件的内容生成 hash 值。不同文件 hash 值一定不一样(文件内容修改，文件名里的 hash 才会改变)
修改 css 文件内容，打包后的 css 文件名 hash 值就改变，而 js 文件没有改变 hash 值就不变，这样 css 和 js 缓存就会分开判断要不要重新请求资源 --> 让代码上线运行缓存更好使用

#### 多进程打包
多进程打包：某个任务消耗时间较长会卡顿，多进程可以同一时间干多件事，效率更高。
优点是提升打包速度，缺点是每个进程的开启和交流都会有开销（babel-loader消耗时间最久，所以使用thread-loader针对其进行优化）
```javascript
{
    test: /\.js$/,
    exclude: /node_modules/,
    use: [
        /* 
          thread-loader会对其后面的loader（这里是babel-loader）开启多进程打包。 
          进程启动大概为600ms，进程通信也有开销。(启动的开销比较昂贵，不要滥用)
          只有工作消耗时间比较长，才需要多进程打包
        */
        {
            loader: 'thread-loader',
            options: {
                workers: 2 // 进程2个
            }
        },
        {
            loader: 'babel-loader',
            options: {
                presets: [
                    [
                        '@babel/preset-env',
                        {
                            useBuiltIns: 'usage',
                            corejs: { version: 3 },
                            targets: {
                                chrome: '60',
                                firefox: '50'
                            }
                        }
                    ]
                ],
                // 开启babel缓存
                // 第二次构建时，会读取之前的缓存
                cacheDirectory: true
            }
        }
    ]
},
```

#### externals
externals：让某些库不打包，通过 cdn 引入
webpack.config.js 中配置：
```javascript
externals: {
    // 拒绝jQuery被打包进来(通过cdn引入，速度会快一些)
    // 忽略的库名 -- npm包名
    jquery: 'jQuery'
}
```

需要在 index.html 中通过 cdn 引入：
```html
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
```

#### dll
dll：让某些库单独打包，后直接引入到 build 中。可以在 code split 分割出 node_modules 后再用 dll 更细的分割，优化代码运行的性能。
webpack.dll.js 配置：(将 jquery 单独打包)
```javascript
/*
  node_modules的库会打包到一起，但是很多库的时候打包输出的js文件就太大了
  使用dll技术，对某些库（第三方库：jquery、react、vue...）进行单独打包
  当运行webpack时，默认查找webpack.config.js配置文件
  需求：需要运行webpack.dll.js文件
    --> webpack --config webpack.dll.js（运行这个指令表示以这个配置文件打包）
*/
const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
    entry: {
        // 最终打包生成的[name] --> jquery
        // ['jquery] --> 要打包的库是jquery
        jquery: ['jquery']
    },
    output: {
        // 输出出口指定
        filename: '[name].js', // name就是jquery
        path: resolve(__dirname, 'dll'), // 打包到dll目录下
        library: '[name]_[hash]', // 打包的库里面向外暴露出去的内容叫什么名字
    },
    plugins: [
        // 打包生成一个manifest.json --> 提供jquery的映射关系（告诉webpack：jquery之后不需要再打包和暴露内容的名称）
        new webpack.DllPlugin({
            name: '[name]_[hash]', // 映射库的暴露的内容名称
            path: resolve(__dirname, 'dll/manifest.json') // 输出文件路径
        })
    ],
    mode: 'production'
};
```

webpack.config.js 配置：(告诉 webpack 不需要再打包 jquery，并将之前打包好的 jquery 跟其他打包好的资源一同输出到 build 目录下)

```javascript
// 引入插件
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

// plugins中配置：
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html'
  }),
  // 告诉webpack哪些库不参与打包，同时使用时的名称也得变
  new webpack.DllReferencePlugin({
    manifest: resolve(__dirname, 'dll/manifest.json')
  }),
  // 将某个文件打包输出到build目录下，并在html中自动引入该资源
  new AddAssetHtmlWebpackPlugin({
    filepath: resolve(__dirname, 'dll/jquery.js')
  })
],
```

### 优化代码运行的性能
#### 缓存
#### tree shaking（树摇）
tree shaking：去除无用代码
前提：1. 必须使用 ES6 模块化 2. 开启 production 环境 （这样就自动会把无用代码去掉）
作用：减少代码体积
在 package.json 中配置：
"sideEffects": false 表示所有代码都没有副作用（都可以进行 tree shaking）
这样会导致的问题：可能会把 css / @babel/polyfill 文件干掉（副作用）
所以可以配置："sideEffects": ["*.css", "*.less"] 不会对css/less文件tree shaking处理

#### code split（代码分割）
代码分割。将打包输出的一个大的 bundle.js 文件拆分成多个小文件，这样可以并行加载多个文件，比加载一个文件更快。
1. 多入口拆分
```javascript
entry: {
    // 多入口：有一个入口，最终输出就有一个bundle
    index: './src/js/index.js',
    test: './src/js/test.js'
},
output: {
    // [name]：取文件名
    filename: 'js/[name].[contenthash:10].js',
    path: resolve(__dirname, 'build')
},
```

2. optimization：
```javascript
optimization: {
    splitChunks: {
        chunks: 'all'
    }
},
```
- 将 node_modules 中的代码单独打包（大小超过30kb）
- 自动分析多入口chunk中，有没有公共的文件。如果有会打包成单独一个chunk(比如两个模块中都引入了jquery会被打包成单独的文件)（大小超过30kb）

3. import 动态导入语法：
```javascript
/*
  通过js代码，让某个文件被单独打包成一个chunk
  import动态导入语法：能将某个文件单独打包(test文件不会和index打包在同一个文件而是单独打包)
  webpackChunkName:指定test单独打包后文件的名字
*/
import ( /* webpackChunkName: 'test' */ './test')
.then(({ mul, count }) => {
    // 文件加载成功~
    // eslint-disable-next-line
    console.log(mul(2, 5));
})
.catch(() => {
    // eslint-disable-next-line
    console.log('文件加载失败~');
});
```

#### lazy loading（懒加载/预加载）
1. 懒加载：当文件需要使用时才加载（需要代码分割）。但是如果资源较大，加载时间就会较长，有延迟。
2. 正常加载：可以认为是并行加载（同一时间加载多个文件）没有先后顺序，先加载了不需要的资源就会浪费时间。
3. 预加载 prefetch（兼容性很差）：会在使用之前，提前加载。等其他资源加载完毕，浏览器空闲了，再偷偷加载这个资源。这样在使用时已经加载好了，速度很快。所以在懒加载的基础上加上预加载会更好。
code:
```javascript
document.getElementById('btn').onclick = function() {
    // 将import的内容放在异步回调函数中使用，点击按钮，test.js才会被加载(不会重复加载)
    // webpackPrefetch: true表示开启预加载
    import ( /* webpackChunkName: 'test', webpackPrefetch: true */ './test').then(({ mul }) => {
        console.log(mul(4, 5));
    });
    import ('./test').then(({ mul }) => {
        console.log(mul(2, 5))
    })
};
```

#### pwa（离线可访问技术）
pwa：离线可访问技术（渐进式网络开发应用程序），使用 serviceworker 和 workbox 技术。优点是离线也能访问，缺点是兼容性差。
webpack.config.js 中配置：
```javascript
const WorkboxWebpackPlugin = require('workbox-webpack-plugin'); // 引入插件

// plugins中加入：
new WorkboxWebpackPlugin.GenerateSW({
    /*
      1. 帮助serviceworker快速启动
      2. 删除旧的 serviceworker

      生成一个 serviceworker 配置文件
    */
    clientsClaim: true,
    skipWaiting: true
})
```

index.js 中还需要写一段代码来激活它的使用：
```javascript
/*
  1. eslint不认识 window、navigator全局变量
    解决：需要修改package.json中eslintConfig配置
    "env": {
      "browser": true // 支持浏览器端全局变量
    }
  2. sw代码必须运行在服务器上
    --> nodejs
    或-->
      npm i serve -g
      serve -s build 启动服务器，将打包输出的build目录下所有资源作为静态资源暴露出去
*/
if ('serviceWorker' in navigator) { // 处理兼容性问题
    window.addEventListener('load', () => {
        navigator.serviceWorker
            .register('/service-worker.js') // 注册serviceWorker
            .then(() => {
                console.log('sw注册成功了~');
            })
            .catch(() => {
                console.log('sw注册失败了~');
            });
    });
}
```

## Webpack 配置详情
### entry
entry: 入口起点
1. string --> './src/index.js'，单入口
打包形成一个 chunk。 输出一个 bundle 文件。此时 chunk 的名称默认是 main

2. array --> ['./src/index.js', './src/add.js']，多入口
所有入口文件最终只会形成一个 chunk，输出出去只有一个 bundle 文件（一般只用在 HMR 功能中让 html 热更新生效）。

3. object，多入口
有几个入口文件就形成几个 chunk，输出几个 bundle 文件，此时 chunk 的名称是 key 值

特殊用法：
```javascript
entry: {
    // 最终只会形成一个chunk, 输出出去只有一个bundle文件。
    index: ['./src/index.js', './src/count.js'],
    // 形成一个chunk，输出一个bundle文件。
    add: './src/add.js'
}
```

### output
```javascript
output: {
    // 文件名称（指定名称+目录）
    filename: 'js/[name].js',
    // 输出文件目录（将来所有资源输出的公共目录）
    path: resolve(__dirname, 'build'),
    // 所有资源引入公共路径前缀 --> 'imgs/a.jpg' --> '/imgs/a.jpg'
    publicPath: '/',
    chunkFilename: 'js/[name]_chunk.js', // 指定非入口chunk的名称
    library: '[name]', // 打包整个库后向外暴露的变量名
    libraryTarget: 'window' // 变量名添加到哪个上 browser：window
        // libraryTarget: 'global' // node：global
        // libraryTarget: 'commonjs' // conmmonjs模块 exports
},
```

### module
```javascript
module: {
    rules: [
        // loader的配置
        {
            test: /\.css$/,
            // 多个loader用use
            use: ['style-loader', 'css-loader']
        },
        {
            test: /\.js$/,
            // 排除node_modules下的js文件
            exclude: /node_modules/,
            // 只检查src下的js文件
            include: resolve(__dirname, 'src'),
            enforce: 'pre', // 优先执行
            // enforce: 'post', // 延后执行
            // 单个loader用loader
            loader: 'eslint-loader',
            options: {} // 指定配置选项
        },
        {
            // 以下配置只会生效一个
            oneOf: []
        }
    ]
},
```

### resolve
```javascript
// 解析模块的规则
resolve: {
    // 配置解析模块路径别名: 优点：当目录层级很复杂时，简写路径；缺点：路径不会提示
    alias: {
        $css: resolve(__dirname, 'src/css')
    },
    // 配置省略文件路径的后缀名（引入时就可以不写文件后缀名了）
    extensions: ['.js', '.json', '.jsx', '.css'],
    // 告诉 webpack 解析模块应该去找哪个目录
    modules: [resolve(__dirname, '../../node_modules'), 'node_modules']
}
```
这样配置后，引入文件就可以这样简写：import '$css/index';

### dev server
```javascript
devServer: {
    // 运行代码所在的目录
    contentBase: resolve(__dirname, 'build'),
    // 监视contentBase目录下的所有文件，一旦文件变化就会reload
    watchContentBase: true,
    watchOptions: {
        // 忽略文件
        ignored: /node_modules/
    },
    // 启动gzip压缩
    compress: true,
    // 端口号
    port: 5000,
    // 域名
    host: 'localhost',
    // 自动打开浏览器
    open: true,
    // 开启HMR功能
    hot: true,
    // 不要显示启动服务器日志信息
    clientLogLevel: 'none',
    // 除了一些基本信息外，其他内容都不要显示
    quiet: true,
    // 如果出错了，不要全屏提示
    overlay: false,
    // 服务器代理，--> 解决开发环境跨域问题
    proxy: {
        // 一旦devServer(5000)服务器接收到/api/xxx的请求，就会把请求转发到另外一个服务器3000
        '/api': {
            target: 'http://localhost:3000',
            // 发送请求时，请求路径重写：将/api/xxx --> /xxx （去掉/api）
            pathRewrite: {
                '^/api': ''
            }
        }
    }
}
```

其中，跨域问题：同源策略中不同的协议、端口号、域名就会产生跨域。

正常的浏览器和服务器之间有跨域，但是服务器之间没有跨域。代码通过代理服务器运行，所以浏览器和代理服务器之间没有跨域，浏览器把请求发送到代理服务器上，代理服务器替你转发到另外一个服务器上，服务器之间没有跨域，所以请求成功。代理服务器再把接收到的响应响应给浏览器。这样就解决开发环境下的跨域问题。

### optimization
contenthash 缓存会导致一个问题：修改 a 文件导致 b 文件 contenthash 变化。
因为在 index.js 中引入 a.js，打包后 index.js 中记录了 a.js 的 hash 值，而 a.js 改变，其重新打包后的 hash 改变，导致 index.js 文件内容中记录的 a.js 的 hash 也改变，从而重新打包后 index.js 的 hash 值也会变，这样就会使缓存失效。（改变的是a.js文件但是 index.js 文件的 hash 值也改变了）
解决办法：runtimeChunk --> 将当前模块记录其他模块的 hash 单独打包为一个文件 runtime，这样 a.js 的 hash 改变只会影响 runtime 文件，不会影响到 index.js 文件
```javascript
output: {
        filename: 'js/[name].[contenthash:10].js',
        path: resolve(__dirname, 'build'),
        chunkFilename: 'js/[name].[contenthash:10]_chunk.js' // 指定非入口文件的其他chunk的名字加_chunk
    },
    optimization: {
        splitChunks: {
            chunks: 'all',
            /* 以下都是splitChunks默认配置，可以不写
            miniSize: 30 * 1024, // 分割的chunk最小为30kb（大于30kb的才分割）
            maxSize: 0, // 最大没有限制
            minChunks: 1, // 要提取的chunk最少被引用1次
            maxAsyncRequests: 5, // 按需加载时并行加载的文件的最大数量为5
            maxInitialRequests: 3, // 入口js文件最大并行请求数量
            automaticNameDelimiter: '~', // 名称连接符
            name: true, // 可以使用命名规则
            cacheGroups: { // 分割chunk的组
              vendors: {
                // node_modules中的文件会被打包到vendors组的chunk中，--> vendors~xxx.js
                // 满足上面的公共规则，大小超过30kb、至少被引用一次
                test: /[\\/]node_modules[\\/]/,
                // 优先级
                priority: -10
              },
              default: {
                // 要提取的chunk最少被引用2次
                minChunks: 2,
                prority: -20,
                // 如果当前要打包的模块和之前已经被提取的模块是同一个，就会复用，而不是重新打包
                reuseExistingChunk: true
              }
            } */
        },
        // 将index.js记录的a.js的hash值单独打包到runtime文件中
        runtimeChunk: {
            name: entrypoint => `runtime-${entrypoint.name}`
        },
        minimizer: [
            // 配置生产环境的压缩方案：js/css
            new TerserWebpackPlugin({
                // 开启缓存
                cache: true,
                // 开启多进程打包
                parallel: true,
                // 启用sourceMap(否则会被压缩掉)
                sourceMap: true
            })
        ]
    }
```

## Webpack5 介绍和使用
此版本重点关注以下内容：
- 通过持久缓存提高构建性能
- 使用更好的算法和默认值来改善长期缓存
- 通过更好的树摇和代码生成来改善捆绑包大小
- 清除处于怪异状态的内部结构，同时在 v4 中实现功能而不引入任何重大更改
- 通过引入重大更改来为将来的功能做准备，以使我们能够尽可能长时间地使用 v5

### 下载
```javascript
npm i webpack@next webpack-cli -D
```

### 自动删除 Node.js Polyfills
早期，webpack 的目标是允许在浏览器中运行大多数 node.js 模块，但是模块格局发生了变化，许多模块用途现在主要是为前端目的而编写的。webpack <= 4 附带了许多 node.js 核心模块的 polyfill，一旦模块使用任何核心模块（即 crypto 模块），这些模块就会自动应用。
尽管这使使用为 node.js 编写的模块变得容易，但它会将这些巨大的 polyfill 添加到包中。在许多情况下，这些 polyfill 是不必要的。
webpack 5 会自动停止填充这些核心模块，并专注于与前端兼容的模块。
迁移：
- 尽可能尝试使用与前端兼容的模块。
- 可以为 node.js 核心模块手动添加一个 polyfill。错误消息将提示如何实现该目标。

Chunk 和模块 ID
添加了用于长期缓存的新算法。在生产模式下默认情况下启用这些功能。
```javascript
chunkIds: "deterministic", moduleIds: "deterministic"
```

### Chunk ID
你可以不用使用 import(/* webpackChunkName: "name" */ "module") 在开发环境来为 chunk 命名，生产环境还是有必要的
webpack 内部有 chunk 命名规则，不再是以 id(0, 1, 2)命名了

### Tree Shaking
1. webpack 现在能够处理对嵌套模块的 tree shaking
```javascript
// inner.js
export const a = 1;
export const b = 2;

// module.js
import * as inner from './inner';
export { inner };

// user.js
import * as module from './module';
console.log(module.inner.a);
```
在生产环境中, inner 模块暴露的 b 会被删除

2. webpack 现在能够多个模块之前的关系
```javascript
import { something } from './something';

function usingSomething() {
    return something;
}

export function test() {
    return usingSomething();
}
```
当设置了"sideEffects": false时，一旦发现test方法没有使用，不但删除test，还会删除"./something"

3. webpack 现在能处理对 Commonjs 的 tree shaking

### Output
webpack 4 默认只能输出 ES5 代码
webpack 5 开始新增一个属性 output.ecmaVersion, 可以生成 ES5 和 ES6 / ES2015 代码.
如：output.ecmaVersion: 2015

### SplitChunk
```javascript
// webpack4
minSize: 30000;
// webpack5
minSize: {
    javascript: 30000,
    style: 50000,
}
```

### Caching
```javascript
// 配置缓存
cache: {
    // 磁盘存储
    type: "filesystem",
    buildDependencies: {
        // 当配置修改时，缓存失效
        config: [__filename]
    }
}
```
缓存将存储到 node_modules/.cache/webpack

### 监视输出文件
之前 webpack 总是在第一次构建时输出全部文件，但是监视重新构建时会只更新修改的文件。
此次更新在第一次构建时会找到输出文件看是否有变化，从而决定要不要输出全部文件。

### 默认值
- entry: "./src/index.js
- output.path: path.resolve(__dirname, "dist")
- output.filename: "[name].js"

## 参考链接
[尚硅谷](https://www.bilibili.com/video/BV1e7411j7T5)
[Webpack入门-学习总结](http://www.woc12138.com/article/45)