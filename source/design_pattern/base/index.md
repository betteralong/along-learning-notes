# this new bind call apply
## 神奇的this指向
刚入门js的时候，对于this这个神奇的东西，它的指向经常能让新手琢磨许久，让我们来看看this的那些事情。  
this是在函数被调用时确定的，它的指向*完全取决于函数调用的地方，而不是它被声明的地方*（这里不包括箭头函数）。当一个函数被调用时，会创建一个执行上下文，它包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息， this就是这个记录的一个属性，它会在函数执行的过程中被用到。  
this在函数的指向有以下几种场景：  

1. 作为构造函数被*new*调用
2. 作为对象的方法使用
3. 作为函数直接调用
4. 被*call*、*apply*、*bind*调用
5. 箭头函数中的this  
  接下来让我们来看看这个new
### 1.1 new绑定
函数如果作为构造函数使用new调用时,this绑定的时新创建的构造函数的实例。  

```
  function Foo() {
    console.log(this)
  }

  var bar = new Foo()
  输出：Foo实例，this就是bar
```
实际上使用new调用构造函数时，会依次执行行下面的操作：   
1. 创建一个新的对象
2. 构造函数的*prototype*被赋值给这个新对象的__prote__
3. 将新对象赋给当前的*this*
4. 执行构造函数
5. 如果函数没有返回其他对象,那么*new*表达式中的函数调用会自动返回这个新对象，如果返回的不是对象将被忽略

### 1.2 显示绑定
通过call、apply、bind可以修改函数绑定的*this*,使其成为我们指定的对象。通过以上方法的第一个参数我们可以显示绑定*this*.  
```
function foo(name, price) {
  this.name = name
  this.price = price
}

function Food(category, name, price) {
  foo.call(this, name, price)
  // foo.apply(this, [name, price])
  this.category = category
}

new Food('食品'， '鸡腿'， '5块钱')
输出： {name:'鸡腿', 'price': '汉堡', 'category': '食品'}
```
call和apply的区别是call接受的是列表参数，而apply接受的是数组。
```
func.call(thisArg, arg1, arg2, ...)
func.apply(thisArg, [arg1, arg2])
```
而bind方法是设置this为给定的值，并返回一个新的函数，且在调用新函数时，将给定参数列表作为原来函数的参数.  
例如:
```
 var food = {
   name: '汉堡',
   price: '5块钱',
   getPrice: function(place) {
     console.log(place + this.price)
   }
 }

 food.getPrice('KFC')
 // 输出 "KFC 五块钱"

 var getPrice1 = food.getPrice.bind({name: '鸡腿', price: '7块钱'}, '肯德基')
 getPrice1()
 // 输出肯德基 七块钱
```
关于bind的原理， 我们可以用apply方式实现一个bing：
```
// ES5方式
Function.prototype.bind = Function.prototype.bind || function() {
  var self = this
  var rest1 = Array.prototype.slice.call(argmengts)
  var context = rest1.shift()
  return function() {
    var rest2 = Arry.prototype.slice.call(arguments)
    return self.apply(contenxt, rest1.concat(res2))
  }
}

//ES6f方式
Function.prototype.bind = Function.prototype.bind || function(...rest1) {
  const self = this
  const context = rest1.shift()
  return function(...rest2) {
    return self.apply(content, [...rest1, ...rest2])
  }
}
```
### 1.3 隐式绑定
函数是否在某个上下文对象中调用，如果是的话*this*绑定的是那个上下文对象。
```
 var a = 'hello'
 var obj = {
   a: 'world',
   foo: function() {
     console.log(this.a)
   }
 }
 obj.foo()
 输出 world
```
如果存在多层嵌套那么指向最后一个调用这个方法的对象
```
var a = 'hello'
var obj = {
  a: 'world',
  b: {
    a: 'China',
    foo: function() {
      console.log(this.a)
    }
  }
}

obj.b.foo()
输出China
```
### 1.4 默认绑定
函数独立调用，直接使用不带任何修饰的函数引用进行调用，也是上面几种绑定途径之外的方式。非严格模式下this绑定到全局对象(浏览器下是window， node是global),严格模式下this绑定到undefined(因为严格模式下不允许this指向全局对象)。
```
var a = 'hello'
function foo() {
  var a = 'world'
  sonsole.log(this.a)
  console.log(this)
}
// 浏览器下输出 world window对象
```

注意有一种情况:
```
var a = 'hello'
var obj = {
    a: 'world',
    foo: function() {
        console.log(this.a)
    }
}
var bar = obj.foo
bar()
浏览器中输出: "hello"
```
## this绑定的优先级
this 存在多个使用场景，那么多个场景同时出现的时候，this到底该如何指向呢。这里存在一个优先级的概念，this根据 优先级来确定指向。优先级: new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定  
所以this的判断顺序：
1. new 绑定： 函数是否存在new 中调用？ 如果是的话this指向的是新的创建对象
2. 显示绑定: 函数是否通过bind、apply、call调用 ? 如果是的话，this绑定的是指定的对象
3. 隐式绑定: 函数是否存在某个上下文调用？ 如果是的话，this绑定的是那个上下文对象
4. 如果都不是的话，使用默认绑定。在严格模式下绑定到undefined，否则绑定到全局对象

## 箭头函数中的this
箭头函数是根据其声明的地方来决定this的，它是ES6中新出的知识点。
箭头函数的this绑定是无法通过call、apply、bind被修改的，因为箭头函数没有构造函数constructor，所以也不可以使用new调用，即不能作为构造函数，否则会报错

___
下一节  [闭包和高阶函数](https://github.com/betteralong/along-learning-notes/blob/master/source/design_pattern/base/closure.md)
