## 概念
本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
## 四个核心概念
- 入口(entry)
- 输出(output)
- loader
- 插件(plugins)

## webpack的安装
### 全局安装
```
cnpm uninstall webpack webpack-cli -g
```
### 局部安装
```
cnpm install webpack webpack-cli -D
```
### npx运行
```
npx webpack -v
```
### webpack的信息
```
npm info webpack
```
### webpack的信息
#### 新建一个webpack.config.js文件
```
const path = require('path')
module.exports = {
    entry: './index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    }
}
```
#### 运行
```
webpack
或
npx webpack  执行的默认文件为webpack.config.js
```

#### 设置指定的js文件
```
npx webpack --config 
```

#### 设置命令 即可用 npm run build
```
"scripts": {
    "build":"webpack"
},
```
#### mode
development 打包后不会压缩
production 打包后会压缩

## module
### rules
#### url-loader

url-loader和file-loader的区别：url-loader的图片转base64

```
rules: [{
	test: /\.(jpg|png|gif)$/,
			use: {
		loader: 'url-loader',
		options: {
			name: '[name]_[hash].[ext]',
			outputPath: 'images/',
			limit: 10240
		}
	} 
}, {
	test: /\.(eot|ttf|svg)$/,
	use: {
		loader: 'file-loader'
	} 
}, {
	test: /\.scss$/,
	use: [
		'style-loader', 
		{
			loader: 'css-loader',
			options: {
				importLoaders: 2
			}
		},
		'sass-loader',
		'postcss-loader'
	]
}, {
	test: /\.css$/,
	use: [
		'style-loader',
		'css-loader',
		'postcss-loader'
	]
}]
```

#### bable
```
npm i babel-loader -D
npm i @babel/core -D
npm i @babel/preset-env -D
npm i babel-polifill --save

rules:[   
	{
		test: /\.js$/,   //匹配JS文件  
		use: 'babel-loader',
		exclude: /node_modules/,   //排除node_modules目录
		option: {
			presets: [
				[
					'@babel/preset-env',
					{
						'targets': {
							chrome: '67'
						},
						"useBuiltIns": "usage"
					}
				]
			]
		}
	} 
]
```

