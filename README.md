# COB Auto Claims — Planning & Documentation

This repository is the planning, business, and reference documentation home for **COB Flow** — a healthcare subrogation SaaS for coordination of benefits, auto med-pay/PIP recovery, and post-payment subrogation.

The Next.js application code lives in a separate repo: [cob-flow-app](https://github.com/jimmuell/cob-flow-app.git) (private).

## What lives here

### strategy/
The why and what — product thesis, GTM playbook, and the research that informed both.
- `COB_Auto_Claims_App_Strategy.docx` — original strategic plan and product thesis
- `COB_Flow_GTM_Roadmap.docx` — go-to-market plan
- `COB_Research_Resources_and_Links.docx` — research index
- `COB_Health_Insurance_Courses_Research.pdf` — research notes on COB training landscape and strategic opportunities

### gtm/
Customer-facing instruments used during sales, discovery, and onboarding.
- `COB_Flow_Target_List_53045.xlsx` — prospect list (Wisconsin pilot area)
- `COB_Flow_Onboarding_Playbook_v0.1.docx` — customer onboarding playbook
- `COB_Flow_SPD_Review_Template.docx` — front-door discovery template
- `COB_Flow_Demand_Letter_Template.docx` — operational template; moves into cob-flow-app `src/lib/letters/` when the engine ports in Phase G

### reference/
External authoritative material — regulations, handbooks, case law, training.
- `regulations/Wisconsin Legislature_ Ins 3.40(11)(a).pdf` — Wisconsin's primary COB regulation
- `handbooks/` — COB-TPL Handbook (Medicare COB/TPL), COB Smart webinar (industry overview)
- `case-law/` — subrogation case-law textbook scans (IMG_1366–1371)
- `training/` — external reference training materials: ForwardHealth PDFs, catalog index. The Auto COB Syllabus (12-module curriculum spine) was relocated to cob-flow-app at `content/courses/auto-cob-wisconsin/COB_Flow_Auto_COB_Syllabus.docx` on 2026-05-21. Course content is build-driving and lives in the application repo alongside the future Content Manager (planned Phase I). External reference training materials (ForwardHealth PDFs, the catalog index) stay here.

### archive/
Frozen historical material; not actively consulted.
- `kickoff/` — original Claude Code kickoff prompts (pre-build artifacts)
- `specs/` — superseded Product Spec versions (v0.5, v0.6, v0.7)

## What lives in cob-flow-app/docs/ (NOT here)

Build-driving documents that the cob-flow-app agent reads every session live in the app repo:

- `COB_Flow_Handoff.md` — phase log, updated as each build phase lands
- `COB_Flow_NextJS_Conversion_Handoff.md` — conversion brief and architectural principles
- `COB_Flow_Product_Spec_v0.8.docx` — current product spec
- `COB_Flow_Dashboard_Spec_v0.1.docx` — role/authority/audit architecture
- `COB_Flow_WI_Workflow_v1.0.docx` — 9-phase analyst workflow
- `COB_Flow_MVP.html` — authoritative UI reference prototype
- `superpowers/specs/` and `superpowers/plans/` — per-phase build artifacts

If you're working on strategy, GTM, or reference material — you're in the right place. If you're working on build features or specs that drive implementation, you want cob-flow-app/docs/.

## Working norms

See `CLAUDE.md` for agent guidance, working norms, and architectural guardrails.

## Lifecycle rules

**Spec versions**: when a new Product Spec version supersedes the current one in cob-flow-app/docs/, the previous version moves to `archive/specs/` here. cob-flow-app/docs/ always holds exactly one current version per spec.

**Templates**: working template documents (e.g., demand letter, SPD review) live in `gtm/` while they're being authored. When the engine ports in Phase G, the demand letter template moves into cob-flow-app as a typed data file under `src/lib/letters/`.
