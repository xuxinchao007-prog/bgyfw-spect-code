---
name: java-testing
description: Java testing patterns including JUnit 5, AssertJ, Mockito, TestContainers, and test coverage. Follows TDD methodology with modern Java testing practices.
---

# Java Testing Patterns

Comprehensive Java testing patterns for writing reliable, maintainable tests following TDD methodology with JUnit 5, AssertJ, and modern testing frameworks.

## When to Activate

- Writing new Java functions or methods
- Adding test coverage to existing code
- Creating benchmarks for performance-critical code
- Implementing integration tests with TestContainers
- Following TDD workflow in Java projects
- Testing Spring Boot applications

## TDD Workflow for Java

### The RED-GREEN-REFACTOR Cycle

```
RED     → Write a failing test first
GREEN   → Write minimal code to pass the test
REFACTOR → Improve code while keeping tests green
REPEAT  → Continue with next requirement
```

### Step-by-Step TDD in Java

```java
// Step 1: Define the interface
// Calculator.java
package com.example.calculator;

public class Calculator {
    public int add(int a, int b) {
        throw new UnsupportedOperationException("not implemented");
    }
}

// Step 2: Write failing test (RED)
// CalculatorTest.java
package com.example.calculator;

import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

class CalculatorTest {
    @Test
    void shouldAddTwoPositiveNumbers() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertThat(result).isEqualTo(5);
    }
}

// Step 3: Run test - verify FAIL
// $ mvn test
// [ERROR] shouldAddTwoPositiveNumbers()  Not yet implemented

// Step 4: Implement minimal code (GREEN)
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

// Step 5: Run test - verify PASS
// $ mvn test
// [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

// Step 6: Refactor if needed, verify tests still pass
```

## JUnit 5 Fundamentals

### Basic Test Structure

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
import static org.assertj.core.api.Assertions.*;

@DisplayName("Calculator Tests")
class CalculatorTest {

    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    @AfterEach
    void tearDown() {
        calculator = null;
    }

    @Test
    @DisplayName("should add two positive numbers correctly")
    void shouldAddTwoPositiveNumbers() {
        // Arrange
        int a = 5;
        int b = 3;

        // Act
        int result = calculator.add(a, b);

        // Assert
        assertThat(result).isEqualTo(8);
    }

    @Test
    @DisplayName("should throw exception when dividing by zero")
    void shouldThrowExceptionWhenDividingByZero() {
        // Arrange
        int dividend = 10;
        int divisor = 0;

        // Act & Assert
        assertThatThrownBy(() -> calculator.divide(dividend, divisor))
            .isInstanceOf(ArithmeticException.class)
            .hasMessage("Division by zero");
    }
}
```

### Lifecycle Methods

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)  // Optional: Single instance for all tests
class UserRepositoryTest {

    private UserRepository repository;
    private DataSource dataSource;

    @BeforeAll
    static void setUpClass() {
        // Runs once before all tests
        System.out.println("Starting UserRepository tests");
    }

    @BeforeEach
    void setUp() {
        // Runs before each test
        dataSource = createTestDataSource();
        repository = new UserRepository(dataSource);
    }

    @AfterEach
    void tearDown() {
        // Runs after each test
        repository.close();
        dataSource.close();
    }

    @AfterAll
    static void tearDownClass() {
        // Runs once after all tests
        System.out.println("Finished UserRepository tests");
    }
}
```

### Parameterized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.params.provider.Arguments.*;

class ParameterizedCalculatorTest {

    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "10, 15, 25",
        "-5, 3, -2"
    })
    @DisplayName("should add various numbers correctly")
    void shouldAddNumbers(int a, int b, int expected) {
        Calculator calc = new Calculator();
        assertThat(calc.add(a, b)).isEqualTo(expected);
    }

    @ParameterizedTest
    @ValueSource(strings = {"racecar", "madam", "level"})
    @DisplayName("should detect palindromes")
    void shouldDetectPalindromes(String input) {
        assertThat(StringUtils.isPalindrome(input)).isTrue();
    }

    @ParameterizedTest
    @EnumSource(ValueType.class)
    @DisplayName("should process all value types")
    void shouldProcessAllTypes(ValueType type) {
        assertThat(processor.canProcess(type)).isTrue();
    }

    @ParameterizedTest
    @MethodSource("provideTestCases")
    @DisplayName("should validate email addresses")
    void shouldValidateEmail(String email, boolean expected) {
        assertThat(EmailValidator.isValid(email)).isEqualTo(expected);
    }

    private static Stream<Arguments> provideTestCases() {
        return Stream.of(
            arguments("test@example.com", true),
            arguments("invalid-email", false),
            arguments("@example.com", false),
            arguments("test@", false)
        );
    }
}
```

## Test Organization

### Nested Tests

```java
@DisplayName("User Service Tests")
class UserServiceTest {

