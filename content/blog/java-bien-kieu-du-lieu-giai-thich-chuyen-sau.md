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

## 🔤 Kiểu dữ liệu nguyên thủy vs tham chiếu - Hiểu sâu về Memory Model

**Kiểu dữ liệu nguyên thủy (Primitive Types):**
Java có 8 kiểu dữ liệu nguyên thủy được thiết kế để tối ưu hiệu suất và sử dụng bộ nhớ:

1. **Integer Types**: `byte`, `short`, `int`, `long`
2. **Floating-point Types**: `float`, `double`
3. **Character Type**: `char`
4. **Boolean Type**: `boolean`

**Đặc điểm của Primitive Types:**

- **Lưu trực tiếp giá trị** trong bộ nhớ stack
- **Không phải object** - không có methods
- **Hiệu suất cao** - không có overhead của object
- **Giá trị mặc định** được gán tự động (0, false, '\u0000')

**Kiểu dữ liệu tham chiếu (Reference Types):**
Tất cả các kiểu không phải primitive đều là reference types:

- **String**: Chuỗi ký tự
- **Array**: Mảng các phần tử
- **Class**: Các class do bạn định nghĩa
- **Interface**: Các interface
- **Enum**: Các enum type

**Đặc điểm của Reference Types:**

- **Lưu địa chỉ** (reference) đến object trong heap
- **Có thể null** - không trỏ đến object nào
- **Có methods** và có thể gọi các phương thức
- **Giá trị mặc định là null**

```java
// Primitive types - lưu trực tiếp giá trị
int age = 25;                    // 4 bytes, giá trị: 25
double height = 1.75;            // 8 bytes, giá trị: 1.75
boolean isStudent = true;        // 1 bit, giá trị: true
char grade = 'A';                // 2 bytes, giá trị: 'A'

// Reference types - lưu địa chỉ đến object
String name = "Bao";             // Biến name chứa địa chỉ đến String object
int[] numbers = {1, 2, 3};       // Biến numbers chứa địa chỉ đến array object
Object obj = new Object();       // Biến obj chứa địa chỉ đến Object instance

// So sánh để hiểu rõ sự khác biệt
int a = 10;
int b = a;                       // b có giá trị 10 (copy giá trị)
b = 20;                          // a vẫn là 10, b là 20

String str1 = "Hello";
String str2 = str1;              // str2 trỏ đến cùng object với str1
str2 = "World";                  // str1 vẫn là "Hello", str2 trỏ đến "World"
```

**Tại sao Java có cả Primitive và Reference Types?**

- **Performance**: Primitive types nhanh hơn, ít tốn bộ nhớ hơn
- **Simplicity**: Một số operations đơn giản hơn với primitive
- **Backward Compatibility**: Giữ nguyên thiết kế từ Java 1.0
- **Memory Efficiency**: Tránh overhead của object wrapper

## 📐 Phạm vi biến (Variable Scope) - Quy tắc truy cập

**Variable Scope là gì?**
Phạm vi biến (scope) định nghĩa vùng code mà biến có thể được truy cập và sử dụng. Java có 4 loại scope chính:

1. **Class Scope (Static Variables)**: Biến thuộc về class, tồn tại trong suốt lifecycle của class
2. **Instance Scope (Instance Variables)**: Biến thuộc về instance, tồn tại trong suốt lifecycle của object
3. **Method Scope (Local Variables)**: Biến chỉ tồn tại trong method
4. **Block Scope**: Biến chỉ tồn tại trong block code `{}`

**Chi tiết từng loại scope:**

```java
public class ScopeDemo {
    // Class Scope - Static variables
    static int staticVar = 10;           // Tồn tại trong suốt chương trình
    static final String CONSTANT = "ABC"; // Constant - không thể thay đổi

    // Instance Scope - Instance variables
    private String instanceVar = "Instance"; // Tồn tại trong suốt lifecycle của object
    private int instanceNumber = 100;

    public static void main(String[] args) {
        // Method Scope - Local variables
        int methodVar = 5;               // Chỉ tồn tại trong method main
        String methodString = "Local";

        System.out.println("Static var: " + staticVar);
        System.out.println("Method var: " + methodVar);

        // Block Scope
        if (methodVar > 0) {
            int blockVar = 7;            // Chỉ tồn tại trong block if
            System.out.println("Block var: " + blockVar);
            System.out.println("Method var accessible: " + methodVar);
        }
        // System.out.println(blockVar); // Lỗi: blockVar không tồn tại ở đây

        // Tạo instance để truy cập instance variables
        ScopeDemo demo = new ScopeDemo();
        System.out.println("Instance var: " + demo.instanceVar);
    }

    public void instanceMethod() {
        // Method scope trong instance method
        int localVar = 20;

        // Có thể truy cập instance variables
        System.out.println("Instance var: " + instanceVar);
        System.out.println("Local var: " + localVar);

        // Có thể truy cập static variables
        System.out.println("Static var: " + staticVar);
    }
}
```

**Quy tắc Shadowing:**
Khi biến local có cùng tên với biến ở scope cao hơn, biến local sẽ "che" (shadow) biến ở scope cao hơn.

```java
public class ShadowingDemo {
    static int x = 10;  // Class scope

    public static void main(String[] args) {
        int x = 20;     // Method scope - shadow class variable

        System.out.println("Local x: " + x);        // 20
        System.out.println("Class x: " + ShadowingDemo.x); // 10

        if (x > 0) {
            int x = 30; // Block scope - shadow method variable
            System.out.println("Block x: " + x);    // 30
        }
    }
}
```

