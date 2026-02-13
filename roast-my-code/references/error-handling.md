# Error Handling Checks

## Activation

Always active.

## Check Items

### Empty Catch Blocks

- **Severity:** critical
- **Detect:** Grep for `catch\s*\(.*\)\s*\{\s*\}` and `catch\s*\{\s*\}`. Also check for catch blocks containing only comments (`catch\s*\(.*\)\s*\{\s*//.*\s*\}`). Scan `.js`, `.ts`, `.jsx`, `.tsx`, `.java`, `.cs`, `.go` files. In Python, look for `except.*:\s*pass` with nothing else in the block.
- **Deduction:** -20 points
- **Roast:** "You're catching errors and feeding them to the void. The void says thanks. Your catch block is emptier than your commitment to reliability. Errors are screaming for help and you're putting them on mute. This is the software equivalent of a smoke detector with no batteries."
- **Fix:** At minimum, re-throw the error, log it with context, or handle it with a user-facing fallback. If the error is intentionally ignored, add an explicit comment explaining why (e.g., `// Expected during shutdown`). Consider using a linting rule like `no-empty` to prevent this pattern from ever reaching production.

### Catch-and-Log-Only

- **Severity:** warning
- **Detect:** Grep for catch blocks whose body contains only `console.log`, `console.error`, `console.warn`, `logger.error`, or `print` with no additional recovery logic, re-throw, or return statement. Pattern: `catch\s*\(.*\)\s*\{[\s\n]*console\.(log|error|warn)\(.*\);?\s*\}`. In Python, check for `except.*:\s*print\(` or `except.*:\s*logging\.\w+\(` with no subsequent logic.
- **Deduction:** -4 points
- **Roast:** "Logging an error and moving on is not handling -- it's journaling. Dear diary, today I caught an exception and did absolutely nothing about it. The user? Still broken. But at least your logs are thriving. Somewhere a monitoring dashboard has a beautiful spike and zero remediation."
- **Fix:** After logging, either re-throw the error, return a meaningful fallback value, show a user-friendly message, or trigger a retry mechanism. Logging is a side effect of error handling, not error handling itself. Define a clear recovery strategy for each catch block.

### Generic Error Messages

- **Severity:** warning
- **Detect:** Grep for `new Error\(["']error["']\)`, `new Error\(["']something went wrong["']\)`, `new Error\(["']an error occurred["']\)`, `new Error\(["']unknown error["']\)`, `new Error\(["']fail["']\)`, and `throw new Error\(\)` with no message at all. Case-insensitive matching. Also flag single-word error messages that provide no actionable context.
- **Deduction:** -4 points
- **Roast:** "Your error message is 'something went wrong'. Gee, thanks Sherlock. What a groundbreaking diagnostic. You could have written 'oopsie daisy' and it would contain exactly the same amount of useful information: zero. Future-you debugging this at 2 AM will have strong words for present-you."
- **Fix:** Include specific context in error messages: what operation failed, what input caused it, and what the user or developer can do about it. Example: `new Error('Failed to parse config file at /etc/app/config.json: invalid JSON at line 42')`. Good error messages are documentation for your future self.

### No Error Boundaries

- **Severity:** error
- **Detect:** Identify React projects by checking for `react` or `react-dom` in `package.json` dependencies. Search for `ErrorBoundary`, `componentDidCatch`, `getDerivedStateFromError`, or error boundary libraries like `react-error-boundary`. If none are found in a React project with multiple component files, flag it.
- **Deduction:** -10 points
- **Roast:** "A React app with no error boundaries is a house of cards in an earthquake. One bad render and your entire component tree goes white-screen-of-death. Your users get to stare at a blank page and contemplate the meaning of nothing. A single undefined property and the whole UI disintegrates like Thanos snapped it."
- **Fix:** Wrap major UI sections with `<ErrorBoundary>` components. Use `react-error-boundary` for a battle-tested implementation with reset capabilities. At minimum, wrap the top-level `<App>` component. Provide user-friendly fallback UIs that offer recovery actions like "Try again" or "Go back to home."

### Unhandled Promise Rejections

