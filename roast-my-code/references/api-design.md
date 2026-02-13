# API Design Checks

## Activation

Conditional: Active only when Express, Hono, Fastify, Koa, NestJS, or similar HTTP framework is detected in dependencies or imports.

## Check Items

### No Input Validation

- **Severity:** critical
- **Detect:** Grep for route handlers (`app.get`, `app.post`, `router.get`, `@Get()`, `@Post()`, etc.) and check if request body/query/params are consumed without validation. Look for `req.body` or `req.query` used directly without prior validation calls. Check dependencies for absence of `zod`, `joi`, `yup`, `class-validator`, `ajv`, `superstruct`, `express-validator`, or `typebox`. Also look for missing `.validate()`, `.parse()`, `.safeParse()`, `@IsString()`, `@IsEmail()` decorators near handler definitions. In Hono, check for missing `zValidator` middleware.
- **Deduction:** -20 points
- **Roast:** "You're trusting raw user input. That's not brave, that's reckless. Anyone with curl can send you `{ "role": "admin" }` and you'd just accept it with open arms. Validation isn't optional -- it's the bare minimum. This is how mass assignment vulnerabilities are born."
- **Fix:** Add request validation using Zod, Joi, or class-validator. Validate all user input (body, query, params, headers) at the boundary. Use schema-based validation that strips unknown fields by default. Consider middleware-level validation (e.g., `express-zod-api`, `@nestjs/class-validator`, `zValidator` for Hono). Define reusable schemas and share them between frontend and backend.

### Inconsistent Response Format

- **Severity:** error
- **Detect:** Grep for `res.json(`, `c.json(`, `reply.send(`, and inspect the top-level keys of response objects across handlers. Flag when endpoints mix different envelope structures like `{ data }`, `{ result }`, `{ items }`, `{ response }`, or return raw arrays alongside wrapped objects. Check if some endpoints wrap responses in an envelope while others return bare objects. Also look for pagination inconsistency -- some endpoints returning `{ items, total }` while others return `{ data, count, hasMore }`.
- **Deduction:** -10 points
- **Roast:** "Every endpoint returns a different shape. Your API is a box of chocolates -- you never know what you're gonna get. Frontend devs are writing a unique type for every single endpoint because you can't commit to a response structure."
- **Fix:** Define a standard response envelope and use it everywhere: `{ data, meta, error }` or similar. Create a response helper function that enforces the structure. Standardize pagination format across all list endpoints. Document the contract in your API spec. Wrap all responses through a single serialization layer or interceptor.

### No Error Response Standard

- **Severity:** error
- **Detect:** Grep for error responses across handlers: `res.status(4`, `res.status(5`, `reply.code(4`, `.catch`, `throw new Http`. Check if error payloads differ in shape -- some returning `{ message }`, others `{ error }`, `{ errors }`, `{ detail }`, or raw strings. Look for `res.send("error")` returning plain text errors mixed with JSON error responses. Flag absence of a centralized error handler middleware or error response factory function.
- **Deduction:** -10 points
- **Roast:** "Your 400 errors return `{ message }`, your 404s return `{ error }`, and your 500s return a raw string. Clients parsing your errors need a Rosetta Stone. Pick a format -- RFC 7807 exists for exactly this reason."
- **Fix:** Adopt RFC 7807 Problem Details format or define a consistent error schema: `{ status, code, message, details }`. Implement a centralized error handler middleware that catches all errors and serializes them uniformly. Create custom error classes that map to HTTP status codes. Include a machine-readable error code alongside the human-readable message.

### Missing Status Codes

