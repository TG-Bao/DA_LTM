---
title: "Bí quyết quản lý Exception hiệu quả trong Java"
date: 2025-09-15T10:00:00+07:00
draft: false
tags: ["Java", "Exception", "Error Handling", "Best Practices", "Mẹo hay"]
category: "Tips"
---

# Bí quyết quản lý Exception hiệu quả trong Java

Xử lý ngoại lệ (exception) là một phần quan trọng trong việc phát triển ứng dụng Java chất lượng cao. Bài viết này sẽ chia sẻ những bí quyết giúp bạn quản lý exception hiệu quả, tạo ra code ổn định và dễ bảo trì.

## Custom Exception - Tạo ngoại lệ tùy chỉnh

Việc tạo các lớp exception riêng giúp code của bạn rõ ràng và dễ hiểu hơn, đồng thời cung cấp thông tin cụ thể về lỗi.

### Khi nào nên tạo Custom Exception

- Khi bạn cần cung cấp thông tin chi tiết về lỗi nghiệp vụ
- Khi bạn muốn phân loại lỗi theo domain của ứng dụng
- Khi bạn cần xử lý các trường hợp lỗi đặc biệt

### Cách tạo Custom Exception

```java
// Exception cơ bản
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }

    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Exception với thông tin bổ sung
public class ValidationException extends RuntimeException {
    private final List<String> errors;

    public ValidationException(String message, List<String> errors) {
        super(message);
        this.errors = Collections.unmodifiableList(new ArrayList<>(errors));
    }

    public List<String> getErrors() {
        return errors;
    }
}
```

### Sử dụng Custom Exception

```java
public User findUserById(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException("Không tìm thấy user với ID: " + id));
}

public void validateUser(User user) {
    List<String> errors = new ArrayList<>();

    if (user.getName() == null || user.getName().isEmpty()) {
        errors.add("Tên không được để trống");
    }

    if (user.getEmail() == null || !user.getEmail().contains("@")) {
        errors.add("Email không hợp lệ");
    }

    if (!errors.isEmpty()) {
        throw new ValidationException("Dữ liệu user không hợp lệ", errors);
    }
}
```

## Try-with-resources - Tự động đóng tài nguyên

Try-with-resources giúp đảm bảo các tài nguyên như file, connection được đóng đúng cách, ngay cả khi có exception xảy ra.

### Cách sử dụng

```java
// Trước Java 7 - Dài dòng và dễ gây lỗi
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    String line = reader.readLine();
    // Xử lý dữ liệu
} catch (IOException e) {
    // Xử lý exception
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            // Xử lý exception khi đóng tài nguyên
        }
    }
}

// Từ Java 7 - Ngắn gọn và an toàn hơn
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line = reader.readLine();
    // Xử lý dữ liệu
} catch (IOException e) {
    // Xử lý exception
}
```

### Sử dụng với nhiều tài nguyên

```java
try (
    FileInputStream input = new FileInputStream("input.txt");
    FileOutputStream output = new FileOutputStream("output.txt")
) {
    // Đọc từ input và ghi vào output
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = input.read(buffer)) != -1) {
        output.write(buffer, 0, bytesRead);
    }
} catch (IOException e) {
    // Xử lý exception
}
```

### Tạo resource tùy chỉnh

```java
public class DatabaseConnection implements AutoCloseable {
    private final Connection connection;

    public DatabaseConnection(String url) throws SQLException {
        this.connection = DriverManager.getConnection(url);
    }

    public Connection getConnection() {
        return connection;
    }

    @Override
    public void close() throws Exception {
        if (connection != null && !connection.isClosed()) {
            connection.close();
        }
    }
}

// Sử dụng
try (DatabaseConnection dbConn = new DatabaseConnection("jdbc:mysql://localhost:3306/mydb")) {
    Connection conn = dbConn.getConnection();
    // Thực hiện các thao tác với connection
} catch (Exception e) {
    // Xử lý exception
}
```

## Logging - Ghi log exception đúng cách

Ghi log exception đúng cách giúp dễ dàng debug và theo dõi lỗi trong ứng dụng.

### Sử dụng thư viện logging

```java
// Sử dụng SLF4J và Logback
private static final Logger logger = LoggerFactory.getLogger(MyClass.class);

public void processData() {
    try {
        // Code có thể ném exception
    } catch (Exception e) {
        logger.error("Lỗi khi xử lý dữ liệu", e);
        throw new ProcessingException("Không thể xử lý dữ liệu", e);
    }
}
```

### Các nguyên tắc logging exception