    private UserService userService;
    private UserRepository mockRepository;

    @BeforeEach
    void setUp() {
        mockRepository = mock(UserRepository.class);
        userService = new UserService(mockRepository);
    }

    @Nested
    @DisplayName("When creating users")
    class CreateUserTests {

        @Test
        @DisplayName("should create user with valid data")
        void shouldCreateUserWithValidData() {
            // Test implementation
        }

        @Test
        @DisplayName("should reject user with invalid email")
        void shouldRejectUserWithInvalidEmail() {
            // Test implementation
        }
    }

    @Nested
    @DisplayName("When updating users")
    class UpdateUserTests {

        @Test
        @DisplayName("should update existing user")
        void shouldUpdateExistingUser() {
            // Test implementation
        }

        @Test
        @DisplayName("should throw when user not found")
        void shouldThrowWhenUserNotFound() {
            // Test implementation
        }
    }
}
```

### Test Helpers and Fixtures

```java
class UserTestFixture {
    static User validUser() {
        return new User(
            "user-123",
            "john.doe@example.com",
            "John",
            "Doe"
        );
    }

    static User invalidUser() {
        return new User(
            "user-456",
            "invalid-email",
            "",
            "Doe"
        );
    }

    static UserCreateRequest validCreateRequest() {
        return new UserCreateRequest(
            "jane.doe@example.com",
            "Jane",
            "Doe"
        );
    }
}

// Usage in tests
class UserServiceTest {
    @Test
    void shouldCreateUser() {
        User user = UserTestFixture.validUser();
        // Test with user fixture
    }
}
```

## AssertJ Assertions

### Fluent Assertions

```java
import static org.assertj.core.api.Assertions.*;

class AssertJExamples {

    @Test
    void objectAssertions() {
        User user = new User("123", "john@example.com");

        assertThat(user)
            .isNotNull()
            .hasFieldOrPropertyWithValue("id", "123")
            .extracting("email")
            .isNotEmpty();
    }

    @Test
    void collectionAssertions() {
        List<String> names = List.of("Alice", "Bob", "Charlie");

        assertThat(names)
            .hasSize(3)
            .contains("Bob", "Charlie")
            .doesNotContain("David")
            .allSatisfy(name -> {
                assertThat(name).isNotBlank();
                assertThat(name).isUpperCase();
            });
    }

    @Test
    void stringAssertions() {
        String email = "user@example.com";

        assertThat(email)
            .isNotEmpty()
            .contains("@")
            .endsWith(".com")
            .matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }

    @Test
    void numberAssertions() {
        int value = 42;

        assertThat(value)
            .isPositive()
            .isGreaterThan(40)
            .isLessThan(50)
            .isBetween(40, 45);
    }

    @Test
    void exceptionAssertions() {
        assertThatThrownBy(() -> {
            throw new IllegalStateException("Error occurred");
        })
            .isInstanceOf(IllegalStateException.class)
            .hasMessage("Error occurred")
            .hasNoCause();

        assertThatExceptionOfType(IllegalArgumentException.class)
            .isThrownBy(() -> {
                throw new IllegalArgumentException("Invalid argument");
            })
            .withMessageContaining("Invalid");
    }

    @Test
    void optionalAssertions() {
        Optional<String> optional = Optional.of("value");

        assertThat(optional)
            .isPresent()
            .hasValue("value");

        Optional<String> empty = Optional.empty();

        assertThat(empty)
            .isNotPresent()
            .isEmpty();
    }
}
```

### Custom Assertions

```java
public class UserAssertions extends AbstractAssert<UserAssertions, User> {

    private UserAssertions(User actual) {
        super(actual, UserAssertions.class);
    }

    public static UserAssertions assertThat(User actual) {
        return new UserAssertions(actual);
    }

    public UserAssertions hasId(String id) {
        isNotNull();
        if (!actual.getId().equals(id)) {
            failWithMessage("Expected user to have id <%s> but was <%s>", id, actual.getId());
        }
        return this;
    }

    public UserAssertions isActive() {
        isNotNull();
        if (!actual.isActive()) {
            failWithMessage("Expected user to be active but was not");
        }
        return this;
    }

