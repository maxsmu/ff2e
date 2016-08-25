一. 规则

1. 基本设置

  1.1. 一个tab缩进,一个Tab = 4个空格

2. 空格与格式

  2.1. 在!前面加空格,后面不加空格`(!default,!glabal,!important,!optional)`
  
    	//bad
    	color: #000!important;
    	color: #000 ! important;
    	
    	//good
    	color: #000 !important;

  2.2 选择器与大括号之间有一个空格
  
	     //bad
	       .box{
	         ...
	       }
	       
	     //good
	       .box {
	         ...
	       }
	       
  2.3. 变量名属性名冒号前不加空格,冒号后加一个空格
    
	    //bad
	      $color:#0f0;
	      color:#0f0;
	    //good
	      $color: #0f0;
	      color: #0f0;
	      
  2.4. 运算符前后需要一个空格,运算符包括+, -, *, /, %, ==, !=, >, >=, <, <=
  
	    //bad
	      margin: 5px+5px;
	    //good
	      margin: 5px + 5px;
	      
  2.5. 括号不需要padding
  
	    //bad
	      color: rgba( 0, 0, 0, .1 );
	    //good
	      color: rgba(0, 0, 0, .1);
  
  2.6. 每个属性占据新的一行,多选择器时,每个选择器占据新的一行
    
	    //bad
	      .box { margin: 0; padding: 0; }
	      .box, .box2, .box3 {
	        ...
	      }
	    //good
	      .box {
	        margin: 0;
	        padding: 0;
	      }
	      .box,
	      .box2,
	      .box3 {
	        ...
	      }
  
  2.7. 选择器嵌套不超过3层
    
	    //bad
	      .one .two .three .four {
	        ...
	      }
	    //good
	      .one .two .three {
	        ...
	      }
	      
  2.8.嵌套不超过4层
  
	    //bad
	    .one {
	      .two {
	        .three {
	          .four {
	            .five {
	              ...
	            }
	          }
	        }
	      }
	    }
  
  2.9. 文件最后需要新的空的一行

  2.10. @else写在同一行
    
	      //bad
	        @if {
	          ...
	        }
	        @else {
	          ...
	        }
	      //good
	        @if {
	            ...
	        } @else {
	            ...
	        }
3. 书写规范
  
  3.1. 一条规则里禁止重复的属性
    
	    //bad
	      .box {
	        margin: 10px;
	        margin: 0;
	      }
  
  3.2. 禁止空规则
 
	    //bad
	      .box {}
 
  3.3. import路径不带下划线也不带后缀名
    
	    //bad
	      @import "foo/_bar.scss";
	      @import "_bar.scss";
	      @import "_bar";
	      @import "bar.scss";
	    //good
	      @import "foo/bar";
	      @import "bar";
  
  3.4. 字符串单引号
    
	    //bad
	      content: "hello";
	    //good
	      content: 'hello';

  3.5. 省略不必要的&
    
	    //bad
	      .parent {
	        & .son {
	          ...
	        }
	      }
	    //good
	      .parent {
	        .son {
	          ...
	        }
	      }
	      
  3.6. 省略URL前面的协议和域名
    
	    //bad
	      background: url('https://example.com/assets/image.png');
	    //good
	      background: url('assets/image.png');

  3.7. URL要以单引号包裹起来
   
	    //bad
	      background: url(example.png);
	    //good
	      background: url('example.png');

  3.8. 0值后面不带单位
    
	    //bad
	      margin: 0px;
	    //good
	      margin: 0;

  3.9. 属性值后面加分号
    
	    //bad
	      color: #fff
	      color: #fff ;
	    //good
	      color: #fff;

  3.10. 省略不必要的小数
  
	    //bad
	      margin: 1.0em;
	    //good
	      margin: 1em;
	      
  3.11. 省略小数点前面的0
   
	    //bad
	      margin: 0.5em;
	    //good
	      margin: .5em;

4. 命名规范

  4.1. 使用十六进制代替纯颜色,十六进制颜色值可以缩写时尽量使用缩写形式,属性值小写并保证是正确的
   
	    //bad
	      color: green;
	      color: #00ff00;
	      color: #00FF00;
	      color: #00ff0;
	    //good
	      color: #0f0;
	      
  4.2. `functions,mixins,variables,placeholdes`的命名使用小写字母和连字符,禁止出现大写
    
	    //bad
	      $mySize: 12px;
	      .box {
	        font-size: $mySize;
	      }
	    //good
	      $my-size: 12px;
	      .box {
	        font-size: $my-size;
	      }
5. 通用
  
  5.1. 使用`border:0`代替`border:none`

  5.2. 使用//注释代替/* ... */注释

  5.3. 禁止使用ID选择器

  5.4. 不建议使用`!important`

  5.5. 伪元素和伪类冒号表达适当
  
     伪元素 `::before ::after ::first-letter ::first-line`
     伪类 `:nth-child()`等等

  5.6. 不推荐标签选择器
  
	    //bad
	      ul.list {}
	      ul li.list {}
	    //good
	      .list {}
	      ul .list {}

  5.7. 尽量合并属性值
    
	    //bad
	      margin: 1px 1px 1px 1px;
	    //good
	      margin: 1px;

  5.8. 不写浏览器前缀,浏览器前缀用工具生成
   
	    //bad
	      @-webkit-keyframes fade {
	        0% { opacity: 0; }
	      }
	      ::-webkit-placeholder {
	        color: #0f0;
	      }
	      .box {
	        -webkit-transition: none;
	      }
	    //good
	      @keyframes fade {
	        0% { opacity: 0; }
	      }
	      ::placeholder {
	        color: #0f0;
	      }
	      .box {
	        transition: none;
	      }

二. 工具及配置

1. 使用工具
  
  1.1. [scss-lint Github](https://github.com/brigade/scss-lint)

2. 配置文件
  [配置文件下载](./assets/scss-lint.yml)

3. 安装方法

  因为我自己用的webstorm,所以就拿这个举例,比较简洁,有兴趣的同学可以研究下文档
  
  3.1. 首先确保你的`ruby2.0.0+`,`sass 3.4.20+`
  
  3.2. `gem install scss_lint`
  
  3.3. 在plugin中搜索`scss-lint`,下载安装
  
  3.4. 在左侧othersettings中搜索安装好的scss-lint,然后如下图进行配置