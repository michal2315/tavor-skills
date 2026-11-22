# Client (Angular) - @tavor/observability-angular

What the client gets: page load times, every route change named by pattern (`/orders/:id`), JavaScript
errors, the signed-in user, and the trace id sent with calls to the server, so browser and server show
as one trace. Off when `serverUrl` is empty, so local dev and tests are unaffected.

## 1. Install

```bash
npm install --save-exact @tavor/observability-angular
```

Same version as `@tavor/observability-node`.

In `angular.json`, in the build options of the app, add (or extend) to silence CommonJS warnings:

```json
"allowedCommonJsDependencies": ["opentracing", "error-stack-parser"]
```

## 2. Config value

Add `apmServerUrl` to the app's existing config, **empty by default**:

| Project uses | Do |
|---|---|
| `environment*.ts` | add `apmServerUrl: ''` to the base/dev environment; add the field to its type/interface if there is one |
| runtime `env-config.js` / `window._env_` | add `apmServerUrl` there with an empty value, same pattern as the other keys |
| a config service / `APP_CONFIG` token | add the field there |

Use the project's existing environment name value if it has one (for `environment` below). Don't add a
new config mechanism.

## 3. Provider

Standalone app, `app.config.ts`, at the end of `providers`:

```ts
import { provideObservability } from '@tavor/observability-angular';

provideObservability({
  project: '<project>',
  component: '<project>-client',
  environment: <existing env name value, if any>,
  serverUrl: <the apmServerUrl config value>,
}),
```

NgModule app: the same call in `AppModule`'s `providers`.

- **Existing `ErrorHandler` provider**: keep it. Put `provideObservability(...)` **before** it, and inside
  their `handleError` add `getApm()?.captureError(error)` (import `getApm` from the package).
- **API on another origin** (absolute URL to a different host): add `apiOrigins: ['<that origin>']`
  and check CORS on the server (backend.md section 7).
- If config is read at runtime through DI (config service / injection token), pass the value the same
  way the app's other providers get it. If that isn't possible in a provider list, report it.

## 4. User

Where the app gets the signed-in user (auth service after login, current-user load, app init):

```ts
import { setUser } from '@tavor/observability-angular';

setUser({ id: <user unique id>, username: <display name> });
```

Only id and display name. Add `email` only if the project's `CLAUDE.md` or the user says it's allowed.
If there are several places, use the one that runs on every app load (not only on the login form).

## 5. Client logger (user monitoring)

Find the service that sends user events to the monitoring endpoint (e.g. a `track(...)` method doing
`fetch` / `HttpClient.post` to a monitoring URL). Where the event object is built, add the trace id:

```ts
import { currentTraceIds } from '@tavor/observability-angular';

body: JSON.stringify({ ...event, 'trace.id': currentTraceIds()?.traceId }),
```

- Add only this field. Keep every existing field and the send logic.
- `currentTraceIds()` returns `undefined` when tracing is off or nothing is measured; JSON drops the
  field then. Safe to call always.
- No client logger, or unclear which one: skip and report.

## 6. Content-Security-Policy

If the app sets a CSP (`index.html` meta tag, nginx config, server headers), the APM URL must be in
`connect-src`. The URL isn't known yet, so **report it** with the file and line; don't edit the CSP.
