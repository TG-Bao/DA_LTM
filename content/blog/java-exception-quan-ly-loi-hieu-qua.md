---
title: "Quản lý lỗi và Exception trong Java: Bí quyết tránh crash"
date: 2025-09-07T13:00:00+07:00
draft: false
description: "try-catch, throws, custom exception, finally và best practices để app bền vững"
category: "Java"
tags: ["Java", "Exception", "Error Handling"]
readingTime: "14 phút"
author: "TG-Bao"
---

## 🧯 try-catch cơ bản - Nền tảng xử lý lỗi

**Exception Handling là gì?**
Exception handling là cơ chế cho phép chương trình xử lý các tình huống bất thường (exceptions) một cách có kiểm soát, thay vì để chương trình crash. Java có hệ thống exception handling mạnh mẽ với các từ khóa `try`, `catch`, `finally`, và `throws`.

**Cấu trúc try-catch-finally:**
```java
public class TryCatchDemo {
    public static int parseIntSafe(String s) {
        try {
            // Code có thể ném exception
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {
            // Xử lý exception cụ thể
            System.err.println("Lỗi chuyển đổi số: " + e.getMessage());
            return -1; // fallback value
        } finally {
            // Luôn thực thi, dùng để dọn dẹp tài nguyên
            System.out.println("Hoàn thành xử lý số");
        }
    }
    
    // Ví dụ với nhiều catch blocks
    public static void processFile(String filename) {
        try {
            File file = new File(filename);
            Scanner scanner = new Scanner(file);
            
            while (scanner.hasNextLine()) {
                String line = scanner.nextLine();
                System.out.println(line);
            }
            
            scanner.close();
            
        } catch (FileNotFoundException e) {
            System.err.println("Không tìm thấy file: " + filename);
        } catch (SecurityException e) {
            System.err.println("Không có quyền truy cập file: " + filename);
        } catch (Exception e) {
            System.err.println("Lỗi không xác định: " + e.getMessage());
        } finally {
            System.out.println("Hoàn thành xử lý file");
        }
    }
}
```

**Các loại Exception trong Java:**
1. **Checked Exceptions**: Phải được xử lý hoặc khai báo với `throws`
   - `IOException`, `SQLException`, `ClassNotFoundException`
2. **Unchecked Exceptions**: Không bắt buộc phải xử lý
   - `RuntimeException`, `NullPointerException`, `IllegalArgumentException`
3. **Errors**: Lỗi nghiêm trọng, thường không thể xử lý
   - `OutOfMemoryError`, `StackOverflowError`

## 📤 throws & rethrow - Chuyển tiếp và bao bọc Exception

**throws keyword:**
Từ khóa `throws` được sử dụng trong khai báo method để chỉ ra rằng method có thể ném exception. Điều này cho phép caller biết trước và xử lý exception phù hợp.

**Rethrow và Exception Wrapping:**
Khi một method gặp exception nhưng không thể xử lý trực tiếp, nó có thể:
1. **Rethrow**: Ném lại exception gốc
2. **Wrap**: Bao bọc exception gốc trong exception mới với context phù hợp

```java
// Method khai báo throws
public void readFile(Path p) throws IOException {
    Files.readAllLines(p); // có thể ném IOException
}

// Service layer xử lý và wrap exception
public void service(Path p) {
    try {
        readFile(p);
    } catch (IOException e) {
        // Wrap exception với context business
        throw new RuntimeException("Đọc file thất bại", e); // giữ nguyên cause
    }
}

// Ví dụ phức tạp hơn với custom exception
public class FileProcessingService {
    
    public void processFile(String filename) throws FileProcessingException {
        try {
            // Đọc file
            List<String> lines = Files.readAllLines(Paths.get(filename));
            
            // Xử lý dữ liệu
            for (String line : lines) {
                processLine(line);
            }
            
        } catch (IOException e) {
            // Wrap IOException thành business exception
            throw new FileProcessingException(
                "Không thể xử lý file: " + filename, e);
        } catch (DataValidationException e) {
            // Rethrow business exception
            throw e;
        }
    }
    
    private void processLine(String line) throws DataValidationException {
        if (line == null || line.trim().isEmpty()) {
            throw new DataValidationException("Dòng dữ liệu không hợp lệ");
        }
        // Xử lý dòng dữ liệu...
    }
}

// Custom exception với cause
class FileProcessingException extends Exception {
    public FileProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}

class DataValidationException extends Exception {
    public DataValidationException(String message) {
        super(message);
    }
}
```

