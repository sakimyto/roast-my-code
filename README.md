# 🔥 roast-my-code

[日本語](README.ja.md)

**Code reviews that hit different. Sharp feedback meets actual fixes.**

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that delivers brutally honest code reviews as entertaining roasts — with actionable fixes. No API keys. No dependencies. Just markdown.

## Demo

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🔥 ROAST MY CODE — REPORT CARD 🔥
  Level: spicy | Target: ./
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 OVERALL: B (74/100)

┌─────────────────────────────────────┐
│ Category          Score    Grade    │
├─────────────────────────────────────┤
│ Security          80/100     A     │
│ Architecture      70/100     B     │
│ Complexity        80/100     A     │
│ TDD               60/100     C     │
│ Type Safety       70/100     B     │
│ Error Handling    60/100     C     │
│ Naming            76/100     B     │
│ Dead Code         76/100     B     │
│ Performance       90/100     S     │
│ Dependencies      90/100     S     │
│ Frontend          76/100     B     │
│ Git Hygiene       88/100     A     │
└─────────────────────────────────────┘

━━━ 🔥 FINDINGS ━━━━━━━━━━━━━━━━━━━━━━

## TDD (60/100)

### 🚨 Low Test-to-Source Ratio [error]
📍 apps/web/src/
> 180 source files, 46 test files (26% ratio)

💬 "Your test coverage is giving 'I'll write tests later'
    energy. Later never came."

🔧 Fix: Aim for at least 1 test file per source file
   in critical paths. Start with business logic, then
   work outward to API handlers and utilities.

---

## Error Handling (60/100)

### 🚨 Empty Catch Blocks [critical]
📍 src/lib/api-client.ts:42
> } catch (e) {}

💬 "Ah yes, catch and ignore — the ostrich pattern.
    If you can't see the error, it doesn't exist."

🔧 Fix: At minimum, log the error. Better: rethrow
   with context or handle the specific failure case.

━━━ 📋 SUMMARY ━━━━━━━━━━━━━━━━━━━━━━━

🏆 Top 3 Strengths:
  1. Performance — no N+1 queries, async I/O used
  2. Dependencies — clean lockfile, no wildcards
  3. Git Hygiene — meaningful commits, .gitignore ok

💀 Top 3 Priorities to Fix:
  1. Empty catch blocks (src/lib/api-client.ts)
  2. Test coverage for business logic (apps/web/)
  3. Explicit `any` in 8 type declarations

📈 Quick Wins (< 5 min each):
  - Add `strict: true` to tsconfig.json
  - Replace console.log with structured logger
  - Remove 3 unused imports in src/utils/

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Roasted with ❤️ by roast-my-code
  ⭐ github.com/sakimyto/roast-my-code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Installation

Three steps. No npm. No config. Just copy.

### 1. Clone

```bash
git clone https://github.com/sakimyto/roast-my-code.git
```

### 2. Copy the skill

**Per-project (recommended):**

```bash
cp -r roast-my-code/roast-my-code your-project/.claude/skills/
```

**Global (available in all projects):**

```bash
cp -r roast-my-code/roast-my-code ~/.claude/skills/
```

### 3. Roast

In Claude Code, inside your project:

```
/roast
```

That's it.

## Usage

```bash
/roast                              # Review cwd at spicy level
/roast src/                         # Review specific directory
/roast --level=savage               # Maximum roast intensity
/roast src/ --level=sensei          # TDD-focused wisdom
/roast --diff                       # Review staged changes only
/roast --quick                      # Fast scan (top 3 checkers)
/roast --lang=ja                    # Force Japanese output
```

## Roast Levels

| Level | Tone | Example |
|-------|------|---------|
| `gentle` | Friendly mentor | "This works, but we could make it even better by..." |
| **`spicy`** | Sarcastic reviewer (default) | "I admire the courage it took to push this without tests." |
| `savage` | Brutally honest senior | "I've seen cleaner code in a PHP tutorial from 2008." |
| `sensei` | TDD master (t_wada style) | "Red, Green, Refactor — you appear to have skipped all three." |

