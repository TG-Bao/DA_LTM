---
title: "Java Concurrency: Thread, Executor và quản lý đa luồng"
date: 2025-09-09T14:00:00+07:00
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

### Thread là gì? Vòng đời ra sao?

- New → Runnable → Running → Terminated (có thể Waiting/Blocked giữa đường)
- `start()` tạo thread mới, `run()` là hàm được thực thi trong thread đó. Không gọi `run()` trực tiếp trừ khi bạn muốn chạy đồng bộ trong cùng thread hiện tại.
- `join()` giúp chờ 1 thread kết thúc — tránh race ở bước đọc kết quả.

```
Main Thread
   └── start() → Worker Thread → run() ... → kết thúc → join() xong mới tiếp tục
```

### Khi nào nên dùng Thread trực tiếp?

- Tác vụ rất đơn giản, số lượng ít, quản lý vòng đời rõ ràng.
- Không phù hợp khi bạn cần nhiều task lặp lại, hàng đợi công việc, hoặc giới hạn số luồng — lúc đó chuyển sang Executor.

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

### Vì sao nên dùng Executor?

- Tái sử dụng pool luồng, giảm chi phí tạo/destroy thread.
- Dễ đặt chính sách: số luồng tối đa, hàng đợi, từ chối nhiệm vụ,…

### Các kiểu Executor thường gặp

- `newFixedThreadPool(n)`: n luồng cố định — phù hợp CPU-bound vừa phải.
- `newCachedThreadPool()`: co giãn theo tải — phù hợp I/O-bound ngắn, có thể bùng nổ số luồng nếu không kiểm soát.
- `newSingleThreadExecutor()`: bảo đảm tuần tự — phù hợp công việc cần thứ tự.
- `newScheduledThreadPool(n)`: lập lịch định kỳ/hoãn.

### Future, timeout và hủy

```java
Future<Integer> f = pool.submit(() -> heavy());
try {
    Integer v = f.get(500, TimeUnit.MILLISECONDS); // timeout để tránh treo mãi
} catch (TimeoutException e) {
    f.cancel(true); // yêu cầu hủy (nếu task tôn trọng interrupt)
}
```

> Lưu ý: Muốn hủy an toàn, trong task nên kiểm tra `Thread.currentThread().isInterrupted()` hoặc bắt `InterruptedException` đúng cách.

### CompletableFuture: compose bất đồng bộ

```java
CompletableFuture<Integer> price = CompletableFuture.supplyAsync(() -> fetchPrice());
CompletableFuture<Integer> discount = CompletableFuture.supplyAsync(() -> fetchDiscount());

CompletableFuture<Integer> finalPrice = price.thenCombine(discount, (p, d) -> p - d)
    .thenApply(Math::abs);

int result = finalPrice.join();
```

Khi workload mang tính I/O (gọi HTTP/DB), `CompletableFuture` giúp ghép các bước mà không chặn.

## 🔒 Đồng bộ hoá

```java
class Counter {
    private int value = 0;
    public synchronized void inc(){ value++; }
    public synchronized int get(){ return value; }
}
```

### synchronized bảo đảm điều gì?

- Tính mutual exclusion: cùng thời điểm chỉ một thread vào vùng bảo vệ.
- Tính visibility: vào/ra `synchronized` là điểm memory barrier, đảm bảo các write trước đó có thể thấy được cho thread khác.

### `volatile` vs `synchronized`

- `volatile` chỉ đảm bảo visibility (đọc thấy giá trị mới nhất), KHÔNG đảm bảo atomicity.
- Dùng `volatile` cho cờ điều khiển đơn giản: `volatile boolean running = true;`
- Với cập nhật cộng/trừ, cấu trúc phức tạp → cần `synchronized` hoặc `Atomic*` (vd `AtomicInteger`).

```java
class Flag {
    private volatile boolean running = true;
    void stop(){ running = false; }
    void loop(){ while (running) { /* work */ } }
}
```

### Lock tường minh

