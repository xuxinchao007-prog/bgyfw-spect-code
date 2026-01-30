---
name: java-patterns
description: Idiomatic Java patterns, best practices, and conventions for building robust, efficient, and maintainable Java applications following modern Java standards (Java 17+).
---

# Java Development Patterns

Idiomatic Java patterns and best practices for building robust, efficient, and maintainable applications with modern Java (Java 17+).

## When to Activate

- Writing new Java code
- Reviewing Java code
- Refactoring existing Java code
- Designing Java packages/modules
- Migrating legacy Java to modern versions

## Core Principles

### 1. Explicit is Better Than Implicit

Java favors clarity and explicitness over magic and convention.

```java
// Good: Clear and explicit
public class UserService {
    private final UserRepository repository;
    private final PasswordEncoder encoder;

    public UserService(UserRepository repository, PasswordEncoder encoder) {
        this.repository = Objects.requireNonNull(repository, "repository must not be null");
        this.encoder = Objects.requireNonNull(encoder, "encoder must not be null");
    }
}

// Bad: Hiding dependencies through magic
public class UserService {
    // Dependencies injected through reflection or magic
    @Autowired private UserRepository repository;
}
```

### 2. Immutability as Default

Design for immutability to enable thread-safety and easier reasoning.

```java
// Good: Immutable value object
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public BigDecimal amount() { return amount; }
    public Currency currency() { return currency; }

    // Returns new instance instead of mutating
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currency mismatch");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money)) return false;
        Money money = (Money) o;
        return Objects.equals(amount, money.amount) &&
               Objects.equals(currency, money.currency);
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
}

// Bad: Mutable state
public class Money {
    private BigDecimal amount;
    public void setAmount(BigDecimal amount) { this.amount = amount; }
}
```

### 3. Program to Interfaces

Accept interfaces as parameters, return concrete implementations.

```java
// Good: Accept interface, return concrete type
public class ReportGenerator {
    public List<String> generate(DataProvider provider) {
        // Implementation
        return provider.fetchData().stream()
            .map(this::formatLine)
            .toList();
    }
}

// Bad: Tight coupling to concrete implementation
public class ReportGenerator {
    public List<String> generate(SqlDataProvider provider) {
        // Tightly coupled to SQL
    }
}
```

## Exception Handling Patterns

### Prefer Specific Exceptions

```java
// Good: Specific exception types
public class UserRepository {
    public User findById(String id) throws UserNotFoundException {
        return repository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }
}

// Bad: Generic exceptions
public class UserRepository {
    public User findById(String id) throws Exception {
        // Vague exception handling
    }
}
```

### Exception Translation

```java
// Good: Translate low-level exceptions to domain-specific ones
public class UserService {
    public User createUser(String email) {
        try {
            return repository.save(new User(email));
        } catch (DataIntegrityViolationException e) {
            throw new DuplicateUserException("User with email " + email + " already exists", e);
        }
    }
}

// Domain exception
public class DuplicateUserException extends RuntimeException {
    public DuplicateUserException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Try-with-Resources

```java
// Good: Automatic resource management
public class FileProcessor {
    public void processFile(Path path) throws IOException {
        try (var reader = Files.newBufferedReader(path);
             var writer = Files.newBufferedWriter(outputPath)) {
            // Resources automatically closed
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(processLine(line));
            }
        }
    }
}

// Bad: Manual resource management
public void processFile(Path path) throws IOException {
    var reader = Files.newBufferedReader(path);
    try {
        // Process
    } finally {
        reader.close(); // Can fail or be forgotten
    }
}
```

### Exception Chaining

```java
// Good: Preserve cause chain
public class PaymentService {
    public void processPayment(Payment payment) throws PaymentException {
        try {
            gateway.charge(payment);
        } catch (GatewayException e) {
            throw new PaymentException("Payment processing failed", e);
            // Original exception preserved as cause
        }
    }
}
```

## Concurrency Patterns

### Virtual Threads (Java 21+)

```java
// Good: Virtual threads for I/O-bound tasks
public class WebServer {
    private final ServerSocket serverSocket;

