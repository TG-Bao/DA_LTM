---
title: "Hiểu sâu về Biến và Kiểu Dữ liệu trong Java"
date: 2025-09-01T10:00:00+07:00
draft: false
description: "Biến, phạm vi, kiểu nguyên thủy, tham chiếu và cách quản lý bộ nhớ cơ bản"
category: "Java"
tags: ["Java", "Kiểu dữ liệu", "Biến", "Memory"]
readingTime: "12 phút"
author: "TG-Bao"
---

## 🔤 Kiểu dữ liệu nguyên thủy vs tham chiếu

- Nguyên thủy: `byte, short, int, long, float, double, char, boolean`
- Tham chiếu: `String`, `Array`, `Class`, `Object`, các class do bạn định nghĩa

```java
int age = 25;            // primitive - lưu trực tiếp giá trị
String name = "Bao";    // reference - biến giữ tham chiếu đến đối tượng String
```

## 📐 Phạm vi biến

```java
public class ScopeDemo {
    static int staticVar = 10; // phạm vi class

    public static void main(String[] args) {
        int x = 5;             // phạm vi block method
        if (x > 0) {
            int y = 7;         // phạm vi block if
            System.out.println(x + y);
        }
        // System.out.println(y); // lỗi: y không tồn tại ở đây
    }
}
```

## 🧠 Auto-boxing, Unboxing

```java
Integer boxed = 10;     // auto-boxing: int -> Integer
int unboxed = boxed;    // unboxing: Integer -> int
```

## 🧮 Kích thước & miền giá trị (tóm tắt)

| Kiểu | Byte | Min | Max |
|-----|------|-----|-----|
| byte | 1 | -128 | 127 |
| int | 4 | -2^31 | 2^31-1 |
| long | 8 | -2^63 | 2^63-1 |

## 🗃️ Quản lý bộ nhớ cơ bản

- Stack: lưu biến cục bộ, reference
- Heap: lưu object; GC sẽ thu hồi khi không còn tham chiếu

```java
public class MemoryDemo {
    static class User { String name; User(String n){ this.name = n; } }
    public static void main(String[] args) {
        User u1 = new User("Bao"); // object ở heap, u1 ở stack tham chiếu đến
        u1 = null;                  // object trở thành ứng viên GC
    }
}
```

## ✅ Bài tập
Viết chương trình đọc tên, tuổi (int), chiều cao (double) và in ra định dạng đẹp; thử chuyển đổi giữa wrapper và primitive.


