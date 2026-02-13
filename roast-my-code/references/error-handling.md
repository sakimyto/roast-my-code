# Error Handling Checks

## Activation

Always active.

## Check Items

### Empty Catch Blocks

- **Severity:** critical
- **Detect:** Grep for `catch\s*\(.*\)\s*\{\s*\}` and `catch\s*\{\s*\}`. Also check for catch blocks containing only comments (`catch\s*\(.*\)\s*\{\s*//.*\s*\}`). Scan `.js`, `.ts`, `.jsx`, `.tsx`, `.java`, `.cs`, `.go` files. In Python, look for `except.*:\s*pass` with nothing else in the block.
- **Deduction:** -20 points
- **Roast (en):** "You catch errors and feed them directly to the void. `catch (e) {}` — the software equivalent of a smoke detector with no batteries."
  - **Roast (ja):** "なんだろう、エラーをcatchして何もしないの、郵便受けの中身を読まずにシュレッダーにかけるのと同じですよね。せめて開封してもらっていいですか。"
- **Fix:** At minimum, re-throw the error, log it with context, or handle it with a user-facing fallback. If the error is intentionally ignored, add an explicit comment explaining why (e.g., `// Expected during shutdown`). Consider using a linting rule like `no-empty` to prevent this pattern from ever reaching production.

### Catch-and-Log-Only

- **Severity:** warning
- **Detect:** Grep for catch blocks whose body contains only `console.log`, `console.error`, `console.warn`, `logger.error`, or `print` with no additional recovery logic, re-throw, or return statement. Pattern: `catch\s*\(.*\)\s*\{[\s\n]*console\.(log|error|warn)\(.*\);?\s*\}`. In Python, check for `except.*:\s*print\(` or `except.*:\s*logging\.\w+\(` with no subsequent logic.
- **Deduction:** -4 points
- **Roast (en):** "`console.error(e)` and move on. That's not error handling — that's journaling. 'Dear diary, today I caught an exception and did nothing about it.'"
  - **Roast (ja):** "それってエラーハンドリングじゃなくて日記ですよね。『今日もエラーが出ました。おしまい。』って書いてるだけですよ。対処してください。"
- **Fix:** After logging, either re-throw the error, return a meaningful fallback value, show a user-friendly message, or trigger a retry mechanism. Logging is a side effect of error handling, not error handling itself. Define a clear recovery strategy for each catch block.

### Generic Error Messages

- **Severity:** warning
- **Detect:** Grep for `new Error\(["']error["']\)`, `new Error\(["']something went wrong["']\)`, `new Error\(["']an error occurred["']\)`, `new Error\(["']unknown error["']\)`, `new Error\(["']fail["']\)`, and `throw new Error\(\)` with no message at all. Case-insensitive matching. Also flag single-word error messages that provide no actionable context.
- **Deduction:** -4 points
- **Roast (en):** "Your error message is 'something went wrong'. Truly groundbreaking diagnostics. Future-you debugging this at 3 AM will have some words for present-you."
  - **Roast (ja):** "エラーメッセージが'something went wrong'って、病院行って『どこか悪いです』って言うのと同じですよね。もうちょっと情報ください。"
- **Fix:** Include specific context in error messages: what operation failed, what input caused it, and what the user or developer can do about it. Example: `new Error('Failed to parse config file at /etc/app/config.json: invalid JSON at line 42')`. Good error messages are documentation for your future self.

### Unhandled Promise Rejections

- **Severity:** error
- **Detect:** Grep for `.then\(` without a corresponding `.catch\(` in the promise chain. Check for `await` expressions not wrapped in `try/catch`. Pattern: standalone `await` calls in async functions without any surrounding try-catch block. Also flag `new Promise` constructors that never call `reject` despite having failure paths, and floating promises (async calls without `await`).
- **Deduction:** -10 points
- **Roast (en):** "An unhandled promise rejection is just an exception with commitment issues — it promised to resolve, didn't, and nobody was there to catch it."
  - **Roast (ja):** "なんだろう、Promiseをcatchしないの、既読スルーと同じですよね。相手は返事待ってるのに無視してるんですよ。いつかクラッシュで返事きますよ。"
- **Fix:** Always add `.catch()` at the end of promise chains. Wrap `await` calls in `try/catch`. Use a global handler (`process.on('unhandledRejection')` or `window.addEventListener('unhandledrejection')`) as a safety net, not a primary strategy. Consider using the `no-floating-promises` ESLint rule for compile-time prevention.

### Swallowed Errors in Callbacks

