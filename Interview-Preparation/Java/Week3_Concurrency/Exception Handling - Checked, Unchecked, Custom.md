# Exception Handling — Checked, Unchecked, Custom

## 1. What is an Exception?

An exception is an event that disrupts the normal flow of program execution. Java handles exceptions using objects that represent the error/condition.

```
Throwable (root class)
├── Error                    (serious, JVM-level problems — don't catch these)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception                (problems your program CAN handle)
    ├── Checked Exceptions     (must handle or declare)
    │   ├── IOException
    │   ├── SQLException
    │   └── ClassNotFoundException
    └── Unchecked Exceptions   (RuntimeException and its subclasses)
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── ArithmeticException
        ├── ClassCastException
        └── IllegalArgumentException
```

---

## 2. Checked vs Unchecked Exceptions — The Core Distinction

### Checked Exceptions

```
- Subclasses of Exception (but NOT RuntimeException)
- Checked by the COMPILER at compile time
- Must be either:
  1. Caught using try-catch, OR
  2. Declared using 'throws' in method signature
- Represent conditions a well-written program
  SHOULD anticipate and recover from
  (e.g. file not found, network failure)
```

```java
import java.io.*;

public class Test {
    public static void main(String[] args) {
        readFile();   // COMPILE ERROR if not handled!
    }

    // Must declare 'throws IOException' or use try-catch
    static void readFile() throws IOException {
        FileReader fr = new FileReader("data.txt");
        // if file doesn't exist, throws FileNotFoundException
        // (which IS-A IOException, which IS checked)
    }
}
```

```java
// Alternative — handle with try-catch instead of declaring
public class Test {
    public static void main(String[] args) {
        try {
            FileReader fr = new FileReader("data.txt");
        } catch (IOException e) {
            System.out.println("File not found: " + e.getMessage());
        }
    }
}
```

### Unchecked Exceptions

```
- Subclasses of RuntimeException
- NOT checked by the compiler
- Program compiles fine even without handling them
- Usually represent PROGRAMMING BUGS/mistakes
  (e.g. null reference, invalid array index, bad cast)
- You CAN catch them, but you are not FORCED to
```

```java
public class Test {
    public static void main(String[] args) {
        int[] arr = new int[5];
        System.out.println(arr[10]);
        // ArrayIndexOutOfBoundsException — UNCHECKED
        // Compiles fine! Crashes only at RUNTIME
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        String s = null;
        System.out.println(s.length());
        // NullPointerException — UNCHECKED
        // Compiles fine! Crashes only at RUNTIME
    }
}
```

---

## 3. Checked vs Unchecked — Comparison Table

| Feature | Checked Exception | Unchecked Exception |
|---|---|---|
| Parent class | `Exception` (not `RuntimeException`) | `RuntimeException` |
| Compiler enforcement | MUST catch or declare (`throws`) | No enforcement |
| When detected | Compile time (forced handling) | Runtime only |
| Represents | Recoverable external conditions (I/O, DB, network) | Programming bugs/logic errors |
| Examples | `IOException`, `SQLException`, `ClassNotFoundException` | `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException` |
| Typical fix | Handle gracefully (retry, fallback, user message) | Fix the BUG in code, don't just catch it |

---

## 4. try-catch-finally — The Basics

```java
public class Test {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;   // throws ArithmeticException
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        } finally {
            System.out.println("This ALWAYS runs!");
        }
    }
}

// Output:
// Caught: / by zero
// This ALWAYS runs!
```

### Multiple catch Blocks

```java
public class Test {
    public static void main(String[] args) {
        try {
            String s = null;
            System.out.println(s.length());
        } catch (NullPointerException e) {
            System.out.println("Null pointer issue");
        } catch (Exception e) {
            System.out.println("Some other exception");
        }
        // Order matters! More SPECIFIC exceptions must
        // come BEFORE more GENERAL ones (Exception is general)
        // Otherwise: compile error - unreachable catch block
    }
}
```

### Multi-catch (Java 7+)

```java
try {
    // some risky code
} catch (IOException | SQLException e) {
    // handles EITHER exception type with same logic
    System.out.println("I/O or DB error: " + e.getMessage());
}
```

### finally — When Does It NOT Run?

```
finally block ALWAYS runs EXCEPT when:
1. JVM crashes (System.exit() called inside try/catch)
2. The thread running the try block is killed
```

```java
public class Test {
    public static void main(String[] args) {
        try {
            System.out.println("In try");
            System.exit(0);   // JVM exits immediately!
        } finally {
            System.out.println("This will NOT print!");
        }
    }
}
// Output: "In try" only — finally is skipped!
```

