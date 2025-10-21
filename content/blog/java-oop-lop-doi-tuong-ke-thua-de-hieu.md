---
title: "Java OOP: Lớp, Đối tượng và Kế thừa một cách dễ hiểu nhất"
date: 2025-09-03T11:00:00+07:00
draft: false
description: "Giải thích OOP trong Java với ví dụ rõ ràng: class, object, inheritance, override, polymorphism"
category: "Java"
tags: ["Java", "OOP", "Kế thừa", "Polymorphism"]
readingTime: "14 phút"
author: "TG-Bao"
---

## 🧱 Class & Object - Nền tảng của OOP

**Class là gì?**
Class là một blueprint (bản thiết kế) để tạo ra các object. Nó định nghĩa:
- **Attributes (Thuộc tính)**: Dữ liệu mà object sẽ có
- **Methods (Phương thức)**: Hành vi mà object có thể thực hiện
- **Constructor**: Phương thức đặc biệt để khởi tạo object

**Object là gì?**
Object là một instance (thể hiện) cụ thể của class. Mỗi object có:
- **State**: Giá trị hiện tại của các thuộc tính
- **Behavior**: Các phương thức có thể thực hiện
- **Identity**: Địa chỉ bộ nhớ duy nhất

```java
// Class definition - Bản thiết kế
class Animal {
    // Attributes (Thuộc tính)
    protected String name;
    protected int age;
    protected String species;
    
    // Constructor - Khởi tạo object
    public Animal(String name, int age, String species) {
        this.name = name;
        this.age = age;
        this.species = species;
    }
    
    // Methods (Phương thức)
    public void speak() {
        System.out.println(name + " makes a sound");
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Getter methods
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getSpecies() { return species; }
}

// Tạo objects từ class
public class OopDemo {
    public static void main(String[] args) {
        // Tạo các object cụ thể
        Animal animal1 = new Animal("Con vật", 5, "Unknown");
        Animal animal2 = new Animal("Con vật khác", 3, "Unknown");
        
        // Mỗi object có state riêng
        System.out.println("Animal 1: " + animal1.getName() + ", Age: " + animal1.getAge());
        System.out.println("Animal 2: " + animal2.getName() + ", Age: " + animal2.getAge());
        
        // Gọi methods
        animal1.speak();
        animal2.eat();
    }
}
```

**Tại sao cần OOP?**
- **Modularity**: Chia code thành các module độc lập
- **Reusability**: Tái sử dụng code thông qua inheritance
- **Maintainability**: Dễ bảo trì và mở rộng
- **Abstraction**: Ẩn chi tiết phức tạp, chỉ hiển thị interface cần thiết

## 🔁 Kế thừa (Inheritance) - Tái sử dụng code

**Inheritance là gì?**
Kế thừa cho phép một class (subclass/child class) kế thừa các thuộc tính và phương thức từ class khác (superclass/parent class). Điều này giúp:
- **Code Reusability**: Tái sử dụng code đã viết
- **Method Overriding**: Ghi đè phương thức của class cha
- **Polymorphism**: Đa hình - cùng một interface, nhiều implementation

```java
// Parent class (Superclass)
class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void speak() {
        System.out.println(name + " makes a sound");
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Getter methods
    public String getName() { return name; }
    public int getAge() { return age; }
}

// Child class (Subclass) - Kế thừa từ Animal
class Dog extends Animal {
    private String breed;
    
    // Constructor của subclass phải gọi super()
    public Dog(String name, int age, String breed) {
        super(name, age); // Gọi constructor của parent class
        this.breed = breed;
    }
    
    // Method Overriding - Ghi đè phương thức của parent
    @Override
    public void speak() {
        System.out.println(name + ": Gâu gâu!");
    }
    
    // Method mới chỉ có ở Dog
    public void wagTail() {
        System.out.println(name + " is wagging tail");
    }
    
    public String getBreed() { return breed; }
}

// Child class khác
class Cat extends Animal {
    private boolean isIndoor;
    
    public Cat(String name, int age, boolean isIndoor) {
        super(name, age);
        this.isIndoor = isIndoor;
    }
    
    @Override
    public void speak() {
        System.out.println(name + ": Meo meo!");
    }
    
    public void climb() {
        System.out.println(name + " is climbing");
    }
    
    public boolean isIndoor() { return isIndoor; }
}

// Demo Inheritance
public class InheritanceDemo {
    public static void main(String[] args) {
        // Tạo objects
        Animal animal = new Animal("Con vật", 5);
        Dog dog = new Dog("Milu", 3, "Golden Retriever");
        Cat cat = new Cat("Mimi", 2, true);
        
        // Gọi methods
        animal.speak(); // "Con vật makes a sound"
        dog.speak();    // "Milu: Gâu gâu!" (overridden)
        cat.speak();    // "Mimi: Meo meo!" (overridden)
        
        // Methods chỉ có ở subclass
        dog.wagTail();  // "Milu is wagging tail"
        cat.climb();    // "Mimi is climbing"
        
        // Polymorphism - Đa hình
        Animal[] animals = {animal, dog, cat};
        for (Animal a : animals) {
            a.speak(); // Mỗi loại sẽ gọi phương thức riêng
        }
    }
}
```

