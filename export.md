[demo](https://codesandbox.io/s/sad-platform-gf0e7?fontsize=14&hidenavigation=1&theme=dark)

#### export let ... 语法

```javascript
export let x = 2;

import { x } from "./a";
```

```javascript
export let x = 2,y = 3,z = 4;
  
import { x, y, z } from "./a";
```

#### export default ...

default 是一个特殊的名字

```javascript
// 导出
export default 2
```

由于export default ...没有显式地约定名字 default 应该按let/const/var的哪一种来创建，因此 JavaScript 缺省将它创建成一个普通的变量（var），但即使是在当前模块环境中，它事实上也是不可写的，因为你无法访问一个命名为“default”的变量——它不是一个合法的标识符。

#### import x ... 本身就是声明语句 

声明的是一个常量。

```javascript
export ...
```

```javascript
import x from "./a"

x=3 // Error: "x" is read-only.
```

#### import 的变量提升

* import 的变量提升是指什么？

`import`具有**提升效果**

```javascript
export let x = 2;
```

```javascript
console.log(x);

import { x } from "./a";
```

输出2，说明在 `console.log(x)`执行之前，该模块就已经存在值为2的变量x了。

* 为什么会有import的变量提升现象？

关键的一点，**为 import 语句中的名字x进行值的绑定**这一步，是 import 模块最先执行的代码。具体来说，ESModule 根据 import 构建依赖树，然后在运行import模块最顶层代码，给import语句中的名字绑定值，就出现了‘变量提升’的效果。

#### 对 export,import 的静态分析（parse）

```javascript
export let x=2
```

向某个名字表中添加一个名字 x，注意 export 不会执行表达式。比如 `export default x` ，表达式 x 的求值只发生在代码执行阶段，不发生在这个对 export 的 parse 阶段。 



注意 `export let x =2` 不属于声明语句。let x=2 应该看作独立的声明语句，有它自己在 parse 阶段的一系列行为。`export let x=2`在export角度来说，意义仅仅是**“向某个名字表中添加一个名字 x”**。



```javascript
import x from './a'
```

1. 在当前模块中声明名字

2. 添加一个当前模块对目标模块的依赖项（从上面说的名字表获取映射关系）。



依据 import 关系构建模块依赖树（与 export 无关）

![dxrcOe.png](https://s1.ax1x.com/2020/09/01/dxrcOe.png)



#### 模块静态装配

从根模块开始，遍历树中所有模块，执行这些模块最顶层的代码（即执行 import 命令，执行对应 export 模块的代码，从而绑定 export 模块中导出变量的值），为 import 的名字绑定值。

---

#### nodejs 模块机制与 ES6 Module 的不同

nodejs中，是在require()函数执行过程中来执行模块的顶层代码的。nodejs模块被封装在一个函数中（亦即是作为一个函数的函数体），由require()在加载完指定模块的文本代码之后，用普通的调用函数的方法调用，从而实现模块装载的。

