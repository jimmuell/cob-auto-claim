# Claude Code — Kickoff Prompt

> **How to use this file.** Open Claude Code in `~/Documents/Claude/Projects/cob-flow-app/` (cloned from `https://github.com/jimmuell/cob-flow-app.git`). Paste everything below the `---` line as your first message. After Claude Code finishes reading the docs and confirms what it understood, the build begins.

---

I'm Jim Mueller — healthcare subrogation SME, founder of COB Flow. You're going to convert a single-file HTML React prototype into a real Next.js application. This is pre-revenue, pre-pilot work; nothing in this pass touches real PHI or real customers.

## Step 1 — Read these in order, before doing anything

Everything you need to understand the project lives in `docs/` in this repo. Start with the README:

1. `docs/README.md` — reading order + conflict-resolution rule
2. `docs/COB_Flow_NextJS_Conversion_Handoff.md` — **your build brief.** Tech stack (§5), architectural principles (§6), ingest architecture (§11), build phases (§15), acceptance criteria (§16), open decisions (§17), helper TypeScript signatures + env-var list + README skeleton in the appendices (§20).
3. `docs/COB_Flow_Handoff.md` — project context, guardrails (§9), likely next workstreams.
4. `docs/COB_Flow_Product_Spec_v0.8.docx` — canonical product spec. Minimum read: §§3, 6, 8, 9, 10, 11, 14, and Appendix A glossary.
5. `docs/COB_Flow_Dashboard_Spec_v0.1.docx` — technical companion. Read §§3–8.
6. `docs/COB_Flow_WI_Workflow_v1.0.docx` — the 9-phase analyst workflow.
7. `docs/COB_Flow_MVP.html` — the prototype. Open it in a browser; sign in via the demo-accounts picker. This is the authoritative reference for layout, copy, component composition, and interaction.

When you're done, give me a short ack: one paragraph confirming you understood the conversion goal, the locked stack, the pass-1 scope boundaries, and any questions you have before scaffolding. Don't start coding until I've replied.

## Step 2 — Pass-1 decisions already locked

These are settled — don't relitigate without a specific reason from something you read in the docs:

