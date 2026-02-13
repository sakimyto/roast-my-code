# Type Safety Checks

## Activation

Active when TypeScript files (`*.ts`, `*.tsx`) are detected, or when JSDoc type annotations (`@type`, `@param`, `@returns`) are present in JavaScript files.

## Check Items

### Explicit `any` Usage

- **Severity:** error
- **Detect:** Use Grep to find `: any`, `<any>`, and `as any` patterns in source files.
  Exclude `node_modules`, test files, and declaration files (`*.d.ts`).
  Count total occurrences across the project.
- **Deduction:** -10 points
- **Roast (en):** "`any` is TypeScript's safe word, and you've used it {n} times. At this point just use JavaScript — at least that's honest about not having types."
  - **Roast (ja):** "なんだろう、`any`を{n}回も使うなら最初からJavaScript書いてもらっていいですか。それってTypeScript使ってる意味ないですよね。"
- **Fix:** Replace with specific types, `unknown` for truly dynamic values, or generics
  where the type varies but should be tracked. Use `unknown` and narrow with type guards.

### Type Assertions Abuse

- **Severity:** warning
- **Detect:** Use Grep to count `as <Type>` casts (excluding `as const`) and `!` non-null
  assertions. For non-null assertions, match patterns like `\w+!\.` or `\w+!\[` to avoid
  false positives with logical NOT. Flag when the combined total exceeds 5 per file.
- **Deduction:** -4 points
- **Roast (en):** "Every `as` cast is a pinky promise to the compiler. You've made {n} promises you definitely can't keep."
  - **Roast (ja):** "それって`as`でコンパイラに嘘ついてるだけですよね。{n}回も嘘ついたら普通に信用なくしますよ。"
- **Fix:** Use type guards (`if ('prop' in obj)`), narrowing via control flow, or refactor
  data flow so TypeScript can infer the correct type without manual assertions.

### Missing Return Types

- **Severity:** warning
- **Detect:** Use Grep to find exported function declarations (`export function`) and arrow
  functions assigned to exported variables (`export const fn =`). Check if they have explicit
  return type annotations (`: ReturnType` between the closing paren and the opening brace
  or arrow). Flag those without.
- **Deduction:** -4 points
- **Roast (en):** "Your exported functions return... surprise! Every call is a mystery box. Your coworkers love mystery boxes, right?"
  - **Roast (ja):** "なんだろう、exportした関数の戻り値がガチャなのやめてもらっていいですか。呼ぶ側は毎回ドキドキしたくないんですよ。"
- **Fix:** Add explicit return type annotations to all exported functions. This catches
  accidental return type changes and improves IDE performance on large projects.

### Implicit `any`

- **Severity:** error
- **Detect:** Use Grep to locate `tsconfig.json`. Read it and check for
  `"noImplicitAny": false` or confirm that `noImplicitAny` is absent when `strict` is also
  not enabled. Both conditions allow parameters and variables to silently receive `any`.
- **Deduction:** -10 points
- **Roast (en):** "`noImplicitAny` is off. That's like disabling your smoke alarm because you don't like the beeping sound."
  - **Roast (ja):** "それって火災報知器がうるさいから電池抜くのと同じですよね。`noImplicitAny`をオフにするのは自由ですけど、火事になっても知らないですよ。"
- **Fix:** Set `"noImplicitAny": true` in tsconfig (or enable `"strict": true` which
  includes it). Fix resulting errors by adding proper type annotations.

### `@ts-ignore` / `@ts-expect-error`

- **Severity:** warning
- **Detect:** Use Grep to find `@ts-ignore` and `@ts-expect-error` comments in source files.
  Count total occurrences. Exclude declaration files and generated code.
- **Deduction:** -4 points
- **Roast (en):** "Every `@ts-ignore` is a tiny white flag of surrender. You've planted {n} of them. This codebase is a battlefield and you're losing."
  - **Roast (ja):** "`@ts-ignore`が{n}個あるんですけど、それって型エラーと向き合うのを諦めた回数ですよね。降参するの早くないですか。"
- **Fix:** Fix the underlying type error. If suppression is truly necessary, prefer
  `@ts-expect-error` (which fails when the error is resolved) with a comment explaining why.

### Loose tsconfig

- **Severity:** error
- **Detect:** Read `tsconfig.json` and check for `"strict": false` or the absence of the
  `strict` field entirely. Also flag if key strict sub-options are explicitly disabled:
  `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`,
  `strictPropertyInitialization`.
- **Deduction:** -10 points
- **Roast (en):** "`strict: false` — you're using TypeScript with the training wheels on, the bumpers up, and the safety net deployed. Why even bother?"
  - **Roast (ja):** "なんだろう、`strict: false`でTypeScript書くのって補助輪つけたままツール・ド・フランス出るようなものですよね。出る意味あります？"
- **Fix:** Set `"strict": true` in tsconfig and address type errors incrementally. This
  single flag enables null checks, bind/call/apply typing, and property initialization.

### Untyped API Responses

