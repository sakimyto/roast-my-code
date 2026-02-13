# Naming Convention Checks

## Activation

Always active.

## Check Items

### Single-Letter Variables

- **Severity:** error
- **Detect:** Use Grep for `(const|let|var)\s+[a-z]\s*[=:]` to find single-letter variable declarations.
  Exclude loop iterators (`for (let i`), short lambda params (`=> x`), and catch blocks (`catch (e)`).
- **Deduction:** -10 points
- **Roast (en):** "You named this variable `d`. Is it a date? A dog? A dimension? A disappointment? All of the above?"
  - **Roast (ja):** "変数名が`d`なんですけど、dateなのかdogなのかdisappointmentなのかわからないですよね。なんだろう、1文字変数でコード書くのやめてもらっていいですか。"
- **Fix:** Replace with a descriptive name. `d` becomes `deadline`, `x` becomes `coordinateX`.

### Generic Names

- **Severity:** error
- **Detect:** Use Grep for standalone identifiers:
  `(const|let|var)\s+(data|info|result|temp|obj|val|item|value|stuff|thing)\s*[=:]`.
  Flag when used as-is without a qualifying prefix or suffix.
  Composite names like `userData` or `validationResult` are acceptable.
- **Deduction:** -10 points
- **Roast (en):** "You named this `data`. What data? All data? The platonic ideal of data? Even `data` would be embarrassed by how generic this is."
  - **Roast (ja):** "変数名が`data`って、それどのデータですか。世の中のすべてはデータなんですよ。もうちょっと具体的に名前つけてもらっていいですか。"
- **Fix:** Add context. `data` becomes `userData`, `result` becomes `validationResult`.

### Misleading Names

- **Severity:** error
- **Detect:** Check booleans lacking `is`/`has`/`should`/`can`/`will` prefix.
  Check function names that imply a pure read (e.g., `get*`, `find*`, `fetch*`)
  but contain side effects (writes, mutations, API calls).
  Cross-reference the name's implied behavior with the function body.
- **Deduction:** -10 points
- **Roast (en):** "`getUser` — sounds innocent. But it also deletes the session, sends an email, and updates three tables. Quite the 'getter'."
  - **Roast (ja):** "`getUser`って名前なのにセッション削除してメール送ってテーブル3つ更新してるんですけど、それってgetじゃなくてdestroyですよね。名前と中身が全然違うの詐欺ですよ。"
- **Fix:** Boolean: add `is`/`has`/`should` prefix. Functions: rename to match actual behavior or split into separate functions.

### Inconsistent Casing

- **Severity:** warning
- **Detect:** Scan the file for variable and function declarations.
  Use Grep to find both `[a-z]+_[a-z]+` (snake_case) and `[a-z]+[A-Z][a-z]+` (camelCase)
  patterns in the same file.
  Ignore external library identifiers, constants in UPPER_SNAKE_CASE, and class names in PascalCase.
  Flag when both camelCase and snake_case appear in user-defined variable or function names.
- **Deduction:** -4 points
- **Roast (en):** "Half your variables are camelCase, the other half are snake_case. Your codebase is bilingual and fluent in neither."
  - **Roast (ja):** "camelCaseとsnake_caseが混在してるんですけど、それってルールがないのと同じですよね。どっちかに統一してもらっていいですか。"
- **Fix:** Choose one convention per language (camelCase for JS/TS, snake_case for Python/Rust) and enforce with a linter rule.

### Abbreviated Names

- **Severity:** warning
- **Detect:** Use Grep for common abbreviations in identifiers:
  `(usr|msg|btn|cfg|mgr|impl|req|res|err|cb|fn|ctx|evt|elem|attr|prop|prev|cur|num|str|arr|idx)`
  as whole words or segments in variable and function names.
  Accept abbreviations that are industry-standard within their context
  (e.g., `req`/`res` in Express middleware, `ctx` in Go).
- **Deduction:** -4 points
- **Roast (en):** "`usr`, `msg`, `btn`, `cfg`. You're not paying per character. Your IDE has autocomplete. Spell it out."
  - **Roast (ja):** "なんだろう、`usr`とか`msg`とか`btn`とか、文字数に課金されてるわけじゃないんだから省略しないでもらっていいですか。IDEの補完機能使ってください。"
- **Fix:** Spell it out. `usr` becomes `user`, `msg` becomes `message`, `btn` becomes `button`. Your IDE has autocomplete — use it.

### Hungarian Notation

