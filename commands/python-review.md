---
description: Run Python code review using python-reviewer agent (MANDATORY for Python projects)
---

# Python Code Review

Use the **python-reviewer** agent to review Python code changes. This is MANDATORY for all Python projects.

The agent will:
1. Check PEP 8 compliance
2. Verify async/await patterns
3. Review type hints usage
4. Identify security vulnerabilities
5. Ensure idiomatic Python patterns
6. Check framework best practices (Django, FastAPI, Flask)
7. Review context manager usage

**When to use:**
- After writing/modifying any Python code
- Before committing Python changes
- During pull request review
- For refactoring Python codebases

**Key checks:**
- PEP 8 style guide compliance
- Type hints on function signatures
- Async/await antipatterns
- Error handling (bare except, exception chaining)
- Security (SQL injection, command injection, eval/exec, pickle)
- Code quality (long functions, deep nesting, magic numbers)
- Framework-specific patterns:
  - Django: N+1 queries, transaction management
  - FastAPI: dependency injection, response models
  - Flask: context locals, blueprints
