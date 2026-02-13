# Checks Index

Compact pattern index for 2-phase execution.

- **Phase 1**: Batch-grep these patterns to detect findings per category.
- **Phase 2**: Read full reference files only for hit categories.
- **(P)** = Presence: pattern found = potential issue.
- **(A)** = Absence: pattern NOT found = issue.

162 checks across 13 categories.

---

## Security (22 checks, weight: 1.5x)

### Grep [critical] (P)

- **Hardcoded Secrets**: `(password|secret|api[_-]?key|token)\s*=\s*["']`
- **SQL Injection**: `["']\s*\+.*(SELECT|INSERT|UPDATE|DELETE)`
- **Command Injection**: `exec\(.*\+|spawn\(.*\+|system\(.*\+|eval\(.*req\.|subprocess.*shell=True`
- **XSS Vulnerabilities**: `innerHTML\s*=|dangerouslySetInnerHTML|v-html|document\.write\(`
- **SSRF**: `fetch\(.*req\.(query|params|body)|axios\.\w+\(.*req\.|http\.get\(.*req\.`
- **Path Traversal**: `readFile\(req\.|fs\.\w+\(req\.|path\.join\(.*req\.`

### Grep [error] (P)

- **JWT Vulnerabilities**: `jwt\.sign\(|localStorage\.setItem.*token|jwt\.decode`
- **Mass Assignment**: `\.create\(req\.body|\.update\(req\.body|Object\.assign\(.*req\.body`
- **Prototype Pollution**: `_\.merge\(.*req\.|_\.defaultsDeep\(.*req\.|deepMerge`
- **Insecure Deserialization**: `\beval\(|\bnew Function\(|vm\.run`

### Grep [error] (A)

- **Missing CSRF Protection**: `csurf|csrf-csrf|@fastify/csrf-protection`

### Grep [warning] (P)

- **Insecure Randomness**: `Math\.random`
- **Sensitive Data in Logs**: `console\.log\(.*password|logger\..*(token|secret|apiKey)`
- **Missing HTTPS**: `http://` (in config/env files, exclude localhost)
- **Permissive CORS**: `Access-Control-Allow-Origin.*\*|origin:\s*\*`
- **Open Redirect**: `redirect\(req\.(query|body|params)`
- **Insecure Cookie Config**: `res\.cookie\(` (verify httpOnly/secure absent)
- **Timing Attack**: `===.*(secret|token|apiKey|password)`

### Grep [info] (A)

- **Missing Security Headers**: `helmet`

### Glob

- **.env in Repository** [critical]: `.env`, `.env.local`, `.env.production`

### Bash

- **Insecure Dependencies** [error]: `npm audit --json 2>/dev/null | head -5`
- **.env in Repository** [critical]: `git ls-files '.env*' 2>/dev/null`

---

## Architecture (15 checks, weight: 1.2x)

### Grep [error] (P)

- **Global Mutable State**: `^(let|var)\s+\w+\s*=` (module-level)
- **Tight Coupling**: `new\s+(Database|Service|Repository|Connection)`
- **Layer Violations**: component files importing prisma/typeorm/sequelize/knex

### Grep [warning] (P)

- **Barrel File Explosion**: `export\s+\*\s+from` in index files
- **Law of Demeter**: `\w+(\.\w+){4,}`
- **Side Effects in Constructors**: constructor with fetch/axios/fs/query

### Grep [info] (P)

- **Dependency Inversion**: `import.*from '.*(database|infrastructure|adapters)'`

### Bash

- **God Files** [error]: `wc -l` on source files, flag > 500 lines
- **Circular Dependencies** [critical]: trace import/require chains
- **Mixed Concerns** [error]: fetch/axios in pure logic functions
- **Deep Nesting** [warning]: indentation depth > 4 levels
- **No Clear Entry Point** [warning]: check for main/index/app files
- **Monolithic Modules** [warning]: directories with 30+ source files
- **Config Scattered** [info]: hardcoded URLs in multiple files
- **DRY Violations** [warning]: duplicate multi-line blocks