    public void start() throws IOException {
        while (true) {
            var socket = serverSocket.accept();
            // Each connection gets its own virtual thread
            Thread.ofVirtual().start(() -> handleConnection(socket));
        }
    }

    private void handleConnection(Socket socket) {
        try (socket) {
            // Handle request - blocking is fine with virtual threads
            var request = readRequest(socket);
            var response = processRequest(request);
            writeResponse(socket, response);
        } catch (IOException e) {
            logger.error("Connection error", e);
        }
    }
}
```

### Structured Concurrency (Preview in Java 21)

```java
// Good: Structured concurrency for related tasks
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.Future;

public class AggregationService {
    public AggregatedData fetchAll(String id) throws InterruptedException {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            StructuredTaskScope.Subtask<User> userTask =
                scope.fork(() -> userService.fetchUser(id));
            StructuredTaskScope.Subtask<Orders> ordersTask =
                scope.fork(() -> orderService.fetchOrders(id));
            StructuredTaskScope.Subtask<Preferences> prefsTask =
                scope.fork(() -> prefsService.fetchPreferences(id));

            // Wait for all tasks or first failure
            scope.join();

            // If any task failed, throw its exception
            scope.throwIfFailed();

            return new AggregatedData(
                userTask.get(),
                ordersTask.get(),
                prefsTask.get()
            );
        }
    }
}
```

### Synchronized vs Lock

```java
// Good: Use synchronized for simple cases
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int get() {
        return count;
    }
}

// Good: Use ReentrantLock for advanced features
public class TimeoutLock {
    private final ReentrantLock lock = new ReentrantLock();

    public void tryUpdate(long timeoutMillis) throws InterruptedException {
        if (lock.tryLock(timeoutMillis, TimeUnit.MILLISECONDS)) {
            try {
                // Critical section
            } finally {
                lock.unlock();
            }
        }
    }
}
```

### Concurrent Collections

```java
// Good: Use appropriate concurrent collection
import java.util.concurrent.*;

public class CacheManager {
    // For high concurrency with reads > writes
    private final ConcurrentMap<String, Data> cache = new ConcurrentHashMap<>();

    // For producer-consumer scenarios
    private final BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(1000);

    // For copy-on-write semantics (rare writes)
    private final CopyOnWriteArrayList<Listener> listeners = new CopyOnWriteArrayList<>();
}
```

### Thread Pools

```java
// Good: Use proper thread pools
import java.util.concurrent.Executors;

public class AsyncTaskExecutor {
    // For CPU-bound tasks
    private final ExecutorService cpuPool = Executors.newFixedThreadPool(
        Runtime.getRuntime().availableProcessors()
    );

    // For I/O-bound tasks with virtual threads
    private final ExecutorService ioPool = Executors.newVirtualThreadPerTaskExecutor();

    // For scheduled tasks
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

    public void submitCpuTask(Runnable task) {
        cpuPool.submit(task);
    }

    public void submitIoTask(Runnable task) {
        ioPool.submit(task);
    }

    public void schedule(Runnable task, long delay, TimeUnit unit) {
        scheduler.schedule(task, delay, unit);
    }

    public void shutdown() {
        cpuPool.shutdown();
        ioPool.shutdown();
        scheduler.shutdown();
    }
}
```

## Interface Design

### Interface Segregation

```java
// Good: Small, focused interfaces
public interface Reader {
    String read() throws IOException;
    boolean hasNext() throws IOException;
}

public interface Writer {
    void write(String data) throws IOException;
    void flush() throws IOException;
}

public interface Closeable {
    void close() throws IOException;
}

// Compose interfaces as needed
public interface ReadWriter extends Reader, Writer {}

public interface ReadWriteCloseable extends ReadWriter, Closeable {}
```

### Functional Interfaces

```java
// Good: Define custom functional interfaces for clarity
@FunctionalInterface
public interface ValidationErrorHandler {
    void handle(String field, String message);

    default ValidationErrorHandler andThen(ValidationErrorHandler other) {
        return (field, message) -> {
            this.handle(field, message);
            other.handle(field, message);
        };
    }
}

