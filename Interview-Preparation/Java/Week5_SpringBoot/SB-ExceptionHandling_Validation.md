# Exception Handling and Validation in Spring Boot

## 1. Exception Handling

### The Problem
Without centralized exception handling, every controller method needs its own try-catch blocks, leading to duplicated error-handling logic scattered across the codebase, and inconsistent error responses sent to API clients.

```java
// Without centralized handling — repetitive and inconsistent
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    try {
        return userService.findById(id);
    } catch (UserNotFoundException e) {
        // custom error response logic repeated in every method
    }
}
```

### `@ExceptionHandler` — Handling Exceptions at Method/Controller Level

Handles exceptions thrown within a **specific controller**:

```java
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id); // throws UserNotFoundException if not found
    }

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```
**Limitation**: This only handles exceptions thrown within *this specific controller class*. If you have 10 controllers, you'd need to duplicate this handler in each one.

### `@ControllerAdvice` / `@RestControllerAdvice` — Global Exception Handling

Solves the duplication problem — define exception handling logic **once**, and it applies **across all controllers** in the application.

```java
@RestControllerAdvice   // = @ControllerAdvice + @ResponseBody (like RestController vs Controller)
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(Exception.class)   // catch-all fallback
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

`@RestControllerAdvice` is a class-level annotation that Spring detects as a **global interceptor for exceptions** thrown from any `@RequestMapping`-annotated method across the app. Internally, it works via `ExceptionHandlerExceptionResolver`, part of Spring MVC's exception resolution chain — when a controller method throws an exception, Spring checks if any `@ExceptionHandler` method (local first, then global `@ControllerAdvice`) matches that exception type before letting it propagate further.

### Custom Exceptions

You typically define your own exception classes for domain-specific errors:

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

**Checked vs unchecked**: Custom exceptions almost always extend `RuntimeException` (unchecked) rather than `Exception` (checked) — this avoids forcing every calling method up the stack to declare `throws` or wrap calls in try-catch, keeping service/repository code clean. Spring's own exception hierarchy (e.g., `DataAccessException`) is also unchecked for this same reason.

### Building a Consistent Error Response Structure

Good practice — a standard shape for all error responses so API clients can parse them predictably:

```java
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;

    // constructor, getters, setters
}
```

### Handling Multiple Exception Types in One Handler

```java
@ExceptionHandler({UserNotFoundException.class, OrderNotFoundException.class})
public ResponseEntity<ErrorResponse> handleNotFoundExceptions(RuntimeException ex) {
    // shared handling logic
}
```

### `ResponseEntityExceptionHandler`

Spring provides this base class with pre-built handlers for common Spring MVC exceptions (e.g., `HttpMessageNotReadableException`, `MethodArgumentNotValidException`). You can extend it in your `@ControllerAdvice` class and override specific methods instead of writing every handler from scratch:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    // override handleMethodArgumentNotValid(), etc. as needed
}
```

---

## 2. Validation

### The Problem
Without validation, invalid data (empty names, malformed emails, negative ages) can flow straight into your business logic or database, causing bugs or corrupt data. Manually checking each field in every controller method is repetitive and error-prone.

### Bean Validation (JSR 380) via `spring-boot-starter-validation`

Spring Boot integrates the **Bean Validation** specification (implemented by Hibernate Validator by default) — you annotate fields on your DTO/entity with constraints, and Spring validates incoming data automatically.

```java
public class UserRequest {

    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Email should be valid")
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    private int age;

    // getters, setters
}
```

### Common Validation Annotations

| Annotation | Checks |
|---|---|
| `@NotNull` | Value must not be `null` |
| `@NotBlank` | String must not be `null` and must contain at least one non-whitespace character |
| `@NotEmpty` | Collection/String/Array must not be `null` or empty (whitespace-only string still passes) |
| `@Size(min=, max=)` | String/Collection length within bounds |
| `@Min` / `@Max` | Numeric value within bounds |
| `@Email` | Valid email format |
| `@Pattern(regexp=)` | Matches a given regex |
| `@Positive` / `@Negative` | Numeric sign constraints |
| `@Past` / `@Future` | Date must be in the past/future |

