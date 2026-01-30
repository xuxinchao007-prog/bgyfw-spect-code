# 插件项目适配变更建议

> **生成时间**: 2026-01-30
> **项目**: everything-claude-code v1.2.0
> **目的**: 基于新增的skills和agents进行整个插件项目的再适配

---

## 一、概述

本项目新增了多个语言特定的agents和skills，需要进行全面的配置适配，以确保新增功能能够被正确识别和使用。

### 新增内容统计

| 类型 | 新增数量 | 现有总数 | 需适配 |
|------|---------|---------|--------|
| **Agents** | 14个 | 26个 | 是 |
| **Skills** | 6个+ | 33个+ | 部分是 |
| **Commands** | 无新增 | 23个 | 否 |

---

## 二、详细变更点

### 1. 核心配置文件适配

#### 1.1 `.claude-plugin/plugin.json` 🔴 **CRITICAL**

**当前状态**: 仅引用12个agents，遗漏14个新增agents

**需要新增的Agents引用**:
```json
{
  "agents": [
    // === 现有agents (保留) ===
    "./agents/architect.md",
    "./agents/build-error-resolver.md",
    "./agents/code-reviewer.md",
    "./agents/database-reviewer.md",
    "./agents/doc-updater.md",
    "./agents/e2e-runner.md",
    "./agents/go-build-resolver.md",
    "./agents/go-reviewer.md",
    "./agents/planner.md",
    "./agents/refactor-cleaner.md",
    "./agents/security-reviewer.md",
    "./agents/tdd-guide.md",

    // === 新增agents (需要添加) ===
    "./agents/java-build-resolver.md",
    "./agents/java-reviewer.md",
    "./agents/python-build-resolver.md",
    "./agents/python-reviewer.md",
    "./agents/javascript-build-resolver.md",
    "./agents/javascript-reviewer.md",
    "./agents/vue-reviewer.md",
    "./agents/performance-auditor.md",
    "./agents/smell-detector.md",
    "./agents/refactor-cleaner-java.md",
    "./agents/db-mysql-reviewer.md",
    "./agents/db-sqlserver-reviewer.md",
    "./agents/db-oracle-reviewer.md",
    "./agents/db-mongo-reviewer.md",
    "./agents/db-postgresql-reviewer.md"
  ]
}
```

**变更影响**:
- 新agents将无法被系统识别和使用
- 语言特定的代码审查功能无法生效

**优先级**: 🔴 CRITICAL - 必须立即修复

---

#### 1.2 版本号更新

**建议更新版本**:
```json
{
  "version": "1.3.0",  // 从 1.2.0 升级
  "description": "Complete collection of battle-tested Claude Code configs - agents, skills, hooks, and rules evolved over 10+ months of intensive daily use. Now with comprehensive language-specific agents for Java, Python, JavaScript, Vue and database specialists for PostgreSQL, MySQL, MongoDB, Oracle, SQL Server."
}
```

---

### 2. Rules目录适配

#### 2.1 `rules/agents.md` 🟡 **HIGH**

**当前状态**: 仅列出8个agents，未包含新增的14个agents

**需要新增的Agent描述**:

```markdown
## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| // === 现有agents (保留) ===
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| tdd-guide | Test-driven development | New features, bug fixes |
| code-reviewer | Code review | After writing code |
| security-reviewer | Security analysis | Before commits |
| build-error-resolver | Fix build errors | When build fails |
| e2e-runner | E2E testing | Critical user flows |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation | Updating docs |
| go-reviewer | Go code review | Go projects |
| go-build-resolver | Go build issues | Go build failures |

| // === 新增agents (需要添加) ===
| java-reviewer | Java code review | Java projects - MUST BE USED |
| java-build-resolver | Java build issues | Maven/Gradle build failures |
| python-reviewer | Python code review | Python projects - MUST BE USED |
| python-build-resolver | Python build issues | pip/pipenv/poetry failures |
| javascript-reviewer | JavaScript/TypeScript review | JS/TS projects - MUST BE USED |
| javascript-build-resolver | JavaScript build issues | npm/yarn/pnpm build failures |
| vue-reviewer | Vue.js code review | Vue projects - MUST BE USED |
| performance-auditor | Performance analysis | Frontend optimization |
| smell-detector | Code smell detection | Refactoring sessions |
| refactor-cleaner-java | Java code cleanup | Java dead code removal |
| db-postgresql-reviewer | PostgreSQL review | PostgreSQL queries |
| db-mysql-reviewer | MySQL review | MySQL queries |
| db-oracle-reviewer | Oracle review | Oracle queries |
| db-sqlserver-reviewer | SQL Server review | SQL Server queries |
| db-mongo-reviewer | MongoDB review | MongoDB queries |
```

