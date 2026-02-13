# Naming Convention Checks

## Activation

Always active.

## Check Items

### Single-Letter Variables

- **Severity:** error
- **Detect:** Use Grep for `(const|let|var)\s+[a-z]\s*[=:]` to find single-letter variable declarations.
  Exclude loop iterators (`for (let i`), short lambda params (`=> x`), and catch blocks (`catch (e)`).
- **Deduction:** -10 points
- **Roast:** "You named this variable `d`. Is it a date? A dog? A dimension? A disappointment?"
- **Fix:** Replace with a descriptive name. `d` becomes `deadline`, `x` becomes `coordinateX`.

### Generic Names

- **Severity:** error
- **Detect:** Use Grep for standalone identifiers:
  `(const|let|var)\s+(data|info|result|temp|obj|val|item|value|stuff|thing)\s*[=:]`.
  Flag when used as-is without a qualifying prefix or suffix.
  Composite names like `userData` or `validationResult` are acceptable.
- **Deduction:** -10 points
- **Roast:** "You named this `data`. What data? All data? The concept of data itself?"
- **Fix:** Add context. `data` becomes `userData`, `result` becomes `validationResult`.

### Misleading Names

- **Severity:** error
- **Detect:** Check booleans lacking `is`/`has`/`should`/`can`/`will` prefix.
  Check function names that imply a pure read (e.g., `get*`, `find*`, `fetch*`)
  but contain side effects (writes, mutations, API calls).
  Cross-reference the name's implied behavior with the function body.
- **Deduction:** -10 points
- **Roast:** "This function is called `getUser` but it also deletes the session, sends an email, and launches a missile. Quite the getter."
- **Fix:** Boolean: add `is`/`has`/`should` prefix. Functions: rename to match actual behavior or split into separate functions.

### Inconsistent Casing

- **Severity:** warning
- **Detect:** Scan the file for variable and function declarations.
  Use Grep to find both `[a-z]+_[a-z]+` (snake_case) and `[a-z]+[A-Z][a-z]+` (camelCase)
  patterns in the same file.
  Ignore external library identifiers, constants in UPPER_SNAKE_CASE, and class names in PascalCase.
  Flag when both camelCase and snake_case appear in user-defined variable or function names.
- **Deduction:** -4 points
- **Roast:** "Half your variables are camelCase, the other half are snake_case. Pick a lane — this isn't a bilingual codebase."
- **Fix:** Choose one convention per language (camelCase for JS/TS, snake_case for Python/Rust) and enforce with a linter rule.

### Abbreviated Names

- **Severity:** warning
- **Detect:** Use Grep for common abbreviations in identifiers:
  `(usr|msg|btn|cfg|mgr|impl|req|res|err|cb|fn|ctx|evt|elem|attr|prop|prev|cur|num|str|arr|idx)`
  as whole words or segments in variable and function names.
  Accept abbreviations that are industry-standard within their context
  (e.g., `req`/`res` in Express middleware, `ctx` in Go).
- **Deduction:** -4 points
- **Roast:** "Life is too short for abbreviations. Actually no, variable names should be long enough."
- **Fix:** Spell it out. `usr` becomes `user`, `msg` becomes `message`, `btn` becomes `button`. Your IDE has autocomplete — use it.

### Hungarian Notation

- **Severity:** info
- **Detect:** Use Grep for type-prefix patterns in variable declarations:
  `(str|int|bool|arr|obj|fn|num|chr|dbl|flt)[A-Z]\w+`.
  Also flag `b[A-Z]\w+` for booleans and `i[A-Z]\w+` for integers
  when the prefix clearly encodes the type rather than being a natural word.
- **Deduction:** -1 point
- **Roast:** "Welcome to 1995. We have Hungarian notation and dial-up internet."
- **Fix:** Remove the type prefix. Modern languages and IDEs provide type information.
  `strName` becomes `name`, `iCount` becomes `count`, `bFlag` becomes `isEnabled`.

### Meaningless Prefixes

- **Severity:** warning
- **Detect:** Use Grep for `(const|let|var)\s+(my|the|a|an|this_)[A-Z]\w+` and similar patterns.
  Flag variables prefixed with articles or possessives that add zero semantic value.
  Exclude legitimate uses like `aNodeInTree` where `a` clarifies cardinality.
- **Deduction:** -4 points
- **Roast:** "Adding 'my' to a variable doesn't make it yours — it makes it confusing."
- **Fix:** Drop the prefix. `myData` becomes `userData` or just `data` with proper scope. `theList` becomes `orders`.

### File Name Mismatch

- **Severity:** warning
- **Detect:** Read the file to find the default export (`export default`) or `module.exports` assignment.
  Compare the exported identifier name with the filename (minus extension).
  Account for case conventions — a PascalCase class in a PascalCase or kebab-case filename is fine.
  Flag when the export name and filename are semantically unrelated.
- **Deduction:** -4 points
- **Roast:** "The file is called `utils.ts` but the default export is `AuthenticationService`. That's not a util, that's an identity crisis."
- **Fix:** Rename the file to match the default export, or rename the export to match the file. One source of truth.

### Overly Long Names

- **Severity:** info
- **Detect:** Use Grep for identifiers exceeding 40 characters:
  `[a-zA-Z_$][a-zA-Z0-9_$]{40,}`.
  Check variable, function, class, and type declarations.
  Accept long names in test descriptions and string constants.
- **Deduction:** -1 point
- **Roast:** "This variable name is so long it needs its own scroll bar. Descriptive is good; a novel title is not."
- **Fix:** Shorten while preserving meaning.
  `getUserAuthenticationTokenFromExternalIdentityProvider` becomes `getExternalAuthToken`.

### Pluralization Errors

- **Severity:** warning
- **Detect:** Find array/list declarations (type annotations like `[]`, `Array<>`, `Set<>`, `Map<>`,
  or initializations with `[`) where the variable name is singular.
  Also flag single-value variables with plural names.
  Use Grep to cross-reference the identifier name with its type annotation
  or assigned value to determine cardinality.
- **Deduction:** -4 points
- **Roast:** "You named an array `item`. One item? No, it's an array. Arrays have friends — they deserve an 's'."
- **Fix:** Arrays get plural names (`items`, `users`). Single values get singular names (`item`, `user`). Match the name to the cardinality.

### Negative Booleans

- **Severity:** warning
- **Detect:** Use Grep for boolean-like identifiers with negative phrasing:
  `(isNot|isNo|not[A-Z]|disable|disabled|no[A-Z]|without|exclude|never|dont|lack)`
  in variable declarations, function parameters, and object properties.
  Especially flag double-negative usage patterns like `!isNotReady` or `!disableFeature`
  by searching for `!\s*(isNot|isNo|disable|not[A-Z])`.
- **Deduction:** -4 points
- **Roast:** "Double negatives in boolean names — `!isNotDisabled` is not not confusing."
- **Fix:** Invert the name to a positive form. `isNotReady` becomes `isReady`, `disableFeature` becomes `featureEnabled`. Flip the logic to match.

## Scoring

Start at 100. Apply deductions per finding. Minimum score is 0. Weight: 1.0x.
