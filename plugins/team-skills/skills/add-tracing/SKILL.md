---
name: add-tracing
description: Connect a project to Elastic APM tracing with our tavor-trace packages (@tavor/observability-node for NestJS, @tavor/observability-angular for Angular) - survey the app, plan, get approval, wire the server and client, verify, and report the env vars to set in OpenShift/ArgoCD. Use this whenever the user asks to add/connect/enable tracing, APM, Elastic APM, OpenTelemetry, observability or tavor-trace in a project ("connect this app to trace", "add APM to summer", "wire tavor-trace"), or to upgrade the @tavor/observability packages.
---

# Add tracing

Wires one project to tavor-trace. The goal is the smallest change that gives full tracing, done the
same way in every project, without touching anything the project depends on (logging pipeline,
build, deployment secrets).

Apps differ, so **look first, change second**. When something doesn't match what this skill expects,
stop and report it instead of guessing.

How each step is done is in:
- `references/backend.md` - NestJS server
- `references/frontend.md` - Angular client

## 0. Load project context

Read the project's `CLAUDE.md`: **Commands**, **Conventions**, **Do not touch without asking**, and any
project rules (for example "no tests"). Follow them over anything here.

## 1. Survey (read only)

Find and write down, with file paths:

**Whole project**
- Where the server and the client live (one repo with both, or only one of them).
- **Existing tracing.** Search for `elastic-apm-node`, `@elastic/apm-rum`, `@opentelemetry/`,
  `dd-trace`, `newrelic`, `--require` / `--import` in start commands and `NODE_OPTIONS`, and the
  OpenTelemetry Operator annotation `instrumentation.opentelemetry.io/inject-nodejs` in deployment files.
  **If anything is found, stop and report.** Two tracers in one app record everything twice.
- Deployment files in the repo (Dockerfile, Helm values, k8s / OpenShift YAML, kustomize). Often the
  ArgoCD config lives in a separate repo; then env vars go in the report instead.

**Server** (see `references/backend.md` for what to look for)
- Node version (Dockerfile `FROM`, `.nvmrc`, `engines`). Needs **18.19+ or 20.6+**.
- NestJS version. Needs **9+**.
- The command that starts the app **in production**.
- Bundling: `nest-cli.json` and any `webpack.config.js`.
- Logging: the winston LoggerService and its `format`, or Nest's built-in `Logger` only.
- Background work: `@Cron`, `@Interval`, `@Timeout`, `setInterval`, queue consumers, startup jobs.
- CORS: `enableCors(...)` and whether the client calls the API on the same origin.

**Client** (see `references/frontend.md`)
- Angular version. Needs **16+**.
- Standalone (`app.config.ts`) or NgModule (`AppModule`).
- How config is done (`environment*.ts`, runtime `env-config.js` / `window._env_`, config service).
- An existing `ErrorHandler` provider.
- Where the signed-in user is loaded.
- The client logger (user monitoring service that sends events).
- A Content-Security-Policy (`index.html` meta, nginx config).

A version below the minimum is a **blocker**: report it and don't wire that part.

## 2. Plan and wait for approval

```
## Tracing plan - <project>

Blockers: none | <what, where, what the user must do>

Server (<path>)
- install @tavor/observability-node
- start command: <file> - <before> → <after>
- logger: <winston json: no change | printf: add trace.id to template in <file> | Nest Logger: TavorLogger in main.ts>
- background jobs wrapped in withSpan: <file:line - span name> ...
- CORS: no change (same origin) | <file> add traceparent, tracestate
- .env.example: add OTEL_* / OBS_* (commented)

Client (<path>)
- install @tavor/observability-angular
- config: apmServerUrl (empty by default) in <file>
- provideObservability in <file> with project=<..> component=<..>
- setUser in <file>
- client logger: add trace.id in <file> | none found
- CSP: none | <file>

Names: project=<..>, server component=<..>, client component=<..>
Not changing: <anything unusual you found and are leaving alone>
```

Propose names the user can confirm: `project` = the project's name, `component` = the package /
deployment name (`<project>-server`, `<project>-client`). Use the same project and environment values
the project's logs already use.

**Stop and wait for the user to approve.**

## 3. Apply

Server first, then client, following the references. For each change:
- Change only what the plan says. Keep every existing field, option and line around it.
- Install both packages at the **same version** (`npm install --save-exact`).
- Never write endpoints, tokens or secrets into files. The user fills them in on the deployment.

## 4. Verify

Use the **Commands** in `CLAUDE.md`:
1. Install, typecheck and build the server and the client.
2. If the server can run locally, start it once: the first line must be
   `[tavor-observability] tracing off (OTEL_EXPORTER_OTLP_ENDPOINT is not set)`. That proves the
   start command loads the package. Stop it again.
3. Don't add tests unless the project's `CLAUDE.md` asks for them.

A failing check is a blocker: fix it if it's caused by this change, otherwise report it.

## 5. Report

```
## Tracing - <project>

| Part | Status | Where |
|------|--------|-------|
| Server package + start command | ✅ / ⏭ skipped (why) / ⛔ blocked (why) | file |
| Logs carry trace.id | ✅ / ... | file |
| Background jobs | ✅ n wrapped / none | files |
| Client provider | ✅ / ... | file |
| User | ✅ / ... | file |
| Client logger trace.id | ✅ / ... | file |
| Build / typecheck | ✅ / ❌ | |

### Set in OpenShift / ArgoCD (server)
OTEL_EXPORTER_OTLP_ENDPOINT = <APM endpoint>          (you fill in)
OTEL_EXPORTER_OTLP_HEADERS  = from Secret             (you fill in)
OBS_PROJECT = <..>   OBS_COMPONENT = <..>   OBS_ENVIRONMENT = <per environment>

### Set for the client
apmServerUrl = <APM URL reachable from browsers>      (you fill in)

### Left for you
- <anything skipped, unusual, or needing a decision>
```

Don't commit unless the user asks. Follow **Git & MR** in `CLAUDE.md` when they do.

## Upgrading

When asked to upgrade: bump `@tavor/observability-node` and `@tavor/observability-angular` to the same
new version, read the package `CHANGELOG.md` for required app changes, apply them, then verify (step 4).