**新增使用场景**:

```markdown
## Language-Specific Agent Usage

### Java Projects
- **MANDATORY**: Use `java-reviewer` for all Java code changes
- Use `java-build-resolver` when Maven/Gradle build fails
- Use `refactor-cleaner-java` for Java-specific code cleanup

### Python Projects
- **MANDATORY**: Use `python-reviewer` for all Python code changes
- Use `python-build-resolver` when pip/pipenv/poetry fails

### JavaScript/TypeScript Projects
- **MANDATORY**: Use `javascript-reviewer` for all JS/TS code changes
- Use `javascript-build-resolver` when npm/yarn/pnpm build fails

### Vue.js Projects
- **MANDATORY**: Use `vue-reviewer` for all Vue code changes

### Database Projects
- Use appropriate database reviewer based on your DB:
  - PostgreSQL → `db-postgresql-reviewer`
  - MySQL → `db-mysql-reviewer`
  - MongoDB → `db-mongo-reviewer`
  - Oracle → `db-oracle-reviewer`
  - SQL Server → `db-sqlserver-reviewer`

### Code Quality
- Use `smell-detector` during refactoring sessions
- Use `performance-auditor` when developing frontend features
```

---

#### 2.2 `rules/coding-style.md` 🟢 **MEDIUM**

**建议新增语言特定的编码规范章节**:

```markdown
## Language-Specific Guidelines

### Java (See: java-patterns skill)
- Use records for immutable data carriers (Java 16+)
- Use sealed classes for restricted inheritance (Java 17+)
- Prefer virtual threads for I/O-bound tasks (Java 21+)
- Use Optional for return values, not for parameters
- Constructor injection over field injection
- Try-with-resources for AutoCloseable resources

### Python (See: python-patterns skill)
- Follow PEP 8 style guide
- Use dataclasses for data containers
- Use type hints for function signatures
- Prefer context managers (with statements)
- Use pathlib instead of os.path
- F-strings for string formatting

### JavaScript/TypeScript (See: javascript-patterns skill)
- Use TypeScript for type safety
- Prefer composition over inheritance
- Immutability patterns (spread operator)
- Async/await over promises
- Proper error boundaries
```

---

#### 2.3 `rules/testing.md` 🟡 **HIGH**

**建议新增语言特定的测试指南**:

```markdown
## Language-Specific Testing

### Java Testing (See: java-testing skill)
- Use JUnit 5 for unit tests
- Use AssertJ for assertions
- Use Mockito for mocking
- Table-driven tests for multiple scenarios
- Test coverage: 80%+

### Python Testing (See: python-testing skill)
- Use pytest for testing
- Use pytest-cov for coverage
- Use pytest-mock for mocking
- Parametrize tests for multiple scenarios
- Test coverage: 80%+

### Go Testing (See: golang-testing skill)
- Use testing package for unit tests
- Table-driven tests are idiomatic
- Use testify for assertions
- Test coverage: 80%+
```

---

#### 2.4 `rules/security.md` 🟢 **LOW**

**建议新增数据库特定的安全检查**:

```markdown
## Database Security

### SQL Injection Prevention
- **CRITICAL**: Always use parameterized queries
- Never concatenate user input into SQL
- Use ORM query builders when possible

### NoSQL Injection Prevention
- Validate all query inputs
- Use sanitized operators
- Avoid $where clauses with user input
```

---

### 3. 新增Commands (可选)

考虑到新增了多个语言特定的agents，建议创建对应的快捷commands:

#### 3.1 建议新增的Commands

| Command File | Purpose | Content |
|-------------|---------|---------|
| `commands/java-review.md` | Java代码审查 | 调用java-reviewer agent |
| `commands/python-review.md` | Python代码审查 | 调用python-reviewer agent |
| `commands/js-review.md` | JavaScript代码审查 | 调用javascript-reviewer agent |
| `commands/performance.md` | 性能审计 | 调用performance-auditor agent |
| `commands/smell-detect.md` | 代码异味检测 | 调用smell-detector agent |