- **Severity:** warning
- **Detect:** Use Grep to find `fetch(`, `axios.get(`, `axios.post(`, `axios.put(`,
  `axios.delete(`, and similar HTTP client call patterns. Check if the response or `.data`
  property is typed — look for generic type parameters (e.g., `axios.get<UserResponse>`)
  or type annotations on the receiving variable. Flag any untyped API call results.
- **Deduction:** -4 points
- **Roast (en):** "You're trusting API responses like unverified luggage at an airport. That `data` could be anything. Literally anything. Validate it."
  - **Roast (ja):** "APIのレスポンスをノーチェックで信用してるんですけど、それって知らない人から受け取った荷物をそのまま開けるのと同じですよね。中身確認しましょうよ。"
- **Fix:** Define response types or interfaces matching the API contract. Use runtime
  validation (Zod, io-ts, Valibot) with TypeScript generics for compile-time and runtime safety.

### String Enums vs Union Types

- **Severity:** info
- **Detect:** Use Grep to find `enum` declarations where all members are string-initialized
  (pattern: `= ["']`). Flag especially small enums with 2-4 members where a string literal
  union type would be simpler and provide equivalent or better type narrowing.
- **Deduction:** -1 point
- **Roast (en):** "A string enum for two values. That's like renting a U-Haul to move a backpack."
  - **Roast (ja):** "2個しか値がないのにenumって、それってコンビニ行くのに引越しトラック出すようなものですよね。ユニオン型で十分ですよ。"
- **Fix:** Consider string literal union types (`type Status = 'active' | 'inactive'`) for
  simple cases. Use discriminated unions for complex cases. Union types tree-shake better.

### `Object` / `Function` / `{}` Types

- **Severity:** error
- **Detect:** Use Grep to find type annotations using `: Object`, `: Function`, or `: {}`
  as a type annotation. Pattern: `:\s*(Object|Function|\{\})\b` in source files. These
  top-level types accept almost anything and provide no meaningful type safety.
- **Deduction:** -10 points
- **Roast (en):** "The type `Object` communicates exactly nothing. You might as well annotate it as `: vibes`."
  - **Roast (ja):** "なんだろう、型が`Object`って何も伝わらないんですよ。`: なんか` って書いてるのと同じですよね。"
- **Fix:** Replace `Object` with `Record<string, unknown>`, `Function` with a specific
  signature like `(arg: string) => void`, and `{}` with `Record<string, never>` or a proper interface.

### No Readonly for Immutable Data

- **Severity:** info
- **Detect:** Use Grep to find `const` declarations holding arrays (`[]`, `Array<`) or
  objects (`{}`) that are never mutated in the surrounding scope. Check for the absence
  of mutation operations: `.push()`, `.splice()`, `.pop()`, `.shift()`, bracket assignment,
  or property reassignment. Flag candidates that could benefit from `readonly` or `as const`.
- **Deduction:** -1 point
- **Roast (en):** "This data never changes but nothing stops it from changing. `readonly` is right there. It's literally one word."
  - **Roast (ja):** "このデータ一回も変更してないのに`readonly`つけないの、鍵かけずに家出るのと同じですよね。つけるだけでいいのに。"
- **Fix:** Use `readonly` modifier for array types (`readonly string[]`), `Readonly<T>` for
  objects, or `as const` for literal values. Prevents accidental mutations at compile time.

### Missing Generic Constraints

- **Severity:** warning
- **Detect:** Use Grep to find generic type parameters (`<T>`, `<T, U>`, etc.) in function
  signatures and class/interface declarations. Flag cases where `T` lacks an `extends`
  constraint but is accessed with `.` operator, indexed (`[key]`), or passed to a typed
  function — indicating an implicit shape assumption that should be explicit.
- **Deduction:** -4 points
- **Roast (en):** "Your generic `<T>` accepts literally anything — a string, a number, a cat, a promise of a cat. Constraints exist for a reason."
  - **Roast (ja):** "この`<T>`、文字列でも数値でも猫でも何でも入るんですけど、それって玄関のドアを全開にして『誰でもどうぞ』って言ってるのと同じですよね。`extends`つけましょうよ。"
- **Fix:** Add `extends` constraints: `<T extends Record<string, unknown>>`,
  `<T extends BaseEntity>`, or `<T extends { id: string }>`. Documents intent and catches misuse.

### Null/Undefined Overuse

- **Severity:** warning
- **Detect:** Grep for excessive optional chaining (3+ `?.` in one expression). Pattern: `(\?\.\w+){3,}`. Also flag functions with multiple `return null` or `return undefined` statements — more than 3 per file suggests systematic null-threading instead of proper type narrowing.
- **Deduction:** -4 points
- **Roast (en):** "`user?.profile?.settings?.theme?.color` — five question marks in one line. Your code is as uncertain about its own data as I am about its quality."
  - **Roast (ja):** "`user?.profile?.settings?.theme?.color`って、1行に`?`が5つもあるんですけど。コード自体が自分のデータに自信ないんですよね。型をちゃんと定義してもらっていいですか。"
- **Fix:** Define proper types so optional chaining isn't needed everywhere. Use discriminated unions or the Null Object pattern. Handle absence explicitly at the boundary rather than threading uncertainty through the entire call chain.

## Scoring

Start at 100. Apply deductions per finding. Minimum score is 0. Weight: 1.0x.
