# Data Storage Options

Dynatrace AppEngine provides several storage backends. Choose based on data type, lifetime, and access pattern.

## Decision guide

| Need | Use |
|------|-----|
| Query logs, metrics, events, traces | Grail (DQL) |
| Persist UI state across sessions | App state (`client-state`) |
| Store documents users create | Document store (`client-document`) |
| App configuration (typed schema) | App settings v2 (`client-app-settings-v2`) |
| General Dynatrace platform settings | Settings SDK (`client-settings`) |
| Store and sync external credentials/API keys | Credential vault |

## Grail (Observability Data)

Grail is the unified storage for logs, metrics, traces, and events. Query it from an app with the DQL query API or ingest custom data via the events ingest API.

```typescript
import { queryExecutionClient } from "@dynatrace-sdk/client-query";

const result = await queryExecutionClient.queryExecute({
  body: {
    query: `fetch logs
| filter loglevel == "ERROR"
| limit 20`,
    requestTimeoutMilliseconds: 10000,
  },
});
const records = result.result?.records ?? [];
```

Required scope: `storage:logs:read` (or the appropriate data-type scope).

## App State

Key-value store for persisting small chunks of state (< a few KB). Supports per-user state and shared app state.

```typescript
import { stateClient } from "@dynatrace-sdk/client-state";

// Write
await stateClient.createOrUpdateAppState({
  key: "lastSelectedHost",
  body: { value: JSON.stringify({ hostId: "HOST-123" }) },
});

// Read
const state = await stateClient.getAppState({ key: "lastSelectedHost" });
const value = JSON.parse(state.value ?? "null");
```

For user-specific state, use `createOrUpdateUserAppState` / `getUserAppState` (same API shape).

## Document Store

Stores structured JSON documents created by users. Ideal for saved queries, dashboards configs, templates.

```typescript
import { documentsClient } from "@dynatrace-sdk/client-document";

// Create
const doc = await documentsClient.createDocument({
  body: {
    name: "My saved query",
    type: "custom-query",
    content: JSON.stringify({ dql: "fetch logs | limit 5" }),
    isPrivate: false,
  },
});

// List
const list = await documentsClient.listDocuments({ type: "custom-query" });
```

Required scope: `document:documents:write`, `document:documents:read`.

## App Settings (v2)

Typed, schema-validated configuration settings defined by the app. Use for structured app configuration that admins manage.

```typescript
import { appSettingsClient } from "@dynatrace-sdk/client-app-settings-v2";

// Read all settings for this app's schema
const settings = await appSettingsClient.getEffectiveSettings({
  schemaId: "app:com.mycompany.myapp:config",
});
```

Define the schema inside `app.config.json` under `"settings"`.

Required scopes: `app-settings:objects:read`, `app-settings:objects:write`.

## Dynatrace Settings (General)

Access and manage any Dynatrace settings object (OneAgent config, alerting, etc.):

```typescript
import { settingsObjectsClient } from "@dynatrace-sdk/client-settings";

const objects = await settingsObjectsClient.getSettingsObjects({
  schemaIds: "builtin:anomaly-detection.metric-events",
});
```

Required scope: `settings:objects:read`.

## Credential Vault

Use credential vault when you need to sync secrets with an external vault system. For most cases, store secrets via App Settings instead — it is simpler and does not require external infrastructure.

Credential vault integration is configured under `"credentials"` in `app.config.json` and accessed via the platform credential API inside app functions (never expose credentials to the frontend).