![](https://user-gold-cdn.xitu.io/2020/4/17/17183ff7d507baff?w=809&h=324&f=png&s=38044)

#### postcss-loader新建文件如图
![](https://user-gold-cdn.xitu.io/2020/4/15/1717b628821fd4c2?w=633&h=272&f=png&s=25037)

## plugins
### html-webpack-plugin
### CleanWebpackPlugin
### HotModuleReplacementPlugin
```
plugins: [
	new HtmlWebpackPlugin({
		template: 'src/index.html'
	}), 
	new CleanWebpackPlugin(['dist']),
	new webpack.HotModuleReplacementPlugin()
],
```
## output
```
output: {
	publicPath:'./',
	filename: '[name].js',
	path: path.resolve(__dirname, 'dist')
}
```

## devtool

### sourceMap
```
devtool: 'cheap-module-eval-source-map',
```

![](https://user-gold-cdn.xitu.io/2020/4/24/171aa11054fe87ea?w=893&h=1053&f=png&s=81531)

## devServer
```
"scripts": {
    "dev-build": "webpack --config ./build/webpack.common.js",
    "dev": "webpack-dev-server --config ./build/webpack.common.js",
    "build": "webpack --env.production --config ./build/webpack.common.js"
},
```

```
devServer: {
	contentBase: './dist',
	open: true,
	port: 8080,
	hot: true,
	hotOnly: true
},
```

## tree shaking
- 只支持ES Module
```
optimization: {
	runtimeChunk: {
		name: 'runtime'
	},
	usedExports: true,
	splitChunks: {
		chunks: 'all',
		cacheGroups: {
			vendors: {
				test: /[\\/]node_modules[\\/]/,
				priority: -10,
				name: 'vendors',
			}
		}
	}
}
```
- JSON文件里面：false或者数组指定不处理的
```
sideEffects: false 
```

## Develoment 和 Production 模式的区分打包
### JSON文件夹 

```
"scripts": {
    "dev": "webpack-dev-server --config ./build/webpack.dev.js",
    "build": "webpack --config ./build/webpack.prod.js",
},
```

### 把公共的提取出来放入common.js里面
#### webpack.dev.js添加如下代码
```
<!--先 npm install --save-dev webpack-merge-->
const merge = require('webpack-merge');
const commonConfig = require('./webpack.common.js');
module.exports = merge(commonConfig, devConfig); 
```

## SplitChunksPlugin 
```
optimization: {
	splitChunks: {
		chunks: "all",          //async异步代码分割 initial同步代码分割 all同步异步分割都开启
		minSize: 30000,         //字节 引入的文件大于30kb才进行分割
		//maxSize: 50000,       //50kb，尝试将大于50kb的文件拆分成n个50kb的文件
		minChunks: 1,           //模块至少使用次数
		maxAsyncRequests: 5,    //同时加载的模块数量最多是5个，只分割出同时引入的前5个文件
		maxInitialRequests: 3,  //首页加载的时候引入的文件最多3个
		automaticNameDelimiter: '~', //缓存组和生成文件名称之间的连接符
		name: true,                  //缓存组里面的filename生效，覆盖默认命名
		cacheGroups: { //缓存组，将所有加载模块放在缓存里面一起分割打包
			vendors: {  //自定义打包模块
				test: /[\\/]node_modules[\\/]/,
				priority: -10, //优先级，先打包到哪个组里面，值越大，优先级越高
				filename: 'vendors.js',
            },
			default: { //默认打包模块
				priority: -20,
				reuseExistingChunk: true, //模块嵌套引入时，判断是否复用已经被打包的模块
				filename: 'common.js'
			}
		}
	}
}
```
### Lazy loading 即import()这种写法
### chunks 
- chunks的含义是拆分模块的范围，它有三个值async、initial和all。
## 打包分析（即analysis工具）
```
npm install --save-dev webpack-bundle-analyzer

在webpack的plugins中配置： new BundleAnalyzerPlugin()

在package.json 中增加： "analyz": "NODE_ENV=production npm_config_report=true npm run build"
```
## Preloading 
```
document.addEventListener('click',()=>{
    import(/* webpackPreload: true */'./util/click').then(({default:func}) =>{
        func();
    })
})
```
## Prefetching 空闲时间加载
```
document.addEventListener('click', () => {
  import(/* webpackPrefetch: true */ './util/click').then(({default: func}) => {
    func();
  })
});
```
- Preloading 和 Prefetching 有什么别？
```
两者的最大区别在于，Prefetching 是在核心代码加载完成之后带宽空闲的时候再去加载，而 Preloading 是和核心代码文件一起去加载的
```

## CSS文件代码分割和压缩
- 使用mini-css-extract-plugin后，就可以把css提取到单独的文件。
- 使用optimize-css-assets-webpack-plugin可以压缩css文件。
- 通过配合html-webpack-plugin插件的使用，生成的html就会自动引入css文件。

```
<!--先 npm install --save-dev mini-css-extract-plugin optimize-css-assets-webpack-plugin-->

const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
const prodConfig = {
	module: {
		rules:[{
			test: /\.scss$/,
			use: [
				MiniCssExtractPlugin.loader, 
				{
					loader: 'css-loader',
					options: {
						importLoaders: 2
					}
				},
				'sass-loader',
				'postcss-loader'
			]
		}, {
			test: /\.css$/,
			use: [
				MiniCssExtractPlugin.loader,
				'css-loader',
				'postcss-loader'
			]
		}]
	},
	optimization: {
		minimizer: [new OptimizeCSSAssetsPlugin({})]
	},
	plugins: [
		new MiniCssExtractPlugin({
			filename: '[name].css',
			chunkFilename: '[name].chunk.css'
		})
	],
}
```

- JSON文件里面：false修改为数组，里面是不处理的文件
```
sideEffects: ["*.css"] 
```

## hash解决浏览器cache
```
output: {
  filename: '[name].[contenthash].js',
  chunkFilename: '[name].[contenthash].js'
}
```

## Shimming 的作用
- 如果使用了一些版本比较老的模块如jquery、lodash，这些老模块的用法不是ES Module的使用方式，如果用webpack打包用这种模块会报错，为了解决这样的错误就要用到webpack.ProvidePlugin()
```
const webpack = require('webpack');

plugins: [
  new HtmlWebpackPlugin({
      template: 'src/index.html'
  }), 
  new CleanWebpackPlugin(),
  new webpack.ProvidePlugin({
      $: 'jquery',
      _join: ['lodash', 'join']
      //如果一个模块中使用了$字符串,就会在模块中自动得引用jquery
  })
]
```
## 让this===window true
```
<!--先 npm install imports-loader -D-->

rules: [
  {
      test: require.resolve("some-module"),
      use: "imports-loader?this=>window"
  }
]
```

## 库打包
- externals 忽略指定库
- webpack默认打包之后的代码形式是这样的（假设我导出 module.exports = 'hello world' ）
```
(function () {
  return 'hello world'
})()
```
- 外界想获取函数里面的返回值怎么办，那么就需要配置一个 library
```
module.exports = {
	mode: 'production',
	entry: './src/index.js',
	externals: 'lodash', // 忽略这些库
	output: {
		path: path.resolve(__dirname, 'dist'),
		filename: 'library.js',
		library: 'root', // 模块名称
		libraryTarget: 'umd' // 输出格式 也可以绑定this/global等
	}
}
```
- 打包之后是这样的形式
```
var result = (function () {
  return 'hello world'
})()
```
- 详解webpack的out.libraryTarget属性 https://blog.csdn.net/frank_yll/article/details/78992778
### 知识补充:引入方式
- 传统方式：script标签
```
<script src="demo.js"></script>
<script>demo();</script>
```

- AMD和CMD
```
define(['demo'], function(demo) {
demo();
});
```

- CommonJS
```
const demo = require('demo');
demo();
```

- ES6 module
```
import demo from 'demo';
demo();
```
- 各方式区别 https://juejin.im/post/5aaa37c8f265da23945f365c
## PWA 的打包配置
### 安装
```
cnpm install -D workbox-webpack-plugin
```
![](https://user-gold-cdn.xitu.io/2020/4/19/17191d7465ffb23f)

![](https://user-gold-cdn.xitu.io/2020/4/19/17191d778f0a667b?w=446&h=284&f=png&s=118764)

![](https://user-gold-cdn.xitu.io/2020/4/19/17191d8362600aaa?w=1008&h=367&f=png&s=42952)
## 启动一个服务
### 安装
```
cnpm i http-server -D
```
### JSON文件中
```
"scripts": {
  "server": "http-server dist",
},
```
## TypeScript 的打包配置
### 在node开发中使用npm init会生成一个pakeage.json文件
npm install ts-loader typescript --save-dev
```
const path = require('path');
module.exports = {
    mode: 'production',
    entry: {
        main: './src/index.ts'
    },
    module: {
        rules: [{
            test: /\.ts?$/,
            use: 'ts-loader', // 借助ts-loader依赖进行打包
            exclude: /node_modules/ // 除node_modules文件夹下之外的以.ts结尾的文件
        }]
    },
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist')
    }
}
```
### 根目录下建一个文件 tsconfig.json
```
{
    "compilerOptions": {
        "outDir": "./dist",  // 当用ts-loader做ts打包的时候，把打包生成的文件会放到dist目录下，这个不写也行，因为在webpack.config.js里已经配置过了output
        "module": "es6",   // 在index.tsx代码里，我们用ES6这种模块引入方式
        "target": "ES5",  // 打包ts语法的时候，最终转换成ES5的代码，方便大部分浏览器运行
        "allowJs": true  // 运行引入JS文件
    }
}
```
### 库也支持ts的话，要安装库的类型文件
npm install @types/xxx --save-dev

## 使用 Webpack DevServer 实现请求转发
## WebpackDevServer 解决单页面应用路由问题
```
devServer: {
    port: 8082,
    open: true,
    compress: true,
    historyApiFallback:true,
    proxy: {
        '/api': {
            target: 'http://localhost: 3002',
            changeOrigin: true, 
            pathRewrite: {
                '^/api': '/api'
            },
            changeOrigin:true
        }
    }
}
```
## Eslint
npm i -D eslint
新建一个配置文件.eslintrc.js
```
文件里面内容是规则（省略）
```

```
rules: [
  {
    test: /\.(vue|js|jsx)$/,
    loader: 'eslint-loader',
    exclude: /node_modules/,
    enforce: 'pre'
  },
]
```

## webpack性能优化
###  更新node，npm版本
###  exclude/include的使用
```
{
    module: {
        loaders: [{
            test: /\.js$/,
            loader: 'babel-loader',
            include: [
                path.resolve(__dirname, "app/src"),
                path.resolve(__dirname, "app/test")
            ],
            exclude: /node_modules/
        }]
    }
}
```
#### 基本大部分的node_modules都压缩过的，不要再次打包 
```
rules:[   
	{
		test: /\.js$/,   //匹配JS文件  
		use: 'babel-loader',
		exclude: /node_modules/,   //排除node_modules目录
	} 
]
```
###  不同环境使用合理的插件Plugin（选社区认可的Plugin）
#### dev环境不压缩 OptimizeCSSAssetsPlugin

###  resolve的合理配置  而资源类写后缀（如图片）
####  通过extensions配置寻找顺序
```
resolve: {
    extensions: ['.js', '.vue', '.json'],
    mainFiles:[
        'index','main'
    ],// 当只引入文件夹，默认寻找（其实这个配置影响打包性能）
    alias: {
        'vue$': 'vue/dist/vue.esm.js',
        '@': resolve('src'),
    }
},
```
###  DllPlugin
- 新建一个webpack.dll.js文件
```
const path = require('path');
const webpack = require('webpack');

module.exports = {
	mode: 'production',
	entry: {
		vendors: ['lodash'],
		react: ['react', 'react-dom'],
		jquery: ['jquery']
	},
	output: {
		filename: '[name].dll.js',
		path: path.resolve(__dirname, '../dll'),
		library: '[name]'
	},
	plugins: [
		new webpack.DllPlugin({
			name: '[name]',
			path: path.resolve(__dirname, '../dll/[name].manifest.json'),
		})
	]
}
```
- 新建一个webpack.common.js文件
npm i add-asset-html-webpack-plugin -D
```
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');
plugins: [
	new HtmlWebpackPlugin(),
	new webpack.DllReferencePlugin({
		context:path.join(__dirname),
		manifest:require('./build/vendor-manifest.json'),
	}),
	new AddAssetHtmlPlugin({ filepath:path.resolve(__dirname, './build/*.dll.js'), }),
],
```
- 批量处理
```
const path = require('path');
const fs = require('fs');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');
const webpack = require('webpack');


const makePlugins = (configs) => {
	const files = fs.readdirSync(path.resolve(__dirname, '../dll'));
	files.forEach(file => {
		if (/.*\.dll.js/.test(file)) {
			plugins.push(new AddAssetHtmlWebpackPlugin({
				filepath: path.resolve(__dirname, '../dll', file)
			}))
		}
		if (/.*\.manifest.json/.test(file)) {
			plugins.push(new webpack.DllReferencePlugin({
				manifest: path.resolve(__dirname, '../dll', file)
			}))
		}
	});
	return plugins;
}

const configs = {
	entry: {
		index: './src/index.js',
		list: './src/list.js',
		detail: './src/detail.js',
	},
	resolve: {
		extensions: ['.js', '.jsx'],
	},
	module: {
		rules: [{
			test: /\.jsx?$/,
			include: path.resolve(__dirname, '../src'),
			use: [{
				loader: 'babel-loader'
			}]
		}, {
			test: /\.(jpg|png|gif)$/,
			use: {
				loader: 'url-loader',
				options: {
					name: '[name]_[hash].[ext]',
					outputPath: 'images/',
					limit: 10240
				}
			}
		}, {
			test: /\.(eot|ttf|svg)$/,
			use: {
				loader: 'file-loader'
			}
		}]
	},
	optimization: {
		runtimeChunk: {
			name: 'runtime'
		},
		usedExports: true,
		splitChunks: {
			chunks: 'all',
			cacheGroups: {
				vendors: {
					test: /[\\/]node_modules[\\/]/,
					priority: -10,
					name: 'vendors',
				}
			}
		}
	},
	performance: false,
	output: {
		path: path.resolve(__dirname, '../dist')
	}
}

configs.plugins = makePlugins(configs);

module.exports = configs
```
### tree shaking
### thread-loader 多进程打包
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve("src"),
        use: [
          {
            loader: "thread-loader",
            // loaders with equal options will share worker pools
            // 设置同样option的loaders会共享
            options: {
              // worker的数量，默认是cpu核心数
              workers: 2,
              // 一个worker并行的job数量，默认为20
              workerParallelJobs: 50,
              // 添加额外的node js 参数
              workerNodeArgs: ['--max-old-space-size=1024'],
              // 允许重新生成一个dead work pool
              // 这个过程会降低整体编译速度
              // 开发环境应该设置为false
              poolRespawn: false,
              //空闲多少秒后，干掉work 进程
              // 默认是500ms
              // 当处于监听模式下，可以设置为无限大，让worker一直存在
              poolTimeout: 2000,
              // pool 分配给workder的job数量
              // 默认是200
              // 设置的越低效率会更低，但是job分布会更均匀
              poolParallelJobs: 50,
              // name of the pool
              // can be used to create different pools with elsewise identical options
              // pool 的名字
              //
              name: "my-pool"
            }
          },
          // your expensive loader (e.g babel-loader)
        ]
      }
    ]
  }
}
```
### parallel-webpack 多进程打包
### happypack 多进程打包
```
const HappyPack = require('happypack');
var happyThreadPool = HappyPack.ThreadPool({ size: 5 });

exports.module = {
  rules: [
    {
      test: /.js$/,
      // 1) replace your original list of loaders with "happypack/loader":
      // loaders: [ 'babel-loader?presets[]=es2015' ],
      use: ['happypack/loader' ? id = babel], // 这里的id 就是定义在plugin里面HappyPack实例构造参数传入的id
      include: [ /* ... */],
      exclude: [ /* ... */]
    },
    {
      test: /\.less$/,
      use: 'happypack/loader?id=styles'
    },
  ]
};

exports.plugins = [
  // 2) create the plugin:
  new HappyPack({
    // 3) re-add the loaders you replaced above in #1:
    loaders: ['babel-loader?presets[]=es2015'],
    threadPool: happyThreadPool,
    id: 'babel'
  }),
  new HappyPack({
    id: 'styles',
    threadPool: happyThreadPool,
    loaders: ['style-loader', 'css-loader', 'less-loader']
  })
];
```
### 结合stats分析打包结果

## 多页面打包配置
```
const plugins = [
    new CleanWebpackPlugin(['dist'], {
        root: path.resolve(__dirname, '../')
    })
];
Object.keys(configs.entry).forEach(item => {
    plugins.push(
        new HtmlWebpackPlugin({
            template: 'src/index.html',
            filename: `${item}.html`,
            chunks: ['runtime', 'vendors', item]
        })
    )
});
```

## 如何编写一个 Loader
- 新建一个loaders目录
- 目录下新建一个replaceLoader.js文件
```
const loaderUtils = require('loader-utils');
module.exports = function(source) {
	return source.replace('blue', 'black');
}
```
- 目录下新建一个replaceLoaderAsync.js文件
```
const loaderUtils = require('loader-utils');
module.exports = function(source) {
	const options = loaderUtils.getOptions(this);
	const callback = this.async();
	setTimeout(() => {
		const result = source.replace('dalao', options.name);
		callback(null, result);
	}, 1000);
}
```
- webpack.config.js的配置文件
```
const path = require('path');
module.exports = {
	mode: 'development',
	entry: {
		main: './src/index.js'
	},
	resolveLoader: {
		modules: ['node_modules', './loaders']
	},
	module: {
		rules: [{
			test: /\.js/,
			use: [
				// {
				// 	loader: 'replaceLoader',
				// },
				// {
				// 	loader: 'replaceLoaderAsync',
				// 	options: {
				// 		name: 'uncle'
				// 	}
				// },
				{
					loader: path.resolve(__dirname, './loaders/replaceLoader.js')
				},
				{
					loader: path.resolve(__dirname, './loaders/replaceLoaderAsync.js'),
					options: {
						name: 'uncle'
					}
				}
			]
		}]
	},
	output: {
		path: path.resolve(__dirname, 'dist'),
		filename: '[name].js'
	}
}
```

## 如何编写一个 plugins
- 新建一个plugins目录
- 目录下新建一个copyright-webpack-plugin.js文件
```
class CopyrightWebpackPlugin {
	apply(compiler) {
		compiler.hooks.compile.tap('CopyrightWebpackPlugin', (compilation) => {
			console.log('compiler');
		})
		compiler.hooks.emit.tapAsync('CopyrightWebpackPlugin', (compilation, cb) => {
			debugger;
			compilation.assets['copyright.txt'] = {
				source: function () {
					return 'copyright by black uncle'
				},
				size: function () {
					return 24;
				}
			};
			cb();
		})
	}
}
module.exports = CopyrightWebpackPlugin;
```
- webpack.config.js的配置文件
```
const path = require('path');
const CopyRightWebpackPlugin = require('./plugins/copyright-webpack-plugin');

module.exports = {
	mode: 'development',
	entry: {
		main: './src/index.js'
	},
	plugins: [
		new CopyRightWebpackPlugin()
	],
	output: {
		path: path.resolve(__dirname, 'dist'),
		filename: '[name].js'
	}
}
```