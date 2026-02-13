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
- **Roast:** "I found your test suite. Just kidding — there isn't one."
- **Fix:** Create test files following the Red-Green-Refactor cycle. Every source file deserves a spec.

### Low Test-to-Source Ratio

- **Severity:** error
- **Detect:** Count test files vs source files. Ratio below 50% triggers this check.
- **Deduction:** -10 points
- **Roast:** "Your test coverage is like a raincoat made of tissue paper."
- **Fix:** Add test files for uncovered modules. Aim for at least 1:1 ratio.

### Test Without Assertion

- **Severity:** error
- **Detect:** Test blocks (`it(`, `test(`) containing no `expect`, `assert`, or `should` calls.
- **Deduction:** -10 points
- **Roast:** "A test without assertions is just code that runs — congratulations, you invented a script."
- **Fix:** Add meaningful assertions that verify behavior, not just execution.

### Implementation Without Matching Test

- **Severity:** error
- **Detect:** Source file exists (e.g., `foo.ts`) but no corresponding test file (`foo.test.ts`, `foo.spec.ts`).
- **Deduction:** -10 points
- **Roast:** "This file is running through production with no safety net. Bold strategy."
- **Fix:** Write a test file for each source module before (or immediately after) implementation.

### Commented-Out Tests

- **Severity:** error
- **Detect:** Patterns like `// it(`, `// test(`, `/* describe`, `xit(`, `xdescribe(`, `test.skip(`.
- **Deduction:** -10 points
- **Roast:** "Commented-out tests are lies about your system. Delete them or fix them."
- **Fix:** Either restore the tests and make them pass, or remove them entirely.

### Test Describes Only Happy Path

- **Severity:** warning
- **Detect:** No tests containing keywords: "error", "throw", "reject", "invalid", "edge", "fail", "boundary".
- **Deduction:** -4 points
- **Roast:** "Your tests assume the world is kind. The world is not kind."
- **Fix:** Add error-case and edge-case tests. Think about what can go wrong, then prove it is handled.

### Overmocking

- **Severity:** warning
- **Detect:** More than 5 `jest.mock()`, `vi.mock()`, or `sinon.stub()` calls in a single test file.
- **Deduction:** -4 points
- **Roast:** "Your test mocks so much reality it's basically fiction."
- **Fix:** Reduce mocks by using dependency injection or testing smaller units. Mock boundaries, not internals.

### Slow Tests

- **Severity:** warning
- **Detect:** Test timeout set over 10000ms, or `setTimeout` used inside test blocks.
- **Deduction:** -4 points
- **Roast:** "Fast feedback is the soul of TDD. This test is meditating, not testing."
- **Fix:** Isolate slow dependencies. Use fakes for I/O. Keep unit tests under 1s total.

### Test File Larger Than Source

- **Severity:** info
- **Detect:** Test file line count exceeds 3x the corresponding source file line count.
- **Deduction:** -1 point
- **Roast:** "Your test is writing a novel about a haiku. Consider simplifying."
- **Fix:** Extract shared setup into helpers. Over-specified tests are brittle — test behavior, not implementation.

### No CI Test Step

- **Severity:** warning
- **Detect:** No `test` script in `package.json`, and no test step in CI config (`.github/workflows/*.yml`, etc.).
- **Deduction:** -4 points
- **Roast:** "Tests exist but nothing runs them. Schrodinger's test suite."
- **Fix:** Add a `test` script to `package.json` and a CI workflow step that runs it on every push.

### Snapshot Overuse

- **Severity:** warning
- **Detect:** More than 10 `toMatchSnapshot()` or `toMatchInlineSnapshot()` calls across test files.
- **Deduction:** -4 points
- **Roast:** "Snapshot tests don't verify behavior — they verify that nothing changed. That's not the same thing."
- **Fix:** Replace snapshots with explicit assertions on the values that matter.

### Missing Coverage Config

- **Severity:** info
- **Detect:** No `coverage` configuration in `jest.config.*`, `vitest.config.*`, or `package.json`.
- **Deduction:** -1 point
- **Roast:** "You can't improve what you don't measure. Set up coverage and face the truth."
- **Fix:** Add coverage configuration with thresholds. Track lines, branches, functions, and statements.

## Scoring

- **Base score:** 100
- **Apply deductions** per finding (each instance counts separately)
- **Minimum score:** 0
- **Weight:** 1.0x normal, 1.5x in sensei mode