- **Severity:** info
- **Detect:** Use Grep for type-prefix patterns in variable declarations:
  `(str|int|bool|arr|obj|fn|num|chr|dbl|flt)[A-Z]\w+`.
  Also flag `b[A-Z]\w+` for booleans and `i[A-Z]\w+` for integers
  when the prefix clearly encodes the type rather than being a natural word.
- **Deduction:** -1 point
- **Roast (en):** "`strName`, `iCount`, `bFlag` — welcome to 1995. We have Hungarian notation and dial-up internet."
  - **Roast (ja):** "`strName`、`iCount`、`bFlag`って、1995年からタイムスリップしてきました？ハンガリアン記法はダイヤルアップ回線と一緒に卒業してください。"
- **Fix:** Remove the type prefix. Modern languages and IDEs provide type information.
  `strName` becomes `name`, `iCount` becomes `count`, `bFlag` becomes `isEnabled`.

### Meaningless Prefixes

- **Severity:** warning
- **Detect:** Use Grep for `(const|let|var)\s+(my|the|a|an|this_)[A-Z]\w+` and similar patterns.
  Flag variables prefixed with articles or possessives that add zero semantic value.
  Exclude legitimate uses like `aNodeInTree` where `a` clarifies cardinality.
- **Deduction:** -4 points
- **Roast (en):** "`myData`, `theList`, `aResult`. Adding an article doesn't add meaning — it adds confusion. This isn't creative writing."
  - **Roast (ja):** "`myData`とか`theList`とか、冠詞つけても意味は増えないんですよ。それって英作文の練習じゃなくてプログラミングですよね。"
- **Fix:** Drop the prefix. `myData` becomes `userData` or just `data` with proper scope. `theList` becomes `orders`.

### File Name Mismatch

- **Severity:** warning
- **Detect:** Read the file to find the default export (`export default`) or `module.exports` assignment.
  Compare the exported identifier name with the filename (minus extension).
  Account for case conventions — a PascalCase class in a PascalCase or kebab-case filename is fine.
  Flag when the export name and filename are semantically unrelated.
- **Deduction:** -4 points
- **Roast (en):** "File is called `utils.ts`, default export is `AuthenticationService`. That's not a util -- that's an identity crisis in a trench coat."
  - **Roast (ja):** "ファイル名が`utils.ts`なのにexportしてるのが`AuthenticationService`って、それutilじゃないですよね。名札と中身が一致してないの、迷子案内所行きですよ。"
- **Fix:** Rename the file to match the default export, or rename the export to match the file. One source of truth.

### Overly Long Names

- **Severity:** info
- **Detect:** Use Grep for identifiers exceeding 40 characters:
  `[a-zA-Z_$][a-zA-Z0-9_$]{40,}`.
  Check variable, function, class, and type declarations.
  Accept long names in test descriptions and string constants.
- **Deduction:** -1 point
- **Roast (en):** "`getUserAuthenticationTokenFromExternalIdentityProvider` -- this variable name needs its own scroll bar. Descriptive is good; a dissertation title is not."
  - **Roast (ja):** "`getUserAuthenticationTokenFromExternalIdentityProvider`って、それ変数名じゃなくて論文タイトルですよね。説明的なのはいいですけど限度がありますよ。"
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
- **Roast (en):** "You named an array `item`. One item? No -- it's an array. Arrays have friends. They deserve an 's'."
  - **Roast (ja):** "配列なのに変数名が`item`って、それ複数形にしてもらっていいですか。中に何個も入ってるのに単数形なの、集合写真を『自撮り』って呼ぶようなものですよ。"
- **Fix:** Arrays get plural names (`items`, `users`). Single values get singular names (`item`, `user`). Match the name to the cardinality.

### Negative Booleans

- **Severity:** warning
- **Detect:** Use Grep for boolean-like identifiers with negative phrasing:
  `(isNot|isNo|not[A-Z]|disable|disabled|no[A-Z]|without|exclude|never|dont|lack)`
  in variable declarations, function parameters, and object properties.
  Especially flag double-negative usage patterns like `!isNotReady` or `!disableFeature`
  by searching for `!\s*(isNot|isNo|disable|not[A-Z])`.
- **Deduction:** -4 points
- **Roast (en):** "`!isNotDisabled` -- congratulations, you've invented a double negative in code. Even the compiler is confused."
  - **Roast (ja):** "`!isNotDisabled`って、二重否定で頭バグりますよね。結局有効なの無効なの、書いた本人も3日後には分からなくなりますよ。"
- **Fix:** Invert the name to a positive form. `isNotReady` becomes `isReady`, `disableFeature` becomes `featureEnabled`. Flip the logic to match.

## Scoring

Start at 100. Apply deductions per finding. Minimum score is 0. Weight: 1.0x.
