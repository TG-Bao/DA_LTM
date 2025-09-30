---
title: "Xây dựng RESTful API với Spring Boot - Từ cơ bản đến nâng cao"
date: 2024-10-17T10:00:00+07:00
draft: false
author: "Nguyễn Văn A"
description: "Học cách tạo RESTful API mạnh mẽ với Spring Boot"
category: "Java"
tags: ["Java", "Spring Boot", "REST API", "Microservices", "Backend"]
readingTime: 15
---

## RESTful API - Cầu nối của thế giới web 🌐

RESTful API đã trở thành **tiêu chuẩn** cho việc xây dựng các ứng dụng web hiện đại. Với Spring Boot, việc tạo ra một API mạnh mẽ, an toàn và dễ bảo trì trở nên vô cùng đơn giản.

## REST là gì? 🤔

**REST (Representational State Transfer)** là một kiến trúc phần mềm cho các hệ thống phân tán:

- **Stateless** - Không lưu trạng thái
- **Client-Server** - Tách biệt client và server
- **Cacheable** - Có thể cache
- **Uniform Interface** - Giao diện thống nhất

### HTTP Methods trong REST

```http
GET    /api/users        # Lấy danh sách users
GET    /api/users/1      # Lấy user có ID = 1
POST   /api/users        # Tạo user mới
PUT    /api/users/1      # Cập nhật toàn bộ user
PATCH  /api/users/1      # Cập nhật một phần user
DELETE /api/users/1      # Xóa user
```

## Spring Boot Project Setup

### 1. Tạo Project với Spring Initializr

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

### 2. Cấu hình Application

```yaml
# application.yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

logging:
  level:
    org.springframework.web: DEBUG
```

## Xây dựng RESTful API

### 1. Entity Model

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Tên không được để trống")
    @Size(min = 2, max = 50, message = "Tên phải từ 2-50 ký tự")
    private String name;
    
    @Email(message = "Email không hợp lệ")
    @NotBlank(message = "Email không được để trống")
    private String email;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status = UserStatus.ACTIVE;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Constructors, Getters, Setters
    public User() {}
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters...
}

enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}
```

### 2. Repository Layer

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Tìm user theo email
    Optional<User> findByEmail(String email);
    
    // Tìm user theo status
    List<User> findByStatus(UserStatus status);
    
    // Tìm user theo tên (case insensitive)
    List<User> findByNameContainingIgnoreCase(String name);
    
    // Đếm số user theo status
    long countByStatus(UserStatus status);
    
    // Tìm user được tạo trong khoảng thời gian
    @Query("SELECT u FROM User u WHERE u.createdAt BETWEEN :startDate AND :endDate")
    List<User> findUsersCreatedBetween(
        @Param("startDate") LocalDateTime startDate,
        @Param("endDate") LocalDateTime endDate
    );
}
```

### 3. Service Layer

```java
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // Lấy tất cả users
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    // Lấy user theo ID
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User không tồn tại với ID: " + id));
    }
    
    // Tạo user mới
    public User createUser(User user) {
        // Kiểm tra email đã tồn tại chưa
        if (userRepository.findByEmail(user.getEmail()).isPresent()) {
            throw new EmailAlreadyExistsException("Email đã được sử dụng: " + user.getEmail());
        }
        
        return userRepository.save(user);
    }
    
    // Cập nhật user
    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id);
        
        user.setName(userDetails.getName());
        user.setEmail(userDetails.getEmail());
        user.setStatus(userDetails.getStatus());
        
        return userRepository.save(user);
    }
    
    // Xóa user
    public void deleteUser(Long id) {
        User user = getUserById(id);
        userRepository.delete(user);
    }
    
    // Tìm kiếm users
    public List<User> searchUsers(String name, UserStatus status) {
        if (name != null && status != null) {
            return userRepository.findByNameContainingIgnoreCaseAndStatus(name, status);
        } else if (name != null) {
            return userRepository.findByNameContainingIgnoreCase(name);
        } else if (status != null) {
            return userRepository.findByStatus(status);
        }
        return getAllUsers();
    }
}
```

### 4. Controller Layer

```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // GET /api/users
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) UserStatus status) {
        
        List<User> users = userService.searchUsers(name, status);
        return ResponseEntity.ok(users);
    }
    
    // GET /api/users/{id}
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    // POST /api/users
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User createdUser = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }
    
    // PUT /api/users/{id}
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, 
            @Valid @RequestBody User userDetails) {
        
        User updatedUser = userService.updateUser(id, userDetails);
        return ResponseEntity.ok(updatedUser);
    }
    
    // DELETE /api/users/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    // GET /api/users/stats
    @GetMapping("/stats")
    public ResponseEntity<Map<String, Object>> getUserStats() {
        Map<String, Object> stats = new HashMap<>();
        stats.put("totalUsers", userService.getAllUsers().size());
        stats.put("activeUsers", userService.searchUsers(null, UserStatus.ACTIVE).size());
        stats.put("inactiveUsers", userService.searchUsers(null, UserStatus.INACTIVE).size());
        
        return ResponseEntity.ok(stats);
    }
}
```

