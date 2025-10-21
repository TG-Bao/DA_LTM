---
title: "Xử lý Collections trong Java: List, Set và Map"
date: 2025-09-05T12:00:00+07:00
draft: false
description: "Hướng dẫn sử dụng List, Set, Map; so sánh ưu nhược điểm với ví dụ thực tế"
category: "Java"
tags: ["Java", "Collections", "List", "Set", "Map"]
readingTime: "15 phút"
author: "TG-Bao"
---

## 🧺 Tổng quan về Java Collections Framework

**Collections Framework là gì?**
Java Collections Framework (JCF) là một kiến trúc thống nhất để đại diện và thao tác với các collections. Nó được thiết kế để:
- Giảm thiểu effort lập trình
- Tăng hiệu suất và chất lượng code
- Thúc đẩy tái sử dụng code
- Cho phép interoperability giữa các APIs

**Kiến trúc Collections Framework:**

```
Collection Interface
├── List Interface (có thứ tự, cho phép trùng)
│   ├── ArrayList (array-based, random access)
│   ├── LinkedList (doubly-linked list)
│   └── Vector (synchronized, legacy)
├── Set Interface (không trùng, không đảm bảo thứ tự)
│   ├── HashSet (hash table)
│   ├── LinkedHashSet (hash table + linked list)
│   └── TreeSet (red-black tree)
└── Queue Interface (FIFO)
    ├── PriorityQueue (heap)
    └── Deque (double-ended queue)

Map Interface (key-value pairs)
├── HashMap (hash table)
├── LinkedHashMap (hash table + linked list)
└── TreeMap (red-black tree)
```

**Ví dụ cơ bản:**

```java
import java.util.*;

public class CollectionsDemo {
    public static void main(String[] args) {
        // List: có thứ tự, cho phép trùng
        List<String> list = new ArrayList<>();
        list.add("A"); list.add("B"); list.add("A");

        // Set: không trùng, không đảm bảo thứ tự
        Set<String> set = new HashSet<>(list); // loại trùng

        // Map: key-value, key không trùng
        Map<String, Integer> map = new HashMap<>();
        map.put("A", 1); map.put("B", 2);

        System.out.println(list); // [A, B, A]
        System.out.println(set);  // [A, B]
        System.out.println(map);  // {A=1, B=2}
    }
}
```

**Tại sao cần Collections Framework?**
- **Trước JCF**: Developers phải tự implement data structures
- **Với JCF**: Có sẵn các implementation được tối ưu hóa
- **Lợi ích**: Code ngắn gọn, hiệu suất cao, dễ maintain

## ⚖️ So sánh chi tiết các Collections

**Bảng so sánh hiệu suất:**

| Cấu trúc | Trùng | Thứ tự | Tìm kiếm | Insert | Delete | Use Case |
|---------|------|--------|---------|--------|--------|----------|
| **ArrayList** | ✔ | theo index | O(1) random access | O(1) end, O(n) middle | O(n) | Random access, ít insert/delete |
| **LinkedList** | ✔ | theo chèn | O(n) | O(1) | O(1) nếu có reference | Frequent insert/delete |
| **HashSet** | ✖ | không | O(1) trung bình | O(1) | O(1) | Unique elements, fast lookup |
| **TreeSet** | ✖ | sắp xếp | O(log n) | O(log n) | O(log n) | Sorted unique elements |
| **HashMap** | key ✖ | không | O(1) trung bình | O(1) | O(1) | Key-value mapping |
| **TreeMap** | key ✖ | sắp xếp | O(log n) | O(log n) | O(log n) | Sorted key-value mapping |

**Khi nào dùng cái gì?**

**List Implementations:**
- **ArrayList**: Khi cần random access, ít thay đổi kích thước
- **LinkedList**: Khi cần frequent insert/delete ở đầu/cuối
- **Vector**: Legacy, synchronized (thread-safe nhưng chậm)

**Set Implementations:**
- **HashSet**: Khi cần unique elements, không quan tâm thứ tự
- **LinkedHashSet**: Khi cần unique + giữ thứ tự insertion
- **TreeSet**: Khi cần unique + sorted elements

**Map Implementations:**
- **HashMap**: Khi cần key-value mapping, không quan tâm thứ tự
- **LinkedHashMap**: Khi cần key-value + giữ thứ tự insertion
- **TreeMap**: Khi cần key-value + sorted by key
- **Hashtable**: Legacy, synchronized (thread-safe nhưng chậm)

