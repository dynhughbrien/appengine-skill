# App Functions (Backend)

App functions are the serverless backend for your Dynatrace app. They run inside Dynatrace's JS runtime (not in the browser) and are automatically exposed as API endpoints.

## When to use app functions

- Accessing third-party APIs from the internet (keeps credentials server-side)
- Processing large datasets outside the browser
- Transforming data into a UI-consumable shape
- Executing operations that require hidden credentials

## Creating a function

Every `.ts` file directly inside the `api/` directory becomes a callable endpoint. The file must export a single default function:

```typescript
// api/hello.ts
export default async function (payload?: Record<string, unknown>) {
  return { message: "Hello from the backend", received: payload };
}
```

The `payload` parameter is the parsed JSON request body from the caller.

## Calling a function from the frontend

Use `@dynatrace-sdk/app-utils`:

```typescript
import { functions } from "@dynatrace-sdk/app-utils";

const result = await functions.call("hello", { key: "value" });
// result is the value returned from api/hello.ts
```

The function name is the filename without extension (`api/hello.ts` → `"hello"`).

## HTTP status codes

| Code | Meaning |
|------|---------|
| 200 | Successful execution |
| 400 | Client request error |
| 404 | App or function not found |
| 500 | Server error |
| 540 | JavaScript/code error in the function |
| 541 | Runtime error (memory exceeded or timeout) |
| 429 | Rate limit — too many concurrent requests |

## Logging

```typescript
export default async function () {
  console.log("Informational log");
  console.warn("Warning");
  console.error("Error detail");
  return "done";
}
```

Logs appear in your terminal during `npx dt-app dev`. When deployed, they are stored in Grail and queryable via DQL:

```dql
fetch logs
| filter app.id == "com.mycompany.myapp"
| sort timestamp desc
```

## Calling Dynatrace platform APIs from a function

Use `fetch` with the relative `/platform/` path — authentication is handled automatically:

```typescript
export default async function () {
  const response = await fetch("/platform/storage/query/v1/query:execute", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ query: "fetch logs | limit 10" }),
  });
  return response.json();
}
```

## Calling external (internet) APIs

External URLs must be explicitly allowed in `app.config.json`:

```json
{
  "allowedOutboundConnections": {
    "hosts": ["api.example.com"]
  }
}
```

Then call normally with `fetch`:

```typescript
export default async function () {
  const res = await fetch("https://api.example.com/data");
  return res.json();
}
```

## Runtime constraints

| Limit | Value |
|-------|-------|
| Execution timeout | 120 seconds |
| Memory | 256 MB |
| Max input/output size | 5 MB |
| Concurrent calls | Rate-limited (429 when exceeded) |

No WebSocket, no TCP/UDP sockets, no file system access. See `references/runtime.md` for the full list.