// Usage
public class Validator {
    public void validate(User user, ValidationErrorHandler handler) {
        if (user.email() == null) {
            handler.handle("email", "is required");
        }
        if (user.age() < 18) {
            handler.handle("age", "must be 18 or older");
        }
    }
}

validator.validate(user,
    ValidationErrorHandler.logging()
    .andThen(ValidationErrorHandler.collecting(errors))
);
```

### Default Methods

```java
// Good: Provide default implementations in interfaces
public interface Repository<T, ID> {
    Optional<T> findById(ID id);

    // Default method for common functionality
    default T findByIdOrElseThrow(ID id) {
        return findById(id)
            .orElseThrow(() -> new NotFoundException("Entity not found: " + id));
    }

    // Default method with customization point
    default void deleteAll(Iterable<ID> ids) {
        ids.forEach(this::delete);
    }
}
```

## Package Organization

### Standard Java Package Structure

```
com.example.project/
├── api/                    # Public API (interfaces, DTOs)
│   ├── request/
│   ├── response/
│   └── exception/
├── application/            # Application services (use cases)
│   ├── service/
│   └── dto/
├── domain/                 # Domain model (business logic)
│   ├── model/
│   ├── repository/
│   └── service/
├── infrastructure/         # External dependencies
│   ├── persistence/
│   ├── messaging/
│   └── client/
├── common/                 # Shared utilities
│   ├── util/
│   └── exception/
└── config/                 # Configuration
```

### Modular Code (Java 9+)

```
src/
├── main/
│   ├── java/
│   │   └── com.example.project/
│   │       ├── module-info.java      # Module descriptor
│   │       └── ...
│   └── resources/
│       └── application.yml
└── test/
    └── java/
        └── com.example.project/
```

```java
// module-info.java
module com.example.project {
    requires java.sql;
    requires java.logging;

    requires transitive com.example.commons;

    exports com.example.project.api;
    exports com.example.project.application;

    // Hidden implementation
    // com.example.project.infrastructure not exported
}
```

## Record Classes (Java 16+)

### Immutable Data Carriers

```java
// Good: Record for immutable data
public record User(String id, String name, String email) {
    // Compact constructor for validation
    public User {
        Objects.requireNonNull(name, "name must not be null");
        Objects.requireNonNull(email, "email must not be null");

        if (name.isBlank()) {
            throw new IllegalArgumentException("name must not be blank");
        }

        if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("invalid email format");
        }
    }

    // Custom methods
    public String displayName() {
        return name + " (" + email + ")";
    }

    // Static factory method
    public static User anonymous() {
        return new User("anonymous", "Anonymous", "anonymous@example.com");
    }
}

// Usage
var user = new User("123", "Alice", "alice@example.com");
System.out.println(user.displayName()); // Alice (alice@example.com)
System.out.println(user.name());        // Alice
```

### Record with Validation

```java
// Good: Record with validation logic
public record Email(String value) {
    public Email {
        Objects.requireNonNull(value);
        if (!isValid(value)) {
            throw new IllegalArgumentException("Invalid email: " + value);
        }
    }

    private static boolean isValid(String email) {
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }

    // Conversion method
    public String lowerCase() {
        return value.toLowerCase();
    }
}
```

### Sealed Classes (Java 17+)

```java
// Good: Sealed classes for restricted inheritance
public sealed interface Shape
    permits Circle, Rectangle, Triangle {

    double area();
    double perimeter();
}

public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }

    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}

public record Rectangle(double width, double height) implements Shape {
    @Override
    public double area() {
        return width * height;
    }

    @Override
    public double perimeter() {
        return 2 * (width + height);
    }
}

public final class Triangle implements Shape {
    private final double a, b, c;

    public Triangle(double a, double b, double c) {
        this.a = a;
        this.b = b;
        this.c = c;
    }

    @Override
    public double area() {
        // Heron's formula
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s - a) * (s - b) * (s - c));
    }

    @Override
    public double perimeter() {
        return a + b + c;
    }
}

