##也谈webpack及其开发模式

###从模块化谈起

近年来，js开发涌现出了诸多模块化解决方案，例如以在浏览器环境之外（服务端）构建 `JavaScript` 生态系统为目标而产生的``CommonJS``规范，从``CommonJS``社区中独立出来的`AMD`规范（异步模块定义），还有国人制定的`CMD`规范等。随着遵循`AMD`规范的`RequireJS`的流行，`AMD`规范在前端界已被广泛认同。后来，随着npm社区的逐渐壮大，`CommonJS`也越来越受欢迎，于是产生了统一这两种规范的需求，即希望提供一个前后端跨平台的解决方案，也因此产生了`UMD`（通用模块定义）规范。

`CommonJS`定义的是模块的同步加载，主要用于Node端；而遵循AMD规范的`RequireJS`则是异步加载，适用于浏览器端。`requirejs`是一种在线”编译” 模块的方案，相当于在页面上加载一个AMD 解释器，以便于览器能够识别`define、exports、module`，而这些东西就是用于模块化的关键。
>
`CommonJS` 同步式的require

Node端的模块加载遵循 
`CommonJS`规范，该规范的核心思想是允许模块通过 `require` 方法来加载。

该规范首先加载所要依赖的其他模块，然后通过 `exports` 或 `module.exports` 来导出需要暴露的接口。但它的缺点也是显而易见的，即一个文件一个文件的加载很容易发生阻塞。

	require("module");//find from node_modules
	require("../file.js");
	exports.something = function() {};
	module.exports = someValue;
	
>
使你的模块兼容AMD规范

	//web.js
	(function (root, factory) {
	        //判断define是否存在
	    if (typeof define === 'function' && define.amd) {
	        // 存在则使用AMD方式加载模块
	        define(['b'], factory);
	    } else {
	        // 不存在则使用浏览器全局变量暴露模块 
	        root.web = factory(root.b);
	    }
	}(this, function (b) {
	    //use b in some fashion.
	
	    // Just return a value to define the module export.
	    // This example returns an object, but the module
	    // can return a function as the exported value.
	    return {};
	}));
	
定义一个叫`web.js`的模块，依赖于另一个叫b的模块。如果你不想支持浏览器全局路径，那么你可以移除`root`并传递`this`参数在函数顶部。

>
define.amd属性

为了清晰的标识全局函数（为浏览器加载script必须的）遵从AMD编程接口，任何全局函数应该有一个"amd"的属性，它的值为一个对象。

关于RequireJS的使用不在本文范围之内，因此不展开讲解，有兴趣的请移步我的另一篇文章：

