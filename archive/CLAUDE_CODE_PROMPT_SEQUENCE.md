# Claude Code — Prompt Sequence

This is the control panel for driving Claude Code through the conversion. Each prompt is one focused turn. Paste, wait for output, review, then paste the next.

**Cadence:** never paste the next prompt without reading what Claude Code produced for the previous one. If something looks off — a fixture got the wrong shape, an architectural seam got skipped, a guardrail got missed — push back before moving on. Phase 2 reworks are expensive; Phase A reworks are cheap.

---

## Prompt 1 — Orientation (paste first)

The orientation prompt is in `CLAUDE_CODE_KICKOFF_PROMPT.md` (sibling file in this folder). Open it, copy everything below the `---` line, paste into Claude Code as your first message.

Wait for Claude Code's ack. The ack should mention: the conversion goal, the locked stack, the local-Supabase choice, the pass-1 scope boundaries, the four-role / two-workspace model, and the critical guardrails (Job Levels ≠ Tiers, NO_FAULT_STATES intentionally empty, CoB ≠ CoC, centralized `canPerform`, append-only audit log, tenant context). If any of those are missing from the ack, push back: *"Reread `docs/COB_Flow_NextJS_Conversion_Handoff.md` §6 and `docs/COB_Flow_Handoff.md` §5 — your ack didn't mention X. Update the ack."*

When the ack is solid, paste Prompt 2.

---

## Prompt 2 — Greenlight Phase A scaffolding

