## 可迭代接口--Iterable

### 前言
for...of循环是一种遍历所有数据的统一方式，但经过我们实际尝试它只能遍历数组之类的数据结构，对于普通对象如果直接去遍历它就会报出错误，那这到底是什么原因导致的呢

其实是因为ES中能够表示有结构的数据类型越来越多，从最早的数组和对象，到现在新增的Set和Map，我们开发者还可以组合使用这些类型去定义符合我们业务的数据结构，为了给各种各样的数据结构提供统一遍历方式，
ES2015就提供了Iterable接口，可以了解为规格标准，例如我们在ES当中任意一种数据类型都有toString方法。

实现Iterable接口就是for...of的前提，换句话说就是这个数据结构有Iterable接口，它就能使用for...of进行遍历。

### 看看能被使用for...of进行遍历的数据结构共同特点

- 数组

```
console.log([])
```
查看数组原型对象可以看到 Symbol.iterator 方法

![image](https://user-images.githubusercontent.com/37037802/143867854-e3e85632-0f87-49e1-8ace-9474a99ef658.png)

- Set对象
```
console.log(new Set())
```

查看Set对象原型对象也可以看到 Symbol.iterator 方法

![image](https://user-images.githubusercontent.com/37037802/143867854-e3e85632-0f87-49e1-8ace-9474a99ef658.png)

- Map对象
```
console.log(new Map())
```

查看Map对象原型对象也可以看到 Symbol.iterator 方法

![image](https://user-images.githubusercontent.com/37037802/143867854-e3e85632-0f87-49e1-8ace-9474a99ef658.png)


待更新...