- **Severity:** warning
- **Detect:** Grep for `res.json(` or `res.send(` without a preceding `res.status()` call, or routes that always return `200` regardless of outcome. Check for POST handlers creating resources without `201`, DELETE handlers without `204`, and error paths that return `200` with an error payload embedded in the body. Also flag `res.status(200).json({ error: ... })` patterns where an error is disguised as success.
- **Deduction:** -4 points
- **Roast:** "Returning 200 for errors? 'The operation was a success, but the patient died.' HTTP has over 40 status codes and you've decided to use exactly one. Your monitoring tools are crying in a corner because every request looks 'successful.'"
- **Fix:** Use semantically correct status codes: `201` for resource creation, `204` for successful deletion with no body, `400` for validation errors, `401` for authentication failures, `403` for authorization failures, `404` for missing resources, `409` for conflicts, `422` for unprocessable entities, `429` for rate limiting. Consider a response helper that enforces correct status codes per operation type.

### No Rate Limiting

- **Severity:** warning
- **Detect:** Grep for rate limiter middleware in dependencies and middleware setup: `express-rate-limit`, `@nestjs/throttler`, `rate-limiter-flexible`, `hono/rate-limit`, `fastify-rate-limit`, `koa-ratelimit`. Also check for custom rate limiting implementations via Redis (`INCR`, `EXPIRE` patterns), sliding window counters, or API gateway configuration (nginx `limit_req`, AWS API Gateway throttling). If none found, flag it.
- **Deduction:** -4 points
- **Roast:** "No rate limiting? One angry script kiddie with a while loop can bring your entire API to its knees. You're basically running an all-you-can-eat buffet with no bouncer and wondering why the food keeps running out."
- **Fix:** Add rate limiting middleware (`express-rate-limit`, `@nestjs/throttler`). Configure different limits for different endpoints -- auth endpoints should be stricter (5-10 req/min), general endpoints more relaxed (100-1000 req/min). Use Redis-backed rate limiting for distributed deployments. Return `429 Too Many Requests` with a `Retry-After` header so clients know when to retry.

### Versioning Absent

- **Severity:** warning
- **Detect:** Grep route definitions for versioned paths (`/v1/`, `/v2/`, `/api/v`). Check for header-based versioning (`Accept-Version`, `API-Version`, custom version headers) in middleware or request handling. Inspect NestJS `enableVersioning()`, Fastify versioning plugin, or similar framework-level versioning config. Check for content-type based versioning (`application/vnd.api+json;version=1`). If no versioning strategy is found at all, flag it.
- **Deduction:** -4 points
- **Roast:** "No API versioning? Bold of you to assume you'll get it right the first time. One breaking change and every client integration explodes simultaneously. That's not a deployment, that's a coordinated service outage."
- **Fix:** Add URL-based (`/v1/users`) or header-based (`Accept-Version: 1`) versioning from the start. Use framework support where available (`@nestjs/common` `VersioningType`, Fastify versioning plugin). Document your versioning and deprecation policy. Communicate sunset timelines clearly. Never introduce breaking changes without a new version.

### Overfetching

- **Severity:** warning
- **Detect:** Grep for database queries in route handlers that select all columns: `SELECT *`, `.find()` or `.findMany()` without a `select` clause, `.lean()` returning full documents. Check if handlers return the raw database model directly (e.g., `res.json(user)` where `user` is a full ORM entity potentially containing password hashes, internal IDs, soft-delete flags, internal timestamps, and audit metadata). Also flag endpoints that return nested relations the client didn't request.
- **Deduction:** -4 points
- **Roast:** "Your /users endpoint returns 47 fields when the client needs 3. You're not building an API, you're building a full database dump service. Bandwidth isn't free, and neither is your users' patience waiting for all that unnecessary data to download. Oh, and you're leaking internal fields too."
- **Fix:** Use DTOs or response serializers to shape output. Select only needed fields in database queries. Implement field selection via query params (`?fields=id,name,email`) for flexible clients. Consider GraphQL if overfetching is a persistent architectural problem. Never expose raw database models directly -- always transform through a presentation layer.

### No Request Logging

