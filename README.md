# Vue-Cli with webpack4 - 一个基于 webpack4 建立的 Vue-Cli 脚手架应用(包含最新可用部件)

> 通过 vue init webpack xxx 建立基于 webpack3 的脚手架应用更新到最新到 webpack4, 内置填坑指南 (敢于使用 ESLint 的一套系统)

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

## Differences from webpack3 to webpack4 - 从 webpack3 到 webpack4 的不同

> 1. webpack4 推荐使用插件 mini-css-extract-plugin 代替旧有 extract-text-webpack-plugin (需要手动更改配置)    
> 目标位置: /build/webpack.prod.conf.js    

```js
// ...省略 (原始行号 10 上下)    
// const ExtractTextPlugin = require('extract-text-webpack-plugin') // 原有插件引用行注释掉    
const MiniCssExtractPlugin = require('mini-css-extract-plugin') // 引入新插件代替    
// ...省略

// ...省略 (原始行号 27 上下)
devtool: config.build.productionSourceMap ? config.build.devtool : false, // 此行原始文件中应该已自动生成, 如没找到可以自己添加
performance: { hints: false }, // 此行手动添加, 与 devtool,output,plugins 等同级
// ...省略

// ...省略 (原始行号 40 上下)
uglifyOptions: {
    compress: {
        warnings: false
    }
},
// 把其中的 compress 节点删除掉, 在新版的 UglifyJsPlugin 2.1或以上的版本中, warning已经是 options的一个直接属性, 不需要跟在 compress 节点下, 如下:
uglifyOptions: { warning: false },
// ...省略

// ...省略 (原始行号 45 上下) 把原旧有插件 extract-text-webpack-plugin 配置注释掉, 添加 mini-css-extract-plugin 插件配置
// new ExtractTextPlugin({
//     filename: utils.assetsPath('css/[name].[contenthash].css'),
//     allChunks: true,
// }),
new miniCssExtractPlugin({

}),
```

> 2. 去掉 webpack.optimize.CommonsChunkPlugin 相关配置, 原始文件共有3处相关配置需要注释掉    
> 目标位置: /build/webpack.prod.conf.js     
>    
> (原始行号 82 上下) new webpack.optimize.CommonsChunkPlugin    
> (原始行号 97 上下) new webpack.optimize.CommonsChunkPlugin    
> (原始行号 104 上下) new webpack.optimize.CommonsChunkPlugin

> 3. 增加 optimization 配置, 与 output,devtool,plugins 等同级    
> 目标位置: /build/webpack.prod.conf.js    

```js
optimization: {
	runtimeChunk: { name: 'manifest' },
	minimizer: [
		new UglifyJsPlugin({
			cache: true,
			parallel: false,
			sourceMap: config.build.productionSourceMap,
			uglifyOptions: {
				warnings: false
			}
		}),
		new OptimizeCSSPlugin({
			cssProcessorOptions: config.build.productionSourceMap ? { safe: true, map: { inline: false } } : { safe: true }
		})
	],
	splitChunks: {
		chunks: 'async',
		minSize: 30000,
		minChunks: 1,
		maxAsyncRequests: 5,
		maxInitialRequests: 3,
		name: false,
		cacheGroups: {
			vendors: {
				test: /[\\/]node_modules[\\/]/,
				name: 'vendor',
				chunks: 'initial',
				priority: -10
			}
		}
	}
},
```