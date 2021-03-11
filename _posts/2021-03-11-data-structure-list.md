---
layout:     post
title:      数据结构List笔记
date:       2021-03-11
author:     Yimeng Ren
header-img: 
catalog: true
tags:
    - Data Structure
    - List
---

# List 笔记

## IntList

不定义IntNode子类，只通过定义first item和rest list，是定义List类的最简单方法，但是要通过递归思想来更新list，手动写出next的指向。

可以添加常用方法，如size()

```java
public class IntList {
  public int first;
  public IntList rest;
  
  public IntList(int f, IntList r) {
    first = f;
    rest = r;
  }
  
  public int size() {
    if (rest == null) {
      return 1;
    }
    return 1 + this.rest.size();
  }
  
  public static void main(args[0]) {
    IntList L = new IntList(5, null);
    L.rest = new IntList(10, null);
    L.rest = new IntList(15, null);
  }
}
```

IntList的缺点：必须用复杂的递归思想来定义和调用。

## SLList

Single Linked List

定义IntNode

```java
public class IntNode() {
  public int item;
  public IntNode next;
  
	public IntNode(int i, IntNode n) {
    item = i;
    next = n;
  }
}
```

基于IntNode定义SLList，将IntNode类定义在SLList类中作为子类，常用的方法 size(), addFirst(), getFirst()

```java
public class SLList() {
  public class IntNode() {
    public int item;
    public IntNode next;

    public IntNode(int i, IntNode n) {
      item = i;
      next = n;
    }
  }
  
  public IntNode first;
  
  public SLList(int x) {
    first = new IntNode(x, null);
  }
  
  public void addFirst(int x) {
    first = new IntNode(x, first);
  }
  public int getFirst() {
    return first.item;
  }
  
  public static void main(args[0]) {
    SLList L = new SLList(15);
    L.addFirst(10);
    L.addFirst(5);
    System.out.println(L.getFirst());
  }
}
```

为了避免不断调用first变量（例如`L.first.next.next = L.first.next`, 进而产生死循环的情况，将first变量定义为private。同时，子类IntNode不会调用SLList类中、IntNode子类之外的变量，因此把IntNode类声明为static类型。

再添加size()方法。为了避免递归/迭代调用size()导致计算量较大的情况，应当将size方法的时间复杂度设计为常数，使用Caching思想（储存），使用变量size，时刻更新该变量的值。

```java
public class SLList() {
  public static class IntNode() {
    public int item;
    public IntNode next;

    public IntNode(int i, IntNode n) {
      item = i;
      next = n;
    }
  }
  
  private IntNode first;  // user 不能调用first变量
  private int size;   // user 不能调用size变量
  
  public SLList(int x) {
    first = new IntNode(x, null);
    size = 1;
  }
  
  public void addFirst(int x) {
    first = new IntNode(x, first);
    size += 1;
  }
  public int getFirst() {
    return first.item;
  }
  public int size() {
    return size;
  }
  
  public static void main(args[0]) {
    SLList L = new SLList(15);
    L.addFirst(10);
    L.addFirst(5);
    System.out.println(L.getFirst());
  }
}
```

下面添加addLast()方法，如果按照常规思路，要考虑原来的列表为空的情况下，addLast需要创建一个IntNode，实例化为first，并返回；如果原来的列表非空，则将first node递推到最后一个node，返回item值。这个思路比较麻烦，因此首先添加一个空节点sentinel。

![](https://img-blog.csdn.net/20180722102629644?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTUwOTU3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```java
public void addLast(int x) {
  size += 1;
  IntNode p = sentinel;
  while(p.next != null) {
    p = p.next;
  }
  
  p.next = new IntNode(x, null);
}
```

为了避免链表越长，执行addLast()方法的时间越长的问题，仍然使用caching思想，添加last变量，储存最后一个节点的位置

```java
public class SLList() {
  private IntNode sentinel;
  private IntNode last;
  private int size;
  
  public void addLast(int x) {
    last.next = new IntNode(x, null);
    last = last.next;
    size += 1;
  }
  ...
}
```

SLList缺点：如果要添加removeLast()方法，需要获取倒数第二个节点sectolast，然后sectolast.next = null，如果从sentinel节点向后遍历，这个方法具有线性复杂度，所以引入DLList数据结构。

## DLList

Doubly Linked List：双向链表

在每一个IntNode中添加prev变量

```java
public static class IntNode() {
  public IntNode prev;
  public int item;
  public IntNode next;
}
```

![](https://img-blog.csdn.net/20180722105334321?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTUwOTU3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

last节点有可能指向带有数据的节点，也有可能指向sentinel节点，为了避免混乱，有以下两种方法。

### 给链表尾部添加sentinel空节点

![](https://img-blog.csdn.net/20180722110147162?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTUwOTU3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 采用circle思想，让首尾指针都指向同一个sentinel空节点

![](https://img-blog.csdn.net/20180722110325279?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwOTUwOTU3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

