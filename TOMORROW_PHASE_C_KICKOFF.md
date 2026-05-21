# Tomorrow — Phase C kickoff

**As of:** end of day 2026-05-20
**Status:** Phase B fully closed (CP1 → CP4 + mobile polish, all pushed to `origin/main` at `79cc377`)
**Tomorrow's first move:** kick off Phase C planning with Claude Code in the cob-flow-app repo.

---

## Where we left off

Phase B shipped a working auth + app shell in the Next.js app. Sign-in page, mock cookie auth, four-state role toggle, tenant dropdown, role-gated sidebar nav (desktop + mobile sheet), account menu with sign-out and "Switch role (demo)", boundary pages (`error.tsx`, `not-found.tsx`, `global-error.tsx`). 26 unit tests + 8 Playwright E2E tests; build/typecheck/lint clean. Mobile polish landed in three follow-up commits (`9b314fb`, `597b275`, `79cc377`): compact top bar at `< sm`, wordmark restored next to logo at smaller font, role toggle relocated into the account menu on mobile, sidebar sheet backdrop deepened to `bg-black/40`.

The full Phase B summary is in `cob-flow-app/docs/COB_Flow_Handoff.md` § 11.

## Decisions locked today

- **Phase C scope (corrected from Claude Code's initial handoff drift):** three workspaces only — **Dashboard, Claims & Triage list, Recovery Tracker**. NOT COB Analyzer, NOT Claim Detail tabs, NOT Management, NOT Admin. Those are Phases D / E / F respectively. The small-checkpoint cadence that made Phase B reviewable depends on respecting this boundary.

- **Schema timing — Option 1 (stay fully fixture-based through pass 1).** Phase C reads from `src/lib/mock/`. No real schema work in Phase C. Reason: the seam is well-understood, proving it on one table doesn't tell us much about how it'll handle Claim/Case/Recovery, and real schema design is better done after GTM Phase 1 customer discovery surfaces actual claim feed shapes. Drizzle stays wired; `db/schema.ts` stays a placeholder.

- **Working pattern:** keep using the section-by-section design review approach that worked for Phase B. Claude Code proposes the design in 3–4 sections; Cowork (this surface) reviews each before code lands; then the spec gets committed to `docs/superpowers/specs/` and the implementation plan to `docs/superpowers/plans/` before any execution.

## Working norms now established

- **Reply prompts go in fenced code blocks** so the chat UI exposes a copy button.
- **Push to `origin/main` at the end of every checkpoint**, not just end of phase. CP summary includes the remote HEAD hash.
- **Commits stay small and focused.** Eight readable commits per checkpoint beat one giant CP commit.
- **PR-style summaries at end of phase** with deviation log + what next phase inherits.
- **Spec deviations get documented in the spec doc itself** as the implementation discovers them — don't let the spec drift from what's built.

## First action tomorrow

Open Claude Code in `~/Documents/Claude/Projects/cob-flow-app/`. Paste this:

```
Good morning. Phase B is fully closed and pushed (main at 79cc377, including mobile polish commits 9b314fb / 597b275 / 79cc377). Ready to start Phase C planning.

Phase C scope per Conversion Handoff §15: three read-only workspaces — Dashboard, Claims & Triage list, Recovery Tracker. NOT COB Analyzer, NOT Claim Detail tabs, NOT Management, NOT Admin (those are D / E / F). Schema-timing decision is locked: stay fully fixture-based through pass 1, read from src/lib/mock/, no Drizzle table definitions in Phase C.

Before writing any spec: read these and ack with one paragraph confirming you understand scope + schema decision + what's already in place from Phase B that Phase C will leverage:
1. docs/COB_Flow_Handoff.md § 11 (full phase log, Phase B closure, Phase C scope & schema decision)
2. docs/superpowers/specs/2026-05-20-phase-b-auth-app-shell-design.md (the working patterns that Phase C should follow: Server Components by default, revalidatePath flow, AppShellContext, canPerform stub, role gating via lib/authority/roles.ts)
3. docs/COB_Flow_NextJS_Conversion_Handoff.md §15 (Phase C deliverables specifically)
4. docs/COB_Flow_Product_Spec_v0.8.docx — Dashboard widgets per §7, Claims & Triage list per §7, Recovery Tracker per §7
5. COB_Flow_MVP.html prototype — open in browser, sign in, click through Dashboard, Claims & Triage, Recovery Tracker to see the layout and content the conversion needs to match

Then propose the Phase C design in 3–4 sections (route structure, fixture access pattern, per-workspace component design, acceptance criteria) and walk me through them one at a time. Don't start the spec write-up until I've greenlighted each section.
```

After Claude Code's ack, work through the section-by-section design review the same way we did Phase B.

## Open items for later (not Phase C)

- **Pilot tenant identity** still unresolved (Common Ground Healthcare Cooperative vs. Diversified Benefit Services vs. ProHealth Care vs. other). Phase 0 GTM work.
- **GTM Phase 1 customer discovery** has not started. SPD Review Template + Onboarding Playbook are the front-door instruments when ready.
- **Real schema design** waits for pass 2, gated on customer discovery and the HIPAA cost cliff (~$3.5–4k/mo for Vercel Enterprise BAA + Supabase HIPAA add-on).
- **Cob-flow-app deployed to Vercel** — repo is at github.com/jimmuell/cob-flow-app, not yet imported into Vercel for hosting. Wait until at least end of Phase C or when first shareable demo is needed.