- **Severity:** error
- **Detect:** Grep for Node-style callback patterns where the `err` parameter is declared but never referenced in the body: `function\s*\(err[,)]` or `\(err\s*,` where `err` (or `error`) does not appear in the function body. Also flag `if (err)` blocks that are empty or only contain a bare `return` with no logging or propagation. Check `fs.readFile`, `fs.writeFile`, and similar Node APIs.
- **Deduction:** -10 points
- **Roast (en):** "You accept an error callback parameter and then ghost it completely. The error showed up, you pretended it wasn't there. It's still there."
  - **Roast (ja):** "コールバックでerrを受け取っておいて完全無視って、それって着信拒否してるのと同じですよね。エラーはちゃんと出てるんですよ、見えないフリしてるだけで。"
- **Fix:** Always check the `err` parameter first in callbacks. If an error occurred, handle it by propagating it to the caller, logging with context, or returning an appropriate error response. Migrate to async/await with `util.promisify` where possible to use structured try/catch instead of callback error parameters.

### String Throws

- **Severity:** warning
- **Detect:** Grep for `throw\s+["']` and `throw\s+` followed by a string literal or template literal rather than a `new Error` expression. Pattern: `throw\s+["'\`]` without `new\s+\w*Error`. In Python, check for`raise\s+["']` which is a syntax error but sometimes attempted.
- **Deduction:** -4 points
- **Roast (en):** "Throwing a raw string. No stack trace, no error type, no dignity. Just a sad lonely string floating through the call stack with no identity."
  - **Roast (ja):** "文字列をthrowするのやめてもらっていいですか。スタックトレースもない、型もない、身元不明の文字列がコールスタックを彷徨ってますよ。"
- **Fix:** Always throw `Error` objects or custom error classes: `throw new Error('message')`. This preserves stack traces and allows `instanceof` checks for typed error handling. Create custom error classes for domain-specific errors: `class ValidationError extends Error { constructor(field, message) { super(message); this.field = field; } }`.

### Missing Finally Blocks

- **Severity:** info
- **Detect:** Look for resource acquisition patterns (file handles, database connections, network sockets, locks, streams) followed by `try/catch` without a `finally` block. Grep for `try\s*\{` without a corresponding `finally` in the same block. Focus on code involving `open`, `connect`, `acquire`, `createConnection`, `createReadStream`, `createWriteStream`, `lock`, and database pool `getConnection`.
- **Deduction:** -1 point
- **Roast (en):** "You open a resource and never clean up in `finally`. Congrats on the slow memory leak. Your server will die not with a bang but with `EMFILE: too many open files`."
  - **Roast (ja):** "リソースを開いて`finally`で閉じないの、水道出しっぱなしで外出するのと同じですよね。いつか`EMFILE`で溺れますよ。"
- **Fix:** Use `finally` blocks to release resources regardless of success or failure. Better yet, use language patterns that handle cleanup automatically: `using` in C#/TypeScript 5.2+, try-with-resources in Java, context managers (`with`) in Python, or wrapper functions that manage the full resource lifecycle.

### Inconsistent Error Types

- **Severity:** warning
- **Detect:** Scan the codebase for a mix of error-throwing styles: count occurrences of `throw new Error`, `throw new \w+Error` (custom errors), `throw "` or `throw '` (string throws), `throw {` (object throws), and `reject("` (string rejections). If more than two distinct styles are used across the project without a clear pattern, flag the inconsistency.
- **Deduction:** -4 points
- **Roast (en):** "You throw `Error` in one file, a raw string in another, and a plain object in a third. Your error handling has multiple personality disorder."
  - **Roast (ja):** "ファイルごとにErrorだったり文字列だったりオブジェクトだったり、エラーの投げ方が統一されてないんですよ。それって会議で全員違う言語で喋ってるのと同じですよね。"
- **Fix:** Establish a project-wide error hierarchy with a base custom error class. All errors should extend `Error`. Define domain-specific subclasses (`NotFoundError`, `ValidationError`, `AuthError`, `TimeoutError`). Document the convention in your contributing guide and enforce it with custom linting rules or code review checklists.

### No Global Error Handler

- **Severity:** warning
- **Detect:** In Node.js projects, grep for `process.on\(["']uncaughtException` and `process.on\(["']unhandledRejection`. In browser projects, grep for `window.onerror`, `window.addEventListener\(["']error`, and `window.addEventListener\(["']unhandledrejection`. In Express apps, also check for a 4-argument error middleware `\(err,\s*req,\s*res,\s*next\)`. If none are found, flag it.
- **Deduction:** -4 points
- **Roast (en):** "No global error handler. When something unexpected explodes, your app just... dies silently into the night. That's not minimalism — that's negligence."
  - **Roast (ja):** "グローバルエラーハンドラがないの、番犬のいない家と同じですよね。泥棒が入っても誰も気づかないんですよ。それをミニマリストとは言わないです。"
