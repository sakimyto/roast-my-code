# Complexity Checks

## Activation

Always active.

## Check Items

### Long Functions

- **Severity:** error
- **Detect:** Count lines between function declaration and closing brace. Use Grep to find function declarations, then Bash `awk` to measure line count. Flag any function exceeding 50 lines.
- **Deduction:** -10 points
- **Roast (en):** "This function is longer than a CVS receipt and harder to read. Somewhere around line 200, it forgot what it was doing — just like you."
  - **Roast (ja):** "なんだろう、関数が50行超えてるのに分割しないの、やめてもらっていいですか。それってもう関数じゃなくて卒業論文ですよね。"
- **Fix:** Extract logical blocks into smaller, well-named helper functions. Each function should do one thing.

### High Cyclomatic Complexity

- **Severity:** error
- **Detect:** Use Grep to count branching keywords (`if`, `else if`, `else`, `switch`, `case`, `&&`, `||`, `?`) within each function body. Flag when the total exceeds 10.
- **Deduction:** -10 points
- **Roast (en):** "This function has more branches than a national park and twice as many ways to get lost. Even your debugger needs a map."
  - **Roast (ja):** "それって分岐が10個超えてますよね。人間の脳で追える限界、知ってます？　7個ですよ。もう超えてますよね。"
- **Fix:** Decompose into smaller functions, use early returns, replace conditionals with polymorphism or strategy patterns.

### Deep Callback Nesting

- **Severity:** error
- **Detect:** Use Grep with multiline mode to find callback patterns (anonymous functions passed as arguments) nested more than 3 levels deep. Check indentation depth as a heuristic.
- **Deduction:** -10 points
- **Roast (en):** "Welcome to callback hell — population: you, three levels of indentation, and a growing sense of regret. async/await has been out since 2017, by the way."
  - **Roast (ja):** "なんだろう、コールバック地獄って2017年に解決した問題なんですけど、タイムスリップでもしてきたんですか？　それって技術的負債じゃなくて技術的化石ですよね。"
- **Fix:** Refactor to async/await, Promises, or named functions. Flatten the nesting.

### Nested Ternaries

- **Severity:** warning
- **Detect:** Use Grep to find lines containing two or more `?` operators that form ternary-inside-ternary patterns (e.g., `? ... ? ... : ... : ...`).
- **Deduction:** -4 points
- **Roast (en):** "A ternary inside a ternary — you're playing Inception with boolean logic. Christopher Nolan would be proud; your coworkers will not be."
  - **Roast (ja):** "三項演算子の中に三項演算子って、それってマトリョーシカですよね。読む人の気持ち、考えたことあります？"
- **Fix:** Replace with `if/else` blocks or extract conditions into named variables for readability.

### Long Parameter Lists

- **Severity:** warning
- **Detect:** Use Grep to find function declarations/signatures. Count commas between parentheses. Flag functions with more than 5 parameters.
- **Deduction:** -4 points
- **Roast (en):** "This function takes more arguments than a Thanksgiving dinner debate. That's not a signature — that's a desperate cry for an options object."
  - **Roast (ja):** "引数が5個超えてるんですけど、それっておじいちゃんの遺言状より長い関数シグネチャですよね。オブジェクトにまとめるって発想、ないんですか？"
- **Fix:** Group related parameters into an options object or configuration type.

### Boolean Parameters

- **Severity:** warning
- **Detect:** Use Grep to find function parameters with boolean type annotations (`: boolean`) or default values (`= true`, `= false`). Cross-reference with `if` statements that branch on those params.
- **Deduction:** -4 points
- **Roast (en):** "A boolean parameter is just a polite way of admitting this function does two completely different things wearing a trench coat pretending to be one."
  - **Roast (ja):** "なんだろう、boolean引数って要するに『この関数は2つのことをやります』って自白してるのと同じなんですけど、気づいてます？"
- **Fix:** Split into two separate functions with clear names, or use an enum/string literal for intent.

### Complex Regular Expressions

