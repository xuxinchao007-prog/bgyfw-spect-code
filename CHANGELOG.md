# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - 2026-01-30

### Added
- **14 new language-specific agents** for comprehensive code review:
  - `java-reviewer` - Expert Java code review (MANDATORY for Java projects)
  - `java-build-resolver` - Maven/Gradle build error resolution
  - `python-reviewer` - Expert Python code review (MANDATORY for Python projects)
  - `python-build-resolver` - pip/pipenv/poetry error resolution
  - `javascript-reviewer` - JavaScript/TypeScript code review (MANDATORY for JS/TS projects)
  - `javascript-build-resolver` - npm/yarn/pnpm build error resolution
  - `vue-reviewer` - Vue.js code review (MANDATORY for Vue projects)
  - `performance-auditor` - Frontend performance optimization specialist
  - `smell-detector` - Code smell detection and refactoring guidance
  - `refactor-cleaner-java` - Java-specific dead code cleanup
  - `db-postgresql-reviewer` - PostgreSQL query optimization
  - `db-mysql-reviewer` - MySQL query optimization
  - `db-oracle-reviewer` - Oracle/PLSQL optimization
  - `db-sqlserver-reviewer` - SQL Server/T-SQL optimization
  - `db-mongo-reviewer` - MongoDB query optimization

- **5 new language-specific commands**:
  - `/java-review` - Quick Java code review
  - `/python-review` - Quick Python code review
  - `/js-review` - Quick JavaScript/TypeScript code review
  - `/performance` - Frontend performance audit
  - `/smell-detect` - Code smell detection

### Changed
- **Updated agent orchestration rules** (`rules/agents.md`):
  - Added complete documentation for all 26 agents
  - Added language-specific agent usage guidelines
  - Added agent selection flowchart
  - Added agent priority matrix

- **Enhanced testing documentation** (`rules/testing.md`):
  - Added Java testing patterns (JUnit 5, AssertJ, Mockito)
  - Added Python testing patterns (pytest, fixtures)
  - Added Go testing patterns (table-driven tests)
  - Added JavaScript/TypeScript testing (Jest, Vitest)
  - Added Vue.js testing (Vue Test Utils)
  - Added language-specific test commands

- **Enhanced coding style documentation** (`rules/coding-style.md`):
  - Added Java coding patterns (records, sealed classes, Optional)
  - Added Python coding patterns (dataclasses, type hints, f-strings)
  - Added JavaScript/TypeScript patterns (TypeScript, async/await)
  - Added Vue.js patterns (Composition API)
  - Added Go patterns (interfaces, channels, error handling)

### Fixed
- All 14 new agents now properly registered in `.claude-plugin/plugin.json`
- Plugin version updated to 1.3.0
- Added comprehensive language support documentation

### Dependencies
- Updated keywords in package.json to include: java, python, javascript, typescript, vue, database, performance

## [1.2.0] - Previous Release

### Features
- Core agents: architect, planner, tdd-guide, code-reviewer, security-reviewer
- Build error resolvers for general builds
- Database reviewer (general)
- E2E test runner
- Refactoring cleaner
- Documentation updater
- Comprehensive skills library
- Hook system for workflow automation

---

## Agent Reference (v1.3.0)

### Language-Specific Agents (MANDATORY)
| Agent | Language | Use Case |
|-------|----------|----------|
| java-reviewer | Java | All Java projects |
| python-reviewer | Python | All Python projects |
| javascript-reviewer | JS/TS | All JavaScript/TypeScript projects |
| vue-reviewer | Vue.js | All Vue projects |
| go-reviewer | Go | All Go projects |

### Database Agents
| Agent | Database | Use Case |
|-------|----------|----------|
| db-postgresql-reviewer | PostgreSQL | PostgreSQL queries |
| db-mysql-reviewer | MySQL | MySQL queries |
| db-mongo-reviewer | MongoDB | MongoDB queries |
| db-oracle-reviewer | Oracle | Oracle/PLSQL |
| db-sqlserver-reviewer | SQL Server | T-SQL queries |

### Code Quality Agents
| Agent | Purpose | Use Case |
|-------|---------|----------|
| performance-auditor | Frontend performance | PROACTIVE use |
| smell-detector | Code smells | Refactoring sessions |

---

## Migration Guide (v1.2.0 → v1.3.0)

### For Java Projects
```bash
# Before: General code reviewer
/code-review

# After: Java-specific reviewer (MANDATORY)
/java-review
```

### For Python Projects
```bash
# Before: General code reviewer
/code-review

# After: Python-specific reviewer (MANDATORY)
/python-review
```

### For JavaScript/TypeScript Projects
```bash
# Before: General code reviewer
/code-review

# After: JS/TS-specific reviewer (MANDATORY)
/js-review
```

### For Performance Audits
```bash
# New command available
/performance
```

### For Code Smell Detection
```bash
# New command available
/smell-detect
```

---

[1.3.0]: https://github.com/affaan-m/everything-claude-code/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/affaan-m/everything-claude-code/releases/tag/v1.2.0
