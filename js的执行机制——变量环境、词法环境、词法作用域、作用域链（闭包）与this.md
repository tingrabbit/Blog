写在前头：该文章整理自[极客时间-浏览器工作原理与实践-浏览器中的Js执行机制0](https://time.geekbang.org/column/article/119046)7-12篇幅，致敬一下这几篇文章的作者，写的很好。

# 变量环境与词法环境

关于变量，我们知道，在js执行前，在编译器中会对js代码进行预编译，在编译阶段，会对代码中的变量进行变量提升。

关于变量提升，有几点以前没有注意到的知识点
1. **函数表达式**和**函数定义**的变量提升是不一样的，对于**函数表达式**`var showName = function(){};`，实际上变量提升的部分是：`var showName = undefind`，对于**函数定义**`function showName(){};`,变量提升的部分是整个函数体。
2. 对于同名的变量和函数名，在后的函数声明会覆盖在前的变量声明，而在后的变量不会覆盖在前的函数声明。
3. 之前一直搞不懂执行上下文中变量环境和词法环境有什么不同，这里给出了答案，假设一个函数执行上下文，函数体内用`var`声明的变量和函数，存在于变量环境中。而`let`和`const`声明的变量存在于词法环境中。
4. 关于函数词法作用域还有一个细节，在函数体内，当执行另一个块级作用域，如果有通过`let`或者`const`声明了变量，那么这些变量会作为一个单独的词法作用域就会被压栈到词法作用域中，当执行完这个块级作用域时，对应的词法作用域将会推出，以维护块级作用域的作用，辅助理解如下图

```

function foo(){
    var a = 1
    let b = 2
    {
      let b = 3
      var c = 4
      let d = 5
      console.log(a)
      console.log(b)
    }
    console.log(b) 
    console.log(c)
    console.log(d)
}   
foo()
```


![image](https://static001.geekbang.org/resource/image/f9/67/f9f67f2f53437218baef9dc724bd4c67.png)



> 当执行到`foo`函数体内时，编译并创建的函数上下文。此时`foo`函数中的大括号内的块级作用域中的`var c = 4`进行了变量提升，放置在变量环境中。



![image](https://static001.geekbang.org/resource/image/7e/fa/7e0f7bc362e0dea21d27dc5fb08d06fa.png)






> 继续执行到`foo`函数中的块级作用域内，`let`和`const`定义的变量单独存放并压入栈中








![image](https://static001.geekbang.org/resource/image/06/08/06c06a756632acb12aa97b3be57bb908.png)
> 例子：其中一个变量查找









![image](https://static001.geekbang.org/resource/image/d4/28/d4f99640d62feba4202aa072f6369d28.png)
> 当执行完`foo`函数内的块级作用域的代码时，对应的词法环境块将被推出









通过上面的压栈和出栈，我们不仅要学会词法环境是如何维护一个词法作用域的，也应该懂栈是一种非常重要的数据结构，不光应用在 `JavaScript` 语言中，其他的编程语言，如 `C/C++、Java、Python` 等语言，在执行过程中也使用了栈来管理函数之间的调用关系。

# 词法作用域与作用域链
词法作用域就是词在声明时所处的作用域，也就是说作用域是由声明的位置来决定的，所以词法作用域是静态的，跟函数调用时的作用域链的动态不一样。

**函数调用栈**是在执行时动态生成的，是为了管理函数之间的调用关系，知道程序执行到哪一步，要进什么函数，要出什么函数

**词法作用域**是在代码阶段而不是执行时就决定好的，是为了能够在执行时顺着词法作用域查找变量


```

function bar() {
    console.log(myName)
}
function foo() {
    var myName = "极客邦"
    bar()
}
var myName = "极客时间"
foo()
```
![image](https://static001.geekbang.org/resource/image/20/a7/20a832656434264db47c93e657e346a7.png)





> 如上图，`bar`函数的声明位置在全局，所以作用域链中`bar`函数指向的下一个词法作用域就是全局执行上文文中的词法作用域。而调用栈还是那个调用栈，两者没有联系。


![image](https://static001.geekbang.org/resource/image/25/a7/25053af5ae30c8be991fa14631cde0a7.png)





**作用域链**就是上图中，查找变量时那条红色虚线的链条，再次强调，红色虚线指向的词法作用域是静态的，是根据声明时所处的位置决定的。

### 闭包
理解了词法作用域链，那么网络上传的闭包（各式各样的定义）也跟词法作用域链有关。


```
function foo() {
    var myName = "极客时间"
    let test1 = 1
    const test2 = 2
    var innerBar = {
        getName:function(){
            console.log(test1)
            return myName
        },
        setName:function(newName){
            myName = newName
        }
    }
    return innerBar
}
var bar = foo()
bar.setName("极客邦")
bar.getName()
console.log(bar.getName())
```

我们知道块级作用域可以访问它们外部的变量，如果这个块级作用域是一个函数`innerBar.getName/innerBar.setName`，那么这个内部函数总是可以访问定义他的函数`foo`内的变量。

那如果这个内部函数作为`return`进行了返回给了参数，那么这个参数就是一个函数，当这个参数执行的时候，也可以访问到引用的外部函数的变量。

这个返回的内部函数就是一个闭包，当然关于闭包的定义千千万，最重要还是掌握到一点知识：内部函数可以访问外部函数的变量，即使外部函数已经执行完了，但是内部函数要访问的外部变量依然保存在内存中，当调用内部函数时，这些变量将可以使用。

还有两点有用的小知识
- 在使用`chrom`调试工具查看函数调用栈的情况时，`Closure(foo)` 就是指 `foo` 函数的闭包
- 闭包可以访问的变量存在于内存中，如果不进行及时的回收，会造成内存泄漏
- 可以通过把引用的参数`bar`及时置为`null`（我猜的），或者把`bar`变成一个局部变量，当使用这个局部变量的函数销毁后，闭包将会回收

# this
`this`是为了解决在对象内部的方法中使用对象内部的属性而出现的，我们知道词法作用域是根据定义的位置生成的，所以在对象中的方法想要读取对象中属性是无法读取的，但是通过`this`，我们可以读取对象内部的属性。

```

var bar = {
    myName:"time.geekbang.com",
    printName: function () {
        console.log(this.myName) //time.geekbang.com
        console.log(myName)  //极客
    }    
}
var myName="极客";
bar();
```

执行上下文包括变量环境、词法环境、`outer`（作用域链中指向下一个作用域）以及`this`
![imgae](https://static001.geekbang.org/resource/image/b3/8d/b398610fd8060b381d33afc9b86f988d.png)
`this`与执行上下文有关，而执行上下文包括全局执行上下文，函数执行上下文和`eval`中的执行上下文。而eval用得比较少，我们看看另外两个。

- 全局对象中的 this 是指向 window 对象
- 默认情况下调用一个函数，其执行上下文中的 `this` 也是指向 `window` 对象的
- 在严格模式下，默认执行一个函数，其函数的执行上下文中的 `this` 值是 `undefined`


改变`this`的几种方法
- 通过函数的 `call、bind` 和 `apply` 方法设置
- 通过对象调用方法设置
- 通过构造函数中设置

##### 通过构造函数中设置

```
function CreateObj(){
  this.name = "极客时间"
}
var myObj = new CreateObj()
```
相当于

```
  var tempObj = {}
  CreateObj.call(tempObj)
  return tempObj
```

`this`的设计缺陷
1. 嵌套函数中的 `this` 不会从外层函数中继承

```
var myObj = {
  name : "极客时间", 
  showThis: function(){
    console.log(this) // this: myObj
    function bar(){console.log(this)} // this: window对象
    bar()
  }
}
myObj.showThis()
```

2. 普通函数中的 `this` 默认指向全局对象 `window`,这样会打破数据的边界，造成一些误操作

处理`this`的设计缺陷，除了上文改变`this`的几种方法之外，还有以下几种
- 使用箭头函数，箭头函数没有自己的执行上下文，箭头函数中的 this 取决于它的外部函数
- 使用变量缓存外部的`this`
- 使用立即试行函数`IIFC`，传入外部的`this`值

