---
title: "Bắt đầu với Java: Hướng dẫn chi tiết cho người mới"
date: 2024-10-16T09:00:00+07:00
draft: false
description: "Giới thiệu tổng quan về Java, cài đặt môi trường và viết chương trình đầu tiên"
category: "Java"
tags: ["Java", "Cơ bản", "Hello World", "JDK"]
readingTime: "10 phút"
author: "TG-Bao"
---

## 🚀 Java là gì? Tại sao nên học?

Java là ngôn ngữ lập trình hướng đối tượng, đa nền tảng ("Write once, run anywhere"), được dùng nhiều trong backend, Android, enterprise.

### 🧰 Cài đặt môi trường

1. Cài JDK (17 hoặc 21 LTS) từ `https://adoptium.net/`
2. Kiểm tra:
```bash
java -version
javac -version
```
3. Cài IDE: IntelliJ IDEA Community (khuyên dùng) hoặc VS Code + Extension Java.

### ✍️ Chương trình đầu tiên

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Xin chào Java!");
    }
}
```

Chạy bằng IDE, hoặc dòng lệnh:
```bash
javac Main.java
java Main
```

### 📦 Cấu trúc một project Maven tối giản

```text
my-app/
 ├─ pom.xml
 └─ src/main/java/com/example/App.java
```

`pom.xml` rút gọn:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0.0</version>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>
</project>
```

`App.java`:
```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello Maven Java");
    }
}
```

### 💡 Mẹo nhanh cho người mới
- Dùng `System.out.println` để in log khi học.
- Đặt tên class PascalCase, biến/hàm camelCase.
- Tạo package theo domain: `com.tenban.duan`.

### ✅ Bài tập nhỏ
Tạo app `Calculator` nhận 2 số từ args và in tổng, hiệu, tích, thương.


