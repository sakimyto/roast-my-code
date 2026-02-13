# Security Checks

## Activation

Always active.

## Check Items

### Hardcoded Secrets

- **Severity:** critical
- **Detect:** Grep for `password\s*=\s*["']`, `api[_-]?key\s*=\s*["']`, `secret\s*=\s*["']`, `token\s*=\s*["']`, and AWS access key pattern `AKIA[0-9A-Z]{16}`. Also scan for `BEGIN RSA PRIVATE KEY`, `Bearer [A-Za-z0-9\-._~+/]+=*`, and hex strings longer than 32 characters assigned to suspicious variable names.
- **Deduction:** -20 points
- **Roast:** "Congratulations, you just published your API keys to the entire internet. Hackers are sending you a thank-you card as we speak. Why bother with a vault when you can just hardcode secrets like it's 2005?"
- **Fix:** Move all secrets to environment variables or a secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler). Use `.env` files locally and ensure they are in `.gitignore`. Rotate any secrets that were ever committed.

### SQL Injection

- **Severity:** critical
- **Detect:** Grep for string concatenation with SQL keywords: `["']\s*\+.*SELECT`, `["']\s*\+.*INSERT`, `["']\s*\+.*UPDATE`, `["']\s*\+.*DELETE`, template literals containing SQL like `` `SELECT.*\$\{` ``, and `f"SELECT` or `f"INSERT` patterns in Python. Also check for `query\(["']SELECT.*\+` and `.execute\(.*%s` without parameterized form.
- **Deduction:** -20 points
- **Roast:** "Oh look, you're building SQL queries with string concatenation. Little Bobby Tables called -- he wants to DROP your entire production database. Parameterized queries exist for a reason, but sure, keep rolling the dice."
- **Fix:** Use parameterized queries or prepared statements exclusively. Employ an ORM (Prisma, SQLAlchemy, ActiveRecord) or query builder (Knex, Diesel) that handles escaping. Never interpolate user input into raw SQL strings.

### Command Injection

- **Severity:** critical
- **Detect:** Grep for `exec\(.*\+`, `spawn\(.*\+`, `system\(.*\+`, `child_process`, `os\.system\(.*\+`, `subprocess\.call\(.*shell=True`, `eval\(.*req\.`, and backtick execution with variable interpolation. Check for any user-controlled input flowing into shell execution functions.
- **Deduction:** -20 points
- **Roast:** "You're passing user input straight into a shell command. That's not a feature, that's a remote code execution CVE waiting for its number. Might as well give attackers SSH access and save them the trouble."
- **Fix:** Avoid shell execution entirely where possible. Use language-native libraries instead of shelling out. If shell commands are unavoidable, use `execFile`/`spawn` with argument arrays (no shell interpolation), validate and sanitize all inputs with strict allowlists, and never use `shell=True` in Python subprocess calls.

### XSS Vulnerabilities

- **Severity:** critical
- **Detect:** Grep for `innerHTML\s*=`, `dangerouslySetInnerHTML`, `v-html`, `\[innerHTML\]`, `document\.write\(`, `\$\(.*\)\.html\(`, and `{{{` (unescaped Handlebars). Also check for user input rendered without escaping in template engines: `<%-`, `| safe`, `mark_safe`.
- **Deduction:** -20 points
- **Roast:** "Using innerHTML with user data is like leaving your front door open and putting up a sign that says 'free stuff inside.' Every script kiddie on the planet thanks you for the XSS playground. dangerouslySetInnerHTML has 'dangerously' in the name -- did you think that was just decoration?"
- **Fix:** Use framework-safe rendering (React JSX auto-escapes, Vue `{{ }}` auto-escapes). When raw HTML is required, sanitize with DOMPurify or a similar library. Implement a strict Content Security Policy (CSP) as defense in depth.

### Insecure Dependencies

- **Severity:** error
- **Detect:** Check `package.json`, `requirements.txt`, `Gemfile`, `go.mod` for known vulnerable packages like `event-stream`, `ua-parser-js` (compromised versions), `lodash` < 4.17.21, `minimist` < 1.2.6. Run `npm audit`, `pip-audit`, or `bundle-audit` via Bash. Also flag wildcard version ranges `"*"` and overly broad ranges like `">="` without upper bounds.
- **Deduction:** -10 points
- **Roast:** "Your dependency tree reads like a CVE greatest hits album. You've got packages in here that were compromised before you even started this project. Ever heard of `npm audit`? It's free, unlike the breach response you're going to need."
- **Fix:** Run `npm audit fix`, `pip-audit`, or the equivalent for your ecosystem. Pin dependency versions. Set up Dependabot or Renovate for automated updates. Remove unused dependencies. Prefer well-maintained packages with active security response teams.

### Missing Authentication

- **Severity:** error
- **Detect:** Grep for route definitions without auth middleware: `app\.(get|post|put|patch|delete)\(["'/]` without adjacent `auth`, `authenticate`, `protect`, `requireAuth`, `isAuthenticated`, or `jwt` middleware. Check for `@app\.route` in Flask without `@login_required`. Look for API endpoints in controller files with no auth decorator or middleware reference.
- **Deduction:** -10 points
- **Roast:** "Wide open endpoints with no authentication -- very bold strategy. You didn't build an API, you built a self-service data buffet for anyone with curl. Why even have a login page if half your routes don't check for it?"
- **Fix:** Apply authentication middleware globally and explicitly opt-out for public routes rather than opting-in per route. Use established auth libraries (Passport.js, Flask-Login, Spring Security). Implement role-based access control (RBAC) for sensitive operations.

