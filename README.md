# 🔥 roast-my-code

**Your code is bad and you should feel entertained about it.**

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that delivers brutally honest code reviews as entertaining roasts — with actionable fixes. No API keys. No dependencies. Just markdown.

## Demo

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🔥 ROAST MY CODE — REPORT CARD 🔥
  Level: spicy | Target: src/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 OVERALL: C (64/100)

┌─────────────────────────────────────┐
│ Category          Score    Grade    │
├─────────────────────────────────────┤
│ Security          45/100     D     │
│ Architecture      70/100     B     │
│ Complexity        80/100     A     │
│ TDD               30/100     F     │
│ Error Handling    55/100     D     │
│ Naming            85/100     A     │
│ Dead Code         72/100     B     │
│ Performance       90/100     S     │
└─────────────────────────────────────┘

━━━ 🔥 FINDINGS ━━━━━━━━━━━━━━━━━━━━━━

## TDD (30/100)

### 🚨 No Test Files [critical]
📍 src/
> Zero test files found. Glob: **/*.test.* returned nothing.

💬 "I found your test suite. Just kidding — there isn't one."

🔧 Fix: Create a __tests__/ directory and add unit tests
   for core business logic. Start with the most critical
   path: user authentication.

---

## Security (45/100)

### 🚨 Hardcoded API Key [critical]
📍 src/config/api.ts:12
> const API_KEY = "sk-live-abc123..."

💬 "You hardcoded an API key. In 2026. I'm not even angry,
    I'm impressed by the audacity."

🔧 Fix: Move to environment variables. Use dotenv for local
   dev and your platform's secret manager for production.

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

```bash
cp -r roast-my-code/roast-my-code /path/to/your/project/.claude/skills/
```

### 3. Roast

In Claude Code, inside your project:

```
/roast
```

That's it.

## Roast Levels

| Level | Tone | Example |
|-------|------|---------|
| `gentle` | Friendly mentor | "This works, but we could make it even better by..." |
| **`spicy`** | Sarcastic reviewer (default) | "I admire the courage it took to push this without tests." |
| `savage` | Brutally honest senior | "I've seen cleaner code in a PHP tutorial from 2008." |
| `sensei` | TDD master (t_wada style) | "Red, Green, Refactor — you appear to have skipped all three." |

```bash
/roast --level=savage        # Maximum entertainment
/roast src/ --level=sensei   # TDD-focused wisdom
```

## What It Checks

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

Conditional checkers (API Design, Frontend, Git Hygiene) activate only when the relevant framework or tool is detected.

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
