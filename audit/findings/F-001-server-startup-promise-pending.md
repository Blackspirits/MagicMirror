# F-001 — Server startup promise remains pending after listen errors

This file/record belongs to the **audit workstream**.

## Required fields

- Domain: reliability / correctness
- Severity: MEDIUM
- Confidence: HIGH
- Evidence strength: OBSERVED
- Verification summary: Deterministic source review shows the listen-error path logs and returns without resolving or rejecting the promise returned by `Server.open()`. Runtime reproduction has not yet been executed.
- Status: OPEN
- Location: `js/server.js` (`Server.open()`), caller `js/app.js` (`await this.#httpServer.open()`)

## Observed

`Server.open()` returns a new Promise that only receives `resolve`. Before calling `listen()`, the server registers a one-shot `error` handler.

When `listen()` emits an error such as `EADDRINUSE`, the handler logs a friendly message and returns. It does not resolve or reject the startup promise.

`App.start()` awaits `this.#httpServer.open()`. Because the promise remains pending after the listen error, the surrounding startup `try/catch` cannot observe the failure and execute its failure/exit path.

## Expected

A terminal listen failure should settle the startup promise, normally by rejecting it, so the caller can handle startup failure deterministically and exit or recover rather than remain pending.

## Evidence

- Baseline: `MagicMirrorOrg/MagicMirror@b0c88b4a5dab37958c0ac2c90db5a5c736a88d42`.
- `js/server.js`: `open()` constructs `new Promise((resolve) => { ... })`; its `this.#server.once("error", ...)` handler logs `EADDRINUSE`/other errors and returns without settling the promise.
- `js/app.js`: startup executes `const { app, io } = await this.#httpServer.open();`; only a rejected/throwing path reaches the surrounding catch.
- Historical context: upstream PR #3890 added clearer occupied-port messaging, while PR #3920 explicitly addresses port release in watch mode. No current open issue matching the exact `PORT IN USE` condition was found during this pass.

## Impact

A user starting MagicMirror² when the configured port cannot be bound can receive the error message but the startup operation may remain alive/pending rather than failing cleanly. This can confuse process managers, scripts, service supervisors, and manual troubleshooting because the process does not reach the normal startup-failure exit path.

The same unresolved-promise behavior applies to other server `error` events emitted during the initial listen operation.

## Verification checks

| Check | State | Result/evidence |
|---|---|---|
| Source control-flow reconstruction | EXECUTED | Error handler cannot settle the Promise because no `reject` callback exists; caller awaits it. |
| Search existing upstream issue/PR history | EXECUTED | Related PRs #3890/#3920 found; no open issue for the exact pending-startup behavior found. |
| Runtime reproduction with occupied port | NOT_RUN | Requires executable target environment. |
| Regression test against proposed fix | NOT_RUN | No fix has been authorized or implemented. |

## Conditional fields

### Reproduction

Not yet reproduced at runtime.

Expected minimal reproduction:

1. Bind the configured MagicMirror² port with another process.
2. Start MagicMirror² server-only or full Electron mode.
3. Observe the `PORT IN USE` message.
4. Verify whether the MagicMirror² process remains pending instead of rejecting startup/exiting non-zero.

### Root cause

The `Server.open()` Promise has no rejection path for the server's initial `error` event.

### Proposed fix

When remediation is authorized, use the smallest change that settles the Promise on pre-listen/listen failure, e.g. construct it with `(resolve, reject)` and reject from the one-shot startup error handler after logging. Ensure this does not introduce duplicate/unhelpful logging at the `App.start()` layer.

### Regression test

Add a focused test that occupies a port, invokes the server startup path, and asserts that startup rejects/terminates rather than remaining pending.

### Fix risk

Low-to-medium. Changing startup failure propagation can affect existing process-manager/watch-mode behavior and logging, so server-only, Electron, and watch-mode paths should be checked.

### Validation performed

Static source/control-flow analysis and upstream-history search only.

### Priority / triage

High within the reliability workstream because it affects startup failure handling, but severity remains MEDIUM due to availability/operability impact rather than data loss or privilege escalation.

### Independent review

Not yet performed.

## Notes

A finding does not authorize a fix. No target production change has been made.