// Exhaustive pattern matching (Java 21)
public double describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with area " + c.area();
        case Rectangle r -> "Rectangle with area " + r.area();
        case Triangle t -> "Triangle with area " + t.area();
        // No default needed - compiler verifies exhaustiveness
    };
}
```

## Memory and Performance

### Stream Best Practices

```java
// Good: Lazy streams with terminal operation
public List<String> getActiveUserNames(List<User> users) {
    return users.stream()
        .filter(User::isActive)
        .map(User::name)
        .toList();  // Java 16+ - unmodifiable list
}

// Bad: Multiple passes
public List<String> getActiveUserNamesBad(List<User> users) {
    var active = new ArrayList<User>();
    for (var user : users) {
        if (user.isActive()) active.add(user);
    }

    var names = new ArrayList<String>();
    for (var user : active) {
        names.add(user.name());
    }
    return names;
}
```

### Avoid Premature Optimization

```java
// Good: Clear and readable
public String formatName(User user) {
    return String.format("%s %s", user.firstName(), user.lastName());
}

// Bad: Optimized prematurely (hard to read)
public String formatNameBad(User user) {
    StringBuilder sb = new StringBuilder();
    sb.append(user.firstName());
    sb.append(' ');
    sb.append(user.lastName());
    return sb.toString();
}

// Only optimize after profiling shows it's needed
```

### String Concatenation

```java
// Good: Use + for simple concatenation
String message = "Hello, " + name + "!";

// Good: Use String.format for complex formatting
String formatted = String.format("User %s (ID: %d) has %d items",
    user.name(), user.id(), user.itemCount());

// Good: Use StringBuilder in loops
public String join(List<String> parts) {
    var sb = new StringBuilder();
    for (int i = 0; i < parts.size(); i++) {
        if (i > 0) sb.append(", ");
        sb.append(parts.get(i));
    }
    return sb.toString();
}

// Best: Use String.join()
public String join(List<String> parts) {
    return String.join(", ", parts);
}
```

### Collection Size Hints

```java
// Good: Provide size hint when known
public List<String> toUpperCase(List<String> input) {
    var result = new ArrayList<String>(input.size());
    for (var s : input) {
        result.add(s.toUpperCase());
    }
    return result;
}

// Good: Use Guava's precise sizing if available
import com.google.common.collect.Lists;

public List<String> transform(List<String> input) {
    return Lists.newArrayListWithCapacity(input.size());
}
```

## Optional Usage

### When to Use Optional

```java
// Good: Optional for return values that might be empty
public class UserRepository {
    public Optional<User> findByEmail(String email) {
        return repository.findByEmail(email);
    }
}

// Bad: Optional for fields (don't do this)
public class User {
    private Optional<String> middleName;  // WRONG
}

// Bad: Optional for parameters (don't do this)
public void sendEmail(Optional<String> address) {  // WRONG
    // Use overloading instead
}
```

### Optional Best Practices

```java
// Good: Provide default value
User user = repository.findById(id)
    .orElse(User.UNKNOWN);

// Good: Throw exception if empty
User user = repository.findById(id)
    .orElseThrow(() -> new NotFoundException("User not found"));

// Good: Conditional action
repository.findById(id)
    .ifPresent(user -> sendWelcomeEmail(user));

// Good: Transform if present
String displayName = repository.findById(id)
    .map(User::name)
    .orElse("Anonymous");

// Good: Filter and transform
Optional<String> upperEmail = repository.findById(id)
    .filter(User::isActive)
    .map(User::email)
    .map(String::toUpperCase);
```

## Null Handling

### Null Checks

```java
// Good: Explicit null checks
public void processUser(User user) {
    if (user == null) {
        throw new IllegalArgumentException("user must not be null");
    }
    // Process user
}

// Good: Use Objects.requireNonNull
public class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = Objects.requireNonNull(repository, "repository must not be null");
    }
}
```

### Null Annotations

```java
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

// Good: Use null annotations
public class UserService {
    public @NotNull User findUser(@NotNull String id) {
        return repository.findById(id)
            .orElseThrow(() -> new NotFoundException("User not found"));
    }

