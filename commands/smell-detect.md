---
description: Detect code smells and suggest refactoring using smell-detector agent
---

# Code Smell Detection

Use the **smell-detector** agent to identify code smells and suggest refactoring strategies. Use during code reviews, refactoring sessions, or when technical debt accumulates.

The agent will:
1. Identify code smell categories (Bloaters, OO Abusers, Change Preventers, etc.)
2. Detect specific smells (Long Method, God Class, Feature Envy, etc.)
3. Assess severity and technical debt impact
4. Provide specific refactoring recommendations
5. Generate prioritized action plan

**When to use:**
- During code reviews
- Before major refactoring efforts
- When technical debt is high
- For regular code quality checks
- When onboarding to legacy codebases

**Code smell categories:**

**Bloaters:**
- Long Method (>50 lines)
- God Class (>500 lines, >20 methods)
- Large Class
- Primitive Obsession
- Long Parameter List
- Data Clumps

**Object-Orientation Abusers:**
- Switch Statements
- Temporary Field
- Refused Bequest
- Alternative Classes with Different Interfaces
- Inappropriate Intimacy

**Change Preventers:**
- Divergent Change
- Parallel Inheritance Hierarchies
- Shotgun Surgery

**Dispensables:**
- Comments (explaining "what" not "why")
- Code Duplication
- Lazy Class
- Data Class
- Dead Code
- Speculative Generality

**Couplers:**
- Feature Envy
- Inappropriate Intimacy
- Message Chains
- Middle Man

**Common refactoring techniques:**
- Extract Method/Class/Interface
- Move Method
- Replace Conditional with Polymorphism
- Extract Value Object
- Hide Delegate
- Replace Magic Number with Constant
