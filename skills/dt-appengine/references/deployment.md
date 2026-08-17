# Deployment

## Prerequisites

The identity doing the deploy (user or OAuth client) needs these IAM permissions:
- `app-engine:apps:install`
- `app-engine:apps:run`
- `app-engine:apps:delete` (only needed for uninstall)

## Deploy from local machine

```bash
npx dt-app deploy
```

Run this from the project root. The toolkit builds the app and deploys it to the `environmentUrl` in `app.config.json`.

> **Important:** You must increment the `version` field in `app.config.json` before every re-deploy. If the version has not changed, the deploy fails — you must first uninstall with `npx dt-app uninstall` before re-deploying the same version.

## Uninstall

```bash
npx dt-app uninstall
```

## CI/CD deployment

### Step 1 — Create an OAuth client

In your Dynatrace environment, create an OAuth client (IAM → OAuth clients) with the scopes:
- `app-engine:apps:install`
- `app-engine:apps:run`

### Step 2 — Set environment variables

Expose the client credentials to your pipeline:

```
DT_APP_OAUTH_CLIENT_ID=<client-id>
DT_APP_OAUTH_CLIENT_SECRET=<client-secret>
```

When these variables are present, `npx dt-app deploy` uses them instead of interactive browser login.

### GitHub Actions

```yaml
name: Deploy Dynatrace App

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "24"

      - run: npm ci

      - name: Deploy
        run: npx dt-app deploy
        env:
          DT_APP_OAUTH_CLIENT_ID: ${{ secrets.DT_APP_OAUTH_CLIENT_ID }}
          DT_APP_OAUTH_CLIENT_SECRET: ${{ secrets.DT_APP_OAUTH_CLIENT_SECRET }}
```

### GitLab CI

```yaml
deploy:
  image: node:24
  script:
    - npm ci
    - npx dt-app deploy
  variables:
    DT_APP_OAUTH_CLIENT_ID: $DT_APP_OAUTH_CLIENT_ID
    DT_APP_OAUTH_CLIENT_SECRET: $DT_APP_OAUTH_CLIENT_SECRET
  only:
    - main
```

### Jenkins (Kubernetes agent)

```groovy
pipeline {
  agent {
    kubernetes {
      defaultContainer 'node'
      yaml """
spec:
  containers:
  - name: node
    image: node:24
"""
    }
  }
  stages {
    stage('Deploy') {
      steps {
        sh 'npm ci'
        withCredentials([
          string(credentialsId: 'dt-oauth-client-id', variable: 'DT_APP_OAUTH_CLIENT_ID'),
          string(credentialsId: 'dt-oauth-client-secret', variable: 'DT_APP_OAUTH_CLIENT_SECRET')
        ]) {
          sh 'npx dt-app deploy'
        }
      }
    }
  }
}
```

## Calling platform APIs from outside Dynatrace

For external tools or scripts that need to call Dynatrace APIs (not deploy via dt-app), use an OAuth token:

```bash
# Get a token
TOKEN=$(curl -s -X POST \
  "https://{sso-endpoint}/sso/oauth2/token" \
  -d "grant_type=client_credentials&client_id=$CLIENT_ID&client_secret=$CLIENT_SECRET&scope=storage:logs:read" \
  | jq -r .access_token)

# Use the token
curl -H "Authorization: Bearer $TOKEN" \
  "https://{environment-id}.apps.dynatrace.com/platform/storage/query/v1/query:execute" \
  -d '{"query":"fetch logs | limit 5"}'
```

See `/develop/guides/access-platform-apis-from-outside/` for the full OAuth flow.
