<!--
 * @Author: iwan
 * @Description: 
 * @Date: 2021-07-12 15:02:53
 * @FilePath: \VS-Code-Markdown\summer holiday\diary.md
-->
# 2021/7/12

## STL

### allocator

#### 对象的构造和析构

###### trivial_destructor
析构的第二种方法，将[first,last]里的全都析构掉

假如这一堆接无关痛痒的析构函数，每次都调用会很浪费时间，伤害效率。使用```value_type()```获得迭代器所指对象的型别，再利用```__type_traits<T>```判断析构函数的类型，如果是```__true_type```的话就什么都不做，不是的话就要遍历，对每个对象调用第一个版本的```destory()```(调用对象的析构函数)

##### 内存的配置和释放

在```c++```中的内配置和释放时使用```operator::new()```和```operator::delete()```实现的，而```c```是用```malloc()```和```free()```实现的，所以```c++```底层是由这两个实现的,当然还要看配置区块所需要的大小，但比较大超超过128bytes，视为足够大，调用第一级配置，使用```malloc()``` 和 ```delete()``` 否则会使用更加复杂的```memory pool``

## 算法导论

### 什么是算法

**算法**就是任何良定义的计算过程，该过程去某个值或值的集合作为**输入**并产生某个值或值的集合作为**输出**。


# 2021/7/13

忘记写啦水了一天，搞了一下导师制项目的环境，打算把这个东西做成我以后的后端后台框架。

# 2021/7/14

## STL

看了看三种内存处理的方式```uninitialized_fill```，```unitialized_copy```,```unitialized_fill_n```。针对是不是```POD```做出为了效率和安全的不同操作。



# 2021/7/19
去上海玩了一天很舒服,人活着不就是为了明日香吗

# 2021/7/20
把登录的东西弄了个大概，登陆也是复杂的啊啊啊啊

# 2021/7/21

## session token cookie
[彻底理解session token cookie](https://www.cnblogs.com/moyand/p/9047978.html)
这篇文章感觉很八错，牛逼

# 2021/7/28
## backstage
好几天没有记录了都没怎么写代码。

最近把```layout```写了个大概知道了```vue```的组件该怎么使用,还有就是在```springboot```里面设置```tokenService```，对前端发来的请求进行进行拦截分析，把登陆的用户信息存储在```redis```里面，并且用```uuid```和一些关键字作为键来存储，同时根据```uuid```生成```token```，返回前端，每次拦截纠就分析这个```token```获取用户的信息。

## 算法
最近算法还是多写了点，```stl```的书没怎么看，影响比较深的就是一个八向搜索,还有就是树的问题不一定要把树建出来。还是只能做做简单的题啊，哎
