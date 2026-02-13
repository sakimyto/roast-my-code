# Security Checks

## Activation

Always active.

## Check Items

### Hardcoded Secrets

- **Severity:** critical
- **Detect:** Grep for `password\s*=\s*["']`, `api[_-]?key\s*=\s*["']`, `secret\s*=\s*["']`, `token\s*=\s*["']`, and AWS access key pattern `AKIA[0-9A-Z]{16}`. Also scan for `BEGIN RSA PRIVATE KEY`, `Bearer [A-Za-z0-9\-._~+/]+=*`, and hex strings longer than 32 characters assigned to suspicious variable names.
- **Deduction:** -20 points
- **Roast (en):** "You hardcoded your API key in source code pushed to GitHub. You didn't leave the door unlocked -- you published the key on a billboard and added a QR code."
  - **Roast (ja):** "なんだろう、APIキーをソースコードにベタ書きしてGitHubにpushするの、やめてもらっていいですか。それって鍵を玄関に貼り付けてるのと同じですよね。"
- **Fix:** Move all secrets to environment variables or a secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler). Use `.env` files locally and ensure they are in `.gitignore`. Rotate any secrets that were ever committed.

### SQL Injection

- **Severity:** critical
- **Detect:** Grep for string concatenation with SQL keywords: `["']\s*\+.*SELECT`, `["']\s*\+.*INSERT`, `["']\s*\+.*UPDATE`, `["']\s*\+.*DELETE`, template literals containing SQL like `` `SELECT.*\$\{` ``, and `f"SELECT` or `f"INSERT` patterns in Python. Also check for `query\(["']SELECT.*\+` and `.execute\(.*%s` without parameterized form.
- **Deduction:** -20 points
- **Roast (en):** "String concatenation in SQL queries -- Little Bobby Tables sends his regards. Your database is one form submission away from becoming a crime scene."
  - **Roast (ja):** "それって文字列結合でSQL組み立ててますよね。ユーザーが `'; DROP TABLE users; --` って入力したらどうするんですか。いや、聞いてるんですけど。"
- **Fix:** Use parameterized queries or prepared statements exclusively. Employ an ORM (Prisma, SQLAlchemy, ActiveRecord) or query builder (Knex, Diesel) that handles escaping. Never interpolate user input into raw SQL strings.

### Command Injection

- **Severity:** critical
- **Detect:** Grep for `exec\(.*\+`, `spawn\(.*\+`, `system\(.*\+`, `child_process`, `os\.system\(.*\+`, `subprocess\.call\(.*shell=True`, `eval\(.*req\.`, and backtick execution with variable interpolation. Check for any user-controlled input flowing into shell execution functions.
- **Deduction:** -20 points
- **Roast (en):** "User input piped straight into a shell command. You didn't build a feature -- you built a reverse shell as a service, and it's free tier only."
  - **Roast (ja):** "ユーザー入力をそのままシェルに渡してるの、それってリモートコード実行をサービスとして提供してますよね。バグじゃなくて機能ってことですか。"
- **Fix:** Avoid shell execution entirely where possible. Use language-native libraries instead of shelling out. If shell commands are unavoidable, use `execFile`/`spawn` with argument arrays (no shell interpolation), validate and sanitize all inputs with strict allowlists, and never use `shell=True` in Python subprocess calls.

### XSS Vulnerabilities

- **Severity:** critical
- **Detect:** Grep for `innerHTML\s*=`, `dangerouslySetInnerHTML`, `v-html`, `\[innerHTML\]`, `document\.write\(`, `\$\(.*\)\.html\(`, and `{{{` (unescaped Handlebars). Also check for user input rendered without escaping in template engines: `<%-`, `| safe`, `mark_safe`.
- **Deduction:** -20 points
- **Roast (en):** "`dangerouslySetInnerHTML` -- the name is literally a warning label and you used it anyway. This is the dev equivalent of reading 'do not eat' on silica gel and reaching for a spoon."
  - **Roast (ja):** "なんだろう、`dangerouslySetInnerHTML`って名前に『危険』って書いてあるのに使うの、『押すな』って書いてあるボタン押す人と同じですよね。"
- **Fix:** Use framework-safe rendering (React JSX auto-escapes, Vue `{{ }}` auto-escapes). When raw HTML is required, sanitize with DOMPurify or a similar library. Implement a strict Content Security Policy (CSP) as defense in depth.

### Insecure Dependencies