- **Severity:** error
- **Detect:** Grep for `.then\(` without a corresponding `.catch\(` in the promise chain. Check for `await` expressions not wrapped in `try/catch`. Pattern: standalone `await` calls in async functions without any surrounding try-catch block. Also flag `new Promise` constructors that never call `reject` despite having failure paths, and floating promises (async calls without `await`).
- **Deduction:** -10 points
- **Roast:** "An unhandled promise rejection is just an exception with commitment issues. It promised it would resolve, it didn't, and nobody was there to catch it falling. In Node.js, this literally crashes your process since v15. But sure, live dangerously -- your uptime was overrated anyway."
- **Fix:** Always add `.catch()` at the end of promise chains. Wrap `await` calls in `try/catch`. Use a global handler (`process.on('unhandledRejection')` or `window.addEventListener('unhandledrejection')`) as a safety net, not a primary strategy. Consider using the `no-floating-promises` ESLint rule for compile-time prevention.

### Swallowed Errors in Callbacks

- **Severity:** error
- **Detect:** Grep for Node-style callback patterns where the `err` parameter is declared but never referenced in the body: `function\s*\(err[,)]` or `\(err\s*,` where `err` (or `error`) does not appear in the function body. Also flag `if (err)` blocks that are empty or only contain a bare `return` with no logging or propagation. Check `fs.readFile`, `fs.writeFile`, and similar Node APIs.
- **Deduction:** -10 points
- **Roast:** "You accept an error parameter in your callback and then ghosted it. That error showed up to the party, and you pretended it wasn't there. It's still there. It's always been there. Your app just doesn't know it yet. Meanwhile, data is silently corrupted and you're whistling past the graveyard."
- **Fix:** Always check the `err` parameter first in callbacks. If an error occurred, handle it by propagating it to the caller, logging with context, or returning an appropriate error response. Migrate to async/await with `util.promisify` where possible to use structured try/catch instead of callback error parameters.

### String Throws

- **Severity:** warning
- **Detect:** Grep for `throw\s+["']` and `throw\s+` followed by a string literal or template literal rather than a `new Error` expression. Pattern: `throw\s+["'\`]` without `new\s+\w*Error`. In Python, check for`raise\s+["']` which is a syntax error but sometimes attempted.
- **Deduction:** -4 points
- **Roast:** "Throwing a string is the error-handling equivalent of passing a note in class. No stack trace, no error type, no `instanceof` check possible. Just a sad lonely string floating through your call stack with no identity and no way to trace where it came from."
- **Fix:** Always throw `Error` objects or custom error classes: `throw new Error('message')`. This preserves stack traces and allows `instanceof` checks for typed error handling. Create custom error classes for domain-specific errors: `class ValidationError extends Error { constructor(field, message) { super(message); this.field = field; } }`.

### Missing Finally Blocks

- **Severity:** info
- **Detect:** Look for resource acquisition patterns (file handles, database connections, network sockets, locks, streams) followed by `try/catch` without a `finally` block. Grep for `try\s*\{` without a corresponding `finally` in the same block. Focus on code involving `open`, `connect`, `acquire`, `createConnection`, `createReadStream`, `createWriteStream`, `lock`, and database pool `getConnection`.
- **Deduction:** -1 point
- **Roast:** "You open a resource, use it in a try block, and never clean up in finally. Congrats, you've invented a slow memory leak. Your server will die not with a bang, but with an 'EMFILE: too many open files' whimper at 3 AM on a Saturday."
- **Fix:** Use `finally` blocks to release resources regardless of success or failure. Better yet, use language patterns that handle cleanup automatically: `using` in C#/TypeScript 5.2+, try-with-resources in Java, context managers (`with`) in Python, or wrapper functions that manage the full resource lifecycle.

### Inconsistent Error Types

- **Severity:** warning
- **Detect:** Scan the codebase for a mix of error-throwing styles: count occurrences of `throw new Error`, `throw new \w+Error` (custom errors), `throw "` or `throw '` (string throws), `throw {` (object throws), and `reject("` (string rejections). If more than two distinct styles are used across the project without a clear pattern, flag the inconsistency.
- **Deduction:** -4 points
- **Roast:** "Your codebase throws Error objects in one file, raw strings in another, and plain objects in a third. It's like a potluck where everyone brought a different cuisine and none of it goes together. Pick a strategy. Any strategy. Even a bad consistent strategy beats this chaos."
- **Fix:** Establish a project-wide error hierarchy with a base custom error class. All errors should extend `Error`. Define domain-specific subclasses (`NotFoundError`, `ValidationError`, `AuthError`, `TimeoutError`). Document the convention in your contributing guide and enforce it with custom linting rules or code review checklists.