**Các loại kế thừa:**
- **Single Inheritance**: Java chỉ hỗ trợ kế thừa đơn (một class chỉ kế thừa từ một class cha)
- **Multiple Inheritance**: Không được hỗ trợ trực tiếp, nhưng có thể dùng interfaces
- **Multilevel Inheritance**: Class A → Class B → Class C
- **Hierarchical Inheritance**: Một class cha có nhiều class con

**Method Overriding vs Method Overloading:**
- **Overriding**: Ghi đè phương thức của class cha (cùng tên, cùng tham số)
- **Overloading**: Tạo nhiều phương thức cùng tên nhưng khác tham số

## 🔒 Đóng gói (Encapsulation) - Bảo vệ dữ liệu

**Encapsulation là gì?**
Đóng gói là nguyên tắc ẩn dữ liệu bên trong object và chỉ cho phép truy cập thông qua các phương thức công khai. Điều này giúp:
- **Data Hiding**: Ẩn chi tiết implementation
- **Data Validation**: Kiểm tra dữ liệu trước khi lưu
- **Maintainability**: Dễ thay đổi implementation mà không ảnh hưởng code khác

**Access Modifiers:**
- **private**: Chỉ truy cập trong cùng class
- **protected**: Truy cập trong cùng class và subclass
- **public**: Truy cập từ mọi nơi
- **default (package-private)**: Truy cập trong cùng package

```java
class User {
    // Private fields - Ẩn dữ liệu
    private String name;
    private String email;
    private int age;
    private String password;
    
    // Constructor
    public User(String name, String email, int age) {
        this.name = name;
        this.setEmail(email); // Sử dụng setter để validation
        this.setAge(age);
    }
    
    // Getter methods - Đọc dữ liệu
    public String getName() { 
        return name; 
    }
    
    public String getEmail() { 
        return email; 
    }
    
    public int getAge() { 
        return age; 
    }
    
    // Setter methods - Ghi dữ liệu với validation
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Tên không được để trống");
        }
        this.name = name;
    }
    
    public void setEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email không hợp lệ");
        }
        this.email = email;
    }
    
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Tuổi không hợp lệ");
        }
        this.age = age;
    }
    
    // Business methods
    public void changePassword(String oldPassword, String newPassword) {
        if (!this.password.equals(oldPassword)) {
            throw new IllegalArgumentException("Mật khẩu cũ không đúng");
        }
        if (newPassword.length() < 6) {
            throw new IllegalArgumentException("Mật khẩu mới phải có ít nhất 6 ký tự");
        }
        this.password = newPassword;
    }
    
    public boolean isAdult() {
        return age >= 18;
    }
    
    // toString method
    @Override
    public String toString() {
        return "User{name='" + name + "', email='" + email + "', age=" + age + "}";
    }
}

// Demo Encapsulation
public class EncapsulationDemo {
    public static void main(String[] args) {
        try {
            User user = new User("John Doe", "john@example.com", 25);
            
            // Truy cập thông qua getter
            System.out.println("Name: " + user.getName());
            System.out.println("Email: " + user.getEmail());
            System.out.println("Age: " + user.getAge());
            
            // Thay đổi thông qua setter (có validation)
            user.setAge(26);
            System.out.println("Updated age: " + user.getAge());
            
            // Kiểm tra business logic
            System.out.println("Is adult: " + user.isAdult());
            
            // Thử set dữ liệu không hợp lệ
            user.setEmail("invalid-email"); // Sẽ throw exception
            
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Lợi ích của Encapsulation:**
- **Data Integrity**: Đảm bảo dữ liệu luôn hợp lệ
- **Flexibility**: Có thể thay đổi implementation mà không ảnh hưởng client code
- **Security**: Ngăn chặn truy cập trực tiếp vào dữ liệu nhạy cảm
- **Maintainability**: Dễ debug và maintain code

## 🧩 Trừu tượng (Abstraction) - Ẩn chi tiết phức tạp

**Abstraction là gì?**
Trừu tượng là nguyên tắc ẩn chi tiết implementation phức tạp và chỉ hiển thị những gì cần thiết cho người dùng. Có 2 cách thực hiện:
- **Abstract Class**: Class không thể tạo object trực tiếp
- **Interface**: Contract định nghĩa các method mà class phải implement

**Abstract Class:**
```java
// Abstract class - không thể tạo object trực tiếp
abstract class Shape {
    protected String color;
    protected boolean filled;
    