- **Severity:** error
- **Detect:** Check `package.json`, `requirements.txt`, `Gemfile`, `go.mod` for known vulnerable packages like `event-stream`, `ua-parser-js` (compromised versions), `lodash` < 4.17.21, `minimist` < 1.2.6. Run `npm audit`, `pip-audit`, or `bundle-audit` via Bash. Also flag wildcard version ranges `"*"` and overly broad ranges like `">="` without upper bounds.
- **Deduction:** -10 points
- **Roast (en):** "Your dependency tree is a CVE museum -- half these packages were compromised before your `git init`. `npm audit` is free, unlike the incident response retainer you'll need."
  - **Roast (ja):** "それって依存パッケージが脆弱性の展示会みたいになってますよね。`npm audit`はタダなんで、一回実行してもらっていいですか。インシデント対応費よりは安いと思いますよ。"
- **Fix:** Run `npm audit fix`, `pip-audit`, or the equivalent for your ecosystem. Pin dependency versions. Set up Dependabot or Renovate for automated updates. Remove unused dependencies. Prefer well-maintained packages with active security response teams.

### Missing Authentication

- **Severity:** error
- **Detect:** Grep for route definitions without auth middleware: `app\.(get|post|put|patch|delete)\(["'/]` without adjacent `auth`, `authenticate`, `protect`, `requireAuth`, `isAuthenticated`, or `jwt` middleware. Check for `@app\.route` in Flask without `@login_required`. Look for API endpoints in controller files with no auth decorator or middleware reference.
- **Deduction:** -10 points
- **Roast (en):** "Endpoints wide open with no auth. You didn't build an API -- you built a public FTP server with extra steps and a Swagger page."
  - **Roast (ja):** "認証なしでエンドポイント全開放してるの、それってAPIじゃなくて公衆トイレですよね。誰でも入れるし、何されても文句言えないですよ。"
- **Fix:** Apply authentication middleware globally and explicitly opt-out for public routes rather than opting-in per route. Use established auth libraries (Passport.js, Flask-Login, Spring Security). Implement role-based access control (RBAC) for sensitive operations.

### Insecure Randomness

- **Severity:** warning
- **Detect:** Grep for `Math\.random\(\)` used near security contexts: token generation, password reset, session IDs, OTP, nonce, CSRF. Also check for `random\.random\(\)` in Python and `rand\(\)` in C/C++ without cryptographic seeding. Pattern: `Math\.random` in files containing `token`, `session`, `auth`, `secret`, `password`, `otp`.
- **Deduction:** -4 points
- **Roast (en):** "`Math.random()` for security tokens -- that's like locking your front door with vibes. `crypto.getRandomValues()` has been right there this whole time, waiting patiently."
  - **Roast (ja):** "セキュリティトークンに`Math.random()`使ってるの、それってサイコロ振って暗証番号決めてるのと同じですよね。`crypto.getRandomValues()`知らないんですか。"
- **Fix:** Use `crypto.getRandomValues()` or `crypto.randomUUID()` in JavaScript, `secrets` module in Python, `SecureRandom` in Java/Ruby. Never use `Math.random()`, `random.random()`, or `rand()` for anything security-related.

### Sensitive Data in Logs

- **Severity:** warning
- **Detect:** Grep for logging statements containing sensitive variable names: `console\.log\(.*password`, `logger\..*(token|secret|apiKey|creditCard|ssn)`, `print\(.*password`, `log\..*(authorization|cookie)`. Also check for `JSON\.stringify\(req\)`, `JSON\.stringify\(user\)` which may dump entire objects containing sensitive fields.
- **Deduction:** -4 points
- **Roast (en):** "You're logging passwords in plaintext. Your log aggregator just became the most valuable target in your entire infrastructure -- congratulations on the accidental honeypot."
  - **Roast (ja):** "パスワードを平文でログに出力してるの、それってわざわざ攻撃者のためにカンペ用意してるようなものですよね。親切すぎませんか。"
- **Fix:** Implement structured logging with explicit field selection. Create a sanitization layer that redacts sensitive fields before logging. Use logging libraries that support field-level redaction. Never log full request/response objects without filtering.

### Missing HTTPS

- **Severity:** warning
- **Detect:** Grep for `http://` URLs in production config files, environment templates, and API base URL definitions. Pattern: `http://` in files matching `*config*`, `*env*`, `*.yaml`, `*.yml`, `*.toml`, `*.properties`. Exclude `http://localhost`, `http://127.0.0.1`, `http://0.0.0.0`, and comments.
- **Deduction:** -4 points
- **Roast (en):** "HTTP in production in this day and age. You're sending data in a clear plastic bag and wondering why people can read it. Let's Encrypt is free -- like, actually free."
  - **Roast (ja):** "いまどきHTTPで本番運用してるの、それって葉書に暗証番号書いて郵送してるのと同じですよね。Let's Encryptは無料なんですけど、知ってました？"
