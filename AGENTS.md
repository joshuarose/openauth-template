# Repository Guidelines

## Project Structure & Module Organization
- `src/index.ts`: Cloudflare Worker entrypoint configuring the OpenAuth issuer, password provider UI, and D1-backed user lookup/creation.
- `migrations/*.sql`: D1 schema migrations; apply in order when the schema changes.
- `wrangler.json`: Worker name, entry, KV and D1 bindings, compatibility flags, and observability defaults.
- `worker-configuration.d.ts`: Wrangler-generated types for `Env`; regenerate with `npm run cf-typegen`.
- `README.md`: Setup walkthrough for creating D1/KV resources and deploying.

## Build, Test, and Development Commands
- `npm install`: Install dependencies.
- `npm run dev`: `wrangler dev` for local Worker preview; uses current `wrangler.json` bindings.
- `npm run check`: Type-checks with `tsc` and runs `wrangler deploy --dry-run` to catch deploy-time issues.
- `npm run deploy`: Deploys to Cloudflare (ensure `wrangler.json` bindings are valid).
- `npm run predeploy`: Applies D1 migrations to the remote binding named `AUTH_DB` before deployment.
- `npx wrangler d1 migrations apply --local openauth-template-auth-db`: Run migrations against a local D1 instance when iterating.
- `npx wrangler tail`: Stream Worker logs to verify flows like code delivery.

## Coding Style & Naming Conventions
- Language: TypeScript with `"strict": true`; prefer typed helpers from OpenAuth and `valibot` schemas.
- Formatting: Follow existing tab indentation in `src/index.ts`; keep imports sorted by package path and group related config blocks.
- Naming: Use clear binding names that mirror `wrangler.json` (e.g., `AUTH_STORAGE`, `AUTH_DB`); keep subject keys (`user`) and schema fields (`id`, `email`) consistent across clients.
- Avoid committing generated secrets or production IDs; prefer `.dev.vars`/dash-separated environment files managed outside VCS.

## Testing Guidelines
- There is no dedicated test harness yet; rely on `npm run check` plus manual flows in `wrangler dev`.
- When adding logic, favor small pure helpers in `src/` that can be locally unit-tested (Vitest is a good default if introduced).
- For auth flows, exercise `/` → `/authorize` → `/callback` in preview and confirm D1 writes via `AUTH_DB.prepare(...).all()` or `wrangler d1 execute`.
- Document any new test commands in `package.json` scripts to keep the surface consistent.

## Commit & Pull Request Guidelines
- Current history is sparse; use concise, imperative summaries (e.g., `Add password copy tweak`) and consider Conventional Commit prefixes for clarity.
- In PRs include: purpose, notable config changes (KV/D1 bindings, compatibility flags), migration notes, and screenshots/log snippets if UI or flow behavior changed.
- Ensure migrations are committed alongside schema-dependent code; note whether they were applied locally or remotely.
- Before opening a PR, run `npm run check` and, when relevant, a full local auth flow to catch integration regressions.
