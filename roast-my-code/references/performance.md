# Performance Checks

## Activation

Always active.

## Check Items

### N+1 Query Pattern

- **Severity:** critical
- **Detect:** Grep for database query calls (`query`, `findOne`, `findById`, `fetch`, `select`)
  inside loop constructs (`for`, `forEach`, `map`, `while`, `for...of`). Also check for ORM
  calls (`.find()`, `.where()`, `.load()`) inside iteration blocks.
- **Deduction:** -20 points
- **Roast (en):** "A database query inside a loop. Congrats, you've invented a self-inflicted DDoS. Your DB is filing a restraining order."
  - **Roast (ja):** "ループの中でDBクエリ投げてるんですけど、それってセルフDDoSですよね。なんだろう、1回のJOINで済むことを1万回に分けてやるの、やめてもらっていいですか。"
- **Fix:** Use eager loading (`include`, `populate`, `prefetch_related`, `joinedload`), batch
  queries with `WHERE id IN (...)`, or restructure with JOINs. Fetch all needed data in a
  single query and process in-memory.

### Synchronous File I/O

- **Severity:** error
- **Detect:** Grep for `readFileSync`, `writeFileSync`, `appendFileSync`, `existsSync`,
  `mkdirSync`, `readdirSync` in non-CLI code. Check if the file is a server entry point,
  route handler, or middleware. Exclude scripts, CLI tools, and build configs.
- **Deduction:** -10 points
- **Roast (en):** "`readFileSync` in a server handler. Node.js handed you the gift of async I/O and you said 'no thanks, I enjoy watching the world freeze.'"
  - **Roast (ja):** "サーバーのハンドラで`readFileSync`使ってるの、せっかくNodeが非同期I/Oくれてるのに全部無視してますよね。それって高速道路を自転車で走ってるようなものなんですよ。"
- **Fix:** Use async equivalents (`readFile`, `writeFile` from `fs/promises`) with `await`.
  For streams, use `createReadStream`/`createWriteStream`. Sync I/O is acceptable only in
  CLI tools, build scripts, or one-time startup config loading.

### Missing Pagination

- **Severity:** error
- **Detect:** Grep for `SELECT` without `LIMIT`/`OFFSET`/`TOP`. Check ORM calls like
  `.findAll()`, `.find({})`, `.all()`, `.list()` without `.limit()`, `.take()`, `.paginate()`.
  Flag API endpoints returning unbounded arrays.
- **Deduction:** -10 points
- **Roast (en):** "`SELECT * FROM everything` with no LIMIT. Bold strategy when the table has a million rows. Hope your cloud budget is as unbounded as your query."
  - **Roast (ja):** "LIMIT無しで全件取得してるんですけど、100万行のテーブルに対してそれやるの、食べ放題で全メニュー注文する人と同じですよね。サーバーのRAMは無限じゃないんで。"
- **Fix:** Add `LIMIT`/`OFFSET` or cursor-based pagination to all list queries. Return pagination metadata (`total`, `page`, `per_page`, `next_cursor`). Set a sensible default limit (20-100).

### Memory Leaks

- **Severity:** error
- **Detect:** Grep for `addEventListener` without corresponding `removeEventListener`,
  `setInterval` without `clearInterval`, `setTimeout` accumulation in loops, event emitter
  `.on()` without `.off()`. Check for missing cleanup in React `useEffect` return functions.
- **Deduction:** -10 points
- **Roast (en):** "`addEventListener` without `removeEventListener`. You're collecting event listeners like Pokemon — gotta leak 'em all until OOM kills your process."
  - **Roast (ja):** "`addEventListener`してるのに`removeEventListener`がないですよね。それってサブスク契約だけして一生解約しない人と同じで、最終的にOOM Killerに強制退場させられますよ。"
- **Fix:** Always pair `addEventListener` with `removeEventListener`. Clear intervals and
  timeouts on unmount or process exit. Use `WeakMap`/`WeakRef` for caches. In React, return
  cleanup functions from `useEffect`. Monitor memory with `--inspect` and heap snapshots.