**示例**: `commands/java-review.md`
```markdown
---
description: Run Java code review using java-reviewer agent
---

Use the **java-reviewer** agent to review Java code changes. This is MANDATORY for all Java projects.

The agent will:
1. Check for Java-specific issues (concurrency, exceptions, resources)
2. Verify Spring/Jakarta EE best practices
3. Identify security vulnerabilities
4. Ensure idiomatic Java patterns
```

---

### 4. Skills目录适配 🟢 **LOW**

Skills目录使用通配符引用，大部分新增技能应该能被自动识别。但建议验证以下skills是否正确配置:

#### 4.1 需要验证的Skills

| Skill | Path | Status |
|-------|------|--------|
| java-patterns | `skills/java-patterns/SKILL.md` | ✅ 需验证 |
| java-testing | `skills/java-testing/SKILL.md` | ✅ 需验证 |
| mongodb-patterns | `skills/mongodb-patterns/SKILL.md` | ✅ 需验证 |
| mysql-patterns | `skills/mysql-patterns/SKILL.md` | ✅ 需验证 |
| redis-patterns | `skills/redis-patterns/SKILL.md` | ✅ 需验证 |
| influxdb-patterns | `skills/influxdb-patterns/SKILL.md` | ✅ 需验证 |
| security-review | `skills/security-review/SKILL.md` | ✅ 需验证 |
| iterative-retrieval | `skills/iterative-retrieval/SKILL.md` | ✅ 需验证 |
| eval-harness | `skills/eval-harness/SKILL.md` | ✅ 需验证 |
| continuous-learning-v2 | `skills/continuous-learning-v2/SKILL.md` | ✅ 需验证 |

---

### 5. MCP配置适配 🟢 **LOW**

检查 `mcp-configs/mcp-servers.json` 是否需要添加数据库相关的MCP服务器配置。

**建议新增配置** (如果不存在):
```json
{
  "mcpServers": {
    // ... existing servers ...

    "postgres-inspector": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/inspector-postgres"]
    },
    "mongodb-inspector": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/inspector-mongodb"]
    }
  }
}
```

---

## 三、优先级排序

### 🔴 CRITICAL (必须立即修复)

1. **`.claude-plugin/plugin.json`** - 添加14个新增agents的引用
   - 影响: 新agents无法被识别和使用
   - 工作量: 5分钟

### 🟡 HIGH (建议尽快修复)

2. **`rules/agents.md`** - 添加新增agents的文档说明
   - 影响: 用户不知道如何使用新agents
   - 工作量: 15分钟

3. **`rules/testing.md`** - 添加语言特定的测试指南
   - 影响: 测试指导不完整
   - 工作量: 10分钟

### 🟢 MEDIUM (可以在后续版本添加)

4. **`rules/coding-style.md`** - 添加语言特定的编码规范
   - 影响: 编码规范不够细化
   - 工作量: 10分钟

5. **新增Commands** - 创建语言特定的快捷命令
   - 影响: 用户体验优化
   - 工作量: 30分钟

### ⚪ LOW (可选)

6. **Skills验证** - 确认所有skills能被正确识别
   - 影响: 部分skills可能不可用
   - 工作量: 5分钟

7. **MCP配置** - 添加数据库相关的MCP服务器
   - 影响: 数据库辅助功能
   - 工作量: 5分钟

---

## 四、实施建议

### 实施顺序

```
┌─────────────────────────────────────────────────────────┐
│  Phase 1: CRITICAL Fixes (立即实施)                      │
├─────────────────────────────────────────────────────────┤
│  1. 更新 .claude-plugin/plugin.json                      │
│  2. 验证所有agents能被正确识别                           │
│  3. 测试新增agents的功能                                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  Phase 2: Documentation Updates (本周内完成)             │
├─────────────────────────────────────────────────────────┤
│  4. 更新 rules/agents.md                                 │
│  5. 更新 rules/testing.md                                │
│  6. 更新 rules/coding-style.md                           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  Phase 3: Enhancements (下个版本)                        │
├─────────────────────────────────────────────────────────┤
│  7. 创建新的commands                                     │
│  8. 验证skills配置                                       │
│  9. 更新MCP配置                                          │
│  10. 更新版本号到 v1.3.0                                 │
└─────────────────────────────────────────────────────────┘
```

### 测试验证

完成适配后，建议进行以下测试:

```bash
# 1. 验证plugin.json格式
cat .claude-plugin/plugin.json | jq .

# 2. 验证所有agents文件存在
for agent in $(cat .claude-plugin/plugin.json | jq -r '.agents[]'); do
  if [ -f "$agent" ]; then
    echo "✅ $agent"
  else
    echo "❌ $agent NOT FOUND"
  fi
done

# 3. 验证所有skills目录存在
for skill_dir in skills/*/; do
  if [ -f "${skill_dir}SKILL.md" ]; then
    echo "✅ ${skill_dir}"
  else
    echo "⚠️  ${skill_dir} missing SKILL.md"
  fi
done
```

---

## 五、新增Agents功能矩阵

| Agent | 语言/领域 | 核心功能 | 优先级 |
|-------|---------|---------|--------|
| java-reviewer | Java | 代码审查、并发、异常处理、Spring | 🔴 CRITICAL |
| java-build-resolver | Java | Maven/Gradle构建错误修复 | 🟡 HIGH |
| python-reviewer | Python | PEP 8、异步、类型提示、Django | 🔴 CRITICAL |
| python-build-resolver | Python | pip/pipenv/poetry错误修复 | 🟡 HIGH |
| javascript-reviewer | JS/TS | ES6+、React、异步、类型 | 🔴 CRITICAL |
| javascript-build-resolver | JS/TS | npm/yarn/pnpm构建错误 | 🟡 HIGH |
| vue-reviewer | Vue.js | Composition API、响应式、性能 | 🟡 HIGH |
| performance-auditor | Frontend | Lighthouse、Core Web Vitals | 🟡 HIGH |
| smell-detector | All | 代码异味检测、重构建议 | 🟡 HIGH |
| refactor-cleaner-java | Java | Java死代码清理 | 🟢 MEDIUM |
| db-postgresql-reviewer | PostgreSQL | 查询优化、索引、安全 | 🟢 MEDIUM |
| db-mysql-reviewer | MySQL | 查询优化、索引、安全 | 🟢 MEDIUM |
| db-oracle-reviewer | Oracle | PL/SQL、优化、安全 | 🟢 MEDIUM |
| db-sqlserver-reviewer | SQL Server | T-SQL、优化、安全 | 🟢 MEDIUM |
| db-mongo-reviewer | MongoDB | 聚合、索引、安全 | 🟢 MEDIUM |

---

## 六、变更检查清单

### 立即执行 (CRITICAL)

- [ ] 更新 `.claude-plugin/plugin.json` 添加14个新agents
- [ ] 验证plugin.json格式正确
- [ ] 测试agents能被正确调用

### 本周完成 (HIGH)

- [ ] 更新 `rules/agents.md` 添加新agents说明
- [ ] 更新 `rules/testing.md` 添加语言特定测试指南
- [ ] 验证所有skills目录包含SKILL.md
- [ ] 更新README.md中的agents列表

### 后续版本 (MEDIUM/LOW)

- [ ] 更新 `rules/coding-style.md` 添加语言特定规范
- [ ] 创建新的快捷commands
- [ ] 更新 `mcp-configs/mcp-servers.json`
- [ ] 更新版本号到 v1.3.0
- [ ] 编写CHANGELOG.md

---

## 七、版本发布建议

### v1.3.0 (当前适配版本)

```markdown
## [1.3.0] - 2026-01-XX

### Added
- 14 new language-specific agents for comprehensive code review
- Java build resolver and code reviewer
- Python build resolver and code reviewer
- JavaScript/TypeScript build resolver and code reviewer
- Vue.js specialist code reviewer
- Performance auditor for frontend optimization
- Code smell detector for refactoring guidance
- 5 database-specific reviewers (PostgreSQL, MySQL, MongoDB, Oracle, SQL Server)
- Language-specific testing guidelines
- Enhanced coding standards for Java, Python, JavaScript

### Changed
- Updated agent orchestration rules to include 26 total agents
- Enhanced testing documentation with language-specific patterns
- Improved coding style guidelines with multi-language support

### Fixed
- All 14 new agents now properly registered in plugin.json
- Skills directory now correctly indexes all language patterns
```

---

## 八、总结

本次适配主要解决了以下问题：

1. **注册遗漏**: 14个新增agents未被plugin.json引用
2. **文档缺失**: rules/agents.md未包含新agents的说明
3. **指南不完整**: testing.md缺少语言特定的测试指南
4. **规范不细化**: coding-style.md缺少语言特定的编码规范

**预计工作量**: 1-2小时
**风险等级**: 低
**向后兼容**: 是

---

**文档生成者**: Claude (glm-4.7)
**审核状态**: 待审核
**下一步**: 执行Phase 1: CRITICAL Fixes
