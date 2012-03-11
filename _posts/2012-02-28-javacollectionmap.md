---
layout: post
title: "javaCollection和Map源码分析"
category: Study Notes 
tags: ["读书", "源码阅读"]
---
{% include JB/setup %}

集合包中最常用的是Collection和Map两个接口的实现类。Collection用于存放多个单对象，Map用于存放Key-Value形式的键值对。

Coolection中常用的是两种类型的接口：List和Set。两者最明显的差别是List支持放入重复的对象，而Set不支持。

### 1. ArrayList
实现方式：
ArrayList采用的是数组方式来存放数据。在调用空构造器的情况下会创建一个大小为10的Object数组。

**插入对象：add（E）**
如果插入一个对象以后，ArrayList依然没有满，那就是单纯的将数组中某个元素的值赋值为传入的对象。

如果插入一个对象后，ArrayList会产生溢出，则要将数组的容量扩大。新的数组的容量计算方法为：旧数组容量*1.5 + 1。在得到新的数组容量以后，将原数据的数据拷贝到新数组中Arrays.copyOf。

-add（int，E）允许将元素直接插入置顶的int位置上。这个方法的实现首先要确保插入的位置是当前Array数组中存在的，之后还要确保数组的容量是够用的。在完成以上动作后，它要将数据对象进行一次复制，然后将int指定的index后面的数据往后挪动一位，然后才能将指定的index位置的复制为传入的对象。

{
	fastRemove（）
	将index后的对象往前复制一位


}

**删除对象：remove（E）**
ArrayList首先判断对象是否为null。如果是null，则遍历数组中已有值的元素，并比较其是否为null，若为null，则调用fastRemove来删除相应位置的对象。fastRemove（）会将index后的对象往前复制一位，并将数组中的最后一个元素的值设置为null，即释放了对此对象的引用。

如果这个对象不是null，唯一的不同在于通过E的equals来比较元素的值是否相同，如相同则认为是需要删除对象的位置，然后同样是通过调用fastRemove来完成对象的删除。

-remove（int）的实现比remove（E）多了一个数组范围的检测，但少了对象位置的查找，因此性能会更好。


**遍历对象：iterator（）**
当调用hasNext（）方法时，比较当前指向的数组的位置是否和数组中已有的元素的大小相等，如相等则返回false，否则返回true。

当调用next方法时，首先比较在创建此Iterator时获取的modCount与当前的modCount，如果这两个modCount不相等，则说明在获取next元素时，发生了对于集合大小产生影响（新增，删除）的动作。当产生这种情况时，则抛出ConcurrentModificationException。如果modCount相等，则调用get方法来获取相应位置的元素。


**判断对象是否存在：contains（E）**
做法为遍历整个ArrayList中已有的元素，通过equals来判断是否和元素相等。


indexOf和lastIndexOf是ArrayList中用于获得对象所在位置的方法，其中indexOf为从前往后寻找，而lastIndexOf为从后往前寻找。



**注意要点：**
	- ArrayList基于数组方式实现，无容量的限制
	- ArrayList在执行插入元素时可能要扩容，在删除元素时并不会减小数组的容量（如希望相应的减少数组容量，可以调用ArrayList的trimToSize（）），在查找元素时要遍历数组，对于非null的元素采取equals的方式寻找。
	- ArrayList是非线程安全的。



### 2. LinkedList
linkedList基于双向链表机制，所谓双向链表机制是指集合中的每个元素都知道其前一个元素及其后一个元素的位置。这样的机制可以快速实现集合中元素的移动。

**LinkedList（）**
在创建LinkedList对象时，应首先创建一个element属性为null，next属性为null及previous属性为null的Entry对象，并赋值给全局的header属性。在执行构造器时，LinkedList将header的next及previous都指向header。


**add（E）**
LinkedList的add方法不用像ArrayList那样，要考虑扩容及复制数组的问题，但它每增加一个元素，都要创建一个新的Entry对象，并要修改相邻的两个元素的属性。


**remove（E）**
删除时只须直接删除链表上的当前元素，并将当前元素中的element，previous及next属性设置为null，即可完成对象的删除。


**get（int）**
由于LinkedList的元素并没有存储在一个数组中，因此其get操作过程比ArrayList更为复杂。

首先判断当前要获取的位置是否小于LinkedList值的一半，如小于，则从头一直找到index位置所对象的next元素；如大于，则从队列的尾部往前，一直找到index位置所对应的previous元素。


**iterator（）**
由于LinkedList是基于双向链接实现的，因此其在遍历时还可往前遍历，通过调用hasPrevious和previous来完成遍历过程。


**注意要点：**
	- LinkedList基于双向链表机制实现。
	- LinkedList在插入元素时，须创建一个新的Entry对象，并切换相应元素的前后元素的引用；在查找元素时，须遍历链表；在删除元素时，要遍历链表，找到要删除的元素，然后从链表上将此元素删除即可
	- LinkedList是非线程安全

## REFERENCE
林昊，《分布式Java应用》，电子工业出版社，2011