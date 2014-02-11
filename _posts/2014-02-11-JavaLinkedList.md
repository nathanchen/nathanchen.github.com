---
layout: post
title: "Java LinkedList源码阅读"
category: Reading Notes
tags: ["读文章", "Java"]
---
{% include JB/setup %}

## LinkedList

LinkedList是通过双向链表实现的。

**它的顺序访问会非常高效，而随机访问效率比较低**

	private transient Entry<E> header = new Entry<E>(null, null, null);

	// 获取LinkedList的最后一个元素
	public E getLast()  {
          if (size==0)
              throw new NoSuchElementException();
  
          // 由于LinkedList是双向链表；而表头header不包含数据。
          // 因而，这里返回表头header的前一个节点所包含的数据。
          return header.previous.element;
      }

	 // 获取双向链表中指定位置的节点
     private Entry<E> entry(int index) {
         if (index < 0 || index >= size)
             throw new IndexOutOfBoundsException("Index: "+index+
                                                 ", Size: "+size);
         Entry<E> e = header;
         // 获取index处的节点。
         // 若index < 双向链表长度的1/2,则从前先后查找;
         // 否则，从后向前查找。
         if (index < (size >> 1)) {
             for (int i = 0; i <= index; i++)
                 e = e.next;
         } else {
             for (int i = size; i > index; i--)
                 e = e.previous;
         }
         return e;
     }


	 // 从LinkedList末尾向前查找，删除第一个值为元素(o)的节点
     // 从链表开始查找，如存在节点的值为元素(o)的节点，则删除该节点
     public boolean removeLastOccurrence(Object o) {
         if (o==null) {
             for (Entry<E> e = header.previous; e != header; e = e.previous) {
                 if (e.element==null) {
                     remove(e);
                     return true;
                 }
             }
         } else {
             for (Entry<E> e = header.previous; e != header; e = e.previous) {
                 if (o.equals(e.element)) {
                     remove(e);
                     return true;
                 }
             }
         }
         return false;
     }