    public UserAssertions hasEmailEndingWith(String domain) {
        isNotNull();
        if (!actual.getEmail().endsWith(domain)) {
            failWithMessage("Expected email to end with <%s> but was <%s>",
                domain, actual.getEmail());
        }
        return this;
    }
}

// Usage
class CustomAssertionTest {
    @Test
    void testUserWithCustomAssertions() {
        User user = new User("123", "john@example.com");

        UserAssertions.assertThat(user)
            .hasId("123")
            .isActive()
            .hasEmailEndingWith("@example.com");
    }
}
```

## Mocking with Mockito

### Basic Mocking

```java
import org.mockito.*;
import static org.mockito.Mockito.*;

class MockitoExamples {

    private UserRepository userRepository;
    private EmailService emailService;
    private UserService userService;

    @BeforeEach
    void setUp() {
        userRepository = mock(UserRepository.class);
        emailService = mock(EmailService.class);
        userService = new UserService(userRepository, emailService);
    }

    @Test
    void shouldReturnUserFromRepository() {
        // Arrange
        User expectedUser = new User("123", "john@example.com");
        when(userRepository.findById("123")).thenReturn(Optional.of(expectedUser));

        // Act
        User actualUser = userService.getUser("123");

        // Assert
        assertThat(actualUser).isEqualTo(expectedUser);
        verify(userRepository).findById("123");
    }

    @Test
    void shouldThrowWhenUserNotFound() {
        // Arrange
        when(userRepository.findById("999")).thenReturn(Optional.empty());

        // Act & Assert
        assertThatThrownBy(() -> userService.getUser("999"))
            .isInstanceOf(UserNotFoundException.class);

        verify(userRepository).findById("999");
    }

    @Test
    void shouldSendWelcomeEmail() {
        // Arrange
        User user = new User("123", "john@example.com");
        when(userRepository.save(any(User.class))).thenReturn(user);
        doNothing().when(emailService).sendWelcomeEmail(anyString());

        // Act
        userService.createUser("john@example.com", "John", "Doe");

        // Assert
        verify(emailService).sendWelcomeEmail("john@example.com");
        verify(emailService, times(1)).sendWelcomeEmail(anyString());
        verify(emailService, atLeastOnce()).sendWelcomeEmail(anyString());
    }

    @Test
    void shouldNeverSendEmailWhenUserCreationFails() {
        // Arrange
        when(userRepository.save(any(User.class)))
            .thenThrow(new DatabaseException("Connection failed"));

        // Act & Assert
        assertThatThrownBy(() ->
            userService.createUser("john@example.com", "John", "Doe")
        ).isInstanceOf(DatabaseException.class);

        verify(emailService, never()).sendWelcomeEmail(anyString());
    }
}
```

### Argument Matchers

```java
class ArgumentMatcherExamples {

    @Test
    void shouldMatchAnyArgument() {
        when(repository.save(any(User.class))).thenReturn(new User("123", "test@example.com"));
        when(repository.findById(anyString())).thenReturn(Optional.of(new User("123", "test@example.com")));

        userService.createUser("any@example.com", "Any", "User");

        verify(repository).save(any(User.class));
        verify(repository).save(argThat(user -> user.email().contains("@")));
    }

    @Test
    void shouldMatchSpecificArgument() {
        when(repository.findById(eq("123"))).thenReturn(Optional.of(user));
        when(repository.save(argThat(user -> user.email().endsWith("@example.com"))))
            .thenReturn(user);

        // Test implementation
    }

