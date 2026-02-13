# Architecture Checks

Category weight: **1.2x**

## Activation

Always active. These checks apply to every codebase regardless of language or framework.
Scan project structure, import graphs, and module boundaries to find architectural sins.

---

## Check Items

### God Files

- **Severity:** error
- **Detect:** Any single file exceeding 500 lines of code (excluding comments and blank lines)
- **Deduction:** -10 points per file
- **Roast:** "This file has more responsibilities than a Swiss Army knife. It wakes up, makes breakfast, does your taxes, AND deploys to prod."
- **Fix:** Split by responsibility. Extract cohesive groups of functions into focused modules. If you can't name what the file does in 5 words, it's doing too much.

### Circular Dependencies

- **Severity:** critical
- **Detect:** Trace import/require chains. If module A imports B and B imports A (directly or transitively), flag it. Check for cycles of any length in the dependency graph.
- **Deduction:** -20 points per cycle
- **Roast:** "Congratulations, your modules are in a relationship. A needs B, B needs A... this is codependency, not code dependency."
- **Fix:** Break the cycle with dependency inversion. Extract the shared interface into a third module. One of them needs to let go.

### Deep Nesting

- **Severity:** warning
- **Detect:** More than 4 levels of indentation from control flow structures (if/for/while/switch/try). Count nesting depth by braces or indentation level.
- **Deduction:** -4 points per occurrence
- **Roast:** "This nesting goes deeper than a Christopher Nolan movie. We need to go deeper... actually no, we really don't."
- **Fix:** Use early returns, guard clauses, and extract helper functions. Flatten the pyramid of doom.

### Mixed Concerns

- **Severity:** error
- **Detect:** Business logic functions that also perform I/O (file reads, HTTP calls, database queries). UI components that fetch data directly. Look for fetch/axios/fs/sql inside pure logic functions.
- **Deduction:** -10 points per violation
- **Roast:** "This function calculates prices AND calls the database AND sends emails. It's not a function, it's a whole department."
- **Fix:** Separate pure logic from side effects. Business rules should take data in and return data out. Let the caller handle I/O.

### No Clear Entry Point

- **Severity:** warning
- **Detect:** Missing conventional entry point file pattern: no main.*, index.*, app.*, or equivalent. No obvious "start here" file in the project root or src directory.
- **Deduction:** -4 points
- **Roast:** "Where does this app even start? I've been wandering this codebase for 20 minutes like it's an IKEA with no exit signs."
- **Fix:** Establish a clear entry point. Use conventional names (main, index, app) and document the startup path.

### Barrel File Explosion

- **Severity:** warning
- **Detect:** index.ts/index.js files that re-export everything from a directory. Especially dangerous when barrels re-export from other barrels, creating chains of hidden coupling.
- **Deduction:** -4 points per barrel chain
- **Roast:** "Your index.ts re-exports 47 things. It's not an index, it's a phone book. And half the entries are unlisted."
- **Fix:** Import directly from source modules instead of through barrels. Keep barrel files thin or remove them entirely. Your bundler will thank you.

### Global Mutable State

- **Severity:** error
- **Detect:** Module-level `let` or `var` declarations that get reassigned. Singleton patterns with mutable internal state. Global variables. Shared mutable caches without clear ownership.
- **Deduction:** -10 points per instance
- **Roast:** "Global mutable state? Bold strategy. Nothing says 'impossible to debug' like a variable that anyone, anywhere, can change at any time."
- **Fix:** Use dependency injection. Pass state explicitly. If you need shared state, use a proper state management pattern with controlled access.

### Tight Coupling

- **Severity:** error
- **Detect:** Classes or functions that directly instantiate their dependencies with `new` instead of receiving them. Hardcoded references to concrete implementations rather than interfaces or abstractions.
- **Deduction:** -10 points per violation
- **Roast:** "Your service `new`s up its own database connection. That's not dependency management, that's a hostage situation."
- **Fix:** Accept dependencies as constructor or function parameters. Depend on abstractions, not concretions. Make it possible to swap implementations without rewriting the consumer.

### Layer Violations

- **Severity:** error
- **Detect:** UI/presentation layer importing directly from database/persistence layer, skipping the service/business layer. API route handlers containing raw SQL. Components importing ORM models.
- **Deduction:** -10 points per violation
- **Roast:** "Your React component imports a Prisma model directly. The architecture diagram said 3 layers but the code said 'nah, shortcut.'"
- **Fix:** Enforce layer boundaries. UI talks to services, services talk to repositories. No skipping layers. Draw the boundary and respect it.

### Monolithic Modules

- **Severity:** warning
- **Detect:** Single module or directory handling all application logic with no separation. Everything lives in one folder. No clear boundaries between features, domains, or technical concerns.
- **Deduction:** -4 points per monolith
- **Roast:** "One folder, 30 files, zero boundaries. This isn't a module, it's a storage unit that someone gave up organizing."
- **Fix:** Decompose by domain or feature. Group related files together and draw clear boundaries. Even a simple src/auth, src/users, src/orders split is a massive improvement.

### Config Scattered

- **Severity:** info
- **Detect:** Configuration values (URLs, ports, API keys, feature flags, magic numbers) hardcoded across multiple source files instead of centralized in config/env files.
- **Deduction:** -1 point per scattered config
- **Roast:** "I found your API URL hardcoded in 7 different files. Updating it in prod should be a fun game of whack-a-mole."
- **Fix:** Centralize configuration. Use environment variables, a config module, or a .env file. One source of truth, referenced everywhere.

---

## Scoring

| Step | Detail |
|------|--------|
| Start | 100 points |
| Deductions | Apply per finding using the values above |
| Floor | Minimum score is 0 (no negatives) |
| Weight | Multiply final score by **1.2x** for category contribution |

Example: 2 god files (-20) + 1 circular dependency (-20) + 3 deep nestings (-12) = 100 - 52 = 48. Weighted: 48 * 1.2 = 57.6.
