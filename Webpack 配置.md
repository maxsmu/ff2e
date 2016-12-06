### Webpack 配置
devtool
-- 

 * source-map: 在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包文件的构建速度；
 * cheap-module-source-map: 在一个单独的文件中生成一个不带列映射的map，不带列映射提高项目构建速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便；
 * eval-source-map: 使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。不过在开发阶段这是一个非常好的选项，但是在生产阶段一定不要用这个选项；
 * cheap-module-eval-source-map: 这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点；(构建速度更快，但是不利于调试，推荐在大型项目考虑da时间成本是使用)

入口文件配置：entry参数
--

 entry可以是字符串（单入口），可以是数组（多入口）
 
输出文件：output参数
--

 output参数告诉webpack以什么方式来生成/输出文件，值得注意的是，与entry不同，output相当于一套规则，所有的入口都必须使用这一套规则，不能针对某一个特定的入口来制定output规则。
 
 **output**参数里有这几个子参数是比较常用的：
 
* path
 
 path参数表示生成文件的根目录，需要传入一个绝对路径。path参数和后面的filename参数共同组成入口文件的完整路径。 

* publicPath

 publicPath参数表示的是一个URL路径（指向生成文件的根目录），用于生成css/js/图片/字体文件等资源的路径，以确保网页能正确地加载到这些资源。
		
 publicPath参数跟path参数的区别是：path参数其实是针对本地文件系统的，而publicPath则针对的是浏览器；因此，publicPath既可以是一个相对路径，如示例中的`../../../../build/`，也可以是一个绝对路径如`http://www.xxxxx.com/`。一般来说，我还是更推荐相对路径的写法，这样的话整体迁移起来非常方便。那什么时候用绝对路径呢？其实也很简单，当你的html文件跟其它资源放在不同的域名下的时候，就应该用绝对路径了，这种情况非常多见于后端渲染模板的场景。
		
* filename
  
 filename属性表示的是如何命名生成出来的入口文件，规则有以下三种：

 * `[name]`，指代入口文件的name，也就是上面提到的entry参数的key，因此，我们可以在name里利用/，即可达到控制文件目录结构的效果。
 * `[hash]`，指代本次编译的一个hash版本，值得注意的是，只要是在同一次编译过程中生成的文件，这个`[hash]`的值就是一样的；在缓存的层面来说，相当于一次全量的替换。
 * `[chunkhash]`，指代的是当前chunk的一个hash版本，也就是说，在同一次编译中，每一个chunk的hash都是不一样的；而在两次编译中，如果某个chunk根本没有发生变化，那么该chunk的hash也就不会发生变化。这在缓存的层面上来说，就是把缓存的粒度精细到具体某个chunk，只要chunk不变，该chunk的浏览器缓存就可以继续使用。

下面来说说如何利用filename参数和path参数来设计入口文件的目录结构，如示例中的`path:path.resolve(__dirname, './build'),`和`filename: '[name]/entry.js'`，那么对于key为`'index/login'`的入口文件，生成出来的路径就是`build/index/login/entry.js`了，怎么样，是不是很简单呢？
	
 * chunkFilename

 chunkFilename参数与filename参数类似，都是用来定义生成文件的命名方式的，只不过，chunkFilename参数指定的是除入口文件外的chunk（这些chunk通常是由于webpack对代码的优化所形成的，比如因应实际运行的情况来异步加载）的命名。
 
各种Loader配置：module参数
--
webpack拥有一个类似于插件的机制，名为Loader，通过Loader，webpack能够针对每一种特定的资源做出相应的处理。[Loader list](http://webpack.github.io/docs/list-of-loaders.html)

module正是配置什么资源使用哪个Loader的参数（因为同一种资源，可能会有不同的Loader处理）。

module参数有几个子参数，但是最常用的自然还是loaders子参数：

* `test`参数用来指示当前配置项针对哪些资源。
* `exclude`参数用来剔除掉需要忽略的资源。
* `include`参数用来表示本loader配置仅针对哪些目录/文件。注意同时使用这两者的时候，实际上是and的关系。
* `loader/loaders`参数用来指示用哪个/哪些loader来处理目标资源，这两个配置项表达的是同一种方式，只是写法不同，我个人推荐用loader写成一行，多个loader间使用!分割，这种形式类似于管道的概念，又或者说是函数式编程。形如`loader: 'css?!postcss!less'`，可以很明显地看出，目标资源先经`less-loader`处理过后将结果交给`postcss-loader`作进一步处理，然后最后再交给`css-loader`。

官方示例：

	module.loaders: [{
	    // "test" is commonly used to match the file extension
	    test: /\.jsx$/,
	
	    // "include" is commonly used to match the directories
	    include: [
	      path.resolve(__dirname, "app/src"),
	      path.resolve(__dirname, "app/test")
	    ],
	
	    // "exclude" should be used to exclude exceptions
	    // try to prefer "include" when possible
	
	    // the "loader"
	    loader: "babel-loader"
 	 }]
 	 
添加额外功能：plugins参数
--
plugins参数相当于一个插槽位（类型是数组），你可以先按某个plugin要求的方式初始化好了以后，把初始化后的实例丢到这里来。
