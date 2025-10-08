---
title: "Best practices khi làm việc với Collections và Streams trong Java"
date: 2025-09-12T10:00:00+07:00
draft: false
tags: ["Java", "Collections", "Streams", "Performance", "Mẹo hay"]
category: "Tips"
---

# Best practices khi làm việc với Collections và Streams trong Java

Collections Framework và Stream API là hai công cụ mạnh mẽ trong Java giúp xử lý dữ liệu hiệu quả. Bài viết này sẽ chia sẻ những best practices khi làm việc với chúng để tối ưu code và hiệu suất.

## Chọn đúng Collection cho từng trường hợp

Việc lựa chọn cấu trúc dữ liệu phù hợp có thể tạo ra sự khác biệt lớn về hiệu suất và tính dễ đọc của code.

### List

```java
// ArrayList: Truy cập ngẫu nhiên nhanh, thêm/xóa ở cuối nhanh
List<String> names = new ArrayList<>();  // Tốt cho truy cập phần tử theo index

// LinkedList: Thêm/xóa ở đầu và cuối nhanh, nhưng truy cập ngẫu nhiên chậm
List<String> queue = new LinkedList<>();  // Tốt cho queue hoặc stack
```

### Set

```java
// HashSet: Kiểm tra tồn tại nhanh, không đảm bảo thứ tự
Set<String> uniqueNames = new HashSet<>();  // O(1) cho contains(), add(), remove()

// LinkedHashSet: Giữ thứ tự chèn, kiểm tra tồn tại nhanh
Set<String> orderedUniqueNames = new LinkedHashSet<>();  // Kết hợp tốc độ của HashSet và thứ tự chèn

// TreeSet: Tự động sắp xếp, tìm kiếm nhanh
Set<String> sortedNames = new TreeSet<>();  // Tốt khi cần duy trì thứ tự
```

### Map

```java
// HashMap: Truy xuất nhanh, không đảm bảo thứ tự
Map<Integer, String> idToName = new HashMap<>();  // O(1) cho get(), put(), containsKey()

// LinkedHashMap: Giữ thứ tự chèn, truy xuất nhanh
Map<Integer, String> orderedIdToName = new LinkedHashMap<>();  // Khi cần duy trì thứ tự chèn

// TreeMap: Tự động sắp xếp theo key, tìm kiếm nhanh
Map<Integer, String> sortedIdToName = new TreeMap<>();  // Khi cần key được sắp xếp
```

## Khởi tạo Collections hiệu quả

### Khởi tạo với kích thước ban đầu

Nếu bạn biết trước kích thước của collection, hãy chỉ định nó khi khởi tạo để tránh việc phải mở rộng nhiều lần.

```java
// Không tốt
List<String> names = new ArrayList<>();  // Mặc định capacity là 10

// Tốt
List<String> names = new ArrayList<>(1000);  // Chỉ định capacity ban đầu
```

### Sử dụng Factory Methods (Java 9+)

```java
// Tạo List bất biến
List<String> names = List.of("Alice", "Bob", "Charlie");

// Tạo Set bất biến
Set<String> uniqueNames = Set.of("Alice", "Bob", "Charlie");

// Tạo Map bất biến
Map<String, Integer> ages = Map.of(
    "Alice", 30,
    "Bob", 25,
    "Charlie", 35
);
```

## Tối ưu hiệu suất với Stream

### Khi nào nên dùng Stream

Streams rất mạnh mẽ nhưng không phải lúc nào cũng là lựa chọn tốt nhất:

- Dùng Stream khi cần xử lý dữ liệu phức tạp, biến đổi dữ liệu qua nhiều bước
- Dùng vòng lặp truyền thống cho các thao tác đơn giản hoặc khi cần hiệu suất tối đa

```java
// Trường hợp phù hợp cho Stream
List<Integer> evenSquares = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());

// Trường hợp đơn giản, vòng lặp có thể tốt hơn
int sum = 0;
for (int n : numbers) {
    sum += n;
}
```

### Tối ưu Stream pipeline

