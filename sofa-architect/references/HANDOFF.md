# Architecture Handoff Documentation

## Overview
This document provides comprehensive handoff information for AI agents (Director and Implementor) to successfully build and implement SOFA Boot applications.

## Project Context

### Project Information
- **Project Name**: [Project Name]
- **Project Type**: [New/Existing]
- **Tech Stack**: Java 8, SOFA Boot 3.26.0, ZDAL 5.21.1
- **Root Path**: [Project Root Path]
- **Module Structure**: [7-module / Custom]

### Architecture Overview
[Brief description of the system architecture and key components]

## Module Structure

### Module Descriptions

#### 1. Bootstrap Module (`bootstrap/`)
**Purpose**: Application startup and configuration
**Key Components**:
- Main application class with `@SpringBootApplication`
- Configuration classes (ZDAL, SOFA RPC, etc.)
- Property files (`application.properties`, `application-{profile}.properties`)

**Responsibilities**:
- Initialize Spring context
- Configure data sources
- Set up RPC services
- Load application properties

#### 2. Web Module (`web/`)
**Purpose**: HTTP API endpoints and request handling
**Key Components**:
- REST Controllers (`@RestController`)
- Request/Response DTOs
- API documentation (Swagger/OpenAPI)
- Exception handlers

**Responsibilities**:
- Handle HTTP requests
- Validate input
- Delegate to service layer
- Format responses

**Guardrails**:
- ❌ Never put business logic in controllers
- ✅ Always delegate to service layer
- ✅ Use proper HTTP status codes

#### 3. Service Module (`service/`)
**Purpose**: Business logic implementation
**Key Components**:
- Service interfaces and implementations
- Business rules and validations
- Transaction management
- Event publishing

**Responsibilities**:
- Implement business logic
- Coordinate between multiple entities
- Manage transactions
- Publish domain events

**Guardrails**:
- ❌ Never access database directly
- ✅ Always use DAL layer
- ✅ Use DTOs for data transfer

#### 4. DAL Module (`dal/`)
**Purpose**: Data access layer with ZDAL and MyBatis
**Key Components**:
- MyBatis Mapper interfaces
- Mapper XML files
- ZDAL configuration
- Database schemas

**Responsibilities**:
- Database operations
- Query optimization
- Data transformation (DO ↔ DTO)

**Guardrails**:
- ❌ Never use raw JDBC
- ✅ Always use ZDAL
- ✅ Never return DOs from facade

#### 5. Model Module (`model/`)
**Purpose**: Domain models and entities
**Key Components**:
- Domain entities (DO)
- Value objects
- Domain events
- Enums

**Responsibilities**:
- Define domain model
- Encapsulate business rules
- Maintain data integrity

**Guardrails**:
- ❌ Never use float/double for money
- ✅ Always use BigDecimal or Money
- ✅ Use proper data types

#### 6. Utils Module (`utils/`)
**Purpose**: Utility classes and helpers
**Key Components**:
- Common utilities
- Constants
- Helper functions
- Converters

**Responsibilities**:
- Provide reusable utilities
- Common transformations
- Helper methods

#### 7. Facade Module (`facade/`)
**Purpose**: RPC interfaces and DTOs for external/internal communication
**Key Components**:
- RPC service interfaces
- Request/Response DTOs
- API contracts

**Responsibilities**:
- Define RPC contracts
- Expose business logic
- Data transfer objects

**Guardrails**:
- ❌ Never return DOs
- ✅ Always return DTOs
- ✅ Keep interfaces stable

## Architecture Decisions

### Key Design Decisions

1. **7-Module Architecture**: Chosen for clear separation of concerns and maintainability
2. **ZDAL for Data Access**: Provides database sharding and routing capabilities
3. **Interface-Driven Design**: Enables loose coupling and easier testing
4. **Event-Driven Architecture**: Supports async processing and scalability

### Trade-offs Considered
[Document any trade-offs made during architecture design]

## Technology Stack

### Core Technologies
- **Java**: 8
- **SOFA Boot**: 3.26.0
- **ZDAL**: 5.21.1
- **MyBatis**: Latest compatible version
- **Spring Framework**: As included in SOFA Boot

### Additional Libraries
- **Testing**: JUnit, Mockito, Mist SDK
- **Async**: AntScheduler, SOFAMQ
- **Observability**: Tracer
- **RPC**: SOFA RPC

## Data Layer

### Database Schema
[Describe database structure, tables, relationships]

### ZDAL Configuration
[ZDAL routing rules, sharding strategy]

### Key Entities
[List important domain entities and their purposes]

## API Contracts

### REST APIs
[List REST endpoints with descriptions]

### RPC Services
[List RPC interfaces and methods]

### DTOs
[List important DTOs and their structures]

## Business Logic

### Key Business Processes
[Describe main business workflows]

### Business Rules
[Document important business rules and validations]

### Event Handling
[Describe domain events and their handlers]

## Integration Points

### External Systems
[List external system integrations]

### Internal Services
[List internal service dependencies]

### Message Queues
[Describe SOFAMQ usage]

## Deployment

### Environment Configuration
- **Development**: [Configuration details]
- **Staging**: [Configuration details]
- **Production**: [Configuration details]

### Build and Deploy Process
[Describe build and deployment steps]

## Testing Strategy

### Unit Tests
- Service layer tests with mocked dependencies
- Business logic validation
- Edge case handling

### Integration Tests
- Database integration with test containers
- RPC service integration
- End-to-end workflows

### Performance Tests
- Load testing targets
- Response time requirements
- Throughput expectations

## Monitoring and Observability

### Metrics
[Key metrics to monitor]

### Logging
[Logging strategy and levels]

### Tracing
[Tracing configuration for distributed tracing]

## Known Issues and Limitations

### Current Limitations
[List any known limitations]

### Planned Improvements
[List planned enhancements]

## Next Steps for Implementor

### Immediate Tasks
1. [ ] Review architecture documentation
2. [ ] Set up development environment
3. [ ] Implement core domain models
4. [ ] Implement service layer
5. [ ] Implement DAL layer
6. [ ] Implement web layer
7. [ ] Write tests

### Implementation Order
[Suggested order of implementation]

### Dependencies
[List any external dependencies or prerequisites]

## Contact and Support

### Architecture Questions
[Contact information for architecture-related questions]

### Implementation Support
[Contact information for implementation support]

## Appendix

### Glossary
- **DO**: Database Object
- **DTO**: Data Transfer Object
- **VO**: View Object
- **ZDAL**: ZooKeeper-based Data Access Layer
- **SOFA**: Scalable Open Financial Architecture

### References
- [Link to additional documentation]
- [Link to external resources]
