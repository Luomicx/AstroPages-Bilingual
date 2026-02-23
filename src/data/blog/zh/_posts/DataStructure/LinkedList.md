---
title: 数据结构 - [LinkedList]
tags: []
copyright: true
date: 2025-09-26 20:20:40
permalink:
categories: 数据结构
description:
cover: 
mathjax: true
katex: true
---
# 链表的数据结构

### 简介

链表是一种常见的数据结构，它通过**节点**（Node）来存储元素，并使用指针或引用将这些节点链接在一起。链表的主要优点是插入和删除操作非常高效，因为它不需要移动其他元素。

### Java中LinkedList使用的链表类型

- **双向链表**：Java中的`LinkedList`实现基于**双向链表**。这意味着每个节点不仅知道下一个节点的地址，还知道前一个节点的地址。这种设计使得从列表两端进行插入和删除操作都同样高效。

### 链表的基本操作及其时间复杂度

1. **插入**
   - 在给定位置插入元素的时间复杂度为O(n)，因为可能需要遍历链表找到正确的插入点。
   - 但在头尾插入时，由于`LinkedList`是双向链表，所以时间复杂度为O(1)。
2. **删除**
   - 删除特定值的元素通常需要遍历链表以找到该元素，因此时间复杂度为O(n)。
   - 如果已经知道了要删除的节点的位置，则可以在O(1)时间内完成删除。
3. **获取元素**
   - 获取指定索引处的元素需要遍历链表直到找到目标位置，这导致了O(n)的时间复杂度。

### 使用链表的最佳场景

- 当你的应用程序频繁地在列表中间执行插入或删除操作时，链表是一个很好的选择。这是因为数组在这种情况下效率较低，因为它需要移动大量元素来维持顺序。
- 对于实现队列或栈等数据结构，特别是当你希望保持高效的添加和移除操作时，链表也非常适用。
- 在内存分配不连续的情况下，链表可以提供更好的性能，因为它不需要像数组那样要求连续的内存空间。

## 相关的实现代码

LinkedList.java

```java
/**
 * @Author: 无糖茶 wiretender.top
 * @CreateTime: 2025-09-26
 * @Description:
 * @Version: 1.0
 */
package linked_list;

public class LinkedListTest<E> implements List<E> {

    transient int size = 0;

    transient Node<E> first;

    transient Node<E> last;

    public LinkedListTest() {}

    void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null) {
            last = newNode;
        } else {
            f.prev = newNode;
        }
        size ++;
    }

    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null) {
            first = newNode;
        } else {
            l.next = newNode;
        }
        size ++;
    }

    @Override
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public boolean addFirst(E e) {
        linkFirst(e);
        return true;
    }

    @Override
    public boolean addLast(E e) {
        linkLast(e);
        return true;
    }

    @Override
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    E unlink(Node<E> x) {
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size --;
        return element;
    }
    @Override
    public E get(int index) {
        return node(index).item;
    }

    @Override
    public void printLinkList() {
        if (this.size == 0) {
            System.out.println("链表为空");
        } else {
            Node<E> temp = first;
            System.out.println("目前的列表，头节点：" + first.item + " 尾节点：" + last.item + " 整体：");
            while(temp != null) {
                System.out.println(temp.item + "，");
                temp = temp.next;
            }
            System.out.println();
        }
    }
    
    Node<E> node(int index) {
        // 这里是 size >> 1 是右移一位，也就是除以二，看距离头节点近还是尾节点近
       if (index < (size >> 1)) {
           Node<E> x = first;
           for (int i = 0; i < index; i ++) {
               x = x.next;
           }
           return x;
       } else {
           Node<E> x = last;
           for (int i = size - 1; i > index; i --) {
               x = x.prev;
           }
           return x;
       }
    }
    /*
        链表节点的私有类
         */
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;


        /**
         * 节点的构造函数
         * @param element 传入的节点的值
         * @param prev 前置节点的指针
         * @param next 后置节点的指针
         */
        public Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.prev = prev;
            this.next = next;
        }
    }


}
```

Link.java

```java
package linked_list;

/**
 * @Author: 无糖茶 wiretender.top
 * @CreateTime: 2025-09-26
 * @Description: List 接口
 */
public interface List<E> {

    boolean add(E e);

    boolean addFirst(E e);

    boolean addLast(E e);

    boolean remove(Object o);

    E get(int index);

    void printLinkList();

}

```

