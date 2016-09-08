---
layout: post
title:  "Java笔记整理：持有对象"
date:   2016-09-07 13:39
categories: java
tags: java
excerpt: 
---


《Java编程思想》第十一章

> 如果一个程序只包含固定数量的且生命周期都是已知的对象，那么这是一个非常简单的程序





<br/>
JAVA实用类库提供了一套容器类，其中基本的类型是List，Set，Queue和Map。也称为集合类

可以将任意数量的对象放置到容器中

JAVA中的容器都是类，也就是说JAVA没有直接的关键字支持


---------------
## 泛型

泛型指定了容器中元素的类型，访问元素时返回的是泛型类型的引用，可能需要自己转型

可以为容器指定多个泛型

使用泛型，可以在编译期防止错误类型的放置

<br/>
打印对象时可能产生的无符号十六进制数，是由hashCode方法产生的


----------------
## 基本概念

容器类可以分为两个概念

 - Collection：独立元素的序列，元素的排序服从一条或多条规则。其中包含：

   - List：必须按照插入的顺序保存元素
   - Set：不能有重复元素
   - Queue：按照排队规则来确定对象产生的顺序（通常与被插入顺序相同）

   所有的Collection对象能使用foreach语法遍历

 - Map：一组成对的“键值对”对象，允许使用键来查找值。映射表允许使用一个对象（不是引用）来查找另一个对象
	
### 添加一组元素
 Arrays.asList()方法接受一个数组，或者一个用逗号分隔的元素列表（可变参数），并将其转换为一个List对象

 Collections.addAll()方法接受一个Collection对象，以及一个数组或是逗号分隔的列表，将元素添加到Collection对象中

Collection.addAll()方法接受一个Collection对象，将该对象的元素添加到方法调用者中

<br/>		
Collection.addAll()运行更快，但不够灵活

<br/>
Arrays.asList()方法返回的List是和数组链接在一起的，因此不能对返回的List做插入和删除操作

可以指定asList方法返回列表的目标类型：Arrays.<E>asList()

<br/>
对于Map的初始化，除了用另一个Map之外，没有其他的初始化方式
		
### 容器的打印
对于数组，必须使用Arrays.toString()方法进行打印

对于容器，则无需任何帮助

对于Set和Map，除了LinkedHash的，其他类型的都不是按插入顺序存储的
		
### List
有两种类型的List：

 - ArrayList：擅长随机访问元素，但是中间插入和移除元素较慢
 - LinkedList：优化中间插入和移除元素，随机访问较慢，但是特性集比ArrayList大

contain：确定某个对象是否在列表中（containsAll，顺序无关）

indexOf：发现对象在List中所处位置的索引编号，如果没有则返回-1

remove：移除一个对象，如果成功返回true，否则（比如不存在）返回false（removeAll，顺序无关）

subList：从列表中创建出一个片段

retainAll：查找两个列表的交集

set：在指定的索引处将原先的元素替换

toArray：默认将列表转换成一个Object数组，如果参数提供特定类型的数组，则会返回一个这个类型的数组（如果能通过类型检查），如果提供的数组够大，会将元素放在这个数组中

<br/>
List的addAll方法支持在中间插入一个序列
		
### 迭代器
JAVA的Iterator只能单向移动

使用容器的iterator()方法获得该容器的一个迭代器，该迭代器将准备好返回序列的第一个元素

next()：返回下一个元素，即第一次调用next方法时将返回第一个元素

hasNext()：检查是否还有元素

remove()：将迭代器next产生的最后一个元素删除，即使用remove前必须先调用next

<br/>
迭代器的好处：将遍历序列的操作和序列底层的结构分离。迭代器统一了对容器的访问方式
		
#### ListIterator
ListIterator可以双向移动

previous()：返回当前元素并移动到上一个元素（和next相反）

nextIndex()：返回下一次next方法将访问的下标

previousIndex()：返回下一次previous方法将访问的下标

previousIndex = nextIndex - 1

set()：替换访问过的最后一个元素

<br/>
可以给listIterator()方法传入一个int参数来返回一个指向指定下标元素的迭代器
			
### LinkedList
LinkedList添加了可以使其用作栈，队列或双端队列的方法

getFirst()：返回列表的头，如果列表为空则抛出NoSuchElementException

element()：同上

peek()：同上，但列表为空时返回null

<br/>
remove()：移除并返回列表的头，如果列表为空则抛出NoSuchElementException

removeLast()：同上

poll()：同上，但列表为空时返回null


element

getFirst，get(int)，getLast

offerFirst，offer，offerLast

peekFirst，peek，peekLast

pollFirst，poll，pollLast

<br/>
pop，push

remove() = removeFirst()，remove(int)， remove(Object)

removeFirst， removeLast

removeFirstOccurrence，removeLastOccurrence
		

### Stack
Stack继承自Vector类，因此是线程安全的
		
### Set
通常会选择一个HashSet的实现，因为查找是Set最重要的操作

Set具有和Collection完全一样的接口，因此没有任何额外的功能（实际上Set就是Collection）

<br/>
HashSet使用了散列，TreeSet将元素存储在红黑树数据结构中

如果希望得到排序的结果，可以使用TreeSet

可以向TreeSet的构造器传入一个比较器Comparator（String.CASE_INSENSITIVE_ORDER）

contains()：测试对象是否在集合中
		
### Map
容器不能用于基本类型

containsKey：键是否存在

containsValue：值是否存在

keySet：返回包含所有键的集合（在foreach语句被使用）

values：返回包含所有值得Collection（值可以重复）

entrySet：返回所有键值对的集合
		
### Queue
先进先出（FIFO）的容器

LinkedList之所有支持队列的行为，是因为它实现了Queue接口

相关方法：offer，peek，poll，add，remove

#### PriorityQueue

当使用offer方法往PriorityQueue插入元素时，会在队列中被排序（维护一个堆），默认排序方式是自然排序。当调用peek，poll，remove方法，都会确保获得优先级最大的元素（最小的值拥有最大优先级）

可以自定义Comparator（Collections.reverseOrder()）
			
### Collection和Iterator

AbstractCollection提供了Collection的默认实现，如果要自定义Collection，可以创建AbstractCollection的子类型

必须要自己实现iterator()和size()方法
		
### foreach和迭代器

foreach语法可以适用于任何Collection对象

Iterable接口包含了一个能产生迭代器的iterator()方法，任何实现Iterable的类都能用于foreach语句

Collection类是Iterable，但Map不是

数组不是Iterable，但可以用于foreach，而且数组没有自动包装机制，数组不能转成Iterable

#### 适配器方法惯用法

如果直接将对象置于foreach语句中，将得到默认的迭代器（调用了iterable()）

如果在Iterable类中添加方法，返回一个产生反向迭代器的Iterable对象（该类操作的是原Iterable对象的数据），相当于调用一个方法，将自己变成另一种Iterable在返回，而这种Iterable能够返回反向的迭代器

<br/>
Arrays.asList方法产生的队列，如果对其进行修改，也会影响到原数组


----------	
## 总结

1. suzuki一旦生成，就不能改变容量
2. Collection保存单一元素，Map保存相关联的键值对。容器会自动调整尺寸。容器不能持有基本类型。容器有自动包装机制
3. 如果需要大量的随机访问，应该使用ArrayList，如果需要经常从表中插入或删除元素，应该使用LinkedList
4. 各种Queue以及栈的行为，由LinkedList提供支持
5. 新程序不应使用过时的Vector，Hashtable和Stack
	
				
		
			
