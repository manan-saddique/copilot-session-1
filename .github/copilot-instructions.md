# Copilot Instructions for This Repository

## Big picture
- This is an npm workspace monorepo with two packages: `packages/frontend` (React app) and `packages/backend` (Express API).
- The product flow is intentionally simple: frontend lists items and posts new items; backend serves and stores them in an in-memory SQLite DB.
- `packages/backend/src/index.js` only starts the server; `packages/backend/src/app.js` defines the app and data layer. Keep this split so tests can import `app` without binding a port.

## Architecture and boundaries
- Backend API surface is currently:
  - `GET /api/items` → returns `[{ id, name, created_at }]`
  - `POST /api/items` with `{ name }` → validates non-empty string and returns created row.
- Backend uses `better-sqlite3` with `new Database(':memory:')` and seeds `Item 1..3` at module load in `packages/backend/src/app.js`.
- Frontend consumes the API with `fetch('/api/items')` and `fetch('/api/items', { method: 'POST' })` in `packages/frontend/src/App.js`.
- Cross-package communication in dev relies on CRA proxy (`"proxy": "http://localhost:3030"`) in `packages/frontend/package.json`; keep API calls relative (`/api/...`) unless proxy strategy changes.

## Developer workflows
- Install dependencies from repo root: `npm install` (workspace-aware).
- Run both apps from root: `npm run start`.
- Run one side only:
  - Frontend: `npm run start:frontend`
  - Backend: `npm run start:backend`
- Test from root:
  - All: `npm test`
  - Frontend only: `npm run test:frontend`
  - Backend only: `npm run test:backend`

## Testing patterns to follow
- Backend tests (`packages/backend/__tests__/app.test.js`) use `supertest` against imported `app` and close shared `db` in `afterAll`.
- Frontend tests (`packages/frontend/src/__tests__/App.test.js`) use React Testing Library + MSW (`rest.get('/api/items')`, `rest.post('/api/items')`) to mock network.
- When adding API fields/endpoints, update both backend supertest assertions and frontend MSW handlers so tests reflect the same contract.

## Project-specific conventions
- Use CommonJS in backend (`require/module.exports`) and ES modules in frontend (`import/export`).
- Keep backend route-level try/catch style and JSON error shape (`{ error: '...' }`) consistent with existing handlers.
- Keep frontend state flow consistent with `App.js`: `loading`, `error`, list state, and optimistic append after successful POST response.
- Prefer minimal, focused changes in this training repo; avoid introducing extra layers/frameworks unless required by the task.