- **Severity:** info
- **Detect:** Grep for logging middleware in dependencies and app setup: `morgan`, `pino-http`, `express-winston`, `nestjs-pino`, `@koa/logger`, `hono/logger`. Also look for custom middleware that logs `req.method`, `req.url`, `req.path`, or response status codes. Check for structured logging setup that captures request ID, method, path, status code, and response time. If no request-level logging middleware is found, flag it.
- **Deduction:** -1 point
- **Roast:** "No request logging? When something breaks in production -- and it will -- you'll be debugging by staring at the ceiling and hoping for divine inspiration. 'What requests came in before the crash?' Nobody knows. Add a logging middleware. Future-you will thank present-you."
- **Fix:** Add request logging middleware (`morgan` for simple setups, `pino-http` for structured high-performance logging). Log method, path, status code, response time, and a unique request ID. Use correlation IDs for tracing requests across services. Configure log levels per environment (verbose in dev, structured JSON in production).

### Inconsistent Naming

- **Severity:** warning
- **Detect:** Grep for route path definitions and check for mixed conventions: verbs in paths (`/getUsers`, `/createOrder`, `/deleteItem`), mixed casing (`/userProfile` vs `/user-list`), singular/plural inconsistency (`/user` vs `/orders`), or non-RESTful patterns (`/users/get`, `/api/fetch-data`). Flag when more than one naming style is used across routes.
- **Deduction:** -4 points
- **Roast:** "Your routes mix camelCase, kebab-case, and verbs. Pick a convention. Any convention. You've got `/getUsers`, `/user-list`, and `/fetchAllOrders` in the same app. REST is crying. Your API looks like it was designed by a committee that never met."
- **Fix:** Use plural nouns for resource collections (`/users`, `/orders`). Use kebab-case for multi-word paths (`/order-items`). Let HTTP methods express the action -- no verbs in paths. Be consistent with singular vs. plural. Document your naming convention and enforce it with a linter or route registration helper.

### Missing CORS Configuration

- **Severity:** warning
- **Detect:** Grep for CORS middleware or headers: `cors()`, `@nestjs/platform-express` CORS config, `Access-Control-Allow-Origin`, `enableCors()`, `hono/cors`. Check if public-facing API routes lack any CORS configuration. Only flag for APIs intended to be consumed by browser clients (presence of frontend build, SPA references, or explicit public API documentation).
- **Deduction:** -4 points
- **Roast:** "No CORS configuration on a public API? Browser clients are getting rejected at the door while you're wondering why the frontend can't fetch anything. The preflight request isn't a suggestion -- it's a requirement."
- **Fix:** Add CORS middleware with explicit origin allowlist. Configure allowed methods, headers, and credentials per environment. For public APIs, document the CORS policy. Use framework-native CORS support (`app.enableCors()` in NestJS, `cors()` middleware in Express).

### No OpenAPI/Swagger Spec

- **Severity:** info
- **Detect:** Grep for OpenAPI or Swagger tooling in dependencies and config: `swagger-ui-express`, `@nestjs/swagger`, `fastify-swagger`, `@hono/swagger-ui`, `express-openapi-validator`, `openapi`, `swagger.json`, `swagger.yaml`, `openapi.yaml`, `openapi.json`. Check for JSDoc-style API annotations (`@openapi`, `@swagger`) and code-first spec generators (`zod-to-openapi`, `ts-rest`). If none found, flag it.
- **Deduction:** -1 point
- **Roast:** "No API documentation? Your consumers are reverse-engineering your endpoints by reading your source code -- assuming they even have access. An undocumented API is a useless API. Swagger UI takes 5 minutes to set up. Five. Minutes."
- **Fix:** Add OpenAPI/Swagger spec generation using `@nestjs/swagger`, `swagger-jsdoc`, `fastify-swagger`, or `zod-to-openapi`. Annotate routes with request/response schemas. Serve Swagger UI at `/docs` or `/api-docs`. Keep the spec in sync with implementation using code-first generation rather than maintaining a separate YAML file manually.

## Scoring

Start at 100. Apply deductions per finding. Minimum 0. Multiply final score by weight 1.0x.