---

## 5. try-with-resources (Java 7+) — Modern Cleanup

```
PROBLEM with manual finally cleanup:
Verbose, error-prone, easy to forget closing resources
```

```java
// OLD WAY — verbose
FileReader fr = null;
try {
    fr = new FileReader("data.txt");
    // use fr
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fr != null) {
        try {
            fr.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
// NEW WAY — try-with-resources
try (FileReader fr = new FileReader("data.txt")) {
    // use fr
} catch (IOException e) {
    e.printStackTrace();
}
// fr.close() called AUTOMATICALLY, even if exception occurs!
```

```
Requirement: The resource class must implement
AutoCloseable (or Closeable) interface
```

```java
class MyResource implements AutoCloseable {
    public void doWork() {
        System.out.println("Working...");
    }

    @Override
    public void close() {
        System.out.println("Resource closed automatically!");
    }
}

public class Test {
    public static void main(String[] args) {
        try (MyResource res = new MyResource()) {
            res.doWork();
        }
        // Output:
        // Working...
        // Resource closed automatically!
    }
}
```

### Multiple Resources

```java
try (FileReader fr = new FileReader("in.txt");
     FileWriter fw = new FileWriter("out.txt")) {
    // use both
}
// Closed in REVERSE order of declaration: fw first, then fr
```

---

## 6. throw vs throws — Don't Confuse These!

```java
// throw — used to ACTUALLY throw an exception (a statement)
public void validateAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("Must be 18+");
    }
}

// throws — used to DECLARE that a method MIGHT throw
// a checked exception (part of method signature)
public void readFile() throws IOException {
    // method body that might throw IOException
}
```

| Keyword | Purpose | Used Where |
|---|---|---|
| `throw` | Actually throws an exception instance | Inside method body, as a statement |
| `throws` | Declares possible checked exceptions | In method signature |

---

## 7. Custom Exceptions — Creating Your Own

### Custom Checked Exception

```java
class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message);   // pass message to Exception's constructor
    }
}

class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Insufficient balance: tried to withdraw " + amount +
                " but only " + balance + " available");
        }
        balance -= amount;
    }
}

public class Test {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000);
        try {
            account.withdraw(5000);
        } catch (InsufficientFundsException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

### Custom Unchecked Exception

```java
class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }
}

class OrderService {
    public void placeOrder(int quantity) {
        if (quantity <= 0) {
            throw new InvalidOrderException("Quantity must be positive");
        }
        System.out.println("Order placed for quantity: " + quantity);
    }
}

public class Test {
    public static void main(String[] args) {
        OrderService service = new OrderService();
        service.placeOrder(-5);
        // Compiles fine WITHOUT try-catch (unchecked)
        // but crashes at runtime if not handled
    }
}
```

### When to Make a Custom Exception Checked vs Unchecked

```
Make it CHECKED when:
- The caller can reasonably be expected to RECOVER
  (e.g. retry payment, ask user for different input)
- It represents an EXTERNAL condition (file, network, DB)

Make it UNCHECKED when:
- It represents a PROGRAMMING ERROR or invalid state
  that shouldn't normally happen if code is correct
- Forcing every caller to catch it would be excessive
  boilerplate for something that's usually a bug
  (most modern frameworks like Spring favor unchecked
  exceptions for this reason!)
```

---

## 8. Exception Chaining — Preserving Root Cause

```java
class ServiceException extends RuntimeException {
    public ServiceException(String message, Throwable cause) {
        super(message, cause);   // preserves original exception!
    }
}

public class Test {
    public static void main(String[] args) {
        try {
            connectToDatabase();
        } catch (SQLException e) {
            throw new ServiceException("Failed to process request", e);
            // original SQLException is preserved as the "cause"
            // visible in stack trace as "Caused by: ..."
        }
    }

    static void connectToDatabase() throws SQLException {
        throw new SQLException("Connection timeout");
    }
}
```

```
Why this matters: Without chaining, you'd LOSE the
original error details (e.g. the exact DB error)
when wrapping it in a more general/custom exception.
getCause() lets you trace back to the ROOT problem.
```

---

## 9. Common Interview Trap — Overriding Methods and Exceptions

```java
class Parent {
    void method() throws IOException {
        // ...
    }
}

class Child extends Parent {
    @Override
    void method() throws FileNotFoundException {
        // OK! FileNotFoundException IS-A IOException
        // (narrower checked exception allowed)
    }
}