---

## Complexity (12 checks, weight: 1.0x)

### Grep [warning] (P)

- **Nested Ternaries**: `\?[^?:]*\?`
- **Boolean Parameters**: `:\s*boolean[\s,)]|=\s*(true|false)\s*[,)]`
- **Deeply Nested Objects**: `\w+(\.\w+){4,}`

### Grep [info] (P)

- **Magic Numbers**: bare numeric literals in logic (not 0, 1, -1)

### Bash

- **Long Functions** [error]: function bodies > 50 lines
- **High Cyclomatic Complexity** [error]: 10+ branch keywords per function
- **Deep Callback Nesting** [error]: 3+ nested callback levels
- **Long Parameter Lists** [warning]: 5+ parameters
- **Complex Regular Expressions** [warning]: 80+ char regex without comment
- **Excessive Switch Cases** [warning]: 10+ case per switch
- **Multiple Return Types** [warning]: mixed return types
- **Missing Guard Clauses** [warning]: deeply nested if/else

---

## TDD (12 checks, weight: 1.0x / 1.5x sensei)

### Glob

- **No Test Files** [critical]: `**/*.test.*`, `**/*.spec.*`, `**/__tests__/**`
- **No CI Test Step** [warning]: `.github/workflows/*.yml` (check for test step)

### Grep [error] (P)

- **Commented-Out Tests**: `//\s*it\(|//\s*test\(|xit\(|xdescribe\(|test\.skip\(`

### Grep [warning] (P)

- **Overmocking**: `jest\.mock\(|vi\.mock\(|sinon\.stub\(` (5+ per file)
- **Slow Tests**: `timeout.*\d{5,}|setTimeout` in test files
- **Snapshot Overuse**: `toMatchSnapshot\(|toMatchInlineSnapshot\(` (10+ total)

### Grep [warning] (A)

- **Test Describes Only Happy Path**: `error|throw|reject|invalid|fail|boundary`

### Grep [info] (A)

- **Missing Coverage Config**: `coverage` in jest.config/vitest.config

### Bash

- **Low Test-to-Source Ratio** [error]: test vs source file count < 50%
- **Implementation Without Matching Test** [error]: source without test counterpart
- **Test Without Assertion** [error]: test blocks without expect/assert
- **Test File Larger Than Source** [info]: test > 3x source lines

---

## Type Safety (12 checks, weight: 1.0x)

### Grep [error] (P)

- **Explicit any**: `:\s*any\b|<any>|as any`
- **Object/Function/{} Types**: `:\s*(Object|Function|\{\})\b`

### Grep [warning] (P)

- **Type Assertions Abuse**: `\bas\s+(?!const)\w+`
- **@ts-ignore/@ts-expect-error**: `@ts-ignore|@ts-expect-error`
- **Null/Undefined Overuse**: `(\?\.\w+){3,}`

### Grep [warning] (A)

- **Missing Return Types**: explicit return types on `export function`
- **Untyped API Responses**: generic types on fetch/axios calls
- **Missing Generic Constraints**: `extends` on `<T>` parameters

### Grep [info] (P)

- **String Enums**: `enum\s+\w+\s*\{[^}]*=\s*["']`

### Bash

- **Implicit any** [error]: tsconfig.json noImplicitAny/strict check
- **Loose tsconfig** [error]: tsconfig.json strict:false or absent
- **No Readonly** [info]: const arrays/objects without readonly

---

## Error Handling (11 checks, weight: 1.0x)

### Grep [critical] (P)

- **Empty Catch Blocks**: `catch\s*\(.*\)\s*\{\s*\}|except.*:\s*pass`

### Grep [error] (P)

- **Swallowed Errors in Callbacks**: `function\s*\(err` where err unused

### Grep [warning] (P)

