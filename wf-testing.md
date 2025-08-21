## QA and Testing Strategy: Playable Character Cards

Purpose: A concise, actionable overview for QA leadership and AI agents covering
strategy, ownership, workflows, gates, and triage across CI/CD and local
testing. This consolidates and refines our current documentation from
`TESTING.md`, `DEVELOPMENT.md`, `ARCHITECTURE.md`, and `tests/`.

---

## 1) Executive Summary

- **Three-tier testing model**
  - **Tier 1: CI‑Lite** (fast, automated, no Docker) — runs on every push/PR,
    finishes in ~60 seconds, blocks merges.
  - **Tier 2: Local Integration** (comprehensive, manual) — real API, DB, and AI
    calls against a local Supabase stack.
  - **Tier 3: Browser** (on‑demand, manual) — Browserbase MCP UI flows, not part
    of CI‑Lite.
- **Real API Testing**: We never mock AI model outputs. Tests use real services
  (Imagen, Gemini) and our production‑grade code paths.
- **Strict quality gates**: CI‑Lite must be green to merge. Security and
  documentation standards are continuously verified.

References

- Strategy and commands: `TESTING.md`
- Dev workflows and scripts: `DEVELOPMENT.md`
- Architectural principles: `ARCHITECTURE.md`
- Test suites: `tests/`

---

## 2) Roles, Ownership, and When Tests Are Written

- **Head of QA**
  - Defines acceptance criteria and test strategy evolution.
  - Owns regression suite scope, flakiness reviews, and release go/no‑go
    criteria.
  - Curates Tier 3 browser scenarios and ensures environment isolation is
    respected.

- **System Architect (`system-architect.mdc`)**
  - Plans features and explicitly includes testing and validation steps.
  - Ensures security and architectural checks are enforced in CI‑Lite.

- **Feature Developer (`feature-dev.mdc`)**
  - Writes unit tests for new logic (`tests/unit/`).
  - Writes integration tests for new endpoints and flows (`tests/integration/`).
  - Keeps `supabase/config.toml` up to date for all functions.

- **Refactoring Engineer**
  - Ensures all existing tests remain green during code changes.
  - Adds or updates regression tests when historical issues are touched.

- **Docs Writer**
  - Maintains code‑first, semantic anchor‑based documentation.

When tests are written

- During feature implementation (Definition of Done includes tests and CI‑Lite
  green).
- Immediately after fixing a defect (add a regression test).

---

## 3) Test Suite Organization and Purposes

Directory map

- `tests/unit/`: Fast unit tests executed in CI‑Lite.
- `tests/integration/`: Real API + DB + AI flows (local Supabase required).
- `tests/regression/`: Locks in known fixes (expand continuously).
- `tests/browser/`: Browserbase MCP tests for Admin UI (on‑demand).
- `tests/utils/`: Shared testing utilities (clients, validators, perf).
- `tests/fixtures/`: Typed test data.

Key authentication patterns

- **Custom admin Bearer token**: `Authorization: Bearer ${ADMIN_SECRET_KEY}` for
  admin Edge Functions.
- **Service role key**: `SUPABASE_SERVICE_ROLE_KEY` only for database admin
  operations.
- These are distinct systems and must never be mixed.

Environment isolation

- Tests force local base URLs like `http://127.0.0.1:45321` to avoid production.
- Secrets are read from `.env`; tests skip or fail fast if required vars are
  missing.

---

## 4) Tier 1 — CI‑Lite (Automated Frontline)

Trigger: every push and PR. Workflow: `.github/workflows/ci-lite.yml`.

Primary checks (scoped for speed and relevance)

- ESLint (Node/TS) — scoped to `supabase/functions/`, `tests/unit/`,
  `tests/utils/test_helpers.ts`.
- Prettier — repository‑wide formatting check.
- Deno lint — scoped to Deno sources only.
- TypeScript check — run from `./supabase/admin_page`.
- Supabase configuration validation — ensures each Edge Function is listed with
  `verify_jwt = false` to avoid “Invalid JWT”.
- Documentation validation — semantic anchors enforced.
- Test configuration validation — ports and lockfile sanity.
- Private endpoint security verification — admin endpoints excluded from public
  spec; generated spec filters public visibility.
- Unit tests — quick pass/fail signal.

Triage (use step names from logs)

- ESLint: add cross‑runtime globals; keep scope tight.
- Prettier: run `npm run fix` and maintain `.prettierignore` for generated/noisy
  paths.
- Deno lint: lint only Deno‑relevant paths; exclude Node‑only code.
- TypeScript: ensure `working-directory: ./supabase/admin_page`.
- Security verification: ensure spec generation and metadata exclude admin
  endpoints and include visibility checks.
- Config: add missing functions to `supabase/config.toml` with
  `verify_jwt = false`.

Fetching CI logs (GitHub CLI)

- Use `gh run list` and `gh run view <id> --log` as documented in `TESTING.md`
  (section 2.1).

Merge policy

- CI‑Lite must be green for merges (branch protection recommended).

---

## 5) Tier 2 — Local Integration Testing (Comprehensive Gate)

Purpose: Validate end‑to‑end behavior using real services and local Supabase.

Pre‑requisites

- `.env` file with at least `ADMIN_SECRET_KEY`, `GCP_API_KEY`.
- Local Supabase running: `npm run start:supabase` and `npm run dev:edge`.

Run tests

- All integration tests: `npm run test:integration` or `npm test`.
- Individual suites: see scripts in `DEVELOPMENT.md` and commands in
  `TESTING.md`.

Common pitfalls (and fixes)

- “Invalid JWT”: missing entry in `supabase/config.toml` for the function → add
  with `verify_jwt = false`.
- Storage deletion: use mock public URLs for delete paths; validate behavior,
  not object existence.