    public @Nullable User findUserOrNull(@NotNull String id) {
        return repository.findById(id).orElse(null);
    }
}
```

## Dependency Injection

### Constructor Injection

```java
// Good: Constructor injection with final fields
@Service
public class OrderService {
    private final UserRepository userRepository;
    private final ProductRepository productRepository;
    private final PaymentGateway paymentGateway;

    // Spring automatically autowires constructor
    public OrderService(UserRepository userRepository,
                       ProductRepository productRepository,
                       PaymentGateway paymentGateway) {
        this.userRepository = userRepository;
        this.productRepository = productRepository;
        this.paymentGateway = paymentGateway;
    }
}
```

### Avoid Field Injection

```java
// Bad: Field injection (hard to test, hides dependencies)
@Service
public class OrderService {
    @Autowired
    private UserRepository repository;  // DON'T DO THIS
}
```

## Java Tooling Integration

### Essential Commands

```bash
# Maven commands
mvn clean compile
mvn clean package
mvn test
mvn verify
mvn clean install

# Gradle commands
./gradlew build
./gradlew test
./gradlew check
./gradlew bootRun

# Format code
./gradlew spotlessApply

# Static analysis
./gradlew spotbugsMain
./gradlew checkstyleMain

# Dependency updates
./gradlew dependencyUpdates
mvn versions:display-dependency-updates
```

### Build Configuration

```groovy
// build.gradle (modern Groovy DSL)
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id 'com.diffplug.spotless' version '6.23.3'
    id 'com.github.spotbugs' version '6.0.0'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    // Validation
    implementation 'jakarta.validation:jakarta.validation-api'
    implementation 'org.hibernate.validator:hibernate-validator'

    // Lombok (optional)
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // Testing
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.junit.jupiter:junit-jupiter'
    testImplementation 'org.mockito:mockito-core'
    testImplementation 'org.assertj:assertj-core'
}

test {
    useJUnitPlatform()
    testLogging {
        events "passed", "skipped", "failed"
    }
}

spotless {
    java {
        googleJavaFormat()
        importOrder()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
    }
}
```

## Quick Reference: Java Idioms

| Idiom | Description |
|-------|-------------|
| Program to interfaces | Accept interfaces as parameters |
| Favor immutability | Use records, final fields |
| Use Optional for returns | Optional for nullable return values |
| Constructor injection | Prefer constructor over field injection |
| Try-with-resources | Auto-closeable resources |
| Stream API | Functional-style collection operations |
| Records (Java 16+) | Immutable data carriers |
| Sealed classes (Java 17+) | Restricted inheritance hierarchies |
| Pattern matching (Java 21+) | Type-safe pattern matching |
| Virtual threads (Java 21+) | Lightweight concurrency |

## Anti-Patterns to Avoid

```java
// Bad: Returning null from Optional-returning method
public Optional<User> findById(String id) {
    return null;  // WRONG - return Optional.empty() instead
}

// Bad: Calling get() without checking
User user = findById(id).get();  // WRONG - may throw NoSuchElementException

// Bad: Swallowing exceptions
try {
    riskyOperation();
} catch (Exception e) {
    // Empty catch - WRONG
}

// Bad: Using raw types
List list = new ArrayList();  // WRONG - use generics
List<String> list = new ArrayList<>();  // CORRECT

// Bad: Using == for string comparison
if (str == "value") { }  // WRONG - use str.equals("value")

// Bad: Exposing mutable state
public class ListHolder {
    private List<String> items = new ArrayList<>();

    public List<String> getItems() { return items; }  // WRONG - returns internal reference
    public List<String> getItems() { return new ArrayList<>(items); }  // CORRECT - defensive copy
}

// Bad: Using static mutable state
public class Counter {
    private static int count = 0;  // WRONG - causes issues with parallel tests
    public static synchronized void increment() { count++; }
}

// Bad: Over-engineering with unnecessary abstractions
public interface UserServiceFactoryProvider {  // WRONG - too many layers
    UserServiceFactory getFactory();
}
```

**Remember**: Java code should be clear, explicit, and follow modern Java conventions. When in doubt, prefer simplicity over cleverness. Use the latest Java features (records, pattern matching, sealed classes, virtual threads) to write more concise and maintainable code.
