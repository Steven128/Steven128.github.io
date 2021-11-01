---
title: webpack
date: 2021-11-01 10:29:36
tags: webpack
category: webpack
---

# Webpack

![Webpack](webpack-1.png)

<!-- more -->

## 什么是 Webpack

Webpack 是一个**模块打包工具** (module bundler)，在 Webpack 里一切文件皆模块。通过 loader 转换文件，通过 plugin 注入钩子，最后输出由多个模块组合的文件。Webpack 专注构建模块化项目。

> 当 Webpack 处理应用程序时，它会递归地构建一个依赖关系图 (dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
>
> Webpack 就像一条生产线，要经过一系列处理流程后才能将源文件转换成输出结果。 这条生产线上的每个处理流程的职责都是单一的，多个流程之间有存在依赖关系，只有完成当前处理后才能交给下一个流程去处理。插件就像是一个插入到生产线中的一个功能，在特定的时机对生产线上的资源做处理。
>
> Webpack 通过 Tapable 来组织这条复杂的生产线。 Webpack 在运行过程中会广播事件，插件只需要监听它所关心的事件，就能加入到这条生产线中，去改变生产线的运作。 Webpack 的事件流机制保证了插件的有序性，使得整个系统扩展性很好。 -- 深入浅出 Webpack 吴浩麟

![Webpack](webpack-2.png)

## Webpack 核心概念

### 1. Entry

入口起点 (entry point) 指示 Webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。

进入入口起点后，Webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

每个依赖项随即被处理，最后输出到称之为 bundles 的文件中。

### 2. Output

Output 属性告诉 Webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 `./dist`。

基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。

### 3. Module

模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。

### 4. Chunk

代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。

### 5. Loader

Loader 让 Webpack 能够去处理那些非 JavaScript 文件（Webpack 自身只理解 JavaScript）。

Loader 可以将所有类型的文件转换为 Webpack 能够处理的有效模块，然后你就可以利用 Webpack 的打包能力，对它们进行处理。

本质上，Webpack Loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

### 6. Plugin

Loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。

插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

## Webpack 构建流程

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. **初始化参数**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数。
2. **开始编译**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译。
3. **确定入口**：根据配置中的 entry 找出所有的入口文件。
4. **编译模块**：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理。
    > 可以分为三个部分帮助记忆：
    >
    > - 调用各 loader 处理模块之间的依赖
    > - 调用 acorn 解析经 loader 处理后的源文件生成抽象语法树 AST
    > - 遍历 AST，构建该模块所依赖的模块
5. **完成模块编译**：在经过第 4 步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系。
6. **输出资源**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会。
7. **输出完成**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

## Loader

Loader 用于对模块的源代码进行转换，在 import 或"加载"模块时预处理文件

### 1. 配置方式

Loader 的配置写在 `module.rules` 属性中：

-   `rules` 是一个数组的形式，可以配置很多个 Loader
-   每一个 Loader 对应一个对象的形式，对象属性 `test` 为匹配的规则，一般情况为正则表达式
-   属性 `use` 针对匹配到文件类型，调用对应的 Loader 进行处理

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    { loader: "style-loader" },
                    {
                        loader: "css-loader",
                        options: {
                            modules: true,
                        },
                    },
                    { loader: "sass-loader" },
                ],
            },
        ],
    },
};
```

### 2. 特性

Loader 支持链式调用，链中的每个 Loader 会处理之前已处理过的资源，最终变为 js 代码，顺序为相反的顺序执行。

Loader 的特性：

-   Loader 可以是同步的，也可以是异步的
-   Loader 运行在 Node.js 中，并且能够执行任何操作
-   除了常见的通过 package.json 的 main 来将一个 npm 模块导出为 Loader，还可以在 module.rules 中使用 Loader 字段直接引用一个模块
-   插件(plugin) 可以为 Loader 带来更多特性
-   Loader 能够产生额外的任意文件

### 3. 常见的 Loader

| Loader              | 说明                                                                    |
| ------------------- | ----------------------------------------------------------------------- |
| style-loader        | 将 css 添加到 DOM 的内联样式标签 `style` 里                             |
| css-loader          | 允许将 css 文件通过 `require` 的方式引入，并返回 css 代码               |
| less-loader         | 处理 less                                                               |
| sass-loader         | 处理 sass                                                               |
| postcss-loader      | 用 postcss 来处理 CSS                                                   |
| autoprefixer-loader | 处理 CSS3 属性前缀，已被弃用，建议直接使用 postcss                      |
| file-loader         | 分发文件到 output 目录并返回相对路径                                    |
| url-loader          | 和 file-loader 类似，但是当文件小于设定的 limit 时可以返回一个 Data Url |
| html-minify-loader  | 压缩 HTML                                                               |
| babel-loader        | 用 babel 来转换 ES6 文件到 ES5                                          |

## Plugin

Plugin 赋予其各种灵活的功能，例如打包优化、资源管理、环境变量注入等，它们会运行在 Webpack 的不同阶段（钩子 / 生命周期），贯穿了 Webpack 整个编译周期。

Plugin 目的在于解决 Loader 无法实现的其他事。

### 1. 配置方式

通过配置文件导出对象中 `plugins` 属性传入 new 实例对象。

```js
const HtmlWebpackPlugin = require("html-Webpack-plugin");
const Webpack = require("Webpack"); // 访问内置的插件
module.exports = {
    plugins: [
        new Webpack.ProgressPlugin(),
        new HtmlWebpackPlugin({ template: "./src/index.html" }),
    ],
};
```

### 2. 特性

其本质是一个具有 `apply` 方法 javascript 对象

`apply` 方法会被 Webpack compiler 调用，并且在整个编译生命周期都可以访问 compiler 对象

```js
const pluginName = "ConsoleLogOnBuildWebpackPlugin";