```java
final ReentrantLock lock = new ReentrantLock(true); // fair lock
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

Ưu điểm: tryLock/timeout, công bằng, điều kiện chờ `Condition`. Nhược: code dài hơn, dễ quên unlock.

## 🚫 Tránh deadlock

```java
// Quy ước thứ tự khoá hoặc dùng tryLock với timeout để tránh chờ vĩnh viễn
```

### 4 điều kiện gây bế tắc (Coffman):

1. Mutual exclusion, 2) Hold-and-wait, 3) No preemption, 4) Circular wait.
   Chỉ cần phá vỡ một điều kiện (thường là 4) sẽ tránh được deadlock.

### Quy ước thứ tự khóa

```java
void transfer(Account a, Account b, int amount) {
    Account first  = a.id < b.id ? a : b;
    Account second = a.id < b.id ? b : a;
    synchronized (first) {
        synchronized (second) {
            // chuyển tiền
        }
    }
}
```

### Dùng `tryLock` với timeout

```java
if (lock1.tryLock(50, TimeUnit.MILLISECONDS)) {
    try {
        if (lock2.tryLock(50, TimeUnit.MILLISECONDS)) {
            try { /* do work */ }
            finally { lock2.unlock(); }
        }
    } finally { lock1.unlock(); }
}
```

> Logging đầy đủ khi timeout để chẩn đoán.

## ✅ Bài tập

- Viết chương trình tải 10 URL song song và tổng hợp kết quả.
- Đo thời gian giữa `new Thread` vs `ExecutorService`.

## 📌 Gợi ý thiết kế & best practices

- Ưu tiên `ExecutorService`/`CompletableFuture` thay vì tạo Thread trần.
- Giới hạn số luồng theo tài nguyên: CPU-bound ≈ số core, I/O-bound có thể cao hơn nhưng phải đo đạc.
- Đặt tên luồng: `Executors.defaultThreadFactory()` có thể wrap để setName → giúp log dễ hiểu.
- Luôn `shutdown()`/`shutdownNow()` và chờ `awaitTermination` để đóng ứng dụng sạch.
- Bọc tác vụ bằng lớp nhỏ có log bắt đầu/kết thúc/thời lượng.
- Tránh chặn trong `ForkJoinPool.commonPool()` khi dùng `CompletableFuture` (có thể gây starvation). Khi cần, tự cung cấp `Executor` riêng.

## 🧪 Bài kiểm tra nhanh (self-check)

1. Sự khác nhau giữa `volatile` và `synchronized`? Khi nào dùng mỗi cái?
2. Vì sao `get()` trên `Future` có thể gây nghẽn? Bạn xử lý timeout/hủy ra sao?
3. Hãy nêu 2 kỹ thuật tránh deadlock trong hệ thống chuyển tiền giữa tài khoản.

---

Nếu bạn muốn, mình có thể bổ sung thêm phần: mô hình bộ nhớ Java (JMM), `Semaphore`, `ReadWriteLock`, `StampedLock`, và ví dụ thực tế (crawl web song song + cache).
## 🧠 Java Memory Model (JMM) — nền tảng đúng-sai của đa luồng

- **Visibility**: Khi một thread ghi dữ liệu, thread khác có nhìn thấy không? JMM định nghĩa các điểm “happens-before” đảm bảo nhìn thấy được.
- **Reordering**: JVM và CPU có thể đảo lệnh để tối ưu, miễn là không phá vỡ quan hệ happens-before. Vì thế, code nhìn “đúng” vẫn có thể sai nếu thiếu rào chắn bộ nhớ.
- **Happens-before (HB)** quan trọng:
  - Ghi/đọc trên cùng một biến `volatile` thiết lập HB.
  - Vào/ra `synchronized` thiết lập HB giữa critical sections trên cùng monitor.
  - Kết thúc `Thread.start()` HB trước `run()`; `run()` HB trước `Thread.join()` hoàn thành.
  - Gán trường `final` hoàn tất trong constructor HB trước lần đầu tiên thấy object từ thread khác (nếu không “thoát” this sớm).

Hệ quả thực tế:
- Dùng `volatile` cho cờ điều khiển để thread khác “thấy” cập nhật ngay.
- Không publish `this` khỏi constructor (tránh rò rỉ đối tượng chưa hoàn tất khởi tạo).
- Đừng tự nghĩ “viết xong là thread khác đọc thấy ngay” nếu không qua một HB hợp lệ.

```java
class StopSignal {
    private volatile boolean running = true; // đảm bảo visibility và HB
    public void stop() { running = false; }
    public void loop() {
        while (running) { /* work */ }
    }
}
```

## 🧰 Synchronizers quan trọng trong java.util.concurrent

### Semaphore — giới hạn mức song song
Lý thuyết: Bộ đếm giấy phép (permits). Hữu ích để giới hạn số tác vụ đồng thời truy cập tài nguyên hữu hạn (kết nối, file, API).

```java
Semaphore permits = new Semaphore(10);
void handle() throws InterruptedException {
    permits.acquire();
    try { process(); }
    finally { permits.release(); }
}
```

### CountDownLatch — chờ n sự kiện hoàn tất (một lần)
```java
CountDownLatch latch = new CountDownLatch(3);
executor.submit(() -> { step1(); latch.countDown(); });
executor.submit(() -> { step2(); latch.countDown(); });
executor.submit(() -> { step3(); latch.countDown(); });
latch.await(); // tiếp tục khi cả 3 xong
```

### CyclicBarrier — đồng bộ nhiều thread theo chặng (tái sử dụng)
```java
CyclicBarrier barrier = new CyclicBarrier(4);
for (int i = 0; i < 4; i++) {
    executor.submit(() -> {
        computePhase1();
        barrier.await(); // đợi đủ 4 thread
        computePhase2();
    });
}
```

### Phaser — phiên bản linh hoạt hơn cho nhiều pha, số bên tham gia thay đổi được

## 🔓 Advanced Locks — ReadWriteLock, StampedLock

### ReadWriteLock
- Cho phép nhiều độc giả song song, độc quyền khi ghi.
- Hữu ích khi workload chủ yếu đọc, ghi ít.

```java
final ReadWriteLock rw = new ReentrantReadWriteLock();
int read() {
    rw.readLock().lock();
    try { return value; }
    finally { rw.readLock().unlock(); }
}
void write(int v) {
    rw.writeLock().lock();
    try { value = v; }
    finally { rw.writeLock().unlock(); }
}
```

### StampedLock — optimistic read cực nhanh khi không có ghi

```java
final StampedLock sl = new StampedLock();
double distance() {
    long stamp = sl.tryOptimisticRead();
    double x = this.x, y = this.y;
    if (!sl.validate(stamp)) {
        stamp = sl.readLock();
        try { x = this.x; y = this.y; }
        finally { sl.unlockRead(stamp); }
    }
    return Math.hypot(x, y);
}
```

Lưu ý: StampedLock không reentrant, cẩn thận deadlock tự thân; chỉ dùng khi hiểu rõ đặc tính.

## 🧩 ThreadPoolExecutor — tinh chỉnh thực chiến

Các tham số cốt lõi:
- corePoolSize, maximumPoolSize, keepAliveTime, workQueue, threadFactory, rejectedExecutionHandler.

Chọn kích thước pool:
- CPU-bound: xấp xỉ số core (N hoặc N±1). Thử nghiệm với `Runtime.getRuntime().availableProcessors()`.
- I/O-bound: cao hơn số core vì phần lớn thời gian chờ; cần đo đạc latency và load.

Chiến lược hàng đợi:
- `SynchronousQueue` cho hệ thống “thả việc ngay cho thread” (thường dùng với cached).
- `LinkedBlockingQueue` (không giới hạn) có thể che giấu tắc nghẽn -> backpressure kém.
- `ArrayBlockingQueue` (giới hạn) giúp kiểm soát độ trễ và backpressure tốt hơn.

Từ chối nhiệm vụ (RejectionPolicy):
- `AbortPolicy` (mặc định), `CallerRunsPolicy` (giảm tải bằng cách chạy trên thread gọi), `DiscardPolicy`, `DiscardOldestPolicy`.

```java
ThreadFactory named = r -> {
    Thread t = new Thread(r);
    t.setName("worker-" + THREAD_ID.incrementAndGet());
    t.setDaemon(false);
    return t;
};