**Best Practices cho throws và rethrow:**
- **Giữ nguyên cause**: Luôn truyền exception gốc vào constructor của exception mới
- **Thêm context**: Cung cấp thông tin bổ sung về ngữ cảnh xảy ra lỗi
- **Chọn đúng exception type**: Sử dụng exception phù hợp với từng layer (business, technical)
- **Documentation**: Ghi chú rõ ràng trong JavaDoc về exceptions có thể được ném

## 🧰 Custom Exception - Tạo Exception phù hợp với Business Logic

**Tại sao cần Custom Exception?**
Custom exceptions giúp:
- **Phân loại lỗi**: Tạo các loại exception cụ thể cho từng domain
- **Cung cấp context**: Thêm thông tin chi tiết về lỗi
- **Kiểm soát flow**: Cho phép caller xử lý exception phù hợp
- **Maintainability**: Code dễ đọc và maintain hơn

**Các loại Custom Exception:**

```java
// 1. Business Exception - Lỗi nghiệp vụ
class BusinessException extends RuntimeException {
    private final String errorCode;
    
    public BusinessException(String message) {
        super(message);
        this.errorCode = "BUSINESS_ERROR";
    }
    
    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// 2. Validation Exception - Lỗi validation
class ValidationException extends BusinessException {
    private final List<String> validationErrors;
    
    public ValidationException(String message) {
        super(message, "VALIDATION_ERROR");
        this.validationErrors = new ArrayList<>();
    }
    
    public ValidationException(String message, List<String> errors) {
        super(message, "VALIDATION_ERROR");
        this.validationErrors = new ArrayList<>(errors);
    }
    
    public List<String> getValidationErrors() {
        return validationErrors;
    }
    
    public void addError(String error) {
        validationErrors.add(error);
    }
}

// 3. Resource Exception - Lỗi tài nguyên
class ResourceNotFoundException extends BusinessException {
    private final String resourceType;
    private final String resourceId;
    
    public ResourceNotFoundException(String resourceType, String resourceId) {
        super(String.format("%s với ID '%s' không tồn tại", resourceType, resourceId));
        this.resourceType = resourceType;
        this.resourceId = resourceId;
    }
    
    public String getResourceType() {
        return resourceType;
    }
    
    public String getResourceId() {
        return resourceId;
    }
}

// 4. Service Exception - Lỗi service layer
class ServiceException extends Exception {
    private final String serviceName;
    private final String operation;
    
    public ServiceException(String serviceName, String operation, String message, Throwable cause) {
        super(String.format("Lỗi trong %s.%s: %s", serviceName, operation, message), cause);
        this.serviceName = serviceName;
        this.operation = operation;
    }
    
    public String getServiceName() {
        return serviceName;
    }
    
    public String getOperation() {
        return operation;
    }
}
```

**Sử dụng Custom Exception trong thực tế:**

```java
public class UserService {
    
    public void registerUser(String email, String password) throws ValidationException {
        ValidationException validationException = new ValidationException("Dữ liệu người dùng không hợp lệ");
        
        // Validate email
        if (email == null || !email.contains("@")) {
            validationException.addError("Email không hợp lệ");
        }
        
        // Validate password
        if (password == null || password.length() < 8) {
            validationException.addError("Mật khẩu phải có ít nhất 8 ký tự");
        }
        
        // Ném exception nếu có lỗi validation
        if (!validationException.getValidationErrors().isEmpty()) {
            throw validationException;
        }
        
        // Tiếp tục xử lý đăng ký...
    }
    
    public User findUserById(Long id) {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("ID người dùng không hợp lệ");
        }
        
        // Giả sử không tìm thấy user
        throw new ResourceNotFoundException("User", id.toString());
    }
    
    public void updateUserProfile(Long userId, UserProfile profile) {
        try {
            // Xử lý cập nhật profile
            validateUserProfile(profile);
            saveUserProfile(userId, profile);
            
        } catch (ValidationException e) {
            // Log và rethrow
            logger.error("Lỗi validation profile cho user {}: {}", userId, e.getMessage());
            throw e;
        } catch (Exception e) {
            // Wrap thành ServiceException
            throw new ServiceException("UserService", "updateUserProfile", 
                "Không thể cập nhật profile", e);
        }
    }
    
    private void validateUserProfile(UserProfile profile) throws ValidationException {
        ValidationException validationException = new ValidationException("Profile không hợp lệ");
        
        if (profile.getName() == null || profile.getName().trim().isEmpty()) {
            validationException.addError("Tên không được để trống");
        }
        
        if (profile.getAge() < 0 || profile.getAge() > 150) {
            validationException.addError("Tuổi không hợp lệ");
        }
        
        if (!validationException.getValidationErrors().isEmpty()) {
            throw validationException;
        }
    }
    
    private void saveUserProfile(Long userId, UserProfile profile) {
        // Logic lưu profile...
    }
}
```

