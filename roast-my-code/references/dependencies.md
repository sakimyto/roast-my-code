# Dependencies Reference

Reference file for dependency-related code roasts.

## Activation

Active when the project contains a package manifest file:
`package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `Gemfile`, or similar.

---

## Checks

### No Lockfile

- **Severity:** critical
- **Detect:** Project has a package manifest but no corresponding lockfile (e.g., `package.json` without `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, or `bun.lockb`).
- **Deduction:** -20 points
- **Roast (en):** "No lockfile. Every `npm install` is a surprise party where the surprise is 'will it build today?' Spoiler: probably not."
  - **Roast (ja):** "ロックファイルがないんですけど、それって毎回`npm install`がガチャになるってことですよね。なんだろう、再現性のないビルド、やめてもらっていいですか。"
- **Fix:** Run the package manager's install command to generate a lockfile and commit it to version control.

### Wildcard Versions

- **Severity:** error
- **Detect:** Dependencies specified with `*`, `latest`, or empty version strings in the manifest.
- **Deduction:** -10 points
- **Roast (en):** "Version `*` in your dependencies. That's not semver — that's a YOLO deploy strategy. You've outsourced your stability to chaos."
  - **Roast (ja):** "バージョンに`*`って書いてあるんですけど、それって依存管理じゃなくてロシアンルーレットですよね。せめてsemver指定してもらっていいですか。"
- **Fix:** Pin dependencies to specific semver ranges (e.g., `^2.1.0` or `~3.4.5`).

### Excessive Dependencies

- **Severity:** warning
- **Detect:** More than 30 direct dependencies listed in the manifest. Count both `dependencies` and `devDependencies` separately; flag whichever exceeds the threshold.
- **Deduction:** -4 points
- **Roast (en):** "{n} direct dependencies. That's not a project — that's a small nation-state. Your `node_modules` folder has its own gravitational field."
  - **Roast (ja):** "直接依存が{n}個あるんですけど、それってプロジェクトじゃなくてエコシステムですよね。`node_modules`に住所つけたほうがいいんじゃないですかね。"
- **Fix:** Audit dependencies for overlap and unused packages. Remove what you don't need. Consider lighter alternatives.

### Duplicate Functionality

- **Severity:** warning
- **Detect:** Multiple packages that serve the same purpose installed simultaneously. Common overlaps:
  - HTTP clients: `axios` + `node-fetch` + `got` + `undici`
  - Date libs: `moment` + `dayjs` + `date-fns` + `luxon`
  - Utility belts: `lodash` + `underscore` + `ramda`
  - Schema validation: `joi` + `yup` + `zod` + `ajv`
- **Deduction:** -4 points
- **Roast (en):** "You have axios AND node-fetch installed. Pick a side — this isn't a dependency buffet. Even HTTP clients deserve monogamy."
  - **Roast (ja):** "axiosとnode-fetch両方入ってるんですけど、HTTPクライアントを二股かけるの、やめてもらっていいですか。どっちか1つに決めてください。"
- **Fix:** Standardize on one library per concern and remove the duplicates.

### DevDependencies in Dependencies

- **Severity:** warning
- **Detect:** Test frameworks, linters, or build tools listed under `dependencies` instead of `devDependencies`. Common offenders: `jest`, `mocha`, `eslint`, `prettier`, `typescript`, `webpack`, `vite`, `biome`.
- **Deduction:** -4 points
- **Roast (en):** "Jest in production `dependencies`. Your users don't need to run your tests — though given this code, maybe they should."
  - **Roast (ja):** "Jestが`dependencies`に入ってるんですけど、本番環境でテスト走らせたいんですかね。それって引っ越し先に工具箱ごと持っていくようなものですよね。"
- **Fix:** Move development-only packages to `devDependencies`.

### Outdated Major Versions

