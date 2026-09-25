---
name: fastify-best-practices
description: >-
  Use when writing or reviewing Fastify apps: plugins, encapsulation, routes,
  JSON Schema validation/serialization, hooks, decorators, errors, logging,
  and production setup. Triggers on Fastify servers, plugins, routes, schemas,
  hooks, fastify-plugin, or @fastify/*. Do not use for Express, Koa, or Hono.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.2.0"
  docs: https://fastify.dev/
---

# Fastify best practices

Apply these practices when writing or changing Fastify code. Prefer project conventions when they conflict; discover them first (`package.json`, existing plugins, `fastify-cli` layout, TypeScript types).

Official docs: [fastify.dev](https://fastify.dev/).

Deeper detail: [examples & patterns](./references/patterns.md), [decorators, logging, production, TypeScript](./references/advanced.md).

## Defaults

1. Treat **everything as a plugin** — routes, connectors, and shared utilities register via `fastify.register`.
2. Rely on Fastify’s **async bootstrap**: register plugins in dependency order; loading runs on `listen` / `ready` / `inject`.
3. Prefer **JSON Schema** for request validation and response serialization on public routes.
4. Prefer **native hooks and plugins** over Express-style middleware on hot paths.
5. Enable **`logger: true`** (or a structured Pino config) in non-trivial apps; use `request.log` in handlers.

## Plugins & encapsulation

- Use `register` as the only way to add routes, plugins, hooks, and related setup.
- Keep **route plugins encapsulated** (default). Shared infrastructure (DB, auth, config) should break encapsulation with **`fastify-plugin` (`fp`)** so decorators reach parent/sibling contexts.
- Do **not** wrap route modules in `fp` unless you intentionally want their hooks/decorators to leak upward.
- Recommended load order: `ecosystem plugins → your plugins → decorators → hooks → services/routes` (mirror inside nested service plugins).
- Prefer official `@fastify/*` packages when they fit (e.g. `@fastify/sensible`, `@fastify/cors`, `@fastify/helmet`, `@fastify/rate-limit`, `@fastify/under-pressure`).

## Routes

- Prefer route shorthand (`get`/`post`/…) or `route()` with a clear `schema` and `handler`.
- Prefer **async handlers** that `return` a value (or `reply.send`) — avoid mixing callback style unless required.
- Prefer **static or simple parametric** paths on hot routes. Avoid RegExp routes and heavy multi-param patterns when possible.
- Group related routes in a plugin with `{ prefix: '/api/v1' }` instead of repeating prefixes.
- For Docker/K8s, listen on `0.0.0.0` (or configure probes to the actual bind address). Default is `127.0.0.1`.

## Validation & serialization

- Validate untrusted input with route `schema` for `body`, `querystring`/`query`, `params`, and `headers`.
- Define **`schema.response`** for success (and important error) status codes — faster serialization and less accidental field leakage.
- Keep Ajv **`allErrors` disabled** by default; enable only when clients need full validation feedback, never casually on latency-sensitive public endpoints.
- Share schemas via `$id` / `$ref` or a shared schema module when the same shapes repeat.
- After validation, treat `request.body` / `params` / `query` as shaped data; do not re-parse ad hoc.

## Hooks & lifecycle

- Prefer hooks for cross-cutting concerns (auth, tracing, tenancy) scoped to the plugin that owns those routes.
- Lifecycle: `onRequest` → `preParsing` → `preValidation` → `preHandler` → handler → `preSerialization` → `onSend` → `onResponse`.
- **`request.body` is not available in `onRequest`** — use `preValidation` / `preHandler` when you need the body.
- Always `await` or call `done` correctly; do not mix async functions with `done` callbacks.
- Use `onClose` for graceful shutdown of connections opened in plugins.
- Prefer Fastify hooks over generic middleware; use `@fastify/middie` / `fastify-express` only when integrating legacy middleware.

## Errors & replies

- Prefer throwing `Error` (or `@fastify/sensible` / `http-errors` helpers) and a centralized **`setErrorHandler`** over ad-hoc try/catch in every route.
- Error handlers are **encapsulated** — set them where the routes live, or on the root instance for a global default.
- Use **`setNotFoundHandler`** for 404s — `setErrorHandler` does not catch not-found.
- Do not rely on `setErrorHandler` for failures in `onResponse` (response already sent); use `onSend` / logging instead.
- Map validation failures explicitly when the API contract needs a stable shape (`error.validation`).

## Anti-patterns

- Registering everything on the root instance with no encapsulation (no plugin boundaries).
- Wrapping **all** plugins in `fastify-plugin` (defeats encapsulation).
- Skipping response schemas on APIs that return DB documents (easy to leak fields).
- Using RegExp routes or `allErrors: true` on untrusted, high-traffic inputs without need.
- Binding only to `127.0.0.1` in containers while K8s probes hit the pod IP.
- Exposing the Node process directly to the internet for TLS and multi-tenant virtual hosting.

## Agent checklist

Before finishing Fastify work:

1. Routes/plugins use `register`; encapsulation/`fp` usage is intentional.
2. Public routes have request validation and response schemas where appropriate.
3. Hooks are scoped correctly; no async/`done` mixing; body read only after parsing.
4. Errors go through `setErrorHandler` / `setNotFoundHandler` as needed.
5. Logging uses `request.log` with secrets redacted when configured.
6. Listen bind address and `trustProxy` match deployment (local vs container/proxy).
