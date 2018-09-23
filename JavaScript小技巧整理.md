## 1. call比apply性能更好
> apply 最后还是转化成 call 来执行的，call 要更快毫无疑问，详细介绍见 https://tc39.github.io/ecma262/#sec-function.prototype.apply

**underscore**源码有一段对应的函数优化的技巧：

```
  var optimizeCb = function(func, context, argCount) {
    if (context === void 0) return func;
    switch (argCount) {
      case 1: return function(value) {
        return func.call(context, value);
      };
      // The 2-parameter case has been omitted only because no current consumers
      // made use of it.
      case null:
      case 3: return function(value, index, collection) {
        return func.call(context, value, index, collection);
      };
      case 4: return function(accumulator, value, index, collection) {
        return func.call(context, accumulator, value, index, collection);
      };
    }
    return function() {
      return func.apply(context, arguments);
    };
  };
```
将少量的参数使用call调用，也说明了call比apply性能更快

## 2. Object.create是原型式继承的天然实现
> js原型式继承要求必须有一个对象可以作为另一个对象的基础来实现继承。es5新增的Object.create正是实现继承的一种方式

**Object.create** 内部实现
```
  Object.create = Object.create || function (obj) {
    function F () {}
    F.prototype = obj
    return new F()
  }
```
看demo

```
  const parent = {
    name: 'parent',
    sayName() {
      return this.name
    }
  }
  
  let child = Object.create(parent)
  child.setName = function (name) {
    return this.name = name
  }
  console.log(child)
  /** 
  {
    setName: function () { xxxx },
    __proto__: {
      name: 'parent',
      sayName() { xxxx }
    }
  }
  由此可见使用Object.create实现了继承
  **/
  
  child.setName('child')
  child.sayName() // child
```
我们也可以利用Object.create创建一个没有原型的对象

```
  var o = Object.create(null) 
  // o不具备任何属性和方法
```

## 3. 对象解构赋值

> 使用ES6可以从数组和对象中提取值，对变量进行解构赋值。如果解构不成功，变量的值就等于undefined

数组是一个对象, 所以可以从数组中解构赋值。

```
  Array.prototype.__proto__ === Object.prototype
  // true
```

```
  const [first] = [1, 2, 3, 4] // 取第一个元素
  
  const [_, ...tail] = [1, 2, 3, 4] // 取尾部元素
  
  const [a, b, c] = [1, 2] // 取全部元素
  a // 1
  b // 2
  c // undefined
  
```
我们也可以在解构赋值的同时给不存在的项设置默认参数

```
const [a, b, c = 3] = [1, 2]
  a // 1
  b // 2
  c // 3
```
从对象中解构赋值
```
  let { foo, bar } = { foo: "aaa", bar: "bbb" };
  foo // "aaa"
  bar // "bbb"
  
  let { foo: f, bar: b } = { foo: "aaa", bar: "bbb" };
  f // "aaa"
  b // "bbb"
```
解构赋值设置默认参数
```
  let { foo = 'a', bar = 'b', c = '1' } = { foo: "aaa", bar: "bbb" };
  foo // "aaa"
  bar // "bbb"
  c // '1'
```
解构一个复杂对象
```
  const user = {
    name: '张三',
    address: {
      city: '北京',
      street: '王府井大街'
    },
    messages: [
      {
        id: 1,
        name: '第一条消息'
      },
      {
        id: 2,
        name: '第二条消息'
      }
    ]
  }
  
  const {name, address: {city, street}, messages: [firstMessage]} = user
  name // 张三
  city // 北京
  street // 王府井大街
  firstMessage // {id: 1, name: '第一条消息'}
```

解构赋值也可以用于函数，函数也是一个对象(所有的函数对象是被 Function 这个函数对象构造出来的)

```
  function logArr([first, ...tail], ...otherArgs) {
    console.log(first, tail, otherArgs)
  }
  logArr([1, 2, 3], 4, 5) // 1, [2, 3], [4, 5]
  
  function logFirstMessage({messages: [firstMessage]}) {
    console.log(firstMessage)
  }
  logFirstMessage(user) // {id: 1, name: '第一条消息'}
```

解构赋值用于其他类型

拥有包装对象的类型Number, String, Boolean都可以解构赋值，null和undefined没有包装对象，无法解构赋值

Number类型
```
  const a = 3
  const { constructor } = 3
  constructor === a.constructor // true
```
String类型
```
  const { length } = '123'
  length // 3
  length === '123'.length // true
  
  const [a, b, c, d, e] = 'hello';
  a // "h"
  b // "e"
  c // "l"
  d // "l"
  e // "o"
  
  const {0: first} = '123'
  first // 1
```
Boolean类型
```
  let {toString: s} = true;
  s === Boolean.prototype.toString // true
```
Array类型
```
  const {0: first, length: len} = [1, 2, '']
  first // 1
  len // 3
```

总结： js真是万物皆对象