- **Severity:** warning
- **Detect:** Dependencies that are 2 or more major versions behind the latest published release.
- **Deduction:** -4 points
- **Roast (en):** "You're on v2, the world is on v5. That's not 'stability' — that's mass denial with a side of security vulnerabilities."
  - **Roast (ja):** "世の中v5なのにまだv2使ってるんですけど、それって安定性じゃなくて現実逃避ですよね。チェンジログ読んでもらっていいですか。"
- **Fix:** Review changelogs, update incrementally, and run tests after each major bump.

### Direct GitHub Dependencies

- **Severity:** warning
- **Detect:** Dependencies pointing to GitHub URLs, Git repositories, or tarball links instead of a registry version string.
- **Deduction:** -4 points
- **Roast (en):** "Depending on a GitHub URL. Your build is one force-push away from becoming a crime scene. Pin to a commit or publish to a registry."
  - **Roast (ja):** "GitHub URLに直接依存してるんですけど、それって相手がforce-pushした瞬間にビルド壊れますよね。なんだろう、レジストリに公開するっていう文明的な方法、ご存知ないですかね。"
- **Fix:** Publish the package to a registry or fork it under your own scope. If a Git dependency is truly necessary, pin it to a specific commit SHA.

### Unpinned Peer Dependencies

- **Severity:** info
- **Detect:** Peer dependencies with overly broad ranges (e.g., `>=16.0.0` with no upper bound) that may allow incompatible versions.
- **Deduction:** -1 point
- **Roast (en):** "Your peer dependency range is so wide it could match versions from a parallel universe. `>=16.0.0` with no ceiling is not 'flexible' — it's reckless."
  - **Roast (ja):** "peer dependencyの範囲が広すぎて、まだリリースされてないバージョンまでマッチしますよね。それって互換性テストしてないって自白してるのと同じなんですけど。"
- **Fix:** Narrow peer dependency ranges to tested major versions (e.g., `^18.0.0 || ^19.0.0`).

### No Engine Specification

- **Severity:** info
- **Detect:** Missing `engines` field in `package.json`, or no equivalent runtime version constraint in other ecosystems.
- **Deduction:** -1 point
- **Roast (en):** "No `engines` field. Bold of you to assume your code runs on every Node.js version from v0.10 to v22. It doesn't, by the way."
  - **Roast (ja):** "`engines`フィールドがないんですけど、それってどのNodeバージョンでも動くと思ってるってことですよね。動かないですよ、普通に。"
- **Fix:** Add an `engines` field specifying the minimum supported runtime version.

### Unused Scripts

- **Severity:** info
- **Detect:** Scripts in `package.json` that reference files or binaries that do not exist in the project.
- **Deduction:** -1 point
- **Roast (en):** "A `deploy` script pointing to a file that doesn't exist. That's not a script — that's a vision board."
  - **Roast (ja):** "`deploy`スクリプトの参照先のファイルが存在しないんですけど、それってもはやスクリプトじゃなくて願望ですよね。"
- **Fix:** Remove or update stale script entries.

### Missing License Check

- **Severity:** info
- **Detect:** No `license` field in the manifest, or dependencies with licenses incompatible with the project's license (e.g., GPL dependency in an MIT project).
- **Deduction:** -1 point
- **Roast (en):** "No license field. Schrodinger's open source — it's simultaneously free to use and a potential lawsuit. Nobody knows until someone tries."
  - **Roast (ja):** "ライセンスの指定がないんですけど、それってシュレディンガーのオープンソースですよね。使っていいのか悪いのか、訴えられるまで誰にもわからないっていう。"
- **Fix:** Add a `license` field and audit dependency licenses for compatibility.

---

## Scoring

| Parameter     | Value |
|---------------|-------|
| Starting score | 100  |
| Minimum score  | 0    |
| Weight         | 1.0x |

Deductions are applied per finding. Multiple instances of the same check
(e.g., 3 wildcard versions) each incur the listed deduction independently.