class ConsoleLogOnBuildWebpackPlugin {
    apply(compiler) {
        compiler.hooks.run.tap(pluginName, compilation => {
            console.log("Webpack 构建过程开始！");
        });
    }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```

compiler hook 的 `tap` 方法的第一个参数，应是驼峰式命名的插件名称

关于整个编译生命周期钩子，有如下：

> -   `entry-option`：初始化 option
> -   `run`
> -   `compile`：真正开始的编译，在创建 compilation 对象之前
> -   `compilation`：生成好了 compilation 对象
> -   `make`： 从 entry 开始递归分析依赖，准备对每个模块进行 build
> -   `after-compile`：编译 build 过程结束
> -   `emit`：在将内存中 assets 内容写到磁盘文件夹之前
> -   `after-emit`：在将内存中 assets 内容写到磁盘文件夹之后
> -   `done`：完成所有的编译过程
> -   `failed`：编译失败的时候

### 3. 常见的 Plugin

| Plugin                        | 说明                                                       |
| ----------------------------- | ---------------------------------------------------------- |
| AggressiveSplittingPlugin     | 将原来的 chunk 分成更小的 chunk                            |
| BabelMinifyWebpackPlugin      | 使用 babel-minify 进行压缩                                 |
| BannerPlugin                  | 在每个生成的 chunk 顶部添加 banner                         |
| CommonsChunkPlugin            | 提取 chunks 之间共享的通用模块                             |
| CompressionWebpackPlugin      | 预先准备的资源压缩版本，使用 Content-Encoding 提供访问服务 |
| ContextReplacementPlugin      | 重写 require 表达式的推断上下文                            |
| CopyWebpackPlugin             | 将单个文件或整个目录复制到构建目录                         |
| DefinePlugin                  | 允许在编译时(compile time) 配置的全局常量                  |
| Dllplugin                     | 为了极大减小构建时间，进行分离打包                         |
| EnvironmentPlugin             | DefinePlugin 中的 process.env 键的简写方式                 |
| ExtractTextWebpackPlugin      | 从 bundel 中提取文本(CSS) 到单独的文件                     |
| HotModuleReplacementPlugin    | 启用模块热替换 (Enable Hot Module Replacement - HMR)       |
| HtmlWebpackPlugin             | 简单创建 HTML 文件，用于服务器访问                         |
| I18nWebpackPlugin             | 为 bundle 增加国际化支持                                   |
| IgnorePlugin                  | 从 bundle 中排除某些模块                                   |
| LimitChunkCountPlugin         | 设置 chunk 的最小/最大限制，以微调和控制 chunk             |
| LoaderOptionsPlugin           | 用于从 Webpack1 迁移到 Webpack2                            |
| MinChunkSizePlugin            | 确保 chunk 大小超过指定限制                                |
| NoEmitOnErrorsPlugin          | 在输出阶段时，遇到编译错误跳过                             |
| NormalModuleReplacementPlugin | 替换与正则表达式匹配的资源                                 |

## Loader 和 Plugin 的区别

-   Loader 运行在打包文件之前
-   Plugin 在整个编译周期都起作用

在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

对于 Loader，实质是一个转换器，比如将 A.scss 或 A.less 转变为 B.css，单纯的文件转换过程。

## Webpack 热更新

### 1. 实现原理

<img src="../.images/image-20210526112504058.png" alt="image-20210526112504058" style="width: 600px;" />

-   Webpack Compile：将 JS 源代码编译成 bundle.js
-   HMR Server：用来将热更新的文件输出给 HMR Runtime
-   Bundle Server：静态资源文件服务器，提供文件访问路径
-   HMR Runtime：socket 服务器，会被注入到浏览器，更新文件的变化
-   bundle.js：构建输出的文件
-   在 HMR Runtime 和 HMR Server 之间建立 Websocket，即 4⃣️ ，用于实时更新文件变化

#### 启动阶段

1 -> 2 -> A -> B
在编写未经过 Webpack 打包的源代码后，Webpack Compile 将源代码和 HMR Runtime 一起编译成 bundle 文件，传输给 Bundle Server 静态资源服务器。

#### 更新阶段

1 -> 2 -> 3 -> 4

当某一个文件或者模块发生变化时，Webpack 监听到文件变化对文件重新编译打包，编译生成唯一的 hash 值（下图 `h` 属性），这个 hash 值用来作为下一次热更新的标识。

根据变化的内容生成两个补丁文件：`manifest`（包含了 hash 和 chundId，用来说明变化的内容）和 `chunk.js` 模块。

由于 socket 服务器在 HMR Runtime 和 HMR Server 之间建立 Websocket 链接，当文件发生改动的时候，服务端会向浏览器推送一条消息，消息包含文件改动后生成的 hash 值，作为下一次热更新的标识。
<img src="../.images/image-20210526145412513.png" alt="image-20210526145412513" style="width:600px;" />

在浏览器接受到这条消息之前，浏览器已经在上一次 socket 消息中已经记住了此时的 hash 标识，这时候会去服务端获取变化内容的 manifest 文件。

mainfest 文件包含重新 build 生成的 hash 值，以及变化的模块（上图 `c` 属性）。

浏览器根据 manifest 文件获取模块变化的内容，从而触发 render 流程，实现局部模块更新。

### 2. 总结

-   通过 `Webpack-dev-server` 创建两个服务器：提供静态资源的服务（express）和 Socket 服务
-   express server 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
-   socket server 是一个 Websocket 的长连接，双方可以通信
-   当 socket server 监听到对应的模块发生变化时，会生成两个文件.json（manifest 文件）和.js 文件（update chunk）
-   通过长连接，socket server 可以直接将这两个文件主动发送给客户端（浏览器）
-   浏览器拿到两个新的文件后，通过 HMR runtime 机制，加载这两个文件，并且针对修改的模块进行更新

## Webpack Proxy

Webpack Proxy，即 Webpack 提供的代理服务，基本行为就是接收客户端发送的请求后转发给其他服务器，其目的是为了便于开发者在开发模式下解决跨域问题（浏览器安全策略限制）。

想要实现代理首先需要一个中间服务器，Webpack 中提供服务器的工具为 Webpack-dev-server。

### 1. Webpack-dev-server

Webpack-dev-server 是 Webpack 官方推出的一款开发工具，将自动编译和自动刷新浏览器等一系列对开发友好的功能全部集成在了一起，目的是为了提高开发者日常的开发效率，**「只适用在开发阶段」**

配置方面，在 Webpack 配置对象属性中通过 devServer 属性提供，如下：

```js
// ./Webpack.config.js
const path = require("path");

module.exports = {
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        compress: true,
        port: 3000,
        proxy: {
            "/api": {
                target: "https://api.xxx.com",
            },
        },
    },
};
```

> 属性：
>
> -   target：表示的是代理到的目标地址
> -   pathRewrite：默认情况下 `/api` 也会被写入到 URL 中，如果希望删除，可以使用 `pathRewrite`
> -   secure：默认情况下不接收转发到 https 的服务器上，如果希望支持，可以设置为 `false`
> -   changeOrigin：它表示是否更新代理后请求的 `headers` 中 `host` 地址

### 2. 工作原理

proxy 工作原理实质上是利用 `http-proxy-middleware` 这个 http 代理中间件，实现请求转发给其他服务器。

### 3. 跨域

在开发阶段， Webpack-dev-server 会启动一个本地开发服务器，所以我们的应用在开发阶段是独立运行在 localhost 的一个端口上，而后端服务又是运行在另外一个地址上，所以在开发阶段中，由于浏览器同源策略的原因，当本地访问后端就会出现跨域请求的问题。

通过设置 Webpack proxy 实现代理请求后，相当于浏览器与服务端中添加一个代理者，当本地发送请求的时候，代理服务器响应该请求，并将请求转发到目标服务器，目标服务器响应数据后再将数据返回给代理服务器，最终再由代理服务器将数据响应给本地。

**注意：「服务器与服务器之间请求数据并不会存在跨域行为，跨域行为是浏览器安全策略限制」**

## 前端性能优化

### 1. JS 代码压缩

`terser` 是一个 JavaScript 的解释、绞肉机、压缩机的工具集，可以帮助我们压缩、丑化我们的代码，让 bundle 更小

在 `production` 模式下，Webpack 默认使用 TerserPlugin 来处理代码。自定义配置方法如下：

```js
const TerserPlugin = require("terser-Webpack-plugin");
module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({
                parallel: true, // 电脑cpu核数-1
            }),
        ],
    },
};
```

> 属性：
>
> -   extractComments：默认值为 `true`，表示会将注释抽取到一个单独的文件中，开发阶段，我们可设置为 `false` ，不保留注释
> -   parallel：使用多进程并发运行提高构建的速度，默认值是 `true`，并发运行的默认数量：`os.cpus().length - 1`
> -   terserOptions：设置我们的 terser 相关的配置：
>
>     compress：设置压缩相关的选项
>     mangle：设置丑化相关的选项，可以直接设置为 `true`
>     toplevel：底层变量是否进行转换
>     keep_classnames：保留类的名称
>     keep_fnames：保留函数的名称

### 2. CSS 代码压缩

使用 `css-minimizer-Webpack-plugin`

```shell
npm install css-minimizer-Webpack-plugin -D
```

```js
const CssMinimizerPlugin = require("css-minimizer-Webpack-plugin");
module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            new CssMinimizerPlugin({
                parallel: true,
            }),
        ],
    },
};
```

### 3. HTML 文件代码压缩

使用 `HtmlWebpackPlugin` 插件来生成 HTML 的模板时候，通过配置属性 `minify` 进行 html 优化。

```js
module.exports = {
    plugin: [
        new HtmlWebpackPlugin({
            minify: {
                minifyCSS: false, // 是否压缩css
                collapseWhitespace: false, // 是否折叠空格
                removeComments: true, // 是否移除注释
            },
        }),
    ],
};
```

设置了 `minify`，实际会使用另一个插件 `html-minifier-terser`

### 4. 文件大小压缩

对文件的大小进行压缩，减少 http 传输过程中宽带的损耗，使用 `compression-Webpack-plugin`。

```js
new ComepressionPlugin({
    test: /\.(css|js)$/, // 哪些文件需要压缩
    threshold: 500, // 设置文件多大开始压缩
    minRatio: 0.7, // 至少压缩的比例
    algorithm: "gzip", // 采用的压缩算法
});
```

### 5. 图片压缩

```js
module: {
    rules: [
        {
            test: /\.(png|jpg|gif)$/,
            use: [
                {
                    loader: "file-loader",
                    options: {
                        name: "[name]_[hash].[ext]",
                        outputPath: "images/",
                    },
                },
                {
                    loader: "image-Webpack-loader",
                    options: {
                        // 压缩 jpeg 的配置
                        mozjpeg: {
                            progressive: true,
                            quality: 65,
                        },
                        // 使用 imagemin**-optipng 压缩 png，enable: false 为关闭
                        optipng: {
                            enabled: false,
                        },
                        // 使用 imagemin-pngquant 压缩 png
                        pngquant: {
                            quality: "65-90",
                            speed: 4,
                        },
                        // 压缩 gif 的配置
                        gifsicle: {
                            interlaced: false,
                        },
                        // 开启 webp，会把 jpg 和 png 图片压缩为 webp 格式
                        webp: {
                            quality: 75,
                        },
                    },
                },
            ],
        },
    ];
}
```

### 6. Tree Shaking

Tree Shaking 是一个术语，在计算机中表示消除死代码，依赖于 ES Module 的静态语法分析（不执行任何的代码，可以明确知道模块的依赖关系）。

在 Webpack 实现 Tree shaking 有两种不同的方案：

-   usedExports：通过标记某些函数是否被使用，之后通过 Terser 来进行优化的
-   sideEffects：跳过整个模块/文件，直接查看该文件是否有副作用
    两种不同的配置方案， 有不同的效果

#### usedExports

```js
module.exports = {
    optimization: {
        usedExports: true,
    },
};
```

使用之后，没被用上的代码在 Webpack 打包中会加入 `/* unused harmony export mul */` 注释，用来告知 Terser 在优化时，可以删除掉这段代码。

#### sideEffects

sideEffects 用于告知 Webpack Compiler 哪些模块时有副作用，配置方法是在 `package.json` 中设置 `sideEffects` 属性。

如果 sideEffects 设置为 `false`，就是告知 Webpack 可以安全的删除未用到的 exports。如果有些文件需要保留，可以设置为数组的形式。

```json
{
    "sideEffects": [
        "./src/util/format.js",
        "*.css" // 所有的css文件
    ]
}
```

#### css tree shaking

css 进行 tree shaking 优化可以安装 `PurgeCss` 插件。

```js
const PurgeCssPlugin = require("purgecss-Webpack-plugin");
module.exports = {
    plugins: [
        new PurgeCssPlugin({
            path: glob.sync(`${path.resolve("./src")}/**/*`),
            nodir: true, // src里面的所有文件
            satelist: function () {
                return {
                    standard: ["html"],
                };
            },
        }),
    ],
};
```

> paths：表示要检测哪些目录下的内容需要被分析，配合使用 glob。
> 默认情况下，PurgeCss 会将我们的 html 标签的样式移除掉，如果我们希望保留，可以添加一个 `safelist` 的属性。

### 7. 代码分割 (Code Splitting)

将代码分离到不同的 bundle 中，之后我们可以按需加载，或者并行加载这些文件。

默认情况下，所有的 JavaScript 代码（业务代码、第三方依赖、暂时没有用到的模块）在首页全部都加载，就会影响首页的加载速度。

代码分离可以分出出更小的 bundle，以及控制资源加载优先级，提供代码的加载性能。

这里通过 `splitChunksPlugin` 来实现，该插件 Webpack 已经默认安装和集成，只需要配置即可。默认配置中，chunks 仅仅针对于异步（async）请求，我们可以设置为 `initial` 或者 `all`。

```js
module.exports = {
    optimization: {
        splitChunks: {
        minSize: 30000,
        maxInitialRequests: 5,
        cacheGroups: {
          lib: {
            name: 'vendors',
            priority: 10,
            test: /[\\/]node_modules[\\/]/,
            chunks: 'initial',
          },
          react: {
            name: 'react',
            priority: 20,
            test: /[\\/]node_modules[\\/]react[\\/]/,
            chunks: 'all',
          },
          antd: {
            name: 'antd',
            priority: 20,
            test: /[\\/]node_modules[\\/]antd[\\/]/,
            chunks: 'all',
          },
          moment: {
            name: 'moment',
            priority: 20,
            test: /[\\/]node_modules[\\/]moment[\\/]/,
          },
          default: {
            minChunks: 2,
            priority: -20,
            reuseExistingChunk: true,
          },
        },
    },
};
```

> minSize：拆分包的大小，至少为 minSize，如何包的大小不超过 minSize，这个包不会拆分
> maxSize：将大于 maxSize 的包，拆分为不小于 minSize 的包

#### chunks

1. `async` 表示只从异步加载得模块（动态加载 `import()`）里面进行拆分
2. `initial` 表示只从入口模块进行拆分
3. `all` 表示以上两者都包括

#### cacheGroups

1. `name`：chunk 的文件名
2. `test`：过滤 modules，默认为所有的 modules，可匹配模块路径或 chunk 名字，当匹配到某个 chunk 的名字时，这个 chunk 里面引入的所有 module 都会选中
3. `priority`：权重，数字越大表示优先级越高。一个 module 可能会满足多个 cacheGroups 的正则匹配，到底将哪个缓存组应用于这个 module，取决于优先级
4. `reuseExistingChunk`：是否使用已有的 chunk，true 则表示如果当前的 chunk 包含的模块已经被抽取出去了，那么将不会重新生成新的，即几个 chunk 复用被拆分出去的一个 module

### 8. 内联 chunk

可以通过 `InlineChunkHtmlPlugin` 插件将一些 chunk 的模块内联到 html，如 runtime 的代码（对模块进行解析、加载、模块信息相关的代码），代码量并不大，但是必须加载的。

```js
const InlineChunkHtmlPlugin = require("react-dev-utils/InlineChunkHtmlPlugin");
const HtmlWebpackPlugin = require("html-Webpack-plugin");
module.exports = {
    plugin: [new InlineChunkHtmlPlugin(HtmlWebpackPlugin, [/runtime.+\.js/])],
};
```

### 9. 总结

关于 Webpack 对前端性能的优化，可以通过文件体积大小入手，其次还可通过分包的形式、减少 http 请求次数等方式，实现对前端性能的优化。

## 提高 Webpack 的构建速度

### 1. 优化 Loader 配置

通过配置 include、exclude、test 属性来匹配文件，接触 include、exclude 规定哪些匹配应用 loader。

```js
// 配置 babel-loader
module.exports = {
    module: {
        rules: [
            {
                // 如果项目源码中只有 js 文件就不要写成 /\.jsx?$/，提升正则表达式性能
                test: /\.js$/,
                // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
                use: ["babel-loader?cacheDirectory"],
                // 只对项目根目录下的 src 目录中的文件采用 babel-loader
                include: path.resolve(__dirname, "src"),
            },
        ],
    },
};
```

### 2. 合理使用 resolve.extensions

通过 `resolve.extensions` 是解析到文件时自动添加扩展名，默认情况如下：

```js
module.exports = {
    extensions: [".warm", ".mjs", ".js", ".json"],
};
```

引入文件的时候，若没有文件后缀名，则会根据数组内的值依次查找。配置的时候，不要随便把所有后缀都写在里面，这会调用多次文件的查找，减慢打包速度。

### 3. 优化 resolve.modules

`resolve.modules` 用于配置 Webpack 去哪些目录下寻找第三方模块。默认值为 `['node_modules']` ，所以默认会从 node_modules 中查找文件 当安装的第三方模块都放在项目根目录下的 `./node_modules` 目录下时，所以可以指明存放第三方模块的绝对路径，以减少寻找，配置如下：

```js
module.exports = {
    resolve: {
        // 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
        // 其中 __dirname 表示当前工作目录，也就是项目根目录
        modules: [path.resolve(__dirname, "node_modules")],
    },
};
```

### 4. 优化 resolve.alias

alias 给一些常用的路径起一个别名，特别当我们的项目目录结构比较深的时候，一个文件的路径可能是 `../../../` 的形式。

通过配置 alias 以减少查找过程：

```js
module.exports = {
    resolve: {
        alias: {
            "@": path.resolve(__dirname, "./src"),
        },
    },
};
```

### 5. 使用 DLLPlugin 插件

DLL 全称是 动态链接库，是为软件在 Windows 中实现共享函数库的一种实现方式，而 Webpack 也内置了 DLL 的功能，为的就是可以共享，不经常改变的代码，抽成一个共享的库。这个库在之后的编译过程中，会被引入到其他项目的代码中

使用步骤分成两部分：

-   打包一个 DLL 库
-   引入 DLL 库

#### 打包一个 DLL 库

Webpack 内置了一个 DllPlugin 可以帮助我们打包一个 DLL 的库文件

```js
module.exports = {
    plugins: [
        new Webpack.DllPlugin({
            name: "dll_[name]",
            path: path.resolve(__dirname, "./dll/[name].mainfest.json"),
        }),
    ],
};
```

#### 引入 DLL 库

使用 Webpack 自带的 `DllReferencePlugin` 插件对 `mainfest.json` 映射文件进行分析，获取要使用的 DLL 库，然后再通过 `AddAssetHtmlPlugin` 插件，将打包的 DLL 库引入到 Html 模块中。

```js
module.exports = {
    plugins: [
        new Webpack.DllReferencePlugin({
            context: path.resolve(__dirname, "./dll/dll_react.js"),
            mainfest: path.resolve(__dirname, "./dll/react.mainfest.json"),
        }),
        new AddAssetHtmlPlugin({
            outputPath: "./auto",
            filepath: path.resolve(__dirname, "./dll/dll_react.js"),
        }),
    ],
};
```

### 6. 使用 cache-loader

在一些性能开销较大的 loader 之前添加 `cache-loader`，以将结果缓存到磁盘里，显著提升二次构建速度。

保存和读取这些缓存文件会有一些时间开销，所以应只对性能开销较大的 loader 使用此 loader。

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.ext$/,
                use: ["cache-loader", ...loaders],
                include: path.resolve("src"),
            },
        ],
    },
};
```

### 7. terser 启动多线程

使用多进程并行运行来提高构建速度。

TerserWebpackPlugin 插件使用 terser 来压缩 JavaScript。

```js
module.exports = {
    optimization: {
        minimizer: [
            new TerserPlugin({
                parallel: true,
            }),
        ],
    },
};
```

### 8. 合理使用 sourceMap

打包生成 sourceMap 的时候，如果信息越详细，打包速度就会越慢。对应属性取值如下所示：

| devtool                        | 构建速度 | 重新构建速度 | 生产环境 | 品质(quality)         |
| ------------------------------ | -------- | ------------ | -------- | --------------------- |
| (none)                         | +++      | +++          | yes      | 打包后的代码          |
| eval                           | +++      | +++          | no       | 生成后的代码          |
| cheap-eval-source-map          | +        | ++           | no       | 转换过的代码 (仅限行) |
| cheap-module-eval-source-map   | o        | ++           | no       | 原始源代码 (仅限行)   |
| eval-source-map                | --       | +            | no       | 原始源代码            |
| cheap-source-map               | +        | o            | yes      | 转换过的代码 (仅限行) |
| cheap-module-source-map        | o        | -            | yes      | 原始源代码 (仅限行)   |
| inline-cheap-source-map        | +        | o            | no       | 转换过的代码 (仅限行) |
| inline-cheap-module-source-map | o        | -            | no       | 原始源代码 (仅限行)   |
| source-map                     | --       | --           | yes      | 原始源代码            |
| inline-source-map              | --       | --           | no       | 原始源代码            |
| hidden-source-map              | --       | --           | yes      | 原始源代码            |
| nosources-source-map           | --       | --           | yes      | 无源代码内容          |

> +++ 非常快速 &emsp; ++ 快速 &emsp; + 比较快 &emsp; o 中等 &emsp; - 比较慢 &emsp; -- 慢

### 总结

优化 Webpack 构建的方式有很多，主要可以从优化搜索时间、缩小文件搜索范围、减少不必要的编译等方面入手。

## Webpack 与 grunt、gulp 有什么不同

<img src="../.images/image-20210526232017808.png" alt="image-20210526232017808" style="width: 600px;" />

从上图中可以看出来三者都不在一个频道，自然没有可比性。

grunt 和 gulp 在早期比较流行，属于前端工具类，主要优化前端工作流程，比如自动刷新页面、combo、压缩 css、js、css 预编译等等。

现在 Webpack 相对来说比较主流，属于预编译模块的方案，不需要在浏览器中加载解释器。另外，在本地直接写 JS，不管是 AMD / CMD / ES6 风格的模块化，Webpack 都能认识，并且编译成浏览器认识的 js。

### 使用上的区别

grunt 和 gulp 是基于任务和流（Task、Stream）的。类似 jQuery，找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个 web 的构建流程。

```js
gulp.task("dev", function () {
    // 这个任务的名称是 dev
    gulp.src("src/js/*.js") //找到 `src/js/` 目录下的所有js
        .pipe(concat("all.js")) //对其进行合并并且命名为 `all.js`
        .pipe(uglify()) //压缩
        .pipe(rename("all.min.js")) //重命名
        .pipe(gulp.dest("dist/js/")); //输出压缩后的js
});

//在目录下执行 `gulp dev` 即可执行上述操作
//grunt 与 gulp 类似，只是稍微繁索一些，现在基本上用不上了
```

Webpack 是基于入口的。会自动地递归解析入口所需要加载的所有资源文件，然后用不同的 Loader 来处理不同的文件，用 Plugin 来扩展 Webpack 功能。

```js
// Webpack 具体配置这里不做讲解，形式都大同小异

module.exports = {
    entry: "./entry.js", //入口文件
    output: {
        filename: "./bundle.js", //输出文件
    },
    module: {
        loaders: [...loaders],
    },
};
```

从构建思路来说，gulp 和 grunt 需要开发者将整个前端构建过程拆分成多个 Task，并合理控制所有 Task 的调用关系
Webpack 需要开发者找到入口，并需要清楚对于不同的资源应该使用什么 Loader 做何种解析和加工。

### 总结

-   gulp、grunt 是 web 构建工具
-   Webpack 是模块化方案
-   gulp、grunt 是基于任务和流
-   Webpack 基于入口文件