```java
// Không tốt: filter sau map xử lý không cần thiết
List<String> longNames = persons.stream()
    .map(Person::getName)
    .filter(name -> name.length() > 5)
    .collect(Collectors.toList());

// Tốt: filter trước map giảm số lượng phần tử cần biến đổi
List<String> longNames = persons.stream()
    .filter(person -> person.getName().length() > 5)
    .map(Person::getName)
    .collect(Collectors.toList());
```

### Sử dụng các collectors hiệu quả

```java
// Chuyển đổi Stream thành Collection
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
Map<Integer, String> map = stream.collect(Collectors.toMap(Person::getId, Person::getName));

// Nhóm dữ liệu
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Phân vùng dữ liệu
Map<Boolean, List<Employee>> seniorJunior = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getYearsOfExperience() > 5));

// Tính toán thống kê
DoubleSummaryStatistics stats = employees.stream()
    .collect(Collectors.summarizingDouble(Employee::getSalary));
System.out.println("Average: " + stats.getAverage());
System.out.println("Max: " + stats.getMax());
```

## Tránh pitfalls khi dùng parallel stream

Parallel streams có thể cải thiện hiệu suất nhưng cũng có thể gây ra vấn đề nếu không sử dụng đúng cách.

### Khi nào nên dùng parallel stream

```java
// Tốt cho parallel: dữ liệu lớn, thao tác độc lập, không có tranh chấp
List<BigInteger> results = hugeList.parallelStream()
    .map(this::complexCalculation)
    .collect(Collectors.toList());

// Không tốt cho parallel: dữ liệu nhỏ, thao tác đơn giản
List<String> upperCaseNames = smallList.parallelStream() // Overhead > benefit
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### Tránh trạng thái chia sẻ

```java
// Nguy hiểm: trạng thái chia sẻ trong parallel stream
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
int sum = 0;
numbers.parallelStream().forEach(n -> sum += n); // Race condition!

// An toàn: sử dụng reduce
int sum = numbers.parallelStream().reduce(0, Integer::sum);
```

### Chọn đúng cấu trúc dữ liệu

```java
// Tốt cho parallel: ArrayList, arrays (truy cập ngẫu nhiên nhanh)
List<Integer> list = new ArrayList<>(numbers);
list.parallelStream()...

// Không tốt cho parallel: LinkedList (truy cập ngẫu nhiên chậm)
List<Integer> linkedList = new LinkedList<>(numbers);
linkedList.parallelStream()... // Hiệu suất kém
```

## Best practices chung khi làm việc với Collections

### Sử dụng interface thay vì implementation

```java
// Tốt: sử dụng interface trong khai báo
List<String> names = new ArrayList<>();
Map<String, Integer> scores = new HashMap<>();

// Không tốt: sử dụng implementation cụ thể
ArrayList<String> names = new ArrayList<>();
```

### Bảo vệ Collections nội bộ

```java
public class User {
    private final List<String> roles = new ArrayList<>();
    
    // Không tốt: trả về reference trực tiếp
    public List<String> getRoles() {
        return roles; // Người gọi có thể thay đổi collection nội bộ
    }
    
    // Tốt: trả về bản sao hoặc view không thể thay đổi
    public List<String> getRoles() {
        return Collections.unmodifiableList(roles);
    }
    
    // Hoặc trả về bản sao
    public List<String> getRolesCopy() {
        return new ArrayList<>(roles);
    }
}
```

### Sử dụng isEmpty() thay vì kiểm tra size

```java
// Không tốt
if (list.size() == 0) {
    // list trống
}

// Tốt: rõ ràng và có thể hiệu quả hơn với một số implementation
if (list.isEmpty()) {
    // list trống
}
```

## Kết luận

Việc sử dụng đúng Collections và Streams không chỉ giúp code của bạn dễ đọc hơn mà còn cải thiện đáng kể hiệu suất. Hãy nhớ chọn cấu trúc dữ liệu phù hợp với nhu cầu, tối ưu Stream pipeline và cẩn thận khi sử dụng parallel streams. Với những best practices này, bạn sẽ tận dụng tối đa sức mạnh của Collections Framework và Stream API trong Java.