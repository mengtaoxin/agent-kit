---
name: fastify-best-practices
description: >-
  Apply Fastify best practices when writing or reviewing Fastify apps.
  Covers plugins, encapsulation, routes, JSON Schema validation/serialization,
  hooks, decorators, errors, logging, testing, and production recommendations.
  Use when editing Fastify servers, registering plugins, defining routes,
  schemas, or hooks, or when the user mentions Fastify, fastify-plugin,
  @fastify/*, or Fastify best practices.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
  docs: https://fastify.dev/
---

# Fastify best practices

Apply these practices when writing or changing Fastify code. Prefer project conventions when they conflict; discover them first (`package.json`, existing plugins, `fastify-cli` layout, TypeScript types).

Official docs: [fastify.dev](https://fastify.dev/).

## Defaults

1. Treat **everything as a plugin** — routes, connectors, and shared utilities register via `fastify.register`.
2. Rely on Fastify’s **async bootstrap**: register plugins in dependency order; loading runs on `listen` / `ready` / `inject`.
3. Prefer **JSON Schema** for request validation and response serialization on public routes.
4. Prefer **native hooks and plugins** over Express-style middleware on hot paths.
5. Enable **`logger: true`** (or a structured Pino config) in non-trivial apps; use `request.log` in handlers.

```js
import Fastify from 'fastify'

const fastify = Fastify({ logger: true })

fastify.get('/', async () => ({ hello: 'world' }))

await fastify.listen({ port: 3000 })
```

## Plugins & encapsulation

- Use `register` as the only way to add routes, plugins, hooks, and related setup.
- Keep **route plugins encapsulated** (default). Shared infrastructure (DB, auth, config) should break encapsulation with **`fastify-plugin` (`fp`)** so decorators reach parent/sibling contexts.
- Do **not** wrap route modules in `fp` unless you intentionally want their hooks/decorators to leak upward.
- Recommended load order in each scope:

```text
ecosystem plugins → your plugins → decorators → hooks → services/routes
```

- Mirror that order inside nested service plugins (`prefix` / bounded contexts).
- Prefer official `@fastify/*` packages when they fit (e.g. `@fastify/sensible`, `@fastify/cors`, `@fastify/helmet`, `@fastify/rate-limit`, `@fastify/under-pressure`).

```js
import fp from 'fastify-plugin'
import fastifyMongo from '@fastify/mongodb'

async function dbConnector(fastify, opts) {
  await fastify.register(fastifyMongo, { url: opts.url })
}

export default fp(dbConnector) // expose mongo to outer scope
```

## Routes

- Prefer route shorthand (`get`/`post`/…) or `route()` with a clear `schema` and `handler`.
- Prefer **async handlers** that `return` a value (or `reply.send`) — avoid mixing callback style unless required.
- Prefer **static or simple parametric** paths on hot routes. Avoid RegExp routes and heavy multi-param patterns when possible.
- Use **route constraints** (e.g. version) sparingly; async custom constraints are a last resort.
- Group related routes in a plugin with `{ prefix: '/api/v1' }` instead of repeating prefixes.
- For Docker/K8s, listen on `0.0.0.0` (or configure probes to the actual bind address). Default is `127.0.0.1`.

```js
async function userRoutes(fastify) {
  fastify.get('/:id', {
    schema: {
      params: {
        type: 'object',
        required: ['id'],
        properties: { id: { type: 'string' } },
      },
    },
  }, async (request) => {
    return fastify.users.findById(request.params.id)
  })
}

fastify.register(userRoutes, { prefix: '/users' })
```

## Validation & serialization

- Validate untrusted input with route `schema` for `body`, `querystring`/`query`, `params`, and `headers`.
- Define **`schema.response`** for success (and important error) status codes — faster serialization and less accidental field leakage.
- Keep Ajv **`allErrors` disabled** by default; enable only when clients need full validation feedback (e.g. forms), never casually on latency-sensitive public endpoints.
- Share schemas via `$id` / `$ref` or a shared schema module when the same shapes repeat.
- After validation, treat `request.body` / `params` / `query` as shaped data; do not re-parse ad hoc.

```js
const schema = {
  body: {
    type: 'object',
    required: ['email'],
    properties: {
      email: { type: 'string', format: 'email' },
    },
  },
  response: {
    201: {
      type: 'object',
      properties: {
        id: { type: 'string' },
        email: { type: 'string' },
      },
    },
  },
}

fastify.post('/users', { schema }, async (request, reply) => {
  const user = await createUser(request.body)
  return reply.code(201).send(user)
})
```

## Hooks & lifecycle

- Prefer hooks for cross-cutting concerns (auth, tracing, tenancy) scoped to the plugin that owns those routes.
- Remember lifecycle order: `onRequest` → `preParsing` → `preValidation` → `preHandler` → handler → `preSerialization` → `onSend` → `onResponse`.
- **`request.body` is not available in `onRequest`** — use `preValidation` / `preHandler` when you need the body.
- Always `await` or call `done` correctly; do not mix async functions with `done` callbacks.
- Use `onClose` for graceful shutdown of connections opened in plugins.
- Prefer Fastify hooks over generic middleware; use `@fastify/middie` / `fastify-express` only when integrating legacy middleware.

## Decorators

- Use `decorate` / `decorateRequest` / `decorateReply` for shared utilities and request/reply state — avoid globals and hidden module singletons.
- Declare request/reply decorator **defaults** before assigning per-request values (required for safe prototypes).
- Encapsulate decorations with plugins; use `fp` only when outer scopes must see them.
- Prefer TypeScript declaration merging (or project type modules) so `fastify.xxx` / `request.xxx` stay typed.

## Errors & replies

- Prefer throwing `Error` (or `@fastify/sensible` / `http-errors` helpers) and a centralized **`setErrorHandler`** over ad-hoc try/catch in every route.
- Error handlers are **encapsulated** — set them where the routes live, or on the root instance for a global default.
- Use **`setNotFoundHandler`** for 404s — `setErrorHandler` does not catch not-found.
- Do not rely on `setErrorHandler` for failures in `onResponse` (response already sent); use `onSend` / logging instead.
- Map validation failures explicitly when the API contract needs a stable shape (`error.validation`).

```js
fastify.setErrorHandler((error, request, reply) => {
  request.log.error({ err: error }, 'request failed')
  if (error.validation) {
    return reply.status(400).send({ error: 'validation_failed', details: error.validation })
  }
  const statusCode = error.statusCode ?? 500
  return reply.status(statusCode).send({
    error: statusCode >= 500 ? 'internal_error' : error.message,
  })
})
```

## Logging

- Prefer the built-in Pino logger over `console.log`.
- Use **`request.log`** (child logger) in handlers/hooks so logs carry request ids.
- Prefer structured fields (`{ err, userId }`) over string interpolation alone.
- Redact secrets (headers, tokens, passwords) via logger `redact` config.

## Testing

- Prefer **`app.inject()`** (light-my-request) — no real TCP listen required.
- Build the app through the same plugin tree as production (`build()` helper); call `ready()` as needed.
- Close the app after tests (`after` / `afterAll`) to avoid open handles.
- Test validation failures, auth hooks, and error-handler mapping — not only happy paths.

```js
import { build } from './app.js'

const app = await build()
const res = await app.inject({ method: 'GET', url: '/health' })
// assert res.statusCode / res.json()
await app.close()
```

## Performance & production

- Put a **reverse proxy** (Nginx, HAProxy, cloud LB) in front for TLS, multi-domain, static assets, and horizontal scale — do not terminate public TLS / multi-vhost concerns inside the Node process by default.
- Define **response schemas** on hot JSON endpoints.
- Prefer plugins/hooks over middleware adapters when latency matters.
- Behind a proxy, set **`trustProxy`** appropriately so protocol/IP headers are trusted only from known hops.
- Run multiple Fastify instances in one process only for deliberate isolation (e.g. private metrics port); otherwise scale with multiple processes/pods.
- Capacity rule of thumb from Fastify recommendations: ~2 vCPU/instance for lowest latency (GC/libuv); fewer vCPUs can favor throughput — measure with autocannon/k6.

## TypeScript

- Prefer official Fastify TypeScript support: generic `FastifyInstance`, route generics for `Params`/`Querystring`/`Body`/`Reply`.
- Type shared decorators via module augmentation so plugins stay type-safe across encapsulation.
- Prefer `async` plugins (`async function (fastify, opts)`) for clearer control flow with `await register`.

## Anti-patterns

- Registering everything on the root instance with no encapsulation (no plugin boundaries).
- Wrapping **all** plugins in `fastify-plugin` (defeats encapsulation).
- Skipping response schemas on APIs that return DB documents (easy to leak fields).
- Using RegExp routes or `allErrors: true` on untrusted, high-traffic inputs without need.
- Binding only to `127.0.0.1` in containers while K8s probes hit the pod IP.
- Exposing the Node process directly to the internet for TLS and multi-tenant virtual hosting.
