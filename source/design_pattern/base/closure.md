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
foo函数传递出了一个函数bar，传递出来的bar被赋值给baz，虽然这时baz是在foo作用域外执行的，但baz在调用的时候可以访问到前面的bar函数所在的foo的内部作用域。  
由于bar声明在foo函数内部，bar拥有涵盖foo内部作用域的闭包，使得foo的内部作用域一直存活不被回收。一般来说，函数在执行完后其整个内部作用域都会被销毁，因为js的GC（Garbage Collection）垃圾回收机制会自动回收不再使用的内存空间。但是闭包会阻止某些GC，比如本例中foo()执行完，因为返回的bar函数依然持有其所在作用域的引用，所以其内部作用域不会被回收。  
**注意：如果不是必须使用闭包，那么请尽量避免创建闭包，因为闭包再处理速度和内存消耗方面对性能具有负面影响。**
### 1.1 利用闭包实现结果缓存(备忘录模式)
**备忘录模式**就是应用闭包的特点的一个典型应用。比如有个函数：
```
function add (a) {
  return a + 1 
}
```
多次运行add()时， 每次得到的结果都是重新计算得到的，如果是开销很大的计算比较消耗性能，这里可以对已经计算过的输入做一个缓存。  
所以这里可以利用闭包的特点来实现一个简单的缓存，再函数内部用一个对象存储输入的参数，如果下次再输入相同的参数，那就比较一下对象的属性，如果有缓存就直接把这个值从这个对象里面取出来。
```
/*备忘函数*/
function memorize(fn) {
  var cache = {}
  return function() {
    var ages = Array.prototype.slice.call(arguments)
    var key = JSON.stringfy(args)
    return cache[key] || (cache[key] = fn.apply(fn,args))
  }
}

/*模拟复杂计算*/
function add(a) {
  return a + 1
}

var adder = memorize(add)
add(1) // 输出：2 当前：cache{`[1]: 2`}
add(2) // 输出：2 当前：cache{`[1]: 2`}
add(3) // 输出：3 当前：cache{`[1]: 2`, `[2]:2`}
```
ES6版本：
```
 function memorize(fn) {
  let cache = {}
  return function(...args) {
    const key = JSON.stringfy(args)
    return cache[key] || (cache[key] = fn.apply(fn,args))
  }
 }
/*模拟复杂计算*/
function add(a) {
  return a + 1
}

var adder = memorize(add)
add(1) // 输出：2 当前：cache{`[1]: 2`}
add(2) // 输出：2 当前：cache{`[1]: 2`}
add(3) // 输出：3 当前：cache{`[1]: 2`, `[2]:2`}
```
**稍微解释一下：**
备忘录函数中使用JSON.stringfy把传给adder函数的参数序列化成字符串，把它当作cache的索引，将add函数运行的结果当做索引的值传递给cache，这样adder运行的时候如果之前传递的参数有传入过，那么就返回缓存好的计算结果，不用再计算，如果传递的参数没有计算过，则计算并且缓存fn.apply(fn,args),再返回计算的结果。  
这里只是一个简单的例子， 如果要拿来使用需要适当改进一下，比如：  
1. 缓存不可以永远扩张下去，这样太消耗内存，我们可以只缓存最新传入的n个
2. 在浏览器使用的时候，我们可以借助浏览器的持久化手段来进行缓存  
**注意：cache不可以使用Map 因为Map使用的键是使用===b比较的，因此当传入引用类型值作为键时，实际并不一定相等比如 [1]!==[1]。**

```
//  X 错误示范
function memorize(fn) {        
  const cache = new Map()
  return function(...args) {
    return cache.get(args) || cache.set(args, fn.apply(fn, args)).get(args)
  }
}

function add(a) {
  return a + 1
}

const adder = memorize(add)

adder(1)    // 2    cache: { [ 1 ] => 2 }
adder(1)    // 2    cache: { [ 1 ] => 2, [ 1 ] => 2 }
adder(2)    // 3    cache: { [ 1 ] => 2, [ 1 ] => 2, [ 2 ] => 3 }
```
## 2.高阶函数