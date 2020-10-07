**SyntaxError 不一定在静态语法分析期抛出，也可能在运行期抛出**

```javascript
function f(x=1) {
  let x = 100;
}
```

不同的引擎，表现不同

* 在 chrome 环境中，上述代码会报错：SyntaxError: Identifier 'x' has already been declared。说明在函数`f`的`parse`阶段，就抛出了错误。
* 在 Node.js 环境中，上述代码不会报错，但是如果调用函数 `f`，会报错。说明在函数的运行阶段，才抛出了错误。

```javascript
function f(x=1) {
  let x = 100;
}

f(10)
// Identifier 'x' has already been declared
```