> Ack looks good. Proceed with Phase A scaffolding, with these specifics:
>
> **1. Scaffold the Next.js app** with `create-next-app@latest` using these flags: TypeScript yes, ESLint yes, Tailwind yes, App Router yes, `src/` directory yes, Turbopack yes, import alias `@/*` yes. Run it in the current directory (don't create a nested subfolder). If `create-next-app` won't run in a non-empty directory because of `CLAUDE.md` / `docs/` / `.git/`, scaffold into a temp dir and move the files in — preserving `CLAUDE.md`, `docs/`, and `.git/`.
>
> **2. Update `CLAUDE.md`** with the project's persistent guardrails. Use the structure: (a) project identity (one paragraph), (b) locked decisions (stack, tenant model, auth posture, Supabase local-instance for pass 1, domain deferred to `cob-flow-app.vercel.app`), (c) architectural principles from `docs/COB_Flow_NextJS_Conversion_Handoff.md` §6 (centralized `canPerform`, append-only audit log with `justification` from day one, single roles helper, tenant context through session, Server Components by default, audit ≠ coaching notes), (d) critical guardrails (Job Levels ≠ Tiers, `NO_FAULT_STATES` intentionally empty for the Wisconsin-only build, CoB ≠ CoC, port the COB primacy engine with no behavioral changes), (e) working norms (PR-style summaries per phase, ask before guessing when prototype and spec disagree, small focused commits). Show me the proposed `CLAUDE.md` diff before committing it.
>
> **3. Configure Tailwind** — set up `tailwind.config.ts` with the brand colors the prototype uses. Open `docs/COB_Flow_MVP.html` and extract the brand tokens (look at the `<style>` block and the Tailwind utility classes used on top-level layout elements — header, sidebar, primary buttons, status pills). Mirror those as `theme.extend.colors` entries with semantic names (e.g., `brand`, `brand-fg`, `surface`, etc.). Pull state-pill colors (case state: GATHERING / READY / ASSIGNED / IN_RECOVERY / CLOSED / REOPENED) as a separate token set.
>
> **4. Stand up the directory structure** per Conversion Handoff §9: `src/app/` (routes already created by `create-next-app`), `src/components/{ui,layout,claim,management,admin,cob,shared}/`, `src/lib/{auth,authority,audit,engine,mock,types,utils}/`, `src/styles/globals.css`, `src/tests/{unit,e2e}/`. Empty placeholder files (`.gitkeep` or stub `index.ts`) are fine — populate in later phases.
>
> **5. Port the prototype fixtures** from `docs/COB_Flow_MVP.html` into `src/lib/mock/`, one file per fixture per Conversion Handoff §7 (look for the line-number references). **Preserve exact shapes and exact values.** Type each one against a corresponding type in `src/lib/types/`. The fixtures to port: `TENANTS`, `SAMPLE_CLAIMS`, `LETTER_TYPES`, `SAMPLE_RECOVERIES`, `JOB_LEVELS`, `TEAMS`, `USERS`, `SUPERVISOR_CEILING`, `FILE_AUTHORITY_GRANTS`, `COACHING_NOTES`, `SUPERVISOR_PERFORMANCE`, `LEVEL_HEALTH_SIGNALS`, `ROLE_LABELS`, `APPROVAL_QUEUE`, `QC_SAMPLES`, `TEAM_WORKLOAD`, `CUSTOMER_KPIS`, `SYSTEM_EVENTS`, `NO_FAULT_STATES` (intentionally empty — preserve as an empty `Set<string>` typed as `Set<USState>` so the engine can still query it).
>
> **6. Supabase is already running locally.** I ran `supabase init` and `supabase start` before this session. `.env.local` is already populated with `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, and `DATABASE_URL`. Verify by running `supabase status` and confirming the stack is up. Commit the `supabase/` directory (config.toml plus any `.gitignore` inside it). Confirm that `.env*.local` is in the repo `.gitignore` after `create-next-app` finishes — if not, add it before committing anything else.
>
> **7. Initialize Drizzle** pointed at the local Supabase Postgres. Install `drizzle-orm`, `drizzle-kit`, `postgres` (or `pg`). Configure `drizzle.config.ts` to read from `DATABASE_URL`. Create `src/lib/db/schema.ts` with a placeholder schema (no tables yet — Phase 2 work). Add `drizzle-kit` commands to `package.json` scripts (`db:generate`, `db:migrate`, `db:studio`).
>
> **8. Stub the architectural seams** as documented in Conversion Handoff §6:
> - `src/lib/auth/session.ts` — exports `getCurrentUser()`, `signIn()`, `signOut()` (returning mock data for pass 1, but with the signatures Phase 2 will fill in)
> - `src/lib/authority/can-perform.ts` — exports `canPerform(user, action, context)` returning `{ allowed: true }` if signed in, in the discriminated-union shape from §6.1
> - `src/lib/audit/log.ts` — exports `auditLog.record(event)` writing to an in-memory array, with the event shape including `justification` from day one
> - `src/lib/authority/roles.ts` — exports `hasRole`, `isAnalyst`, `isSupervisor`, `isManager`, `isAdmin`, `effectiveRoles`
> - `src/lib/engine/` — empty placeholders for `primacy.ts`, `wi-overlay.ts`, `triage.ts` (port in Phase G, but create the files now)
>
> **9. Health checks.** Run `npx tsc --noEmit`, `npm run lint`, `npm run build`. All three must be clean before commit. Fix any errors.
>
> **10. Commit and push** in small, focused commits — at minimum: (a) `chore: scaffold Next.js app`, (b) `chore: configure Tailwind with brand tokens`, (c) `feat: stand up directory structure and architectural seams`, (d) `feat: port prototype fixtures to typed TS modules`, (e) `chore: initialize Supabase local + Drizzle`, (f) `docs: update CLAUDE.md with project guardrails`. Push to `origin/main`.
>
> **11. PR-style summary.** When you finish, summarize: what landed, what decisions you had to make, what wasn't obvious from the docs, anything you want me to look at before Phase B. Include the diff for `CLAUDE.md` in the summary.
>
> Do not start Phase B until I greenlight.

---

## Prompt 3 — Greenlight Phase B (drafted after Phase A review)

Placeholder. After I see Claude Code's Phase A summary, we'll draft Prompt 3 to direct Phase B (mock auth + app shell — landing page, top bar, sidebar with role-conditional nav, the four-state demo role toggle).

The pattern continues:
- Prompt 4 → Phase C (read-only workspaces: Dashboard, Claims & Triage list, Recovery Tracker)
- Prompt 5 → Phase D (Claim Detail with 7 tabs + COB Analyzer)
- Prompt 6 → Phase E (Management workspace)
- Prompt 7 → Phase F (Admin workspace)
- Prompt 8 → Phase G (engine + utility ports with unit tests)
- Prompt 9 → Phase H (QA + polish + acceptance pass)

Each greenlight prompt names the phase, the specific work items from Conversion Handoff §15, and any decisions surfaced by the previous phase's summary.

---

## Review checklist (apply to each Claude Code summary before greenlighting the next prompt)

- Did it produce the listed deliverables, or just claim it did? Spot-check by reading the files it changed.
- Did it preserve the **exact shapes and values** of the prototype fixtures? Diff a few against the prototype.
- Did the guardrails make it into the work (e.g., `canPerform` is centralized; audit log is append-only with `justification`; Server Components are the default)?
- Did `tsc --noEmit`, `npm run lint`, and `npm run build` all pass cleanly? Ask for the output if not shown.
- Are commits small and focused, with sensible messages?
- Are there decisions Claude Code made that should be reflected in `CLAUDE.md` or the master `COB_Flow_Handoff.md`?

When in doubt, ask Claude Code to show specific files or run specific commands and report output. Don't take "done" at face value.
