## ECMAScript新特性

### ECMAScript与JavaScript

前言: ECMAScript（ES）本身就是一门脚本语言，通常看作JavaScript(JS)的标准化规范，实际上JS是ES的扩展语言。

浏览器环境下 JS = ES + Web API(BOM + DOM)

![image](https://user-images.githubusercontent.com/37037802/126583500-895bf3d5-cb20-41a2-bc20-96bafab88e9c.png)

node环境下 JS = ES + Node APIs(fs+net+etc.)

![image](https://user-images.githubusercontent.com/37037802/126583872-8b7e1f28-66e5-46db-9084-c41d229fc4d2.png)

ECMAScript的发展经历

![image](https://user-images.githubusercontent.com/37037802/126586607-b90db8fe-3984-4f31-8e2f-cf191e7442b0.png)

### ECMAScript2015（ES2015）

ps:我们现在所说的ES6准确来说是指ES2015以上的版本

• let 和 const

• Arrow functions

• Classes（类）

• Promises

• Generators

• Modules

• Template literals（模板字⾯量）

• Default parameters（默认参数）

• Enhanced object literals（对象字⾯量增强）

• Destructuring assignments（解构分配）

• Spread operator（展开操作符）

• for ... of 循环

• Map 和 Set

• Proxy

  ......
  
  这里只介绍常见的几个新特性
  
  #### Symbol类型
  
  ES6 引入了一种新的原始数据类型 Symbol ，表示独一无二的值，最大的用法是用来定义对象的唯一属性名
  
  Symbol 函数栈不能用 new 命令，因为 Symbol 是原始数据类型，不是对象。可以接受一个字符串作为参数，为新创建的 Symbol 提供描述，用来显示在控制台或者作为字符串的时候使用，便于区分。
  
  ```
  let sy = Symbol("KK");
  console.log(sy);   // Symbol(KK)
  typeof(sy);        // "symbol"
 
  // 两个 Symbol 永远不会相等
  let sy1 = Symbol("kk"); 
  sy === sy1;       // false
  ```
  
  应用场景1：Symbol 模拟实现私有成员
  
  ```
  // a.js ======================================
  
  const name = Symbol()
  const person = {
    [name]: 'zce',
    say () {
      console.log(this[name])
    }
  }
  
// b.js =======================================  
// 由于无法创建出一样的 Symbol 值，
// 所以无法直接访问到 person 中的「私有」成员
// person[Symbol()]  无法访问
  person.say()
  ```
  
  应用场景2：实现对象的内置接口----内置 Symbol 常量
  
  ```
  // console.log(Symbol.iterator)
  // console.log(Symbol.hasInstance)
  // const obj = {}
  // console.log(obj.toString()) //[object Object]
  // const obj = {
  //   [Symbol.toStringTag]: 'XObject'
  // }
  // console.log(obj.toString()) //[object XObject]
  ```
