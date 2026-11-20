---
name: build-app-with-workspace
description: While building a new CodeNOW application from a conversation with the user, create and continuously maintain the workspace component (spec, architecture, decisions, deploy state, session log, memory) as the single source of truth. Works against any CodeNOW instance.
---

# Build an application and its workspace together

## Purpose

When a user asks for a new application, create the code components and, from the first message, the `<workspace>` knowledge repository. The workspace is written **during** the work, not afterwards. It records what the user asked, what was decided and why, what was built, and what is deployed.

Layout, placeholders (`<app>`, `<workspace-template>`, `<portal>`, `<lang>`, ...) and principles are in `1-workspace-reference.md`. Read it first. Write in `<lang>`.

## Rules

1. **Workspace first.** Create it before any code. If the API cannot create it (components from some templates may not be creatable with a personal key on every instance; verify), ask the user to create `<workspace>` from `<workspace-template>` in `<portal>` and keep notes locally until it exists.
2. **Update as you go.** Follow the table below. Never batch the updates for the end; a stale workspace is a defect.
3. **Never write secrets.** Record names and where they are configured (deployment configuration, not git). Do not echo secret values into the session.
4. **Record the user's words.** Keep the brief, the answers and the approval close to verbatim, with date and author.
5. **Do not invent.** If something is undecided, write `OPEN` and ask.
6. **Label authorship.** Mark what the user decided and what the agent decided.
7. **Do not assume this instance.** Do not hard-code account, host, template ids, environment names or URLs; take them from the user or from API responses.
8. **Leave the workspace scaffold alone.** It is not deployed.

## Conversation

1. Capture the brief verbatim.
2. Ask at most 3 to 5 clarifying questions in one batch, covering: stack and persistence, external services, must-have features, what is out of scope, and whether and where to deploy. Offer defaults.
3. Restate the scope as a short spec and obtain approval. An explicit instruction such as "just build it" counts as approval; record the quote.

## What to write and when

| Moment | Update |
|---|---|
| Brief received | Create the workspace repository; `.sessions/<date>.md` gets "Brief" |
| Questions answered | `.sessions/`: the answers; `prod.md` draft: what it is, scope, out of scope, status `DRAFT` |
| Scope approved | `prod.md` status `APPROVED` with approver, date and quote; version 1.0 |
| Each design choice (stack, storage, external service, authentication, business rule) | New `ADR-n` in `decisions.md`: decision, who, reason, trade-off, date |
| Components created | `README.md` component table; `architecture.md` components |
| Before coding a component | `architecture.md`: domain model (fields, types, constraints) and API (methods, paths, errors) |
| A change alters a contract | Update `architecture.md` and the `prod.md` changelog in the same change |
| MCP server added | `architecture.md`: its tools, and where its address and credentials are configured |
| Each build or release | `context/deploy-state.md`: release version, component versions, result |
| Each deploy | `context/deploy-state.md`: environment table, endpoints, what was verified, where secrets are configured |
| Incident or surprise | `deploy-state.md` incident note; `.sessions/` lesson |
| End of each session | `.sessions/<date>.md`; refresh `memory/README.md` |

## File templates

### `prod.md`
```
# <app> — <one-line description>
**Spec version:** x.y · **Status:** DRAFT | APPROVED | LIVE on <environment> (<release>, <date>)
## What it is
## Scope
## Out of scope
## Approval
- <name>, <date> — "<quote>"
## Changelog
- **x.y** (<date>) — <change>. Reason: <why>
```

### `decisions.md`
```
## ADR-n: <title> (<date>)
Decided by: user | agent
<decision, reason, trade-off, when to revisit>
```

### `architecture.md`
Sections: components (stack, port), domain model (field table and rules), API, MCP tools, configuration per environment (names only), environments and URLs, lifecycle.

### `context/deploy-state.md`
Sections: release mapping, environment table (version, state, what was verified), endpoints, secrets (location only), incidents and lessons.

### `.sessions/<date>.md`
Sections: brief, answers, course, decisions (with ADR links), deploy, iterations, local development notes.

## Finishing

Before telling the user the work is done:

1. The versions and endpoints in `deploy-state.md` match what is actually deployed.
2. `prod.md`, `architecture.md` and the code agree, including every API route and every MCP tool.
3. Every ADR has a date, a reason and an author label.
4. A search of the workspace finds no secret values.
5. The workspace is pushed. Report what was recorded, what is `OPEN`, and what was not verified.