**Best Practices cho Variable Scope:**

- **Minimize scope**: Khai báo biến ở scope nhỏ nhất có thể
- **Avoid shadowing**: Tránh đặt tên biến trùng với biến ở scope cao hơn
- **Use meaningful names**: Đặt tên biến có ý nghĩa
- **Initialize variables**: Luôn khởi tạo giá trị cho biến

## 🧠 Auto-boxing và Unboxing - Cầu nối giữa Primitive và Wrapper

**Auto-boxing và Unboxing là gì?**
Auto-boxing và Unboxing là tính năng tự động chuyển đổi giữa primitive types và wrapper classes trong Java (từ Java 5). Điều này giúp code trở nên ngắn gọn và dễ đọc hơn.

**Wrapper Classes:**
Mỗi primitive type đều có một wrapper class tương ứng:

- `byte` → `Byte`
- `short` → `Short`
- `int` → `Integer`
- `long` → `Long`
- `float` → `Float`
- `double` → `Double`
- `char` → `Character`
- `boolean` → `Boolean`

**Auto-boxing:**
Tự động chuyển đổi từ primitive type sang wrapper class.

```java
// Trước Java 5 - Manual boxing
Integer boxed1 = new Integer(10);        // Manual boxing
Integer boxed2 = Integer.valueOf(20);    // Manual boxing

// Từ Java 5 - Auto-boxing
Integer boxed3 = 10;                      // Auto-boxing: int -> Integer
Integer boxed4 = 20;                      // Auto-boxing: int -> Integer

// Auto-boxing trong method calls
List<Integer> numbers = new ArrayList<>();
numbers.add(1);                           // Auto-boxing: int -> Integer
numbers.add(2);                           // Auto-boxing: int -> Integer
numbers.add(3);                           // Auto-boxing: int -> Integer

// Auto-boxing trong expressions
Integer result = 10 + 20;                 // Auto-boxing kết quả
```

**Unboxing:**
Tự động chuyển đổi từ wrapper class sang primitive type.

```java
// Trước Java 5 - Manual unboxing
Integer boxed = new Integer(10);
int primitive1 = boxed.intValue();        // Manual unboxing

// Từ Java 5 - Auto-unboxing
Integer boxed = 10;
int primitive2 = boxed;                   // Auto-unboxing: Integer -> int

// Auto-unboxing trong expressions
Integer a = 10;
Integer b = 20;
int sum = a + b;                          // Auto-unboxing a và b, auto-boxing kết quả

// Auto-unboxing trong method calls
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int first = numbers.get(0);              // Auto-unboxing: Integer -> int
```

**Các trường hợp sử dụng Auto-boxing/Unboxing:**

```java
public class BoxingDemo {
    public static void main(String[] args) {
        // 1. Collections với primitive values
        List<Integer> integers = new ArrayList<>();
        integers.add(1);                  // Auto-boxing
        integers.add(2);                  // Auto-boxing

        int first = integers.get(0);      // Auto-unboxing

        // 2. Generic types
        Map<String, Integer> map = new HashMap<>();
        map.put("age", 25);               // Auto-boxing
        int age = map.get("age");         // Auto-unboxing

        // 3. Method parameters và return values
        Integer result = calculateSum(10, 20); // Auto-boxing return value
        int sum = result;                 // Auto-unboxing

        // 4. Conditional expressions
        Integer nullable = getValue();
        int value = nullable != null ? nullable : 0; // Auto-unboxing

        // 5. Arithmetic operations
        Integer a = 10;
        Integer b = 20;
        Integer c = a + b;                // Auto-unboxing, auto-boxing result
    }

    public static Integer calculateSum(int a, int b) {
        return a + b;                     // Auto-boxing return value
    }

    public static Integer getValue() {
        return Math.random() > 0.5 ? 100 : null;
    }
}
```

**Performance Considerations:**
Auto-boxing/unboxing có overhead về performance, đặc biệt trong loops:

```java
// Không tốt - Auto-boxing trong loop
List<Integer> numbers = new ArrayList<>();
for (int i = 0; i < 1000000; i++) {
    numbers.add(i);                       // Auto-boxing mỗi lần lặp
}

// Tốt hơn - Sử dụng primitive array
int[] numbers = new int[1000000];
for (int i = 0; i < 1000000; i++) {
    numbers[i] = i;                       // Không có boxing/unboxing
}

// Hoặc sử dụng specialized collections
IntList numbers = new IntArrayList();     // Trove4j library
for (int i = 0; i < 1000000; i++) {
    numbers.add(i);                       // Không có boxing/unboxing
}
```

**Null Pointer Exception với Auto-unboxing:**

```java
Integer nullable = null;
int value = nullable;                    // NullPointerException!

// Cách tránh
int value = nullable != null ? nullable : 0;
// Hoặc
int value = Optional.ofNullable(nullable).orElse(0);
```

## 🧮 Kích thước & miền giá trị (tóm tắt)

| Kiểu | Byte | Min   | Max    |
| ---- | ---- | ----- | ------ |
| byte | 1    | -128  | 127    |
| int  | 4    | -2^31 | 2^31-1 |
| long | 8    | -2^63 | 2^63-1 |

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