### No Global Error Handler

- **Severity:** warning
- **Detect:** In Node.js projects, grep for `process.on\(["']uncaughtException` and `process.on\(["']unhandledRejection`. In browser projects, grep for `window.onerror`, `window.addEventListener\(["']error`, and `window.addEventListener\(["']unhandledrejection`. In Express apps, also check for a 4-argument error middleware `\(err,\s*req,\s*res,\s*next\)`. If none are found, flag it.
- **Deduction:** -4 points
- **Roast:** "No global error handler? So when something unexpected blows up, your app just... dies silently into the night? That's not minimalism, that's negligence. You're running a production app with no safety net and calling it 'optimistic engineering.' Your users call it 'unreliable.'"
- **Fix:** Add global handlers as a last line of defense. In Node.js: `process.on('uncaughtException')` and `process.on('unhandledRejection')`. In Express: add a final error-handling middleware. In browsers: `window.addEventListener('error')`. Log the error with full context, report to a monitoring service (Sentry, Datadog, Bugsnag), and exit gracefully if the error is unrecoverable.

### Error Logging Without Context

- **Severity:** warning
- **Detect:** Grep for `console.error\(err\)`, `console.error\(error\)`, `console.error\(e\)`, `logger.error\(err\)` where the error object is the sole argument with no additional context such as request ID, user ID, operation name, or timestamp. Pattern: `console\.error\(\w+\)\s*;` with a single short identifier as the only argument. Also flag `catch(e) { console.error(e.message) }` that strips the stack trace.
- **Deduction:** -4 points
- **Roast:** "Your error log says 'Error: something failed'. Which request? Which user? Which of your 47 microservices? Nobody knows. You've created a murder mystery with no clues. Good luck debugging this at 3 AM with nothing but vibes and a prayer."
- **Fix:** Always include context when logging errors: operation name, relevant IDs (request, user, transaction), input parameters that triggered the failure, and the full stack trace. Use structured logging: `logger.error({ err, requestId, userId, operation: 'processPayment', input: { orderId } })`. Never strip the stack trace by logging only `e.message`.

## Priority Order

When multiple error handling issues are found, report them in this order:

1. **Critical** -- Empty Catch Blocks (silent failure is the worst kind of failure)
2. **Error** -- Unhandled Promise Rejections, No Error Boundaries, Swallowed Errors in Callbacks
3. **Warning** -- Catch-and-Log-Only, Generic Error Messages, String Throws, Inconsistent Error Types, No Global Error Handler, Error Logging Without Context
4. **Info** -- Missing Finally Blocks

## Language-Specific Detection Notes

- **JavaScript/TypeScript:** Focus on promise chains, async/await patterns, and React error boundaries. Check for both CommonJS (`require`) and ESM (`import`) projects.
- **Python:** Look for bare `except:` (catches everything including `KeyboardInterrupt`), `except Exception:` without re-raise, and `pass` in except blocks.
- **Java:** Check for `catch (Exception e)` that is too broad, empty catch blocks, and missing `finally` for `AutoCloseable` resources.
- **Go:** Check for `_ = someFunc()` patterns that discard error returns, and `if err != nil` blocks that do nothing meaningful.

## Interaction with Other Checks

- Empty Catch Blocks + Swallowed Errors often appear together -- count both deductions separately.
- Generic Error Messages + Error Logging Without Context compound to make debugging nearly impossible -- call this out explicitly in the roast as a "double whammy."
- No Error Boundaries is only applicable to React projects -- skip for non-React codebases entirely.
- No Global Error Handler check should adapt to the runtime: Node.js, browser, Deno, Bun each have different APIs for global error capture.
- Catch-and-Log-Only is a subset of poor error handling -- if the catch block also has a generic message, apply both deductions.
- String Throws and Inconsistent Error Types often co-occur -- if string throws are found, also check whether the project mixes error styles.

## Roast Escalation

- 1-2 findings: Light roast. Mention issues conversationally.
- 3-5 findings: Medium roast. Express genuine concern for production reliability.
- 6+ findings: Dark roast. Question whether errors are handled at all. Suggest the codebase treats exceptions like spam emails -- ignored and deleted.

## Scoring

Start at 100. Apply deductions per finding (each occurrence counts separately). Minimum score is 0. Multiply final score by weight 1.0x.
