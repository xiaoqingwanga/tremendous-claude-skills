# Data Layer Documentation

## Overview
This document describes the ZDAL and MyBatis configuration for data access in the SOFA Boot application.

## ZDAL Configuration

### Database Connection Setup

```yaml
# application.properties
zdal.datasource.url=jdbc:mysql://localhost:3306/database_name
zdal.datasource.username=username
zdal.datasource.password=password
zdal.datasource.driver-class-name=com.mysql.jdbc.Driver
```

### ZDAL Routing Configuration

For multi-tenancy or sharding:

```java
@Configuration
public class ZdalConfig {
    
    @Bean
    public DataSource zdalDataSource() {
        // Configure ZDAL data source with routing rules
        return ZdalDataSourceBuilder.create()
            .withRule("sharding_rule.xml")
            .build();
    }
}
```

### Sharding Rules Example

```xml
<!-- sharding_rule.xml -->
<rule>
    <tableRule name="user_table_rule">
        <rule>
            <column>user_id</column>
            <algorithm>mod-long</algorithm>
        </rule>
    </tableRule>
    
    <tableRule name="order_table_rule">
        <rule>
            <column>order_id</column>
            <algorithm>mod-long</algorithm>
        </rule>
    </tableRule>
</rule>
```

## MyBatis Mapper Configuration

### Mapper Interface

```java
@Mapper
public interface UserMapper {
    
    /**
     * Insert user record
     */
    @Insert("INSERT INTO user (id, name, email, created_at) " +
            "VALUES (#{id}, #{name}, #{email}, #{createdAt})")
    int insert(UserDO user);
    
    /**
     * Find user by ID
     */
    @Select("SELECT * FROM user WHERE id = #{id}")
    UserDO findById(Long id);
    
    /**
     * Find users with pagination
     */
    @Select("SELECT * FROM user LIMIT #{offset}, #{limit}")
    List<UserDO> findWithPagination(@Param("offset") int offset, 
                                      @Param("limit") int limit);
    
    /**
     * Update user
     */
    @Update("UPDATE user SET name = #{name}, email = #{email} " +
            "WHERE id = #{id}")
    int update(UserDO user);
    
    /**
     * Delete user
     */
    @Delete("DELETE FROM user WHERE id = #{id}")
    int deleteById(Long id);
}
```

### Complex Queries with XML

```xml
<!-- UserMapper.xml -->
<mapper namespace="com.example.mapper.UserMapper">
    
    <resultMap id="UserResultMap" type="com.example.model.UserDO">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="email" column="email"/>
        <result property="createdAt" column="created_at"/>
    </resultMap>
    
    <select id="findUsersByCriteria" resultMap="UserResultMap">
        SELECT * FROM user
        <where>
            <if test="name != null">
                AND name LIKE CONCAT('%', #{name}, '%')
            </if>
            <if test="email != null">
                AND email = #{email}
            </if>
        </where>
        ORDER BY created_at DESC
    </select>
    
</mapper>
```

## Data Access Best Practices

1. **Always Use ZDAL**: Never use raw JDBC connections
2. **Mapper Interfaces Only**: All database access through Mapper interfaces
3. **DTO Pattern**: Never return DO (Database Object) from facade layers
4. **Transaction Management**: Use SOFA's @Transactional annotation
5. **Connection Pooling**: Configure appropriate pool settings
6. **Query Optimization**: Use EXPLAIN to analyze slow queries
7. **Index Strategy**: Create indexes based on query patterns

## Transaction Management

```java
@Service
public class UserServiceImpl implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Transactional(rollbackFor = Exception.class)
    public void createUserWithProfile(UserDO user, UserProfileDO profile) {
        userMapper.insert(user);
        profile.setUserId(user.getId());
        // profileMapper.insert(profile);
    }
}
```

## Database Schema Guidelines

### Naming Conventions
- Table names: snake_case (e.g., `user_profile`)
- Column names: snake_case (e.g., `created_at`)
- Primary keys: `id` (auto-increment or sequence)
- Foreign keys: `{table}_id` (e.g., `user_id`)

### Common Columns
```sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
is_deleted TINYINT(1) DEFAULT 0
```

### Index Creation
```sql
-- Single column index
CREATE INDEX idx_user_email ON user(email);

-- Composite index
CREATE INDEX idx_order_user_status ON order(user_id, status);

-- Unique index
CREATE UNIQUE INDEX idx_user_email ON user(email);
```

## Data Migration

Use Flyway or Liquibase for database migrations:

```sql
-- V1__create_user_table.sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
