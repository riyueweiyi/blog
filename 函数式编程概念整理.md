学习函数式编程之前先理解几个有关的概念

### 1.高阶函数
> 参数或返回值为函数的函数

详见代码：
```
  const hoc = function (func) { // 参数为函数
    return func('hello')
  }
  
  hoc(function (text) {
    return text + 'world!'
  })
  
  //=> hello world!
  
  const sayHelloWorld = function () { // 返回值为函数
    return function () {
      return 'hello world!'
    }
  }
  
  sayHelloWorld()() 
  
  //=> hello world!
```

### 2.纯函数
> 相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用

详见代码:

例1:
```
  var xs = [1, 2, 3, 4, 5]
  xs.slice(0, 3) // 纯的
  //=> [1, 2, 3]
  
  xs.slice(0, 3) // 纯的
  //=> [1, 2, 3]
  
  xs.splice(0, 3) // 不纯，改变了原数组
  //=> [1, 2, 3]
  
  xs.splice(0, 3) // 不纯, 相同的输入没有得到相同的输出
  //=> [4, 5] 
```
例2:
```
  // 不纯，依赖外部变量
  var minimum = 21;
  
  var checkAge = function (age) {
    return age >= minimum
  }
```

### 3.curry化
> 只传递给函数一部分参数来调用它，让他返回一个函数去处理剩下的参数。curry化利用闭包特性缓存参数

详见代码
```
  // 实现curry函数
  var curry = function (fn, ...args) {
    return (...innerArgs) => fn.apply(null, [...args, ...innerArgs])
  }

  var add = function (a, b) {
	  return a + b 
  }
  
  var increment = curry(add, 1)
  increment(2) // 闭包记住了上一次调用的参数
  // 3
  
```
我们可以使用lodash的curry函数来使这类函数的定义和调用更容易

```
  var curry = _.curry
  
  var match = curry(function (what, str) {
    return str.match(what)
  })
  
  match(/\s+/g, 'hello world')
  // [' ']
  
  match(/\s+/g)('hello world')
  // [' ']
  
  var hasSpaces = match(/\s+/g)
  
  hasSpaces('hello world')
  // [' ']
  
  hasSpaces('spaceless')
  // null

```
上述代码遵循一种简单，同时也是非常重要的模式。即策略性的把要操作的数据放到最后一个参数里。

这里表明的是一种”预加载“函数的能力，通过传递一到两个参数调用函数，就能得到一个记住了这些参数的新函数。只传给函数一部分参数通常也叫做局部调用，能够大量减少样板式代码。

通常我们不定义直接操作数据的函数，因为只需要内联调用`map(fn)`就能达到目地。这一点同样适用于`sort`, `filter` 以及其他高阶函数。

### 4. pointfree模式

pointfree模式指的是函数无须提及将要操作的数据是什么样的。即不需要命名一些准瞬即逝的中间变量

```
  // 非 pointfree, 因为提到了数据： word
  var snakeCase = function (word) {
    return word.toLowerCase().replace(/\s+/g, '_')
  }
  
  // pointfree
  var snakeCase = compose(replace(/\s+/g, '_'), toLowerCase)
```
### 5. functor

> functor是实现 `map` 函数并遵守一些特定规则的容器类型

详见代码:
```
  var Container = function (x) { 
    return this.__value = x
  }
  
  Container.of = function (x) {
    return new Container(x)
  }
  
  Container.prototype.map = function (f) {
    return Container.of(f(this.__value))
  }
  
  Container.of(3).map(function (three) {
    return three + 3
  })
  // => Container(6)

```

### 6. Maybe functor
> Maybe会先检查自己的值是否为空然后才参与运算

```
  var Maybe = function (x) {
    this.__value = x
  }
  
  Maybe.of = function (x) {
    return new Maybe(x)
  }
  
  Maybe.prototype.isNothing = function () {
    return this.__value === null || this.__value === void 0
  }
  
  Maybe.prototype.map = function (f) {
    return this.isNothing ? Maybe.of(null) :   Maybe.of(f(this__value))
  } // 检查其值是否为空然后参与运算
  
  Maybe.of(null).map(match(/a/ig))
  // => Maybe(null)
  
  Maybe.of({name: 'Boris'}).map(_.prop('age')).map(add(10))
  // => Maybe(null)
  
  Maybe.of({name: 'Dinah', age: 14}).map(_.prop('age')).map(add(10))
  // => Maybe(24)
```

### 7. lift
> 一个非functor函数转换成一个functor函数，我们把这个过程叫lift

### 8. monad
> monad是可以flatten的pointed functor，一个functor，只要它定义了一个join方法和一个of方法，并遵守一些定律，那么他就是monad