>
详解JavaScript模块化开发：[https://segmentfault.com/a/1190000000733959#articleHeader15](https://segmentfault.com/a/1190000000733959#articleHeader15)

###AMD与异步加载

因为`CommonJS`阻塞式的缺点，所以并不适合前端。于是有了AMD异步加载模块的规范。

`Asynchronous Module Definition` 规范其实只有一个主要接口 `define(id?, dependencies?, factory)`，它要在声明模块的时候指定所有的依赖 `dependencies`，并且还要当做形参传到 `factory` 中，对于依赖的模块提前执行，依赖前置。

因为浏览器端的需求和同步require的问题，所以社区引进了异步模块加载的规范，即AMD规范。

	define("module", ["dep1", "dep2"], function(d1, d2) {
	  return someExportedValue;
	});
	require(["module", "../file"], function(module, file) { /* ... */ });
	使你的模块兼容于UMD规范
	//UMD，兼容AMD和`CommonJS`规范
	(function (root, factory) {
	  if (typeof exports === 'object') {
	    // `CommonJS`
	    module.exports = factory(require('b'));
	  } else if (typeof define === 'function' && define.amd) {
	    // AMD
	    define(['b'], function (b) {
	      return (root.returnExportsGlobal = factory(b));
	    });
	  } else {
	    // 浏览器全局变量，root即window
	    root.returnExportsGlobal = factory(root.b);
	  }
	}(this, function (b) {
	  // 你的实际模块
	  return {};
	}));
	
>
UMD规范实现的思路：

* 首先判断是否支持Node.js模块格式，即exports对象是否存在。
* 然后判断是否支持AMD格式（require是否存在），存在则使用AMD方式加载
* 若前两个都不存在，则将模块暴露到全局，Node即global，浏览器即window。

例如，创建一个兼容UMD规范的jQuery插件：

	// Uses `CommonJS`, AMD or browser globals to create a jQuery plugin.
	
	(function (factory) {
	    if (typeof define === 'function' && define.amd) {
	        // AMD. Register as an anonymous module.
	        define(['jquery'], factory);
	    } else if (typeof module === 'object' && module.exports) {
	        // Node/`CommonJS`
	        module.exports = function( root, jQuery ) {
	            if ( jQuery === undefined ) {
	                // require('jQuery') returns a factory that requires window to
	                // build a jQuery instance, we normalize how we use modules
	                // that require this pattern but the window provided is a noop
	                // if it's defined (how jquery works)
	                if ( typeof window !== 'undefined' ) {
	                    jQuery = require('jquery');
	                }
	                else {
	                    jQuery = require('jquery')(root);
	                }
	            }
	            factory(jQuery);
	            return jQuery;
	        };
	    } else {
	        // Browser globals
	        factory(jQuery);
	    }
	}(function ($) {
	    $.fn.jqueryPlugin = function () { return true; };
	    
###CMD

`Common Module Definition` 规范和 AMD 很相似，尽量保持简单，并与 `CommonJS` 和 `Node.js` 的 `Modules` 规范保持了很大的兼容性。

	define(function(require, exports, module) {
	  var $ = require('jquery');
	  var Spinning = require('./spinning');
	  exports.doSomething = ...
	  module.exports = ...
	})
	
CMD规范地址：[seajs/seajs#242](https://github.com/seajs/seajs/issues/242)

###ES6 module

`ECMAScript6` 內建的用法：

	import "jquery";
	export function doStuff() {}
	module "localModule" {}

###为什么只载入JavaScript文件？

为什么模块化系统只帮助开发者处理`JavaScript`？然而还有其他静态资源需要被处理，比如：

	stylesheets
	images
	webfonts
	html for templating
	其他..
 
	coffeescript ➞ javascript
	less stylesheet ➞ css
	jade ➞ html
	i18n ➞ something
	require("./style.css");
	require("./style.less");
	require("./template.jade");
	require("./image.png");

因为上面这些动机，所以有了webpack

###webpack

`webpack`是一款模块封装工具(module bundler，是打包工具，也是模块加载工具，各种资源都可以当成模块来处理)，`webpack` 会将模块与其他相关联的模块，函数库，其他需要预编译的文件等整合，编译输出此模块的静态资源文件。

	// webpack.config.js `like=>`  gulpfile.js/gruntfile.js
	
	module.exports = {
	    entry: "./entry.js",
	    output: {
	        path: __dirname,
	        filename: "bundle.js"
	    },
	    //module 对象用于添加loaders
	    module: {
	        //一个用于加载loader的数组
	        loaders: [
	            { test: /\.css$/, loader: "style!css" }
	        ]
	    }
	}; 
	
module具有如下属性：

* test: 需要满足的条件A condition that must be met
* exclude: 不需要满足的条件A condition that must not be met
* include: 需要满足的条件A condition that must be met
* loader: !用于分隔loaders
* loaders: 一个loaders数组An array of loaders as string

`include` 通常被用于匹配目录：

      include: [
        path.resolve(__dirname, "app/src"),
        path.resolve(__dirname, "app/test")
      ]
      
简单的说，webpack会把我们常用的 `.less, .scss, .jade, .jsx` 等等文件编译成纯 `js + 图片`(图片有时也可以被编译成 `base64` 格式的 `dataUrl`)。

>
webpack的优势和特点

* 将依赖项分块，按需加载
* 尽可能减少初始化载入的时间
* 使每一个静态资源都能够作为组件使用
* 有能力整合其他第三方函数库为模块
* 高度可配置化
* 适合大型项目

webpack 拥有更聪明的解析工具可以处理几乎所有的第三方函数库。甚至允许在相依性设定上使用表达式，例如: `require("./templates/" + name + ".jade")`。

这几乎能处理大部分的模块化标准(CommonJS, AMD)

###webpack常用命令

	webpack 最基本的启动webpack命令
	webpack -w 提供watch方法，实时进行打包更新
	webpack -p 对打包后的文件进行压缩
	webpack -d 提供SourceMaps，方便调试
	webpack --colors 输出结果带彩色，比如：会用红色显示耗时较长的步骤
	webpack --profile 输出性能数据，可以看到每一步的耗时
	webpack --display-modules 默认情况下 node_modules 下的模块会被隐藏，加上这个参数可以显示这些被隐藏的模块
	
###在项目中使用webpack

首先在项目根目录新建一个`package.json`或者通过`$ npm init`指令来产生：

接着通过`npm`指令安装`webpack`：

	$ npm install webpack --save-dev
	
	or => $ cnpm i webpack -g

>单纯的编译指令

	$ webpack <entry> <output>
	
###配置对象内容

>context：用来指明entry选项的基础目录（绝对路径）。

默认值为`process.cmd()`,即`webpack.config.js`文件所在路径

>对象entry

定义了打包后的入口文件，可以是数组（所有文件打包生成一个filename文件），对象或者字符串

	{
	    entry: {
	        page1: "./page1",
	        page2: ["./entry1", "./entry2"]
	    },
	    output: {
	        // 在 output.filename 中使用 [name]或者[id]，当使用多个entry points时
	        filename: "[name].bundle.js",
	        path: "dist/js/page",
	        chunkFilename: "[id].bundle.js" //chunkFilename是非主入口的文件名
	    }
	}
	
该段代码最终会生成一个`page1.bundle.js` 和 `page2.bundle.js`，并存放到 `./dist/js/page` 文件夹下

`chunkFilename`是非主入口的文件名，当按需异步加载模块的时候，这时生成的文件名是以`chunkname`配置的

* `output`：该参数是个对象，定义了输出文件的位置及名字：
* `path`: 打包文件存放的绝对路径
* `publicPath`: 网站运行时的访问路径URL
* `filename`:打包后的文件名

你可以使用`<name>=<filename>` 的格式来替代 `entry point` 建立一个別名：

	// webpack.config.js
	module.exports = {
	  output: {
	    filename: "[name].bundle.js"
	  }
	}

接着执行如下命令：

	$ webpack index=./entry.js
	>> Output a file that is named index.bundle.js

>对象output

表示欲输出的路径，其会被映射到设定档中的 `output.path` 以及 `output.filename`

* `output.filename`：指定每一个在磁盘上输出的文件名，不允许指定绝对路径

* `output.path`：输出绝对路径目录（必须）

* `output.publicPath`：指定在浏览器端引用的文件公开URL地址

>对象resolve

webpack在构建包的时候会按目录进行文件的查找，`resolve`属性中的`extensions`数组可用于配置程序可以自行补全哪些文件后缀。`extensions` 第一个是空字符串，对应不需要后缀的情况。比如，为了查找`CoffeeScript`文件，你的数组应当包含字符串`".coffee"`。使用`extensions`，在引入模块的时候就不需要写后缀，会自动补全

	resolve: {
	        extensions: ['', '.js', '.jsx','.es6','css','scss','png','jpg']
	    },
* `resolve.alias`:定义别名

	alias:{
	       'react-dom':path.join(nodeModulesPath,'/dist/react-dom'),
	        'redux': path.join(nodeModulesPath,'dist/redux')
	    },
	    
>对象externals

当我们想在项目中`require`一些其他的类库或者API，而又不想让这些类库的源码被构建到运行时文件中，
这在实际开发中很有必要。此时我们就可以通过配置`externals`参数来解决这个问题：

	 externals: {
	     "jquery": "jQuery"
	 }
	 
这样我们就可以放心的在项目中使用这些API了：

	var $ = require(“jquery”);

配置项详情：[https://webpack.github.io/docs/configuration.html](https://webpack.github.io/docs/configuration.html)

###css样式和图片的加载

你可以在你的js文件里引入css文件和图片，例如：

	require('./bootstrap.css');
	require('./style.less');
	require('../../main.scss');
	
	var img = document.createElement('img');
	img.src = require('./myImg.png');
	
当你`require`了CSS(less或者其他)文件，`webpack`会在页面中插入一个内联的`<style>`标签去引入样式。当你`require`图片的时候，bundle文件会包含图片的`url`，并通过`require()`返回图片的`url`。

当然，你需要在`webpack.config.js`里做相应的配置：

	// webpack.config.js
	module.exports = {
	  entry: './entry.js',//入口文件
	  output: {
	    path: './build', // 输出js和图片的目录
	    publicPath: 'http://mycdn.com/', // 用来生成图片的地址
	    filename: 'bundle.js'
	  },
	  module: {
	    loaders: [
	      { test: /\.less$/, loader: 'style-loader!css-loader!less-loader' }, // 用!去链式调用loader
	      { test: /\.css$/, loader: 'style-loader!css-loader' },
	      { test: /\.scss$/,loaders: ["style", "css", "sass"]}
	      {test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // 内联的base64的图片地址，图片要小于8k，直接的url的地址则不解析
	    ]
	  }
	};
	
当然，你需要先通过`npm`包来安装这些`loader，webpack`会通过`test`查找匹配文件，然后加载相应的`loader`。比如通过`npm`安装`sass loader`：

	$ npm install sass-loader node-sass webpack --save-dev

####实例一

新建一个`content.js`

	// content.js
	module.exports = "It works from content.js";

然后编辑 `entry.js` 加入 `require`

	// File: entry.js
	document.write(require("./content.js"));
	
打开浏览器，看到屏幕输出：`"It works from content.js"`;

Webpack 会给每个模块一个唯一的 `ID` 然后通过 `ID` 存取这些模块，这些模块都会被整合到 `bundle.js` 里面。

###webpack插件和使用方法介绍

>`CommonsChunkPlugin`合并公共代码

它用于提取多个入口文件的公共脚本部分，然后生成一个 公共文件来方便多页面之间进行复用。以下是该插件的使用方法和webpack的具体用法介绍：

	var webpack = require('webpack');
	var path = require('path');
	var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
	
	var config = {
	    //Webpack 本身内置了一些常用的插件，还可以通过 npm 安装第三方插件。
	    plugins: [commonsPlugin],
	    //页面入口文件配置
	    entry: {
	        index : './index.js'
	    },
	    //入口文件输出配置
	    output: {
	        path: 'dist/js/page',
	        filename: '[name].js'
	    },
	    //module 的作用是添加loaders
	    module: {
	        //加载器配置
	        loaders: [
	            //.css 文件使用 style-loader 和 css-loader 来处理
	            { test: /\.css$/, loader: 'style-loader!css-loader' },//test属性匹配css文件
	            //.js 文件使用 jsx-loader 来编译处理
	            { test: /\.js$/, loader: 'jsx-loader?harmony' },
	            //.scss 文件使用 style-loader、css-loader 和 sass-loader 来编译处理
	            { test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
	            //图片文件使用 url-loader 来处理，小于8kb的直接转为base64
	            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'},
	            {//处理字体
	              test: /\.(woff2?|otf|eot|svg|ttf)$/i,
	            },
	            loader: 'style!css'//加载style和css loader
	        ]
	    },
	    //其它解决方案配置
	    resolve: {
	        root: '/Users/trigkit4/webpack', //绝对路径
	        extensions: ['', '.js', '.json', '.scss'],//文件扩展名
	        //模块别名定义，方便后续直接引用别名
	        alias: {
	            a : './assets/a.js',  // require(“a”)即可引用该模块
	            b : './assets/b.js',
	            c : './assets/c.js'
	        }
	    }
	};
	
	module.exports = config;
	
`Webpack`中将打包后的文件都称之为`Chunk`。这个插件可以将多个打包后的资源中的公共部分打包成单独的文件。这里指定公共文件输出为`common.js`

`Webpack` 本身只能处理 `JavaScript` 模块，如果要处理其他类型的文件，就需要使用`loader`进行转换。 
不同模块的加载是通过模块加载器（`webpack-loader`）来统一管理的。

`！`用来定义`loader`的串联关系，`”-loader”`是可以省略不写的，多个`loader`之间用`“!”`连接起来，但所有的加载器都需要通过`npm`来加载。

>UglifyJsPlugin压缩js文件

	//webpack.config.js
	
	//...other webpack settings
	
	plugins: [
	    new Webpack.optimize.UglifyJsPlugin({
	        compress: {
	            warnings: false
	        }
	    })
	]

>Extract Text Plugin

大家都知道在`webpack`中 `CSS` 是可以被 `require()` 的，`webpack` 会自动生成一个 `<style>` 标签并加入到 `html` 的 `<head>`标签 中。但是在发布时，我们可能只希望有一个被打包过后 css 文件。这时 `Extract Text Plugin` 就能帮助我们完成这项任务。

	//webpack.config.js
	var ExtractTextPlugin = require("extract-text-webpack-plugin");
	module.exports = {
	    module: {
	        loaders: [
	            { test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader") }
	        ]
	    },
	    plugins: [
	        new ExtractTextPlugin("app.bundle.css")
	    ]
	}

以上设置会输出一个 `app.bundle.css `的文件。

>html-webpack-plugin

webpack中生成HTML的插件。

	var HtmlWebpackPlugin = require('html-webpack-plugin')
	var webpackConfig = {
	  entry: 'index.js',
	  output: {
	    path: 'dist',
	    filename: 'index_bundle.js'
	  },
	  plugins: [
	    new HtmlWebpackPlugin({ //根据模板插入css/js等生成最终HTML
	            favicon: './src/img/favicon.ico', //favicon路径，通过webpack引入同时可以生成hash值
	            filename: './view/about.html', //生成的html存放路径，相对于path
	            template: './src/view/about.html', //html模板路径
	            inject: true, //js插入的位置，true/'head'/'body'/false
	            hash: true, //为静态资源生成hash值
	            chunks: ['vendors', 'about'],//需要引入的chunk，不配置就会引入所有页面的资源
	            minify: { //压缩HTML文件    
	                removeComments: true, //移除HTML中的注释
	                collapseWhitespace: false //删除空白符与换行符
	            }
	        }),
	  ]      
	}

详情：[https://www.npmjs.com/package/html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

>ProvidePlugin

	plugins: [
	
	        new webpack.ProvidePlugin({
	            React: 'react',
	            ReactDOM: 'react-dom'
	        })
	    ]

`ProvidePlugin` 插件可以定义一个共用的插件入口，以后的文件就不需要`require('react')`也能使用React了。

>babel loader

`Babel-loader`能够将`JSX/ES6` 文件转为`js`文件

	$ npm install babel-loader babel-core babel-preset-es2015 babel-preset-react --save-dev

配置：

	module.exports = {
	    entry: {}, //文件入口
	    output: { }, //输出出口
	    module: {
	        loaders: [ 
	            {
	                test:/\.js[x]?$/,
	                loader: 'babel-loader',
	                exclude:/node_modules/,
	                query:{presets: ['es2015','react']}
	            },
	        ]
	    },
	    plugins: [ ],//编译的时候所执行的插件数组
	    devtool : "source-map" //调试模式
	};

插件列表：[https://github.com/webpack/docs/wiki/list-of-plugins](https://github.com/webpack/docs/wiki/list-of-plugins)

###开发服务器安装

`Webpack`开发服务器需要单独安装，同样是通过`npm`进行：

	$ sudo npm install -g webpack-dev-server

可以使用`webpack-dev-server`直接启动，也可以增加参数来获取更多的功能，

具体配置可以参见官方文档。在终端输入：

	$ webpack-dev-server

然后打开：http://localhost:8080/webpack-dev-server/index.html

在`webpack.config.js`里配置：

	//使用webpack-dev-server，提高开发效率
	
	module.exports={
	    devServer: {
	        contentBase: './',
	        host: 'localhost',
	        port: 9090, //默认8080
	        inline: true, //可以监控js变化
	        hot: true, //热启动
	    }
	}
	
详情：[https://webpack.github.io/docs/webpack-dev-server.html](https://webpack.github.io/docs/webpack-dev-server.html)

###使用browser-sync实时刷新页面

如果每次更改代码都要重新执行`webpack`命令不免太过麻烦，所以这里推荐使用`browser-sync-webpack-plugin`，可以监听代码变化，实时刷新页面。

安装`browser-sync-webpack-plugin`：

	$ npm install --save-dev browser-sync-webpack-plugin
	$ webpack --watch

`webpack.config.js`配置：

	var BrowserSyncPlugin = require('browser-sync-webpack-plugin');
	
	module.exports = {
	  // ... 
	  plugins: [
	    new BrowserSyncPlugin({
	      // browse to http://localhost:3000/ during development, 
	      // ./public directory is being served 
	      host: 'localhost',
	      port: 3000,
	      server: { baseDir: ['public'] } //根目录就填'./'
	    })
	  ]
	}
	
####实例二：react+webpack+es6开发模式

具体项目地址请移步：[https://github.com/hawx1993/react-webpack-demos](https://github.com/hawx1993/react-webpack-demos)

在线DEMO:[http://hawx1993.github.io/react-webpack-es6-demos/](http://hawx1993.github.io/react-webpack-es6-demos/)