    public Shape(String color, boolean filled) {
        this.color = color;
        this.filled = filled;
    }
    
    // Abstract method - phải được implement bởi subclass
    public abstract double area();
    public abstract double perimeter();
    
    // Concrete method - có implementation
    public String getColor() { return color; }
    public boolean isFilled() { return filled; }
    
    public void setColor(String color) { this.color = color; }
    public void setFilled(boolean filled) { this.filled = filled; }
    
    @Override
    public String toString() {
        return "Shape{color='" + color + "', filled=" + filled + "}";
    }
}

// Concrete class - implement abstract methods
class Circle extends Shape {
    private double radius;
    
    public Circle(String color, boolean filled, double radius) {
        super(color, filled);
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
    
    public double getRadius() { return radius; }
    public void setRadius(double radius) { this.radius = radius; }
}

class Rectangle extends Shape {
    private double width;
    private double height;
    
    public Rectangle(String color, boolean filled, double width, double height) {
        super(color, filled);
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
    
    @Override
    public double perimeter() {
        return 2 * (width + height);
    }
    
    public double getWidth() { return width; }
    public double getHeight() { return height; }
    public void setWidth(double width) { this.width = width; }
    public void setHeight(double height) { this.height = height; }
}
```

**Interface:**
```java
// Interface - định nghĩa contract
interface Drawable {
    void draw();
    void erase();
}

interface Movable {
    void move(int x, int y);
    void stop();
}

// Class có thể implement nhiều interface
class Circle extends Shape implements Drawable, Movable {
    private double radius;
    private int x, y;
    
    public Circle(String color, boolean filled, double radius, int x, int y) {
        super(color, filled);
        this.radius = radius;
        this.x = x;
        this.y = y;
    }
    
    // Implement abstract methods từ Shape
    @Override
    public double area() { return Math.PI * radius * radius; }
    
    @Override
    public double perimeter() { return 2 * Math.PI * radius; }
    
    // Implement methods từ Drawable
    @Override
    public void draw() {
        System.out.println("Drawing circle at (" + x + ", " + y + ")");
    }
    
    @Override
    public void erase() {
        System.out.println("Erasing circle");
    }
    
    // Implement methods từ Movable
    @Override
    public void move(int newX, int newY) {
        this.x = newX;
        this.y = newY;
        System.out.println("Circle moved to (" + x + ", " + y + ")");
    }
    
    @Override
    public void stop() {
        System.out.println("Circle stopped");
    }
}
```

## 🎭 Đa hình (Polymorphism) - Một interface, nhiều implementation

**Polymorphism là gì?**
Đa hình cho phép cùng một interface có nhiều implementation khác nhau. Java hỗ trợ 2 loại:
- **Runtime Polymorphism**: Method overriding
- **Compile-time Polymorphism**: Method overloading

```java
// Demo Polymorphism
public class PolymorphismDemo {
    public static void main(String[] args) {
        // Tạo array chứa các Shape khác nhau
        Shape[] shapes = {
            new Circle("Red", true, 5.0),
            new Rectangle("Blue", true, 4.0, 6.0),
            new Circle("Green", false, 3.0)
        };
        
        // Polymorphism - cùng method, khác behavior
        for (Shape shape : shapes) {
            System.out.println("Shape: " + shape.getClass().getSimpleName());
            System.out.println("Area: " + shape.area());
            System.out.println("Perimeter: " + shape.perimeter());
            System.out.println("---");
        }
        
        // Polymorphism với interface
        Drawable[] drawables = {
            new Circle("Red", true, 5.0, 10, 20),
            new Rectangle("Blue", true, 4.0, 6.0)
        };
        
        for (Drawable drawable : drawables) {
            drawable.draw();
            drawable.erase();
        }
    }
}
```

## ✅ Bài tập thực hành: Hệ thống Vehicle

```java
// Abstract class Vehicle
abstract class Vehicle {
    protected String brand;
    protected String model;
    protected int year;
    protected double speed;
    
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.speed = 0;
    }
    
