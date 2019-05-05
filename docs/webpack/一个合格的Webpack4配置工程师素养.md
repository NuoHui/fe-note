# Webpack配置工程师

总结Webpack4常见的配置, 含DEMO, 一步步肥肠详细，略长, 后续结束时候我们给出源码文件。

## 准备开发环境

```
- 安装node
- 安装webpack
- npm init 初始化项目
```

## 目录结构


![目录](https://user-gold-cdn.xitu.io/2019/1/26/16889053d7273593?w=610&h=432&f=png&s=44909)

## 写跑一个小demo

```
// src/index.js

import _ from 'lodash'

function create_div_element () {
    const div_element = document.createElement('div')
    div_element.innerHTML = _.join(['kobe', 'cpul'], ' ')
    return div_element
}

const div_ele = create_div_element()
document.body.appendChild(div_ele)

```

```
// dist/index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webpack4</title>
</head>
<body>
    <script src="./bound.js"></script>
</body>
</html>

```

```
// webpack.config.js

const path = require('path')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    }
}

```

然后通过[npx](https://github.com/zkat/npx#readme)执行webpack进行打包。

或者配成一个script命令也可以。

```
"scripts": {
    "build": "npx webpack -c webpack.config.js"
 }
```

```
npx webpack
```

在浏览器打开index.html就会发现代码执行成功了。


## webpack处理CSS

假设我们现在需要在index.js引入css文件。

```
// index.js

import './style/reset.css'
```


我们需要使用专门的loader来解析css, 并把css注入到html文件
```
 npm i -D css-loader style-loader
```

修改webpack配置文件
```
const path = require('path')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.css/,
                use: ['style-loader', 'css-loader'] // use的顺序从右往左
            }
        ]
    }
}
```
这个时候你在npx webpack, 打包后执行index.html你会发现css已经注入成功了。

## webpack处理sass文件

现在前端项目都是使用一些css预处理器来帮助更好的使用CSS，如Sass等。

假设我们现在index.js中需要引入一个base.scss文件。
那么webpack改如何处理sass/scss文件呢?

```
npm install sass-loader node-sass -D
```

```
// src/style/base.scss

$bd-bg: pink;
body {
    background: $bd-bg;
}
```

```
// index.js

import './style/base.scss'
```

更过配置文件处理scss
```
const path = require('path')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.(sc|sa|c)ss$/,
                use: ['style-loader', 'css-loader', 'sass-loader'] // use的顺序从右往左
            }
        ]
    }
}

```

## webpack为sass添加source map

配置source map是为了当出现错误时候方便我们进行定位调试， 当然我们在生产环境不需要启动这个。

像我们上面例子中, 你会发现打包后我们看不出scss来自哪个文件。


![noe-source-map](https://user-gold-cdn.xitu.io/2019/1/26/168899427382104f?w=2548&h=624&f=png&s=224107)

修改webpack配置文件。

```
const path = require('path')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.(sc|sa|c)ss$/,
                use: [
                    {
                        loader: 'style-loader'
                    },
                    {
                        loader: 'css-loader',
                        options: {
                            sourceMap: true
                        }
                    },
                    {
                        loader: 'sass-loader',
                        options: {
                            sourceMap: true
                        }
                    }
                ]
            }
        ]
    }
}

```

打包后在浏览器打开index.html.


![sourcemap](https://user-gold-cdn.xitu.io/2019/1/26/16889987ec88f4b3?w=5116&h=760&f=jpeg&s=311054)

## webpack为css添加CSS3前缀

[PostCSS](https://www.postcss.com.cn/)是一个用 JavaScript 工具和插件转换 CSS 代码的工具, 功能强大, 我们最常用的就是利用PostCSS帮我们Autoprefixer 自动获取浏览器的流行度和能够支持的属性，并根据这些数据帮你自动为 CSS 规则添加前缀。


```
npm i -D postcss-loader autoprefixer postcss-import

// postcss-import: 在使用@import css文件时候让webpack可以监听并编译
// postcss-nextcss: 支持css4
```

修改配置文件

```
rules: [
    {
        test: /\.(sc|sa|c)ss$/,
        use: [
            {
                loader: 'style-loader'
            },
            {
                loader: 'css-loader',
                options: {
                    sourceMap: true
                }
            },
            {
                loader: 'postcss-loader',
                options: {
                    ident: 'postcss',
                    sourceMap: true,
                    plugins: loader => [
                        // 可以配置多个插件
                        require('autoprefixer')({
                            browsers: [' > 0.15% in CN ']
                        })
                    ]
                }
            },
            {
                loader: 'sass-loader',
                options: {
                    sourceMap: true
                }
            }
        ]
    }
]
```


![css3前缀](https://user-gold-cdn.xitu.io/2019/1/26/16889ac8a2366de2?w=2186&h=494&f=png&s=157203)

## 抽离样式表为单独的css文件并打版本号

抽离css前提是我们只在生产环境这么做, 因此你的配置文件的mode: production。

另外抽离了css就不能在使用style-loader注入到html文件。

```
npm i -D mini-css-extract-plugin
```

配置一个script命名

```
"scripts": {
    "dist": "cross-env NODE_ENV=production npx webpack --progress --config webpack.prod.config.js"
  },
```

添加一个webpack.prod.config.js.当然正式项目我们是会拆分配置文件, 然后通过merge处理。

```
- webpack.base.config.js
- webpack.dev.config.js
- webpack.prod.config.js
- webpack.vue.config.js
```

这里demo就没有这么做, 所以代码有些冗余。

```
// webpack.prod.config.js

const path = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const devMode = process.env.NODE_ENV !== 'production'

module.exports = {
    entry: './src/index.js',
    mode: 'production',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.(sc|sa|c)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    {
                        loader: 'css-loader'
                    },
                    {
                        loader: 'postcss-loader',
                        options: {
                            ident: 'postcss',
                            plugins: (loader) => [
                                require('autoprefixer')({
                                    browsers: [
                                        'last 10 Chrome versions',
                                        'last 5 Firefox versions',
                                        'Safari >= 6',
                                        'ie > 8'
                                    ]
                                })
                            ]
                        }
                    },
                    {
                        loader: 'sass-loader'
                    }
                ]
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: devMode ? '[name].css' : '[name].[hash:5].css', // 设置输出的文件名
            chunkFilename: devMode ? '[id].css': '[id].[hash:5].css'
        })
    ]
}

```

打包后你会发现


![main.css](https://user-gold-cdn.xitu.io/2019/1/26/16889d416b922ff4?w=582&h=482&f=png&s=53322)

这个时候我们如果去使用只能在index.html去引用它了, 很明显这是不方便的, 因为我们css文件肯定很庞大, 后面会解决这个问题, 这里就略过。


## webpack压缩JS和CSS

压缩的作用自然是为了减小包的体积了, 提升加载效率, 因此压缩都是配置在生产环境。

### 压缩css

Webpack后面版本应该会内置CSS压缩, 目前先手工配置。

```
npm i -D optimize-css-assets-webpack-plugin
```

更改配置文件:

```
const OptimizeCSSAssertsPlugin = require('optimize-css-assets-webpack-plugin')

optimization: {
    minimizer: [
        // 压缩CSS
        new OptimizeCSSAssertsPlugin({})
    ]
}
```


![压缩css](https://user-gold-cdn.xitu.io/2019/1/26/16889e0ed14dec47?w=2282&h=730&f=png&s=151485)

### JS压缩


```
npm i -D uglifyjs-webpack-plugin
```

修改配置文件

```
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

optimization: {
    minimizer: [
        // 压缩JS
        new UglifyJsPlugin({
            // 有很多可以配置
            cache: true,
            parallel: true,
            sourceMap: true,
            uglifyOptions: {
                 // 在UglifyJs删除没有用到的代码时不输出警告
                warnings: false,
                output: {
                    // 删除所有的注释
                    comments: false,
                    // 最紧凑的输出
                    beautify: false
                },
                compress: {
                    // 删除所有的 `console` 语句
                    // 还可以兼容ie浏览器
                    drop_console: true,
                    // 内嵌定义了但是只用到一次的变量
                    collapse_vars: true,
                    // 提取出出现多次但是没有定义成变量去引用的静态值
                    reduce_vars: true,
                }
            }
        })
    ]
}
```



这个时候去打包我发现一个错误, [ERROR in js/background.js from UglifyJs Unexpected token: keyword (const)](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/issues/389)。

Uglify-js不支持es6语法，请使用terser插件, 于是我们更改使用terser插件试试, 其实你继续用uglifyjs-webpack-plugin也可以, 只需要配合babel先转下。

```
npm install terser-webpack-plugin -D
```


更多使用见官网[terser-webpack-plugin](https://www.npmjs.com/package/terser-webpack-plugin)。

```

optimization: {
    minimizer: [
        // 压缩JS
        new TerserPlugin({
            cache: true,
            parallel: true,
            sourceMap: true,
            // 等等详细配置见官网
        }),

    ]
}
```


![压缩js](https://user-gold-cdn.xitu.io/2019/1/26/16889f7537a332ee?w=2188&h=734&f=png&s=148909)

## webpack处理带哈希值的文件名引入问题

我们给打包的文件打上hash是为了解决缓存更新问题，常见需要打上hash的地方有。

```
output: {
    filename: 'bound.[hash:5].js',
    path: path.resolve(__dirname, 'dist')
}
```

```
// 提取CSS
new MiniCssExtractPlugin({
    filename: devMode ? '[name].css' : '[name].[hash:5].css', // 设置输出的文件名
    chunkFilename: devMode ? '[id].css': '[id].[hash:5].css'
})
```

但是打上hash我们怎么引入是一个问题。

html-webpack-plugin插件可以把js/css注入到一个模板文件, 所以不需要再手动更改引用。

```
npm i -D html-webpack-plugin
```

更改配置文件

```
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
    // 打包模板
    new HtmlWebpackPlugin({
        inject: true,
        hash: true,
        cache: true,
        chunksSortMode: 'none',
        title: 'Webapck4-demo', // 可以由外面传入
        filename: 'index.html', // 默认index.html
        template: path.resolve(__dirname, 'index.html'),
        minify: {
            collapseWhitespace: true,
            removeComments: true,
            removeRedundantAttributes: true,
            removeScriptTypeAttributes: true,
            removeStyleLinkTypeAttributes: true
        }
    })
],
```

设置一个模板文件。

```
// index.html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta name="google" value="notranslate">
    <meta http-equiv="Cache-Control" content="no-siteapp">
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
    <meta name="format-detection" content="telephone=no">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>

<body>
    <div id="app"></div>
</body>

</html>
```
打包后的文件:

![打包后文件](https://user-gold-cdn.xitu.io/2019/1/26/1688a3ed90238b7d?w=2172&h=1608&f=jpeg&s=198912)

```
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta name="google" value="notranslate">
    <meta http-equiv="Cache-Control" content="no-siteapp">
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
    <meta name="format-detection" content="telephone=no">
    <title></title>
    <link href="main.f37fa.css?f37fab3edd3ae8ecda6a" rel="stylesheet">
</head>

<body>
    <div id="app"></div>
    <script src="bound.f37fa.js?f37fab3edd3ae8ecda6a"></script>
</body>

</html>
```

## webpack清理打包后的dist目录

我们会发现每次打包后dist文件夹都会不断增加文件, 显然这个方面我们需要处理, 但是某些情况下我们不需要去清理, 比如坑爹的微信公众号缓存问题。

```
npm i -D clean-webpack-plugin
```

修改配置文件

```
const CleanWebpackplugin = require('clean-webpack-plugin')

plugins: [
    // 清理dist目录
    new CleanWebpackplugin(['dist'])
]
```

## webpack处理图片以及优化

我们这里只是为了测试, 在index.html模板文件添加一个dom去使用图片。

```
// index.html
<div class="logo"></div>

// base.scss
.logo {
    background: url('../assets/logo.png') no-repeat;
    width: 100px;
    height: 100px;
    background-size: contain;
}
```

使用file-loader来处理文件的导入

```
npm i -D file-loader
```

修改配置文件

```
rules: [
    // 处理文件
    {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        use: [
            {
                loader: 'file-loader',
                options: {
                    // 具体配置见插件官网
                    limit: 1,
                    name: '[name]-[hash:5].[ext]',
                    outputPath: 'img/', // outputPath所设置的路径，是相对于 webpack 的输出目录。
                    // publicPath 选项则被许多webpack的插件用于在生产模式下更新内嵌到css、html文件内的 url , 如CDN地址
                },
            },
        ]
    },
]
```

下面继续对图片进行优化和压缩

```
npm i -D image-webpack-loader
```

修改配置文件

```
// 处理文件
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    use: [
        {
            loader: 'file-loader',
            options: {
                // 具体配置见插件官网
                limit: 10000,
                name: '[name]-[hash:5].[ext]',
                outputPath: 'img/', // outputPath所设置的路径，是相对于 webpack 的输出目录。
                // publicPath 选项则被许多webpack的插件用于在生产模式下更新内嵌到css、html文件内的 url , 如CDN地址
            },
        },
        {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
                quality: 65
              },
              // optipng.enabled: false will disable optipng
              optipng: {
                enabled: false,
              },
              pngquant: {
                quality: '65-90',
                speed: 4
              },
              gifsicle: {
                interlaced: false,
              },
              // the webp option will enable WEBP
              webp: {
                quality: 75
              }
            }
        }
    ]
},
```

压缩前图片大小181.46kb.

![未压缩](https://user-gold-cdn.xitu.io/2019/1/26/1688a6b1b5b5911c?w=2464&h=1352&f=png&s=379430)

压缩后29kb.

![压缩图片后](https://user-gold-cdn.xitu.io/2019/1/26/1688a6c9d26a2def?w=2456&h=1470&f=png&s=657561)


## webpack把图片转为base64以及字体处理

通过把一些小的图片转为base65(DataURl)可以减少http请求, 提升访问效率。

```
npm i -D url-loader
```

修改配置文件

```
// 处理文件
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    use: [
        {
            loader: 'url-loader',
            options: {
                // 具体配置见插件官网
                limit: 10000,
                name: '[name]-[hash:5].[ext]',
                outputPath: 'img/', // outputPath所设置的路径，是相对于 webpack 的输出目录。
                // publicPath 选项则被许多webpack的插件用于在生产模式下更新内嵌到css、html文件内的 url , 如CDN地址
            },
        },
        {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
                quality: 65
              },
              // optipng.enabled: false will disable optipng
              optipng: {
                enabled: false,
              },
              pngquant: {
                quality: '65-90',
                speed: 4
              },
              gifsicle: {
                interlaced: false,
              },
              // the webp option will enable WEBP
              webp: {
                quality: 75
              }
            }
        }
    ]
},
```
这里测试的话我们需要准备一个小的图片即可，如上述配置所述只要小于10kb就会用base64替代。


![转base64](https://user-gold-cdn.xitu.io/2019/1/26/1688a7bc21b4b3a8?w=5096&h=1120&f=jpeg&s=476780).

字体处理的话配置如下：

```
{
    test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        // 文件大小小于limit参数，url-loader将会把文件转为DataUR
        limit: 10000,
        name: '[name]-[hash:5].[ext]',
        output: 'fonts/',
        // publicPath: '', 多用于CDN
    }
},
```



## webpack配置分层

之前有提过webpack根据不同的环境我们会加载不同的配置。我们只需要提取出三部分。

```
- base: 公共的部分
- dev: 开发环境部分
- prod: 生产环境部分
```

```
npm i -D webpack-merge
```

我们这里现在简单分层：正式项目最好创建一个config/webpack目录管理。


![webpack目录](https://user-gold-cdn.xitu.io/2019/1/26/1688a8b03626dd51?w=1156&h=968&f=jpeg&s=89511)

下面是源代码。

```
"scripts": {
    "dev": "cross-env NODE_ENV=development npx webpack --progress --config webpack.dev.config.js",
    "build": "cross-env NODE_ENV=production npx webpack --progress --config webpack.prod.config.js"
 },
```

```
// webapck.base.config.js

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackplugin = require('clean-webpack-plugin')

module.exports = {
    entry: './src/index.js',
    module: {
        rules: [
            // 处理字体
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                loader: 'url-loader',
                options: {
                    // 文件大小小于limit参数，url-loader将会把文件转为DataUR
                    limit: 10000,
                    name: '[name]-[hash:5].[ext]',
                    output: 'fonts/',
                    // publicPath: '', 多用于CDN
                }
            },
            // 处理文件
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: [
                    // 转base64
                    {
                        loader: 'url-loader',
                        options: {
                            // 具体配置见插件官网
                            limit: 10000,
                            name: '[name]-[hash:5].[ext]',
                            outputPath: 'img/', // outputPath所设置的路径，是相对于 webpack 的输出目录。
                            // publicPath 选项则被许多webpack的插件用于在生产模式下更新内嵌到css、html文件内的 url , 如CDN地址
                        },
                    },
                    {
                        loader: 'image-webpack-loader',
                        options: {
                          mozjpeg: {
                            progressive: true,
                            quality: 65
                          },
                          // optipng.enabled: false will disable optipng
                          optipng: {
                            enabled: false,
                          },
                          pngquant: {
                            quality: '65-90',
                            speed: 4
                          },
                          gifsicle: {
                            interlaced: false,
                          },
                          // the webp option will enable WEBP
                          webp: {
                            quality: 75
                          }
                        }
                    }
                ]
            }
        ]
    },
    plugins: [
        // 打包模板
        new HtmlWebpackPlugin({
            inject: true,
            hash: true,
            cache: true,
            chunksSortMode: 'none',
            title: 'Webapck4-demo', // 可以由外面传入
            filename: 'index.html', // 默认index.html
            template: path.resolve(__dirname, 'index.html'),
            minify: {
                collapseWhitespace: true,
                removeComments: true,
                removeRedundantAttributes: true,
                removeScriptTypeAttributes: true,
                removeStyleLinkTypeAttributes: true
            }
        }),
        // 清理dist目录
        new CleanWebpackplugin(['dist'])
    ]
}

```


```
// webpack.dev.config.js

const path = require('path')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.config.js')


module.exports = merge(baseWebpackConfig, {
    mode: 'development',
    output: {
        filename: 'bound.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            // 处理css/scss/sass
            {
                test: /\.(sc|sa|c)ss$/,
                use: [
                    {
                        loader: 'style-loader',
                    },
                    {
                        loader: 'css-loader',
                        options: {
                            sourceMap: true
                        }
                    },
                    {
                        loader: 'postcss-loader',
                        options: {
                            ident: 'postcss',
                            sourceMap: true,
                            plugins: (loader) => [
                                require('autoprefixer')({
                                    browsers: [
                                        'last 10 Chrome versions',
                                        'last 5 Firefox versions',
                                        'Safari >= 6',
                                        'ie > 8'
                                    ]
                                })
                            ]
                        }
                    },
                    {
                        loader: 'sass-loader',
                        options: {
                            sourceMap: true
                        }
                    }
                ]
            }
        ]
    }
})

```


```
// webapck.prod.config.js

const path = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCSSAssertsPlugin = require('optimize-css-assets-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.config.js')
const devMode = process.env.NODE_ENV !== 'production'

module.exports = merge(baseWebpackConfig, {
    mode: 'production',
    output: {
        filename: 'bound.[hash:5].js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            // 处理css/scss/sass
            {
                test: /\.(sc|sa|c)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    {
                        loader: 'css-loader'
                    },
                    {
                        loader: 'postcss-loader',
                        options: {
                            ident: 'postcss',
                            plugins: (loader) => [
                                require('autoprefixer')({
                                    browsers: [
                                        'last 10 Chrome versions',
                                        'last 5 Firefox versions',
                                        'Safari >= 6',
                                        'ie > 8'
                                    ]
                                })
                            ]
                        }
                    },
                    {
                        loader: 'sass-loader'
                    }
                ]
            }
        ]
    },
    plugins: [
        // 提取CSS
        new MiniCssExtractPlugin({
            filename: devMode ? '[name].css' : '[name].[hash:5].css', // 设置输出的文件名
            chunkFilename: devMode ? '[id].css': '[id].[hash:5].css'
        })
    ],
    optimization: {
        minimizer: [
            // 压缩JS
            new TerserPlugin({
                cache: true,
                parallel: true,
                sourceMap: true,
                // 等等详细配置见官网
            }),
            // 压缩CSS
            new OptimizeCSSAssertsPlugin({})
        ]
    }
})

```

## webpack配置js使用sourceMap

在webpack4使用inline-source-map选项就可以启动错误的堆栈跟踪, 只用于开发环境

```
devtool: 'inline-source-map'
```

## 监控文件变化自动编译


简单的方法就是启动watch模式: 如

```
"dev": "cross-env NODE_ENV=development npx webpack --progress --config webpack.dev.config.js --watch"
```

## webpack开启热更新和代理配置

很明显上面watch模式效率不高而且很不方便, 编译完还需要刷新页面, webpack可以开启热更新模式，大大加速开大效率。

```
npm i -D webpack-dev-server
```

修改script脚本。

```
"dev": "cross-env NODE_ENV=development npx webpack-dev-server --progress --config webpack.dev.config.js"
```

修改配置文件

```
const webpack = require('webpack')


plugins: [
    new webpack.NamedModulesPlugin(), // 更方便查看patch的依赖
    new webpack.HotModuleReplacementPlugin() // HMR
],
devServer: {
    clientLogLevel: 'warning', // 输出日志级别
    hot: true, // 启用热更新
    contentBase: path.resolve(__dirname, 'dist'), // 告诉服务器从哪里提供内容
    publicPath: '/', // 此路径下的打包文件可在浏览器下访问
    compress: true, // 启用gzip压缩
    // publicPath: './',
    disableHostCheck: true,
    host: '0.0.0.0',
    port: 9999,
    open: true, // 自动打开浏览器
    overlay: { // 出现错误或者警告时候是否覆盖页面线上错误信息
        warnings: true,
        errors: true
    },
    quiet: true,
    proxy: { // 设置代理
        '/dev': {
            target: 'http://dev.xxxx.com.cn',
            changeOrigin: true,
            pathRewrite: {
                '^/dev': ''
            }
            /**
             * 如果你的配置是
             * pathRewrite: {
                '^/dev': '/order/api'
                }
                即本地请求 /dev/getOrder   =>  实际上是  http://dev.xxxx.com.cn/order/api/getOrder
            */
        },
        '/test': {
            target: 'http://test.xxxx.com.cn',
            changeOrigin: true,
            pathRewrite: {
                '^/test': ''
            }
        },
        '/prod': {
            target: 'http://prod.xxxx.com.cn',
            changeOrigin: true,
            pathRewrite: {
                '^/prod': ''
            }
        }
    },
    watchOptions: { // 监控文件相关配置
        poll: true,
        ignored: /node_modules/, // 忽略监控的文件夹, 正则
        aggregateTimeout: 300, // 默认值, 当你连续改动时候, webpack可以设置构建延迟时间(防抖)
    }
}
```


## webpack启用babel转码

```
npm i -D  babel-loader @babel/core @babel/preset-env @babel/runtime @babel/plugin-transform-runtime
```

修改配置文件

```
// 编译js
{
    test: /\.js$/,
    exclude: /(node_modules|bower_components)/,
    use: {
        loader: 'babel-loader?cacheDirectory', // 通过cacheDirectory选项开启支持缓存
        options: {
          presets: ['@babel/preset-env']
        }
    }
},
```

增加.babelrc配置文件


```
// 仅供参考

{
    "presets": [
      [
        "@babel/preset-env"
      ]
    ],
    "plugins": [
      [
        "@babel/plugin-transform-runtime",
        {
          "corejs": false,
          "helpers": true,
          "regenerator": true,
          "useESModules": false,
          "absoluteRuntime": "@babel/runtime"
        }
      ]
    ]
  }

```

## webpack配置eslint校验

```
npm i -D eslint eslint-loader

// 校验规则
npm i -D babel-eslint standard
```

修改webpack配置文件

```
// 编译js
  {
    test: /\.js$/,
    exclude: /(node_modules|bower_components)/,
    use: [
      {
        loader: 'babel-loader?cacheDirectory', // 通过cacheDirectory选项开启支持缓存
        options: {
          presets: ['@babel/preset-env']
        }
      },
      {
        loader: 'eslint-loader',
        options: {
          fix: true
        }
      }
    ]
  },
```

增加.eslintrc.js文件

```
/*
 * ESLint的JSON文件是允许JavaScript注释的，但在gist里显示效果不好，所以我把.json文件后缀改为了.js
 */

/*
 * ESLint 配置文件优先级：
 * .eslintrc.js(输出一个配置对象)
 * .eslintrc.yaml
 * .eslintrc.yml
 * .eslintrc.json（ESLint的JSON文件允许JavaScript风格的注释）
 * .eslintrc（可以是JSON也可以是YAML）
 *  package.json（在package.json里创建一个eslintConfig属性，在那里定义你的配置）
 */

/*
 * 你可以通过在项目根目录创建一个.eslintignore文件告诉ESLint去忽略特定的文件和目录
 * .eslintignore文件是一个纯文本文件，其中的每一行都是一个glob模式表明哪些路径应该忽略检测
 */

module.exports = {
  //ESLint默认使用Espree作为其解析器
  //同时babel-eslint也是用得最多的解析器
  //parser解析代码时的参数
  "parser": "babel-eslint",
  "parserOptions": {
    //指定要使用的ECMAScript版本，默认值5
    "ecmaVersion": 6
    //设置为script(默认)或module（如果你的代码是ECMAScript模块)
    // "sourceType": "script",
    //这是个对象，表示你想使用的额外的语言特性,所有选项默认都是false
    // "ecmafeatures": {
    //   //允许在全局作用域下使用return语句
    //   "globalReturn": false,
    //   //启用全局strict模式（严格模式）
    //   "impliedStrict": false,
    //   //启用JSX
    //   "jsx": false,
    //   //启用对实验性的objectRest/spreadProperties的支持
    //   "experimentalObjectRestSpread": false
    // }
  },
  //指定环境，每个环境都有自己预定义的全局变量，可以同时指定多个环境，不矛盾
  "env": {
    //效果同配置项ecmaVersion一样
    "es6": true,
    "browser": true,
    "node": true,
    "commonjs": false,
    "mocha": true,
    "jquery": true,
    //如果你想使用来自某个插件的环境时，确保在plugins数组里指定插件名
    //格式为：插件名/环境名称（插件名不带前缀）
    // "react/node": true
  },

  //指定环境为我们提供了预置的全局变量
  //对于那些我们自定义的全局变量，可以用globals指定
  //设置每个变量等于true允许变量被重写，或false不允许被重写
  "globals": {
    "globalVar1": true,
    "globalVar2": false
  },

  //ESLint支持使用第三方插件
  //在使用插件之前，你必须使用npm安装它
  //全局安装的ESLint只能使用全局安装的插件
  //本地安装的ESLint不仅可以使用本地安装的插件还可以使用全局安装的插件
  //plugin与extend的区别：extend提供的是eslint现有规则的一系列预设
  //而plugin则提供了除预设之外的自定义规则，当你在eslint的规则里找不到合适的的时候
  //就可以借用插件来实现了
  "plugins": [
    // "eslint-plugin-airbnb",
    //插件名称可以省略eslint-plugin-前缀
    // "react"
  ],

  //具体规则配置
  //off或0--关闭规则
  //warn或1--开启规则，警告级别(不会导致程序退出)
  //error或2--开启规则，错误级别(当被触发的时候，程序会退出)
  "rules": {
    "eqeqeq": "warn",
    //你也可以使用对应的数字定义规则严重程度
    "curly": 2,
    //如果某条规则有额外的选项，你可以使用数组字面量指定它们
    //选项可以是字符串，也可以是对象
    "quotes": ["error", "double"],
    "one-var": ["error", {
      "var": "always",
      "let": "never",
      "const": "never"
    }],
    //配置插件提供的自定义规则的时候，格式为：不带前缀插件名/规则ID
    // "react/curly": "error"
  },

  //ESLint支持在配置文件添加共享设置
  //你可以添加settings对象到配置文件，它将提供给每一个将被执行的规则
  //如果你想添加的自定义规则而且使它们可以访问到相同的信息，这将会很有用，并且很容易配置
  "settings": {
    "sharedData": "Hello"
  },

  //Eslint检测配置文件步骤：
  //1.在要检测的文件同一目录里寻找.eslintrc.*和package.json
  //2.紧接着在父级目录里寻找，一直到文件系统的根目录
  //3.如果在前两步发现有root：true的配置，停止在父级目录中寻找.eslintrc
  //4.如果以上步骤都没有找到，则回退到用户主目录~/.eslintrc中自定义的默认配置
  "root": true,

  //extends属性值可以是一个字符串或字符串数组
  //数组中每个配置项继承它前面的配置
  //可选的配置项如下
  //1.字符串eslint：recommended，该配置项启用一系列核心规则，这些规则报告一些常见问题，即在(规则页面)中打勾的规则
  //2.一个可以输出配置对象的可共享配置包，如eslint-config-standard
    //可共享配置包是一个导出配置对象的简单的npm包，包名称以eslint-config-开头，使用前要安装
    //extends属性值可以省略包名的前缀eslint-config-
  //3.一个输出配置规则的插件包，如eslint-plugin-react
    //一些插件也可以输出一个或多个命名的配置
    //extends属性值为，plugin：包名/配置名称
  //4.一个指向配置文件的相对路径或绝对路径
  //5.字符串eslint：all，启用当前安装的ESLint中所有的核心规则
    //该配置不推荐在产品中使用，因为它随着ESLint版本进行更改。使用的话，请自己承担风险
  "extends": [
    "standard"
  ]
}

```

增加.eslintignore文件

```
/dist/
/node_modules/
/*.js

```


## webpack配置resolve模块解析

配置alias方便路径的检索效率。
配置文件默认扩展名，import时候自动匹配。

```
resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src/')
    }
},
```

## webpack配置外部扩展externals

externals选项可以提供排除打包某些依赖到boundle的方法.例如我们想通过CDN引入jQuery而不是把jQuery打包到boudle。

这里以jQuery为例:

修改配置文件
```
// 配置外部依赖不会打包到boudle
externals: {
    jquery: 'jQuery'
},
```

在模板文件引入CDN
```
// index.html

<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta name="google" value="notranslate">
    <meta http-equiv="Cache-Control" content="no-siteapp">
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
    <meta name="format-detection" content="telephone=no">
    <script
    src="https://code.jquery.com/jquery-3.3.1.js"
    integrity="sha256-2Kok7MbOyxpgUVvAk/HJ2jigOSYS2auK4Pfzbm7uH60="
    crossorigin="anonymous"></script>
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>

<body>
    <div id="app"></div>
    <div class="logo"></div>
    <div class="man"></div>
</body>

</html>

```

在index.js使用jquery

```
import $ from 'jquery'

// 测试外部扩展配置
$(function () {
  $('.logo').click(function () {
    console.log('click')
  })
})
```





## webpack打包报表分析以及优化


```
npm i -D webpack-bundle-analyzer
```

我单独配置了一个命令进行打包分析:


```
"build:report": "cross-env NODE_ENV=production npx webpack --progress --config webpack.analysis.config.js"
```

当然你其实完全可以通过传参数配置集成到prod那个配置文件如：

```
"build:report": "npm run build --report"
```

然后在prod配置环境中如果参数判断:

```
// 伪代码
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
```

这里给出我的方案

```
const merge = require('webpack-merge')
const prodWebpackConfig = require('./webpack.prod.config.js')
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = merge(prodWebpackConfig, {
  plugins: [
    new BundleAnalyzerPlugin() // 打包分析
  ]
})

```


## 仓库地址

[Rp地址](https://github.com/NuoHui/webpack-demo)

有用的话点个star, 有问题欢迎提issues。
