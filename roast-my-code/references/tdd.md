# TDD Compliance Checks

## Activation

Always active. Weight increases to 1.5x in sensei mode.

## Philosophy

t_wada-style TDD: "テストは仕様書。不安をテストに変えよ。"
Red-Green-Refactor cycle is sacred. Tests come first, not after.

## Checks

### No Test Files

- **Severity:** critical
- **Detect:** Glob for `**/*.test.*`, `**/*.spec.*`, `**/__tests__/**`. Zero matches means zero tests.
- **Deduction:** -20 points
- **Roast (en):** "Zero tests. Not 'low coverage' — literally zero. Your deploy button is just a prayer button with extra steps."
  - **Roast (ja):** "テストがゼロって、それってデプロイボタンを『お祈りボタン』って呼んだほうが正確ですよね。なんだろう、怖くないんですか？"
- **Fix:** Create test files following the Red-Green-Refactor cycle. Every source file deserves a spec.

### Low Test-to-Source Ratio

- **Severity:** error
- **Detect:** Count test files vs source files. Ratio below 50% triggers this check.
- **Deduction:** -10 points
- **Roast (en):** "Your test-to-source ratio is giving 'I'll write tests later' energy. Spoiler: later never comes. It's been three sprints."
  - **Roast (ja):** "テストファイルの比率が50%以下って、それって『あとで書く』の『あと』が永遠に来ないやつですよね。知ってますよそのパターン。"
- **Fix:** Add test files for uncovered modules. Aim for at least 1:1 ratio.

### Test Without Assertion

- **Severity:** error
- **Detect:** Test blocks (`it(`, `test(`) containing no `expect`, `assert`, or `should` calls.
- **Deduction:** -10 points
- **Roast (en):** "A test with no assertions is just code cosplaying as quality assurance. It runs, it passes, it proves absolutely nothing."
  - **Roast (ja):** "assertionがないテストって、それってただの動作確認じゃないですか。テストって名前つけるの、詐欺に近いですよね。"
- **Fix:** Add meaningful assertions that verify behavior, not just execution.

### Implementation Without Matching Test

- **Severity:** error
- **Detect:** Source file exists (e.g., `foo.ts`) but no corresponding test file (`foo.test.ts`, `foo.spec.ts`).
- **Deduction:** -10 points
- **Roast (en):** "Shipping code without tests is like skydiving without checking your parachute. Bold? Sure. Smart? Absolutely not."
  - **Roast (ja):** "なんだろう、テストなしで本番にデプロイするの、ノーヘルでバイク乗るのと同じなんですけど、そのスリルが楽しいんですか？"
- **Fix:** Write a test file for each source module before (or immediately after) implementation.

### Commented-Out Tests

- **Severity:** error
- **Detect:** Patterns like `// it(`, `// test(`, `/* describe`, `xit(`, `xdescribe(`, `test.skip(`.
- **Deduction:** -10 points
- **Roast (en):** "Commented-out tests are just tech debt wearing a disguise. They're not 'temporarily disabled' — they're permanently forgotten lies."
  - **Roast (ja):** "コメントアウトされたテストって、それって『一時的に無効化』じゃなくて『永遠に見て見ぬふり』ですよね。消すか直すかどっちかにしてもらっていいですか。"
- **Fix:** Either restore the tests and make them pass, or remove them entirely.

### Test Describes Only Happy Path

- **Severity:** warning
- **Detect:** No tests containing keywords: "error", "throw", "reject", "invalid", "edge", "fail", "boundary".
- **Deduction:** -4 points
- **Roast (en):** "All your tests assume the happy path, always. Your code lives in a world without errors, edge cases, or Mondays. Must be nice in there."
  - **Roast (ja):** "正常系しかテストしてないんですけど、それってこの世にエラーが存在しないと思ってます？　ユーザーはあなたの想像を超える使い方しますよ。"
- **Fix:** Add error-case and edge-case tests. Think about what can go wrong, then prove it is handled.

