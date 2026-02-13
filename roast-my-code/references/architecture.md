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
- **Roast (en):** "This file has more responsibilities than a startup CEO who also does customer support. It wakes up, makes breakfast, deploys to prod, and handles billing -- all before line 200."
  - **Roast (ja):** "このファイル、責務多すぎませんか。朝起きて、朝食作って、デプロイして、請求処理もしてるんですよね。それって1ファイルじゃなくて1部署ですよね。"
- **Fix:** Split by responsibility. Extract cohesive groups of functions into focused modules. If you can't name what the file does in 5 words, it's doing too much.

### Circular Dependencies

- **Severity:** critical
- **Detect:** Trace import/require chains. If module A imports B and B imports A (directly or transitively), flag it. Check for cycles of any length in the dependency graph.
- **Deduction:** -20 points per cycle
- **Roast (en):** "Module A imports B, B imports A. That's not architecture -- that's a codependent relationship. Someone call a therapist, or at least a mediator module."
  - **Roast (ja):** "AがBをimportして、BがAをimportしてるの、それって設計じゃなくて共依存ですよね。どっちかが大人になって手を離してもらっていいですか。"
- **Fix:** Break the cycle with dependency inversion. Extract the shared interface into a third module. One of them needs to let go.

### Deep Nesting

- **Severity:** warning
- **Detect:** More than 4 levels of indentation from control flow structures (if/for/while/switch/try). Count nesting depth by braces or indentation level.
- **Deduction:** -4 points per occurrence
- **Roast (en):** "This nesting is deeper than a Christopher Nolan plot. Four levels of if-else-for-while -- at some point you're not writing code, you're spelunking."
  - **Roast (ja):** "ネストが深すぎて洞窟探検みたいになってるんですけど、4階層もif-for-while重ねて、ご自身で読めるんですか、これ。"
- **Fix:** Use early returns, guard clauses, and extract helper functions. Flatten the pyramid of doom.

### Mixed Concerns

- **Severity:** error
- **Detect:** Business logic functions that also perform I/O (file reads, HTTP calls, database queries). UI components that fetch data directly. Look for fetch/axios/fs/sql inside pure logic functions.
- **Deduction:** -10 points per violation
- **Roast (en):** "This function calculates prices, queries the database, AND sends emails. It's not a function -- it's an entire department that somehow fits in 40 lines of code."
  - **Roast (ja):** "この関数、価格計算して、DB叩いて、メールも送ってるんですけど、それって関数じゃなくて部署ですよね。単一責任の原則って聞いたことあります？"
- **Fix:** Separate pure logic from side effects. Business rules should take data in and return data out. Let the caller handle I/O.

### No Clear Entry Point

- **Severity:** warning
- **Detect:** Missing conventional entry point file pattern: no main.*, index.*, app.*, or equivalent. No obvious "start here" file in the project root or src directory.
- **Deduction:** -4 points
- **Roast (en):** "Where does this app even start? I've been wandering this codebase for 20 minutes like it's an IKEA with no exit signs and the meatballs are a lie."
  - **Roast (ja):** "このアプリ、どこから起動するんですか。20分探してるんですけど見つからないの、出口のないIKEAを彷徨ってる気分なんですよね。"
- **Fix:** Establish a clear entry point. Use conventional names (main, index, app) and document the startup path.

### Barrel File Explosion

- **Severity:** warning
- **Detect:** index.ts/index.js files that re-export everything from a directory. Especially dangerous when barrels re-export from other barrels, creating chains of hidden coupling.
- **Deduction:** -4 points per barrel chain
- **Roast (en):** "Your index.ts re-exports 47 things. That's not a barrel file -- that's a phone book, and half the entries are for people who moved away three versions ago."
  - **Roast (ja):** "index.tsが47個もre-exportしてるんですけど、それってバレルファイルじゃなくて電話帳ですよね。半分はもう使われてないんじゃないですか。"