**Exception Hierarchy tốt:**
```java
// Base exception cho toàn bộ application
abstract class AppException extends Exception {
    protected AppException(String message) {
        super(message);
    }
    
    protected AppException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Business exceptions
class BusinessException extends AppException {
    // Implementation...
}

class ValidationException extends BusinessException {
    // Implementation...
}

class ResourceNotFoundException extends BusinessException {
    // Implementation...
}

// Technical exceptions
class TechnicalException extends AppException {
    // Implementation...
}

class DatabaseException extends TechnicalException {
    // Implementation...
}

class NetworkException extends TechnicalException {
    // Implementation...
}
```

## ✅ Best Practices - Quy tắc vàng cho Exception Handling

**1. Bắt đúng Exception cụ thể:**
```java
// ❌ Không tốt - catch quá rộng
try {
    processData();
} catch (Exception e) {
    // Xử lý tất cả exceptions giống nhau
}

// ✅ Tốt - catch cụ thể
try {
    processData();
} catch (FileNotFoundException e) {
    // Xử lý file không tồn tại
} catch (IOException e) {
    // Xử lý lỗi IO
} catch (ValidationException e) {
    // Xử lý lỗi validation
}
```

**2. Ghi log với context đầy đủ:**
```java
public class DataProcessor {
    private static final Logger logger = LoggerFactory.getLogger(DataProcessor.class);
    
    public void processFile(String filename) {
        try {
            // Xử lý file
            Files.readAllLines(Paths.get(filename));
            
        } catch (IOException e) {
            // Log với context đầy đủ
            logger.error("Không thể đọc file: {} - Lỗi: {}", filename, e.getMessage(), e);
            
            // Wrap với context business
            throw new FileProcessingException("Xử lý file thất bại: " + filename, e);
        }
    }
}
```

**3. Sử dụng try-with-resources để dọn dẹp tài nguyên:**
```java
// ❌ Không tốt - quên đóng resource
public void readFile(String filename) throws IOException {
    FileInputStream fis = new FileInputStream(filename);
    // Có thể quên đóng fis nếu có exception
    fis.read();
    fis.close();
}

// ✅ Tốt - try-with-resources tự động đóng
public void readFile(String filename) throws IOException {
    try (FileInputStream fis = new FileInputStream(filename);
         BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
        
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
    } // Tự động đóng fis và reader
}

// ✅ Tốt - với custom resource
public class DatabaseConnection implements AutoCloseable {
    private Connection connection;
    
    public DatabaseConnection(String url) throws SQLException {
        this.connection = DriverManager.getConnection(url);
    }
    
    public Connection getConnection() {
        return connection;
    }
    
    @Override
    public void close() throws SQLException {
        if (connection != null && !connection.isClosed()) {
            connection.close();
        }
    }
}

// Sử dụng
try (DatabaseConnection dbConn = new DatabaseConnection("jdbc:mysql://localhost:3306/mydb")) {
    Connection conn = dbConn.getConnection();
    // Sử dụng connection
} catch (SQLException e) {
    logger.error("Lỗi database: {}", e.getMessage(), e);
}
```

**4. Exception Handling Strategy theo Layer:**

