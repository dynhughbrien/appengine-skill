# Dynatrace AppEngine Skill

A [Claude Code](https://claude.ai/code) agent skill for creating and developing [Dynatrace AppEngine](https://developer.dynatrace.com/develop) applications.

## Install

```bash
gh skill install dynhughbrien/appengine-skill dt-appengine
```

## What it covers

- **Project setup** — scaffold a new app with `npx dt-app@latest create`, directory layout, `app.config.json` fields
- **App functions** — write TypeScript backend functions in `/api`, call them from the frontend, logging, HTTP status codes
- **SDK packages** — full `@dynatrace-sdk/*` reference (DQL queries, state, settings, automation, navigation, and more)
- **Data storage** — Grail, app state, document store, settings SDK, credential vault — when to use each
- **Security** — IAM permission scopes, Content Security Policy, secrets management, user permissions
- **Deployment** — local deploy, GitHub Actions / GitLab / Jenkins CI/CD, OAuth credentials
- **JS runtime** — execution limits (120s, 256 MB), supported Web APIs, Node.js compat modules, what's blocked

## Usage

Once installed, invoke in Claude Code:

```
/dt-appengine
```

Or just describe what you want to build — Claude will load the skill automatically when the task involves creating or modifying a Dynatrace app.

## Reference

- [Dynatrace Developer Portal](https://developer.dynatrace.com/develop)
- [App Toolkit CLI (`dt-app`)](https://developer.dynatrace.com/quickstart/first-app-in-5-minutes/)
- [SDK for TypeScript](https://developer.dynatrace.com/develop/sdks/)
- [Strato UI Components](https://developer.dynatrace.com/develop/ui-components/)
