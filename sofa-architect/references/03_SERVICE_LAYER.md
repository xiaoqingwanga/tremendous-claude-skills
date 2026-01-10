# Service Layer Documentation

## Overview
This document describes the service layer implementation patterns and best practices for SOFA Boot applications.

## Service Layer Architecture

### Service Interface Definition

```java
/**
 * Service interface for user operations
 */
public interface UserService {
    
    /**
     * Create a new user
     * @param userDTO user data transfer object
     * @return created user ID
     * @throws BusinessException if validation fails
     */
    Long createUser(UserDTO userDTO) throws BusinessException;
    
    /**
     * Get user by ID
     * @param userId user ID
     * @return user DTO
     * @throws UserNotFoundException if user not found
     */
    UserDTO getUserById(Long userId) throws UserNotFoundException;
    
    /**
     * Update user information
     * @param userId user ID
     * @param userDTO updated user data
     * @throws UserNotFoundException if user not found
     */
    void updateUser(Long userId, UserDTO userDTO) throws UserNotFoundException;
    
    /**
     * Delete user
     * @param userId user ID
     * @throws UserNotFoundException if user not found
     */
    void deleteUser(Long userId) throws UserNotFoundException;
}
```

### Service Implementation

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private UserValidator userValidator;
    
    @Autowired
    private UserEventPublisher eventPublisher;
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long createUser(UserDTO userDTO) throws BusinessException {
        // 1. Validate input
        userValidator.validate(userDTO);
        
        // 2. Convert DTO to DO
        UserDO userDO = convertToDO(userDTO);
        
        // 3. Business logic
        userDO.setCreatedAt(LocalDateTime.now());
        
        // 4. Persist
        userMapper.insert(userDO);
        
        // 5. Publish domain event
        eventPublisher.publishUserCreated(userDO);
        
        return userDO.getId();
    }
    
    @Override
    public UserDTO getUserById(Long userId) throws UserNotFoundException {
        UserDO userDO = userMapper.findById(userId);
        if (userDO == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }
        return convertToDTO(userDO);
    }
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public void updateUser(Long userId, UserDTO userDTO) throws UserNotFoundException {
        UserDO existingUser = userMapper.findById(userId);
        if (existingUser == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }
        
        // Update fields
        existingUser.setName(userDTO.getName());
        existingUser.setEmail(userDTO.getEmail());
        existingUser.setUpdatedAt(LocalDateTime.now());
        
        userMapper.update(existingUser);
    }
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public void deleteUser(Long userId) throws UserNotFoundException {
        UserDO userDO = userMapper.findById(userId);
        if (userDO == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }
        
        // Soft delete
        userDO.setDeleted(true);
        userDO.setDeletedAt(LocalDateTime.now());
        userMapper.update(userDO);
    }
    
    private UserDO convertToDO(UserDTO dto) {
        // Conversion logic
        return new UserDO();
    }
    
    private UserDTO convertToDTO(UserDO userDO) {
        // Conversion logic
        return new UserDTO();
    }
}
```

## Service Layer Best Practices

### 1. Separation of Concerns

**DO (Database Objects)**: Used only in DAL layer
```java
@Entity
public class UserDO {
    private Long id;
    private String name;
    // Database-specific fields
}
```

**DTO (Data Transfer Objects)**: Used in service and facade layers
```java
public class UserDTO {
    private Long id;
    private String name;
    // API-specific fields
}
```

**VO (View Objects)**: Used in web layer for response
```java
public class UserVO {
    private Long id;
    private String name;
    // Presentation-specific fields
}
```

### 2. Transaction Management

```java
@Service
public class OrderServiceImpl implements OrderService {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    /**
     * Create order with transaction
     */
    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long createOrder(OrderDTO orderDTO) {
        // 1. Create order
        OrderDO order = new OrderDO();
        orderMapper.insert(order);
        
        // 2. Deduct inventory
        inventoryService.deductInventory(orderDTO.getItems());
        
        // 3. Process payment
        paymentService.processPayment(orderDTO.getPayment());
        
        return order.getId();
    }
}
```

### 3. Exception Handling

```java
// Custom business exceptions
public class BusinessException extends RuntimeException {
    private final String errorCode;
    
    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
}

public class UserNotFoundException extends BusinessException {
    public UserNotFoundException(String message) {
        super("USER_NOT_FOUND", message);
    }
}

// Service layer exception handling
@Service
public class UserServiceImpl implements UserService {
    
    @Override
    public UserDTO getUserById(Long userId) throws UserNotFoundException {
        UserDO userDO = userMapper.findById(userId);
        if (userDO == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }
        return convertToDTO(userDO);
    }
}
```

### 4. Validation

```java
@Component
public class UserValidator {
    
    public void validate(UserDTO userDTO) throws BusinessException {
        if (userDTO.getName() == null || userDTO.getName().isEmpty()) {
            throw new BusinessException("INVALID_INPUT", "Name is required");
        }
        
        if (userDTO.getEmail() == null || !isValidEmail(userDTO.getEmail())) {
            throw new BusinessException("INVALID_INPUT", "Invalid email format");
        }
    }
    
    private boolean isValidEmail(String email) {
        // Email validation logic
        return email != null && email.contains("@");
    }
}
```

### 5. Async Processing with AntScheduler

```java
@Service
public class ReportServiceImpl implements ReportService {
    
