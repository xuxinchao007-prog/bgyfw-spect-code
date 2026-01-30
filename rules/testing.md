# Testing Requirements

## Minimum Test Coverage: 80%

Test Types (ALL required):
1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations
3. **E2E Tests** - Critical user flows (Playwright)

## Test-Driven Development

MANDATORY workflow:
1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)

## Language-Specific Testing

### Java Testing (See: `java-testing` skill)

**Frameworks:**
- JUnit 5 for unit tests
- AssertJ for assertions
- Mockito for mocking
- Spring Boot Test for integration tests

**Best Practices:**
```java
// Table-driven tests for multiple scenarios
@ParameterizedTest
@CsvSource({
    "valid@example.com, true",
    "invalid-email, false",
    "null, false"
})
void testEmailValidation(String email, boolean expected) {
    assertThat(validator.isValid(email)).isEqualTo(expected);
}

// Use @Nested for related test groups
@Nested
class CreateUserTests {
    @Test
    void withValidData_returnsUser() { }

    @Test
    void withDuplicateEmail_throwsException() { }
}
```

**Commands:**
```bash
# Run tests
mvn test
./gradlew test

# Run with coverage
mvn verify jacoco:report
./gradlew test jacocoTestReport

# Run specific test
mvn test -Dtest=UserServiceTest
./gradlew test --tests UserServiceTest
```

**MANDATORY**: Use `java-reviewer` agent for all Java test code review.

---

### Python Testing (See: `python-testing` skill)

**Frameworks:**
- pytest for testing
- pytest-cov for coverage
- pytest-mock for mocking
- pytest-asyncio for async tests

**Best Practices:**
```python
# Parametrize tests for multiple scenarios
@pytest.mark.parametrize("email,expected", [
    ("valid@example.com", True),
    ("invalid-email", False),
    (None, False),
])
def test_email_validation(email, expected):
    assert validator.is_valid(email) == expected

# Use fixtures for common setup
@pytest.fixture
def user(db):
    return User.objects.create(email="test@example.com")

def test_user_can_login(user, client):
    response = client.post("/login", {"email": user.email})
    assert response.status_code == 200
```

**Commands:**
```bash
# Run tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html

# Run specific test
pytest tests/test_users.py::test_create_user

# Run async tests
pytest -v --asyncio-mode=auto
```

**MANDATORY**: Use `python-reviewer` agent for all Python test code review.

---

### Go Testing (See: `golang-testing` skill)

**Best Practices:**
```go
// Table-driven tests are idiomatic
func TestEmailValidation(t *testing.T) {
    tests := []struct {
        name     string
        email    string
        expected bool
    }{
        {"valid email", "valid@example.com", true},
        {"invalid email", "invalid-email", false},
        {"empty", "", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := validator.IsValid(tt.email)
            if got != tt.expected {
                t.Errorf("IsValid() = %v, want %v", got, tt.expected)
            }
        })
    }
}

// Use t.Run for subtests
func TestUser(t *testing.T) {
    t.Run("create", func(t *testing.T) { })
    t.Run("delete", func(t *testing.T) { })
}
```

**Commands:**
```bash
# Run tests
go test ./...

# Run with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...

# Run specific test
go test -run TestEmailValidation ./...

# Benchmark
go test -bench=. ./...
```

**MANDATORY**: Use `go-reviewer` agent for all Go test code review.

---

### JavaScript/TypeScript Testing

**Frameworks:**
- Jest or Vitest for unit tests
- React Testing Library for React components
- Playwright for E2E tests

**Best Practices:**
```typescript
// Describe/it pattern
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', () => {
      const user = userService.create({
        email: 'valid@example.com',
        name: 'Test User'
      })
      expect(user.email).toBe('valid@example.com')
    })

    it('should throw with invalid email', () => {
      expect(() => userService.create({ email: 'invalid' }))
        .toThrow('Invalid email')
    })
  })
})

// Test each prop
describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click</Button>)
    await userEvent.click(screen.getByText('Click'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

**Commands:**
```bash
# Jest
npm test
npm test -- --coverage

# Vitest
npm run test
npm run test -- --coverage

# Watch mode
npm test -- --watch

# Specific test
npm test -- Button.test.ts
```

**MANDATORY**: Use `javascript-reviewer` agent for all JavaScript/TypeScript test code review.

---

### Vue.js Testing

**Frameworks:**
- Vue Test Utils for component testing
- Vitest for unit tests
- Pinia Testing for store testing

**Best Practices:**
```typescript
import { mount } from '@vue/test-utils'
import { createPinia } from 'pinia'

describe('UserProfile', () => {
  it('displays user name', () => {
    const wrapper = mount(UserProfile, {
      props: { user: { name: 'Alice' } },
      global: { plugins: [createPinia()] }
    })
    expect(wrapper.text()).toContain('Alice')
  })

  it('emits save event', async () => {
    const wrapper = mount(UserProfile)
    await wrapper.find('button').trigger('click')
    expect(wrapper.emitted('save')).toBeTruthy()
  })
})
```

**MANDATORY**: Use `vue-reviewer` agent for all Vue test code review.

---

## Troubleshooting Test Failures

1. Use **tdd-guide** agent
2. Check test isolation
3. Verify mocks are correct
4. Fix implementation, not tests (unless tests are wrong)

## Agent Support

### Testing Agents
- **tdd-guide** - Use PROACTIVELY for new features, enforces write-tests-first
- **e2e-runner** - Playwright E2E testing specialist

### Language-Specific Review Agents (MANDATORY)
- **java-reviewer** - Java test code review (JUnit, AssertJ, Mockito)
- **python-reviewer** - Python test code review (pytest, fixtures)
- **go-reviewer** - Go test code review (table-driven tests)
- **javascript-reviewer** - JavaScript/TypeScript test code review (Jest, Vitest)
- **vue-reviewer** - Vue.js test code review (Vue Test Utils)

### Build Error Resolution
- **java-build-resolver** - Fix Maven/Gradle test failures
- **python-build-resolver** - Fix pytest failures
- **javascript-build-resolver** - Fix npm/yarn test failures
- **go-build-resolver** - Fix Go test failures
