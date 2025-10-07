---
title: "Java Concurrency: Thread, Executor và quản lý đa luồng"
date: 2024-10-16T14:00:00+07:00
draft: false
description: "Hướng dẫn lập trình đa luồng với Thread, ExecutorService; đồng bộ hóa và tránh deadlock"
category: "Java"
tags: ["Java", "Concurrency", "Thread", "Executor", "Synchronized"]
readingTime: "16 phút"
author: "TG-Bao"
---

## 🧵 Thread cơ bản

```java
public class ThreadDemo {
    static class Worker extends Thread {
        private final int id;
        Worker(int id){ this.id = id; }
        @Override public void run(){ System.out.println("Worker " + id + " running"); }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Worker(1);
        t1.start();
        t1.join(); // chờ t1 kết thúc
    }
}
```

## ⚙️ ExecutorService

```java
import java.util.concurrent.*;

public class ExecutorDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(4);
        Future<Integer> f = pool.submit(() -> 40 + 2);
        System.out.println(f.get());
        pool.shutdown();
    }
}
```

## 🔒 Đồng bộ hoá

```java
class Counter {
    private int value = 0;
    public synchronized void inc(){ value++; }
    public synchronized int get(){ return value; }
}
```

## 🚫 Tránh deadlock

```java
// Quy ước thứ tự khoá hoặc dùng tryLock với timeout để tránh chờ vĩnh viễn
```

## ✅ Bài tập
- Viết chương trình tải 10 URL song song và tổng hợp kết quả.
- Đo thời gian giữa `new Thread` vs `ExecutorService`.