    @Autowired
    private ReportGenerator reportGenerator;
    
    @Autowired
    private EmailService emailService;
    
    /**
     * Generate report asynchronously
     */
    @Override
    public void generateReportAsync(Long reportId) {
        AntScheduler.schedule(() -> {
            try {
                // Generate report
                byte[] reportData = reportGenerator.generate(reportId);
                
                // Send email
                emailService.sendReport(reportId, reportData);
                
            } catch (Exception e) {
                // Handle error
                log.error("Failed to generate report: " + reportId, e);
            }
        }, 0); // Execute immediately
    }
    
    /**
     * Scheduled task
     */
    @Scheduled(cron = "0 0 2 * * ?") // Run at 2 AM daily
    public void dailyReportCleanup() {
        // Cleanup old reports
        reportGenerator.cleanupOldReports(30); // 30 days
    }
}
```

### 6. Event-Driven Architecture

```java
// Domain event
public class UserCreatedEvent {
    private final Long userId;
    private final String email;
    private final LocalDateTime createdAt;
    
    public UserCreatedEvent(Long userId, String email) {
        this.userId = userId;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }
}

// Event publisher
@Component
public class UserEventPublisher {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void publishUserCreated(UserDO user) {
        UserCreatedEvent event = new UserCreatedEvent(user.getId(), user.getEmail());
        eventPublisher.publishEvent(event);
    }
}

// Event listener
@Component
public class WelcomeEmailListener {
    
    @Autowired
    private EmailService emailService;
    
    @EventListener
    public void handleUserCreated(UserCreatedEvent event) {
        // Send welcome email
        emailService.sendWelcomeEmail(event.getEmail());
    }
}
```

## Service Layer Patterns

### 1. Template Method Pattern

```java
public abstract class AbstractOrderService {
    
    public final Long processOrder(OrderDTO orderDTO) {
        // Template method
        validateOrder(orderDTO);
        OrderDO order = createOrder(orderDTO);
        processPayment(order);
        completeOrder(order);
        return order.getId();
    }
    
    protected abstract void validateOrder(OrderDTO orderDTO);
    
    protected abstract OrderDO createOrder(OrderDTO orderDTO);
    
    protected abstract void processPayment(OrderDO order);
    
    protected void completeOrder(OrderDO order) {
        // Common logic
        order.setStatus(OrderStatus.COMPLETED);
    }
}
```

### 2. Strategy Pattern

```java
public interface PaymentStrategy {
    void processPayment(PaymentDTO payment);
}

@Component
public class AlipayStrategy implements PaymentStrategy {
    @Override
    public void processPayment(PaymentDTO payment) {
        // Alipay payment logic
    }
}

@Component
public class WeChatPayStrategy implements PaymentStrategy {
    @Override
    public void processPayment(PaymentDTO payment) {
        // WeChat Pay logic
    }
}

@Service
public class PaymentService {
    
    @Autowired
    private Map<String, PaymentStrategy> strategyMap;
    
    public void processPayment(PaymentDTO payment) {
        PaymentStrategy strategy = strategyMap.get(payment.getPaymentMethod());
        if (strategy == null) {
            throw new BusinessException("INVALID_PAYMENT_METHOD", 
                "Unsupported payment method: " + payment.getPaymentMethod());
        }
        strategy.processPayment(payment);
    }
}
```

## Testing Service Layer

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceImplTest {
    
    @Autowired
    private UserService userService;
    
    @MockBean
    private UserMapper userMapper;
    
    @Test
    public void testCreateUser_Success() {
        // Arrange
        UserDTO userDTO = new UserDTO();
        userDTO.setName("Test User");
        userDTO.setEmail("test@example.com");
        
        UserDO userDO = new UserDO();
        userDO.setId(1L);
        
        when(userMapper.insert(any(UserDO.class))).thenReturn(1);
        
        // Act
        Long userId = userService.createUser(userDTO);
        
        // Assert
        assertNotNull(userId);
        assertEquals(1L, userId.longValue());
        verify(userMapper, times(1)).insert(any(UserDO.class));
    }
    
    @Test(expected = UserNotFoundException.class)
    public void testGetUserById_NotFound() {
        // Arrange
        when(userMapper.findById(1L)).thenReturn(null);
        
        // Act
        userService.getUserById(1L);
    }
}
```

## Performance Optimization

### 1. Caching

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Cacheable(value = "users", key = "#userId")
    @Override
    public UserDTO getUserById(Long userId) {
        UserDO userDO = userMapper.findById(userId);
        return convertToDTO(userDO);
    }
    
    @CacheEvict(value = "users", key = "#userId")
    @Override
    public void updateUser(Long userId, UserDTO userDTO) {
        // Update logic
    }
}
```

### 2. Batch Processing

```java
@Service
public class BatchServiceImpl implements BatchService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    @Transactional(rollbackFor = Exception.class)
    public void batchCreateUsers(List<UserDTO> userDTOs) {
        List<UserDO> userDOs = userDTOs.stream()
            .map(this::convertToDO)
            .collect(Collectors.toList());
        
        // Batch insert
        userMapper.batchInsert(userDOs);
    }
}
```