    @Test
    void shouldCaptureArgument() {
        // Arrange
        ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
        when(repository.save(any(User.class))).thenReturn(new User("123", "john@example.com"));

        // Act
        userService.createUser("john@example.com", "John", "Doe");

        // Assert
        verify(repository).save(userCaptor.capture());
        User capturedUser = userCaptor.getValue();
        assertThat(capturedUser.email()).isEqualTo("john@example.com");
        assertThat(capturedUser.firstName()).isEqualTo("John");
    }
}
```

### Mockito Annotations

```java
@ExtendWith(MockitoExtension.class)
class MockitoAnnotationTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Spy
    private PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

    @Captor
    private ArgumentCaptor<User> userCaptor;

    @Test
    void shouldCreateUserWithMocks() {
        // Mocks are automatically injected
        when(userRepository.save(any(User.class)))
            .thenReturn(new User("123", "test@example.com"));

        userService.createUser("test@example.com", "Test", "User");

        verify(userRepository).save(userCaptor.capture());
    }
}
```

## Integration Testing with TestContainers

### Database Integration Tests

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
class UserRepositoryIntegrationTest {

    @Container
    private static final PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    private UserRepository repository;
    private DataSource dataSource;

    @BeforeEach
    void setUp() {
        dataSource = createDataSourceFromContainer(postgres);
        runMigrations(dataSource);
        repository = new JdbcUserRepository(dataSource);
    }

    @Test
    void shouldSaveAndRetrieveUser() {
        // Arrange
        User user = new User(null, "john@example.com", "John", "Doe");

        // Act
        User savedUser = repository.save(user);

        // Assert
        assertThat(savedUser.id()).isNotNull();

        Optional<User> foundUser = repository.findById(savedUser.id());
        assertThat(foundUser).isPresent();
        assertThat(foundUser.get().email()).isEqualTo("john@example.com");
    }

    @Test
    void shouldHandleTransactionRollback() {
        assertThatThrownBy(() -> {
            repository.transactionally(() -> {
                repository.save(new User(null, "test1@example.com", "Test", "1"));
                repository.save(new User(null, "test2@example.com", "Test", "2"));
                throw new RuntimeException("Rollback!");
            });
        }).isInstanceOf(RuntimeException.class);

        assertThat(repository.findAll()).isEmpty();
    }

    private DataSource createDataSourceFromContainer(PostgreSQLContainer<?> container) {
        return DataSourceBuilder.create()
            .url(container.getJdbcUrl())
            .username(container.getUsername())
            .password(container.getPassword())
            .build();
    }
}
```

### Spring Boot TestContainers

```java
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);

    @Autowired
    private OrderService orderService;

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void shouldCreateOrderSuccessfully() {
        // Test implementation with real database
    }
}
```

### Multiple Containers

```java
@Testcontainers
class MultiContainerTest {

    @Container
    private static final PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    private static final GenericContainer<?> redis =
        new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);

    @Container
    private static final KafkaContainer kafka =
        new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.5.0"));

    @DynamicTestFactory
    Stream<DynamicTest> dynamicTests() {
        return Stream.of(
            DynamicTest.dynamicTest("test with postgres", () -> {
                // Test with PostgreSQL
            }),
            DynamicTest.dynamicTest("test with redis", () -> {
                // Test with Redis
            }),
            DynamicTest.dynamicTest("test with kafka", () -> {
                // Test with Kafka
            })
        );
    }
}
```

## Spring Boot Testing

### Controller Testing

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    @DisplayName("should return user when found")
    void shouldReturnUserWhenFound() throws Exception {
        User user = new User("123", "john@example.com", "John", "Doe");
        when(userService.getUser("123")).thenReturn(user);

        mockMvc.perform(get("/api/users/123"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value("123"))
            .andExpect(jsonPath("$.email").value("john@example.com"))
            .andExpect(jsonPath("$.firstName").value("John"));
    }

    @Test
    @DisplayName("should return 404 when user not found")
    void shouldReturn404WhenNotFound() throws Exception {
        when(userService.getUser("999"))
            .thenThrow(new UserNotFoundException("User not found"));

        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }

    @Test
    @DisplayName("should create new user")
    void shouldCreateNewUser() throws Exception {
        User created = new User("123", "jane@example.com", "Jane", "Doe");
        when(userService.createUser(any(UserCreateRequest.class)))
            .thenReturn(created);

        String requestBody = """
            {
                "email": "jane@example.com",
                "firstName": "Jane",
                "lastName": "Doe"
            }
            """;

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"));
    }
}
```

### Service Testing

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    @DisplayName("should create user and send welcome email")
    void shouldCreateUserAndSendWelcomeEmail() {
        UserCreateRequest request = new UserCreateRequest(
            "john@example.com", "John", "Doe"
        );

        User savedUser = new User("123", "john@example.com", "John", "Doe");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        doNothing().when(emailService).sendWelcomeEmail(anyString());

        User result = userService.createUser(request);

        assertThat(result).isEqualTo(savedUser);
        InOrder inOrder = inOrder(userRepository, emailService);
        inOrder.verify(userRepository).save(any(User.class));
        inOrder.verify(emailService).sendWelcomeEmail("john@example.com");
    }
}
```

### Repository Testing

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository userRepository;

    @Test
    @DisplayName("should find user by email")
    void shouldFindByEmail() {
        User user = new User(null, "john@example.com", "John", "Doe");
        entityManager.persist(user);
        entityManager.flush();

        Optional<User> found = userRepository.findByEmail("john@example.com");

        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("john@example.com");
    }

    @Test
    @DisplayName("should return empty when email not found")
    void shouldReturnEmptyWhenNotFound() {
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");

        assertThat(found).isEmpty();
    }
}
```

## Performance Testing

### JMH Benchmarks

```java
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 3, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
public class StringConcatenationBenchmark {

