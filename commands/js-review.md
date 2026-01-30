---
description: Run JavaScript/TypeScript code review using javascript-reviewer agent (MANDATORY for JS/TS projects)
---

# JavaScript/TypeScript Code Review

Use the **javascript-reviewer** agent to review JavaScript/TypeScript code changes. This is MANDATORY for all JS/TS projects.

The agent will:
1. Check modern ES6+ patterns
2. Verify TypeScript type safety
3. Review async/await usage
4. Identify security vulnerabilities
5. Ensure idiomatic JavaScript patterns
6. Check framework best practices (React, Vue, Angular)
7. Review bundle optimization

**When to use:**
- After writing/modifying any JavaScript/TypeScript code
- Before committing JS/TS changes
- During pull request review
- For refactoring JavaScript codebases

**Key checks:**
- TypeScript: type safety, no 'any' types, proper interfaces
- Modern patterns: async/await, optional chaining, nullish coalescing
- Immutability: spread operator, Object.freeze
- Security: XSS, eval, hardcoded secrets
- Code quality: long functions, deep nesting, console.log
- Framework-specific patterns:
  - React: hooks, component design, performance
  - Vue: composition API, reactivity, props/emits
  - Angular: services, dependency injection, observables
- Bundle optimization: tree shaking, code splitting, lazy loading
