# Wizz Native — Architecture, Tooling & Server Deployment

## Architecture & Workflow Decision Matrix

| Goal | Tool / Command | Why |
| :--- | :--- | :--- |
| Project initialization | `wizz init [directory]` | Scaffolds standard project structure (`index.html`, `src/App.wizz`, pages, components, and server API handlers). |
| Development server & live reload | `wizz dev [--port <n>]` | Incremental compilation, live reloading via SSE (`/_wizz/reload`), and SSR document hydration at `http://localhost:3000`. |
| Production build | `wizz build --adapter node` | Compiles `.wizz` into `dist/`, extracts scoped CSS to `dist/app.css`, emits route/API manifests and `dist/server.mjs`. |
| CI/CD diagnostic automation | `wizz build --json` | Emits machine-parsable `wizz-build-diagnostics@1` JSON envelope for automated build gates. |
| Server API handler | `src/server/api/*.js` | Zero-dependency server-side endpoints executing behind a frozen request context. |
| Production deployment | `node dist/server.mjs` | Traversal-guarded, high-performance Node.js production server with SSR and integrated API routing. |
| Server secret management | `.env.server` in project root | Loaded by Node host at startup; kept outside `src/` so secrets are never bundled or exposed to client. |

---

## Canonical Project Architecture

Wizz projects adhere to a clean, convention-based directory tree:

```text
my-wizz-app/
├── index.html              # Document shell with HTML mount point
├── .env.server             # Production & development server secrets (NEVER in src/)
├── package.json            # Project manifest ({"type": "module"})
├── src/
│   ├── App.wizz            # Root component router / application shell
│   ├── App.css             # Global reset, design tokens (:root), and high-level layout
│   ├── pages/              # Routed page components
│   │   ├── Home.wizz
│   │   └── About.wizz
│   ├── components/         # Reusable UI components
│   │   ├── Header.wizz
│   │   └── Card.wizz
│   └── server/
│       └── api/            # Server-side API route handlers
│           ├── status.js
│           └── users.js
└── dist/                   # Build output emitted by `wizz build`
    ├── package.json        # {"type": "module"}
    ├── app.css             # Extracted & scoped CSS bundle
    ├── server.mjs          # Standalone server host (when --adapter node is used)
    └── ...
```

---

## Document Shell (`index.html`)

The document shell acts as the mounting container for Wizz applications:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="/app.css" />
  </head>
  <body>
    <!-- Wizz SSR injects rendered HTML and serialized hydration state here -->
    <div id="app"></div>
    <script type="module" src="/App.js"></script>
  </body>
</html>
```

---

## Server API Handlers (`src/server/api/*.js`)

Server endpoints are placed in `src/server/api/`. Each file exports a default handler function that accepts a frozen request context:

```javascript
// src/server/api/telemetry.js
export default async function handleTelemetry(context) {
  const { method, url, headers, query, body } = context;

  if (method !== "GET") {
    return {
      status: 405,
      headers: { "content-type": "application/json" },
      body: JSON.stringify({ error: "Method not allowed" })
    };
  }

  // Access server secrets loaded from .env.server or process.env
  const apiKey = process.env.SERVICE_SECRET;

  return {
    status: 200,
    headers: {
      "content-type": "application/json",
      "cache-control": "no-store"
    },
    body: JSON.stringify({
      status: "operational",
      timestamp: Date.now(),
      authorized: Boolean(apiKey)
    })
  };
}
```

### Server Security Rules
1. **Never Statically Served**: Wizz dev and node adapters strictly guard `/server/**` paths. Handlers are never served as static assets.
2. **Secrets Stay in Root**: `.env.server` must reside in the project root, never inside `src/`. Real host environment variables are preserved and take precedence.
3. **Route Collisions**: If two API handlers resolve to the same route, the compiler fails immediately rather than guessing precedence.

---

## Build & Production Deployment

### Compiling for Production
Run the build command specifying the Node adapter:
```bash
wizz build --adapter node
```
This generates:
- `dist/package.json` with ESM module configuration.
- `dist/app.css` containing all extracted `<wizz:style>` blocks.
- `dist/server.mjs` standalone SSR & API server.
- Route and API manifests.

### Executing the Production Server
Deploy the `dist/` folder and launch the standalone server:
```bash
# Standard launch (default port 8080)
node dist/server.mjs

# Custom port launch
PORT=3000 node dist/server.mjs
```

---

## Stable Compiler Diagnostic Codes

When a compilation or syntax error occurs, the Wizz compiler produces structured diagnostic codes:

- **`WIZZ-P###`**: Parser stage diagnostics (e.g. malformed tag structure, multiple root elements, invalid template expressions).
- **`WIZZ-G###`**: Generator stage diagnostics (e.g. invalid loop body composition, illegal directives, invalid props).
- **`WIZZ-A###`**: Reserved analyzer namespace.

Use `wizz build --json` in CI/CD pipelines to parse errors deterministically without fragile regex matching on error prose.

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Placing `.env.server` in `src/` or `dist/` | Keep `.env.server` in the project root | Files inside `src/` risk client bundling or unauthorized exposure. |
| Importing npm packages in `.wizz` components | Use native browser APIs and standard ECMAScript | Wizz is designed as a zero-dependency platform compiler; external bundles break portability. |
| Manually writing custom build scripts with Webpack or Rollup | Use `wizz build --adapter node` | Wizz handles CSS extraction, hashing, SSR routes, and API manifests natively. |
| Hardcoding port numbers in server scripts | Read `process.env.PORT` (`PORT=3000 node dist/server.mjs`) | Port hardcoding prevents flexible deployment across container hosts and PaaS environments. |
| Fetching client-side from `/src/server/api` | Fetch from `/api/[endpoint]` | Wizz routes `/api/**` to server handlers; `/src` is not exposed in production. |
| Attempting to render dynamic CSS in `<style>` tags | Use `<wizz:style>` with CSS custom properties | `<wizz:style>` is static and scoped; dynamic appearance should be handled via CSS variables (`style="--accent: {color}"`). |