## Exception Handling

### 1. Custom Exceptions

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}

@ResponseStatus(HttpStatus.CONFLICT)
public class EmailAlreadyExistsException extends RuntimeException {
    public EmailAlreadyExistsException(String message) {
        super(message);
    }
}
```

### 2. Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            "USER_NOT_FOUND", 
            ex.getMessage(), 
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(EmailAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleEmailExists(EmailAlreadyExistsException ex) {
        ErrorResponse error = new ErrorResponse(
            "EMAIL_EXISTS", 
            ex.getMessage(), 
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.toList());
            
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR", 
            "Dữ liệu không hợp lệ: " + String.join(", ", errors), 
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }
}

class ErrorResponse {
    private String code;
    private String message;
    private LocalDateTime timestamp;
    
    // Constructors, Getters, Setters...
}
```

## API Documentation với Swagger

### 1. Thêm Dependencies

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.2</version>
</dependency>
```

### 2. Cấu hình Swagger

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User Management API")
                .version("1.0")
                .description("API quản lý người dùng với Spring Boot"))
            .addServersItem(new Server().url("http://localhost:8080").description("Local Server"));
    }
}
```

### 3. Annotate Controllers

```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "User Management", description = "APIs quản lý người dùng")
public class UserController {
    
    @Operation(summary = "Lấy danh sách users", description = "Lấy tất cả users với tùy chọn filter")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Thành công"),
        @ApiResponse(responseCode = "500", description = "Lỗi server")
    })
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers(
            @Parameter(description = "Tên user để tìm kiếm") 
            @RequestParam(required = false) String name,
            @Parameter(description = "Trạng thái user") 
            @RequestParam(required = false) UserStatus status) {
        // Implementation...
    }
}
```

## Testing REST API

### 1. Unit Tests

```java
@SpringBootTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY)
class UserServiceTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldCreateUserSuccessfully() {
        // Given
        User user = new User("John Doe", "john@example.com");
        
        // When
        User createdUser = userService.createUser(user);
        
        // Then
        assertThat(createdUser.getId()).isNotNull();
        assertThat(createdUser.getName()).isEqualTo("John Doe");
        assertThat(createdUser.getEmail()).isEqualTo("john@example.com");
    }
    
    @Test
    void shouldThrowExceptionWhenEmailExists() {
        // Given
        User user1 = new User("John Doe", "john@example.com");
        User user2 = new User("Jane Doe", "john@example.com");
        userService.createUser(user1);
        
        // When & Then
        assertThatThrownBy(() -> userService.createUser(user2))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessageContaining("Email đã được sử dụng");
    }
}
```

### 2. Integration Tests

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.ANY)
class UserControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldCreateUserViaAPI() {
        // Given
        User user = new User("John Doe", "john@example.com");
        
        // When
        ResponseEntity<User> response = restTemplate.postForEntity(
            "/api/users", user, User.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getName()).isEqualTo("John Doe");
    }
}
```

## Best Practices

### 1. **Response Wrapper** 📦
```java
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;
    
    // Constructors, Getters, Setters...
}
```

### 2. **Pagination** 📄
```java
@GetMapping
public ResponseEntity<Page<User>> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    
    Pageable pageable = PageRequest.of(page, size);
    Page<User> users = userService.getAllUsers(pageable);
    return ResponseEntity.ok(users);
}
```

### 3. **Caching** ⚡
```java
@Cacheable("users")
@GetMapping("/{id}")
public ResponseEntity<User> getUserById(@PathVariable Long id) {
    // Implementation...
}
```

## Kết luận

Spring Boot giúp chúng ta xây dựng RESTful API một cách **nhanh chóng và hiệu quả**:

- ✅ **Rapid Development** - Tạo API trong vài phút
- ✅ **Built-in Features** - Validation, Exception Handling, Testing
- ✅ **Production Ready** - Security, Monitoring, Documentation
- ✅ **Scalable** - Microservices architecture

Trong bài tiếp theo, chúng ta sẽ tìm hiểu về **xử lý bất đồng bộ với CompletableFuture** - một tính năng mạnh mẽ của Java 8+! 🚀

---

**Tài liệu tham khảo:**
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [REST API Best Practices](https://restfulapi.net/)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
