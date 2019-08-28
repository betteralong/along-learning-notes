# 闭包和高阶函数
## 1.闭包
### 1.1 什么是闭包
当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使用函数是在当前词法作用域之外执行。  
我们先来看看闭包的简单栗子:
```
function foo() {
  var a = 2
  function bar() {
    console.log(a)
  }
  return bar
}

var baz = foo()
baz()
输出2
```