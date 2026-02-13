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
- **Roast:** "You're querying the database in a loop. That's not a feature, that's a DDoS on yourself. Your database is crying while you hit it 10,000 times for something one JOIN could solve."
- **Fix:** Use eager loading (`include`, `populate`, `prefetch_related`, `joinedload`), batch
  queries with `WHERE id IN (...)`, or restructure with JOINs. Fetch all needed data in a
  single query and process in-memory.

### Synchronous File I/O

- **Severity:** error
- **Detect:** Grep for `readFileSync`, `writeFileSync`, `appendFileSync`, `existsSync`,
  `mkdirSync`, `readdirSync` in non-CLI code. Check if the file is a server entry point,
  route handler, or middleware. Exclude scripts, CLI tools, and build configs.
- **Deduction:** -10 points
- **Roast:** "readFileSync in a server? Hope your users enjoy waiting in line like it's the DMV. Node gave you async I/O as a superpower and you said 'no thanks, I prefer blocking.'"
- **Fix:** Use async equivalents (`readFile`, `writeFile` from `fs/promises`) with `await`.
  For streams, use `createReadStream`/`createWriteStream`. Sync I/O is acceptable only in
  CLI tools, build scripts, or one-time startup config loading.

### Missing Pagination

- **Severity:** error
- **Detect:** Grep for `SELECT` without `LIMIT`/`OFFSET`/`TOP`. Check ORM calls like
  `.findAll()`, `.find({})`, `.all()`, `.list()` without `.limit()`, `.take()`, `.paginate()`.
  Flag API endpoints returning unbounded arrays.
- **Deduction:** -10 points
- **Roast:** "SELECT * FROM everything -- bold move when the table has a million rows. Your server RAM is not infinite, no matter how much your cloud bill suggests otherwise."
- **Fix:** Add `LIMIT`/`OFFSET` or cursor-based pagination to all list queries. Return pagination metadata (`total`, `page`, `per_page`, `next_cursor`). Set a sensible default limit (20-100).

### Memory Leaks

- **Severity:** error
- **Detect:** Grep for `addEventListener` without corresponding `removeEventListener`,
  `setInterval` without `clearInterval`, `setTimeout` accumulation in loops, event emitter
  `.on()` without `.off()`. Check for missing cleanup in React `useEffect` return functions.
- **Deduction:** -10 points
- **Roast:** "You're adding event listeners like a squirrel hoarding nuts -- and never cleaning up. Give it a few hours and your process will be fatter than a Thanksgiving turkey right before the OOM killer shows up."
- **Fix:** Always pair `addEventListener` with `removeEventListener`. Clear intervals and
  timeouts on unmount or process exit. Use `WeakMap`/`WeakRef` for caches. In React, return
  cleanup functions from `useEffect`. Monitor memory with `--inspect` and heap snapshots.

### Blocking the Event Loop

- **Severity:** error
- **Detect:** Grep for heavy computation in server code: `JSON.parse` on large inputs without
  streaming, nested loops with O(n^2)+ complexity in handlers, `crypto.pbkdf2Sync`,
  `crypto.scryptSync`, `while(true)` loops, and CPU-intensive regex (ReDoS).
- **Deduction:** -10 points
- **Roast:** "You're running O(n^3) computations on the main thread of a Node.js server. Every other user is frozen in time while your algorithm finishes its world tour. Worker threads exist. Use them."
- **Fix:** Offload CPU-intensive work to worker threads, child processes, or a job queue
  (Bull, BullMQ). Break large synchronous operations into chunks using `setImmediate`. Use
  streaming JSON parsers for large payloads. Avoid synchronous crypto in request handlers.

### Unbatched Operations

- **Severity:** warning
- **Detect:** Look for multiple sequential `await` statements where the results are independent
  of each other. Consecutive `await` calls where none uses the return value of the previous
  one. Also check for `await` inside `forEach` (which doesn't actually await).
- **Deduction:** -4 points
- **Roast:** "Three sequential awaits that don't depend on each other. Promise.all is right there, waving at you, begging to be used. You're turning a 200ms operation into a 600ms nap."
- **Fix:** Use `Promise.all()` for independent async operations. Use `Promise.allSettled()` when individual failures should not abort others. For rate-limited APIs, use `p-limit` or `p-queue`.

### Large Bundle Imports

- **Severity:** warning
- **Detect:** Grep for full-library imports of known heavy packages: `import _ from 'lodash'`,
  `require('lodash')`, `import moment from 'moment'`, `import * as Rx from 'rxjs'`. Also
  flag `import * as` from large utility libraries without tree-shaking evidence.
- **Deduction:** -4 points
- **Roast:** "import _ from 'lodash' -- congratulations, you just imported 70KB for one function. Your bundle is so bloated it needs its own CDN. Ever heard of 'lodash/get'? It's called a subpath import."
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
- **Roast:** "You're computing the exact same expensive result on every single request. Your server is like a goldfish -- no memory of anything it's ever done before. A little caching goes a long way, unless you enjoy burning CPU for fun."
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
- **Roast:** "console.log('here') -- the debugging equivalent of leaving Post-its on the Mona Lisa. Your production logs look like a teenager's diary: 'here', 'wtf', 'why is this undefined', and the classic 'REMOVE THIS BEFORE DEPLOY'. Narrator: they did not remove it."
- **Fix:** Use a structured logging library (pino, winston) with log levels. Strip `console.log` with an ESLint rule (`no-console`) or a build-time transform. Use conditional debug logging (`debug` package) toggled via environment variables.

### Missing Database Indexes

- **Severity:** warning
- **Detect:** Check schema definitions and migration files for columns used in `WHERE`, `JOIN`,
  `ORDER BY`, `GROUP BY` that lack index definitions. In ORMs, look for `@Index`, `add_index`,
  `index: true`, `.createIndex()`. Flag foreign key columns without indexes.
- **Deduction:** -4 points
- **Roast:** "You're querying on `user_id` with a million rows and no index. That's not a query, that's a full table scan masquerading as a feature. Your database is doing more work than an unpaid intern on deadline day."
- **Fix:** Add indexes on all foreign key columns, frequently queried fields, and columns used in `WHERE`/`ORDER BY`. Use composite indexes for multi-column queries. Run `EXPLAIN ANALYZE` to identify slow queries.

### Unoptimized Images

- **Severity:** info
- **Detect:** Use Glob to find image files (`*.png`, `*.jpg`, `*.jpeg`, `*.gif`, `*.bmp`,
  `*.svg`) in source directories. Flag files larger than 500KB. Check for missing
  `width`/`height` on `<img>` tags. Look for raster images where SVG would suffice.
- **Deduction:** -1 point
- **Roast:** "You've got a 4MB PNG in your source tree. That's not an image, that's a short film. Your users on mobile are watching their data plan evaporate while your hero image loads one pixel at a time."
- **Fix:** Compress images with `sharp`, `imagemin`, or Squoosh. Use WebP/AVIF formats with fallbacks. Implement responsive images with `srcset`. Lazy-load below-the-fold images. Set explicit `width`/`height` to prevent layout shift.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Multiply final score by weight 1.0x.
