---
title: "Quản lý lỗi và Exception trong Java: Bí quyết tránh crash"
date: 2024-10-16T13:00:00+07:00
draft: false
description: "try-catch, throws, custom exception, finally và best practices để app bền vững"
category: "Java"
tags: ["Java", "Exception", "Error Handling"]
readingTime: "14 phút"
author: "TG-Bao"
---

## 🧯 try-catch cơ bản

```java
public class TryCatchDemo {
    public static int parseIntSafe(String s) {
        try {
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {
            return -1; // fallback
        } finally {
            // dọn dẹp tài nguyên nếu có
        }
    }
}
```

## 📤 throws & rethrow

```java
void readFile(Path p) throws IOException {
    Files.readAllLines(p); // có thể ném IOException
}

void service(Path p) {
    try {
        readFile(p);
    } catch (IOException e) {
        throw new RuntimeException("Đọc file thất bại", e); // wrap để log stack gốc
    }
}
```

## 🧰 Custom Exception

```java
class BusinessException extends RuntimeException {
    public BusinessException(String message) { super(message); }
}

class UserService {
    void register(String email) {
        if (!email.contains("@")) throw new BusinessException("Email không hợp lệ");
    }
}
```

## ✅ Best practices
- Bắt đúng ngoại lệ, không catch chung `Exception` trừ lớp boundary.
- Ghi log kèm context; giữ nguyên `cause` khi wrap.
- Dọn dẹp tài nguyên bằng try-with-resources.

```java
try (var in = Files.newInputStream(Path.of("data.txt"))) {
    // xử lý
}
```


