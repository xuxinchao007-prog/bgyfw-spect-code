# Coding Style

## Universal Principles (ALL Languages)

### Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate:

```javascript
// WRONG: Mutation
function updateUser(user, name) {
  user.name = name  // MUTATION!
  return user
}

// CORRECT: Immutability
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

### File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large components
- Organize by feature/domain, not by type

### Error Handling

ALWAYS handle errors comprehensively:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('Detailed user-friendly message')
}
```

### Input Validation

ALWAYS validate user input:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

---

## Language-Specific Guidelines

### Java (See: `java-patterns` skill)

**Modern Java (Java 17+) Patterns:**

```java
// Use records for immutable data carriers (Java 16+)
public record User(String id, String name, String email) {
    public User {
        Objects.requireNonNull(name);
        Objects.requireNonNull(email);
    }
}

// Use sealed classes for restricted inheritance (Java 17+)
public sealed interface Shape permits Circle, Rectangle {
    double area();
}

// Use Optional for return values (NOT for parameters)
public Optional<User> findById(String id) {
    return repository.findById(id);
}

// Constructor injection over field injection
@Service
public class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = Objects.requireNonNull(repository);
    }
}

// Try-with-resources for AutoCloseable
try (var reader = Files.newBufferedReader(path)) {
    // Auto-closed
}

// Stream API for collection operations
List<String> activeNames = users.stream()
    .filter(User::isActive)
    .map(User::name)
    .toList();  // Java 16+
```

**MANDATORY**: Use `java-reviewer` agent for all Java code review.

---

### Python (See: `python-patterns` skill)

**PEP 8 Best Practices:**

```python
# Use dataclasses for data containers (Python 3.7+)
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    age: int = 0

# Use type hints
def create_user(name: str, email: str) -> User:
    return User(name=name, email=email)

# Use context managers
with open('file.txt') as f:
    content = f.read()

# Use pathlib instead of os.path
from pathlib import Path
path = Path('dir') / 'file.txt'

# Use f-strings for string formatting
message = f"Hello, {name}"

# Use list comprehensions
squares = [x * x for x in range(10)]

# Use Optional for nullable returns
from typing import Optional

def find_user(id: str) -> Optional[User]:
    return repository.get(id)

# Prefer composition over inheritance
# Use @dataclass over class with boilerplate
# Use @property for computed attributes
```

**MANDATORY**: Use `python-reviewer` agent for all Python code review.

---

### JavaScript/TypeScript

**Modern Patterns:**

```typescript
// TypeScript for type safety
interface User {
    id: string
    name: string
    email: string
}

// Immutability with spread operator
function updateUser(user: User, updates: Partial<User>): User {
    return { ...user, ...updates }
}

// Async/await over promises
async function fetchUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`)
    return response.json()
}

// Nullish coalescing
const value = input ?? 'default'

// Optional chaining
const city = user?.address?.city

// Destructuring
const { name, email } = user
const [first, second] = array

// Map/filter/reduce
const active = users.filter(u => u.active)
const names = users.map(u => u.name)

// Avoid any - use specific types or generics
function identity<T>(value: T): T {
    return value
}

// Use const by default, let when reassigning needed
const API_URL = 'https://api.example.com'
let count = 0
```

**MANDATORY**: Use `javascript-reviewer` agent for all JavaScript/TypeScript code review.

---

### Vue.js (See: `vue-reviewer` agent)

**Composition API Patterns:**

```vue
<script setup lang="ts">
// Use script setup with TypeScript
import { ref, computed, onMounted } from 'vue'

// Reactive state
const count = ref(0)
const user = ref<User | null>(null)

// Computed properties
const doubleCount = computed(() => count.value * 2)

// Methods
function increment() {
  count.value++
}

// Lifecycle hooks
onMounted(() => {
  fetchUser()
})
</script>

// Prefer composition API over options API
// Use <script setup> for cleaner code
// Use defineProps and defineEmits with TypeScript
// Use composables for reusable logic
// Use provide/inject for dependency injection
```

**MANDATORY**: Use `vue-reviewer` agent for all Vue code review.

---

### Go (See: `golang-patterns` skill)

**Idiomatic Go:**

```go
// Use interfaces
type Reader interface {
    Read([]byte) (int, error)
}

// Error handling - never ignore errors
file, err := os.Open("file.txt")
if err != nil {
    return err
}
defer file.Close()

// Use struct embedding for composition
type User struct {
    Person
    ID string
}

// Use channels for goroutine communication
ch := make(chan Result)
go func() {
    ch <- process()
}()

// Defer for cleanup
defer file.Close()
defer mutex.Unlock()

// Use pointers for large structs or mutability
func UpdateUser(u *User) {
    u.Name = "Updated"
}

// Table-driven tests
for _, tc := range tests {
    t.Run(tc.name, func(t *testing.T) {
        // test case
    })
}
```

**MANDATORY**: Use `go-reviewer` agent for all Go code review.

---

## Code Quality Checklist

### Universal (ALL Languages)
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No console.log/debug statements in production
- [ ] No hardcoded secrets/values
- [ ] No mutation (immutable patterns used)

### Java-Specific
- [ ] Records used for data carriers
- [ ] Sealed classes for restricted inheritance
- [ ] Optional for return values
- [ ] Try-with-resources for AutoCloseable
- [ ] Constructor injection
- [ ] Stream API for collections

### Python-Specific
- [ ] PEP 8 compliant
- [ ] Type hints on functions
- [ ] Dataclasses for data containers
- [ ] Context managers for resources
- [ ] F-strings for formatting
- [ ] List comprehensions over loops

### JavaScript/TypeScript-Specific
- [ ] TypeScript types defined
- [ ] Async/await used
- [ ] Nullish coalescing (??)
- [ ] Optional chaining (?.)
- [ ] Const by default
- [ ] No 'any' types

### Vue.js-Specific
- [ ] Composition API used
- [ ] `<script setup>` format
- [ ] TypeScript enabled
- [ ] Reactive state properly typed
- [ ] Composables for reusable logic

### Go-Specific
- [ ] Errors handled (never ignored)
- [ ] Defer used for cleanup
- [ ] Channels for goroutine communication
- [ ] Interfaces for abstraction
- [ ] Table-driven tests