### Overmocking

- **Severity:** warning
- **Detect:** More than 5 `jest.mock()`, `vi.mock()`, or `sinon.stub()` calls in a single test file.
- **Deduction:** -4 points
- **Roast (en):** "This test mocks so much it's basically fan fiction. At this point you're not testing your code — you're testing whether Jest works. Spoiler: it does."
  - **Roast (ja):** "mock5個超えって、それってテストじゃなくて妄想シミュレーターですよね。自分のコードじゃなくてモックライブラリをテストしてどうするんですか。"
- **Fix:** Reduce mocks by using dependency injection or testing smaller units. Mock boundaries, not internals.

### Slow Tests

- **Severity:** warning
- **Detect:** Test timeout set over 10000ms, or `setTimeout` used inside test blocks.
- **Deduction:** -4 points
- **Roast (en):** "This test takes {n} seconds. A slow test is a test that stops getting run. A test that stops getting run is just a README with extra steps."
  - **Roast (ja):** "テストに{n}秒かかるって、それって遅すぎて誰も実行しなくなるやつですよね。実行されないテストって、存在しないのと同じですよ。"
- **Fix:** Isolate slow dependencies. Use fakes for I/O. Keep unit tests under 1s total.

### Test File Larger Than Source

- **Severity:** info
- **Detect:** Test file line count exceeds 3x the corresponding source file line count.
- **Deduction:** -1 point
- **Roast (en):** "Your test file is writing a novel about a haiku. 3x the source lines means you're testing implementation details, not behavior — and it'll break on every refactor."
  - **Roast (ja):** "テストがソースの3倍の行数って、それって俳句の解説に論文書いてるようなものですよね。実装の詳細じゃなくて振る舞いをテストしてもらっていいですか。"
- **Fix:** Extract shared setup into helpers. Over-specified tests are brittle — test behavior, not implementation.

### No CI Test Step

- **Severity:** warning
- **Detect:** No `test` script in `package.json`, and no test step in CI config (`.github/workflows/*.yml`, etc.).
- **Deduction:** -4 points
- **Roast (en):** "Tests exist but nothing runs them automatically. Schrodinger's test suite — simultaneously passing and failing until someone bothers to check. That someone is never you."
  - **Roast (ja):** "テストあるのにCIで動かしてないって、それってシュレディンガーのテストスイートですよね。観測するまで成功も失敗もしてないっていう。"
- **Fix:** Add a `test` script to `package.json` and a CI workflow step that runs it on every push.

### Snapshot Overuse

- **Severity:** warning
- **Detect:** More than 10 `toMatchSnapshot()` or `toMatchInlineSnapshot()` calls across test files.
- **Deduction:** -4 points
- **Roast (en):** "Snapshot tests don't verify behavior — they verify nothing changed. That's not testing, that's digital hoarding with a green checkmark."
  - **Roast (ja):** "スナップショットテスト10個超えって、それってテストじゃなくて『変更してないことの証明』ですよね。なんだろう、何も変わってないことを確認して何が嬉しいんですか。"
- **Fix:** Replace snapshots with explicit assertions on the values that matter.

### Missing Coverage Config

- **Severity:** info
- **Detect:** No `coverage` configuration in `jest.config.*`, `vitest.config.*`, or `package.json`.
- **Deduction:** -1 point
- **Roast (en):** "No coverage config means you can't improve what you refuse to measure. It's like dieting without a scale — all vibes, zero accountability."
  - **Roast (ja):** "カバレッジ設定がないって、それって体重計に乗らないダイエットと同じですよね。計測しないものは改善できないって、ドラッカーも言ってますよ。"
- **Fix:** Add coverage configuration with thresholds. Track lines, branches, functions, and statements.

## Scoring

- **Base score:** 100
- **Apply deductions** per finding (each instance counts separately)
- **Minimum score:** 0
- **Weight:** 1.0x normal, 1.5x in sensei mode
