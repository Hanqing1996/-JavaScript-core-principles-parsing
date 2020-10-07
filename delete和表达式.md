#### 表达式是不会在 parse 阶段执行（即求值）的。表达式的计算，一定只发生在代码执行期间。



#### 表达式和声明语句的区别

* 这个是声明语句

```
var x=2
```

* 这个是表达式

```
x=2
```

---

#### 表达式(unarrExpression)的结果

表达式的结果有两种可能的类型

* 值类型/非引用类型

```javascript
// 0 在这里被视为一个单值表达式，这个单值表达式的结果是值类型
delete 0
```

* 引用类型

> 注意这里的“引用”，是指在语法层面上表达的“它是对某种语法元素的引用”，是ECMAScript 规范的一部分。与数据类型上的值类型（Number,String,Boolean）,引用类型（Object）无关。

```javascript
// x 这个单值表达式的结果是引用类型
delete x
```

---

#### 表达式的结果是如何确定下来的？

> 表达式的结果不由这个表达式自身决定，而是由这个表达式的所处位置决定

* <code>LHS</code> 和 <code>RHS</code>

LHS 和 RHS 是两种查找类型。如果一个表达式在**赋值操作符**（typeof、delete 不是赋值操作符）的左边，我们就对它进行 LHS 查询。如果一个表达式在操作符的右边，我们就对它进行 RHS 查询。

```javascript
y = x
```

所有赋值操作的含义，是将右边的“值”，赋给左边用于包含该值的“引用”。那么上面的y=x，其实就是被翻译成：

```javascript
x = GetValue(x)
```

其中，GetValue()是从一个引用中取出值来的行为。由 JS 引擎隐式执行，不暴露。



此外，<code>let a=2;console.log(a)</code> <code>function fn(a){};fn(2)</code> 也会隐式调用 GetValue



**综上，表达式的结果（Result）是一个“未确定引用/值”的东西，如果它用作lhs，就是引用(ref)；如果它用作rhs，那么js会使用GetValue(ref）来得到它的值。**

---

####  一个赋值是如何完成的

在这个问题上，你最好能看一下ECMAScript中“引用（规范类型）”的定义，在这里：https://tc39.es/ecma262/#sec-reference-specification-type



这个规范的这一部分内容，解释了所谓引用，实际上是一个拥有“base和referencedName”等字段的记录。而对于我们这里讨论的“变量x”来说，base就是它的环境上下文，referencedName就是'x'这个名字——注意这个记录（或称之为C语言中的结构）中根本没有“值是多少”这样的东西。



所以，对于“赋值（x = x）”这个行为来说，如果是一个引用(ref)，那么它作为引用就可以理解为一个存储的位置，即“base[referencedName]”——注意这只是说明“该名字x在base环境中的位置”；作为右侧的时候，就是GetValue(ref)，这样GetValue()操作就是去取ref指向的值，因为ref作为结构体，本身并没有包含它指向的值。

---

#### a.x

这是一个表达式。它有一个左操作数，一个点运算符，一个右操作数

```javascript
// 在这里，a.x 这个表达式的 result 是引用类型
a.x=2

//注意，x这个本来不存在的属性仅在第二次赋值操作的时候才会被创建！
```

```javascript
// 在这里，a.x 这个表达式的 result 是值类型 
b=a.x
```

---

#### delete 是在删除什么？

delete 这个操作的正式语法设计并不是“删除某个东西”，而是“删除一个表达式的结果”。准确的说，这个表达式必须是 obj.x 这样某个对象的属性。

```javascript
delete obj.x
```

而对于

```javascript
delete 0 // true  JavaScript 认为所有删除值的 delete 就直接返回 true
```

尽管不报错（由于某些历史包袱，出于向下兼容的需要），但这样的用法是背离 delete 的设计初衷的。

---

#### 删除全局 x 的不同结果

* 用 var 声明的全局变量

> 所有使用`var x`来声明的全局变量`x`的configurable为 false，所以不能删除，且它放在varNames中的名字`x`也不能被移除。

```javascript
var x=2
delete x // false
```

* 通过变量名泄露来在全局创建名字x

> 直接使用 x = 2 通过变量名泄露来在全局创建名字x，那么这个x不会出现在varNames中，且它作为global上的属性时的configurable为true。所以也是可以被移除掉的。

```javascript
x=2
delete x // true
```



上述的 configurable 等信息可以通过 getOwnPropertyDescriptor 查询

```javascript
x=2
Object.getOwnPropertyDescriptor(window,x) 
// {value: Window, writable: false, enumerable: true, configurable: true}
```

---

#### JavaScript 是不支持静态类型检查的

JavaScript的语法解析中，<code>delete 0 </code>的 0 是作为一个标识符来理解的，而不是作为一个字面量值。因为如果要作为字面量值，那么就得对每一个解析出来的标识符做类型检查（类型检查不单单是指整型、字符串这种，还包括常量或是变量这样的可写性）。而JavaScript的语言设计和引擎设计又决定了它是不支持静态类型检查的。

---

#### 字符串是值类型，但按引用方式传递和赋值

在JavaScript中，显然数字2，与字符串"abc"都是值类型。但是，无论是在JavaScript还是在其它语言中，通常这两种值的处理方式都有着区别，就是所谓的“字符串是值类型，但按引用方式传递和赋值”。通常大家都是如此。比如一个引用传递的过程：

```javascript
a = 'abc'
b = a
c = b
```

如果第二、三行代码都是每个字符都复制给b、c的话，那么代价就特别高（例如字符串可能1K字节长、或者1T字节长），所以它们在赋值中就只是传递了a的一个引用。——并且，事实上，在第一行赋值时，a也只是拿到了字符串字面量（值）的一个引用而已。

---

#### delete null 和 delete undefined

```
delete undefined // false
delete null // true
```

虽然现在的引擎上面undefined看起来长得跟null值差不多，而且在ECMAScript规范中它们都还是平级的（是原始值），而且它们的作用也很接近，最后他们都还是从最初的JavaScript 1.x中就存在的概念，但是undefined/null两者却在实现上完全不同：**undefined是一个全局属性，而null是一个关键字。**

由于undefined是全局属性，所以`delete undefined`其实就是`delete global.undefined`，是删除引用，而不是删除值。**而这个属性是只读的，所以就返回false了**。

