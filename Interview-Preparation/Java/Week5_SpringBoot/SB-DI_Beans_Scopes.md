# Dependency Injection, Beans, and Bean Scopes

## 1. Dependency Injection (DI)

### The Problem DI Solves
Without DI, a class creates its own dependencies directly:
```java
public class UserService {
    private UserRepository userRepository = new UserRepository();
}
```
This causes **tight coupling** with real problems:

1. **Cascading construction responsibility** — If `UserRepository` needs its own dependencies (e.g., a `DataSource`), `UserService` must now know how to build those too, even though that has nothing to do with its actual job (business logic for users).
```java
   private UserRepository userRepository = 
       new UserRepository(new DataSource(url, username, password));
```

2. **Can't swap implementations for testing** — Since `UserService` hardcodes `new UserRepository()`, you can't substitute a mock/fake repository for unit testing. You're stuck hitting the real dependency every time.

3. **Duplicate instances / wasted resources** — If 5 different classes each do `new UserRepository()`, you end up with 5 separate objects (possibly 5 separate DB connections) instead of sharing one.

### What DI Does
Instead of a class creating its own dependencies, an external entity (the **Spring IoC Container**) creates the dependency and *injects* it into the class that needs it.

```java
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
`UserService` no longer knows or cares how `UserRepository` is built — it just receives a ready instance.

> **Key concept**: DI is a specific implementation of the broader principle **Inversion of Control (IoC)** — control over object creation is inverted from your code to the framework/container.

### Types of Dependency Injection

| Type | How it works | Recommended? |
|---|---|---|
| **Constructor Injection** | Dependency passed via constructor | ✅ Yes (preferred) |
| **Setter Injection** | Dependency passed via a setter method after construction | Situational |
| **Field Injection** | `@Autowired` directly on the field | ❌ Discouraged |

```java
// Constructor Injection (preferred)
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// Setter Injection
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// Field Injection
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### Why Constructor Injection is Preferred (common interview question)
- Allows the field to be `final` → **immutability**.
- Object is **never in a half-constructed state** — constructor forces all required dependencies to be provided.
- **Circular dependencies fail fast** at startup instead of causing subtle runtime issues (field injection can mask these).
- Easier to **unit test** — can call `new UserService(mockRepository)` in plain JUnit without needing Spring at all.

### Constructor Resolution Rules
- If a class has **only one constructor**, Spring uses it automatically (implicit since Spring 4.3+, no `@Autowired` needed).
- If a class has **multiple constructors**, one must be explicitly marked `@Autowired`, or Spring throws an exception.