- **Fix:** Import directly from source modules instead of through barrels. Keep barrel files thin or remove them entirely. Your bundler will thank you.

### Global Mutable State

- **Severity:** error
- **Detect:** Module-level `let` or `var` declarations that get reassigned. Singleton patterns with mutable internal state. Global variables. Shared mutable caches without clear ownership.
- **Deduction:** -10 points per instance
- **Roast (en):** "Global mutable state -- a variable that anyone, anywhere, anytime can change for any reason. Debugging this is less 'investigation' and more 'seance'."
  - **Roast (ja):** "グローバルなmutableステート置いてますけど、誰がいつ変更したか追えないの、デバッグじゃなくて降霊術ですよね。DIって知ってます？"
- **Fix:** Use dependency injection. Pass state explicitly. If you need shared state, use a proper state management pattern with controlled access.

### Tight Coupling

- **Severity:** error
- **Detect:** Classes or functions that directly instantiate their dependencies with `new` instead of receiving them. Hardcoded references to concrete implementations rather than interfaces or abstractions.
- **Deduction:** -10 points per violation
- **Roast (en):** "Your service `new`s up its own database connection inside itself. That's not dependency injection -- that's a hostage situation with no negotiator."
  - **Roast (ja):** "サービスの中で直接`new`してDB接続作ってるの、それって依存性注入じゃなくて人質事件ですよね。テスト書けないのは仕様じゃなくて設計ミスですよ。"
- **Fix:** Accept dependencies as constructor or function parameters. Depend on abstractions, not concretions. Make it possible to swap implementations without rewriting the consumer.

### Layer Violations

- **Severity:** error
- **Detect:** UI/presentation layer importing directly from database/persistence layer, skipping the service/business layer. API route handlers containing raw SQL. Components importing ORM models.
- **Deduction:** -10 points per violation
- **Roast (en):** "Your React component imports a Prisma model directly. The three-layer architecture diagram on your wiki was aspirational fiction, apparently."
  - **Roast (ja):** "ReactコンポーネントからPrismaモデルを直接importしてるの、Wikiに貼ってある3層アーキテクチャの図、あれフィクションだったんですね。"
- **Fix:** Enforce layer boundaries. UI talks to services, services talk to repositories. No skipping layers. Draw the boundary and respect it.

### Monolithic Modules

- **Severity:** warning
- **Detect:** Single module or directory handling all application logic with no separation. Everything lives in one folder. No clear boundaries between features, domains, or technical concerns.
- **Deduction:** -4 points per monolith
- **Roast (en):** "One folder. Thirty files. Zero boundaries. This isn't a module -- it's a storage unit that someone gave up organizing in 2022 and never came back."
  - **Roast (ja):** "1フォルダに30ファイルで境界ゼロって、それってモジュールじゃなくてトランクルームですよね。2022年に整理を諦めたんですか。"
- **Fix:** Decompose by domain or feature. Group related files together and draw clear boundaries. Even a simple src/auth, src/users, src/orders split is a massive improvement.

### Config Scattered

- **Severity:** info
- **Detect:** Configuration values (URLs, ports, API keys, feature flags, magic numbers) hardcoded across multiple source files instead of centralized in config/env files.
- **Deduction:** -1 point per scattered config
- **Roast (en):** "I found your API URL hardcoded in 7 different files. Changing it in prod is going to be a delightful game of whack-a-mole where every mole bites back."
  - **Roast (ja):** "API URLが7ファイルにハードコードされてるんですけど、本番で変更するときモグラ叩きになりますよね。環境変数って概念、ご存知ですか。"
- **Fix:** Centralize configuration. Use environment variables, a config module, or a .env file. One source of truth, referenced everywhere.

### DRY Violations