- **Fix:** Enforce HTTPS everywhere. Use TLS certificates from Let's Encrypt (free) or your cloud provider. Set up HTTP-to-HTTPS redirects. Add HSTS headers. Update all hardcoded URLs to use `https://`.

### Permissive CORS

- **Severity:** warning
- **Detect:** Grep for `Access-Control-Allow-Origin.*\*`, `cors\(\)` without options (Express default allows all), `origin:\s*true`, `origin:\s*\*`, and `credentials:\s*true` combined with wildcard origin. Check for `@CrossOrigin` without explicit origins in Java Spring.
- **Deduction:** -4 points
- **Roast (en):** "CORS set to `*` -- any website on the internet can now make authenticated requests to your API. That's not an access policy, that's an open house with free Wi-Fi."
  - **Roast (ja):** "CORSを`*`にしてるの、それってインターネット上の全サイトにAPIの鍵渡してるのと同じですよね。アクセス制御ってご存知ですか。"
- **Fix:** Explicitly allowlist trusted origins. Never use `*` with `credentials: true`. Configure CORS per-environment (strict in production, relaxed in development). Use a CORS middleware that validates the `Origin` header against an allowlist.

### Missing Security Headers

- **Severity:** info
- **Detect:** Check server configuration and middleware setup for absence of: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`. Grep for `helmet` (Node.js) or equivalent security header middleware. If no security header middleware is found, flag it.
- **Deduction:** -1 point
- **Roast (en):** "Zero security headers. Your HTTP responses are going out into the world completely naked. Helmet.js is literally one line of code -- one."
  - **Roast (ja):** "セキュリティヘッダーが一個もないんですけど、それってレスポンスを全裸で送り出してるようなものですよね。Helmet.jsは1行で入れられるんで、やってもらっていいですか。"
- **Fix:** Use `helmet` (Express/Node.js), `django-csp` (Django), `secure_headers` (Rails), or configure headers in your reverse proxy (Nginx, Cloudflare). At minimum set: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`.

### .env in Repository

- **Severity:** critical
- **Detect:** Use Glob to check for `.env`, `.env.local`, `.env.production`, `.env.*.local` files tracked in git. Run `git ls-files .env*` via Bash to confirm they are committed. Also grep `.gitignore` to verify `.env` patterns are present. If `.env` files exist AND are not in `.gitignore`, flag immediately.
- **Deduction:** -20 points
- **Roast (en):** "You committed your `.env` file -- the one with all the secrets -- to git, which remembers everything forever. Hope you enjoy rotating credentials at 2 AM on a Saturday."
  - **Roast (ja):** "`.env`ファイルをgitにコミットしてますよね。gitって全部履歴に残るの知ってます？ シークレット全部ローテーションする作業、深夜2時にやることになりますけど大丈夫ですか。"
- **Fix:** Add `.env*` to `.gitignore` immediately. Remove tracked `.env` files with `git rm --cached .env`. Rotate ALL secrets that were ever committed (they are in git history even after removal). Use `git-secrets` or `pre-commit` hooks to prevent future commits of secret files. Consider using `BFG Repo-Cleaner` to purge history.

### SSRF (Server-Side Request Forgery)

- **Severity:** critical
- **Detect:** Grep for HTTP client calls where the URL argument derives from user input. Patterns: `fetch\(.*req\.(query|params|body)`, `axios\.(get|post)\(.*req\.`, `http\.get\(.*req\.`, `got\(.*req\.`. Also flag any HTTP client call where the URL is a function parameter without allowlist validation.
- **Deduction:** -20 points
- **Roast (en):** "You're passing user input directly into `fetch()`. Congrats, your server is now a proxy for attacking your own internal network. That's SSRF — giving strangers your house keys and hoping they only use the front door."
  - **Roast (ja):** "ユーザー入力をそのまま`fetch()`に渡してるの、それってサーバーを踏み台にして内部ネットワーク攻撃できちゃうんですけど。SSRFって知ってます？ 知らないなら今すぐ検索してもらっていいですか。"
- **Fix:** Validate and allowlist URLs before making server-side requests. Block requests to private IP ranges (10.x, 172.16-31.x, 192.168.x, 127.x, 169.254.x). Use a dedicated HTTP client with SSRF protections.

