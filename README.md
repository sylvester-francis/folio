<div align="center">

```
███████╗ ██████╗ ██╗     ██╗ ██████╗
██╔════╝██╔═══██╗██║     ██║██╔═══██╗
█████╗  ██║   ██║██║     ██║██║   ██║
██╔══╝  ██║   ██║██║     ██║██║   ██║
██║     ╚██████╔╝███████╗██║╚██████╔╝
╚═╝      ╚═════╝ ╚══════╝╚═╝ ╚═════╝
```

**Learn the Ory identity stack end to end by building a real NestJS service.**

Authentication with **Kratos**, relationship-based authorization with **Keto** (Google Zanzibar),
and an identity-aware proxy with **Oathkeeper** — wired into one document-sharing API.

A code-verified guide. Every slice typechecks, passes **66 tests** under Bun, and survives a **97.56%** mutation run.

![NestJS](https://img.shields.io/badge/NestJS-11-E0234E?logo=nestjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Bun](https://img.shields.io/badge/Bun-test%20runner-000000?logo=bun&logoColor=white)
![Ory](https://img.shields.io/badge/Ory-Kratos%20·%20Keto%20·%20Oathkeeper-5528FF?logo=ory&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-Apache--2.0-blue.svg)

[Why Folio](#why-folio) · [What you build](#what-you-build) · [The pipeline](#the-request-pipeline) · [Chapters](#the-guide) · [Quick start](#quick-start) · [Stack](#tech-stack)

</div>

---

## Why Folio

Most Ory tutorials show one piece in isolation — a login flow, a single permission check — and leave you to guess how they fit. Folio takes the opposite approach.

You build **one small document-sharing API** and wire it to the *entire* open-source Ory stack the way you would actually deploy it: Kratos answers *who are you*, Keto answers *what may you do*, and Oathkeeper sits in front and applies both before a request ever reaches your code. The result is a request pipeline that authenticates a caller and then decides — **per document** — what that caller may do.

Nothing is left "as an exercise." Every code slice in the prose is taken byte-for-byte from a tree that typechecks, passes its OPL check, passes 66 tests under Bun's runner, and is then mutation-tested with Stryker to a 97.56% score. Checkpoints show real captured output, not idealized output.

## What you build

A tiny HTTP surface backed by the full Ory pipeline:

```
GET   /healthz                public liveness probe
GET   /me                     echo the authenticated identity
POST  /documents              create a document (caller becomes owner)
GET   /documents/:id          read   (requires `view`   on the document)
PATCH /documents/:id          update (requires `edit`   on the document)
DELETE /documents/:id         delete (requires `delete` on the document)
POST  /documents/:id/share    grant another user access (requires `share`)
```

Authorization is **relationship-based**, following Google's Zanzibar model. A document has *owners*, *editors*, and *viewers*; ownership implies delete, which implies edit, which implies view. Folders propagate their grants to the documents inside them. All of it is expressed once, in an Ory Permission Language (OPL) model that the app and Keto share.

By the end you have **two working request modes**: the app validating the Kratos session cookie itself, and the app running behind Oathkeeper trusting an injected `X-User-Id` header — the configuration you reach for when putting Ory in front of a service you are strangling out of a monolith.

## The three pieces

| Component | Question it answers | Role |
|---|---|---|
| **Kratos** | *Who are you?* | AuthN — identities, credentials, sessions, login/registration |
| **Keto** | *What may you do?* | AuthZ — relation tuples, permission checks (Google Zanzibar) |
| **Oathkeeper** | *Let it through?* | Identity & Access Proxy — runs in front, applies the two above |

## The request pipeline

In the default in-app mode, a single authorized request flows through two guards in a fixed order — authentication first, authorization second:

```
Browser
  │  Cookie: ory_kratos_session=MTU...
  ▼
NestJS app
  │  ┌── SessionGuard ──────────────────────────────────────────┐
  │  │  toSession(cookie)  ───────────►  Kratos :4433            │
  │  │      ◄── identity { id: "alice", traits: { email } }      │
  │  │  req.user = { id: "alice", email }                        │
  │  └───────────────────────────────────────────────────────────┘
  │  ┌── PermissionGuard ───────────────────────────────────────┐
  │  │  checkPermission(view, Document:doc-1, alice) ─► Keto:4466│
  │  │      ◄── { allowed: true | false }                        │
  │  │  403 Forbidden if denied                                  │
  │  └───────────────────────────────────────────────────────────┘
  ▼
Controller handler  ──►  DocumentsService  (the row exists? 404 if not)
```

## The guide

Everything lives in **[`folio-ory-nestjs-guide.md`](./folio-ory-nestjs-guide.md)**. Each chapter builds one component and leaves you with something that compiles and passes tests.

| # | Chapter |
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
| — | Appendix: file manifest, offline verification, running live |

## Quick start

**Prerequisites.** You are fluent in TypeScript and comfortable with NestJS' module/provider/guard model, Promises, and decorators, and you can read a `docker-compose.yml`. No prior Ory, Zanzibar, or Keto knowledge is assumed — that is what this is for.

**Toolchain.** Folio uses [mise](https://mise.jdx.dev/) for tool pinning and [Bun](https://bun.sh/) as the package manager *and* test runner; the live Ory stack runs under Docker Compose. The app is a standard NestJS server that builds to CommonJS with `nest build`, so it also runs under plain `node` — only the test runner is Bun-specific.

```bash
# the guide walks you through scaffolding the tree; once you have it:
bun install
bun run test          # scoped to ./src ./test — runs the 66-test suite under Bun
docker compose up      # boots Postgres, Kratos, Keto, Oathkeeper, mailslurper, and the app
```

> **Note.** `bun test` runs Bun's own test runner, and the specs import their primitives from `bun:test`. The end-to-end file is named `documents.e2e.spec.ts` (Bun ignores the dash form Nest scaffolds), and the `test` script is scoped to `bun test ./src ./test` so a stale `dist/` is not re-run.

## A note on verification

The **application** — every line of `src/`, the test suite, and the OPL model — was built and verified where it typechecks, passes 66 tests under Bun, and passes a Stryker mutation run at 97.56%. The **live Ory stack** (Kratos, Keto, Oathkeeper under `docker compose up`) is config-reviewed rather than executed in the build environment; the guide reproduces every file in full so the `docker compose up` and `curl` walkthroughs run on your machine exactly as written. A *verification boundary* note marks every step that depends on the live stack.

## Tech stack

| Component | Technology |
|---|---|
| Language | TypeScript 5 |
| Framework | NestJS 11 |
| AuthN | Ory Kratos (`@ory/client`) |
| AuthZ | Ory Keto (`@ory/keto-client`) — Google Zanzibar / ReBAC |
| Proxy | Ory Oathkeeper (identity & access proxy) |
| Package manager + test runner | Bun (`bun:test`) |
| Mutation testing | Stryker (97.56% score) |
| Tool pinning | mise |
| Live stack | Docker Compose (Postgres + the Ory services) |

## License

Licensed under the **Apache License 2.0**. See [`LICENSE`](./LICENSE).