- **Severity:** warning
- **Detect:** Look for duplicated code blocks across files. Search for identical or near-identical multi-line patterns (5+ lines) appearing in 2+ files. Also flag copy-pasted configuration blocks and duplicated validation logic. Use Grep to spot repeated function signatures with similar bodies.
- **Deduction:** -4 points
- **Roast (en):** "I found the same 15-line function in three different files. That's not reuse — that's a copy-paste family reunion. When you fix a bug, you'll fix it in one place and forget the other two."
  - **Roast (ja):** "同じ15行の関数が3ファイルにあるんですけど、それって再利用じゃなくてコピペの親戚集会ですよね。バグ直すとき1箇所だけ直して残り2箇所忘れるやつですよ。"
- **Fix:** Extract duplicated logic into shared utility functions or modules. Follow the Rule of Three — if you've copied it twice, it's time to abstract. Parameterize the differences.

### Law of Demeter Violations

- **Severity:** warning
- **Detect:** Use Grep for property access chains with 4+ dots. Pattern: `\w+(\.\w+){4,}`. Exclude import paths, package namespaces, and chained method calls on builder/fluent APIs (e.g., query builders, test matchers).
- **Deduction:** -4 points
- **Roast (en):** "`user.address.city.district.name` — you're not accessing data, you're on an archaeological expedition. One refactor to `address` and this entire chain turns to dust."
  - **Roast (ja):** "`user.address.city.district.name`って、データアクセスじゃなくて考古学の発掘調査ですよね。途中の構造変えた瞬間、このチェーン全部壊れますけど。"
- **Fix:** Follow the Law of Demeter — only talk to your immediate friends. Use intermediate variables, destructuring, or delegate methods. If `user.getDistrictName()` exists, call that instead of drilling.

### Side Effects in Constructors

- **Severity:** warning
- **Detect:** Grep for I/O, API calls, or async operations inside constructors. Patterns: `constructor\(` blocks containing `fetch`, `axios`, `http`, `fs\.`, `readFile`, `writeFile`, `database`, `query`, `.save(`, `.create(`, `addEventListener`, `setInterval`, `setTimeout`.
- **Deduction:** -4 points
- **Roast (en):** "Your constructor makes API calls. `new User()` shouldn't need a network connection. Constructors initialize — they don't launch side quests."
  - **Roast (ja):** "コンストラクタでAPI呼んでるの、`new User()`するのにネットワーク接続が必要っておかしいですよね。コンストラクタは初期化する場所であって冒険に出る場所じゃないんですよ。"
- **Fix:** Keep constructors pure — assign parameters to properties only. Use static factory methods (`User.create()`) or builder patterns for async initialization. Separate object creation from I/O.

### Dependency Inversion Violations

- **Severity:** info
- **Detect:** Check for direct imports of infrastructure in business logic. Grep for `import.*from '.*/database'`, `import.*from '.*/infrastructure'`, `import.*from '.*/adapters'` in domain/business logic directories. Flag when core logic directly imports database drivers, HTTP clients, or external service SDKs.
- **Deduction:** -1 point
- **Roast (en):** "Your business logic imports the database driver directly. Swap PostgreSQL for MySQL and watch 47 files burn. That's not architecture — it's a hostage situation."
  - **Roast (ja):** "ビジネスロジックがDBドライバを直接importしてるの、PostgreSQLからMySQL変えるだけで47ファイル書き直しですよね。それアーキテクチャじゃなくて人質事件ですよ。"
- **Fix:** Depend on abstractions (interfaces/types), not concretions. Define repository interfaces in the domain layer. Inject implementations via dependency injection or factory functions. Business logic should never know about infrastructure.

---

## Scoring

| Step | Detail |
|------|--------|
| Start | 100 points |
| Deductions | Apply per finding using the values above |
| Floor | Minimum score is 0 (no negatives) |
| Weight | Multiply final score by **1.2x** for category contribution |

Example: 2 god files (-20) + 1 circular dependency (-20) + 3 deep nestings (-12) = 100 - 52 = 48. Weighted: 48 * 1.2 = 57.6.