### Insecure Randomness

- **Severity:** warning
- **Detect:** Grep for `Math\.random\(\)` used near security contexts: token generation, password reset, session IDs, OTP, nonce, CSRF. Also check for `random\.random\(\)` in Python and `rand\(\)` in C/C++ without cryptographic seeding. Pattern: `Math\.random` in files containing `token`, `session`, `auth`, `secret`, `password`, `otp`.
- **Deduction:** -4 points
- **Roast:** "Using Math.random() for security tokens is like using a coin flip to set your bank PIN -- except the coin is transparent and everyone can see it landing. crypto.getRandomValues() is RIGHT THERE."
- **Fix:** Use `crypto.getRandomValues()` or `crypto.randomUUID()` in JavaScript, `secrets` module in Python, `SecureRandom` in Java/Ruby. Never use `Math.random()`, `random.random()`, or `rand()` for anything security-related.

### Sensitive Data in Logs

- **Severity:** warning
- **Detect:** Grep for logging statements containing sensitive variable names: `console\.log\(.*password`, `logger\..*(token|secret|apiKey|creditCard|ssn)`, `print\(.*password`, `log\..*(authorization|cookie)`. Also check for `JSON\.stringify\(req\)`, `JSON\.stringify\(user\)` which may dump entire objects containing sensitive fields.
- **Deduction:** -4 points
- **Roast:** "You're logging passwords in plaintext. Somewhere, a compliance officer just felt a disturbance in the force. Your log aggregator is now the most valuable target in your infrastructure -- congrats on the accidental honeypot."
- **Fix:** Implement structured logging with explicit field selection. Create a sanitization layer that redacts sensitive fields before logging. Use logging libraries that support field-level redaction. Never log full request/response objects without filtering.

### Missing HTTPS

- **Severity:** warning
- **Detect:** Grep for `http://` URLs in production config files, environment templates, and API base URL definitions. Pattern: `http://` in files matching `*config*`, `*env*`, `*.yaml`, `*.yml`, `*.toml`, `*.properties`. Exclude `http://localhost`, `http://127.0.0.1`, `http://0.0.0.0`, and comments.
- **Deduction:** -4 points
- **Roast:** "HTTP in production? What year is this, 2009? You're basically sending data over the wire in a clear plastic bag. Let's Encrypt is free -- FREE -- and you still can't be bothered. Every coffee shop WiFi hacker appreciates your commitment to plaintext."
- **Fix:** Enforce HTTPS everywhere. Use TLS certificates from Let's Encrypt (free) or your cloud provider. Set up HTTP-to-HTTPS redirects. Add HSTS headers. Update all hardcoded URLs to use `https://`.

### Permissive CORS

- **Severity:** warning
- **Detect:** Grep for `Access-Control-Allow-Origin.*\*`, `cors\(\)` without options (Express default allows all), `origin:\s*true`, `origin:\s*\*`, and `credentials:\s*true` combined with wildcard origin. Check for `@CrossOrigin` without explicit origins in Java Spring.
- **Deduction:** -4 points
- **Roast:** "CORS set to allow literally everyone? You didn't build a web app, you built a community resource. Any website on the internet can make authenticated requests to your API. That's not an access policy, that's a yard sale."
- **Fix:** Explicitly allowlist trusted origins. Never use `*` with `credentials: true`. Configure CORS per-environment (strict in production, relaxed in development). Use a CORS middleware that validates the `Origin` header against an allowlist.

### Missing Security Headers

- **Severity:** info
- **Detect:** Check server configuration and middleware setup for absence of: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`. Grep for `helmet` (Node.js) or equivalent security header middleware. If no security header middleware is found, flag it.
- **Deduction:** -1 point
- **Roast:** "No security headers at all? Your responses are going out into the world completely naked. Not even an X-Frame-Options to keep clickjackers away. Use Helmet.js or equivalent -- it's literally one line of code. One. Line."
- **Fix:** Use `helmet` (Express/Node.js), `django-csp` (Django), `secure_headers` (Rails), or configure headers in your reverse proxy (Nginx, Cloudflare). At minimum set: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`.

### .env in Repository

- **Severity:** critical
- **Detect:** Use Glob to check for `.env`, `.env.local`, `.env.production`, `.env.*.local` files tracked in git. Run `git ls-files .env*` via Bash to confirm they are committed. Also grep `.gitignore` to verify `.env` patterns are present. If `.env` files exist AND are not in `.gitignore`, flag immediately.
- **Deduction:** -20 points
- **Roast:** "You committed your .env file to the repo. That file literally has 'environment' in the name because it's supposed to stay IN your environment, not get published to GitHub for the world to enjoy. Every secret in there is now in the git history forever. Hope you like rotating credentials at 2 AM."
- **Fix:** Add `.env*` to `.gitignore` immediately. Remove tracked `.env` files with `git rm --cached .env`. Rotate ALL secrets that were ever committed (they are in git history even after removal). Use `git-secrets` or `pre-commit` hooks to prevent future commits of secret files. Consider using `BFG Repo-Cleaner` to purge history.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Multiply final score by weight 1.5x.
