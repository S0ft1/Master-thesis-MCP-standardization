# Application workspace: reference layout

A **workspace** is a knowledge repository that belongs to one CodeNOW application. It is the application's single source of truth: what it should do, how it is built, why it was built that way, what is deployed and how the work went. It holds knowledge, not running code.

This layout is a pattern observed in practice, not a CodeNOW requirement. Adapt it if the user or the organisation already uses a different one.

## Placeholders used in these skills

| Placeholder | Meaning |
|---|---|
| `<app>` | Application id or name |
| `<workspace>` | The workspace component, by convention `workspace-<app>` |
| `<api-base>` | CodeNOW API base URL of the instance in use |
| `<portal>` | CodeNOW portal URL of the instance in use |
| `<account>` | CodeNOW account the application belongs to |
| `<api-key>` | API key for the instance; read it from where the user keeps it, never print it |
| `<workspace-template>` | Template used for a component that is **not deployed** (often a generic Docker template); ask the user which one the instance offers |
| `<lang>` | Language the workspace is written in: the language of the existing workspaces or of the user |

## What the workspace component is

- A normal application component, usually created from a generic non-deployed template, named `<workspace>`.
- It is never deployed. Its build and deploy scaffold (Dockerfile, Helm chart, web server config, pipeline file) exists only so the repository exists on the platform. Say this in its README and leave the scaffold alone.

## Content

| Path | Purpose | Contents |
|---|---|---|
| `README.md` | Index | Summary, folder map, table of code components with repository addresses, "not deployed" note |
| `prod.md` | Versioned product spec | Spec version and status, what the application is, in-scope features, out-of-scope items, approval (who, when, quote), changelog per released version with the reason for each change |
| `architecture.md` | Technical design | Components and stack, domain model (fields, types, rules), API, MCP tools if any, configuration per environment (names only), environments and URLs, deployment lifecycle |
| `decisions.md` | Decision records | Numbered `ADR-n: title (date)` entries: decision, who, reason, trade-off, when to revisit |
| `context/deploy-state.md` | Operational state | Release-to-component version mapping, environment table (version, state, what was verified), endpoints, where secrets live, incidents and lessons |
| `.sessions/<date>.md` | Work log | Per session: brief, answers to questions, course of work, decisions, deploy problems, iterations, local development notes |
| `memory/README.md` | Shared project memory | Knowledge that does not fit elsewhere, plus open questions |
| `findings.md` | Optional, retroactive workspaces | Problems found while reading existing code (security, inconsistencies), without secret values |

Names of files and sections may be translated to `<lang>`. Keep the same roles.

## How it differs from code components

Code components hold source, tests and platform configuration (pipeline file, trigger file, environment configuration, quality-analysis config). Their READMEs are often only a title. The reasons, requirements and history live in the workspace.

## Principles

1. **Single source of truth.** Anything about the application that is not obvious from code lives here.
2. **Written during the work** for new applications, and **reconstructed with clear evidence labels** for existing ones.
3. **Versioned and tied to releases.** The spec's changelog and the deploy state refer to released versions.
4. **No secrets.** Record the name of a secret and where it is configured, never its value.
5. **Honest about gaps.** Unknown and open items are written down as such.
