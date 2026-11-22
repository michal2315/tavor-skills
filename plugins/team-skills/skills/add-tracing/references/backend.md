# Server (NestJS) - @tavor/observability-node

What the server gets without any other code: every incoming request traced and named by route
(`GET /orders/:id`), Nest handlers, MongoDB / Mongoose, Postgres, Redis, S3, Elasticsearch (client 8.15+),
outgoing HTTP / fetch calls, errors, and `trace.id` / `span.id` on every winston log written during a
request. Off unless the deployment sets an endpoint, so local dev is unaffected.

## 1. Registry

`@tavor/*` comes from Artifactory. If the project's `.npmrc` has no `@tavor:registry=` line, add the one
the team uses for `@tavor` packages. Don't add tokens to the file.

## 2. Install

```bash
npm install --save-exact @tavor/observability-node
```

## 3. Start command

Tracing must start before NestJS and the database drivers load, so it's a `--require` flag, not an
import in `main.ts`. Change the command that starts the app **in production**:

```diff
- node dist/main.js
+ node --require @tavor/observability-node/register dist/main.js
```

| What starts production | Change |
|---|---|
| Dockerfile runs `npm start` / `npm run start:prod` | the script in `package.json` |
| Dockerfile runs `node dist/main.js` | the Dockerfile `CMD` / `ENTRYPOINT` |
| Neither can change | env `NODE_OPTIONS=--require @tavor/observability-node/register` in the deployment (report it) |

- Keep the entry file path exactly as it is (`dist/main.js`, `dist/src/main.js`, ...).
- Leave `start:dev` / `nest start --watch` alone.
- `import 'dotenv/config'` in `main.ts` runs after the register step. Fine in production (env comes from
  OpenShift). Don't change it.

## 4. Bundling check

The package hooks libraries as Node loads them from `node_modules`. If the build bundles them into
`dist/main.js`, nothing is traced.

- Plain `nest build` (no webpack): fine.
- `nest-cli.json` with `"webpack": true`, `"builder": "webpack"` or `"monorepo": true`: Nest's default
  webpack config keeps `node_modules` external, so fine, **unless** a custom `webpack.config.js` has no
  `webpack-node-externals` / `externals`. Then **stop and report**. Don't change the build.

## 5. Logs

**Project has the shared winston LoggerService** (the usual case): don't replace it and don't add
`TavorLogger`. Winston records get `trace.id` and `span.id` automatically. Check only that they're
printed: find where the logger is created (`createLogger(`, `winston.createLogger(`,
`WinstonModule.forRoot(`) and look at `format`.

| Format | Change |
|---|---|
| `format.json()`, `format.logstash()`, `ecsFormat()`, or `combine(...)` ending in one | none |
| `format.printf(...)` / custom template | add to the template, nothing else: ``${info['trace.id'] ? ` trace.id=${info['trace.id']}` : ''}`` |
| unclear, several loggers | none - report it |

**Project logs only with Nest's built-in `Logger`**: in `main.ts`, right after `NestFactory.create`:

```ts
import { TavorLogger } from '@tavor/observability-node';

const app = await NestFactory.create(AppModule, { bufferLogs: true });
app.useLogger(new TavorLogger());
```

Add `bufferLogs: true` if missing. Don't change existing `new Logger(...)` calls. Output stays Nest's text
format with `trace.id=… span.id=…` appended inside a trace.

**`app.useLogger(...)` already set with something else**: leave it, report it.

## 6. Background work

Work that doesn't start from an HTTP request has no trace. Wrap **one run**, not the loop or the
scheduler:

```ts
import { withSpan } from '@tavor/observability-node';

@Interval(30_000)
async sendPending() {
  await withSpan('mail.send-pending', async () => {
    ...existing body unchanged...
  });
}
```

- Look for `@Cron`, `@Interval`, `@Timeout`, `setInterval`, `setTimeout` loops, queue / message
  consumers, long startup jobs.
- Span names: `<area>.<action>`, lower case, kebab-case action (`mail.send-pending`, `sync.users`).
- Keep the body as it is; only wrap it. Keep `return` values.
- Don't add `@Traced()` or `addSpanAttributes` on your own. They're for the developer to add later where
  they need detail.

## 7. CORS

- Client calls the API on the same origin (proxy, `/api/...`): nothing.
- `enableCors({...})` with an `allowedHeaders` list: add `'traceparent', 'tracestate'` to it.
- `enableCors` without `allowedHeaders`: nothing (requested headers are allowed).

## 8. `.env.example`

If the project has one, append (commented, no values):

```bash
# Tracing (tavor-trace). Unset endpoint = off.
# OTEL_EXPORTER_OTLP_ENDPOINT=
# OTEL_EXPORTER_OTLP_HEADERS=
# OBS_PROJECT=<project>
# OBS_COMPONENT=<project>-server
# OBS_ENVIRONMENT=
```

## 9. Deployment env (report, or edit if the manifests are in this repo)

```yaml
env:
  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: <APM endpoint>
  - name: OTEL_EXPORTER_OTLP_HEADERS
    valueFrom:
      secretKeyRef: { name: <apm secret>, key: <key> }
  - name: OBS_PROJECT
    value: <project>
  - name: OBS_COMPONENT
    value: <project>-server
  - name: OBS_ENVIRONMENT
    value: <dev | test | prod>
```

If the manifests are in the repo, add these with the endpoint and secret name as clearly marked
placeholders and say so in the report. Never invent real URLs or secret names.

Optional knobs (don't set unless asked): `OBS_IGNORE_PATHS` (default skips health/ready/live/metrics),
`OBS_ENABLED=false` (kill switch), `OTEL_TRACES_SAMPLER` / `OTEL_TRACES_SAMPLER_ARG` (sampling).