1. **Luôn bao gồm stack trace**: Đảm bảo bạn truyền exception object vào phương thức logging

```java
// Không tốt - Mất stack trace
logger.error("Lỗi: " + e.getMessage());

// Tốt - Giữ stack trace
logger.error("Lỗi khi xử lý thanh toán", e);
```

2. **Log ở mức độ phù hợp**

```java
// Lỗi nghiêm trọng
logger.error("Database connection failed", e);

// Cảnh báo
logger.warn("User session expired prematurely", e);

// Thông tin debug
logger.debug("Request processing details", e);
```

3. **Cung cấp context đầy đủ**

```java
logger.error("Failed to process order #{} for customer {}", orderId, customerId, e);
```

## Tránh Swallow Exception - Không nuốt chửng ngoại lệ

Một trong những lỗi phổ biến nhất là "nuốt chửng" exception - bắt exception nhưng không xử lý hoặc ghi log.

### Các trường hợp swallow exception phổ biến

```java
// Không tốt - Catch trống
try {
    // Code có thể ném exception
} catch (Exception e) {
    // Không làm gì cả!
}

// Không tốt - Chỉ in ra console
try {
    // Code có thể ném exception
} catch (Exception e) {
    e.printStackTrace();
}
```

### Cách xử lý đúng

```java
// Tốt - Ghi log và xử lý
try {
    // Code có thể ném exception
} catch (IOException e) {
    logger.error("Lỗi đọc file cấu hình", e);
    // Sử dụng cấu hình mặc định
    useDefaultConfiguration();
}

// Tốt - Wrap và throw
try {
    // Code có thể ném exception
} catch (SQLException e) {
    logger.error("Database error", e);
    throw new ServiceException("Không thể truy vấn dữ liệu", e);
}
```

## Các best practices khác

### 1. Phân loại exception rõ ràng

```java
try {
    // Code có thể ném nhiều loại exception
} catch (FileNotFoundException e) {
    // Xử lý khi không tìm thấy file
    logger.error("File không tồn tại", e);
} catch (IOException e) {
    // Xử lý các lỗi IO khác
    logger.error("Lỗi đọc/ghi file", e);
} catch (Exception e) {
    // Xử lý các exception khác
    logger.error("Lỗi không xác định", e);
}
```

### 2. Sử dụng multi-catch (Java 7+)

```java
try {
    // Code có thể ném nhiều loại exception
} catch (FileNotFoundException | SQLException e) {
    // Xử lý chung cho cả hai loại exception
    logger.error("Lỗi truy cập dữ liệu", e);
}
```

### 3. Throw early, catch late

```java
// Kiểm tra đầu vào ngay lập tức
public void processFile(String filePath) {
    if (filePath == null || filePath.isEmpty()) {
        throw new IllegalArgumentException("File path không được để trống");
    }

    // Tiếp tục xử lý với đầu vào hợp lệ
}
```

### 4. Sử dụng Optional thay vì ném exception

```java
// Thay vì ném exception
public User findUserById(Long id) {
    return Optional.ofNullable(userRepository.findById(id))
        .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
}

// Sử dụng
userService.findUserById(123)
    .map(User::getName)
    .orElse("Unknown User");
```

## Bản đồ hệ thống ngoại lệ (Taxonomy) và nguyên tắc

### Checked vs Unchecked vs Error

- Checked Exception (extends `Exception`, không phải `RuntimeException`): buộc khai báo/handle tại compile-time. Thường cho điều kiện có thể phục hồi (I/O, network, file...).
- Unchecked Exception (extends `RuntimeException`): lỗi lập trình/tiền điều kiện (null, illegal state/arg), không bắt buộc phải catch.
- Error (extends `Error`): lỗi môi trường/JVM (OutOfMemoryError, StackOverflowError) — không catch trừ khi để log và dừng hệ thống an toàn.

Nguyên tắc sử dụng:

- Điều kiện có thể dự đoán và xử lý được ở call-site → Checked.
- Vi phạm hợp đồng API/tiền điều kiện, bug logic → Unchecked.
- Tuyệt đối không ném/catch `Throwable` hoặc `Error` để “tiếp tục chạy”.

### Vì sao tồn tại Checked và khi nào chọn Unchecked?

- Checked giúp “ép” luồng xử lý lỗi được nghĩ tới, phù hợp adapter tầng thấp (I/O, file...).
- Unchecked giúp API sạch, không “ô nhiễm” chữ ký phương thức. Dùng cho domain/business hoặc validation tiền điều kiện.
- Một guideline thực tế: public API hướng người dùng thư viện → ưu tiên Unchecked; lớp biên I/O → Checked.