### JWT Vulnerabilities

- **Severity:** error
- **Detect:** Grep for `jwt.sign` without `expiresIn` option, `jwt.verify` without `algorithms` option, `localStorage.setItem` containing `token`/`jwt`/`auth`, and `jwt.decode` used where `jwt.verify` should be. Patterns: `jwt\.sign\(` without `expiresIn`, `localStorage\.setItem.*token`, `jwt\.decode`.
- **Deduction:** -10 points
- **Roast (en):** "Your JWT never expires. That token is immortal — if it leaks, the attacker has access until the heat death of the universe. Also, storing it in localStorage is XSS candy."
  - **Roast (ja):** "JWTに有効期限ないんですけど、それって一回漏洩したら宇宙の終わりまでアクセスされ放題ってことですよね。あとlocalStorageに保存するの、XSSにプレゼントしてるのと同じですよ。"
- **Fix:** Always set `expiresIn` (short-lived: 15min for access tokens). Specify `algorithms` in verify to prevent alg:none attacks. Store tokens in httpOnly cookies, not localStorage. Use refresh token rotation.

### Mass Assignment

- **Severity:** error
- **Detect:** Grep for ORM create/update calls receiving full request body. Patterns: `.create(req.body)`, `.update(req.body)`, `new Model(req.body)`, `Object.assign(model, req.body)`, Prisma `.create({ data: req.body })`.
- **Deduction:** -10 points
- **Roast (en):** "Passing `req.body` straight into your ORM. Any user can add `{ isAdmin: true }` to the request and promote themselves. That's not a feature — it's a self-service privilege escalation."
  - **Roast (ja):** "`req.body`をそのままORMに渡してるの、リクエストに`isAdmin: true`入れるだけで誰でも管理者になれますよね。それって権限昇格のセルフサービスですよ。"
- **Fix:** Explicitly allowlist fields. Use DTOs, pick/omit utilities, or schema validation (Zod, Joi) to extract only expected fields from request bodies. Never pass raw `req.body` to create/update calls.

### Open Redirect

- **Severity:** warning
- **Detect:** Grep for redirect with user-controlled target. Patterns: `redirect(req.query`, `redirect(req.body`, `redirect(req.params`, `window.location\s*=\s*`, `location.replace(`. Flag when the redirect target comes from a query parameter or form input without validation.
- **Deduction:** -4 points
- **Roast (en):** "You redirect to whatever URL the user passes in the query string. Phishing attackers love this — your domain becomes their credibility laundering service."
  - **Roast (ja):** "クエリパラメータのURLにそのままリダイレクトしてるの、フィッシング攻撃の踏み台に使われますよね。あなたのドメインが信頼度ロンダリングに利用されるんですよ。"
- **Fix:** Validate redirect targets against an allowlist of trusted domains. Use relative paths where possible. Never redirect to user-supplied absolute URLs without validation.

### Prototype Pollution

- **Severity:** error
- **Detect:** Grep for deep merge/assign with user input. Patterns: `_.merge(.*req\.`, `_.defaultsDeep(.*req\.`, `Object.assign(.*req\.body`, `deepMerge`. Also check for missing `__proto__` / `constructor` key sanitization on user-facing object merges.
- **Deduction:** -10 points
- **Roast (en):** "Deep-merging user input into objects. One `{\"__proto__\": {\"isAdmin\": true}}` and every object in your app gets admin powers. Prototype pollution is a runtime virus."
  - **Roast (ja):** "ユーザー入力をdeep mergeしてるの、`__proto__`にisAdmin突っ込まれたら全オブジェクトが管理者になるんですけど。プロトタイプ汚染って聞いたことないですかね。"
- **Fix:** Sanitize input keys to reject `__proto__`, `constructor`, `prototype`. Use `Object.create(null)` for dictionaries. Use `Map` instead of plain objects for user-supplied keys.

### Path Traversal

- **Severity:** critical
- **Detect:** Grep for file system operations with user input. Patterns: `readFile(req.`, `readFile(.*\+.*req`, `createReadStream(req.`, `fs\.\w+(req\.(query|params|body))`, `path.join(.*req.`, `sendFile(req.`. Flag any file I/O where the path argument includes user-controlled data.
- **Deduction:** -20 points
- **Roast (en):** "You're building a file path from user input. `../../etc/passwd` says hi. Path traversal is the original hack — it was old when your framework was born."
  - **Roast (ja):** "ファイルパスにユーザー入力使ってるの、`../../etc/passwd`って入れられたらサーバーの中身丸見えですよね。パストラバーサル、ハッキングの古典中の古典なんですけど。"
