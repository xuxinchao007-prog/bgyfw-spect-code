---
description: Run Java code review using java-reviewer agent (MANDATORY for Java projects)
---

# Java Code Review

Use the **java-reviewer** agent to review Java code changes. This is MANDATORY for all Java projects.

The agent will:
1. Check for Java-specific issues (concurrency, exceptions, resources)
2. Verify Spring/Jakarta EE best practices
3. Identify security vulnerabilities
4. Ensure idiomatic Java patterns (Java 17+)
5. Check for proper use of records, sealed classes, Optional
6. Verify constructor injection over field injection
7. Review try-with-resources usage

**When to use:**
- After writing/modifying any Java code
- Before committing Java changes
- During pull request review
- For refactoring Java codebases

**Key checks:**
- Thread safety and concurrency patterns
- Exception handling (specific vs generic exceptions)
- Resource management (try-with-resources)
- Modern Java features (records, sealed classes, pattern matching)
- Spring/Jakarta EE best practices
- Security (SQL injection, command injection, path traversal)
- Code quality (long methods, god classes, code smells)
