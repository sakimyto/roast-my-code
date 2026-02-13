# Dead Code Checks

Category weight: **1.0x**

## Activation

Always active. These checks apply to every codebase regardless of language or framework.
Scan source files, import graphs, and dependency manifests to find code that serves no purpose.

---

## Philosophy

Dead code is technical debt with zero interest — it costs you readability and maintenance
burden while providing absolutely nothing in return. Every line of dead code is a line
that a future developer will read, wonder about, and be afraid to delete.

---

## Check Items

### Commented-Out Code

- **Severity:** error
- **Detect:** Use Grep to find 3 or more consecutive commented lines (`//` or `/* ... */`) that contain code-like patterns such as assignments, function calls, control flow keywords, or variable declarations. Pattern: look for blocks of comments containing `=`, `(`, `)`, `if`, `return`, `const`, `let`, `var`, `function`. Exclude license headers, JSDoc documentation blocks, and intentional explanatory comments.
- **Deduction:** -10 points
- **Roast (en):** "Commented-out code is not version control. That's literally what git is for — delete it or admit you have commitment issues."
  - **Roast (ja):** "なんだろう、コメントアウトしたコードを保存してるの、gitっていうバージョン管理ツールがあるんですけど...ご存知ない感じですかね。"
- **Fix:** Remove the commented code entirely. If you need it later, retrieve it from git history with `git log -p`. Never leave code "just in case" — that is what branches and tags are for.

### Unreachable Code After Return

- **Severity:** error
- **Detect:** Use Grep with multiline mode to find executable statements on lines immediately following `return`, `throw`, `break`, or `continue` within the same block scope. Pattern: `(return|throw|break|continue)\b.*;\n\s*(const|let|var|if|for|while|switch|[a-zA-Z])`. Exclude lines that are only closing braces, comments, or `case` labels following `break`.
- **Deduction:** -10 points
- **Roast (en):** "Code after a `return` statement is the software equivalent of talking after someone hangs up. Nobody's listening."
  - **Roast (ja):** "それって`return`の後にコード書いても一生実行されないですよね。なんだろう、誰にも届かないラブレター書くの、やめてもらっていいですか。"
- **Fix:** Remove the unreachable statements, or restructure the control flow so the code executes when intended. If the code below is meant as a fallback, move it to an `else` branch.

### Unused Imports

- **Severity:** warning
- **Detect:** Use Grep to extract all imported identifiers from `import { X }`, `import X from`, `import * as X from`, and `const X = require(...)` statements. For each identifier, search the rest of the file (excluding the import line itself) for any reference. Flag identifiers that appear only in the import statement and nowhere else in the file body.
- **Deduction:** -4 points
- **Roast (en):** "You imported this module and then ghosted it harder than a Tinder match. It's been sitting at the top of your file waiting for a call that's never coming."
  - **Roast (ja):** "importしておいて一回も使ってないの、それってマッチングアプリでいいねだけして一生メッセージ送らない人と同じですよね。"
- **Fix:** Remove the unused import. Configure your editor or a linter rule (e.g., `no-unused-vars`, `@typescript-eslint/no-unused-vars`) to auto-remove unused imports on save.

### Unused Variables

- **Severity:** warning
- **Detect:** Use Grep to find `const`, `let`, or `var` declarations and extract the variable name. For each declared name, count occurrences in the file. Flag variables that appear exactly once (only at the declaration site). Exclude variables prefixed with underscore (`_`) which conventionally indicate intentionally unused bindings.
- **Deduction:** -4 points
- **Roast (en):** "You declared this variable like you had big plans for it. Spoiler: the plans never materialized. It's just vibes and wasted memory now."
  - **Roast (ja):** "なんだろう、変数宣言だけして使わないの、買って満足して積んでる本と同じなんですよね。せめて読んであげてもらっていいですか。"
- **Fix:** Remove the unused variable. If it is a destructured value you want to skip, prefix with underscore (`_unused`) to signal intent clearly.

### Unused Functions

- **Severity:** warning
- **Detect:** Use Grep to find exported function declarations (`export function name`) and exported arrow function assignments (`export const name =`). Then search all other files in the project for import references to each function name. Flag exports that are never imported or called by any other file. Exclude entry point functions, event handlers registered elsewhere, and framework lifecycle hooks.
- **Deduction:** -4 points
- **Roast (en):** "This exported function has zero consumers. It's been public since day one and still has less engagement than a LinkedIn post at 3 AM."
  - **Roast (ja):** "exportしてるのに誰もimportしてないですよね。それって街中で演説してるのに誰も足を止めない人と同じ状況なんですけど、大丈夫ですかね。"
- **Fix:** Remove the function entirely, or convert it to a non-exported local function if only used within the same file. If planned for future use, track it in an issue rather than shipping dead code.

### Unused Dependencies

- **Severity:** error
- **Detect:** Parse both `dependencies` and `devDependencies` in `package.json` to get the full list of package names. Use Grep across all source files, config files, and scripts for `import ... from '{package}'`, `require('{package}')`, or references in build tool configuration. Flag packages with zero matches anywhere in the project.
- **Deduction:** -10 points
- **Roast (en):** "{n} phantom dependencies that nothing imports. Your `node_modules` is basically a digital hoarder's storage unit at this point."
  - **Roast (ja):** "誰もimportしてないパッケージが{n}個あるんですけど、それって使わないのにサブスク解約しない人と同じですよね。お金もディスクも無限じゃないんで。"
