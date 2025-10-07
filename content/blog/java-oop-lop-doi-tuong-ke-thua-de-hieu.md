---
title: "Java OOP: Lớp, Đối tượng và Kế thừa một cách dễ hiểu nhất"
date: 2024-10-16T11:00:00+07:00
draft: false
description: "Giải thích OOP trong Java với ví dụ rõ ràng: class, object, inheritance, override, polymorphism"
category: "Java"
tags: ["Java", "OOP", "Kế thừa", "Polymorphism"]
readingTime: "14 phút"
author: "TG-Bao"
---

## 🧱 Class & Object

```java
class Animal {
    protected String name;
    public Animal(String name) { this.name = name; }
    public void speak() { System.out.println(name + " makes a sound"); }
}

class Dog extends Animal {
    public Dog(String name) { super(name); }
    @Override
    public void speak() { System.out.println(name + ": Gâu gâu!"); }
}

public class OopDemo {
    public static void main(String[] args) {
        Animal a = new Animal("Con vật");
        Animal d = new Dog("Milu"); // đa hình
        a.speak();
        d.speak(); // gọi phiên bản override ở Dog
    }
}
```

## 🔁 Kế thừa & Override

- `extends` để kế thừa; `super()` gọi constructor cha
- `@Override` đảm bảo chữ ký trùng khớp

## 🔒 Đóng gói (Encapsulation)

```java
class User {
    private String email;
    public String getEmail() { return email; }
    public void setEmail(String email) {
        if (!email.contains("@")) throw new IllegalArgumentException("Email không hợp lệ");
        this.email = email;
    }
}
```

## 🧩 Trừu tượng (Abstraction)

```java
abstract class Shape { abstract double area(); }
class Circle extends Shape { double r; Circle(double r){ this.r = r; } double area(){ return Math.PI * r * r; } }
class Rect extends Shape { double w,h; Rect(double w,double h){ this.w=w; this.h=h; } double area(){ return w*h; } }
```

## ✅ Bài tập
Tạo hệ thống `Vehicle` (class trừu tượng) với `Car`, `Bike` kế thừa; thêm phương thức `move()` và thuộc tính cụ thể; minh họa đa hình.


