---
title: 数据结构 - [Array]
pubDatetime: 2025-09-26T20:20:40Z
description: 数据结构 - 数组笔记
featured: false
draft: false
tags:
  - 数据结构
---
> 本笔记意在巩固数据结构相关知识，详细具体的数据结构知识请查询具体资料 
# 数组

## 前言

数组只是个名称，它可以描述一组操作，也可以命名这组操作。数组的数据操作，是通过 idx->val 的方式来处理。它不是具体要求内存上要存储着连续的数据才叫数据，而是说，通过连续的索引 idx，也可以线性访问相邻的数据。那么当你定义了数据的存储方式，也就定义了数据结构。所以它也是被归类为数据结构。

## 特点

1. 数组是相同数据类型的集合（int 中不能存放 double）
2. 数组中的各元素的存储是有先后顺序的，它们在内存中按照这个数据结构连续存放在一起。内存地址连续。
3. 数组获取元素的时间复杂度为O(1)

## 相关的代码实现

List.java

```java
package array_list;

public interface List<E> {

    boolean add(E e);

    E remove(int index);

    E get(int index);

}

```

ArrayListTest.java

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