## What It Checks

13 categories, 160+ individual check items:

| Category | Checks | Weight |
|----------|--------|--------|
| **Security** | Hardcoded secrets, SQL/XSS injection, insecure deps | 1.5x |
| **Architecture** | God files, circular deps, tight coupling | 1.2x |
| **Complexity** | Long functions, deep nesting, cyclomatic complexity | 1.0x |
| **TDD** | Test coverage, assertion quality, Red-Green-Refactor | 1.0x* |
| **Type Safety** | `any` abuse, strict mode, type assertions | 1.0x |
| **Error Handling** | Empty catches, swallowed errors, string throws | 1.0x |
| **Naming** | Generic names, inconsistent casing, abbreviations | 1.0x |
| **Dead Code** | Commented code, unused imports, stale TODOs | 1.0x |
| **Performance** | N+1 queries, sync I/O, memory leaks | 1.0x |
| **Dependencies** | No lockfile, wildcard versions, unused packages | 1.0x |
| **API Design** | Input validation, response format, status codes | 1.0x |
| **Frontend** | Accessibility, prop drilling, component size | 1.0x |
| **Git Hygiene** | No .gitignore, large files, commit messages | 1.0x |

*\*TDD weight increases to 1.5x in sensei mode*

Conditional checkers (API Design, Frontend, Git Hygiene, Dependencies) activate only when the relevant framework or tool is detected.

## Bilingual Output

roast-my-code automatically detects your language and adapts its output:

- **English**: Tech Twitter-viral dry humor
- **Japanese (日本語)**: ひろゆき style deadpan sarcasm — 「なんだろう、テストないのやめてもらっていいですか」

Use `--lang=ja` or `--lang=en` to override auto-detection.

## Language Support

Check definitions are optimized for **JavaScript / TypeScript** projects. Many checks (Security, Architecture, Complexity, TDD, Naming, Dead Code, Git Hygiene) work across any language. Framework-specific checks (Dependencies, Frontend, API Design) are currently Node.js ecosystem focused.

## Grading Scale

| Grade | Score | Meaning |
|-------|-------|---------|
| **S** | 90-100 | Immaculate — ship it yesterday |
| **A** | 80-89 | Solid — production-ready |
| **B** | 70-79 | Decent — code review approved |
| **C** | 60-69 | Needs Work — another pass required |
| **D** | 40-59 | Rough — significant issues |
| **F** | 0-39 | Dumpster Fire — call the fire department |

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- That's it. No API keys needed — Claude Code IS the LLM.

## How It Works

roast-my-code is a pure-markdown Claude Code skill. It uses Claude's built-in tools (Read, Glob, Grep, Bash) to analyze your codebase and delivers findings in a roast format. The reference files in `references/` define detection patterns, severity levels, and example roasts for each category.

```
.claude/skills/roast-my-code/
├── SKILL.md              # Main skill (detection + scoring + output)
└── references/
    ├── roast-style.md    # Persona definitions per level
    ├── security.md       # Security checks
    ├── architecture.md   # Architecture checks
    ├── complexity.md     # Complexity checks
    ├── tdd.md            # TDD compliance checks
    ├── type-safety.md    # Type safety checks
    ├── error-handling.md # Error handling checks
    ├── naming.md         # Naming convention checks
    ├── dead-code.md      # Dead code checks
    ├── performance.md    # Performance checks
    ├── dependencies.md   # Dependency checks
    ├── api-design.md     # API design checks
    ├── frontend.md       # Frontend quality checks
    └── git-hygiene.md    # Git hygiene checks
```

## Contributing

1. Fork the repository
2. Add or improve check items in `references/*.md`
3. Submit a PR with example output showing the improvement

Please keep roast examples:

- Factual (grounded in real code patterns)
- Entertaining (the whole point)
- Actionable (always pair with a fix)

## License

[MIT](LICENSE)

---

*Built for the AI age. Powered by markdown. Fueled by sarcasm.*