- **Stack:** Next.js 15 (App Router) · TypeScript strict · Tailwind · shadcn/ui · lucide-react · React Hook Form + Zod · TanStack Table · Supabase (auth + Postgres + storage) · Drizzle ORM · Vercel · Sentry + Vercel Analytics added later · Vitest + Playwright. Full table in Conversion Handoff §5.
- **Repo:** `https://github.com/jimmuell/cob-flow-app.git` (private). Already cloned to `~/Documents/Claude/Projects/cob-flow-app/`. You're sitting in it.
- **Node:** v24.x. (Confirmed on the local machine. Node 20 went EOL April 30, 2026; we're using the current release line.)
- **`CLAUDE.md` exists** in the repo root from `/init`. Read what's there. After you've read the docs in Step 1, propose an updated `CLAUDE.md` that captures the locked decisions, the architectural principles from Conversion Handoff §6, and the critical guardrails in Step 4 below — so they persist across sessions. Show me the diff before committing.
- **Supabase: local instance for pass 1.** I have Docker Desktop and the Supabase CLI already installed. Use `supabase init` + `supabase start` for the dev environment, not a hosted Supabase project. The local API URL is `http://127.0.0.1:54321`; local Postgres at `127.0.0.1:54322`. The CLI will print the anon and service-role keys after `supabase start` — capture those into `.env.local`. When a hosted Supabase project is created later (first shared deploy), `supabase link --project-ref <ref>` will port the migrations up.
- **Tenant model:** single-tenant deployment for COB Flow Recovery in pass 1, with multi-tenant seams preserved underneath. The Customer dropdown in the top bar is a demo affordance with dummy data; not real tenants.
- **Auth pass 1:** mock — cookie-backed user-id session, demo-accounts picker on the landing page. Same posture as the prototype. The Supabase Auth boundary is wired but not yet exercised.
- **Domain:** deferred to `cob-flow-app.vercel.app` (Vercel auto-generated). No custom domain in pass 1.
- **Hosting account:** I have a Vercel account already. Don't import the repo yet — scaffold first, then I'll connect.
- **Folder layout:** the app lives in `cob-flow-app/`. The `docs/` folder you just read stays in-repo as the canonical reference.

## Step 3 — Pass-1 scope boundaries (Conversion Handoff §4)

**In scope:** functional parity with the prototype — sign-in landing, four-role conditional UI, all current workspaces (Dashboard, Claims & Triage, COB Analyzer, Recovery Tracker, Management, Admin), Claim Detail with 7 tabs, role-conditional sidebar/nav, the four-state demo role toggle in the top bar, the COB primacy decision engine ported to TypeScript, mobile responsiveness preserved.

**Explicitly out of scope:** real auth providers, real database connections to anything other than local Supabase, real backend integrations (X12 / SFTP / API), LLM-powered document parsing, real email/fax/SMS sending, HIPAA-grade audit, anomaly detection, Profile/Settings pages, marketing content.

## Step 4 — Critical guardrails (don't miss these)

- **The COB primacy decision engine is the product's core.** Port it from the prototype's JS into `lib/engine/primacy.ts` **with no behavioral changes** in pass 1. Unit-test each rule.
- **Wisconsin-only build.** The engine's `NO_FAULT_STATES` set is **intentionally empty**. Do not "fix" it.
- **Job Levels are NOT Tiers.** Four levels on the Analyst role: Trainee / Junior / Mid / Senior. "Tier" is reserved for `LETTER_TYPES` correspondence priority (Tier 1 / Tier 2 letter categories) — different concept; don't rename either.
- **Coordination of Benefits (COB) ≠ Coordination of Care (CoC).** Always disambiguate. Never write the unqualified word "coordination."
- **Centralized authority helper.** Every state-changing action routes through `canPerform(user, action, context)` in `lib/authority/can-perform.ts`. In pass 1 it returns `{ allowed: true }` if the user is signed in. The shape that Phase 2 needs is documented in Conversion Handoff §6.1 — implement that shape now even if the logic is stubbed.
- **Append-only audit log from day one.** Every meaningful state change calls `auditLog.record(event)` with `{ actor, action, target, timestamp, justification?, metadata }`. Pass 1 writes to an in-memory array; the `justification` column is in the schema from the start (nullable in pass 1). Never mutate existing rows.
- **Audit log is NOT coaching notes.** Two separate stores. Coaching notes are role-private; see Dashboard Spec §6.3.
- **Tenant context plumbed through session,** not as free-floating component props. Even in pass 1.
- **Server Components by default.** `"use client"` only for interactivity.
- **Preserve the exact shape and values of the prototype's seeded data.** Move them to typed fixtures in `lib/mock/`; one file per fixture or grouped logically.
- **When the spec and the prototype disagree, ask me.** Do not guess.

## Step 5 — First action: Phase A scaffolding (Conversion Handoff §15)

Once you've acked Step 1, proceed to Phase A:

- `create-next-app` with TypeScript, Tailwind, App Router, ESLint, the `src/` directory layout.
- Configure `tailwind.config.ts` with the brand colors used in the prototype.
- Initialize Drizzle (`drizzle-kit`) pointed at the local Supabase Postgres URL.
- Run `supabase init` to create the `supabase/` directory; check it into the repo.
- Set up `lib/types/`, `lib/mock/`, `lib/auth/`, `lib/authority/`, `lib/audit/`, `lib/engine/`, `lib/utils/` skeleton files (empty stubs are fine; populate in later phases).
- Port the prototype fixtures from JS objects to TypeScript files in `lib/mock/` (preserve exact shapes and values).
- Run `tsc --noEmit` and confirm it's clean. Run `npm run lint`. Run `npm run build`.
- Commit and push.

After Phase A is green, send me a summary of what landed and any decisions you had to make. I'll review before you start Phase B.

## Step 6 — Working norms

- **Domain language is the right register.** COB primacy, made-whole doctrine, recovery cycle, lien reduction, payer hierarchy, ERISA preemption, MSP working aged, etc. Use it when explaining decisions.
- **Check in on architectural decisions before committing them.** Anything that touches `canPerform`, the audit log shape, the data model, the role/authority logic, or the multi-tenant seams gets reviewed before merge.
- **Small, focused commits.** I'd rather review eight readable commits than one giant one.
- **PR-style summaries when you finish a phase.** What changed, what decisions you made, what's still open.
- **The handoff doc gets updated as we go.** When meaningful decisions land, propose an edit to `docs/COB_Flow_Handoff.md` so the next session inherits the new state.

Go. Read the docs first; ack with the summary; wait for my green light; then start Phase A.
