# Hibernate ORM Complete Guide

## Table of Contents
1. [What is ORM?](#what-is-orm)
2. [Hibernate Architecture](#hibernate-architecture)
3. [Entity Annotations](#entity-annotations)
4. [Entity Lifecycle](#entity-lifecycle)
5. [EntityManager Operations](#entitymanager-operations)
6. [JPQL (Java Persistence Query Language)](#jpql)
7. [Transactions](#transactions)
8. [Cascading Operations](#cascading-operations)
9. [Inheritance Strategies](#inheritance-strategies)
10. [Project Structure](#project-structure)
11. [Code Examples](#code-examples)

---

## 1. What is ORM?

**ORM (Object-Relational Mapping)** is a programming technique that converts data between incompatible type systems (object-oriented programming languages and relational databases).

### Key Concepts:
- **Classes → Tables**: Java classes are mapped to database tables
- **Objects → Rows**: Object instances are mapped to table rows
- **Fields → Columns**: Class fields are mapped to table columns
- **Relationships**: Object references are mapped to foreign keys

### Benefits:
- Eliminates boilerplate JDBC code
- Database independence
- Automatic SQL generation
- Object-oriented queries (JPQL)
- Caching mechanisms
- Lazy loading support

### Diagram: ORM Mapping

```
┌─────────────────────────┐           ┌─────────────────────────┐
│   Java Application      │           │   Relational Database   │
│                         │           │                         │
│  ┌─────────────────┐   │           │  ┌─────────────────┐   │
│  │   Employee      │   │  ◄────►   │  │   EMPLOYEE      │   │
│  ├─────────────────┤   │           │  ├─────────────────┤   │
│  │ - id: Long      │───┼───────────┼──│ - ID (PK)       │   │
│  │ - name: String  │───┼───────────┼──│ - NAME          │   │
│  │ - salary: Double│───┼───────────┼──│ - SALARY        │   │
│  │ - dept: Dept    │───┼───────────┼──│ - DEPT_ID (FK)  │   │
│  └─────────────────┘   │           │  └─────────────────┘   │
│                         │           │                         │
└─────────────────────────┘           └─────────────────────────┘
```

---

## 2. Hibernate Architecture

### Diagram: Hibernate Architecture Layers

```
┌────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│              (Your Java/Spring Boot Code)                   │
└──────────────────────┬─────────────────────────────────────┘
                       │
┌──────────────────────▼─────────────────────────────────────┐
│                  Persistence Layer                          │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              EntityManagerFactory                     │  │
│  │        (Creates EntityManager instances)             │  │
│  └────────────────────┬─────────────────────────────────┘  │
│                       │                                     │
│  ┌────────────────────▼─────────────────────────────────┐  │
│  │              EntityManager                            │  │
│  │   - persist()  - find()  - merge()  - remove()       │  │
│  └────────────────────┬─────────────────────────────────┘  │
│                       │                                     │
│  ┌────────────────────▼─────────────────────────────────┐  │
│  │           Persistence Context                         │  │
│  │        (First-level cache / Session)                  │  │
│  └────────────────────┬─────────────────────────────────┘  │
└───────────────────────┼──────────────────────────────────┘
                        │
┌───────────────────────▼──────────────────────────────────┐
│                  Hibernate Core                           │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │   Session    │  │  Transaction │  │     Query     │  │
│  │   Factory    │  │   Manager    │  │   Processor   │  │
│  └──────────────┘  └──────────────┘  └───────────────┘  │
│                                                           │
│  ┌────────────────────────────────────────────────────┐  │
│  │          JDBC Connection Pool                       │  │
│  └────────────────────┬───────────────────────────────┘  │
└────────────────────────┼──────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────┐
│                 Database (MySQL/PostgreSQL/H2)         │
└────────────────────────────────────────────────────────┘
```

### Key Components:

1. **EntityManagerFactory**: Thread-safe, created once per application
2. **EntityManager**: Not thread-safe, manages entity lifecycle
3. **Persistence Context**: First-level cache, tracks managed entities
4. **Transaction**: Ensures ACID properties
5. **JDBC Connection Pool**: Manages database connections

---

## 3. Entity Annotations

### Core JPA Annotations

#### @Entity
Marks a class as a JPA entity (persistent domain object)

```java
@Entity
public class Employee {
    // fields, getters, setters
}
```

#### @Table
Specifies the table name and schema

```java
@Entity
@Table(name = "employees", schema = "hr_schema")
public class Employee {
    // ...
}
```

#### @Id
Marks the primary key field

```java
@Entity
public class Employee {
    @Id
    private Long id;
}
```

#### @GeneratedValue
Specifies primary key generation strategy

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

**Generation Strategies:**
- `IDENTITY`: Database auto-increment
- `SEQUENCE`: Database sequence
- `TABLE`: Separate table for ID generation
- `AUTO`: Provider chooses strategy

#### @Column
Maps field to specific column with properties

```java
@Column(
    name = "employee_name",
    nullable = false,
    unique = true,
    length = 100,
    columnDefinition = "VARCHAR(100) DEFAULT 'Unknown'"
)
private String name;
```

### Relationship Annotations

#### @OneToOne
```java
@Entity
public class Employee {
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id")
    private Address address;
}
```

#### @OneToMany
```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees;
}
```

#### @ManyToOne
```java
@Entity
public class Employee {
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

#### @ManyToMany
```java
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses;
}
```

### Temporal Annotations

```java
@Temporal(TemporalType.DATE)
private Date birthDate;

@Temporal(TemporalType.TIMESTAMP)
private Date createdAt;
```

### Enumerated

```java
@Enumerated(EnumType.STRING)
private EmployeeStatus status;
```

### Transient

```java
@Transient
private int calculatedAge;
```

---

## 4. Entity Lifecycle

### Diagram: Entity States

```
┌─────────────────────────────────────────────────────────────┐
│                     Entity Lifecycle                        │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │     NEW      │  ← Object created with 'new' keyword
    │  (Transient) │     Not associated with EntityManager
    └───────┬──────┘     No database representation
            │
            │ persist()
            ▼
    ┌──────────────┐
    │   MANAGED    │  ← Tracked by Persistence Context
    │ (Persistent) │     Changes auto-synced to DB
    └───────┬──────┘     First-level cache active
            │
            ├──────────────────┐
            │                  │
            │ remove()         │ detach() / clear() / close()
            ▼                  ▼
    ┌──────────────┐    ┌──────────────┐
    │   REMOVED    │    │   DETACHED   │
    │              │    │              │
    └──────────────┘    └──────┬───────┘
            │                  │
            │                  │ merge()
            │                  │
            │ commit()         │
            ▼                  ▼
    ┌──────────────────────────────────┐
    │        Database Synced           │
    └──────────────────────────────────┘
```

### State Descriptions

1. **NEW (Transient)**
   - Created with `new` keyword
   - Not associated with EntityManager
   - No database representation
   - No automatic persistence

2. **MANAGED (Persistent)**
   - Associated with EntityManager
   - Changes are tracked
   - Automatically synchronized with database
   - Part of first-level cache

3. **DETACHED**
   - Was managed but no longer in Persistence Context
   - Still has database representation
   - Changes not tracked
   - Can be reattached with `merge()`

4. **REMOVED**
   - Scheduled for deletion
   - Will be deleted on transaction commit
   - Still in Persistence Context until commit

### Code Example

```java
// NEW state
Employee emp = new Employee();
emp.setName("John Doe");

// MANAGED state (after persist)
entityManager.persist(emp);
emp.setSalary(50000.0); // Change tracked automatically

// DETACHED state
entityManager.detach(emp);
emp.setSalary(60000.0); // Change NOT tracked

// Back to MANAGED (merge)
Employee managed = entityManager.merge(emp);

// REMOVED state
entityManager.remove(managed);
```

---

## 5. EntityManager Operations

### Diagram: EntityManager Methods

```
┌─────────────────────────────────────────────────────────────┐
│                    EntityManager API                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  persist(entity)    → Makes entity MANAGED (INSERT)         │
│                       Throws exception if entity exists     │
│                                                              │
│  find(Class, id)    → Retrieves entity by PK (SELECT)       │
│                       Returns null if not found             │
│                       Result is MANAGED                     │
│                                                              │
│  merge(entity)      → Merges DETACHED entity (UPDATE)       │
│                       Returns new MANAGED instance          │
│                       Original remains DETACHED             │
│                                                              │
│  remove(entity)     → Marks entity as REMOVED (DELETE)      │
│                       Entity must be MANAGED                │
│                       Actual delete on commit               │
│                                                              │
│  detach(entity)     → Makes entity DETACHED                 │
│                       Removes from Persistence Context      │
│                                                              │
│  refresh(entity)    → Reloads from database                 │
│                       Discards in-memory changes            │
│                                                              │
│  flush()            → Synchronizes context with DB          │
│                       Executes pending SQL                  │
│                                                              │
│  clear()            → Detaches all entities                 │
│                       Clears Persistence Context            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Method Details

#### persist()
```java
Employee emp = new Employee("John", 50000);
entityManager.persist(emp);  // INSERT into database
// emp is now MANAGED
```

#### find()
```java
Employee emp = entityManager.find(Employee.class, 1L);
// SELECT * FROM employees WHERE id = 1
// Returns null if not found
```

#### merge()
```java
// emp is DETACHED
emp.setSalary(60000);
Employee managed = entityManager.merge(emp);
// emp remains DETACHED
// managed is the new MANAGED instance
```

#### remove()
```java
Employee emp = entityManager.find(Employee.class, 1L);
entityManager.remove(emp);  // DELETE from database
```

#### getReference()
```java
Employee proxy = entityManager.getReference(Employee.class, 1L);
// Returns lazy-loaded proxy
// Database hit only when accessed
```

---

## 6. JPQL (Java Persistence Query Language)

### What is JPQL?

JPQL is an **object-oriented query language** that operates on entities (not tables).

### Comparison: SQL vs JPQL

```sql
-- SQL (table-oriented)
SELECT e.id, e.name, e.salary 
FROM employees e 
WHERE e.salary > 50000

-- JPQL (entity-oriented)
SELECT e 
FROM Employee e 
WHERE e.salary > 50000
```

### JPQL Examples

#### Basic Query
```java
TypedQuery<Employee> query = entityManager.createQuery(
    "SELECT e FROM Employee e WHERE e.salary > :salary",
    Employee.class
);
query.setParameter("salary", 50000.0);
List<Employee> employees = query.getResultList();
```

#### Join Query
```java
String jpql = "SELECT e FROM Employee e " +
              "JOIN e.department d " +
              "WHERE d.name = :deptName";
              
TypedQuery<Employee> query = entityManager.createQuery(jpql, Employee.class);
query.setParameter("deptName", "IT");
```

#### Aggregate Functions
```java
String jpql = "SELECT AVG(e.salary) FROM Employee e";
Double avgSalary = entityManager.createQuery(jpql, Double.class)
                                .getSingleResult();
```

#### Named Query
```java
@Entity
@NamedQuery(
    name = "Employee.findByDepartment",
    query = "SELECT e FROM Employee e WHERE e.department.name = :deptName"
)
public class Employee {
    // ...
}

// Usage
List<Employee> employees = entityManager
    .createNamedQuery("Employee.findByDepartment", Employee.class)
    .setParameter("deptName", "IT")
    .getResultList();
```

---

## 7. Transactions

### What is a Transaction?

A transaction is a unit of work that follows **ACID** properties:
- **Atomicity**: All or nothing
- **Consistency**: Data remains valid
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Changes persist after commit

### Transaction Management in Spring

```java
@Service
@Transactional  // Class-level: applies to all methods
public class EmployeeService {
    
    @Autowired
    private EntityManager entityManager;
    
    // Method inherits @Transactional from class
    public void createEmployee(Employee emp) {
        entityManager.persist(emp);
    }
    
    // Override with read-only
    @Transactional(readOnly = true)
    public Employee findEmployee(Long id) {
        return entityManager.find(Employee.class, id);
    }
    
    // Custom propagation and isolation
    @Transactional(
        propagation = Propagation.REQUIRES_NEW,
        isolation = Isolation.SERIALIZABLE,
        timeout = 30,
        rollbackFor = Exception.class
    )
    public void complexOperation() {
        // ...
    }
}
```

### Programmatic Transaction

```java
@PersistenceContext
private EntityManager entityManager;

@Autowired
private PlatformTransactionManager transactionManager;

public void manualTransaction() {
    TransactionStatus status = transactionManager.getTransaction(
        new DefaultTransactionDefinition()
    );
    
    try {
        Employee emp = new Employee("John", 50000);
        entityManager.persist(emp);
        transactionManager.commit(status);
    } catch (Exception e) {
        transactionManager.rollback(status);
        throw e;
    }
}
```

---

## 8. Cascading Operations

### Cascade Types

Cascading propagates operations from parent to child entities.
“Cascading in JPA allows operations performed on a parent entity to automatically propagate to its child entities. Common types include PERSIST, MERGE, REMOVE, and ALL, and they control how operations like save, update, and delete are applied across relationships.”

```
┌───────────────────────────────────────────────────────────────────────────┐
│                    Cascade Types                                          │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ALL        → All operations cascaded            -> Everything            │
│  PERSIST    → persist() cascaded to children     -> Save only             │
│  MERGE      → merge() cascaded to children       -> Update only           │
│  REMOVE     → remove() cascaded to children      -> Delete only           │
│  REFRESH    → refresh() cascaded to children     -> Reload from DB        │
│  DETACH     → detach() cascaded to children      -> Remove from session   │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
| EntityManager Method | Cascade Type | Effect          |
| -------------------- | ------------ | --------------- |
| persist()            | PERSIST      | Insert children |
| merge()              | MERGE        | Update children |
| remove()             | REMOVE       | Delete children |
| refresh()            | REFRESH      | Reload children |
| detach()             | DETACH       | Detach children |
| find()               | ❌ No        | No cascade      |
| flush()              | ❌ No        | No cascade      |
| clear()              | ❌ No        | No cascade      |


Note:- 
    “While JPA allows cascading from child to parent, it is generally discouraged because it can lead to unintended operations like deleting a parent when a child is removed. Cascading should typically be defined from parent to child, reflecting ownership.”
```

### Example: Department and Employees

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,  // Cascade all operations
        orphanRemoval = true         // Delete orphaned children
    )
    private List<Employee> employees = new ArrayList<>();
    
    // Helper method
    public void addEmployee(Employee emp) {
        employees.add(emp);
        emp.setDepartment(this);
    }
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}

// Usage
Department dept = new Department("IT");
dept.addEmployee(new Employee("John"));
dept.addEmployee(new Employee("Jane"));

entityManager.persist(dept);  // All employees persisted automatically
```

### Cascade Diagram

```
Parent: Department
    │
    │ cascade = CascadeType.ALL
    │
    ├── persist(dept) ──────► persist(emp1)
    │                         persist(emp2)
    │
    ├── merge(dept) ────────► merge(emp1)
    │                         merge(emp2)
    │
    └── remove(dept) ───────► remove(emp1)
                              remove(emp2)
```

---

## 9. Inheritance Strategies

JPA supports three inheritance mapping strategies.

### 9.1 SINGLE_TABLE Strategy

**All classes in hierarchy stored in ONE table**

#### Diagram
```
┌────────────────────────────────────────────────────────────┐
│                    SINGLE_TABLE                            │
│                    (One table)                             │
├────────────────────────────────────────────────────────────┤
│  ID │ DTYPE │ NAME    │ SALARY │ BONUS │ HOURLY_RATE │    │
├─────┼───────┼─────────┼────────┼───────┼─────────────┤    │
│  1  │ FTE   │ John    │ 50000  │ 5000  │ NULL        │    │
│  2  │ CTR   │ Jane    │ NULL   │ NULL  │ 100         │    │
│  3  │ FTE   │ Bob     │ 60000  │ 6000  │ NULL        │    │
└────────────────────────────────────────────────────────────┘
```

#### Code
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE", discriminatorType = DiscriminatorType.STRING)
public abstract class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}

@Entity
@DiscriminatorValue("FTE")
public class FullTimeEmployee extends Employee {
    private Double salary;
    private Double bonus;
}

@Entity
@DiscriminatorValue("CTR")
public class ContractEmployee extends Employee {
    private Double hourlyRate;
}
```

**Pros:**
- Best performance (no joins)
- Polymorphic queries are fast

**Cons:**
- NOT NULL constraints difficult
- Table can become wide with many columns

---

### 9.2 JOINED Strategy

**Each class has its own table, joined via foreign keys**

#### Diagram
```
┌──────────────────────┐
│   EMPLOYEE (parent)  │
├──────────────────────┤
│ ID │ NAME           │
├────┼────────────────┤
│ 1  │ John           │
│ 2  │ Jane           │
│ 3  │ Bob            │
└────┴────────────────┘
      │
      ├───────────────────────────┐
      │                           │
      ▼                           ▼
┌──────────────────────┐  ┌────────────────────────┐
│ FULL_TIME_EMPLOYEE   │  │ CONTRACT_EMPLOYEE      │
├──────────────────────┤  ├────────────────────────┤
│ ID(PK,FK)│SALARY│BONUS│ │ ID(PK,FK)│HOURLY_RATE │
├──────────┼──────┼────┤  ├──────────┼─────────────┤
│ 1        │50000 │5000│  │ 2        │ 100         │
│ 3        │60000 │6000│  └──────────┴─────────────┘
└──────────┴──────┴────┘
```

#### Code
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}

@Entity
@Table(name = "full_time_employee")
public class FullTimeEmployee extends Employee {
    private Double salary;
    private Double bonus;
}

@Entity
@Table(name = "contract_employee")
public class ContractEmployee extends Employee {
    private Double hourlyRate;
}
```

**Pros:**
- Normalized database design
- Supports NOT NULL constraints properly
- Clear separation of concerns

**Cons:**
- Requires joins for queries
- Slower performance than SINGLE_TABLE

---

### 9.3 TABLE_PER_CLASS Strategy

**Each concrete class has its own table with ALL fields (including inherited)**

#### Diagram
```
┌────────────────────────────────┐
│   FULL_TIME_EMPLOYEE (table)   │
├────────────────────────────────┤
│ ID │ NAME    │ SALARY │ BONUS │
├────┼─────────┼────────┼───────┤
│ 1  │ John    │ 50000  │ 5000  │
│ 3  │ Bob     │ 60000  │ 6000  │
└────────────────────────────────┘

┌────────────────────────────────┐
│   CONTRACT_EMPLOYEE (table)    │
├────────────────────────────────┤
│ ID │ NAME    │ HOURLY_RATE    │
├────┼─────────┼────────────────┤
│ 2  │ Jane    │ 100            │
└────────────────────────────────┘
```

#### Code
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)  // Must use TABLE or AUTO
    private Long id;
    private String name;
}

@Entity
@Table(name = "full_time_employee")
public class FullTimeEmployee extends Employee {
    private Double salary;
    private Double bonus;
}

@Entity
@Table(name = "contract_employee")
public class ContractEmployee extends Employee {
    private Double hourlyRate;
}
```

**Pros:**
- No joins needed
- Each table is independent

**Cons:**
- Polymorphic queries require UNION
- Data duplication (inherited fields in each table)
- Cannot use IDENTITY generation

---

## 10. Project Structure

```
hibernate-orm-complete/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/epam/hibernate/
│   │   │       ├── HibernateOrmApplication.java
│   │   │       ├── config/
│   │   │       │   └── DatabaseConfig.java
│   │   │       ├── entity/
│   │   │       │   ├── basic/
│   │   │       │   │   ├── Employee.java
│   │   │       │   │   ├── Department.java
│   │   │       │   │   └── Address.java
│   │   │       │   ├── inheritance/
│   │   │       │   │   ├── singletable/
│   │   │       │   │   │   ├── PaymentSingleTable.java
│   │   │       │   │   │   ├── CreditCardPayment.java
│   │   │       │   │   │   └── CashPayment.java
│   │   │       │   │   ├── joined/
│   │   │       │   │   │   ├── VehicleJoined.java
│   │   │       │   │   │   ├── Car.java
│   │   │       │   │   │   └── Bike.java
│   │   │       │   │   └── tableperclass/
│   │   │       │   │       ├── ProductTablePerClass.java
│   │   │       │   │       ├── Book.java
│   │   │       │   │       └── Electronics.java
│   │   │       ├── repository/
│   │   │       │   ├── EmployeeRepository.java
│   │   │       │   └── DepartmentRepository.java
│   │   │       ├── service/
│   │   │       │   ├── EmployeeService.java
│   │   │       │   ├── DepartmentService.java
│   │   │       │   └── EntityLifecycleService.java
│   │   │       └── controller/
│   │   │           └── EmployeeController.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── data.sql
│   └── test/
│       └── java/
│           └── com/epam/hibernate/
│               └── HibernateOrmApplicationTests.java
├── pom.xml
└── README.md
```

---

## 11. Code Examples

See the `src/` directory for complete working examples of:

1. Basic entity mapping with annotations
2. EntityManager CRUD operations
3. Entity lifecycle demonstrations
4. JPQL queries (basic, joins, aggregates, named queries)
5. Transaction management
6. Cascading operations
7. All three inheritance strategies
8. Spring Data JPA repositories
9. RESTful controllers

---

## Running the Project

1. **Build the project:**
   ```bash
   mvn clean install
   ```

2. **Run the application:**
   ```bash
   mvn spring-boot:run
   ```

3. **Access H2 Console:**
   ```
   URL: http://localhost:8080/h2-console
   JDBC URL: jdbc:h2:mem:testdb
   Username: sa
   Password: 
   ```

4. **Test REST endpoints:**
   ```bash
   # Get all employees
   curl http://localhost:8080/api/employees
   
   # Create employee
   curl -X POST http://localhost:8080/api/employees \
   -H "Content-Type: application/json" \
   -d '{"name":"John Doe","salary":50000}'
   ```

---

## Summary

This project demonstrates:
- Complete ORM mapping from Java classes to database tables
- All JPA annotations and their usage
- Entity lifecycle management (NEW → MANAGED → DETACHED → REMOVED)
- EntityManager operations (persist, find, merge, remove)
- JPQL for object-oriented queries
- Transaction management with Spring
- Cascading operations for related entities
- Three inheritance strategies (SINGLE_TABLE, JOINED, TABLE_PER_CLASS)

Each concept is accompanied by working code, diagrams, and detailed explanations.
