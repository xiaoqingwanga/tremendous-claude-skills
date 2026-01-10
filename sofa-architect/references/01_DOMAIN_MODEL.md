# Domain Model Documentation

## Overview
This document describes the core domain entities and their relationships for the SOFA Boot application.

## Domain Entities

### Entity Structure
Each domain entity should include:
- **Purpose**: What the entity represents in the business domain
- **Fields**: List of all fields with types and descriptions
- **Relationships**: Relationships to other entities (OneToOne, OneToMany, ManyToMany)
- **Business Rules**: Validation rules and constraints
- **Lifecycle**: Creation, update, and deletion behavior

### Example Entity Template

```java
/**
 * Entity description and purpose
 */
@Entity
@Table(name = "table_name")
public class EntityNameDO {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Fields with proper annotations
    // Use BigDecimal for monetary values
    // Use appropriate types for all fields
    
    // Relationships
    @OneToMany(mappedBy = "fieldName")
    private List<RelatedEntity> relatedEntities;
}
```

## Domain Model Principles

1. **Rich Domain Model**: Domain entities should contain business logic, not just data
2. **Immutable Where Possible**: Use immutable objects for value objects
3. **Validation**: Use Bean Validation annotations (@NotNull, @Size, @Pattern, etc.)
4. **No Database Concerns**: Domain entities should not contain persistence logic
5. **Type Safety**: Use specific types (e.g., Money instead of BigDecimal for currency)

## Value Objects

Identify and extract value objects for concepts that:
- Have no identity
- Are immutable
- Can be shared across entities
- Have validation rules

Example: Money, Email, PhoneNumber, Address

## Aggregates

Define aggregate roots and ensure:
- Only aggregate roots are accessed from outside
- Invariants are maintained within the aggregate
- Transactions are scoped to a single aggregate

## Domain Events

Define domain events for important business occurrences:
```java
public class DomainEvent {
    private final LocalDateTime occurredOn;
    private final String eventType;
    // Event data
}
```
