---
title: 优先队列
author: xxzuo
tags:
  - leetcode
  - java
categories:
  - 算法
date: 2023-10-23 10:01:08
---

## 概念
`PriorityQueue` 优先队列是一种先进先出（FIFO）的数据结构。和普通队列不同的是它出队的顺序是有优先级的


## 使用
### 构造方法
- Integer 或者 String类型
```java
// 小堆
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();

// 大堆
PriorityQueue<Student> priorityQueue = new PriorityQueue<>(new Comparator<Student>() {  
        @Override  
        public int compare(Student o1, Student o2) {  
            return o1.age - o2.age;  
        }  
    }); 

```

- 实体类
如果需要存储对象 需要重写 Comparator 否则会报错
比如下面这个代码
```java
public static void main(String[] args) {  
    PriorityQueue<Student> priorityQueue = new PriorityQueue<>();  
    priorityQueue.offer(new Student( 19,"小明"));  
    priorityQueue.offer(new Student( 24,"大明"));  
}  
  
  
static class Student{  
    private int age;  
    private String name;  
    public Student(int age, String name){  
        this.age = age;  
        this.name = name;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
}

```

会报
> Exception in thread "main" java.lang.ClassCastException: Test$Student cannot be cast to java.lang.Comparable
	at java.util.PriorityQueue.siftUpComparable(PriorityQueue.java:653)
	at java.util.PriorityQueue.siftUp(PriorityQueue.java:648)
	at java.util.PriorityQueue.offer(PriorityQueue.java:345)
	at Test.main(SqlTest.java:9)

需要改写成这样
```java
public static void main(String[] args) {  
    PriorityQueue<Student> priorityQueue = new PriorityQueue<>(new Comparator<Student>() {  
        @Override  
        public int compare(Student o1, Student o2) {  
            return o1.age - o2.age;  
        }  
    });  
    priorityQueue.offer(new Student( 19,"小明"));  
    priorityQueue.offer(new Student( 24,"大明"));  
}  
  
  
static class Student{  
    private int age;  
    private String name;  
    public Student(int age, String name){  
        this.age = age;  
        this.name = name;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
}

```


### 常用方法
```java
public boolean add(E e); //在队尾添加元素，并调整堆结构
public E remove(); //在队头删除元素，并返回，再调整堆结构
public E element(); //返回队头元素（不删除）
public boolean isEmpty(); //判断队列是否为空
​
public int size(); //获取队列中元素个数
public void clear(); //清空队列
public boolean contains(Object o); //判断队列中是否包含指定元素（从队头到队尾遍历）
public Iterator<E> iterator(); //迭代器

public E peek(); //获取优先级最高的元素
public E poll(); //移除优先级最高的元素 并返回
public boolean offer(E e); //插入元素, 成功返回 true, 会自动扩容

```

参考 ![[1464-数组中两元素的最大乘积]]


