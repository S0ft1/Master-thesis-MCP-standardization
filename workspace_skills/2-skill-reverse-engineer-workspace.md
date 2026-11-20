---
name: reverse-engineer-workspace
description: For an existing CodeNOW application that has no workspace component, read its repositories and the platform's component data, and create the workspace (spec, architecture, decisions, deploy state, memory) from evidence. Existing repositories are strictly read-only. Works against any CodeNOW instance.
---

# Reverse-engineer a workspace for an existing application

## Purpose

Some applications have code components but no `<workspace>` knowledge repository. Produce one from what the repositories and the platform actually show, so that later work (including programming an MCP server) has a source of truth.

Layout, placeholders (`<app>`, `<api-base>`, `<workspace-template>`, `<lang>`, ...) and principles are in `1-workspace-reference.md`. Read it first.

## Critical rules

1. **Existing repositories are read-only.** Read and search them; clone shallowly into a temporary folder. Never edit, commit to, format or fix them, and never run their tests, installers or servers.
2. **Never reveal secret values.** Record only the name of a secret and where it is configured. If a secret is committed to git, report it as a finding and recommend rotation. When reading configuration files, filter values out of the output (for example print only the names) so a secret is never echoed into the session.
3. **Evidence only.** Every statement comes from a file, a commit or a confirmed API response and names its source. Do not invent approvals, authors, dates or reasons.
4. **Label uncertainty.** `UNKNOWN` = cannot be determined from what you can read. `INFERRED` = conclusion drawn from code, with the reasoning. A draft is never `APPROVED`.
5. **Write only to the new workspace.** If unsure whether a file belongs to it, ask.
6. **Do not assume this instance.** Do not hard-code account, host, template ids or environment names; take them from the user or from API responses.

## Inputs to confirm with the user

- `<app>`, and which CodeNOW instance and account it is in (`<api-base>`, `<portal>`).
- Where the API key is kept (do not ask the user to paste it into chat).
- Which template the instance offers for a non-deployed workspace component (`<workspace-template>`).
- `<lang>` for the workspace.
- That the workspace may be created.

If the API returns an authorization error (for example 403) for the application, stop and report it. Do not guess its contents.

## Workflow

### 1. Inventory (read-only)

List the application's components through the API. Use only endpoints you have confirmed work on this instance; if unsure, test with a read-only call first. Authentication is normally an API-key header and a versioned `Accept` header, but verify against the instance's documentation or a working call.

Record per component: name, description, template id and version, repository address, state, created date and creator.

Stop and report if a workspace component already exists.

Classify each component from its template id and contents: backend, frontend, MCP server, other, scaffold-only.

### 2. Read the code

Shallow-clone each component into a temporary folder. Capture:

- README, pipeline and trigger files, and the **names** (never values) of configuration entries.
- Dependencies and the runtime port.
- **Backend:** routes with method, path, parameters and response shape; validation rules; authentication; error format; storage; outgoing calls.
- **Frontend:** screens, backend routes and external services it calls, how it finds the backend.
- **MCP server:** tools, parameters, authentication, which backend routes each tool calls.
- Tests: what they cover and which behaviours they pin (do not run them).
- Git history: dates, version tags and commit messages that explain changes. Do not map a tag to a commit unless the repository shows it.

### 3. Derive the knowledge

- **Domain model:** entities, fields, types and constraints from validation code and tests; business rules (uniqueness, merging, limits, role restrictions).
- **Architecture:** components, how they call each other, internal addresses, ports, shared conventions.
- **Decisions:** only where code or commit messages show a deliberate choice. Mark each `INFERRED` with its evidence; use the commit date; do not state who decided or why unless a commit says so.
- **Deploy state:** only what the API or repository confirms. Environments, deployed versions and public URLs are `UNKNOWN` unless a confirmed source shows them.
- **Gaps:** things a reader might assume exist but the code lacks.
- **Findings:** security and consistency problems (committed secrets, unauthenticated endpoints, duplicated rules that disagree, missing validation).

### 4. Write the workspace files

Create them in a new local folder named `<workspace>`, in `<lang>`:

- `README.md`, `prod.md` (`Status: reconstructed <date>, not approved`; changelog from git history), `architecture.md`, `decisions.md` (all `INFERRED`), `context/deploy-state.md` (confirmed vs `UNKNOWN`, secret locations by name, open questions), `.sessions/<today>.md` (what was scanned, found and left unknown), `memory/README.md`, `findings.md`.

Do not copy the scaffold files (Dockerfile, Helm chart, pipeline file); the platform creates them with the component.

### 5. Show the draft before anything is created

Present the files and the findings to the user and wait for approval. Do not create a component or push anything until they approve.

### 6. Create the repository

Creating components through the API with a personal key may not work for every template or instance (for example a "template not found" error). Verify rather than assume; the portal is the dependable fallback.

1. Ask the user to create `<workspace>` from `<workspace-template>` in `<portal>`, or confirm it exists.
2. Clone it, add the files, commit and push to its default branch.
3. Do not touch any other repository.

### 7. Verify and summarise

- Re-read each statement against its source; every one has a source or an `UNKNOWN`/`INFERRED` label.
- Search the output for secret values (for example search for each secret value you saw in memory of the session) before committing.
- Confirm the existing repositories are unchanged (clean status).
- Delete the temporary clones.
- Summarise what was written, what is unconfirmed, what the user must review, and the findings. State that tests were not run.