- **Fix:** Add global handlers as a last line of defense. In Node.js: `process.on('uncaughtException')` and `process.on('unhandledRejection')`. In Express: add a final error-handling middleware. In browsers: `window.addEventListener('error')`. Log the error with full context, report to a monitoring service (Sentry, Datadog, Bugsnag), and exit gracefully if the error is unrecoverable.

### Error Logging Without Context

- **Severity:** warning
- **Detect:** Grep for `console.error\(err\)`, `console.error\(error\)`, `console.error\(e\)`, `logger.error\(err\)` where the error object is the sole argument with no additional context such as request ID, user ID, operation name, or timestamp. Pattern: `console\.error\(\w+\)\s*;` with a single short identifier as the only argument. Also flag `catch(e) { console.error(e.message) }` that strips the stack trace.
- **Deduction:** -4 points
- **Roast (en):** "Your error log says 'Error: failed'. Which request? Which user? Which microservice? Nobody knows. You've created a murder mystery with no clues."
  - **Roast (ja):** "エラーログに`console.error(e)`だけって、110番して『事件です』とだけ言って切るのと同じですよね。いつ、どこで、何が起きたか言ってもらっていいですか。"
- **Fix:** Always include context when logging errors: operation name, relevant IDs (request, user, transaction), input parameters that triggered the failure, and the full stack trace. Use structured logging: `logger.error({ err, requestId, userId, operation: 'processPayment', input: { orderId } })`. Never strip the stack trace by logging only `e.message`.

### No Fail-Fast Validation

- **Severity:** warning
- **Detect:** Look for functions that perform operations before validating inputs. Check for API handlers without early input validation — no Zod/Joi/yup schema at the handler entry point. Flag functions where parameter checks (null checks, type checks, bounds checks) appear deep inside the body (20+ lines in) rather than at the top.
- **Deduction:** -4 points
- **Roast (en):** "Your function processes 40 lines of logic before checking if the input is valid. Fail fast — validate at the door, not after the party has started."
  - **Roast (ja):** "入力チェックの前に40行もロジック走らせてるの、無効なデータで途中まで処理進めてから『やっぱダメでした』って無駄ですよね。入口でバリデーションしてもらっていいですか。"
- **Fix:** Validate inputs at the entry point of every function. Use guard clauses for required parameters. Add schema validation (Zod, Joi) at API boundaries. The first lines of a function should be validation, not business logic.

## Priority Order

When multiple error handling issues are found, report them in this order:

1. **Critical** -- Empty Catch Blocks (silent failure is the worst kind of failure)
2. **Error** -- Unhandled Promise Rejections, Swallowed Errors in Callbacks
3. **Warning** -- Catch-and-Log-Only, Generic Error Messages, String Throws, Inconsistent Error Types, No Global Error Handler, Error Logging Without Context, No Fail-Fast Validation
4. **Info** -- Missing Finally Blocks

## Language-Specific Detection Notes

- **JavaScript/TypeScript:** Focus on promise chains, async/await patterns, and React error boundaries. Check for both CommonJS (`require`) and ESM (`import`) projects.
- **Python:** Look for bare `except:` (catches everything including `KeyboardInterrupt`), `except Exception:` without re-raise, and `pass` in except blocks.
- **Java:** Check for `catch (Exception e)` that is too broad, empty catch blocks, and missing `finally` for `AutoCloseable` resources.
- **Go:** Check for `_ = someFunc()` patterns that discard error returns, and `if err != nil` blocks that do nothing meaningful.

## Interaction with Other Checks

- Empty Catch Blocks + Swallowed Errors often appear together -- count both deductions separately.
- Generic Error Messages + Error Logging Without Context compound to make debugging nearly impossible -- call this out explicitly in the roast as a "double whammy."
- Error Boundaries are checked in `frontend.md` (not here) to avoid duplicate deductions.
- No Global Error Handler check should adapt to the runtime: Node.js, browser, Deno, Bun each have different APIs for global error capture.
- Catch-and-Log-Only is a subset of poor error handling -- if the catch block also has a generic message, apply both deductions.
- String Throws and Inconsistent Error Types often co-occur -- if string throws are found, also check whether the project mixes error styles.

## Roast Escalation

- 1-2 findings: Light roast. Mention issues conversationally.
- 3-5 findings: Medium roast. Express genuine concern for production reliability.
- 6+ findings: Dark roast. Question whether errors are handled at all. Suggest the codebase treats exceptions like spam emails -- ignored and deleted.

## Scoring

Start at 100. Apply deductions per finding (each occurrence counts separately). Minimum score is 0. Multiply final score by weight 1.0x.