> **`@NotNull` vs `@NotBlank` vs `@NotEmpty`** — a common mix-up:
> - `@NotNull` → just not null (empty string `""` passes)
> - `@NotEmpty` → not null AND not empty (whitespace-only string `"   "` passes)
> - `@NotBlank` → not null AND not empty AND not just whitespace (strictest, most commonly used for strings)

### Triggering Validation — `@Valid`

Validation doesn't run automatically — you trigger it in the controller using `@Valid` (or `@Validated` for group validation, covered below):

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody UserRequest request) {
    User user = userService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}
```

**How it works internally**: `@Valid` triggers Spring MVC's `ArgumentResolver` to validate the object against its constraint annotations *before* the method body executes. If validation fails, Spring throws `MethodArgumentNotValidException` — it never even enters your controller method body.

### Handling Validation Errors

Without a handler, `MethodArgumentNotValidException` produces a generic 400 response. To customize it, catch it in your `@RestControllerAdvice`:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return ResponseEntity.badRequest().body(errors);
    }
}
```
This extracts each field's error message (the `message` attribute you set in annotations like `@NotBlank(message = "...")`) into a clean map, e.g., `{"name": "Name is required", "age": "Age must be at least 18"}`.

### `@Valid` vs `@Validated`

| | `@Valid` | `@Validated` |
|---|---|---|
| Source | Standard Java (`jakarta.validation`) | Spring-specific (`org.springframework.validation.annotation`) |
| Supports validation groups | ❌ | ✅ |
| Supports method-level validation on service classes | ❌ | ✅ (combined with `@Validated` on the class) |
| Common usage | Validating `@RequestBody` in controllers | Validating path/query params, or group-based validation |

**Validation groups** (via `@Validated`) let you apply different validation rules for different scenarios using the same DTO — e.g., `id` should be null on create but required on update:
```java
public class UserRequest {
    @Null(groups = OnCreate.class)
    @NotNull(groups = OnUpdate.class)
    private Long id;
}

@PostMapping
public User create(@Validated(OnCreate.class) @RequestBody UserRequest request) { ... }

@PutMapping
public User update(@Validated(OnUpdate.class) @RequestBody UserRequest request) { ... }
```

### Custom Validators

For validation logic beyond built-in annotations (e.g., checking a username isn't already taken), create a custom constraint:

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueUsernameValidator.class)
public @interface UniqueUsername {
    String message() default "Username already taken";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {
    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        return !userRepository.existsByUsername(username);
    }
}
```

### Validating Nested Objects

If a DTO contains another object that also needs validation, annotate the nested field with `@Valid`:

```java
public class OrderRequest {
    @Valid
    private AddressRequest shippingAddress;
}
```
Without `@Valid` on the nested field, constraints inside `AddressRequest` are silently skipped.

---

## Interview Quick-Reference

| Question | Answer |
|---|---|
| Difference between `@ExceptionHandler` and `@ControllerAdvice`? | `@ExceptionHandler` handles exceptions within one controller; `@ControllerAdvice`/`@RestControllerAdvice` applies globally across all controllers |
| What does `@RestControllerAdvice` equal? | `@ControllerAdvice` + `@ResponseBody` |
| Why do custom exceptions usually extend `RuntimeException`? | Avoids forcing `throws` declarations/try-catch up the call stack; keeps service code clean |
| Difference between `@NotNull`, `@NotEmpty`, `@NotBlank`? | `@NotNull`: not null. `@NotEmpty`: not null/empty. `@NotBlank`: not null/empty/whitespace-only (strictest) |
| What triggers Bean Validation on a request body? | `@Valid` (or `@Validated`) on the `@RequestBody` parameter |
| What exception is thrown on validation failure? | `MethodArgumentNotValidException` |
| Difference between `@Valid` and `@Validated`? | `@Valid` is standard Java, no group support. `@Validated` is Spring-specific, supports validation groups and method-level validation |
| How do you validate a nested object inside a DTO? | Annotate the nested field itself with `@Valid` |
| How do you build a custom validation constraint? | Create an annotation with `@Constraint(validatedBy = ...)` + implement `ConstraintValidator<Annotation, Type>` |


-----------------