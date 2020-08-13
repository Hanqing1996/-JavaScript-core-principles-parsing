> 周爱民老师的《 JavaScript核心原理解析》总结


## 声明
#### 标识符
```
let x
```
x 就是一个标识符

#### 变量声明在引擎的处理
> 变量声明在引擎的处理上被分成两个部分：一部分是静态的、基于标识符的词法分析和管理，我们称之为<strong>声明标识符</strong>，它总是在相应上下文的环境构建时作为名字创建的，我们称之为<strong>创建标识符</strong>，注意；另一部分是表达式执行过程，是对上述名字的赋值，这个过程也称为绑定。
1. JavaScript 将可以通过“静态”语法分析（Parser 的工作）发现那些声明的标识符；
2. 标识符对应的变量 / 常量“一定”会在代码执行前就已经被创建在作用域中。

#### var 的初始化
```
var y = "outer";
function f() {
  console.log(y); // undefined
  console.log(x); // throw a Exception
  let x = 100;
  var y = 100;
}
```
> 正是由于var y所声明的那个标识符在函数 f() 创建（它自己的闭包）时就已经存在，所以才阻止了console.log(y)访问全局环境中的y。类似的，let x所声明的那个x其实也已经存在 f() 函数的上下文环境中。访问它之所以会抛出异常（Exception），不是因为它不存在，而是因为<strong>这个标识符被拒绝访问了</strong>。

> 在 ECMAScript 6 之后出现的let/const变量在“声明（和创建）一个标识符”这件事上，与var并没有什么不同，只是 JavaScript 拒绝访问还没有绑定值的let/const标识符而已。

> 回到 ECMAScript 6 之前：JavaScript 是允许访问还没有绑定值的var所声明的标识符的。这种标识符后来统一约定称为“变量声明（varDelcs）”，而“let/const”则称为“词法声明（lexicalDecls）”。>

> 函数的上下文环境在创建一个“变量名”后，会为它初始化绑定一个 undefined 值，而”词法名字（lexicalNames）”在创建之后就没有这项待遇，所以它们在缺省情况下就是“还没有绑定值”的标识符。
---

#### 补充
> 对[我用了两个月的时间才理解 let](https://fangyinghang.com/let-in-js/)的总结
#### var 声明的「创建、初始化和赋值」过程
```
function fn(){
  var x = 1
  var y = 2
}
fn()
```
在执行 fn 时，会有以下过程（不完全）：
1. 进入 fn，为 fn 创建一个环境。
2. 找到 fn 中所有用 var 声明的变量，在这个环境中「创建」这些变量（即 x 和 y）。
3. 将这些变量「初始化」为 undefined。
4. 开始执行代码
5. x = 1 将 x 变量「赋值」为 1
6. y = 2 将 y 变量「赋值」为 2
#### function 声明的「创建、初始化和赋值」过程
```
fn2()
function fn2(){
  console.log(2)
}
```
JS 引擎会有以下过程：

 1. 找到所有用 function 声明的变量，在环境中「创建」这些变量。
 2. 将这些变量「初始化」并「赋值」为 function(){ console.log(2) }。
 3. 开始执行代码 fn2()
  
也就是说 function 声明会在代码执行之前就「创建、初始化并赋值」。
#### let 声明的「创建、初始化和赋值」过程
```
{
  let x = 1
  x = 2
}
```
我们只看 {} 里面的过程：

1. 找到所有用 let 声明的变量，在环境中「创建」这些变量
2. 开始执行代码（注意现在还没有初始化）
3. 执行 x = 1，将 x 「初始化」为 1（这并不是一次赋值，如果代码是 let x，就将 x 初始化为 undefined）
4. 执行 x = 2，对 x 进行「赋值」

#### 总结
* let 的「创建」过程被提升了，但是初始化没有提升。
* var 的「创建」和「初始化」都被提升了。
* function 的「创建」「初始化」和「赋值」都被提升了。

---
## 赋值
> 如果是在一门其它的（例如编译型的）语言中，“为变量 x 绑定一个初值”就可能实现为“在创建环境时将变量 x 指向一个特定的初始值”。这通常是静态语言的处理方法，然而，如前面说过的，JavaScript 是门动态的语言，所以它的“绑定初值”的行为是通过动态的执行过程来实现的，也就是赋值操作。