class Child2 extends Parent {
    @Override
    void method() throws Exception {
        // COMPILE ERROR!
        // Cannot throw a BROADER checked exception
        // than the parent's declared exception
    }
}
```

```
RULE: An overriding method can throw the SAME,
a NARROWER, or NO checked exception —
but NEVER a BROADER checked exception
than the method it overrides.

(This rule does NOT apply to unchecked exceptions —
you can throw any unchecked exception freely,
since the compiler doesn't track them anyway.)
```

---

## 10. Best Practices

```
1. Don't catch generic Exception unless truly necessary
   - catching too broadly can hide real bugs

2. Don't swallow exceptions silently
   catch (Exception e) { }   // BAD! Error disappears silently

3. Always log or handle meaningfully:
   catch (Exception e) {
       log.error("Failed to process", e);
   }

4. Prefer unchecked exceptions for custom exceptions
   in modern applications (less boilerplate,
   used heavily in Spring Boot, etc.)

5. Use try-with-resources for anything implementing
   AutoCloseable (files, DB connections, streams)

6. Don't use exceptions for normal control flow
   (e.g. don't throw an exception just to break a loop)
```

---

## Interview Questions

**Q1. What is the difference between checked and unchecked exceptions?**
A: Checked exceptions (subclasses of Exception, excluding RuntimeException) are verified by the compiler — they must be caught or declared with `throws`. Unchecked exceptions (subclasses of RuntimeException) are not checked by the compiler and typically represent programming bugs.

**Q2. Why does Java force handling of checked exceptions but not unchecked ones?**
A: Checked exceptions represent recoverable external conditions (file I/O, network, database) that a well-written program should anticipate. Unchecked exceptions usually represent bugs (null references, bad array indices) that should be fixed in code rather than handled defensively everywhere.

**Q3. What is the difference between throw and throws?**
A: `throw` is a statement used to actually throw an exception instance at runtime. `throws` is used in a method signature to declare that the method might throw a particular checked exception.

**Q4. When does the finally block NOT execute?**
A: When System.exit() is called inside the try or catch block, or if the JVM crashes, or if the thread executing the try block is forcibly killed.

**Q5. What is try-with-resources and what interface must a resource implement?**
A: A Java 7+ feature that automatically closes resources after the try block, even if an exception occurs. The resource class must implement AutoCloseable (or its subinterface Closeable).

**Q6. Can an overriding method throw a broader checked exception than the parent method?**
A: No — an overriding method can throw the same, a narrower, or no checked exception compared to the method it overrides, but never a broader one. This rule does not apply to unchecked exceptions.

**Q7. What is exception chaining and why is it useful?**
A: Passing the original exception as the "cause" to a new exception (via a constructor like `super(message, cause)`). It preserves the root cause for debugging, visible as "Caused by:" in the stack trace, instead of losing the original error context.

**Q8. Should custom exceptions be checked or unchecked?**
A: Make it checked if callers can reasonably recover from it and it represents an external condition. Make it unchecked if it represents a programming error or invalid state — many modern frameworks (like Spring) favor unchecked custom exceptions to avoid excessive boilerplate.

**Q9. What is wrong with an empty catch block like `catch (Exception e) {}`?**
A: It silently swallows the exception, hiding bugs and making debugging very difficult since there's no log, message, or any trace that something went wrong.

**Q10. In what order should multiple catch blocks be written?**
A: From most specific exception type to most general. Placing a general exception type (like Exception) before a more specific one causes a compile-time "unreachable catch block" error.

**Q11. What is the parent class of all exceptions and errors in Java?**
A: `Throwable`, which has two main subclasses: `Error` (serious JVM-level problems, generally not meant to be caught) and `Exception` (problems a program can potentially handle).

**Q12. Is NullPointerException checked or unchecked?**
A: Unchecked — it extends RuntimeException. It typically indicates a programming bug (dereferencing a null reference) rather than an external recoverable condition.

---

## Summary

| Concept | One Line |
|---|---|
| Checked Exception | Compiler-enforced, must catch or declare, represents recoverable external conditions |
| Unchecked Exception | RuntimeException subclass, not compiler-enforced, usually a programming bug |
| throw | Statement that actually throws an exception |
| throws | Declares possible checked exceptions in method signature |
| try-with-resources | Auto-closes AutoCloseable resources, even on exception |
| finally | Runs always, except on System.exit() or JVM/thread kill |
| Exception chaining | Preserves original cause via `super(message, cause)` |
| Custom exception | Extend Exception (checked) or RuntimeException (unchecked) based on recoverability |
| Override rule | Cannot throw broader checked exception than overridden method |

------------------------