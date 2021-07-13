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

### 