### What `@Autowired` Does Internally
Processed by a `BeanPostProcessor` called `AutowiredAnnotationBeanPostProcessor` — it inspects the constructor/field/setter, determines the required type, and asks the container for a matching bean (resolved by type first, then by name if there's ambiguity).

---

## 2. Beans

### What is a Bean?
A **bean** is a regular Java object whose creation and lifecycle are managed by the **Spring IoC Container**, instead of being manually instantiated with `new`.

> DI is the *mechanism*; **beans are the objects** being managed and injected by that mechanism.

### How Spring Knows What to Turn Into a Bean

**1. Stereotype Annotations** — picked up automatically via component scanning:
```java
@Component
public class UserRepository { ... }

@Service
public class UserService { ... }

@Repository
public class OrderRepository { ... }

@Controller / @RestController
public class UserController { ... }
```
All of these are specializations of `@Component` (meta-annotated with it). They exist for semantic clarity — `@Repository` additionally translates database exceptions into Spring's unified exception hierarchy.

**2. `@Bean` inside a `@Configuration` class** — used when you don't own the class (e.g., third-party library) or need custom construction logic:
```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### Where Beans Live
Beans are stored inside the **ApplicationContext** (the IoC container) — internally, roughly a map keyed by bean name pointing to the object instance. Dependent classes get their dependencies looked up and injected from this container.

### Default Behavior
By default, Spring creates **only one instance** of each bean (singleton scope) and reuses it everywhere — directly solving the "duplicate instances" problem from the DI section, since all classes needing `UserRepository` share the same instance.

---

## 3. ApplicationContext and the IoC Container

### IoC Container (conceptual term)
The general term for whatever Spring component is responsible for **creating objects, wiring dependencies, and managing lifecycle** — the embodiment of the Inversion of Control principle.

### ApplicationContext (concrete implementation)
The actual interface in Spring that implements the IoC container concept. When people say "the Spring container," they mean an `ApplicationContext` instance.
```java
public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
}
```
This bootstraps an `ApplicationContext`, which holds and wires all beans for the app's lifetime.

### BeanFactory vs ApplicationContext

| Feature | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Bean loading | Lazy (created only when requested) | Eager (singletons created at startup) |
| Event publishing | ❌ | ✅ (`ApplicationEvent`, `@EventListener`) |
| Internationalization | ❌ | ✅ |
| Environment abstraction | ❌ | ✅ (properties, profiles) |
| AOP integration | ❌ | ✅ |
| Common usage today | Rarely used directly | Used almost everywhere |

> `ApplicationContext` extends `BeanFactory`. Eager initialization matters because configuration errors (e.g., missing dependency) surface **immediately at startup** rather than at random points during runtime.

Common implementations: `AnnotationConfigApplicationContext` (Java config), and in Spring Boot web apps, something like `AnnotationConfigServletWebServerApplicationContext` (created automatically by `SpringApplication.run()`).

---

## 4. Bean Scopes

Scope defines: **how many instances of a bean exist, and when they're created/destroyed.**

### 1. Singleton (default)
One instance per Spring container, shared everywhere it's injected.
```java
@Component
@Scope("singleton") // redundant, this is the default
public class UserRepository { ... }
```
- "Singleton" here means one-per-**container**, not one-per-JVM (unlike GoF Singleton pattern using private constructor + static instance).
- Since it's shared, singleton beans should be **stateless/thread-safe** — mutable instance fields can cause race conditions across concurrent requests/threads.

### 2. Prototype
A **new instance is created every time** the bean is requested/injected.
```java
@Component
@Scope("prototype")
public class ReportGenerator { ... }
```
- Use case: objects holding request-specific mutable state needing a fresh instance each time.
- **Gotcha**: Spring does NOT manage the full lifecycle of prototype beans after creation — `@PreDestroy` is not automatically called (unlike singletons). Cleanup is the developer's responsibility.

### 3. Request (web-aware)
One instance **per HTTP request**. Only valid in web application contexts. Useful for request-specific data (e.g., a request-tracking ID).

### 4. Session (web-aware)
One instance **per HTTP session** — persists across multiple requests from the same user until session expiry. Useful for things like a shopping cart.

### 5. Application (web-aware)
One instance per `ServletContext` — similar to singleton but scoped to the whole web app rather than just the Spring container. Matters mainly when multiple Spring containers exist within one web app (rare in Boot apps).

### Singleton-Injecting-Prototype Problem (common trick question)
By default, injecting a prototype bean into a singleton bean is problematic — since the singleton is created only once, it receives just **one** instance of the prototype at startup and reuses it forever, defeating the purpose of "prototype."

**Fix**: Use `@Lookup` method injection, or `ObjectProvider<T>` / `Provider<T>` to fetch a fresh prototype instance on each call.

---

## Interview Quick-Reference

| Question | Answer |
|---|---|
| Why is constructor injection preferred? | Immutability (`final` fields), no half-constructed objects, fails fast on circular dependencies, easier unit testing |
| Difference between `@Component`, `@Service`, `@Repository`? | All specializations of `@Component`; mainly semantic, though `@Repository` adds exception translation |
| Difference between `BeanFactory` and `ApplicationContext`? | `BeanFactory` = lazy, basic container; `ApplicationContext` = eager singleton init + events + i18n + environment abstraction |
| What's the default bean scope? | Singleton |
| Why avoid mutable state in singleton beans? | Shared across threads/requests → race conditions |
| Does `@PreDestroy` work on prototype beans? | No — Spring doesn't manage their full lifecycle after creation |
| Can you inject a prototype bean into a singleton safely? | Not by default (only one instance ever injected); use `@Lookup` or `ObjectProvider<T>` |
------