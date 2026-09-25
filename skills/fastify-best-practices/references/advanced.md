# Fastify advanced topics

Decorators, logging, production, and TypeScript notes. Prefer host-project conventions when they conflict.

## Decorators

- Use `decorate` / `decorateRequest` / `decorateReply` for shared utilities and request/reply state — avoid globals and hidden module singletons.
- Declare request/reply decorator **defaults** before assigning per-request values (required for safe prototypes).
- Encapsulate decorations with plugins; use `fp` only when outer scopes must see them.
- Prefer TypeScript declaration merging (or project type modules) so `fastify.xxx` / `request.xxx` stay typed.

## Logging

- Prefer the built-in Pino logger over `console.log`.
- Use **`request.log`** (child logger) in handlers/hooks so logs carry request ids.
- Prefer structured fields (`{ err, userId }`) over string interpolation alone.
- Redact secrets (headers, tokens, passwords) via logger `redact` config.

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
