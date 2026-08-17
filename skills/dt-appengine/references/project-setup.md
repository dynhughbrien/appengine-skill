# Project Setup

## Prerequisites

- Node.js v24 (`node --version` to verify)
- Basic React and TypeScript knowledge
- A Dynatrace environment URL in the form `https://{environment-id}.apps.dynatrace.com/`

## Create a new app

```bash
npx dt-app@latest create
```

When prompted:
- Enter an app name (e.g. `my-first-app`)
- Enter your environment URL

The toolkit downloads all project files and installs Node dependencies automatically. If you are already authenticated, the environment URL is pre-filled.

Navigate into the project and start the dev server:

```bash
cd my-first-app
npx dt-app dev
```

The app opens in Chrome automatically. Source changes trigger instant browser refresh.

> **Chrome 142+**: You may be prompted to allow local network connections. Accept to continue.

## Directory layout

```
my-first-app/
├── api/                  # Backend app functions (one file = one endpoint)
│   └── example.ts
├── ui/
│   └── app/
│       └── pages/
│           └── Home.tsx  # Main React page
├── app.config.json       # App metadata, permissions, environment
├── package.json
└── tsconfig.json
```

Every `.ts` file placed directly inside `api/` is compiled and deployed as an app function endpoint. Subdirectories are **not** picked up.

## app.config.json

```json
{
  "id": "my.app.id",
  "version": "1.0.0",
  "name": "My App",
  "description": "Short description shown in the app catalog",
  "icon": "checkmark",
  "environmentUrl": "https://{environment-id}.apps.dynatrace.com/",
  "scopes": [
    {
      "name": "storage:buckets:read",
      "comment": "Read Grail buckets"
    }
  ]
}
```

### Key fields

| Field | Required | Notes |
|-------|----------|-------|
| `id` | Yes | Reverse-domain style unique identifier, e.g. `com.mycompany.myapp` |
| `version` | Yes | **Must be incremented** before every re-deploy; cannot re-deploy same version without uninstalling first |
| `name` | Yes | Display name in the app catalog |
| `environmentUrl` | Yes | Target Dynatrace environment |
| `scopes` | No | IAM permission scopes the app needs; user must grant these on install |
| `icon` | No | Strato icon name shown in navigation |
| `description` | No | Catalog description |

## UI — Strato components

The default template imports from `@dynatrace/strato-components`:

```tsx
import { Flex, Heading, Paragraph } from "@dynatrace/strato-components";

export function Home() {
  return (
    <Flex flexDirection="column">
      <Heading>Hello World</Heading>
      <Paragraph>My first Dynatrace app</Paragraph>
    </Flex>
  );
}
```

Browse all available components, design tokens, and icons at:
`https://developer.dynatrace.com/develop/ui-components/`

## Local development commands

| Command | Purpose |
|---------|---------|
| `npx dt-app dev` | Start local dev server with hot reload |
| `npx dt-app build` | Production build (used before deploy) |
| `npx dt-app deploy` | Build and deploy to the configured environment |
| `npx dt-app uninstall` | Remove the app from the environment |
