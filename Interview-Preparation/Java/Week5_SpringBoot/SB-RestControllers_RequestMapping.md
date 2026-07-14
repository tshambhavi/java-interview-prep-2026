# Spring Boot — REST Controllers & Request Mapping

## 1. What is a REST Controller?

`@RestController` is a specialized version of `@Controller` used to build RESTful web services. It's a **meta-annotation**:

```java
@RestController = @Controller + @ResponseBody
```

- `@Controller` alone → return values are resolved as **view names** (for server-rendered pages, e.g. Thymeleaf)
- `@RestController` → return values are **serialized directly into the HTTP response body** (typically as JSON via Jackson), skipping view resolution entirely

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id); // serialized to JSON automatically
    }
}
```

**Q: Difference between `@Controller` and `@RestController`?**
A: `@RestController` = `@Controller` + `@ResponseBody` applied to every handler method. Without it, you'd need `@ResponseBody` on each method individually to return raw data instead of a view name.

## 2. `@RequestMapping` — the base mapping annotation

Maps HTTP requests to handler methods/classes. Can specify path, HTTP method, headers, params, produces/consumes.

```java
@RequestMapping(value = "/api/users", method = RequestMethod.GET)
public List<User> getAllUsers() { ... }
```

Class-level `@RequestMapping` sets a common base path for all methods in that controller:

```java
@RestController
@RequestMapping("/api/users")   // base path
public class UserController {

    @RequestMapping(method = RequestMethod.GET)         // GET /api/users
    public List<User> getAll() { ... }

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)  // GET /api/users/{id}
    public User getOne(@PathVariable Long id) { ... }
}
```

## 3. HTTP Method Shortcut Annotations

These are specialized versions of `@RequestMapping` — preferred in practice since Spring 4.3, more concise and readable.

| Annotation | Equivalent to | HTTP Method |
|---|---|---|
| `@GetMapping` | `@RequestMapping(method = RequestMethod.GET)` | GET — retrieve resource |
| `@PostMapping` | `@RequestMapping(method = RequestMethod.POST)` | POST — create resource |
| `@PutMapping` | `@RequestMapping(method = RequestMethod.PUT)` | PUT — full update/replace resource |
| `@PatchMapping` | `@RequestMapping(method = RequestMethod.PATCH)` | PATCH — partial update |
| `@DeleteMapping` | `@RequestMapping(method = RequestMethod.DELETE)` | DELETE — remove resource |

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public List<User> getAll() { ... }

    @GetMapping("/{id}")
    public User getById(@PathVariable Long id) { ... }

    @PostMapping
    public User create(@RequestBody User user) { ... }

    @PutMapping("/{id}")
    public User update(@PathVariable Long id, @RequestBody User user) { ... }

    @PatchMapping("/{id}")
    public User partialUpdate(@PathVariable Long id, @RequestBody Map<String, Object> updates) { ... }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) { ... }
}
```

**Q: PUT vs PATCH?**
A: PUT replaces the entire resource (client sends the full object); PATCH applies a partial update (only the changed fields).

## 4. Extracting Data from Requests

### `@PathVariable` — from the URL path

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) { ... }        // /api/users/5

@GetMapping("/{userId}/orders/{orderId}")
public Order getOrder(@PathVariable Long userId, @PathVariable Long orderId) { ... }
```

Use when the value **identifies a specific resource** (e.g. `/users/5`).

### `@RequestParam` — from the query string

```java
@GetMapping
public List<User> search(@RequestParam String name,
                          @RequestParam(required = false) Integer age,
                          @RequestParam(defaultValue = "0") int page) { ... }
// /api/users?name=John&age=25&page=1
```

Use for **optional filters, pagination, sorting** — not for identifying a specific resource.

**Q: `@PathVariable` vs `@RequestParam`?**
A: `@PathVariable` extracts values embedded in the URL path (identifying a resource); `@RequestParam` extracts values from the query string (typically optional filters/params).

### `@RequestBody` — from the HTTP request body

```java
@PostMapping
public User create(@RequestBody User user) { ... }
```

Deserializes incoming JSON into a Java object using Jackson (`HttpMessageConverter` under the hood). Requires `Content-Type: application/json` header from client.

### `@RequestHeader` — from HTTP headers

```java
@GetMapping
public List<User> getAll(@RequestHeader("Authorization") String token) { ... }
```

## 5. Response Handling

### `ResponseEntity<T>` — full control over response

Instead of returning a raw object, wrap it to control status code, headers, and body explicitly:

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    if (user == null) {
        return ResponseEntity.notFound().build();               // 404
    }
    return ResponseEntity.ok(user);                              // 200 + body
}

@PostMapping
public ResponseEntity<User> create(@RequestBody User user) {
    User saved = userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved); // 201
}
```

**Q: Why use `ResponseEntity` instead of just returning the object?**
A: Gives explicit control over HTTP status code, headers, and body — important for REST correctness (e.g. 201 Created on POST, 404 on not found, 204 No Content on DELETE), rather than always defaulting to 200.

### `produces` / `consumes`

```java
@GetMapping(value = "/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
```

## 6. Exception Handling in REST Controllers

### `@ExceptionHandler` — per-controller

```java
@RestController
public class UserController {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

### `@ControllerAdvice` / `@RestControllerAdvice` — global, across all controllers

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ErrorResponse("Something went wrong"));
    }
}
```

**Q: `@ExceptionHandler` vs `@ControllerAdvice`?**
A: `@ExceptionHandler` on a controller handles exceptions only within that controller. `@ControllerAdvice`/`@RestControllerAdvice` is a global, cross-cutting handler applied to all controllers — avoids duplicating error-handling logic everywhere.

## 7. Validation

```java
@PostMapping
public ResponseEntity<User> create(@Valid @RequestBody User user) {
    // throws MethodArgumentNotValidException if validation fails
    // caught centrally via @ExceptionHandler(MethodArgumentNotValidException.class)
}

public class User {
    @NotBlank
    private String name;

    @Email
    private String email;

    @Min(18)
    private int age;
}
```

Requires `spring-boot-starter-validation` dependency. `@Valid` triggers bean validation (JSR-380/Hibernate Validator) on the incoming request body.

## 8. Common REST Design Conventions (Interview Talking Points)

| Action | HTTP Method | URL | Status Code |
|---|---|---|---|
| Get all | GET | `/api/users` | 200 |
| Get one | GET | `/api/users/{id}` | 200 / 404 |
| Create | POST | `/api/users` | 201 |
| Full update | PUT | `/api/users/{id}` | 200 |
| Partial update | PATCH | `/api/users/{id}` | 200 |
| Delete | DELETE | `/api/users/{id}` | 204 |

- URLs should be **nouns**, not verbs (`/users`, not `/getUsers`)
- Use plural resource names
- Nesting for relationships: `/users/{id}/orders`

## 9. Likely Interview Questions

1. Difference between `@Controller` and `@RestController`?
2. Difference between `@PathVariable` and `@RequestParam`?
3. Difference between PUT and PATCH?
4. Why return `ResponseEntity` instead of the raw object?
5. How do you handle exceptions globally across all REST controllers?
6. How does `@RequestBody` deserialize JSON into a Java object under the hood? (`HttpMessageConverter`, Jackson)
7. How would you validate incoming request data? (`@Valid`, Bean Validation annotations)
8. What status code should a DELETE endpoint return, and why? (204 No Content — successful, nothing to return)

-------------