    // Abstract methods - phải implement
    public abstract void start();
    public abstract void stop();
    public abstract void accelerate(double amount);
    public abstract void brake(double amount);
    
    // Concrete methods
    public void move() {
        System.out.println(brand + " " + model + " is moving at " + speed + " km/h");
    }
    
    public String getInfo() {
        return brand + " " + model + " (" + year + ")";
    }
    
    public double getSpeed() { return speed; }
    public void setSpeed(double speed) { this.speed = speed; }
}

// Concrete class Car
class Car extends Vehicle {
    private int doors;
    private String fuelType;
    
    public Car(String brand, String model, int year, int doors, String fuelType) {
        super(brand, model, year);
        this.doors = doors;
        this.fuelType = fuelType;
    }
    
    @Override
    public void start() {
        System.out.println("Car engine started");
    }
    
    @Override
    public void stop() {
        System.out.println("Car engine stopped");
        speed = 0;
    }
    
    @Override
    public void accelerate(double amount) {
        speed += amount;
        System.out.println("Car accelerated to " + speed + " km/h");
    }
    
    @Override
    public void brake(double amount) {
        speed = Math.max(0, speed - amount);
        System.out.println("Car braked to " + speed + " km/h");
    }
    
    public void honk() {
        System.out.println("Beep beep!");
    }
}

// Concrete class Bike
class Bike extends Vehicle {
    private int gears;
    private boolean hasBasket;
    
    public Bike(String brand, String model, int year, int gears, boolean hasBasket) {
        super(brand, model, year);
        this.gears = gears;
        this.hasBasket = hasBasket;
    }
    
    @Override
    public void start() {
        System.out.println("Bike pedaling started");
    }
    
    @Override
    public void stop() {
        System.out.println("Bike stopped pedaling");
        speed = 0;
    }
    
    @Override
    public void accelerate(double amount) {
        speed += amount;
        System.out.println("Bike pedaled faster to " + speed + " km/h");
    }
    
    @Override
    public void brake(double amount) {
        speed = Math.max(0, speed - amount);
        System.out.println("Bike braked to " + speed + " km/h");
    }
    
    public void ringBell() {
        System.out.println("Ring ring!");
    }
}

// Demo hệ thống Vehicle
public class VehicleDemo {
    public static void main(String[] args) {
        // Tạo các vehicle khác nhau
        Vehicle[] vehicles = {
            new Car("Toyota", "Camry", 2023, 4, "Gasoline"),
            new Bike("Giant", "Escape", 2023, 21, true),
            new Car("Tesla", "Model 3", 2023, 4, "Electric")
        };
        
        // Polymorphism - cùng interface, khác behavior
        for (Vehicle vehicle : vehicles) {
            System.out.println("\n=== " + vehicle.getInfo() + " ===");
            vehicle.start();
            vehicle.accelerate(50);
            vehicle.move();
            vehicle.brake(20);
            vehicle.stop();
            
            // Cast để gọi method riêng
            if (vehicle instanceof Car) {
                ((Car) vehicle).honk();
            } else if (vehicle instanceof Bike) {
                ((Bike) vehicle).ringBell();
            }
        }
    }
}
```

## 🎯 Kết luận

OOP trong Java cung cấp 4 nguyên tắc cơ bản:
1. **Encapsulation**: Bảo vệ dữ liệu
2. **Inheritance**: Tái sử dụng code
3. **Abstraction**: Ẩn chi tiết phức tạp
4. **Polymorphism**: Một interface, nhiều implementation

**Best Practices:**
- Sử dụng `private` cho fields, `public` cho methods cần thiết
- Ưu tiên composition over inheritance
- Sử dụng `@Override` annotation
- Thiết kế interface nhỏ và focused
- Tránh deep inheritance hierarchy


