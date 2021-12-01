## 可迭代接口--Iterable

### 前言
for...of循环是一种遍历所有数据的统一方式，但经过我们实际尝试它只能遍历数组之类的数据结构，对于普通对象如果直接去遍历它就会报出错误，那这到底是什么原因导致的呢

其实是因为ES中能够表示有结构的数据类型越来越多，从最早的数组和对象，到现在新增的Set和Map，我们开发者还可以组合使用这些类型去定义符合我们业务的数据结构，为了给各种各样的数据结构提供统一遍历方式，
ES2015就提供了Iterable接口，可以了解为规格标准，例如我们在ES当中任意一种数据类型都有toString方法。

实现Iterable接口就是for...of的前提，换句话说就是这个数据结构有Iterable接口，它就能使用for...of进行遍历。

### 能被使用for...of进行遍历的数据结构共同特点

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


### 分析Symbol.iterator方法

- 调用Symbol.iterator方法
```
const arr= ['foo','bar','baz']

arr[Symbol.iterator]()
```

发现调用的结果是数组的迭代器对象 Array Iterator{}

![image](https://user-images.githubusercontent.com/37037802/144004595-f4fac8a5-86d3-4cdf-a797-eede426cafa8.png)

- 调用对象中的next方法

```
const iterator = arr[Symbol.iterator]()

iterator.next()
```

结果返回为一个对象，对象中有2个属性（value,done）,其中value值就是我们数组中第1个元素值

![image](https://user-images.githubusercontent.com/37037802/144005072-895c38f3-8f42-42e9-8f57-1013ee793e0e.png)

- 再次调用next方法

![image](https://user-images.githubusercontent.com/37037802/144005371-9061229b-3744-4350-90ea-085f56a3d60c.png)

结果返回为一个与之前相同结构的对象,只不过value值为数组第2个元素值而已

#### 总结：有分析可看出，在迭代器当中内部应该是维护了数据指针，每调用一次next()，指针都会往后移动一位，done属性的作用就是表示数据是否被遍历完成

待更新...








