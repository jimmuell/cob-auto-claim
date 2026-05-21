# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

> Working model: see `DIVISION_OF_LABOR.md` at the repo root for how Jim, Cowork (the planning layer in Claude.ai desktop), and the Claude Code agents (the per-repo execution layer) collaborate.

This is the **planning and documentation repository** for COB Flow — a pre-revenue healthcare SaaS for coordination of benefits (COB) auto-claim processing. It is NOT the application code repo.

- This repo: local at `~/Documents/Claude/Projects/cob-auto-claim/`, GitHub `https://github.com/jimmuell/cob-auto-claim.git` (private).
- App code: local at `~/Documents/Claude/Projects/cob-flow-app/`, GitHub `https://github.com/jimmuell/cob-flow-app.git` (private).

When Jim asks you to build features, run tests, or work on the app, switch to `cob-flow-app/`. This directory is for authoring specs, handoffs, and prototype artifacts.

## Project context

COB Flow automates healthcare coordination of benefits primacy determination, auto med-pay/PIP recovery, and post-payment subrogation. Wisconsin is the pilot state (tort/at-fault; made-whole doctrine applies). The regulatory anchor is **Wis. Admin. Code § Ins 3.40**; § Ins 3.40(18) (coordination with noncomplying plans) is the explicit authority for the recovery workflow.

The product is **decision-support in v1** — the engine recommends, the analyst signs. Four roles: Analyst, Supervisor, Manager, Admin. Analysts also carry a Job Level (Trainee / Junior / Mid / Senior) that drives authority defaults.

## Document reading order

When starting work that touches product behavior or architecture, read these in order:

1. `cob-flow-app/docs/COB_Flow_NextJS_Conversion_Handoff.md` — the conversion brief. Locked tech stack (§5), architectural principles (§6), ingest architecture (§11), build phases (§15).
2. `cob-flow-app/docs/COB_Flow_Handoff.md` — full project context, guardrails (§9), and likely next workstreams (§10).
3. `cob-flow-app/docs/COB_Flow_Product_Spec_v0.8.docx` — canonical product spec. Minimum read: §§3, 6, 8, 9, 10, 11, 14, Appendix A.
4. `cob-flow-app/docs/COB_Flow_Dashboard_Spec_v0.1.docx` — role/authority architecture, approval queue types, audit architecture. Read §§3–8.
5. `cob-flow-app/docs/COB_Flow_WI_Workflow_v1.0.docx` — the 9-phase analyst workflow.
6. `cob-flow-app/docs/COB_Flow_MVP.html` — **authoritative reference** for layout, copy, component composition, and interaction. Open in a browser; sign in via the demo-account picker.

Conflict resolution: companion docs (#3–#6) win on product behavior; Conversion Handoff wins on engineering structure. When prototype and spec disagree, ask Jim — never guess.

## Current phase status (as of 2026-05-21)

- **Phase A** (scaffolding): files written locally during pass 1 setup but never committed; recovered to `origin/main` in Phase B.1 (see `cob-flow-app/docs/COB_Flow_Handoff.md` § 11 → "Phase B.1 — Scaffolding recovery" for the recovery commit log).
- **Phase B** (auth + app shell): complete (CP1–CP4 + mobile polish).
- **Phase B.1** (Phase A scaffolding recovery): complete — `origin/main` at `aace942`.
- **Phase C** (read-only workspaces: Dashboard, Claims & Triage list, Recovery Tracker): planning in progress. Schema stays fixture-based through all of pass 1 — no Drizzle table definitions in Phase C. See the live Phase C kickoff in the active Cowork session (not in this repo).
- **Phases D–H**: Claim Detail + COB Analyzer (D), Management (E), Admin (F), engine + utility ports with unit tests (G), QA + polish (H).

## Locked tech stack

Next.js 15 · App Router · TypeScript strict · React 19 · Tailwind CSS · shadcn/ui · lucide-react · React Hook Form + Zod · TanStack Table · Supabase (auth + Postgres + storage, local instance for pass 1) · Drizzle ORM · Vercel · Vitest + Playwright · ESLint + Prettier · GitHub Actions CI.

Pass 1 auth is mock (cookie-backed user-id session, demo-accounts picker). Real Supabase Auth wires in during pass 2.

## Architectural guardrails (non-negotiable in all phases)

- **`canPerform(user, action, context)`** — every state-changing action routes through `lib/authority/can-perform.ts`. Returns a discriminated union: `{ allowed }`, `{ requiresApproval, approverRole, queueType }`, or `{ denied, reason }`. Pass 1 stub returns `{ allowed: true }` if signed in, but the full shape must be there from day one.
- **Append-only audit log** — `auditLog.record(event)` with `{ actor, action, target, timestamp, justification?, metadata }`. `justification` column exists from day one (nullable). Never mutate existing rows.
- **Single roles helper** — `lib/authority/roles.ts` exports `hasRole`, `isAnalyst`, `isSupervisor`, `isManager`, `isAdmin`, `effectiveRoles`. No scattered string comparisons.
- **Tenant context through session**, not component props. Even in pass 1.
- **Server Components by default.** `"use client"` only for interactivity.
- **Audit log ≠ coaching notes.** Two separate stores with separate visibility rules.

## Critical terminology and guardrails

- **Job Levels ≠ Tiers.** Four Job Levels (Trainee / Junior / Mid / Senior) on the Analyst role. "Tier" is reserved for `LETTER_TYPES` correspondence priority (Tier 1 / Tier 2) — a different concept entirely. Never rename either.
- **`NO_FAULT_STATES` is intentionally empty.** Wisconsin-only build; the mandatory no-fault PIP rule never fires. Do not "fix" it.
- **"Coordination of Benefits" (COB) ≠ "Coordination of Care" (CoC).** Always use the qualified form. Never write bare "coordination."
- **Management workspace ≠ Admin role.** Management (Supervisor + Manager — people work) and Admin (Admin role — system work) are explicitly different sidebar items.
- **"Customer" everywhere in UI**, not "Tenant."
- **The COB primacy engine is the product's core.** Port it to `lib/engine/primacy.ts` with no behavioral changes. Unit-test every rule.
- **Prototype is the authoritative UI reference.** When spec and prototype disagree, ask Jim.
- **Companion docs win on product behavior.** Conversion Handoff wins on engineering structure.

## Working norms

- Propose design in 3–4 sections; Jim reviews each before code lands.
- Commit spec to `cob-flow-app/docs/superpowers/specs/` and implementation plan to `cob-flow-app/docs/superpowers/plans/` before executing a phase.
- Push to `origin/main` at the end of every checkpoint. Include the remote HEAD hash in phase summaries.
- Small focused commits — eight readable commits beat one giant one.
- PR-style summaries at end of each phase: what landed, decisions made, what's open, what next phase inherits.
- Propose updates to `cob-flow-app/docs/COB_Flow_Handoff.md` when meaningful decisions land — the next session inherits what's in it.
- Reply prompts go in fenced code blocks (exposes a copy button in the chat UI).
- Use domain language: COB primacy, made-whole doctrine, recovery cycle, lien reduction, payer hierarchy, ERISA preemption, MSP working aged. Jim thinks in these terms and expects responses framed in them.
