# JPA, Hibernate, Entity, and Repository

## 1. The Core Problem: Object-Relational Impedance Mismatch

Java code works with **objects** (fields, references), while relational databases work with **tables, rows, and columns**. This mismatch means without an abstraction layer, you'd manually write SQL and manually convert `ResultSet` rows into Java objects for every single operation:

```java
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setLong(1, id);
ResultSet rs = stmt.executeQuery();
User user = new User();
user.setId(rs.getLong("id"));
user.setName(rs.getString("name"));
```

This is repetitive across a real app with many entities/queries. **ORM (Object-Relational Mapping)** tools were built to automate this — JPA and Hibernate are Java's answer to this problem.

---

## 2. JPA vs Hibernate

### JPA (Jakarta Persistence API)
- **A specification only** — a set of interfaces, annotations, and rules defined by Jakarta EE (formerly Sun/Oracle).
- Defines things like `@Entity`, `@Table`, `@Id`, `@Column`, the `EntityManager` interface, and JPQL (query language).
- **Has no implementation of its own** — like an interface with no class behind it. Cannot run standalone.

### Hibernate
- The most popular **implementation** of the JPA specification (others exist, e.g., EclipseLink, but Hibernate dominates and is Spring Boot's default).
- Historically, **Hibernate existed before JPA** — it was so influential that Sun standardized much of its approach into the JPA spec.
- Hibernate implements JPA but also offers some proprietary features beyond plain JPA (in Spring Boot, you mostly stick to JPA-standard parts for portability).

### The Relationship

JPA        = the contract/spec (interfaces, annotations, rules)
Hibernate  = the actual working implementation that fulfills that contract

**Analogy**: Like `List` (interface) vs `ArrayList` (implementation) in Java Collections — you code against JPA's interfaces/annotations, but Hibernate does the real work underneath.

---

## 3. `@Entity` — Mapping Java Classes to Database Tables

An **entity** is a Java class representing a database **table**, where each instance represents a **row**.

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Column(name = "user_email", nullable = false, unique = true, length = 100)
    private String email;

    // no-arg constructor, getters, setters
}
```

### Required Pieces

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks the class as persistable. Default table name = class name (`User` → `user`); override with `@Table(name = "users")` |
| `@Id` | Marks the primary key field — **every entity must have one** |
| `@GeneratedValue` | Defines how the primary key is generated |
| No-arg constructor | Required — Hibernate instantiates via reflection, then populates fields |

### `@GeneratedValue` Strategies
- `IDENTITY` — relies on DB auto-increment column (common with MySQL)
- `SEQUENCE` — uses a DB sequence object (common with PostgreSQL/Oracle)
- `AUTO` — Hibernate picks the strategy based on the underlying database

### Field-to-Column Mapping
By default, field name = column name. Override with:
```java
@Column(name = "user_email")
private String email;
```
Also used for constraints: `nullable`, `unique`, `length`, etc.

### Entity Lifecycle States (important interview topic)

| State | Meaning |
|---|---|
| **Transient** | Plain object via `new User()`, not associated with any persistence context — Hibernate doesn't know it exists |
| **Managed (Persistent)** | Saved/persisted or loaded from DB; tracked inside the **Persistence Context** (Hibernate's internal session). Field changes are auto-detected and synced to DB (**dirty checking**) |
| **Detached** | Was managed, but the persistence context/session has since closed (e.g., transaction ended). Still exists in memory but changes are no longer tracked |
| **Removed** | Marked for deletion; deleted from DB once transaction commits |

> **Why this matters**: Explains why modifying a detached entity doesn't auto-update the DB (not tracked anymore), and why accessing a lazy-loaded field outside an active persistence context throws `LazyInitializationException`.

### Entity Relationships
- `@OneToMany` / `@ManyToOne` — e.g., one `User` → many `Order`s
- `@OneToOne` — e.g., one `User` → one `Profile`
- `@ManyToMany` — e.g., many `Student`s ↔ many `Course`s

*(Deep-dive on `fetch = LAZY` vs `EAGER` and `mappedBy` — separate topic worth covering on its own.)*

---

## 4. Repository — Talking to the Database

### The Old Way: `EntityManager`
```java
@PersistenceContext
private EntityManager entityManager;

