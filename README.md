**Language:** English | [繁體中文](docs/zh-TW/README.md) | [简体中文](README.zh-CN.md)

# BGYFW Spect Code

[![Stars](https://img.shields.io/github/stars/xuxinchao007-prog/bgyfw-spect-code?style=flat)](https://github.com/xuxinchao007-prog/bgyfw-spect-code/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

<p align="left">
  <span>English</span> |
  <a href="README.zh-CN.md">简体中文</a>
</p>

**A complete collection of Claude Code configurations with 27 specialized agents.**

Production-ready agents, skills, hooks, commands, rules, and MCP configurations including language-specific reviewers for Java, Python, JavaScript, Vue and database specialists for PostgreSQL, MySQL, MongoDB, Oracle, SQL Server.

---

## Features

### Core Agents (8)
- **architect**: System design and architecture decisions
- **planner**: Feature implementation planning
- **code-reviewer**: General code quality and security review
- **security-reviewer**: Vulnerability analysis (OWASP Top 10)
- **tdd-guide**: Test-driven development workflow
- **e2e-runner**: Playwright E2E testing
- **refactor-cleaner**: Dead code cleanup
- **doc-updater**: Documentation and codemap updates

### Language-Specific Agents (10)
- **java-reviewer**: Expert Java code review (concurrency, Spring, Jakarta)
- **java-build-resolver**: Java Maven/Gradle build fixes
- **python-reviewer**: Expert Python code review (PEP 8, async, Django/FastAPI)
- **python-build-resolver**: Python pip/build dependency fixes
- **javascript-reviewer**: JavaScript/TypeScript review (ES6+, React, Vue)
- **javascript-build-resolver**: JS/TS npm/yarn/pnpm bundler fixes
- **vue-reviewer**: Vue.js specialist (Composition API, reactivity)
- **go-reviewer**: Go code review (idiomatic patterns, goroutines)
- **go-build-resolver**: Go build, vet, and linter fixes
- **refactor-cleaner-java**: Java dead code cleanup (SpotBugs, PMD)

### Database Specialists (6)
- **database-reviewer**: General database review
- **db-postgresql-reviewer**: PostgreSQL query optimization
- **db-mysql-reviewer**: MySQL 8.0+ query optimization
- **db-mongo-reviewer**: MongoDB aggregation and indexing
- **db-oracle-reviewer**: Oracle/PLSQL optimization
- **db-sqlserver-reviewer**: SQL Server T-SQL optimization

### Code Quality Agents (3)
- **performance-auditor**: Frontend performance (Lighthouse, Core Web Vitals)
- **smell-detector**: Code smell detection and refactoring guidance
- **build-error-resolver**: General build error resolution

---

## Quick Start

### Installation

```bash
# Add this repo as a marketplace
/plugin marketplace add xuxinchao007-prog/bgyfw-spect-code

# Install the plugin
/plugin install bgyfw-spect-code@bgyfw-spect-code
```

### Manual Setup

```bash
# Clone the repo
git clone https://github.com/xuxinchao007-prog/bgyfw-spect-code.git

# Copy rules
cp -r bgyfw-spect-code/rules/* ~/.claude/rules/
```

---

## Documentation

See the original guides for detailed information:

- [Shorthand Guide](https://x.com/affaanmustafa/status/2012378465664745795) - Setup, foundations, philosophy
- [Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - Token optimization, memory persistence, evals

---

## What's Inside

| Topic | What You'll Learn |
|-------|-------------------|
| Token Optimization | Model selection, system prompt slimming, background processes |
| Memory Persistence | Hooks that save/load context across sessions automatically |
| Continuous Learning | Auto-extract patterns from sessions into reusable skills |
| Verification Loops | Checkpoint vs continuous evals, grader types, pass@k metrics |
| Parallelization | Git worktrees, cascade method, when to scale instances |
| Subagent Orchestration | The context problem, iterative retrieval pattern |

---

## Cross-Platform Support

This plugin now fully supports **Windows, macOS, and Linux**. All hooks and scripts have been rewritten in Node.js for maximum compatibility.

### Package Manager Detection

The plugin automatically detects your preferred package manager (npm, pnpm, yarn, or bun) with the following priority:

1. **Environment variable**: `CLAUDE_PACKAGE_MANAGER`
2. **Project config**: `.claude/package-manager.json`
3. **package.json**: `packageManager` field
4. **Lock file**: Detection from package-lock.json, yarn.lock, pnpm-lock.yaml, or bun.lockb
5. **Global config**: `~/.claude/package-manager.json`
6. **Fallback**: First available package manager

To set your preferred package manager:

```bash
# Via environment variable
export CLAUDE_PACKAGE_MANAGER=pnpm

# Via global config
node scripts/setup-package-manager.js --global pnpm

# Via project config
node scripts/setup-package-manager.js --project bun

# Detect current setting
node scripts/setup-package-manager.js --detect
```

Or use the `/setup-pm` command in Claude Code.

---

## What's Inside

This repo is a **Claude Code plugin** - install it directly or copy components manually.

```
bgyfw-spect-code/
|-- .claude-plugin/   # Plugin and marketplace manifests
|   |-- plugin.json         # Plugin metadata and component paths
|   |-- marketplace.json    # Marketplace catalog for /plugin marketplace add
|
|-- agents/           # 27 specialized subagents for delegation
|   |-- architect.md              # System design and architecture decisions
|   |-- planner.md                # Feature implementation planning
|   |-- code-reviewer.md          # General code quality review
|   |-- security-reviewer.md      # Vulnerability analysis (OWASP Top 10)
|   |-- tdd-guide.md              # Test-driven development workflow
|   |-- build-error-resolver.md   # General build error resolution
|   |-- e2e-runner.md             # Playwright E2E testing
|   |-- refactor-cleaner.md       # Dead code cleanup
|   |-- refactor-cleaner-java.md  # Java dead code cleanup
|   |-- doc-updater.md            # Documentation and codemap updates
|   |-- smell-detector.md         # Code smell detection
|   |-- database-reviewer.md      # General database review
|   |
|   |-- Language-Specific Reviewers:
|   |   |-- java-reviewer.md           # Java (concurrency, Spring, Jakarta)
|   |   |-- java-build-resolver.md     # Java Maven/Gradile build fixes
|   |   |-- python-reviewer.md         # Python (PEP 8, async, Django/FastAPI)
|   |   |-- python-build-resolver.md   # Python pip/build fixes
|   |   |-- javascript-reviewer.md     # JavaScript/TypeScript (ES6+, React)
|   |   |-- javascript-build-resolver.md# JS/TS npm/bundler build fixes
|   |   |-- vue-reviewer.md            # Vue.js (Composition API, reactivity)
|   |   |-- go-reviewer.md             # Go (idiomatic patterns, goroutines)
|   |   |-- go-build-resolver.md       # Go build/vet/linter fixes
|   |
|   |-- Database Specialists:
|       |-- db-postgresql-reviewer.md  # PostgreSQL optimization
|       |-- db-mysql-reviewer.md       # MySQL 8.0+ optimization
|       |-- db-mongo-reviewer.md       # MongoDB aggregation and indexing
|       |-- db-oracle-reviewer.md      # Oracle/PLSQL optimization
|       |-- db-sqlserver-reviewer.md   # SQL Server T-SQL optimization
|
|-- skills/           # Workflow definitions and domain knowledge
|   |-- brainstorming/              # MUST use before creative work
|   |-- coding-standards/           # Universal coding best practices
|   |-- backend-patterns/           # API, database, caching patterns
|   |-- frontend-patterns/          # React/Next.js and Vue 3/Nuxt.js patterns
|   |-- continuous-learning/        # Auto-extract patterns from sessions
|   |-- continuous-learning-v2/     # Instinct-based learning with confidence
|   |-- iterative-retrieval/        # Progressive context refinement
|   |-- strategic-compact/          # Manual compaction suggestions
|   |-- tdd-workflow/               # TDD methodology
|   |-- security-review/            # Security checklist
|   |-- eval-harness/               # Verification loop evaluation
|   |-- verification-loop/          # Continuous verification
|   |-- finishing-a-development-branch/ # Development completion workflow
|   |-- using-git-worktrees/        # Git worktree isolation
|   |
|   |-- Language Patterns:
|   |   |-- java-patterns/          # Java 17+ idioms and best practices
|   |   |-- java-testing/           # JUnit 5, AssertJ, Mockito patterns
|   |   |-- golang-patterns/        # Go idioms and best practices
|   |   |-- golang-testing/         # Go TDD, benchmarks, fuzzing
|   |   |-- python-patterns/        # Python best practices (via python-review)
|   |   |-- javascript-patterns/    # JS/TS patterns (via js-review)
|   |
|   |-- Database Patterns:
|       |-- postgres-patterns/      # PostgreSQL optimization
|       |-- mysql-patterns/         # MySQL 8.0+ optimization
|       |-- mongodb-patterns/       # MongoDB query optimization
|       |-- redis-patterns/         # Redis caching patterns
|       |-- influxdb-patterns/      # InfluxDB time-series patterns
|       |-- clickhouse-io/          # ClickHouse analytics patterns
|
|-- commands/         # 28 slash commands for quick execution
|   |-- plan.md                # /plan - Implementation planning
|   |-- tdd.md                 # /tdd - Test-driven development
|   |-- code-review.md         # /code-review - Quality review
|   |-- build-fix.md           # /build-fix - Fix build errors
|   |-- refactor-clean.md      # /refactor-clean - Dead code removal
|   |-- e2e.md                 # /e2e - E2E test generation
|   |-- smell-detect.md        # /smell-detect - Code smell detection
|   |-- performance.md         # /performance - Frontend performance audit
|   |-- learn.md               # /learn - Extract patterns mid-session
|   |-- checkpoint.md          # /checkpoint - Save verification state
|   |-- verify.md              # /verify - Run verification loop
|   |-- eval.md                # /eval - Evaluation harness
|   |-- test-coverage.md       # /test-coverage - Coverage check
|   |-- orchestrate.md         # /orchestrate - Parallel agent dispatch
|   |-- setup-pm.md            # /setup-pm - Configure package manager
|   |
|   |-- Language Reviews:
|   |   |-- java-review.md     # /java-review - Java code review
|   |   |-- python-review.md   # /python-review - Python code review
|   |   |-- js-review.md       # /js-review - JavaScript/TypeScript review
|   |   |-- go-review.md       # /go-review - Go code review
|   |   |-- go-test.md         # /go-test - Go TDD workflow
|   |   |-- go-build.md        # /go-build - Fix Go build errors
|   |
|   |-- Continuous Learning:
|       |-- skill-create.md        # /skill-create - Generate skills
|       |-- instinct-status.md     # /instinct-status - View instincts
|       |-- instinct-import.md     # /instinct-import - Import instincts
|       |-- instinct-export.md     # /instinct-export - Export instincts
|       |-- evolve.md              # /evolve - Cluster instincts
|       |-- update-codemaps.md     # /update-codemaps - Update codemaps
|       |-- update-docs.md         # /update-docs - Update documentation
|
|-- rules/            # Always-follow guidelines (copy to ~/.claude/rules/)
|   |-- agents.md           # When to delegate to subagents
|   |-- security.md         # Mandatory security checks
|   |-- coding-style.md     # Immutability, file organization
|   |-- testing.md          # TDD, 80% coverage requirement
|   |-- git-workflow.md     # Commit format, PR process
|   |-- hooks.md            # Hook configuration guidelines
|   |-- patterns.md         # Pattern extraction guidelines
|   |-- performance.md      # Model selection, context management
|
|-- hooks/            # Trigger-based automations
|   |-- hooks.json                # All hooks config
|   |-- memory-persistence/       # Session lifecycle hooks
|   |-- strategic-compact/        # Compaction suggestions
|
|-- scripts/          # Cross-platform Node.js scripts
|   |-- lib/                     # Shared utilities
|   |-- hooks/                   # Hook implementations
|   |-- setup-package-manager.js # Package manager setup
|
|-- tests/            # Test suite
|   |-- lib/                     # Library tests
|   |-- hooks/                   # Hook tests
|   |-- run-all.js               # Run all tests
|
|-- contexts/         # Dynamic system prompt injection contexts
|   |-- dev.md              # Development mode context
|   |-- review.md           # Code review mode context
|   |-- research.md         # Research/exploration mode context
|
|-- examples/         # Example configurations
|   |-- CLAUDE.md           # Example project-level config
|
|-- mcp-configs/      # MCP server configurations
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, etc.
|
|-- marketplace.json  # Self-hosted marketplace config
```

---

## Ecosystem Tools

### Skill Creator

Two ways to generate Claude Code skills from your repository:

#### Option A: Local Analysis (Built-in)

Use the `/skill-create` command for local analysis without external services:

```bash
/skill-create                    # Analyze current repo
/skill-create --instincts        # Also generate instincts for continuous-learning
```

This analyzes your git history locally and generates SKILL.md files.

#### Option B: GitHub App (Advanced)

For advanced features (10k+ commits, auto-PRs, team sharing):

[Install GitHub App](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# Comment on any issue:
/skill-creator analyze

# Or auto-triggers on push to default branch
```

Both options create:
- **SKILL.md files** - Ready-to-use skills for Claude Code
- **Instinct collections** - For continuous-learning-v2
- **Pattern extraction** - Learns from your commit history

### Continuous Learning v2

The instinct-based learning system automatically learns your patterns:

```bash
/instinct-status        # Show learned instincts with confidence
/instinct-import <file> # Import instincts from others
/instinct-export        # Export your instincts for sharing
/evolve                 # Cluster related instincts into skills
```

See `skills/continuous-learning-v2/` for full documentation.

---

## Requirements

### Claude Code CLI Version

**Minimum version: v2.1.0 or later**

This plugin requires Claude Code CLI v2.1.0+ due to changes in how the plugin system handles hooks.

Check your version:
```bash
claude --version
```

### Important: Hooks Auto-Loading Behavior

> ⚠️ **For Contributors:** Do NOT add a `"hooks"` field to `.claude-plugin/plugin.json`. This is enforced by a regression test.

Claude Code v2.1+ **automatically loads** `hooks/hooks.json` from any installed plugin by convention. Explicitly declaring it in `plugin.json` causes a duplicate detection error:

```
Duplicate hooks file detected: ./hooks/hooks.json resolves to already-loaded file
```

**History:** This has caused repeated fix/revert cycles in this repo ([#29](https://github.com/affaan-m/everything-claude-code/issues/29), [#52](https://github.com/affaan-m/everything-claude-code/issues/52), [#103](https://github.com/affaan-m/everything-claude-code/issues/103)). The behavior changed between Claude Code versions, leading to confusion. We now have a regression test to prevent this from being reintroduced.

---

## Installation

### Option 1: Install as Plugin (Recommended)

The easiest way to use this repo - install as a Claude Code plugin:

```bash
# Add this repo as a marketplace
/plugin marketplace add xuxinchao007-prog/bgyfw-spect-code

# Install the plugin
/plugin install bgyfw-spect-code@bgyfw-spect-code
```

Or add directly to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "bgyfw-spect-code": {
      "source": {
        "source": "github",
        "repo": "xuxinchao007-prog/bgyfw-spect-code"
      }
    }
  },
  "enabledPlugins": {
    "bgyfw-spect-code@bgyfw-spect-code": true
  }
}
```

This gives you instant access to all commands, agents, skills, and hooks.

> **Note:** The Claude Code plugin system does not support distributing `rules` via plugins ([upstream limitation](https://code.claude.com/docs/en/plugins-reference)). You need to install rules manually:
>
> ```bash
> # Clone the repo first
> git clone https://github.com/xuxinchao007-prog/bgyfw-spect-code.git
>
> # Option A: User-level rules (applies to all projects)
> cp -r bgyfw-spect-code/rules/* ~/.claude/rules/
>
> # Option B: Project-level rules (applies to current project only)
> mkdir -p .claude/rules
> cp -r bgyfw-spect-code/rules/* .claude/rules/
> ```

---

### Option 2: Manual Installation

If you prefer manual control over what's installed:

```bash
# Clone the repo
git clone https://github.com/xuxinchao007-prog/bgyfw-spect-code.git

# Copy agents to your Claude config
cp bgyfw-spect-code/agents/*.md ~/.claude/agents/

# Copy rules
cp bgyfw-spect-code/rules/*.md ~/.claude/rules/

# Copy commands
cp bgyfw-spect-code/commands/*.md ~/.claude/commands/

# Copy skills
cp -r bgyfw-spect-code/skills/* ~/.claude/skills/
```

#### Add hooks to settings.json

Copy the hooks from `hooks/hooks.json` to your `~/.claude/settings.json`.

#### Configure MCPs

Copy desired MCP servers from `mcp-configs/mcp-servers.json` to your `~/.claude.json`.

**Important:** Replace `YOUR_*_HERE` placeholders with your actual API keys.

---

## Key Concepts

### Agents

Subagents handle delegated tasks with limited scope. Example:

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

You are a senior code reviewer...
```

### Skills

Skills are workflow definitions invoked by commands or agents:

```markdown
# TDD Workflow

1. Define interfaces first
2. Write failing tests (RED)
3. Implement minimal code (GREEN)
4. Refactor (IMPROVE)
5. Verify 80%+ coverage
```

### Hooks

Hooks fire on tool events. Example - warn about console.log:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### Rules

Rules are always-follow guidelines. Keep them modular:

```
~/.claude/rules/
  security.md      # No hardcoded secrets
  coding-style.md  # Immutability, file limits
  testing.md       # TDD, coverage requirements
```

---

## Running Tests

The plugin includes a comprehensive test suite:

```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## Contributing

**Contributions are welcome and encouraged.**

This repo is meant to be a community resource. If you have:
- Useful agents or skills
- Clever hooks
- Better MCP configurations
- Improved rules

Please contribute! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Ideas for Contributions

- Language-specific skills (Python, Rust patterns) - Go now included!
- Framework-specific configs (Django, Rails, Laravel)
- DevOps agents (Kubernetes, Terraform, AWS)
- Testing strategies (different frameworks)
- Domain-specific knowledge (ML, data engineering, mobile)

---

## Background

This project is based on the excellent work by [Affaan Mustafa](https://github.com/affaan-m) and has been adapted and extended with additional language-specific agents and database specialists.

The original configurations are battle-tested across multiple production applications.

---

## Important Notes

### Context Window Management

**Critical:** Don't enable all MCPs at once. Your 200k context window can shrink to 70k with too many tools enabled.

Rule of thumb:
- Have 20-30 MCPs configured
- Keep under 10 enabled per project
- Under 80 tools active

Use `disabledMcpServers` in project config to disable unused ones.

### Customization

These configs work for my workflow. You should:
1. Start with what resonates
2. Modify for your stack
3. Remove what you don't use
4. Add your own patterns

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=xuxinchao007-prog/bgyfw-spect-code&type=Date)](https://star-history.com/#xuxinchao007-prog/bgyfw-spect-code&Date)

---

## Links

- **Original Project:** [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)
- **Shorthand Guide (Start Here):** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **Longform Guide (Advanced):** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **Repository:** [xuxinchao007-prog/bgyfw-spect-code](https://github.com/xuxinchao007-prog/bgyfw-spect-code)

---

## License

MIT - Use freely, modify as needed, contribute back if you can.

---

**Based on [everything-claude-code](https://github.com/affaan-m/everything-claude-code) by [Affaan Mustafa](https://github.com/affaan-m)**
