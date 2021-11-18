## JavaScript性能优化(B)

### Performance工具介绍

#### 为什么使用Performance

a.GC的目的是为了实现内存空间的良性循环

b.良性循环的基石是合理使用

c.时刻光注才能确定是否合理

d.Performance提供多种监控方式


#### Performance使用步骤

a.打开浏览器输入目标地址

b.进入开发人员工具面板，选择性能
![image](https://user-images.githubusercontent.com/37037802/117280050-cf39da00-ae94-11eb-8f31-e0521f573bc1.png)


c.开启录制功能，访问具体界面
![image](https://user-images.githubusercontent.com/37037802/117280595-57b87a80-ae95-11eb-8e5a-16c824d05ed4.png)


d.执行用户行为，一段时间后停止录制
![image](https://user-images.githubusercontent.com/37037802/117280482-38b9e880-ae95-11eb-8e43-0c5ab3ce57e5.png)


e.分析界面中记录的内存信息
![image](https://user-images.githubusercontent.com/37037802/117280362-1e800a80-ae95-11eb-80e0-b2296f1ef9b6.png)


### 内存问题的外在表现

a.页面出现延迟加载或经常性暂停

b.页面持续性出现糟糕的性能

c.页面的性能随时间延长越来越差
