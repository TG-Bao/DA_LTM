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

## 🧺 Tổng quan nhanh

- `List`: có thứ tự, cho phép trùng
- `Set`: không trùng, không đảm bảo thứ tự (trừ `LinkedHashSet`/`TreeSet`)
- `Map`: key-value, key không trùng

```java
import java.util.*;

public class CollectionsDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A"); list.add("B"); list.add("A");

        Set<String> set = new HashSet<>(list); // loại trùng

        Map<String, Integer> map = new HashMap<>();
        map.put("A", 1); map.put("B", 2);

        System.out.println(list); // [A, B, A]
        System.out.println(set);  // [A, B]
        System.out.println(map);  // {A=1, B=2}
    }
}
```

## ⚖️ So sánh nhanh

| Cấu trúc | Trùng | Thứ tự | Tìm kiếm |
|---------|------|--------|---------|
| ArrayList | ✔ | theo index | O(1) truy cập theo index |
| LinkedList | ✔ | theo chèn | O(n) |
| HashSet | ✖ | không | O(1) trung bình |
| TreeSet | ✖ | sắp xếp | O(log n) |
| HashMap | key ✖ | không | O(1) trung bình |
| LinkedHashMap | key ✖ | theo chèn | O(1) |

## 🧪 Ví dụ thực tế: Thống kê từ

```java
import java.util.*;

public class WordCount {
    public static Map<String, Integer> countWords(String text) {
        Map<String, Integer> freq = new HashMap<>();
        for (String w : text.toLowerCase().split("\\W+")) {
            if (w.isBlank()) continue;
            freq.put(w, freq.getOrDefault(w, 0) + 1);
        }
        return freq;
    }

    public static void main(String[] args) {
        System.out.println(countWords("Java Java set map map list"));
    }
}
```

## ✅ Bài tập
- Dùng `TreeMap` để in thống kê theo thứ tự alphabet.
- Dùng `LinkedHashSet` để loại trùng nhưng giữ thứ tự ban đầu.


