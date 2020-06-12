---
title: Chapter 4 Foundations of Shared Memory
date: 2020-04-29 16:31:02
tags: Multi-core Concurrent Programming
categories: 课程笔记
mathjax: true
---

# The space of registers 寄存器空间

## 定义

一个支持读写的寄存器的定义，如果实现的是 `Register<boolean>`，那么就叫做 **布尔寄存器（Boolean Register）**， 如果是 `Register<Integer>` ，那么就是 **多值寄存器（M-valued Register）**

```java
// Register的接口定义
public interface Register<T>{
    T read();
    void write(T val)
}
```

## 寄存器的安全性 Safe / Regular / Atomic

**Safe**

- 如果 `read()` 和 `write()` 不重叠，那么 `read()` 应当返回最近一次 `write()` 写入的值。
- 如果 `read()` 和 `write()` 重叠，那么 `read()` 可以返回寄存器范围内的任意值。

**Regular**

- 如果 `read()` 和 `write()` 不重叠，那么 `read()` 应当返回最近一次 `write()` 写入的值。
- 如果 `read()` 和 `write()` 重叠， 假设 $v_0 - v_k$ 是曾经被写入这个寄存器的值。那么 `read()` 应当返回这些值的其中之一。

**Atomic**

- 在regular的基础上，先发生的 `read()` 不能读取到比后发生的 `read()` 更新的值

## 单线程寄存器

在单线程环境下可以正常运行的寄存器

```java
public class SequentialRegister<T> implements Register<T> {
    private T value;
    public T read(){ 
        return value;
    }
    public void write(T v){
        value = v;
    }
}
```

## MRSW Safe Register

基本思路就是为每一个线程创建一个SRSW的寄存器，每一个线程都去自己对应的SRSW寄存器读取，写入则通过循环写入所有的SRSW寄存器

```java
public class SafeBooleanMRSWRegister implements Register<Boolean> {
    boolean[] s_table; // array of safe SRSW registers
    public SafeBooleanMRSWRegister(int capacity) {
        s_table = new boolean[capacity];
    }
    public Boolean read() {
        return s_table[ThreadID.get()];
    }
    public void write(Boolean x) {
        for (int i = 0; i < s_table.length; i++){
            s_table[i] = x;
        }
    }
}
```

## MRSW Regular Register

对于Boolean值来说，我们分为两种情况：

1. 新写入的值和旧值不同，这种情况下safe和regular的行为相同，都可以返回0或者1
2. 新写入的值和旧值相同，此时regular只能返回旧值，而safe可以返回0或者1

因此我们只需要做的是在写入一个新值时，只有和原来的值不同才进行写入

```java
public class SafeBooleanMRSWRegister implements Register<Boolean> {
    ThreadLocal<Boolean> last;
    boolean s_value; // safe MRSW register
    RegBooleanMRSWRegister(int capacity) { 
        last = new ThreadLocal<Boolean>() {
            protected Boolean initialValue() { return false; }; 
        }; 
    }
    public void write(Boolean x) {
        if(x != last.get()){
            last.set(x);
            s_value = x;
        }
    }
    public Boolean read(){
        return s_value;
    }
}
```

## M-valued MRSW Regular Register

使用unary annotation来表示多值

```java
public class RegMRSWRegister implements Register<Byte> {
    private static int RANGE = Byte.MAX_VALUE - Byte.MIN_VALUE + 1;
    boolean[] r_bit = new boolean[RANGE]; // regular boolean MRSW
    public RegMRSWRegister(int capacity) {
        for (int i = 1; i < r_bit.length; i++){
            r_bit[i] = false;
        }
        r_bit[0] = true;
    }
    public void write(Byte x) {
        // 设置第x位,然后将左边的覆盖为0
        r_bit[x] = true;
        for (int i = x - 1; i >= 0; i--){
            r_bit[i] = false;
        }
    }
    public Byte read() {
        // 从左数直到第一个1
        for (int i = 0; i < RANGE; i++){
            if (r_bit[i]) {
                return i;
            }
        }
    }
}
```

## SRSW Atomic Register

我们给通过给每次写入添加timestamp来解决，每次 `read()` 都记录read到值的timestamp，如果有线程read到了值小于timestamp，那么便直到自己读取到了旧值。

```java
public class AtomicSRSWRegister<T> implements Register<T> {
    ThreadLocal<Long> lastStamp;
    ThreadLocal<StampedValue<T>> lastRead;
    StampedValue<T> r_value;  // stored in regular SRSW register
    public AtomicSRSWRegister(T init) {
        r_value = new StampedValue<T>(init);
        lastStamp = new ThreadLocal<Long>() {
            protected Long initialValue() { return 0; };
        };
        lastRead = new ThreadLocal<StampedValue<T>>(){
            protected StampedValue<T> initialValue() { return r_value; };
        };
    }
    public T read(){
        // 读取当前值
        StampedValue<T> value = r_value;
        // 读取最后一次读取读取的值
        StampedValue<T> last = lastRead.get();
        // 比较二者的timestamp，return较新的那一个
        StampedValue<T> result = StampedValue.max(value, last);
        lastRead.set(result);
        return result.value;
    }
    public void write(T val){
        long stamp = lastStamp.get()+1
    }
}
```

## MRSW Atomic Register

在我们搭建 SRSW Atomic Register 的时候，我们是通过一个 `lastStamp` 和 `lastRead` 变量，让读取到最新值的Reader去帮助其他reader获取最新的值。而在MRSW版本中我们延续了这个思想。让先读取到的Reader帮助其他Reader获取最新的值。具体解释在代码中。

```java
public class AtomicMRSWRegister<T> implements Register<T> {
    ThreadLocal<Long> lastStamp; // 最新时间戳
    private StampedValue<T>[][] a_table; // 每一个值都存在 SRSW atomic 寄存器中
    public AtomicMRSWRegister(T init, int readers) {
        lastStamp = new ThreadLocal<Long>() {
            protected Long initialValue() { return 0; };
        };

        // 使用一个 reader * reader 大小的数组
        a_table = (StampedValue<T>[][]) new StampedValue[readers][readers];

        // 初始化
        StampedValue<T> value = new StampedValue<T>(init);
        for (int i = 0; i < readers; i++) {
            for (int j = 0; j < readers; j++) {
                a_table[i][j] = value;
            }
        }
    }

    public void write(T v) {
        long stamp = lastStamp.get() + 1; // 时间戳++
        lastStamp.set(stamp); // 设置最新时间戳
        StampedValue<T> value = new StampedValue<T>(stamp, v);
        for (int i = 0; i < a_table.length; i++) {
            a_table[i][i] = value; // 将对角线上的寄存器依次写入新值
        }
    }

    public T read(){
        int me = ThreadID.get();
        StampedValue<T> value = a_table[me][me]; // 首先读取自己对应位置的StampedValue

        // 纵向寻求其他Reader的帮助，看他们是否有获取到更新的值
        for (int i = 0; i < a_table.length; i++) {
            value = StampedValue.max(value, a_table[i][me]);
        }

        // 横向覆盖最新值帮助其他Reader，其他Reader纵向扫描就可以扫描到
        for (int i = 0; i < a_table.length; i++) {
            if (i == me) continue;
            a_table[me][i] = value;
        }

        return value
    }
```