```java
// Controller Layer - Xử lý exception và trả về response phù hợp
@RestController
public class UserController {
    
    @PostMapping("/users")
    public ResponseEntity<?> createUser(@RequestBody UserRequest request) {
        try {
            User user = userService.createUser(request);
            return ResponseEntity.ok(user);
            
        } catch (ValidationException e) {
            // Trả về 400 Bad Request với validation errors
            return ResponseEntity.badRequest()
                .body(new ErrorResponse("VALIDATION_ERROR", e.getValidationErrors()));
                
        } catch (ResourceNotFoundException e) {
            // Trả về 404 Not Found
            return ResponseEntity.notFound().build();
            
        } catch (BusinessException e) {
            // Trả về 422 Unprocessable Entity
            return ResponseEntity.unprocessableEntity()
                .body(new ErrorResponse("BUSINESS_ERROR", e.getMessage()));
                
        } catch (Exception e) {
            // Log và trả về 500 Internal Server Error
            logger.error("Unexpected error creating user", e);
            return ResponseEntity.internalServerError()
                .body(new ErrorResponse("INTERNAL_ERROR", "Có lỗi xảy ra"));
        }
    }
}

// Service Layer - Business logic và exception handling
@Service
public class UserService {
    
    public User createUser(UserRequest request) throws ValidationException {
        // Validate input
        validateUserRequest(request);
        
        try {
            // Business logic
            User user = new User(request.getName(), request.getEmail());
            return userRepository.save(user);
            
        } catch (DataAccessException e) {
            // Wrap technical exception thành business exception
            throw new ServiceException("UserService", "createUser", 
                "Không thể tạo user", e);
        }
    }
    
    private void validateUserRequest(UserRequest request) throws ValidationException {
        ValidationException validationException = new ValidationException("Dữ liệu không hợp lệ");
        
        if (request.getName() == null || request.getName().trim().isEmpty()) {
            validationException.addError("Tên không được để trống");
        }
        
        if (request.getEmail() == null || !isValidEmail(request.getEmail())) {
            validationException.addError("Email không hợp lệ");
        }
        
        if (!validationException.getValidationErrors().isEmpty()) {
            throw validationException;
        }
    }
}

// Repository Layer - Chỉ wrap technical exceptions
@Repository
public class UserRepository {
    
    public User save(User user) throws DataAccessException {
        try {
            // Database operations
            return entityManager.persist(user);
            
        } catch (PersistenceException e) {
            // Wrap thành DataAccessException
            throw new DataAccessException("Không thể lưu user", e);
        }
    }
}
```

**5. Global Exception Handler:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException e) {
        logger.warn("Validation error: {}", e.getMessage());
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", e.getValidationErrors()));
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException e) {
        logger.warn("Resource not found: {}", e.getMessage());
        return ResponseEntity.notFound().build();
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        logger.warn("Business error: {}", e.getMessage());
        return ResponseEntity.unprocessableEntity()
            .body(new ErrorResponse("BUSINESS_ERROR", e.getMessage()));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception e) {
        logger.error("Unexpected error", e);
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse("INTERNAL_ERROR", "Có lỗi xảy ra"));
    }
}
```

**6. Testing Exception Handling:**
```java
@Test
public void testUserCreationWithInvalidEmail() {
    // Given
    UserRequest request = new UserRequest("John", "invalid-email");
    
    // When & Then
    ValidationException exception = assertThrows(ValidationException.class, () -> {
        userService.createUser(request);
    });
    
    assertTrue(exception.getValidationErrors().contains("Email không hợp lệ"));
}

@Test
public void testUserCreationWithDatabaseError() {
    // Given
    UserRequest request = new UserRequest("John", "john@example.com");
    when(userRepository.save(any())).thenThrow(new DataAccessException("Database error"));
    
    // When & Then
    ServiceException exception = assertThrows(ServiceException.class, () -> {
        userService.createUser(request);
    });
    
    assertEquals("UserService", exception.getServiceName());
    assertEquals("createUser", exception.getOperation());
}
```

**Tóm tắt Best Practices:**
- **Catch cụ thể**: Bắt exception cụ thể thay vì Exception chung
- **Log đầy đủ**: Ghi log với context và stack trace
- **Try-with-resources**: Sử dụng để tự động dọn dẹp tài nguyên
- **Layer-specific handling**: Xử lý exception phù hợp với từng layer
- **Global handler**: Sử dụng @ControllerAdvice cho web applications
- **Test exception**: Viết test cho exception handling
- **Giữ nguyên cause**: Luôn wrap exception với cause gốc
- **Meaningful messages**: Tạo exception message có ý nghĩa


