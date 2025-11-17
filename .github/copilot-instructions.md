# Copilot / AI Agent Instructions for datagroom-gateway

This file gives focused, actionable guidance for an AI coding agent working on `datagroom-gateway`.
Keep answers concise and reference the project files below when proposing code changes.

1. Big-picture architecture
- **Role**: This repository is the backend API/server for the Datagroom app (frontend is in `datagroom-ui`).
- **Core server**: `server.js` starts an Express app, serves the built React UI, and wires Socket.IO for realtime locks/updates.
- **Database**: MongoDB is the single persistent store. `dbAbstraction.js` provides a singleton wrapper around `mongodb` client and exposes common DB helpers (connect, find, pagedFind, insertOne, updateOne, transactions, archive).
- **Routes**: The `routes/` folder contains REST endpoints used by the UI (notably `routes/dsReadApi.js`, `upload*.js`). Use these to understand request shapes and validation.
- **Auth**: Authentication is cookie + JWT based. `server.js` handles login flow (LDAP via `passport-ldapauth` when enabled, guest fallbacks). Socket.IO connections are validated by inspecting the `jwt` cookie.

2. Developer workflows / common commands
- **Start (dev)**: `node server.js disableAD=true` — disables LDAP for quick local development with the `guest` user.
- **Start (production-like)**: `nohup ./startDg.sh &` — script in root manages restarts and rotates `datagroom.log`.
- **Tests**: `npm test` runs `jest`. Targeted tests available: `npm run test:db`, `npm run test:mongofilters`, `npm run test:api`.
- **Build UI**: The server serves `../datagroom-ui/build`; build the UI with `cd ../datagroom-ui && npm run build` so `server.js` can serve static files.

3. Project-specific conventions & patterns
- **Singleton DB client**: `DbAbstraction` is intentionally a singleton — do not create multiple long-lived clients. Use `new DbAbstraction()` to get the instance (constructor returns existing instance).
- **Graceful shutdown**: `server.js` listens for `SIGINT`/`SIGTERM` to call `DbAbstraction.destroy()` and `DbConnectivityChecker.destroy()`; follow this when adding long-lived resources.
- **Socket message patterns**: Socket events use simple names like `lockReq`, `unlockReq`, `activeLocks`. Locks are tracked in-memory in `server.js`. If you change lock behavior, update both server and the UI.
- **JWT secret**: `Utils.jwtSecret` is used for signing/verifying tokens. Tests and code expect this to be stable across processes.
- **File storage dirs**: `uploads/` and `attachments/` are created at runtime — the server expects them to exist or will create them. When writing code that serves files, use the `attachments` static route configured in `server.js`.

4. Integration points & external dependencies
- **MongoDB**: Connections use `process.env.DATABASE` or default to `mongodb://localhost:27017`. `dbAbstraction.js` listens to many MongoDB client events — changes to the `mongodb` driver version can break event names or lifecycle behavior.
- **LDAP**: `ldapSettings.js` configures `passport-ldapauth`. Production deployments normally enable LDAP; dev often passes `disableAD=true`.
- **JIRA plugin**: `jira/` contains integration code. `server.js` calls `Jira.createFilteredProjectsMetaData()` at startup — be careful when touching startup flows.
- **Socket.IO**: Authentication of sockets happens by parsing cookies in `isAuthorized()` in `server.js`.

5. Tests and test fixtures
- **In-memory MongoDB**: Tests use `mongodb-memory-server` (see `devDependencies`) — good for unit tests that need DB access. Look at `tests/dbAbstraction.test.js` and `setup/dbTestSetup.js` for setup patterns.
- **Test entrypoints**: Use the `npm run test:<name>` scripts for targeted debugging.

6. Typical change patterns & examples
- When adding a new DB helper, follow the style in `dbAbstraction.js`: ensure `await this.connect()` is called, wrap DB calls in try/catch, call `this.handleDbErrors(err)` and rethrow the error.
- When adding a new route, register it in `server.js` and respect the `authenticate` middleware pattern (JWT cookie or basic auth flows). Example: `app.use('/upload', require('./routes/upload'))`.
- When changing socket behavior, update both the event names and the client-side code in `datagroom-ui` (the UI subscribes to `locked`, `unlocked`, `activeLocks`).

7. Files to inspect first for most tasks
- `server.js` — main entrypoint, auth, sockets, and static serving.
- `dbAbstraction.js` — DB patterns and connection lifecycle.
- `routes/dsReadApi.js` — core dataset APIs used heavily by the UI.
- `startDg.sh` — production start script with restart/log rotation behaviour.
- `README.md` — deployment and docker notes useful for environment setup.

8. Safety & testing notes for AI edits
- Avoid changing global process-level behavior (e.g., signal handling) without updating shutdown logic in `server.js`.
- Don't change the singleton pattern in `dbAbstraction.js` to create multiple clients; instead update the class methods to reuse the existing client.
- If updating Mongo driver usage, ensure event listeners and client lifecycle calls map to the driver version in `package.json`.

9. Example prompts you can answer with code changes
- "Add a new REST endpoint to archive records matching {filter} from dataset A to dataset B" — modify `routes/` and use `DbAbstraction.archiveData()`.
- "Make socket auth permissive in dev" — add a `devMode` flag read from process args or env and shortcut `isAuthorized()` accordingly.

If anything in this guide is unclear or missing, ask for repository-specific clarifications and mention which file(s) you want to update next.