- **Catch-and-Log-Only**: catch blocks with only console.log/error
- **Generic Error Messages**: `new Error\(["'](error|something went wrong|fail)["']\)`
- **String Throws**: `throw\s+["'\x60]`
- **Error Logging Without Context**: `console\.error\(\w+\)\s*;$`

### Grep [warning] (A)

- **No Global Error Handler**: `process\.on\(["']uncaughtException|window\.onerror`

### Grep [info] (P)

- **Missing Finally Blocks**: try without finally in resource contexts

### Bash

- **Unhandled Promise Rejections** [error]: .then() without .catch()
- **Inconsistent Error Types** [warning]: mixed throw styles
- **No Fail-Fast Validation** [warning]: late validation in handlers

---

## Naming (11 checks, weight: 1.0x)

### Grep [error] (P)

- **Single-Letter Variables**: `(const|let|var)\s+[a-z]\s*[=:]`
- **Generic Names**: `(const|let|var)\s+(data|info|result|temp|obj|val|item|value|stuff|thing)\s*[=:]`

### Grep [warning] (P)

- **Abbreviated Names**: `\b(usr|msg|btn|cfg|mgr|impl|cb|fn|evt|elem|attr)\b`
- **Meaningless Prefixes**: `(const|let|var)\s+(my|the|a|an)[A-Z]`
- **Negative Booleans**: `(isNot|isNo|disable[^d]|disabled|not[A-Z]|without|never|dont)`

### Grep [info] (P)

- **Hungarian Notation**: `(str|int|bool|arr|obj|fn|num)[A-Z]\w+`
- **Overly Long Names**: `[a-zA-Z_$][a-zA-Z0-9_$]{40,}`

### Bash

- **Misleading Names** [error]: getX with side effects (Phase 2 analysis)
- **Inconsistent Casing** [warning]: camelCase + snake_case in same file
- **File Name Mismatch** [warning]: export name differs from filename
- **Pluralization Errors** [warning]: array with singular name

---

## Dead Code (11 checks, weight: 1.0x)

### Grep [error] (P)

- **Commented-Out Code**: 3+ comment lines with code patterns `=|\(|\)|if|return|const`
- **Unreachable Code**: statements after `return|throw|break|continue`

### Grep [warning] (P)

- **Dead Feature Flags**: `(FF_|FEATURE_|TOGGLE_).*=\s*(true|false)`
- **Deprecated API Usage**: `@deprecated` symbols still referenced

### Grep [info] (P)

- **TODO/FIXME/HACK**: `(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND)\b`

### Bash

- **Unused Imports** [warning]: imported identifiers not referenced
- **Unused Variables** [warning]: declared variables appearing once
- **Unused Functions** [warning]: exported functions never imported
- **Unused Dependencies** [error]: packages not imported anywhere
- **Empty Files** [warning]: source files with no executable code
- **Orphan Files** [warning]: files never imported by others

---

## Performance (12 checks, weight: 1.0x)

### Grep [error] (P)

- **Synchronous File I/O**: `readFileSync|writeFileSync|appendFileSync|mkdirSync|readdirSync`
- **Memory Leaks**: `addEventListener` without removeEventListener
- **Blocking Event Loop**: `crypto\.(pbkdf2|scrypt)Sync|while\(true\)`

### Grep [warning] (P)

- **Large Bundle Imports**: `import _\s+from\s+['"]lodash['"]|import moment\s+from`
- **Console.log in Production**: `console\.(log|debug|info|dir|table|trace)\(`
- **Callback Hell**: `\.then\(.*\.then\(.*\.then\(`

### Glob [info]

- **Unoptimized Images**: `**/*.{png,jpg,jpeg,gif,bmp}` (flag > 500KB)

### Bash

- **N+1 Query Pattern** [critical]: DB calls inside loops
- **Missing Pagination** [error]: SELECT without LIMIT, findAll without limit
- **Unbatched Operations** [warning]: consecutive independent awaits
- **No Caching Strategy** [warning]: absence of cache library
- **Missing Database Indexes** [warning]: unindexed columns in queries

