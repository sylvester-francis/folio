# Folio — Learning the Ory Stack End to End with NestJS

A complete, code-verified guide to the open-source [Ory](https://www.ory.sh/) identity stack, taught by building **Folio**, a small document-sharing API in NestJS. You wire a real service to all three Ory components and finish with a request pipeline that authenticates a caller and then decides — per document — what that caller may do.

- **Authentication** with **Kratos** — identities, credentials, sessions
- **Authorization** with **Keto** — relationship-based access control (Google Zanzibar)
- An **identity-aware reverse proxy** with **Oathkeeper** — the keystone of a strangler-fig migration

**Stack:** NestJS 11 · TypeScript 5 · `@ory/client` · `@ory/keto-client` · Kratos · Keto · Oathkeeper
**Estimated time:** 5–7 hours · **License:** Apache-2.0

## What you build

Folio exposes a tiny HTTP surface backed by the full Ory pipeline:

```
GET   /healthz                public liveness probe
GET   /me                     echo the authenticated identity
POST  /documents              create a document (caller becomes owner)
GET   /documents/:id          read   (requires `view`   on the document)
PATCH /documents/:id          update (requires `edit`   on the document)
DELETE /documents/:id         delete (requires `delete` on the document)
POST  /documents/:id/share    grant another user access (requires `share`)
```

Authorization is relationship-based: a document has *owners*, *editors*, and *viewers*; ownership implies delete, which implies edit, which implies view. Folders propagate grants to the documents inside them. All of it is expressed once, in an Ory Permission Language (OPL) model that the app and Keto share.

By the end you have two working request modes: the app validating the Kratos session cookie itself, and the app running behind Oathkeeper trusting an injected `X-User-Id` header.

## The guide

Everything lives in [`folio-ory-nestjs-guide.md`](./folio-ory-nestjs-guide.md). Every code slice is taken byte-for-byte from a tree that typechecks, passes its OPL check, and passes 66 tests under Bun's runner; a dedicated chapter mutation-tests that suite with Stryker to a 97.56% score.

| Chapter | Topic |
|---|---|
| 0 | The mental model: AuthN, AuthZ, and Zanzibar |
| 1 | Project setup and the shape of the repo |
| 2 | Kratos: identities, the schema, and booting AuthN |
| 3 | AuthN in the app: the Ory seam, the session guard, and `/me` |
| 4 | Exercising the flows: registration, login, and a real session |
| 5 | Keto: the permission model in OPL |
| 6 | AuthZ in the app: permissions, the decorator, and the guard |
| 7 | The domain: documents that keep Keto in sync |
| 8 | Oathkeeper: the identity-aware proxy and the strangler-fig |
| 9 | The full request lifecycle, and what changes in production |
| 10 | Mutation testing: are the tests any good? |
| 11 | Where to go next |
| Appendix | File manifest, offline verification, and running live |

## Prerequisites

You are fluent in TypeScript and comfortable with NestJS' module/provider/guard model, Promises, and decorators, and you can read a `docker-compose.yml`. No prior Ory, Zanzibar, or Keto knowledge is assumed — that is what this is for.

## Toolchain

Folio is developed with [mise](https://mise.jdx.dev/) for tool pinning and [Bun](https://bun.sh/) as the package manager *and* test runner; the live Ory stack runs under Docker Compose. The app is a standard NestJS server that builds to CommonJS with `nest build`, so it also runs under plain `node` — only the test runner is Bun-specific.

> **Note.** `bun test` runs Bun's own test runner, and the specs import their primitives from `bun:test`. The end-to-end file is named `documents.e2e.spec.ts` (Bun ignores the dash form Nest scaffolds), and the `test` script is scoped to `bun test ./src ./test` so a stale `dist/` is not re-run.

## A note on verification

The **application** — every line of `src/`, the test suite, and the OPL model — was built and verified where it typechecks, passes 66 tests under Bun, and passes a Stryker mutation run at 97.56%. The **live Ory stack** (Kratos, Keto, Oathkeeper under `docker compose up`) is config-reviewed rather than executed in the build environment; the guide reproduces every file in full so the `docker compose up` and `curl` walkthroughs run on your machine exactly as written. A *verification boundary* note marks every step that depends on the live stack.
