# Complexity Checks

## Activation

Always active.

## Check Items

### Long Functions

- **Severity:** error
- **Detect:** Count lines between function declaration and closing brace. Use Grep to find function declarations, then Bash `awk` to measure line count. Flag any function exceeding 50 lines.
- **Deduction:** -10 points
- **Roast:** "This function is longer than a CVS receipt. Did you forget functions can call other functions?"
- **Fix:** Extract logical blocks into smaller, well-named helper functions. Each function should do one thing.

### High Cyclomatic Complexity

- **Severity:** error
- **Detect:** Use Grep to count branching keywords (`if`, `else if`, `else`, `switch`, `case`, `&&`, `||`, `?`) within each function body. Flag when the total exceeds 10.
- **Deduction:** -10 points
- **Roast:** "This function has more branches than a national park."
- **Fix:** Decompose into smaller functions, use early returns, replace conditionals with polymorphism or strategy patterns.

### Deep Callback Nesting

- **Severity:** error
- **Detect:** Use Grep with multiline mode to find callback patterns (anonymous functions passed as arguments) nested more than 3 levels deep. Check indentation depth as a heuristic.
- **Deduction:** -10 points
- **Roast:** "Welcome to callback hell — Satan called, he wants his indentation back."
- **Fix:** Refactor to async/await, Promises, or named functions. Flatten the nesting.

### Nested Ternaries

- **Severity:** warning
- **Detect:** Use Grep to find lines containing two or more `?` operators that form ternary-inside-ternary patterns (e.g., `? ... ? ... : ... : ...`).
- **Deduction:** -4 points
- **Roast:** "This ternary has ternaries. It's ternaries all the way down."
- **Fix:** Replace with `if/else` blocks or extract conditions into named variables for readability.

### Long Parameter Lists

- **Severity:** warning
- **Detect:** Use Grep to find function declarations/signatures. Count commas between parentheses. Flag functions with more than 5 parameters.
- **Deduction:** -4 points
- **Roast:** "This function takes more arguments than a debate team."
- **Fix:** Group related parameters into an options object or configuration type.

### Boolean Parameters

- **Severity:** warning
- **Detect:** Use Grep to find function parameters with boolean type annotations (`: boolean`) or default values (`= true`, `= false`). Cross-reference with `if` statements that branch on those params.
- **Deduction:** -4 points
- **Roast:** "A boolean parameter is just a polite way of saying 'this function does two things'."
- **Fix:** Split into two separate functions with clear names, or use an enum/string literal for intent.

### Complex Regular Expressions

- **Severity:** warning
- **Detect:** Use Grep to find regex literals (`/.../`) or `new RegExp(...)` constructs. Use Bash to measure character length. Flag any regex exceeding 80 characters that lacks an accompanying comment on the same or previous line.
- **Deduction:** -4 points
- **Roast:** "This regex looks like your cat walked on the keyboard — and somehow it works."
- **Fix:** Break into named sub-patterns with comments, or use a regex builder library for readability.

### Excessive Switch Cases

- **Severity:** warning
- **Detect:** Use Grep to find `switch` statements, then count the number of `case` keywords within each switch block. Flag when case count exceeds 10.
- **Deduction:** -4 points
- **Roast:** "This switch statement has more cases than a lawyer's filing cabinet."
- **Fix:** Replace with a lookup map/object, polymorphism, or a registry pattern.

### Multiple Return Types

- **Severity:** warning
- **Detect:** Analyze function bodies for `return` statements. Use Grep to find all returns within a function and check if they return different types (e.g., string vs number, object vs null vs undefined). TypeScript union return types with more than 2 variants are also suspect.
- **Deduction:** -4 points
- **Roast:** "This function is an identity crisis — it returns a string, a number, and sometimes just gives up and returns undefined."
- **Fix:** Return a consistent type. Use Result/Either patterns or throw errors instead of returning mixed types.

### Magic Numbers

- **Severity:** info
- **Detect:** Use Grep to find numeric literals (excluding 0, 1, -1, and common ports) used directly in logic, comparisons, or assignments without a named constant. Check that no `const` declaration on the same or previous line gives it a name.
- **Deduction:** -1 point
- **Roast:** "What does 86400 mean? Oh right, seconds in a day. Thanks for making me do math."
- **Fix:** Extract into a named constant (e.g., `const SECONDS_IN_DAY = 86400`).

### Deeply Nested Objects

- **Severity:** warning
- **Detect:** Use Grep to find property access chains with 4 or more dots (e.g., `a.b.c.d.e`). Pattern: `\w+(\.\w+){4,}` — flag any match.
- **Deduction:** -4 points
- **Roast:** "You're drilling so deep into this object you're about to hit magma. Ever heard of destructuring?"
- **Fix:** Use destructuring, intermediate variables, or optional chaining with early extraction to flatten access depth.

## Scoring

Start at 100. Apply deductions per finding. Minimum score is 0. Weight: 1.0x.