### Error vs Exception

- Exception: có thể (hoặc nên) xử lý để tiếp tục vận hành logic phù hợp.
- Error: trạng thái hệ thống không đáng tin; chỉ log và dừng/thoái lui an toàn.

## Anti-patterns cần tránh

- Nuốt ngoại lệ (swallow) hoặc chỉ `printStackTrace()`.
- Bọc exception mà mất nguyên nhân gốc (không truyền `cause`).
- Throw `Exception`/`RuntimeException` chung chung, message mơ hồ.
- Dùng exception cho luồng điều khiển bình thường (control flow).
- Lạm dụng checked khiến call-site bừa bộn; hoặc lạm dụng unchecked khiến thiếu kiểm soát.

## Mapping Exception ↔ HTTP Status (REST)

- ValidationException → 400 Bad Request.
- AuthenticationException → 401 Unauthorized.
- AuthorizationException → 403 Forbidden.
- ResourceNotFoundException → 404 Not Found.
- ConflictException (trùng, trạng thái) → 409 Conflict.
- TooManyRequestsException/RateLimit → 429 Too Many Requests.
- BusinessException (không khớp quy tắc) → 422 Unprocessable Entity.
- Technical/Unexpected → 500 Internal Server Error.

Trong Spring, map bằng `@ControllerAdvice` + `@ExceptionHandler` để thống nhất phản hồi.

## Retry, Idempotency và Backoff

Lý thuyết:

- Retry chỉ cho lỗi tạm thời (transient) như timeout, 5xx; không retry cho 4xx.
- Idempotency đảm bảo retry không gây tác dụng phụ (PUT/GET thường idempotent; POST có thể idempotent qua idempotency key).
- Exponential backoff + jitter tránh “dội bão” vào dịch vụ.

```java
// Minh hoạ retry có backoff + jitter
T result = Retry.decorateSupplier(
    Retry.of("io-call", RetryConfig.<T>custom()
        .maxAttempts(4)
        .waitDuration(Duration.ofMillis(200)) // ví dụ, có thể thay bằng backoff hàm
        .retryOnException(ex -> isTransient(ex))
        .build()),
    () -> remoteCall()
).get();

boolean isTransient(Throwable ex) {
    return ex instanceof SocketTimeoutException
        || ex instanceof ConnectException
        || (ex instanceof HttpStatusException h && h.status() >= 500);
}
```

Gợi ý: có thể dùng Resilience4j/Retry4j hoặc tự cài đặt backoff với jitter ngẫu nhiên.

## Giao dịch (Transactions) và Exception

- `@Transactional` mặc định rollback với unchecked; checked không rollback trừ khi cấu hình `rollbackFor`.
- Bao gói business logic cùng transaction ranh giới rõ ràng; tránh logic dài chặn I/O trong transaction.
- Tránh catch rồi “nuốt” trong service annotated `@Transactional` vì có thể chặn cơ chế rollback.

```java
@Transactional(rollbackFor = {SQLException.class, BusinessException.class})
public void placeOrder(OrderRequest req) {
    validate(req); // có thể ném ValidationException (unchecked) → rollback
    reserveInventory(req);
    chargePayment(req);
    saveOrder(req);
}
```

## Kiểm thử Exception và API

### Unit test service

```java
@Test
void shouldThrowWhenEmailInvalid() {
    var req = new UserRequest("John", "bad");
    assertThrows(ValidationException.class, () -> service.create(req));
}
```

### Test REST mapping

```java
@Test
void createUser_validationError_returns400() throws Exception {
    mockMvc.perform(post("/users").content("{}")
            .contentType(MediaType.APPLICATION_JSON))
        .andExpect(status().isBadRequest());
}
```

### Nguyên tắc test

- Kiểm tra thông điệp/field lỗi có ý nghĩa.
- Mô phỏng lỗi hạ tầng (DB down, timeout) và xác nhận retry/fallback đúng.
- Kiểm tra transaction rollback với các exception khác nhau.

## Kết luận

Quản lý exception hiệu quả là một kỹ năng quan trọng giúp tạo ra ứng dụng Java ổn định và dễ bảo trì. Bằng cách tạo custom exception phù hợp, sử dụng try-with-resources, ghi log đầy đủ và tránh nuốt chửng exception, bạn sẽ xây dựng được hệ thống xử lý lỗi mạnh mẽ và chuyên nghiệp. Bổ sung thêm taxonomy rõ ràng, mapping HTTP, chiến lược retry/idempotency, và quy tắc transaction sẽ giúp hệ thống của bạn bền vững hơn trong môi trường production.
