# Agent Orchestration

## Available Agents

Located in `~/.claude/agents/`:

### Core Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| tdd-guide | Test-driven development | New features, bug fixes |
| code-reviewer | General code review | After writing code |
| security-reviewer | Security analysis | Before commits |
| build-error-resolver | General build errors | When build fails |
| e2e-runner | E2E testing | Critical user flows |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation | Updating docs |

### Language-Specific Agents (MANDATORY for respective languages)

| Agent | Language | Purpose | When to Use |
|-------|----------|---------|-------------|
| **java-reviewer** | Java | Expert Java code review | **MUST BE USED** for all Java projects |
| **java-build-resolver** | Java | Maven/Gradle build fixes | Java build failures |
| **python-reviewer** | Python | Expert Python code review | **MUST BE USED** for all Python projects |
| **python-build-resolver** | Python | pip/pipenv/poetry fixes | Python build/install failures |
| **javascript-reviewer** | JS/TS | JavaScript/TypeScript review | **MUST BE USED** for all JS/TS projects |
| **javascript-build-resolver** | JS/TS | npm/yarn/pnpm build fixes | JavaScript build failures |
| **vue-reviewer** | Vue.js | Vue.js code review | **MUST BE USED** for all Vue projects |
| **go-reviewer** | Go | Go code review | Go projects |
| **go-build-resolver** | Go | Go build/vet fixes | Go build failures |

### Database-Specific Agents

| Agent | Database | Purpose | When to Use |
|-------|----------|---------|-------------|
| database-reviewer | General | General database review | Any database |
| **db-postgresql-reviewer** | PostgreSQL | PostgreSQL optimization | PostgreSQL queries |
| **db-mysql-reviewer** | MySQL | MySQL optimization | MySQL queries |
| **db-mongo-reviewer** | MongoDB | MongoDB optimization | MongoDB queries |
| **db-oracle-reviewer** | Oracle | Oracle/PLSQL optimization | Oracle queries |
| **db-sqlserver-reviewer** | SQL Server | T-SQL optimization | SQL Server queries |

### Code Quality Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| **performance-auditor** | Frontend performance optimization | Use PROACTIVELY when developing features, before releases |
| **smell-detector** | Code smell detection | During code reviews, refactoring sessions |
| refactor-cleaner-java | Java-specific dead code cleanup | Java code maintenance |

## Language-Specific Agent Usage

### Java Projects
- **MANDATORY**: Use `java-reviewer` for all Java code changes
- Use `java-build-resolver` when Maven/Gradle build fails
- Use `refactor-cleaner-java` for Java-specific code cleanup

**The java-reviewer will check for:**
- Idiomatic Java patterns (Java 17+ features)
- Concurrency issues and thread safety
- Exception handling best practices
- Spring/Jakarta EE framework patterns
- Security vulnerabilities
- Resource management (try-with-resources)

### Python Projects
- **MANDATORY**: Use `python-reviewer` for all Python code changes
- Use `python-build-resolver` when pip/pipenv/poetry fails

**The python-reviewer will check for:**
- PEP 8 compliance
- Async/await patterns
- Type hints usage
- Framework best practices (Django, FastAPI, Flask)
- Security issues
- Error handling

### JavaScript/TypeScript Projects
- **MANDATORY**: Use `javascript-reviewer` for all JS/TS code changes
- Use `javascript-build-resolver` when npm/yarn/pnpm build fails

**The javascript-reviewer will check for:**
- Modern ES6+ patterns
- TypeScript type safety
- React/Vue/Angular best practices
- Async patterns
- Security issues
- Bundle optimization

### Vue.js Projects
- **MANDATORY**: Use `vue-reviewer` for all Vue code changes

**The vue-reviewer will check for:**
- Composition API patterns
- Reactivity system usage
- Component design
- Performance optimization
- Vue 3 best practices

### Database Projects
Use appropriate database reviewer based on your DB:
- PostgreSQL → `db-postgresql-reviewer`
- MySQL → `db-mysql-reviewer`
- MongoDB → `db-mongo-reviewer`
- Oracle → `db-oracle-reviewer`
- SQL Server → `db-sqlserver-reviewer`

### Code Quality
- Use `smell-detector` during refactoring sessions
- Use `performance-auditor` when developing frontend features

## Immediate Agent Usage

### Core Agents (No user prompt needed)
1. Complex feature requests → Use **planner** agent
2. Code just written/modified → Use language-specific **reviewer** agent
3. Bug fix or new feature → Use **tdd-guide** agent
4. Architectural decision → Use **architect** agent

### Language-Specific (Automatic)
- Java code changes → **java-reviewer** (MANDATORY)
- Python code changes → **python-reviewer** (MANDATORY)
- JavaScript/TypeScript changes → **javascript-reviewer** (MANDATORY)
- Vue code changes → **vue-reviewer** (MANDATORY)
- Go code changes → **go-reviewer** (MANDATORY)

### Database Operations (Automatic)
- PostgreSQL queries → **db-postgresql-reviewer**
- MySQL queries → **db-mysql-reviewer**
- MongoDB queries → **db-mongo-reviewer**

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:

```markdown
# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Agent 1: Security analysis of auth.ts
2. Agent 2: Performance review of cache system
3. Agent 3: Type checking of utils.ts

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

### Example: Language-Specific Parallel Review

```markdown
# Parallel multi-language review
Launch agents in parallel:
1. java-reviewer: Review Java service layer
2. db-postgresql-reviewer: Review SQL queries
3. security-reviewer: Security audit of authentication
```

## Multi-Perspective Analysis

For complex problems, use split role sub-agents:
- Factual reviewer
- Senior engineer
- Security expert
- Consistency reviewer
- Redundancy checker

## Agent Selection Flowchart

```
┌─────────────────┐
│ Code Change?    │
└────────┬────────┘
         │
    ┌────┴────┐
    │ What    │
    │ Language?│
    └────┬────┘
         │
    ┌────┴────────────────────────────────────┐
    │                                          │
┌───┴────┐ ┌────────┐ ┌─────────┐ ┌────────┐
│ Java   │ │Python │ │JS/TS    │ │Vue     │
└───┬────┘ └───┬────┘ └────┬────┘ └───┬────┘
    │          │           │          │
    ▼          ▼           ▼          ▼
java-    python-  javascript- vue-
reviewer reviewer   reviewer  reviewer
```

## Agent Priority Matrix

| Agent | Priority | Automation |
|-------|----------|------------|
| java-reviewer | 🔴 CRITICAL | Mandatory for Java |
| python-reviewer | 🔴 CRITICAL | Mandatory for Python |
| javascript-reviewer | 🔴 CRITICAL | Mandatory for JS/TS |
| vue-reviewer | 🔴 CRITICAL | Mandatory for Vue |
| go-reviewer | 🔴 CRITICAL | Mandatory for Go |
| security-reviewer | 🔴 CRITICAL | Before all commits |
| performance-auditor | 🟡 HIGH | PROACTIVE use |
| smell-detector | 🟡 HIGH | During refactoring |
| tdd-guide | 🟡 HIGH | Before implementation |
| db-*-reviewer | 🟢 MEDIUM | When writing queries |
