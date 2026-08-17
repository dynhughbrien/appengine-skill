# Security

## Permission scopes

Apps declare the IAM scopes they need in `app.config.json`. Users are prompted to grant these when installing the app.

```json
{
  "scopes": [
    {
      "name": "storage:logs:read",
      "comment": "Read application logs from Grail"
    },
    {
      "name": "storage:metrics:read",
      "comment": "Read metrics from Grail"
    },
    {
      "name": "document:documents:write",
      "comment": "Store user-created documents"
    }
  ]
}
```

### Common scopes

| Scope | Purpose |
|-------|---------|
| `storage:logs:read` | Query logs in Grail |
| `storage:metrics:read` | Query metrics in Grail |
| `storage:events:read` | Query events in Grail |
| `storage:spans:read` | Query distributed traces |
| `storage:buckets:read` | List Grail buckets |
| `document:documents:read` | Read documents |
| `document:documents:write` | Create/update documents |
| `app-settings:objects:read` | Read app settings |
| `app-settings:objects:write` | Write app settings |
| `settings:objects:read` | Read Dynatrace platform settings |
| `app-engine:apps:install` | Deploy apps (needed for CI/CD identity) |
| `app-engine:apps:run` | Execute app functions |

### Deployment scopes (CI/CD OAuth client)

The OAuth client used in CI/CD pipelines needs:
- `app-engine:apps:install`
- `app-engine:apps:run`
- `app-engine:apps:delete` (only if `npx dt-app uninstall` is used in CI)

## Content Security Policy (CSP)

AppEngine enforces CSP on all apps. By default, only Dynatrace-hosted scripts and styles are allowed.

To load resources from external domains, declare them in `app.config.json`:

```json
{
  "csp": {
    "connectSrc": ["https://api.example.com"],
    "imgSrc": ["https://images.example.com"],
    "scriptSrc": ["'unsafe-eval'"]
  }
}
```

Avoid `'unsafe-inline'` and `'unsafe-eval'` unless strictly necessary — they weaken the security boundary.

## Secrets management

**Never expose secrets in frontend code.** All credential handling must happen inside app functions (the `/api` directory):

1. Store secrets using the Settings SDK or credential vault (server-side only)
2. Read them inside an app function
3. Return only the data the frontend needs, never the credential itself

```typescript
// api/fetch-external-data.ts — secret stays server-side
export default async function () {
  const settingsRes = await fetch("/platform/classic/environment/v2/settings/objects?schemaIds=app:com.mycompany.myapp:secrets");
  const { value: { apiKey } } = (await settingsRes.json()).items[0];

  const externalData = await fetch("https://api.example.com/data", {
    headers: { Authorization: `Bearer ${apiKey}` },
  });
  return externalData.json();  // Only the data goes to the frontend
}
```

## Querying user permissions at runtime

Check what the current user can do before rendering sensitive UI:

```typescript
import { iamClient } from "@dynatrace-sdk/client-iam";

const user = await iamClient.getCurrentUser();
const hasAdminRole = user.groups?.includes("admin-group-id");
```

## Third-party dependencies

- Audit all `npm` dependencies before adding them — unpublished packages or packages with broad system access can be a supply-chain risk
- Third-party packages that use TCP/UDP sockets or file system access will **fail at runtime** in app functions (the JS runtime does not support these)
- Pin dependency versions and use `npx dt-app automate-dependency-updates` (Renovate integration) to stay current