- **Severity:** warning
- **Detect:** Use Grep to find regex literals (`/.../`) or `new RegExp(...)` constructs. Use Bash to measure character length. Flag any regex exceeding 80 characters that lacks an accompanying comment on the same or previous line.
- **Deduction:** -4 points
- **Roast (en):** "This regex looks like a cat walked across the keyboard and you just committed it. 120 characters of hieroglyphics with zero comments — future you will suffer."
  - **Roast (ja):** "それって正規表現にコメントつけないの、暗号を解読できるの自分だけって思ってますよね。3ヶ月後の自分すら読めないですよ。"
- **Fix:** Break into named sub-patterns with comments, or use a regex builder library for readability.

### Excessive Switch Cases

- **Severity:** warning
- **Detect:** Use Grep to find `switch` statements, then count the number of `case` keywords within each switch block. Flag when case count exceeds 10.
- **Deduction:** -4 points
- **Roast (en):** "This switch statement has more cases than a law firm. At some point you're not writing logic, you're writing a phone book with extra steps."
  - **Roast (ja):** "case文が10個超えてるんですけど、それってswitch文じゃなくて電話帳ですよね。ルックアップオブジェクトって知ってます？"
- **Fix:** Replace with a lookup map/object, polymorphism, or a registry pattern.

### Multiple Return Types

- **Severity:** warning
- **Detect:** Analyze function bodies for `return` statements. Use Grep to find all returns within a function and check if they return different types (e.g., string vs number, object vs null vs undefined). TypeScript union return types with more than 2 variants are also suspect.
- **Deduction:** -4 points
- **Roast (en):** "This function returns a string, a number, null, and occasionally just vibes. TypeScript's type-checker didn't quit — it filed for emotional damages."
  - **Roast (ja):** "なんだろう、戻り値の型がstring | number | null | undefinedって、それってガチャですよね。呼び出し側の気持ち考えたことあります？"
- **Fix:** Return a consistent type. Use Result/Either patterns or throw errors instead of returning mixed types.

### Magic Numbers

- **Severity:** info
- **Detect:** Use Grep to find numeric literals (excluding 0, 1, -1, and common ports) used directly in logic, comparisons, or assignments without a named constant. Check that no `const` declaration on the same or previous line gives it a name.
- **Deduction:** -1 point
- **Roast (en):** "What does 86400 mean here? Seconds in a day? Milliseconds of regret? A zip code? A named constant costs zero dollars and saves infinite confusion."
  - **Roast (ja):** "86400って何ですか？　1日の秒数？　あなたの煩悩の数？　それって名前付き定数にするだけで解決しますよね。なんでやらないんですか？"
- **Fix:** Extract into a named constant (e.g., `const SECONDS_IN_DAY = 86400`).

### Deeply Nested Objects

- **Severity:** warning
- **Detect:** Use Grep to find property access chains with 4 or more dots (e.g., `a.b.c.d.e`). Pattern: `\w+(\.\w+){4,}` — flag any match.
- **Deduction:** -4 points
- **Roast (en):** "You're chaining `a.b.c.d.e.f` — you're not accessing a property, you're drilling for oil. One undefined and the whole thing blows up."
  - **Roast (ja):** "それって`a.b.c.d.e.f`ですよね。プロパティアクセスじゃなくて地下探査ですよそれ。途中でundefined踏んだら全部爆発しますけど、大丈夫ですか？"
- **Fix:** Use destructuring, intermediate variables, or optional chaining with early extraction to flatten access depth.

### Missing Guard Clauses

- **Severity:** warning
- **Detect:** Look for deeply nested if/else chains (3+ levels) where early returns could flatten the structure. Grep for `if\s*\(` appearing at indentation level 3+ inside a function body. Also flag functions that wrap the entire body in a single `if (condition) { ...100 lines... }` instead of returning early for the negative case.
- **Deduction:** -4 points
- **Roast (en):** "Three levels of nested `if` just to reach the happy path. Flip the conditions, return early, and let your code breathe. Your indentation is giving 'pyramid scheme.'"
  - **Roast (ja):** "if文3段ネストしてやっと正常系にたどり着くの、条件反転して早期リターンしてもらっていいですか。このインデント、ピラミッドスキームって言うんですよ。"
- **Fix:** Use guard clauses — check for invalid conditions first and return early. Transform `if (valid) { ...body... }` into `if (!valid) return; ...body...`. Each early return reduces nesting by one level.

## Scoring

Start at 100. Apply deductions per finding. Minimum score is 0. Weight: 1.0x.