### Blocking the Event Loop

- **Severity:** error
- **Detect:** Grep for heavy computation in server code: `JSON.parse` on large inputs without
  streaming, nested loops with O(n^2)+ complexity in handlers, `crypto.pbkdf2Sync`,
  `crypto.scryptSync`, `while(true)` loops, and CPU-intensive regex (ReDoS).
- **Deduction:** -10 points
- **Roast (en):** "O(n^3) on the main thread. Every other user is frozen in time while your algorithm takes a scenic tour of computational complexity. Worker threads are free, you know."
  - **Roast (ja):** "メインスレッドでO(n^3)回してるんですけど、他のユーザー全員フリーズしてますよ。なんだろう、Worker Threadsっていう便利なもの、使ってもらっていいですか。"
- **Fix:** Offload CPU-intensive work to worker threads, child processes, or a job queue
  (Bull, BullMQ). Break large synchronous operations into chunks using `setImmediate`. Use
  streaming JSON parsers for large payloads. Avoid synchronous crypto in request handlers.

### Unbatched Operations

- **Severity:** warning
- **Detect:** Look for multiple sequential `await` statements where the results are independent
  of each other. Consecutive `await` calls where none uses the return value of the previous
  one. Also check for `await` inside `forEach` (which doesn't actually await).
- **Deduction:** -4 points
- **Roast (en):** "Three sequential `await`s that don't depend on each other. `Promise.all` is standing right there, waving a giant foam finger, and you walked past it."
  - **Roast (ja):** "依存関係ないのに`await`を直列で3つ並べてるの、それって3つのレジが空いてるのに1つだけに並んでる状態ですよね。`Promise.all`使ってもらっていいですか。"
- **Fix:** Use `Promise.all()` for independent async operations. Use `Promise.allSettled()` when individual failures should not abort others. For rate-limited APIs, use `p-limit` or `p-queue`.

### Large Bundle Imports

- **Severity:** warning
- **Detect:** Grep for full-library imports of known heavy packages: `import _ from 'lodash'`,
  `require('lodash')`, `import moment from 'moment'`, `import * as Rx from 'rxjs'`. Also
  flag `import * as` from large utility libraries without tree-shaking evidence.
- **Deduction:** -4 points
- **Roast (en):** "`import _ from 'lodash'` just to use `_.get()`. 70KB for one function. Your bundle is so bloated it needs its own loading screen and intermission."
  - **Roast (ja):** "lodash丸ごとimportして使ってるの1関数だけですよね。それって辞書を買って1ページだけ読むのと同じなんですけど、サブパスインポートって知ってますかね。"
- **Fix:** Use subpath imports (`import get from 'lodash/get'`), switch to `lodash-es` for
  tree-shaking, or replace with native equivalents (`Array.find`, `Object.entries`,
  `structuredClone`). Replace `moment` with `date-fns` or `dayjs`. Use `@aws-sdk/client-*`
  v3 modular packages.

### No Caching Strategy

- **Severity:** warning
- **Detect:** Look for repeated expensive computations (database queries, API calls) in hot
  paths without memoization or caching. Check for missing `Cache-Control` headers on
  semi-static API responses. Flag absence of caching libraries (`lru-cache`, `redis`).
- **Deduction:** -4 points
- **Roast (en):** "Recomputing the same expensive result on every single request. Your server has the memory of a goldfish and the electric bill of a Bitcoin miner."
  - **Roast (ja):** "毎リクエスト同じ重い計算を繰り返してるんですけど、それってメモ取らずに毎回同じこと調べ直してる人ですよね。キャッシュっていう概念、ご存知ないですかね。"
- **Fix:** Add memoization for pure functions (`lodash.memoize`, `lru-cache`). Use HTTP
  caching headers (`Cache-Control`, `ETag`, `Last-Modified`). Implement Redis or in-memory
  caching for database query results. Use React's `useMemo`/`useCallback` for expensive
  component computations.

### Console.log in Production

- **Severity:** warning
- **Detect:** Grep for `console.log`, `console.debug`, `console.info`, `console.dir`,
  `console.table`, `console.trace` in source files. Exclude test files (`*.test.*`,
  `*.spec.*`, `__tests__/`) and scripts. Check if a logging library (winston, pino) exists.
- **Deduction:** -4 points
- **Roast (en):** "`console.log('here')` shipped to production. Your logs read like a teenager's diary: 'here', 'wtf', 'asdf', and the classic 'REMOVE BEFORE DEPLOY'. Narrator: they did not."
  - **Roast (ja):** "本番に`console.log('here')`が残ってるんですけど、それって提出前に消し忘れた落書きですよね。なんだろう、構造化ログっていう大人のツール、使ってもらっていいですか。"
- **Fix:** Use a structured logging library (pino, winston) with log levels. Strip `console.log` with an ESLint rule (`no-console`) or a build-time transform. Use conditional debug logging (`debug` package) toggled via environment variables.

### Missing Database Indexes

- **Severity:** warning
- **Detect:** Check schema definitions and migration files for columns used in `WHERE`, `JOIN`,
  `ORDER BY`, `GROUP BY` that lack index definitions. In ORMs, look for `@Index`, `add_index`,
  `index: true`, `.createIndex()`. Flag foreign key columns without indexes.
- **Deduction:** -4 points
- **Roast (en):** "Querying `user_id` on a million-row table with no index. That's not a query — that's a full table scan cosplaying as a feature."
  - **Roast (ja):** "100万行のテーブルでインデックス無しで`user_id`検索してるんですけど、それってフルテーブルスキャンですよね。図書館で索引使わずに全ページめくってる人と同じなんですよ。"
- **Fix:** Add indexes on all foreign key columns, frequently queried fields, and columns used in `WHERE`/`ORDER BY`. Use composite indexes for multi-column queries. Run `EXPLAIN ANALYZE` to identify slow queries.

### Unoptimized Images

- **Severity:** info
- **Detect:** Use Glob to find image files (`*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.bmp`,
  `*.svg`) in source directories. Flag files larger than 500KB. Check for missing
  `width`/`height` on `<img>` tags. Look for raster images where SVG would suffice.
- **Deduction:** -1 point
- **Roast (en):** "A 4MB PNG in your source tree. That's not an image — that's a hostage situation for mobile users watching their data plan evaporate pixel by pixel."
  - **Roast (ja):** "ソースツリーに4MBのPNGがあるんですけど、それってモバイルユーザーのギガを人質に取ってるのと同じですよね。WebPって聞いたことないですかね。"
- **Fix:** Compress images with `sharp`, `imagemin`, or Squoosh. Use WebP/AVIF formats with fallbacks. Implement responsive images with `srcset`. Lazy-load below-the-fold images. Set explicit `width`/`height` to prevent layout shift.

### Callback Hell

- **Severity:** warning
- **Detect:** Grep for nested `.then()` chains (3+ levels). Patterns: `\.then\(.*\n.*\.then\(.*\n.*\.then\(` or nested callbacks `function.*\{[^}]*function.*\{[^}]*function`. Also flag deeply nested `(err, data) =>` patterns typical of legacy Node.js callback APIs.
- **Deduction:** -4 points
- **Roast (en):** "Nested `.then().then().then()` — the promise pyramid of doom. `async/await` has been stable since 2017. That's not legacy — that's a lifestyle choice."
  - **Roast (ja):** "`.then().then().then()`のネスト、約束のピラミッドですよね。`async/await`は2017年から使えるんですけど、あえて使わない理由あるんですかね。"
- **Fix:** Convert to `async/await`. Replace `.then()` chains with `const result = await asyncFn()`. Use `try/catch` for error handling. Combine parallel operations with `Promise.all()`.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Multiply final score by weight 1.0x.
