# SDK Packages (`@dynatrace-sdk/*`)

All packages are imported from `@dynatrace-sdk/<name>`. They work in both the UI (React) and in app functions (backend).

## Data & Query

| Package | Install name | Use case |
|---------|-------------|---------|
| Grail DQL query API | `@dynatrace-sdk/client-query` | Execute DQL queries against Grail (logs, metrics, events, traces) |
| Grail - resource store | `@dynatrace-sdk/client-resource-store` | Read/write Dynatrace Resource Store entries |
| Grail - storage management | `@dynatrace-sdk/client-bucket-management` | Manage Grail storage buckets |
| Grail - filter segments | `@dynatrace-sdk/client-filter-segment-management` | Manage Grail filter segments |

### DQL query example

```typescript
import { queryExecutionClient } from "@dynatrace-sdk/client-query";

const result = await queryExecutionClient.queryExecute({
  body: {
    query: "fetch logs | limit 5",
    requestTimeoutMilliseconds: 5000,
  },
});
console.log(result.result?.records);
```

## State & Storage

| Package | Use case |
|---------|---------|
| `@dynatrace-sdk/client-state` | Key-value store for persisting small app/user state across sessions |
| `@dynatrace-sdk/client-document` | Create and manage user-generated content documents |
| `@dynatrace-sdk/client-app-settings-v2` | Read and write typed app configuration settings |
| `@dynatrace-sdk/client-settings` | Schema-driven API for any Dynatrace settings object |

## Platform & Infrastructure

| Package | Use case |
|---------|---------|
| `@dynatrace-sdk/client-automation` | Work with AutomationEngine workflows and triggers |
| `@dynatrace-sdk/client-synthetic` | Manage synthetic monitors, locations, and executions |
| `@dynatrace-sdk/client-service-level-objectives` | Manage and evaluate SLOs |
| `@dynatrace-sdk/client-davis-analyzers` | Davis® AI predictive and causal analysis |
| `@dynatrace-sdk/client-iam` | View users and their access/capability configuration |
| `@dynatrace-sdk/client-platform-management-service` | Read-only environment information (name, id, etc.) |
| `@dynatrace-sdk/client-notification` | Manage self-notifications |
| `@dynatrace-sdk/client-notification-v2` | Manage resource/event notifications |
| `@dynatrace-sdk/client-hub` | Access app/extension catalog content |
| `@dynatrace-sdk/client-app-engine-registry` | Manage AppEngine app registrations |
| `@dynatrace-sdk/client-app-engine-edge-connect` | Manage EdgeConnect configurations |

## App Utilities

| Package | Use case |
|---------|---------|
| `@dynatrace-sdk/app-environment` | Get current app ID, environment URL, user info |
| `@dynatrace-sdk/app-utils` | Call app functions from the frontend (`functions.call(...)`) |
| `@dynatrace-sdk/automation-utils` | Helper utilities for AutomationEngine API access |
| `@dynatrace-sdk/addons` | Lifecycle control when running as an addon |
| `@dynatrace-sdk/navigation` | Navigate between apps, send Intents |
| `@dynatrace-sdk/user-preferences` | Get logged-in user's theme, language, and preferences |

## React & UI Utilities

| Package | Use case |
|---------|---------|
| `@dynatrace-sdk/react-hooks` | React hooks for state, DQL, settings — simplifies async data fetching |
| `@dynatrace-sdk/units` | Convert and format metric units and numeric values |

## Classic Environment (legacy)

| Package | Use case |
|---------|---------|
| `@dynatrace-sdk/client-classic-environment-v1` | Dynatrace Environment API v1 (classic) |
| `@dynatrace-sdk/client-classic-environment-v2` | Dynatrace Environment API v2 (classic) |

Prefer the platform services / Grail approach over classic APIs for new apps.

## App environment example

```typescript
import { appEnvironmentClient } from "@dynatrace-sdk/app-environment";

const env = await appEnvironmentClient.getEnvironmentInfo();
console.log(env.environmentUrl, env.appId);
```

## Navigation example

```typescript
import { navigate } from "@dynatrace-sdk/navigation";

// Navigate to another Dynatrace app by ID
navigate({ appId: "dynatrace.dashboards", slotPath: "/" });
```
