# JavaScript Runtime

App functions run in Dynatrace's sandboxed server-side JavaScript runtime, not in Node.js or a browser.

## Hard limits

| Resource | Limit |
|----------|-------|
| Execution timeout | 120 seconds |
| Memory | 256 MB RAM |
| Max request/response payload | 5 MB |
| Concurrent function calls | Rate-limited (429 when exceeded) |

## What IS supported

### Standard JavaScript
- Full ECMAScript (ES2022+) including top-level await
- TypeScript (type annotations, interfaces, generics, enums)
- `Intl` / Internationalization API
- ES module syntax (`import` / `export`) — CommonJS (`require`) is **not** supported

### Web APIs (selected)

| Category | Available APIs |
|----------|---------------|
| Networking | `fetch`, `Request`, `Response`, `Headers`, `URL`, `URLSearchParams` |
| Data | `Blob`, `File`, `FormData`, `TextEncoder`, `TextDecoder`, `ArrayBuffer` |
| Async | `AbortController`, `AbortSignal`, `ReadableStream`, `WritableStream`, `TransformStream` |
| Security | `Crypto`, `SubtleCrypto`, `CryptoKey` |
| Compression | `CompressionStream`, `DecompressionStream` |
| Timing | `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`, `performance` |

> Deprecated properties on these Web APIs may not be available.

### Node.js compatibility modules

These standard Node.js modules are available via import:

`assert`, `buffer`, `crypto`, `events`, `http`, `https`, `net`, `os`, `path`, `process`, `punycode`, `querystring`, `readline`, `stream`, `string_decoder`, `timers`, `tty`, `url`, `util`, `v8`, `vm`, `zlib`

> Third-party packages that use TCP/UDP sockets or file system access will **fail** — the runtime stubs or blocks those operations.

## What is NOT supported

| Feature | Notes |
|---------|-------|
| WebSocket API | Not available |
| File system (`fs`) | No disk access |
| TCP/UDP sockets | No raw socket access |
| Binary responses | Functions cannot return binary data |
| Inter-function calls | An app function cannot call another app function |
| `eval()` | Blocked for security |
| `new Function()` | Blocked for security |
| `globalThis.window` | Deprecated; do not use |
| CommonJS `require()` | Use ES `import` instead |

## `@dynatrace-sdk/*` packages in functions

All SDK packages work in app functions. The runtime pre-bundles them, so imports are resolved without network access:

```typescript
import { queryExecutionClient } from "@dynatrace-sdk/client-query";

export default async function () {
  const res = await queryExecutionClient.queryExecute({
    body: { query: "fetch logs | limit 5" },
  });
  return res.result?.records;
}
```

## Calling internal platform APIs with `fetch`

Within a function, `fetch` calls to `/platform/...` paths are authenticated automatically — no token management needed:

```typescript
export default async function () {
  const res = await fetch("/platform/storage/query/v1/query:execute", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ query: "fetch metrics | limit 5" }),
  });
  return res.json();
}
```

## Calling external APIs

External hosts must be declared in `app.config.json` under `allowedOutboundConnections.hosts`:

```json
{
  "allowedOutboundConnections": {
    "hosts": ["api.github.com", "hooks.slack.com"]
  }
}
```

Calls to unlisted hosts will fail with a network error.

## Performance tips

- Avoid large in-memory datasets — stay well under 256 MB
- Stream or paginate large Grail results instead of fetching everything at once
- Return only the fields the UI needs — keep responses under 5 MB
- Use lazy loading in the React frontend (`React.lazy` + `Suspense`) to keep initial bundle small
- Implement SWR (stale-while-revalidate) caching for frequently-read, slowly-changing data
