# Spring Boot — Auto-Configuration & Annotations

## 1. What is Auto-Configuration?

Spring Boot's core philosophy is **"convention over configuration."** Traditional Spring required manually declaring beans via XML or `@Bean` methods for everything — a `DataSource`, a `DispatcherServlet`, a JPA `EntityManagerFactory`, etc.

Auto-configuration flips this: **Spring Boot inspects what's on the classpath and what beans already exist in the context, and automatically configures sensible default beans** — without writing configuration code.

**Example:** Adding `spring-boot-starter-data-jpa` + an H2 dependency automatically gives you:
- A `DataSource` bean pointing at an in-memory H2 database
- An `EntityManagerFactory`
- A `PlatformTransactionManager`
- Spring Data JPA repository scanning enabled

## 2. How Auto-Configuration Works (Step by Step)

### Step 1: `@SpringBootApplication` is a meta-annotation

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

It bundles three annotations:

| Annotation | What it does |
|---|---|
| `@SpringBootConfiguration` | Marks class as a bean definition source (internally `@Configuration`) |
| `@ComponentScan` | Scans current package + sub-packages for `@Component`/`@Service`/`@Repository`/`@Controller` |
| `@EnableAutoConfiguration` | Triggers auto-configuration (the important one) |

### Step 2: `@EnableAutoConfiguration` loads candidate configuration classes

Every auto-config-capable jar bundles a file listing its configuration classes:
- **Spring Boot 2.x:** `META-INF/spring.factories`
- **Spring Boot 3.x (current):** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

Example contents:
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration

### Step 3: Each candidate is gated by conditional annotations

```java
@Configuration
@ConditionalOnClass(DataSource.class)           // only if DataSource.class is on classpath
@ConditionalOnMissingBean(DataSource.class)      // only if no DataSource bean defined already
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        // builds and returns a DataSource using application.properties values
    }
}
```

> **classpath contents + existing user-defined beans → conditions evaluated → matching auto-configs applied → beans registered**

### Step 4: Conditions are evaluated at startup

Two buckets result:
- **Positive matches** — conditions satisfied, auto-config applied
- **Negative matches** — conditions failed, skipped (with logged reason)

View this by running with `--debug`, or via `/actuator/conditions` if Actuator is on the classpath.

## 3. Key `@Conditional...` Annotations

| Annotation | Applies when... |
|---|---|
| `@ConditionalOnClass` | A specific class is present on the classpath |
| `@ConditionalOnMissingClass` | A specific class is **absent** |
| `@ConditionalOnBean` | A specific bean already exists |
| `@ConditionalOnMissingBean` | No bean of that type exists yet — **overriding mechanism** |
| `@ConditionalOnProperty` | A given property has a specific value (or is present) |
| `@ConditionalOnWebApplication` | App is a web application |
| `@ConditionalOnNotWebApplication` | App is not a web application |
| `@ConditionalOnResource` | A specific resource file exists on classpath |

### `@ConditionalOnMissingBean` — overriding auto-configured beans

```java
@Configuration
public class MyConfig {
    @Bean
    public DataSource dataSource() {
        // your custom DataSource — Spring Boot's default auto-config backs off
        return new HikariDataSource(myCustomConfig);
    }
}
```

**Q: How do you override an auto-configured bean?**
A: Define your own bean of that type; `@ConditionalOnMissingBean` on the auto-config class causes it to step aside.

## 4. Excluding an Auto-Configuration Explicitly

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MyApp { }
```

or in `application.properties`:
```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

**Exclude vs override:** exclude = never even considered; override via `@ConditionalOnMissingBean` = considered, but backs off because your bean exists.

## 5. Annotations by Category

### A. Stereotype annotations (component scanning)

| Annotation | Purpose |
|---|---|
| `@Component` | Generic "let Spring manage this as a bean" |
| `@Service` | Marks business/service layer. Functionally identical to `@Component`. |
| `@Repository` | Marks persistence layer. Same as `@Component`, **plus** wraps exceptions into `DataAccessException` |
| `@Controller` | Web MVC controller; return values resolved as view names |
| `@RestController` | `@Controller` + `@ResponseBody` — return values serialized directly to response body |

**Q: Real difference between `@Component`, `@Service`, `@Repository`?**
A: No functional difference between `@Component`/`@Service` — semantic only. `@Repository` adds real behavior (exception translation).

### B. Dependency Injection annotations

| Annotation | Purpose |
|---|---|
| `@Autowired` | Injects a dependency (field/constructor/setter) |
| `@Qualifier` | Disambiguates when multiple beans of same type exist |
| `@Value` | Injects a single property value |
| `@Primary` | Marks default bean choice among multiple candidates |

**Why constructor injection > field injection:**
1. Allows `final` fields → immutability, thread-safety
2. Easier unit testing — construct directly with mocks, no Spring context needed
3. Dependencies visible explicitly in constructor signature
4. Fails fast — app won't start if a required dependency is missing
5. Exposes circular dependencies immediately at startup (`BeanCurrentlyInCreationException`)

```java
@Service
public class OrderService {
    private final PaymentClient paymentClient; // final possible only with constructor injection

    public OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }
}
```

### C. Configuration annotations

| Annotation | Purpose |
|---|---|
| `@Configuration` | Marks class as source of `@Bean` definitions |
| `@Bean` | Method-level; registers return value as a bean |
| `@ConfigurationProperties(prefix = "...")` | Binds a block of `application.yml` into a typed POJO |

```yaml
app:
  mail:
    host: smtp.example.com
    port: 587
```
```java
@Component
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port;
    // getters/setters
}
```

### D. Web layer annotations

| Annotation | Purpose |
|---|---|
| `@RequestMapping` | Generic URL/method mapping |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` / `@PatchMapping` | Shortcuts for `@RequestMapping(method = ...)` |
| `@PathVariable` | Binds a value from the URL path (e.g. `/users/{id}`) |
| `@RequestParam` | Binds a value from the query string (e.g. `?name=John`) |
| `@RequestBody` | Deserializes request body (JSON) into a Java object |
| `@ResponseBody` | Serializes return value into response body (implicit in `@RestController`) |

### E. Bean lifecycle & scope

| Annotation | Purpose |
|---|---|
| `@Scope("singleton")` | Default — one shared instance per context |
| `@Scope("prototype")` | New instance every time bean is requested |
| `@PostConstruct` | Runs after dependency injection — init logic |
| `@PreDestroy` | Runs before bean destruction — cleanup logic |

## 6. Likely Interview Questions

1. Walk me through what happens when a Spring Boot app starts, from `main()` to a running embedded server.
2. How does Spring Boot decide which auto-configurations to apply?
3. How would you override a Spring Boot auto-configured bean? How is that different from excluding it?
4. Why is constructor injection preferred over field injection?
5. Real difference between `@Component`, `@Service`, and `@Repository`?
6. Difference between `@RestController` and `@Controller`?
7. How would you debug why an auto-configuration isn't applying as expected?

---------------