public User findById(Long id) {
    return entityManager.find(User.class, id);
}

public void save(User user) {
    entityManager.persist(user);
}
```
Works, but repetitive — every entity needs the same CRUD boilerplate.

### The Spring Data JPA Way
Declare an **interface** — Spring generates the implementation at runtime automatically:

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

This alone provides, for free:
- `save(User user)`
- `findById(Long id)`
- `findAll()`
- `deleteById(Long id)`
- `count()`

No implementation class needed — Spring creates a **proxy object** at runtime.

### Repository Interface Hierarchy

Repository (marker interface, no methods)
↓
CrudRepository<T, ID>              — basic CRUD: save, findById, findAll, delete, count
↓
PagingAndSortingRepository<T, ID>  — adds pagination and sorting
↓
JpaRepository<T, ID>               — adds JPA-specific extras: batch operations, flush(), etc.

In practice, almost always extend `JpaRepository<T, ID>` directly. Generic params = `<EntityType, PrimaryKeyType>` (e.g., `JpaRepository<User, Long>`).

### How Spring Generates the Implementation ("the magic")
1. On startup, Spring scans for interfaces extending `Repository` (or sub-interfaces) within packages covered by `@EnableJpaRepositories` (auto-configured by `spring-boot-starter-data-jpa`).
2. For each one found, Spring creates a **dynamic proxy** at runtime, backed by `SimpleJpaRepository`.
3. The proxy intercepts method calls and delegates to `SimpleJpaRepository`'s real logic — which internally uses `EntityManager` (the same manual approach, just automated).

### Query Methods (derived from method names)
Spring parses method names following naming conventions to auto-generate queries:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    User findByEmail(String email);

    List<User> findByNameContaining(String keyword);

    List<User> findByAgeGreaterThan(int age);

    List<User> findByNameAndEmail(String name, String email);
}
```
Convention: `findBy` / `existsBy` / `countBy` / `deleteBy` + field name + optional keywords (`And`, `Or`, `GreaterThan`, `Containing`, `OrderBy`, etc.)

### `@Query` — For Complex Queries
When naming conventions aren't enough, write JPQL explicitly (SQL-like, but operates on entity objects/fields, not raw table/column names):

```java
@Query("SELECT u FROM User u WHERE u.email = :email AND u.active = true")
User findActiveUserByEmail(@Param("email") String email);
```

Or drop to raw native SQL:
```java
@Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
User findByEmailNative(@Param("email") String email);
```

---

## Interview Quick-Reference

| Question | Answer |
|---|---|
| Difference between JPA and Hibernate? | JPA = specification (interfaces/rules); Hibernate = the implementation that fulfills it |
| What does `@Entity` do? | Marks a class as a DB-mapped object; each instance = a row in the mapped table |
| Why does an entity need a no-arg constructor? | Hibernate instantiates it via reflection before populating fields |
| What are the JPA entity lifecycle states? | Transient → Managed (Persistent) → Detached → Removed |
| What is dirty checking? | Automatic detection of field changes on a managed entity, synced to DB without explicit save calls |
| Why does modifying a detached entity not update the DB? | It's no longer tracked by the persistence context |
| Difference between `CrudRepository` and `JpaRepository`? | `JpaRepository` extends `PagingAndSortingRepository` (which extends `CrudRepository`), adding JPA-specific features like batch ops and `flush()` |
| How does Spring Data JPA generate repository implementations? | Dynamic proxies at runtime, backed by `SimpleJpaRepository`, which uses `EntityManager` internally |
| What happens if a derived query method name doesn't match an entity field? | Fails fast at **application startup**, not at runtime |
| When to use `@Query` over a derived method name? | When query logic is too complex for naming conventions (joins, aggregations) or method name would get unwieldy |

--------------