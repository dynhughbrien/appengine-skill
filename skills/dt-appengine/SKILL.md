---
name: dt-appengine
description: Create and develop Dynatrace AppEngine applications — project setup, app.config.json, UI with Strato components, backend app functions, SDK packages, data storage, security, deployment, and CI/CD. Use when building a new Dynatrace app, adding features, scaffolding functions, or deploying to an environment.
license: Apache-2.0
---

# Dynatrace AppEngine Development

Dynatrace AppEngine lets you build and deploy custom apps directly inside the Dynatrace platform. Apps are React + TypeScript frontends optionally backed by serverless TypeScript functions.

## Quick reference

| File | When to load |
|------|-------------|
| [references/project-setup.md](references/project-setup.md) | Scaffolding a new app, `app.config.json` fields, directory layout, running locally |
| [references/app-functions.md](references/app-functions.md) | Writing backend functions in `/api`, function contract, HTTP status codes, logging |
| [references/sdk-packages.md](references/sdk-packages.md) | Full list of `@dynatrace-sdk/*` packages — which one to use for DQL, state, settings, automation, etc. |
| [references/data-storage.md](references/data-storage.md) | Grail, app state, document store, settings, credential vault — when to use each |
| [references/security.md](references/security.md) | Permission scopes, CSP, secrets management, user permissions |
| [references/deployment.md](references/deployment.md) | `npx dt-app deploy`, CI/CD for GitHub Actions / GitLab / Jenkins, OAuth credentials |
| [references/runtime.md](references/runtime.md) | JS runtime limits (timeout, memory, I/O), supported Web APIs, Node.js compat modules |

## Decision tree

**Starting a new app?** → Load `references/project-setup.md`

**Need to call an external API or run heavy processing away from the browser?** → Load `references/app-functions.md`

**Not sure which SDK package does what?** → Load `references/sdk-packages.md`

**Storing data (state, documents, settings, credentials, Grail)?** → Load `references/data-storage.md`

**Deploying, CI/CD, environment variables?** → Load `references/deployment.md`

**Getting a runtime error, timeout, or memory issue?** → Load `references/runtime.md`

**Security, permissions, CSP, secrets?** → Load `references/security.md`

## Core concepts in one paragraph

An AppEngine app is a Node.js/React project created with `npx dt-app@latest create`. The frontend lives in `ui/` and uses Strato components (`@dynatrace/strato-components`). The backend is zero or more TypeScript files in `api/` — each becomes a callable function endpoint. Configuration lives in `app.config.json` (app ID, permissions, environment URL). Run locally with `npx dt-app dev`; deploy with `npx dt-app deploy`. The version field in `app.config.json` **must be incremented** on every re-deploy.