- **Fix:** Use `path.resolve()` and verify the result starts with the expected base directory. Never concatenate user input into file paths. Use a mapping (ID to filename) instead of accepting filenames directly.

### Insecure Cookie Configuration

- **Severity:** warning
- **Detect:** Grep for cookie configuration missing security flags. Patterns: `res.cookie(` without `httpOnly`, `secure`, or `sameSite` in the options object. Check session middleware config (express-session, cookie-session) for missing security flags. Also flag `Set-Cookie` headers without these attributes.
- **Deduction:** -4 points
- **Roast (en):** "Auth cookies without HttpOnly or Secure flags. Any XSS can steal them, and they transmit over plain HTTP. You basically put your session tokens on a billboard."
  - **Roast (ja):** "認証Cookieに`HttpOnly`も`Secure`もついてないの、XSSで盗み放題だしHTTPで平文送信ですよね。セッショントークンを看板に貼り出してるのと同じですよ。"
- **Fix:** Set `httpOnly: true` (prevents JS access), `secure: true` (HTTPS only), `sameSite: 'strict'` or `'lax'` (CSRF protection). For session cookies, also set appropriate `maxAge`.

### Timing Attack on Secrets

- **Severity:** warning
- **Detect:** Grep for direct comparison of secrets with `===` or `==`. Patterns: `===.*secret`, `===.*token`, `===.*apiKey`, `===.*password`, comparison of `req.headers.authorization` or `req.headers['x-api-key']` using equality operators instead of `crypto.timingSafeEqual`.
- **Deduction:** -4 points
- **Roast (en):** "Comparing secrets with `===`. An attacker can measure response time differences to guess your token character by character. `crypto.timingSafeEqual` exists for exactly this reason."
  - **Roast (ja):** "シークレットの比較に`===`使ってるの、レスポンス時間の差でトークンを1文字ずつ特定できるんですけど。`crypto.timingSafeEqual`っていう関数、存在するの知ってます？"
- **Fix:** Use `crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b))` for comparing secrets, tokens, API keys, and signatures. Never use `===` or `==` for security-sensitive string comparisons.

### Insecure Deserialization

- **Severity:** error
- **Detect:** Grep for `eval(`, `new Function(`, `vm.runInContext(`, `vm.runInNewContext(`, `unserialize(` with user input. Also flag `yaml.load` without safe schema option. Patterns: `eval\(`, `new Function\(`, `vm\.run`.
- **Deduction:** -10 points
- **Roast (en):** "`eval()` with user input. You've handed your server's soul to whoever sends a request. This is remote code execution as a service — for attackers."
  - **Roast (ja):** "ユーザー入力を`eval()`に渡してるの、それって攻撃者にサーバーのシェルをプレゼントしてるのと同じですよね。なんだろう、`eval`使うのやめてもらっていいですか。"
- **Fix:** Never use `eval()` or `new Function()` with user input. Use `JSON.parse()` with schema validation (Zod, Joi, ajv). For YAML, use safe load options. Consider a sandboxed environment if dynamic code execution is truly needed.

### Missing CSRF Protection

- **Severity:** error
- **Detect:** Check for state-changing endpoints (POST/PUT/DELETE handlers) without CSRF token validation. Grep for form submissions (`<form method="POST"`) without a CSRF token field. Check for missing CSRF middleware: no `csurf`, `csrf-csrf`, `@fastify/csrf-protection`, or framework-native CSRF config. Also flag state-changing GET routes (`app.get` that performs writes/deletes) — GET should be idempotent.
- **Deduction:** -10 points
- **Roast (en):** "No CSRF protection. Any website can make your logged-in users unknowingly transfer funds, delete accounts, or change passwords — all with one invisible form. That's not a feature gap, it's an open invitation."
  - **Roast (ja):** "CSRF対策ないんですけど、他のサイトからログイン中のユーザーに送金させたりアカウント削除させたりできちゃいますよね。なんだろう、CSRFトークンっていう基本的なもの、実装してもらっていいですか。"
- **Fix:** Add CSRF middleware (`csurf` for Express, `@fastify/csrf-protection` for Fastify, built-in for Next.js/Remix). Include CSRF tokens in all forms and AJAX requests. Use `SameSite=Strict` cookies as an additional layer. Never use GET for state-changing operations.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Multiply final score by weight 1.5x.
