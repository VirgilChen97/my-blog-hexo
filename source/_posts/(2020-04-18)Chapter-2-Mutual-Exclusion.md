---
title: Chapter 2 Mutual Exclusion
date: 2020-04-18 22:48:20
tags: Multi-core Concurrent Programming
categories: 课程笔记
---
# 时间表示

我们使用**事件**来表示程序执行过程中发生的操作， 每一个时间都是**立即发生**并且**不会重合**的。

一个线程 $A$ 会产生一系列的事件 $a_0$, $a_1$, 因为在程序中有许多的循环，因此我们把 $a_i$ 这一事件的第 $j$ 次发生记作 $a_i^j$. 如果事件 $a$ 发生在事件 $b$ 之前，那我们记作 $a\rightarrow b$.

我们把在线程 $A$ 上一个区间 $(a_0, a_1)$ 内的时间表示为  $I_A$, 如果  $I_A \rightarrow I_B$, 那么  $a_1 \rightarrow b_0$。

# Critical Sections

我们使用锁来管理不同的线程进出关键区，一个优秀的锁要做到：

1. **Mutual Exclusion**, 不同线程的Critical Section永远不会重合，即 $CS_A^k \rightarrow CS_B^j$ 或反过来 
2. **Freedom from Deadlock**, 如果多个线程尝试获取锁，那么最终有一个能获取到
3. **Freedom from Starvation**, 任何获取锁的请求最终都会成功

# Peterson Algorithm

Peterson 算法解决了两个线程之间的锁问题

```java
class Peterson implements Lock{
    // 每一个线程自己的flag
    private boolean[] flag = new boolean[2];
    // victim是谁谁就获取不到锁
    private int victim;
    public void lock(){
        // 自己的ID
        int i = ThreadID.get();
        // 对方的ID
        int j = 1 - i;
        flag[i] = true // 我想要获取锁
        victim = i; // 但是您先请
        while (flag[j] && victim == i) {}; // 等待获取锁
    }
    public void unlock(){
        int i = ThreadID.get();
        flag[i] = false; // 我不需要锁了
    }
}
```

# Filter Lock

现在我们需要把Peterson算法扩展，实现能被 $n$ 个线程使用的锁。

Filter算法创建了 $n-1$ 个“等待区”（Levels）,一个线程只有从等待区的最顶层层层穿越到最底层，才能够获取锁。在Filter算法中，每一层都遵循如下规定：

1. 在第 $q$ 层，至少有1个线程能够穿过
2. 如果多个线程想要穿过 $q$ 层，那么至少有一个线程会停留在 $q$ 层。

```java
class Filter implements Lock{
    int[] level // 等待区
    int[] victim // 在等待区最终被留下来的人
    public Filter(int n){
        level = new int[n]; // 表示 N 个线程分别对哪个等待区有兴趣
        victim = new int[n]; // 因为只需要阻止 n-1 个线程，所以只需要用 1 - n-1
        for(int i = 0; i < n; i++){
            level[i] = 0;
        }
    }
    public void lock() {
        int me = ThreadID.get();
        for(int i = 1; i < n; i++){
            level[me] = i; // 我对这一等待区感兴趣
            victim[i] = me; // 其他人先
            while(check(i)){};
        }
    }
    public boolean check(int me, int level){
        boolean res = false;
        for(int i = 0; i < n; i++){
            if(i == me){
                continue;
            }else{
                // 在之后层数的人没走，并且自己是本层留下来的人的时候，等待
                if(level[i] >= level && victim[level] == me){
                    res = true;
                }
            }
        }
        return res;
    }
}
```

# 公平性

Starvation-freedom 只能保证每个获取Lock的线程最终能够获取到Lock，但是并不保证先后顺序，所以我们不能保证先请求锁的线程先获取锁。因此我们把锁算法分为两个部分：

1. **Doorway**，我们把线程A在Doorway区发生的事件称为 $D_A$, 包括了有限个步骤
2. **Waiting**，  我们把线程A在Waiting区发生的事件称为 $W_A$，不限制步骤的数量

其中Doorway必须要有限步骤完成是非常重要的，我们称之为 **boundedwait-free progress**。

因此，我们这样定义公平性：

如果一个锁是FIFO的，那么它满足

 If $D_A^j \rightarrow D_B^k$, then $CS_A^j \rightarrow CS_B^k$

 # Bakery Algorithm

 一个FIFO的锁

 ```java
class Bakery implements Lock{
    boolean[] flag; // 代表线程是否对锁有兴趣
    Label[] label;  // 代表线程进入waiting的顺序
    public Bakery(int n){
        flag = new boolean[n];
        label = new Label[n];
        // Initialize
    }
    public void lock(){
        int i = ThreadID.get();
        flag[i] = true; // 我感兴趣
        label[i] = label.max()+1; // 给自己一个比已经在等待的所有线程都要大的时间戳（最后来的）
        while(check(me)){};
    }
    public void unlock(){
        flag[ThreadID.get()] = false; // 我不感兴趣了
    }
    public boolean check(int me){
        boolean res = false;
        for(int i = 0; i < n; i++){
            if(i == me){
                continue;
            }else{
                // 有线程感兴趣，而且的时间戳比我小，等待
                if(flag[i] && label[i] < label[me]){
                    res = true;
                }
            }
        }
        return res;
    }
}
 ```



