# PROJECT_OVERLAY — MagicMirror²

Stable target-specific context for the Software Audit OS workstream.

## Target

- Repository: `MagicMirrorOrg/MagicMirror`
- Baseline branch: `develop`
- Baseline revision: `b0c88b4a5dab37958c0ac2c90db5a5c736a88d42`
- Purpose: open-source modular smart-mirror platform and local/private-network dashboard runtime.
- Audit methodology: `Blackspirits/software-audit-os@39e294f644e8122103e4a0d18418e769a20159c5`

## Users and critical workflows

- User/system actors:
  - local MagicMirror² operator/administrator;
  - browser/Electron client;
  - MagicMirror² server;
  - installed first- and third-party modules/node helpers;
  - remote calendar/feed/weather/API services.
- Critical workflows:
  - load and validate configuration;
  - start HTTP/HTTPS server and Electron client;
  - load browser modules and server-side node helpers;
  - exchange module notifications over Socket.IO;
  - fetch remote calendar/news/weather data;
  - render module HTML/templates and translations;
  - check and report software/module updates.

## Technology stack

- Languages: JavaScript, CSS, Nunjucks/HTML, JSON/YAML/Markdown.
- Runtime: Node.js; Electron for the desktop wrapper.
- Server framework: Express 5.
- Realtime transport: Socket.IO.
- Frontend: browser JavaScript/ESM, Nunjucks, CSS.
- Package/build system: npm / package.json; no separate compilation step required for the main application.
- Test framework: Vitest, Playwright/browser-driven tests, Electron regression tests.
- Static analysis/quality: ESLint, Prettier, cspell, CodeQL, GitHub dependency review.

## Architecture

- Main entry points:
  - `js/electron.js` — Electron wrapper;
  - `serveronly/index.js` — server-only entry point;
  - `clientonly/index.js` — remote client-only launcher;
  - `js/app.js` — core application lifecycle;
  - `js/server.js` — HTTP/HTTPS + Socket.IO server.
- Module system:
  - browser-side `Module` abstraction in `js/module.js`;
  - server-side `NodeHelper` abstraction in `js/node_helper.js`;
  - first-party modules under `defaultmodules/`;
  - third-party modules are loaded from the configured modules directory.
- Browser/server boundary:
  - HTTP endpoints provide configuration/runtime metadata and static assets;
  - Socket.IO namespaces connect module frontends to node helpers.
- Persistence:
  - no application database is present in the core repository;
  - primary persistent state is file/configuration and module-owned state.
- Background/async:
  - timers, network fetchers, module node helpers and Socket.IO;
  - no core queue/broker was identified during bootstrap.

## Data

- Persistent stores: no core relational/document database identified.
- Configuration sources: JavaScript config, optional environment/config.env values, process environment.
- Sensitive data may include API credentials and module secrets in configuration.
- Secret redaction/restoration exists when `hideConfigSecrets` is enabled.

## APIs and integrations

- Core HTTP endpoints include `/config`, `/cors`, `/version`, `/startup`, `/env`, `/reload`, and `/`.
- Socket.IO provides realtime module/node-helper communication.
- External integrations include calendar feeds, RSS/Atom feeds, weather providers, update checks, and module-defined network integrations.
- Third-party modules are an intentional extension boundary and may execute server-side code.

## Async work and concurrency

- Repeating timers and fetch retry/backoff are used for remote data.
- Node helpers execute in the MagicMirror² server process.
- Socket.IO can deliver concurrent client events.
- No central queue/transaction manager is present in core.

## Build and test

- Install/dev: `node --run install-mm:dev`
- Production dependency install: `node --run install-mm`
- Full test suite: `node --run test`
- Unit: `node --run test:unit`
- E2E: `node --run test:e2e`
- Electron: `node --run test:electron`
- Coverage: `node --run test:coverage`
- Lint/format check: `node --run test:lint`
- Autofix: `node --run lint:fix`
- Spelling: `node --run test:spelling`
- Config validation: `node --run config:check`

## CI/CD and deployment model

- CI: GitHub Actions.
- Automated tests run on Node 22.x, 24.x and 26.x plus a minimum-version installation check.
- CodeQL runs for JavaScript/TypeScript and Actions.
- Pull requests receive dependency review and branch-policy enforcement.
- Deployment is primarily user-managed/local rather than a centralized hosted service.
- Release/rollback mechanics: not yet fully verified.

## Security and trust boundaries

- Project security policy states that MagicMirror² is primarily intended for trusted local/private networks and direct public Internet exposure is not recommended.
- Network access is constrained by configurable IP whitelist/trusted-proxy logic.
- HTTP hardening uses Helmet.
- HTTPS is supported with configured key/certificate files.
- Electron uses `contextIsolation: true` and `nodeIntegration: false` by default.
- User options can deliberately weaken framing/CSP checks.
- Third-party modules/node helpers are trusted executable extensions and therefore a major trust boundary.

## Project conventions

- Contributions target `develop`, not `master`.
- Formatting: Prettier.
- Linting: ESLint.
- Tests: Vitest.
- Current package version at bootstrap: `2.38.0-develop`.
- Supported Node engine: `>=22.22.2 <23 || >=24`.

## High-risk areas

- Configuration/secret redaction and browser exposure.
- IP allow-list and trusted-proxy handling.
- `/cors` network proxy behavior and SSRF boundaries.
- Third-party module/node-helper loading.
- Socket.IO trust and module namespace isolation.
- Electron certificate/CSP/frame-header overrides and remote-client mode.
- Dynamic function serialization/reconstruction in configuration.
- Remote HTTP fetch/retry/TLS behavior.
- Update and release logic.
- Shutdown/error lifecycle and recovery after uncaught exceptions.

## Evidence anchors

- `package.json`
- `README.md`
- `.github/SECURITY.md`
- `.github/CONTRIBUTING.md`
- `.github/workflows/*.yaml`
- `js/app.js`
- `js/server.js`
- `js/electron.js`
- `js/node_helper.js`
- `js/module.js`
- `clientonly/index.js`
- `defaultmodules/`
- `tests/`

## Unknown / not yet verified

- Complete release/tag/rollback process.
- Full external API/provider contract inventory.
- Runtime behavior on every supported OS/display stack.
- Third-party-module ecosystem security, which is outside core-repository source unless explicitly sampled.
