## JavaScript性能优化(A)

### 内存管理介绍

a.内存：由可读写单元组成，表示一片可操作空间

b.管理：人为的去操作一片空间的申请、使用和释放

c.内存管理：开发者主动申请空间、使用空间、释放空间

d.管理流程：申请-使用-释放

```markdown
//申请
let obj = {}

//使用
obj.name = "xxx"

//释放
obj = null
```

### JavaScript中的垃圾回收

#### JavaScript中的垃圾

a.JavaScript中内存管理是自动的

b.对象不再被引用时时垃圾

c.对象不能从根上访问到时是垃圾

#### JavaScript中的可达对象

a.可以访问到的对象就是可达对象（引用、作用域链）

b.可达的标准就是从根出发是否能够被找到

c.JavaScript中的根就是可以理解为是全局变量对象

![image](https://user-images.githubusercontent.com/37037802/116674602-74593c00-a9d7-11eb-84cf-75452cc9a689.png)

![image](https://user-images.githubusercontent.com/37037802/116674693-8dfa8380-a9d7-11eb-8ef8-76000ebc7226.png)
