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
- **Roast:** "No lockfile? Every install is a surprise party — and not the fun kind."
- **Fix:** Run the package manager's install command to generate a lockfile and commit it to version control.

### Wildcard Versions

- **Severity:** error
- **Detect:** Dependencies specified with `*`, `latest`, or empty version strings in the manifest.
- **Deduction:** -10 points
- **Roast:** "Version '*' means 'whatever you feel like today, npm'. That's not dependency management, that's gambling."
- **Fix:** Pin dependencies to specific semver ranges (e.g., `^2.1.0` or `~3.4.5`).

### Excessive Dependencies

- **Severity:** warning
- **Detect:** More than 30 direct dependencies listed in the manifest. Count both `dependencies` and `devDependencies` separately; flag whichever exceeds the threshold.
- **Deduction:** -4 points
- **Roast:** "Your package.json has {n} dependencies. That's not an app, that's an ecosystem."
- **Fix:** Audit dependencies for overlap and unused packages. Remove what you don't need. Consider lighter alternatives.

### Duplicate Functionality

- **Severity:** warning
- **Detect:** Multiple packages that serve the same purpose installed simultaneously. Common overlaps:
  - HTTP clients: `axios` + `node-fetch` + `got` + `undici`
  - Date libs: `moment` + `dayjs` + `date-fns` + `luxon`
  - Utility belts: `lodash` + `underscore` + `ramda`
  - Schema validation: `joi` + `yup` + `zod` + `ajv`
- **Deduction:** -4 points
- **Roast:** "You have both axios AND node-fetch. Pick a side. This isn't a buffet."
- **Fix:** Standardize on one library per concern and remove the duplicates.

### DevDependencies in Dependencies

- **Severity:** warning
- **Detect:** Test frameworks, linters, or build tools listed under `dependencies` instead of `devDependencies`. Common offenders: `jest`, `mocha`, `eslint`, `prettier`, `typescript`, `webpack`, `vite`, `biome`.
- **Deduction:** -4 points
- **Roast:** "Jest in production dependencies? Your users don't need to run your tests."
- **Fix:** Move development-only packages to `devDependencies`.

### Outdated Major Versions

- **Severity:** warning
- **Detect:** Dependencies that are 2 or more major versions behind the latest published release.
- **Deduction:** -4 points
- **Roast:** "You're on v2 and the world is on v5. That's not stability, that's denial."
- **Fix:** Review changelogs, update incrementally, and run tests after each major bump.

### Direct GitHub Dependencies

- **Severity:** warning
- **Detect:** Dependencies pointing to GitHub URLs, Git repositories, or tarball links instead of a registry version string.
- **Deduction:** -4 points
- **Roast:** "Depending on a GitHub URL is one PR away from breaking your build."
- **Fix:** Publish the package to a registry or fork it under your own scope. If a Git dependency is truly necessary, pin it to a specific commit SHA.

### Unpinned Peer Dependencies

- **Severity:** info
- **Detect:** Peer dependencies with overly broad ranges (e.g., `>=16.0.0` with no upper bound) that may allow incompatible versions.
- **Deduction:** -1 point
- **Roast:** "Your peer dependency range is so wide it could match versions that haven't been invented yet."
- **Fix:** Narrow peer dependency ranges to tested major versions (e.g., `^18.0.0 || ^19.0.0`).

### No Engine Specification

- **Severity:** info
- **Detect:** Missing `engines` field in `package.json`, or no equivalent runtime version constraint in other ecosystems.
- **Deduction:** -1 point
- **Roast:** "No engines field? Brave of you to assume your code runs on every Node version ever released."
- **Fix:** Add an `engines` field specifying the minimum supported runtime version.

### Unused Scripts

- **Severity:** info
- **Detect:** Scripts in `package.json` that reference files or binaries that do not exist in the project.
- **Deduction:** -1 point
- **Roast:** "You have a 'deploy' script pointing to a file that doesn't exist. Aspirational."
- **Fix:** Remove or update stale script entries.

### Missing License Check

- **Severity:** info
- **Detect:** No `license` field in the manifest, or dependencies with licenses incompatible with the project's license (e.g., GPL dependency in an MIT project).
- **Deduction:** -1 point
- **Roast:** "No license field. Schrodinger's open source — nobody knows if they can use it."
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
