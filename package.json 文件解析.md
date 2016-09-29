### package.json 文件解析

* Name

 必须字段。

>
小提示：
>
1. 不要在name中包含js, node字样;
2. 这个名字最终会是URL的一部分，命令行的参数，目录名，所以不能以点号或下划线开头；
3. 这个名字可能在require()方法中被调用，所以应该尽可能短；
4. 如果包要发布到NPM平台上的话，最好先检查下有没有重名, 并且字母只能全部小写。

* version

 版本号，必须字段。取值需要符合[node-semver](https://github.com/npm/node-semver)的规则

* description

 描述信息,可选字段，必须是字符串。`npm search`的时候会用到。

* keywords

 关键字,可选字段，字符串数组。npm search的时候会用到。

* homepage

 主页地址，可选字段，没有`http://`等带协议前缀的`URL`。

* bugs

 问题追踪系统的URL或邮箱地址，可选字段。`npm bugs`用的上。

		{
			"url" :"http://github.com/owner/project/issues",
			"email" :"project@hostname.com"
		}

* license

	开源协议名称，可选字段。

	如果是使用一个普遍的license，比如`BSD-3-Clause`或`MIT`，直接使用：

		{ "license" : "BSD-3-Clause" }

* author :`String`or`Object`

 作者，可选字段。
 
 author的格式如下：
 	
 		author: {  
		  "name": "maxsmu",
		  "email": "test@sina.com",
		  "url": "http://www.qiongyoo.com"
		}
		
 或者
 
 		author: "maxsmu <test@sina.com> (http://www.qiongyoo.com)" 
 
* contributors :`Array`

 贡献者，可选字段。

* files

 项目包含的一组文件，可选字段。如果是文件夹，文件夹下的文件也会被包含。如果需要把某些文件不包含在项目中，添加一个`.npmignore`文件。这个文件和`gitignore`类似。

* main

 程序主入口模块的ID，可选字段。如果其他用户需要你的包，当用户调用`require()`方法时，返回的就是这个模块的导出`（exports）`。

* bin

 可执行文件配置，可选字段。如果项目包中有多个执行文件，可以通过此字段安装到系统PATH中。这样执行起来比较方便。

 这个字段对应的是一个Map，每个元素对应一个{ 命令名：文件名 }。

		{ "bin" : { "npm" : "./cli.js" } }

* directories

 用于指示包的目录结构：

* directories.lib

 指示库文件的位置。

* directories.bin

 和前面的bin是一样的，但如果前面已经有bin，那么这个就无效。

 除了以上两个，还有`directories.doc`& `directories.man` & `directories.example`。

* repository

 仓库地址，可选字段。用于指示代码存放的位置。

		"repository":{
				"type": "git",
				"url": "http://github.com/npm/npm.git"
			}
			
* scripts

 脚本命令集，可选字段，`object`。
 
 Key是生命周期事件名，value是在事件点要跑的命令。参考[npm-scripts](https://npmjs.org/doc/misc/npm-scripts.html)。

* config

 可选字段，`object`。

 `config`对象中的值在`scripts`的整个周期中皆可用，专门用于给`scripts`提供配置参数。
 
 		"name":"foo"
 		"config": {
 			"port": "8090"
 		}
 		
 `npm start`的时候会读取到`npm_package_config_port`环境变量。

 同时也可以使用`npm config`命令来修改设置：
 	
 		npm config set foo:port 8001  
 参考[npm-config](https://npmjs.org/doc/misc/npm-config.html)

* dependencies

 当前包所依赖的其他包,可选字段。这些依赖是指包发布后正常执行时所需要的，如果是开发中依赖的包，可以在devDependencies设置。
 
 通常使用下面命令来安装:
 
 		npm install --save packageName
例如：
	
		{ 
			"dependencies" : {
				"foo" : "1.0.0 - 2.9999.9999",
				"bar" : ">=1.0.2 <2.1.2",
				"baz" : ">1.0.2 <=2.3.4",
				"boo" : "2.0.1",
				"qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
				"asd" : "http://asdf.com/asdf.tar.gz",
				"til" : "~1.2",
				"elf" : "~1.2.3",
				"two" : "2.x",
				"thr" : "3.3.x"
			}
		}

 版本格式可以是下面任一种：

		1. version 完全匹配
		2. >version 大于这个版本
		3. >=version大于或等于这个版本
		4. <version
		5. <=version
		6. ~version 非常接近这个版本
		7. ^version 与当前版本兼容
		8. 1.2.x X代表任意数字，因此1.2.1, 1.2.3等都可以
		9. http://... Unix系统下使用的tarball的URL（依赖URL）
		10. * 任何版本都可以
		11. ""任何版本都可以
		12. version1 - version2  等价于 >=version1 <=version2.
		13. range1 || range2 满足任意一个即可
		14. git... 依赖Git URL
		15. user/repo  GitHub URLs 
		16. tag 指定某个tag的版本
		17. path/path 本地包所有文件夹


 依赖URL:可以指定一个tarball URL，这个tarball将在包被初始化的时候下载并初始化。 
 
 依赖Git URL:

 Git urls 可以是下面几种形式：

		git://github.com/user/project.git#commit-ish
		git+ssh://user@hostname:project.git#commit-ish
		git+ssh://user@hostname/project.git#commit-ish
		git+http://user@hostname/project/blah.git#commit-ish
		git+https://user@hostname/project/blah.git#commit-ish

 `commit-ish`是可以被`git checkout`的任何`tag`、`sha`或者`branch`。默认为`master`。
 
 GitHub URLs:
 1.1.65版后，你可以仅仅用“user/foo-project”引用GitHub urls，比如：
 		
 		{
		  "name": "foo",
		  "version": "0.0.0",
		  "dependencies": {
		    "express": "visionmedia/express"
		  }
		}
 
* devDependencies

 开发时候需要的依赖包，可选字段。
 
 通常使用以下命令安装：
 
 		npm install --save-dev packageName

* peerDependencies

 兼容性依赖，可选字段。如果你的包是插件，而用户在使用你的包时候，通常也会需要这些依赖（插件），那么可以将依赖列到这里。
 
 举个例子，如`karma`, 它的`package.json`中有设置：
 
 		"peerDependencies": {
		  "karma-jasmine": "~0.1.0",
		  "karma-requirejs": "~0.2.0",
		  "karma-coffee-preprocessor": "~0.1.0",
		  "karma-html2js-preprocessor": "~0.1.0",
		  "karma-chrome-launcher": "~0.1.0",
		  "karma-firefox-launcher": "~0.1.0",
		  "karma-phantomjs-launcher": "~0.1.0",
		  "karma-script-launcher": "~0.1.0"
		}
		
 这些都是`karma`的相关插件，一般使用`karma`的时候都会需要。
		
* bundledDependencies

 绑定的依赖包,可选字段。发布的时候这些绑定包也会被一同发布。

* optionalDependencies

 可选字段。如果你想在某些依赖即使没有找到，或则安装失败的情况下，npm都继续执行。那么这些依赖适合放在这里。

* engines

 运行环境，可选字段。既可以指定node版本：

		{ "engines" : {"node" : ">=0.10.3 <0.12" } }

 也可以指定npm版本：

		{ "engines" : {"npm" : "~1.0.20" } }

* engineStrick

 可选字段，布尔值。如果强制限制程序运行环境，则设置为true。

* os

 平台，可选字段。指定模块可以在什么操作系统上运行：

		"os" : [ "darwin","linux","!win32" ]
	
 即可以在darwin和linux平台下运行，而不能在win32下。这里设定的取值是来自process.platform的。
	
* cpu

 CPU型号，可选字段。

		"cpu" : [ "x64","ia32" ]
		
 取值来自process.arch。

* preferGlobal

 可选字段，布尔值。如果你的包是个命令行应用程序，需要全局安装（`npm install -g`），就可以设为true。

* private

 可选字段，布尔值。如果private为true，npm会拒绝发布。这可以防止私有repositories不小心被发布出去。

* publishConfig

 发布时使用的配置，尤其方便你希望发布前设定指定的tag或registry。也可以设定其它子字段，但只有tag和registry会影响到发布。可选字段。

 默认值

		"scripts":{"start": "node server.js"}

 如果你的包里有server.js文件，npm默认将执行： node server.js.

		"scripts":{"preinstall":"node-gyp rebuild"}

 如果包里有binding.gyp，npm默认在preinstall命令时，使用node-gyp做编译。
 
### default value 

`npm`会根据包的内容设置一些默认值。

	"scripts": {"start": "node server.js"}

如果包的根目录有`server.js`文件，npm会默认将`start`命令设置为`node server.js`。

	"scripts":{"preinstall": "node-waf clean || true; node-waf configure build"}

如果包的根目录有`wscript`文件，npm会默认将`preinstall`命令用`node-waf`进行编译。

	"scripts":{"preinstall": "node-gyp rebuild"}

如果包的根目录有`binding.gyp`文件，npm会默认将`preinstall`命令用`node-gyp`进行编译。

	"contributors": [...]

如果有`AUTHORS`文件，npm会默认逐行按`Name <email> (url)`格式处理，邮箱和url是可选的。#号和空格开头的行会被忽略。