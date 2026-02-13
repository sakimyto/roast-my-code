# Git Hygiene Checks

## Activation

Conditional: Active only when `.git/` directory exists.

## Check Items

### No .gitignore

- **Severity:** critical
- **Detect:** Use Glob to check for `.gitignore` in the project root. If the file does not exist at all, flag immediately. A project without `.gitignore` is guaranteed to accumulate tracked junk.
- **Deduction:** -20 points
- **Roast (en):** "No `.gitignore`. You're sharing everything with the world -- OS metadata, editor configs, build artifacts, your hopes and dreams."
  - **Roast (ja):** "`.gitignore`がないって、OSのメタデータもエディタ設定もビルド成果物も全部世界に公開してるんですよね。情報公開請求される前に自分から全部出すスタイル、嫌いじゃないですけど。"
- **Fix:** Create a `.gitignore` immediately. Use `gitignore.io` or GitHub's template collection to generate one for your language and framework. At minimum, ignore OS files, editor configs, dependency directories, build output, and `.env` files.

### Committed node_modules

- **Severity:** critical
- **Detect:** Run `git ls-files node_modules` via Bash to check if any files under `node_modules/` are tracked by git. Also Glob for `node_modules/` inside the repository. If any tracked files are found, flag with the count and estimated size.
- **Deduction:** -20 points
- **Roast (en):** "You committed `node_modules`. That's {n}MB of 'I don't understand how npm works.' Every install recreates this -- that's literally the point."
  - **Roast (ja):** "`node_modules`をコミットしたんですか。{n}MBの『npm installの意味を理解してません』宣言ですよね、それ。"
- **Fix:** Add `node_modules/` to `.gitignore`. Remove tracked files with `git rm -r --cached node_modules/`. Commit the removal. Ensure `package-lock.json` or equivalent lockfile is committed so installs are reproducible.

### Large Files Tracked

- **Severity:** error
- **Detect:** Run `git ls-files -z | xargs -0 ls -la` via Bash and filter for files exceeding 1MB. Also check for common binary extensions tracked in git: `.zip`, `.tar.gz`, `.jar`, `.exe`, `.dll`, `.so`, `.dylib`, `.png`, `.jpg`, `.mp4`, `.sqlite`, `.pdf` using `git ls-files`.
- **Deduction:** -10 points
- **Roast (en):** "A {size}MB binary in git. Git is not Google Drive. Every clone downloads this blob for all eternity, even after you 'delete' it."
  - **Roast (ja):** "{size}MBのバイナリをgitに入れてますけど、gitはGoogleドライブじゃないんですよ。削除しても全クローンに永遠に付いてくるの、知ってました？"
- **Fix:** Remove large files with `git rm --cached <file>`. Add binary extensions to `.gitignore`. For files that genuinely need versioning, use Git LFS (`git lfs track "*.psd"`). Consider BFG Repo-Cleaner to purge large files from history.

### Secrets in History

- **Severity:** critical
- **Detect:** Run `git log --all --diff-filter=A --name-only --pretty=format:""` via Bash and grep for `.env`, `.pem`, `.key`, `credentials.json`, `service-account.json`, `*_rsa`, `*.p12` patterns. Also run `git log -p --all -S "AKIA"` to search for AWS keys and `git log -p --all -S "BEGIN RSA PRIVATE KEY"` for leaked private keys.
- **Deduction:** -20 points
- **Roast (en):** "Your API key is in git history. Git never forgets. Never. That secret is now part of the permanent archaeological record of this repo."
  - **Roast (ja):** "APIキーがgit履歴に残ってますよ。gitは絶対に忘れないんですよね。そのシークレット、このリポジトリの永久保存版に刻まれちゃってるんですよ。"
- **Fix:** Rotate ALL compromised credentials immediately -- this is non-negotiable. Use `BFG Repo-Cleaner` or `git filter-repo` to purge sensitive files from history. Install `git-secrets` or `pre-commit` hooks to prevent future leaks. Store secrets in environment variables or a secrets manager.

### No Branching Strategy

- **Severity:** warning
- **Detect:** Run `git branch --list` via Bash to enumerate all local branches. If only `main` or `master` exists with no evidence of other branches (also check `git branch -r` for remote branches), flag it. For solo projects with fewer than 10 commits, reduce severity to info.
- **Deduction:** -4 points
- **Roast (en):** "One branch, zero safety nets. You're deploying straight to main like it's a solo hackathon that never ended."
  - **Roast (ja):** "ブランチ1本、セーフティネットゼロですか。終わらない個人ハッカソンのノリでmainに直接pushし続けてるんですよね。"
- **Fix:** Adopt a branching strategy. Even a simple GitHub Flow (feature branches + main) is better than committing everything to main. Create branches for features, fixes, and experiments. Merge via pull requests.

### Giant Commits