    @Benchmark
    public String stringConcatenation() {
        String result = "";
        for (int i = 0; i < 100; i++) {
            result += i;
        }
        return result;
    }

    @Benchmark
    public String stringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 100; i++) {
            sb.append(i);
        }
        return sb.toString();
    }

    @Benchmark
    public String stringJoin() {
        return IntStream.range(0, 100)
            .mapToObj(String::valueOf)
            .collect(Collectors.joining());
    }
}
```

## Test Coverage

### JaCoCo Configuration

```groovy
// build.gradle
plugins {
    id 'jacoco'
}

test {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}

jacoco {
    toolVersion = "0.8.11"
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        csv.required = false
        html.outputLocation = layout.buildDirectory.dir('jacocoHtml')
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = 0.80  // 80% coverage required
            }
        }

        rule {
            element = 'CLASS'
            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.70
            }
            excludes = [
                '*.config.*',
                '*.dto.*',
                '*.entity.*'
            ]
        }
    }
}

tasks.named('check') {
    dependsOn jacocoTestCoverageVerification
}
```

### Coverage Targets

| Code Type | Target |
|-----------|--------|
| Critical business logic | 100% |
| Public APIs | 90%+ |
| General code | 80%+ |
| Generated code | Exclude |
| DTOs/Entities | Exclude |

## Testing Commands

```bash
# Maven commands
mvn test                                   # Run all tests
mvn test -Dtest=UserServiceTest            # Run specific test class
mvn test -Dtest=UserServiceTest#shouldCreateUser  # Run specific test method
mvn test -Dgroups=unit                     # Run unit tests only
mvn verify                                 # Run tests with coverage verification

# Gradle commands
./gradlew test                            # Run all tests
./gradlew test --tests UserServiceTest    # Run specific test class
./gradlew test --tests "*ServiceTest"     # Run tests matching pattern
./gradlew check                           # Run tests with checks
./gradlew jacocoTestReport                # Generate coverage report
./gradlew cleanTest test                  # Clean and run tests

# Run with specific profile
mvn test -Pintegration                    # Run integration tests
./gradlew integrationTest                  # Run integration tests

# Run with specific tags
mvn test -Dgroups="slow"                  # Run slow tests
mvn test -DexcludedGroups="slow"          # Exclude slow tests
```

## Best Practices

**DO:**
- Write tests FIRST (TDD)
- Use descriptive test names that explain the scenario
- Follow Arrange-Act-Assert pattern
- Use AssertJ for fluent assertions
- Mock external dependencies
- Test behavior, not implementation
- Use test fixtures for common test data
- Keep tests independent and isolated
- Clean up resources in @AfterEach
- Use TestContainers for integration tests

**DON'T:**
- Test private methods directly (test through public API)
- Use Thread.sleep() in tests (use proper synchronization)
- Ignore flaky tests (fix or remove them)
- Mock everything (prefer integration tests when possible)
- Test getters/setters
- Write tests that depend on execution order
- Expose internal state for testing
- Use production configuration in tests
- Share mutable state between tests

## Test Naming Conventions

```java
// Good: Descriptive names
@Test
@DisplayName("should return user when valid id is provided")
void shouldReturnUserWhenValidIdIsProvided() { }

@Test
@DisplayName("should throw exception when user not found")
void shouldThrowExceptionWhenUserNotFound() { }

@Test
@DisplayName("should send welcome email after user creation")
void shouldSendWelcomeEmailAfterUserCreation() { }

// Bad: Vague names
@Test
void testUser() { }
@Test
void test1() { }
@Test
void works() { }
```

## Quick Reference: Testing Annotations

| Annotation | Purpose |
|------------|---------|
| `@Test` | Marks a test method |
| `@BeforeEach` | Runs before each test |
| `@AfterEach` | Runs after each test |
| `@BeforeAll` | Runs once before all tests (static) |
| `@AfterAll` | Runs once after all tests (static) |
| `@DisplayName` | Custom test display name |
| `@Nested` | Group related tests |
| `@ParameterizedTest` | Parameterized test |
| `@RepeatedTest` | Repeat test multiple times |
| `@Disabled` | Disable test |
| `@Tag` | Tag tests for filtering |

**Remember**: Tests are documentation. They show how your code is meant to be used. Write them clearly, keep them maintainable, and ensure they run quickly. A good test suite enables confident refactoring and rapid development.