## 🧪 Ví dụ thực tế: Hệ thống quản lý sinh viên

**Bài toán**: Xây dựng hệ thống quản lý sinh viên với các chức năng:
- Thêm/xóa/sửa thông tin sinh viên
- Tìm kiếm sinh viên theo tên
- Thống kê điểm trung bình
- Sắp xếp sinh viên theo điểm

```java
import java.util.*;

public class StudentManagementSystem {
    // Sử dụng HashMap để lưu trữ sinh viên (ID -> Student)
    private Map<String, Student> students = new HashMap<>();
    
    // Sử dụng TreeSet để sắp xếp sinh viên theo điểm
    private Set<Student> sortedByGrade = new TreeSet<>(
        Comparator.comparing(Student::getAverageGrade).reversed()
    );
    
    public void addStudent(String id, String name, double grade) {
        Student student = new Student(id, name, grade);
        students.put(id, student);
        sortedByGrade.add(student);
    }
    
    public Student findStudent(String id) {
        return students.get(id);
    }
    
    public List<Student> getTopStudents(int n) {
        return sortedByGrade.stream()
            .limit(n)
            .collect(Collectors.toList());
    }
    
    public Map<String, Integer> getGradeDistribution() {
        Map<String, Integer> distribution = new HashMap<>();
        for (Student student : students.values()) {
            String gradeLevel = getGradeLevel(student.getAverageGrade());
            distribution.put(gradeLevel, 
                distribution.getOrDefault(gradeLevel, 0) + 1);
        }
        return distribution;
    }
    
    private String getGradeLevel(double grade) {
        if (grade >= 8.0) return "Giỏi";
        if (grade >= 6.5) return "Khá";
        if (grade >= 5.0) return "Trung bình";
        return "Yếu";
    }
}

class Student {
    private String id, name;
    private double averageGrade;
    
    public Student(String id, String name, double averageGrade) {
        this.id = id;
        this.name = name;
        this.averageGrade = averageGrade;
    }
    
    // Getters, equals, hashCode...
}
```

**Phân tích lựa chọn Collections:**

1. **HashMap cho students**: O(1) lookup theo ID
2. **TreeSet cho sortedByGrade**: Tự động sắp xếp, O(log n) insert
3. **Stream API**: Functional programming, dễ đọc

## 🎯 Best Practices khi sử dụng Collections

**1. Chọn đúng Collection cho use case:**
```java
// ❌ Sai: Dùng ArrayList khi cần frequent lookup
List<String> names = new ArrayList<>();
if (names.contains("John")) { ... } // O(n)

// ✅ Đúng: Dùng HashSet cho fast lookup
Set<String> names = new HashSet<>();
if (names.contains("John")) { ... } // O(1)
```

**2. Sử dụng Generic types:**
```java
// ❌ Sai: Raw types
List list = new ArrayList();

// ✅ Đúng: Generic types
List<String> list = new ArrayList<>();
```

**3. Sử dụng Collections utility methods:**
```java
// Thay vì tự implement
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

**4. Thread Safety:**
```java
// Single-threaded
List<String> list = new ArrayList<>();

// Multi-threaded
List<String> list = Collections.synchronizedList(new ArrayList<>());
// Hoặc
List<String> list = new CopyOnWriteArrayList<>();
```

## ✅ Bài tập thực hành

**Bài 1: E-commerce System**
- Sử dụng `HashMap` để lưu giỏ hàng (Product ID -> Quantity)
- Sử dụng `TreeMap` để hiển thị sản phẩm theo giá
- Sử dụng `HashSet` để lưu danh sách sản phẩm đã xem

**Bài 2: Social Media Feed**
- Sử dụng `LinkedList` để lưu timeline (FIFO)
- Sử dụng `PriorityQueue` để sắp xếp posts theo engagement
- Sử dụng `LinkedHashMap` để cache recent posts

**Bài 3: Analytics Dashboard**
- Sử dụng `TreeMap` để thống kê theo thời gian
- Sử dụng `HashMap` để cache kết quả tính toán
- Sử dụng `ConcurrentHashMap` cho thread-safe operations

## 📚 Tài liệu tham khảo

- **Oracle Collections Tutorial**: https://docs.oracle.com/javase/tutorial/collections/
- **Baeldung Java Collections**: https://www.baeldung.com/java-collections
- **Java Collections Framework Design Patterns**: https://www.javacodegeeks.com/