- **Severity:** warning
- **Detect:** Run `git log --oneline --shortstat -20` via Bash. Parse the output for commits with more than 30 files changed. Also flag commits with more than 1000 lines added in a single commit (excluding lockfiles and auto-generated code).
- **Deduction:** -4 points
- **Roast (en):** "This commit touches {n} files. That's not a commit -- that's a deployment. `git bisect` just filed for emotional damages."
  - **Roast (ja):** "1コミットで{n}ファイル変更って、それコミットじゃなくてデプロイですよね。`git bisect`が精神的苦痛で訴訟起こしてますよ。"
- **Fix:** Break work into small, focused commits. Each commit should represent one logical change. Use `git add -p` to stage hunks selectively. If you find yourself writing "and" in a commit message, that's two commits.

### Meaningless Commit Messages

- **Severity:** warning
- **Detect:** Run `git log --oneline -20` via Bash. Flag messages that match: exact strings `fix`, `update`, `wip`, `asdf`, `.`, `test`, `stuff`, `changes`, `commit`, or messages shorter than 4 characters. Also flag repetitive identical messages in sequence.
- **Deduction:** -4 points
- **Roast (en):** "Commit message: 'fix'. Fix what? My will to review your code? Your git log reads like a drunk person's autocomplete."
  - **Roast (ja):** "コミットメッセージが'fix'って、何をfixしたんですか。僕のコードレビューする気力をですか？ git logが酔っ払いの予測変換みたいになってますよね。"
- **Fix:** Write commit messages that explain WHY, not just WHAT. Follow Conventional Commits (`feat:`, `fix:`, `refactor:`) or a similar convention. A good message completes the sentence: "If applied, this commit will ___."

### No Branch Protection

- **Severity:** info
- **Detect:** Run `git log --oneline --first-parent main -10` via Bash and check for non-merge commits pushed directly to main. If all recent commits on main are direct pushes (no merge commits from PRs), it suggests no branch protection is in place.
- **Deduction:** -1 point
- **Roast (en):** "Pushing directly to main with zero review process. One typo away from deploying a disaster and nobody's there to catch it."
  - **Roast (ja):** "レビューなしでmainに直pushですか。タイポ1個で本番が爆発しても誰も止めてくれないんですよね、それ。"
- **Fix:** Enable branch protection rules on your main branch. Require pull request reviews before merging. Enable status checks. Even solo developers benefit from the forced pause of a PR workflow.

### Merge Commits Galore

- **Severity:** info
- **Detect:** Run `git log --oneline -20` via Bash and count commits matching `^[a-f0-9]+ Merge branch`. If merge commits exceed 30% of recent history, flag it. Excessive merge commits clutter the log and obscure the actual development history.
- **Deduction:** -1 point
- **Roast (en):** "Your git log is 40% merge commits. It reads like a novel where every other page says 'and then the chapters were combined.'"
  - **Roast (ja):** "git logの4割がマージコミットって、小説の1ページおきに『そして章が統合された』って書いてあるようなものですよね。読む気なくしますよ。"
- **Fix:** Use `git pull --rebase` instead of `git pull` to avoid unnecessary merge commits. Configure globally with `git config --global pull.rebase true`. For feature branches, rebase onto main before merging. Reserve merge commits for actual integration points.

### Stale Branches

- **Severity:** info
- **Detect:** Run `git for-each-ref --sort=committerdate --format='%(refname:short) %(committerdate:relative)' refs/heads/` via Bash. Flag branches with last commit older than 30 days. Exclude `main` and `master` from this check.
- **Deduction:** -1 point
- **Roast (en):** "You have branches older than some startups. '{branch}' was last touched 4 months ago. Either finish it or give it a proper funeral."
  - **Roast (ja):** "スタートアップより長生きしてるブランチがありますよね。'{branch}'、4ヶ月前から放置されてるんですけど、完成させるかお葬式するか、どっちかにしてもらっていいですか。"
- **Fix:** Delete merged branches with `git branch -d <branch>`. Force-delete abandoned branches with `git branch -D <branch>`. Clean up remote tracking branches with `git fetch --prune`. Make branch cleanup part of your regular workflow.

### Missing README

- **Severity:** warning
- **Detect:** Use Glob to check for `README.md`, `README`, `README.txt`, `README.rst` in the project root. If none exist, flag it. A repo without a README is a repo that nobody (including future you) will understand.
- **Deduction:** -4 points
- **Roast (en):** "No README. This repo is a mystery box -- and not the fun kind. How does anyone, including future-you, know what this does?"
  - **Roast (ja):** "READMEがないって、このリポジトリ、開けてもワクワクしない方の福袋ですよね。未来の自分も含めて、誰がこれの使い方わかるんですか。"
- **Fix:** Create a `README.md` with at minimum: project name, one-sentence description, how to install, how to run, and how to contribute. A good README is the difference between a project and a pile of files.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Weight 1.0x.