- Wrong auth: never use service role key for Edge Function calls; admin
  functions require `ADMIN_SECRET_KEY`.
- Port mismatches: validate with `npm run validate:test-config`.

---

## 6) Tier 3 — Browser Testing (On‑Demand E2E)

Purpose: Validate Admin UI flows and visual health via Browserbase MCP.

Status

- Not part of CI‑Lite; run on demand. Follows environment‑isolation rules (UI
  must target local Supabase).

Typical flows

- Admin UI smoke (load, no console errors).
- Create card end‑to‑end via UI.
- Chat workflow and goal achievement checks.
- Visual regression snapshots.

Setup and commands

- See `TESTING.md` section 4.x and `DEVELOPMENT.md` BrowserBase scripts.

---

## 7) Failure Handling and QA Triage Workflow

When CI‑Lite fails

1. Fetch logs with GitHub CLI (no copy/paste). Identify the failing step by
   name.
2. Apply the targeted fix locally (lint, format, security, TypeScript, config).
3. Re‑run the specific command locally to verify.
4. Commit and push; CI‑Lite re‑runs automatically.

When local integration fails

1. Follow the Quick Debugging Checklist in `TESTING.md`.
2. Confirm Supabase is running and reachable; verify function config; ensure
   correct auth keys.
3. Re‑run the specific failing test; add regression coverage if the bug was new.

Escalation and responsibilities

- The committer (or owning agent) triages and fixes.
- Head of QA monitors recurring failures, flakiness, and documentation gaps.
- System Architect updates quality gates and CI checks as needed.

---

## 8) Quality Gates, Go/No‑Go, and Metrics

Pre‑merge gate

- CI‑Lite green (all steps pass).

Pre‑release gate (for milestones)

- Integration suite green locally on a clean environment.
- Security verification passes (admin endpoints excluded, public filtering
  enforced).
- No open critical regressions in `tests/regression/`.

Suggested QA metrics (track in backlog)

- CI‑Lite time to green (target: ≤60s median).
- CI‑Lite pass rate on main/dev (target: ≥95%).
- Flakiness rate for integration/browser tests (target: ≤2%).
- Mean time to resolution from CI‑Lite failure to green (target: ≤1 hour during
  office hours).

---

## 9) Security and Authentication in Tests

- Admin functions: require `Authorization: Bearer ${ADMIN_SECRET_KEY}` (from
  environment).
- Database admin operations: use `SUPABASE_SERVICE_ROLE_KEY` (different system;
  never for Edge Function auth).
- Environment isolation: tests must force local URLs and never hit production
  endpoints.
- Secret handling: defined in `.env` locally and in CI environment variables;
  never hardcode real production secrets in code or docs.

---

## 10) Continuous Improvement Backlog

- Modularize `TESTING.md` into `docs/testing/` with focused guides (CI‑CD, local
  integration, troubleshooting cookbook).
- Expand `tests/regression/` continuously as defects are discovered and fixed.
- Evaluate safe automation for portions of Tier 2 in a controlled environment.
- Maintain semantic anchor patterns across documentation to keep it code‑first
  and robust to change.

See `backlogs/04_workflow_and_testing.md` (e.g., `SYS‑003`) for the detailed
plan and phased execution.

### Planned initiatives — What and Why

- **SYS‑002: Automated Project Health Check & Quality Gate System**
  - **What (concise)**: Two‑layer health system: fast CI checks + a single
    deep‑scan script (`scripts/health-checks/run-architecture-audit.js`) using
    Gemini CLI; add a non‑blocking "Architecture Audit" job in CI‑Lite that
    uploads `HEALTH_REPORT.md`.
  - **Why**: Catch architectural drift and doc/quality regressions early, shift
    from reactive fixes to proactive maintenance, and reduce merge risk without
    slowing merges.

- **SYS‑003: TESTING.md Restructure for AI Agent Efficiency**
  - **What (concise)**: Split `TESTING.md` into a hub + 4 focused guides
    (`ci-cd-guide`, `local-development-guide`, `troubleshooting-cookbook`,
    `browser-testing-guide`); enforce cookbook schema (Error, Symptom, Root
    Cause, Solution, Verification); add governance/validation; migrate via safe
    2‑PR rollout.
  - **Why**: Improve findability and reduce cognitive load for AI agents;
    enforce code‑first semantic anchors; ensure maintainable, query‑friendly
    docs.

---

## 11) Command Appendix (Quick Reference)

Local setup and development

```bash
npm install
npm run start:supabase
npm run dev:edge
```

CI‑Lite local equivalents

```bash
npm run lint
npm run format
deno lint
cd supabase/admin_page && npx tsc --noEmit
npm run validate:test-config
npm run verify:private-endpoints
npm run test:unit
```

Integration tests (local)

```bash
npm test
npm run test:integration
# examples
npm run test:create-card
npm run test:delete-card
npm run test:enhance-description
```

GitHub CLI (logs and runs)

```powershell
$env:GH_PAGER=""; $repo=(gh repo view --json nameWithOwner -q .nameWithOwner)
gh run list --repo $repo -w 'CI-Lite (Static Analysis, Config & Unit Tests)' --limit 5
$runId=(gh run list --repo $repo -w 'CI-Lite (Static Analysis, Config & Unit Tests)' --limit 1 --json databaseId -q '.[0].databaseId')
if ($runId) { gh run view $runId --repo $repo --log }
```

---

## 12) Pointers to Source of Truth

- `TESTING.md` — deep procedure details and troubleshooting patterns.
- `DEVELOPMENT.md` — complete development commands and environment setup.
- `ARCHITECTURE.md` — principles, constraints, and endpoint inventory.
- `tests/` — actual test implementations (integration, regression, unit, utils,
  fixtures).