- **Fix:** Run `npm uninstall` or `bun remove` for each unused package. Use `depcheck` or `knip` for automated auditing. Fewer dependencies means faster installs.

### Dead Feature Flags

- **Severity:** warning
- **Detect:** Use Grep to find common feature flag patterns including variables named `isEnabled`, `featureFlag`, `FF_`, `FEATURE_`, `TOGGLE_`, or constants in config files. Trace their values through assignments. Flag flags that are hardcoded to `true` or `false` with no conditional override, no environment-based toggle, and no runtime configuration source.
- **Deduction:** -4 points
- **Roast (en):** "This feature flag has been hardcoded to `true` since the Mesozoic era. It's not a flag — it's a fossil. Just inline the code and move on."
  - **Roast (ja):** "それってフィーチャーフラグがずっと`true`固定なんですけど、もうフラグじゃなくて置物ですよね。なんだろう、`if(true)`書くの、やめてもらっていいですか。"
- **Fix:** Remove the flag and its conditional branches entirely. Keep only the active code path. If the flag is genuinely temporary, add a cleanup date and a tracking issue.

### TODO/FIXME/HACK Comments

- **Severity:** info
- **Detect:** Use Grep to find `TODO`, `FIXME`, `HACK`, `XXX`, `TEMP`, or `WORKAROUND` comments across all source files. Pattern: `(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND)\b`. Report each occurrence with file path and line number. Count the total for the summary.
- **Deduction:** -1 point
- **Roast (en):** "Found a TODO that predates the repo's CI pipeline. At this point it's not a TODO — it's a WONTDO. File an issue or delete it."
  - **Roast (ja):** "このTODO、いつから放置されてるんですかね。それってもうTODOじゃなくてWONTDOですよね。やる気がないならせめてissueにしてもらっていいですか。"
- **Fix:** Resolve the TODO now or convert it into a tracked issue with an assignee and deadline. Stale TODOs are where good intentions go to die — if it has been there for more than a sprint, it is not a TODO, it is a WONTDO.

### Empty Files

- **Severity:** warning
- **Detect:** Use Bash to find source files (`.ts`, `.js`, `.tsx`, `.jsx`, `.py`) whose content is only whitespace, contains only a bare `export {}` statement, or has no meaningful executable statements. Files containing only comments and no actual code also qualify as empty.
- **Deduction:** -4 points
- **Roast (en):** "This file is empty. It's the equivalent of an employee who shows up, sits at their desk, and does absolutely nothing. HR would like a word."
  - **Roast (ja):** "このファイル中身が空なんですけど、それって出社だけして仕事しない人と同じですよね。存在する意味がないので消してもらっていいですか。"
- **Fix:** Delete the file and remove any imports or references that point to it. If it is a placeholder for future work, create an issue instead of an empty file.

### Orphan Files

- **Severity:** warning
- **Detect:** For each source file, use Grep to search the entire project for its filename (without extension) or full module path in import/require statements and route registrations. Flag files that are never imported, required, or referenced by any other file. Exclude known entry points (`index.ts`, `main.ts`, `app.ts`), test files, configuration files, and scripts listed in `package.json`.
- **Deduction:** -4 points
- **Roast (en):** "This file is the lonely island of your codebase. Zero imports, zero references. It's giving main character energy with zero screen time."
  - **Roast (ja):** "このファイル、どこからもimportされてないんですけど、それって友達のグループLINEに入ってるのに一回も発言しない人と同じですよね。"
- **Fix:** Delete orphan files or wire them into the module graph. If it is a standalone script or CLI entry point, document it explicitly in the project configuration.

### Deprecated API Usage

- **Severity:** warning
- **Detect:** Use Grep to find `@deprecated` JSDoc tags and `/** @deprecated */` annotations in the project's own source code. Extract the names of deprecated functions, methods, classes, or interfaces. Then search the rest of the codebase for call sites, instantiations, or references to those deprecated symbols. Flag any active usage.
- **Deduction:** -4 points
- **Roast (en):** "You're calling a deprecated function. It's the condemned building of your codebase — still standing, but one `npm update` away from rubble."
  - **Roast (ja):** "deprecatedな関数をまだ呼んでるんですけど、それって取り壊し予定のビルに住み続けてるのと同じですよね。いつ崩れても文句言えないですよ。"
- **Fix:** Migrate to the recommended replacement API as indicated in the deprecation notice. Check the changelog or migration guide for step-by-step instructions. If no replacement exists yet, open an issue to plan the migration.

---

## Severity Reference

| Severity | Points | Meaning |
|----------|--------|---------|
| critical | -20 | Severe structural issue, must fix immediately |
| error | -10 | Significant dead weight, fix soon |
| warning | -4 | Moderate clutter, plan to clean up |
| info | -1 | Minor hygiene issue, nice to fix |

---

## Scoring

| Step | Detail |
|------|--------|
| Start | 100 points |
| Deductions | Apply per finding using the values above |
| Floor | Minimum score is 0 (no negatives) |
| Weight | Multiply final score by **1.0x** for category contribution |

Example: 2 commented-out blocks (-20) + 3 unused imports (-12) + 1 unused dependency (-10) + 5 TODOs (-5) = 100 - 47 = 53.
Weighted: 53 * 1.0 = 53.
