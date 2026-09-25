# Fastify patterns and examples

Code samples and expanded patterns for the Fastify skill. Prefer host-project conventions when they conflict.

## Minimal app

```js
import Fastify from 'fastify'

const fastify = Fastify({ logger: true })

fastify.get('/', async () => ({ hello: 'world' }))

await fastify.listen({ port: 3000 })
```

## Shared infrastructure with `fastify-plugin`

```js
import fp from 'fastify-plugin'
import fastifyMongo from '@fastify/mongodb'

async function dbConnector(fastify, opts) {
  await fastify.register(fastifyMongo, { url: opts.url })
}

export default fp(dbConnector) // expose mongo to outer scope
```

## Prefixed route plugin

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

## Request/response schema

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

## Central error handler

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
