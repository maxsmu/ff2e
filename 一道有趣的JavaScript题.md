一道有趣的JavaScript题：

``` js
function Foo() {
    getName = function () { 
        console.log('1');
    };
    return this;
}
Foo.getName = function () {
    console.log('2');
};
Foo.prototype.getName = function () { 
    console.log('3');
};
var getName = function () { 
    console.log('4');
};
function getName() { 
    console.log(5);
}

Foo.getName();  
getName();    
Foo().getName(); 
getName();  
new Foo.getName(); 
new Foo().getName();   
new new Foo().getName();        
```

从中可以了解以下几点：

1. 变量提升
2. JavaScript的操作符优先级 [参考地址MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)


运行结果为：

```
2 4 1 1 2 3 3
```

我们先为代码加上一点注释：

```
// 函数声明
function Foo() {
    // 全局变量
    getName = function () { 
        console.log('1');
    };
    return this;
}
// 为函数添加属性getName
Foo.getName = function () {
    console.log('2');
};
// 为Foo的原型添加方法getName
Foo.prototype.getName = function () { 
    console.log('3');
};
// var 变量声明getName
var getName = function () { 
    console.log('4');
};
function getName() { 
    console.log(5);
}
```

1. `Foo.getName()`: 函数Foo本身并没有执行，执行的是函数的属性getName，故输出：**2**

2. `getName()`: 全局范围内执行了getName()，但是`var`声明的变量和函数声明`function`都会被提升，但是函数声明的提升的级别是比`var`高，故输出：**4**

3. `Foo().getName()`: 由于`()`与`.`优先级相同，所以`Foo().getName()`从左至右执行。首先运行`Foo()`,全局的`getName`被覆盖成输出`console.log('1')`,并且返回的`this`此时代表的是`window`。随后相当于执行的`window.getName()`,那么实际就是输出:**1**

4. `getName()`: 因为上面最后其实执行的就是 `window.getName()`,故输出：**1**

5. `new Foo.getName()` : 想知道这个语句执行后的结果就必须知道它是如何执行的，即执行顺序，这个时候就需要用到操作符优先级了，由于`.`操作符的优先级高于`new`，所以执行`new (Foo.getName)()`,函数执行了new操作，故输出：**2**

6. `new Foo().getName()`: 根据运算符优先级可知，这个最后执行的是 `(new Foo()).getName()`,但是由于`new Foo()`返回的新对象上没有`getName()`，所以在`prototype`上找，故输出：**3**


7. `new new Foo().getName()`: 同样可以转换为`new (new Foo().getName())()`,所以执行`(new Foo()).getName`这个函数是对应的`Foo.prototype.getName`,所以执行`new (Foo.prototype.getName)()`肯定输出的是**3**