ExecutorService pool = new ThreadPoolExecutor(
    8, 16, 60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(500),
    named,
    new ThreadPoolExecutor.CallerRunsPolicy()
);

// Shutdown có trật tự
pool.shutdown();
if (!pool.awaitTermination(30, TimeUnit.SECONDS)) {
    pool.shutdownNow();
}
```

## 🔗 Mẫu dùng CompletableFuture ở mức hệ thống

- Fan-out/Fan-in: chạy song song nhiều nguồn rồi tổng hợp.
- Timeouts + fallback: bảo vệ chống dịch vụ chậm.
- Circuit-breaker/bulkhead (kết hợp với thư viện khác) để cô lập lỗi.

```java
Executor ioPool = Executors.newFixedThreadPool(32);

CompletableFuture<String> a = CompletableFuture.supplyAsync(() -> callA(), ioPool);
CompletableFuture<String> b = CompletableFuture.supplyAsync(() -> callB(), ioPool);

String combined = a.thenCombine(b, (ra, rb) -> merge(ra, rb))
    .orTimeout(800, TimeUnit.MILLISECONDS)
    .exceptionally(ex -> fallback())
    .join();

// allOf cho n tác vụ
CompletableFuture<Void> all = CompletableFuture.allOf(listOfFutures.toArray(new CompletableFuture[0]));
all.orTimeout(2, TimeUnit.SECONDS).join();
```

Lưu ý: `exceptionally/handle` cho phép phục hồi; `whenComplete` chỉ để quan sát.

## ⛔ Interruption & Cancellation — hủy việc an toàn

- Luôn tôn trọng cờ interrupt: vòng lặp dài cần `Thread.interrupted()` hoặc kiểm tra `isInterrupted()`.
- Không nuốt `InterruptedException`; nếu bắt, hãy khôi phục interrupt bằng `Thread.currentThread().interrupt()` hoặc truyền trạng thái lên trên.
- Đối với I/O blocking, dùng timeout, `Future.cancel(true)`, hoặc API hỗ trợ hủy.

```java
try {
    while (!Thread.currentThread().isInterrupted()) {
        step();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // khôi phục trạng thái
}
```

## 🔎 Chẩn đoán, giám sát và quan trắc

- Thread dumps (`jstack`, SIGQUIT) để thấy trạng thái, stacktrace, deadlock.
- Java Flight Recorder (JFR) / Mission Control để phân tích contention, block, wait.
- Log theo tên thread, dùng `MDC` để gắn `requestId` giúp lần theo luồng xử lý qua nhiều thread.
- Metric: hàng đợi, độ trễ, reject count, pooled thread đang hoạt động.

## 🧪 Kiểm thử đồng thời

- Dùng thời gian ảo/timers có thể kiểm soát; tránh `Thread.sleep` trong test.
- Awaitility cho điều kiện bất đồng bộ: `await().atMost(2, SECONDS).until(...)`.
- Test khả năng hủy, timeout, backpressure và chính sách từ chối.
- Fuzz/stress: chạy nhiều lần với tham số ngẫu nhiên để lộ race.

---

Phần mở rộng trên cung cấp nền tảng lý thuyết vững chắc (JMM, synchronizers, locks) cùng các mẫu triển khai thực tế (tuning executor, CompletableFuture, hủy an toàn, chẩn đoán). Bạn có thể áp dụng trực tiếp vào hệ thống I/O nặng, dịch vụ vi mô hoặc tác vụ tính toán song song ở môi trường production.