---

## Dependencies (11 checks, weight: 1.0x)

### Glob

- **No Lockfile** [critical] (A): `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`

### Grep [error] (P)

- **Wildcard Versions**: `"\*"|"latest"` in package.json dependencies

### Grep [warning] (P)

- **DevDeps in Dependencies**: jest|mocha|eslint|prettier|typescript|webpack in dependencies
- **Direct GitHub Deps**: `github:|git\+https://` in package.json

### Grep [info] (A)

- **No Engine Specification**: `"engines"` in package.json
- **Missing License**: `"license"` in package.json

### Bash

- **Excessive Dependencies** [warning]: count > 30 direct dependencies
- **Duplicate Functionality** [warning]: overlapping libs (axios+fetch, moment+dayjs)
- **Outdated Major Versions** [warning]: 2+ major versions behind
- **Unpinned Peer Deps** [info]: broad peer dependency ranges
- **Unused Scripts** [info]: scripts referencing nonexistent files

---

## API Design (11 checks, weight: 1.0x)

### Grep [critical] (P)

- **No Input Validation**: `req\.body|req\.query|req\.params` without zod/joi/yup

### Grep [error] (P)

- **Inconsistent Response Format**: mixed response shapes across handlers
- **No Error Response Standard**: mixed error payloads across handlers

### Grep [warning] (P)

- **Missing Status Codes**: `res\.json\(` without `res\.status\(`
- **Overfetching**: `SELECT\s+\*|\.findAll\(\)|\.find\(\{\}\)` without select
- **Inconsistent Naming**: verbs in route paths `/get|/fetch|/create|/delete`

### Grep [warning] (A)

- **No Rate Limiting**: `express-rate-limit|@nestjs/throttler|rate-limiter-flexible`
- **Versioning Absent**: `/v[0-9]+/|enableVersioning`
- **Missing CORS Config**: `cors` middleware setup

### Grep [info] (A)

- **No Request Logging**: `morgan|pino-http|express-winston`
- **No OpenAPI/Swagger**: `swagger|openapi`

---

## Frontend (11 checks, weight: 1.0x)

### Grep [error] (P)

- **No Accessibility**: `<img(?![^>]*alt=)|<button(?![^>]*aria-label)`
- **Missing Key Props**: `\.map\(` with JSX return without key=

### Grep [error] (A)

- **No Error Boundaries**: `ErrorBoundary|componentDidCatch|getDerivedStateFromError`

### Grep [warning] (P)

- **Inline Styles Overuse**: `style=\{\{|:style="`
- **useEffect Abuse**: `useEffect\(` without dependency array
- **Direct DOM Manipulation**: `document\.getElementById|document\.querySelector|\.classList\.`
- **Hardcoded Strings**: raw text in JSX heading/button/label elements

### Grep [warning] (A)

- **No Loading States**: loading/isLoading handling for async data

### Bash

- **Prop Drilling** [error]: same prop through 3+ component levels
- **Massive Components** [error]: component files > 300 lines
- **Missing Semantic HTML** [info]: div-heavy without semantic elements

---

## Git Hygiene (11 checks, weight: 1.0x)

### Glob

- **No .gitignore** [critical] (A): `.gitignore`
- **Missing README** [warning] (A): `README.md`, `README`, `README.txt`

### Bash

- **Committed node_modules** [critical]: `git ls-files node_modules/ | head -5`
- **Large Files Tracked** [error]: tracked files > 1MB
- **Secrets in History** [critical]: .env/.pem/.key in git history
- **No Branching Strategy** [warning]: only main/master branch
- **Giant Commits** [warning]: commits touching 30+ files
- **Meaningless Messages** [warning]: short/generic commit messages
- **No Branch Protection** [info]: direct pushes to main
- **Merge Commits Galore** [info]: 30%+ merge commits in history
- **Stale Branches** [info]: branches untouched > 30 days
