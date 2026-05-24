**COB Flow**

Decision-Support for Coordination of Benefits, Auto Med-Pay & Healthcare Subrogation

Product Spec & MVP Architecture — v0.8 (Wisconsin pilot · ingest architecture)

**Table of Contents**

[1\. Strategic Context	4](#1.-strategic-context)  
[1.1 Market thesis	4](#1.1-market-thesis)  
[1.2 Why now	4](#1.2-why-now)  
[1.3 Positioning relative to existing players	4](#1.3-positioning-relative-to-existing-players)  
[1.4 Product philosophy: decision-support first, automation later	5](#1.4-product-philosophy:-decision-support-first,-automation-later)  
[1.5 Validation before heavy build	5](#1.5-validation-before-heavy-build)  
[2\. Product Overview	6](#2.-product-overview)  
[3\. Target Users & Deployment Modes	7](#3.-target-users-&-deployment-modes)  
[4\. MVP Scope	8](#4.-mvp-scope)  
[4.1 In scope (MVP)	8](#4.1-in-scope-\(mvp\))  
[4.2 Out of scope (MVP)	8](#4.2-out-of-scope-\(mvp\))  
[5\. Canonical Analyst Workflow (9 phases)	9](#5.-canonical-analyst-workflow-\(9-phases\))  
[5.1 Working with personal-injury attorneys (workflow counterparty)	9](#5.1-working-with-personal-injury-attorneys-\(workflow-counterparty\))  
[6\. COB Primacy Decision Engine	11](#6.-cob-primacy-decision-engine)  
[6.1 Decision tree (ordered, v0.3)	11](#6.1-decision-tree-\(ordered,-v0.3\))  
[6.2 Wisconsin overlay	12](#6.2-wisconsin-overlay)  
[6.3 Engine output contract	13](#6.3-engine-output-contract)  
[7\. AI-Assisted Document Parsing (Option B)	14](#7.-ai-assisted-document-parsing-\(option-b\))  
[7.1 Pipeline	14](#7.1-pipeline)  
[7.2 Guardrails	14](#7.2-guardrails)  
[8\. System Architecture	15](#8.-system-architecture)  
[8.1 High-level components	15](#8.1-high-level-components)  
[8.2 Multi-tenancy	15](#8.2-multi-tenancy)  
[9\. Data Model (core entities)	16](#9.-data-model-\(core-entities\))  
[10\. Integrations	17](#10.-integrations)  
[11\. Compliance, Security & Liability	18](#11.-compliance,-security-&-liability)  
[11.1 Liability framing	18](#11.1-liability-framing)  
[11.2 Authority & Approval Model	18](#11.2-authority-&-approval-model)  
[12\. Validation Before Heavy Build	21](#12.-validation-before-heavy-build)  
[13\. Roadmap	22](#13.-roadmap)  
[13.1 Phase 1 — MVP (months 0–4)	22](#13.1-phase-1-—-mvp-\(months-0–4\))  
[13.2 Phase 2 — Workflow completion & production-ready (months 4–9)	22](#13.2-phase-2-—-workflow-completion-&-production-ready-\(months-4–9\))  
[13.3 Phase 3 — Scale & expand (months 9–18)	22](#13.3-phase-3-—-scale-&-expand-\(months-9–18\))  
[14\. Open Questions for Jim	23](#14.-open-questions-for-jim)  
[15\. Research, Standards & References	24](#15.-research,-standards-&-references)  
[15.1 Primary government and standards sources	24](#15.1-primary-government-and-standards-sources)  
[15.2 Industry players (research and partnership targets)	24](#15.2-industry-players-\(research-and-partnership-targets\))  
[15.3 Case-law cited in the engine	24](#15.3-case-law-cited-in-the-engine)  
[15.4 Wisconsin-specific case-law and statutes (pilot state)	25](#15.4-wisconsin-specific-case-law-and-statutes-\(pilot-state\))  
[15.5 Related project documents	25](#15.5-related-project-documents)  
[Appendix A: Glossary	27](#appendix-a:-glossary)  
[Coordination of Benefits — Core Concepts	27](#coordination-of-benefits-—-core-concepts)  
[Medicare & Federal Programs	28](#medicare-&-federal-programs)  
[ERISA & Federal Law	28](#erisa-&-federal-law)  
[Wisconsin Legal Framework	29](#wisconsin-legal-framework)  
[Case Law	30](#case-law)  
[Auto Insurance & PIP	30](#auto-insurance-&-pip)  
[Healthcare & Claims Terminology	31](#healthcare-&-claims-terminology)  
[COB Flow Product — Modules & Workflow	31](#cob-flow-product-—-modules-&-workflow)  
[COB Flow Product — Roles & Authority	32](#cob-flow-product-—-roles-&-authority)  
[Deployment Modes & Customer Types	33](#deployment-modes-&-customer-types)  
[Technical Architecture & Standards	33](#technical-architecture-&-standards)  
[Industry Organizations & Competitors	35](#industry-organizations-&-competitors)

# **1\. Strategic Context** {#1.-strategic-context}

## **1.1 Market thesis** {#1.1-market-thesis}

Coordination of benefits (COB), auto med-pay/PIP recovery, and post-payment subrogation are still predominantly manual workflows inside health plans, TPAs, and recovery vendors. CAQH puts the industry's annual administrative cost from COB inefficiencies at roughly $800 million, with about 60% borne by providers and roughly 60% of total cost attributable to FTE labor (CAQH COB Smart Webinar, 2016). The opportunity is to put analyst-grade reasoning behind every primacy call and every recovery decision, with explainable rationale and audit trail, and to give plans, TPAs, and independent recovery vendors a single tool that does the deeper work the existing infrastructure leaves on the table.

## **1.2 Why now** {#1.2-why-now}

Three shifts make this a buildable product today rather than a 2010-era enterprise integration project. First, CAQH COB Smart and similar utilities have effectively solved the upstream eligibility-discovery problem at scale for commercial COB — roughly half the nation's commercially insured lives are covered — which means a downstream product can assume reliable coverage data exists and focus on the harder question of which coverage pays first when the answer is legally non-obvious. Second, modern LLMs make extracting structured COB clauses from 80–150 page plan documents tractable for the first time, replacing the most expensive minute-by-minute analyst task in the workflow. Third, the auditability and explainability now expected of any AI-assisted product in regulated industries aligns precisely with what payer compliance teams already demand for recovery decisions — making explainability a feature, not a tax.

## **1.3 Positioning relative to existing players** {#1.3-positioning-relative-to-existing-players}

| Player | Category | Relationship to COB Flow |
| :---- | :---- | :---- |
| CAQH COB Smart | Upstream / eligibility | Complementary. COB Smart discovers overlapping coverage and runs primacy at the front-end (commercial COB only — explicitly excludes subrogation). COB Flow consumes COB Smart output as a data source and handles the deeper recovery and post-payment determinations COB Smart leaves alone. |
| Rawlings, Optum/Equian, The Phia Group | Direct competitor (subrogation / COB recovery) | Enterprise vendors with long integration cycles and opaque determinations. COB Flow's wedge is explainability, faster onboarding, and the independent-vendor service mode that lets a small recovery shop deliver vendor-grade outcomes. |
| Cotiviti, Zelis | Adjacent (payment integrity) | Overlap on pre-pay COB editing but core business is broader payment accuracy. Potential partnership or integration target, not a direct competitor on subrogation/recovery. |
| Change Healthcare, Gainwell | Infrastructure (claims platforms / Medicaid systems) | Not competitors. Potential channel partners. Many Medicaid agencies run on Gainwell — relevant for the Medicaid TPL angle in Phase 3\. |

## **1.4 Product philosophy: decision-support first, automation later** {#1.4-product-philosophy:-decision-support-first,-automation-later}

COB Flow v1 ships as a decision-support tool: the engine recommends a primary payer, surfaces the controlling rule and citations, and assigns a confidence score, but an analyst signs off before any demand goes out. This is a deliberate choice with three rationales. First, it limits liability exposure during the period when we have no empirical accuracy data on the engine. Second, it shortens the enterprise sales cycle — buyers who would balk at "autonomous decisioning" sign "productivity tool with audit trail" without their compliance officer raising hands. Third, every analyst override produces labeled training data we can use to graduate the engine toward higher automation in Phase 3, when the accuracy and reviewer-trust story is provable rather than promised.

**Graduation path:** Phase 1 — recommend, human signs. Phase 2 — auto-determine high-confidence calls, route low-confidence or high-dollar to a review queue. Phase 3 — fully autonomous determination with sampled QA, premium pricing tier.

## **1.5 Validation before heavy build** {#1.5-validation-before-heavy-build}

The existing MVP prototype is the discovery asset for the next 6–8 weeks, not the production codebase. The validation work that needs to happen before further engineering investment is the customer-development sequence flagged in the project's strategy doc: structured interviews with COB analysts, subrogation managers, and a handful of carrier and hospital revenue-cycle directors; quantification of labor cost per recovery at each interview site so we can build a defensible ROI calculation; and pricing-sensitivity tests across the three deployment modes (per-seat license, per-recovery contingency, hybrid). Until those data points exist, we resist the urge to build more product.

# **2\. Product Overview** {#2.-product-overview}

COB Flow is a multi-tenant SaaS platform that automates four workflows in healthcare coordination of benefits and post-payment recovery: COB primacy determination, auto med-pay/PIP recovery, claims triage, and recovery correspondence. It is deployable in three modes (carrier in-house, subrogation vendor / TPA, and independent vendor service), runs as a responsive web application for desktop and mobile, and is built around an explainable rules engine whose every output is traceable to a controlling rule and a citation.

# **3\. Target Users & Deployment Modes** {#3.-target-users-&-deployment-modes}

| Mode | Primary user | Data source | Revenue model |
| :---- | :---- | :---- | :---- |
| Carrier in-house | Internal COB and subrogation recovery teams at health plans and carriers | Direct integration with the carrier's claims platform (Facets, QNXT, HealthRules, custom) | Per-seat SaaS license, tiered by claim volume |
| Subrogation vendor / TPA | Recovery analysts at firms like Rawlings, Optum, Phia | Periodic file feeds from multiple client plans | Per-seat plus per-recovery, white-label option |
| Independent vendor service | Your own team delivering recovery-as-a-service to at-risk plans and self-funded employers | Inbound integration with the at-risk customer's claims system; the back office runs entirely in COB Flow | Contingency fee on net recoveries (typically 20–30%) or hybrid monthly \+ lower contingency |

Two additional customer segments are scoped to Phase 3 once the carrier/vendor wedge is proven: hospital revenue-cycle teams running their own COB and post-payment recovery on patient accounts (smaller deals, faster sales cycle, harder ROI proof) and law firms doing health-plan reimbursement work (narrow but high-willingness-to-pay segment).

# **4\. MVP Scope** {#4.-mvp-scope}

## **4.1 In scope (MVP)** {#4.1-in-scope-(mvp)}

* Tenant-aware claims dashboard with intake from CSV/JSON upload, manual entry, and a future hook for COB Smart 271 responses.

* Automated triage using Diagnosis and Trauma Code Edits (the Medicaid TPL term for ICD-based subrogation potential flags), place-of-service signals, and accident indicators.

* COB primacy decision engine — explainable, rule-versioned, citation-bearing — covering the no-fault PIP, ERISA preemption, MSP (working aged / disability / ESRD / TFL), employee-over-dependent, active-over-retiree, birthday rule with QMCSO override and separated-parents fallback, ERISA self-funded coordination with escape-clause unenforceability, payer-of-last-resort doctrine for Medicaid, and the longer/shorter rule.

* Auto med-pay/PIP recovery tracker with a per-state no-fault rule library and conditional-payment workflow language consistent with CMS guidance.

* AI-assisted document parsing (Option B — analyst-confirmed): LLM extracts candidate fields from uploaded plan documents and EOBs; analyst confirms each field with one click before any field enters the engine.

* Templated document generation: COB questionnaire, demand letter to auto carrier, reimbursement notice to member, with audit-trail footer showing the determination rule and confidence.

* Responsive single-page web application that works on a desktop browser and on a phone screen (no native iOS/Android build for v1).

* Role-based access (analyst, supervisor, admin), per-tenant data isolation via row-level security, full audit log of PHI access and every determination.

## **4.2 Out of scope (MVP)** {#4.2-out-of-scope-(mvp)}

* Live X12 270/271 eligibility lookups, 835 remittance ingestion, 837 claim parsing (mocked from sample files in v1).

* Direct integration with carrier claims platforms; v1 uses file drop and a public REST API.

* Payment posting / accounts receivable.

* Predictive ML scoring for recovery likelihood (deterministic rules only in v1).

* Native iOS/Android apps.

* Hospital RCM and law-firm tenant types (Phase 3).

* Dedicated modules for Phases 4 (Liability & Fault), 7 (Ongoing Management), and 9 (Audit & Closure) of the canonical workflow — see Section 5 for the workflow mapping and Section 12 for when these ship.

# **5\. Canonical Analyst Workflow (9 phases)** {#5.-canonical-analyst-workflow-(9-phases)}

The canonical end-to-end analyst workflow for a Wisconsin auto-related COB claim is defined in the separate document COB\_Flow\_WI\_Workflow\_v1.0.docx. It is the source of truth for what the platform must support across the full claim lifecycle. The product surfaces four modules in MVP; six of the nine workflow phases are fully covered, two are scheduled for Phase 2 of the product roadmap, and one is partially covered with the negotiation workspace deferred to Phase 2\.

| \# | Phase | One-line summary | Product status |
| :---- | :---- | :---- | :---- |
| 1 | Claim Intake | Capture accident facts, claimant, medical bills, attorney rep. | MVP |
| 2 | Coverage Discovery | Identify health, auto liability, Med-Pay, UM/UIM, ERISA, Medicare, Medicaid, WC. | MVP |
| 3 | Coverage Classification | Classify each coverage; flag ERISA self-funded for preemption analysis. | MVP |
| 4 | Liability & Fault Investigation | Establish % fault per party; compute recoverable under comparative negligence. | Phase 2 |
| 5 | Apply COB Rules | Engine determines primary payer with citations, confidence, rationale. | MVP |
| 6 | Payment Routing & Demand | Route to primary; generate COB questionnaire or demand letter. | MVP |
| 7 | Ongoing Claim Management | Track duplicate payments, reserves, supplemental EOBs, ongoing treatment. | Phase 2 |
| 8 | Subrogation & Recovery | Made-whole eval, common-fund pro-rata, lien negotiation, settlement. | MVP (partial) |
| 9 | Audit & Closure | Verify primacy, confirm no duplicates, close with audit-ready summary. | Phase 2 |

The three Phase 2 expansion phases (4, 7, 9\) are real workflow surface analysts walk through today. The MVP supports them via free-text notes on the Claim and the underlying audit log, but does not yet surface dedicated user interfaces. Phase 2 of the product roadmap delivers each as a standalone module — see Section 13.2.

## **5.1 Working with personal-injury attorneys (workflow counterparty)** {#5.1-working-with-personal-injury-attorneys-(workflow-counterparty)}

Once a member retains counsel, the claimant-side personal-injury attorney becomes the primary recovery counterparty for any claim with a third-party settlement — not the auto carrier directly. The prototype already models this: PI attorneys are recipients of three Tier-1 and Tier-2 letter templates (LIEN\_NOTICE, LIEN\_REDUCTION\_OFFER, SETTLEMENT\_ACK). Two product implications are worth stating explicitly.

First, the fully-insured vs. ERISA self-funded distinction radically changes the attorney’s leverage on lien-reduction. For fully-insured Wisconsin plans, made-whole (Rimes, Vogt) and common-fund (Petta) apply and the attorney’s arguments are strong. For ERISA self-funded plans with properly drafted plan documents, the Sereboff line preempts those doctrines and the attorney’s reduction arguments largely fall away. The engine flags this distinction in the WI overlay; the customer-facing value is that COB Flow’s analyst sees the ERISA status and the plan-document language and makes the right argument for the file type — a skill differential most claimant-side attorneys do not consistently track.

Second, PI attorneys are not a customer segment for COB Flow — they are the counterparty in the workflow. The Phase 3 customer expansion lists in Section 13.3 (“hospital revenue-cycle and law-firm tenant types”) refer to reimbursement-side law firms (the firm a health plan hires to pursue recovery). That is a different segment with an opposite value prop. Discovery interviews should distinguish the two clearly.

# **6\. COB Primacy Decision Engine** {#6.-cob-primacy-decision-engine}

This is the defensible IP. The engine takes a structured claim plus a roster of identified coverages and returns the primary payer along with the controlling rule, citations, confidence, rationale, and any warnings or manual-review flags. The rules below are encoded as an ordered decision tree; the first rule that fires determines primacy. The v0.3 ruleset reflects refinements drawn from the CMS Coordination of Benefits Workbook (Module 5, 2015), the CMS Medicaid COB/TPL Handbook (2020), and the Wisconsin-specific recovery doctrines required for the pilot.

## **6.1 Decision tree (ordered, v0.3)** {#6.1-decision-tree-(ordered,-v0.3)}

1. Auto accident with mandatory PIP / no-fault. In states with mandatory no-fault statutes (e.g., MI, FL, NY, NJ, PA, HI, KS, KY, MA, MN, ND, UT), the auto carrier is generally primary for accident-related medical expenses up to PIP limits, regardless of health-plan provisions. Where the health plan is self-funded ERISA with a properly drafted coordination clause and the auto policy contains an excess clause, ERISA preemption analysis is required and outcome is circuit-dependent (Buckley v. American Medical Security, 973 F. Supp. 1005).

2. Self-funded ERISA plan with valid coordination clause. The plan's COB terms preempt conflicting state COB regulation under ERISA § 514\. Escape clauses ("this plan pays only what is not covered by any other plan") are unenforceable against another ERISA plan under federal common law (McGurl v. Trucking Employees of N. Jersey Welfare Fund; PM Group Life Ins. Co. v. Western Growers Assurance Trust).

3. Medicare Secondary Payer — working aged. Member is 65 or older and covered as an active employee, or as the spouse of an active employee, where the employer has 20 or more employees. Employer Group Health Plan (EGHP) is primary; Medicare is secondary. 42 U.S.C. § 1395y(b)(1)(A); 42 C.F.R. § 411.170.

4. Medicare Secondary Payer — disability. Member is under 65, entitled to Medicare on the basis of disability, and covered by an EGHP through current employment (own or family member) where the employer has 100 or more employees. EGHP is primary; Medicare is secondary. 42 U.S.C. § 1395y(b)(1)(B).

5. Medicare Secondary Payer — ESRD coordination period. Member is entitled to Medicare on the basis of End-Stage Renal Disease and has EGHP or COBRA coverage. The EGHP/COBRA is primary for the first 30 months; Medicare is secondary during this period. After the 30-month coordination period ends, Medicare becomes primary. 42 U.S.C. § 1395y(b)(1)(C).

6. TRICARE for Life (TFL). Medicare is primary, TFL is secondary. (Distinct from regular TRICARE where coordination follows EGHP rules for active-duty families.)

7. Liability or workers' compensation coverage available. Medicare is the secondary payer where liability or workers' comp insurance is available. Provider must bill the liability/WC carrier first; Medicare may make a conditional payment if the other carrier does not pay within 120 days, recoverable on settlement (Medicare Modernization Act of 2003, Title III § 301).

8. Employee vs. dependent. A plan covering the patient as an employee or subscriber is primary to a plan covering the same patient as a dependent. NAIC Model COB Regulation § 6(D)(1).

9. Active employee vs. retiree / COBRA. Active-employee coverage is primary to retiree or COBRA continuation coverage. (Exception: ESRD during the 30-month coordination period — see Rule 5.)

10. Dependent child — court order (QMCSO). A Qualified Medical Child Support Order designating a primary plan overrides the birthday rule. ERISA § 609(a); 29 U.S.C. § 1169\.

11. Dependent child — birthday rule. When a child is covered under both parents' plans without a QMCSO, the plan of the parent whose birthday (month/day, year ignored) falls earlier in the calendar year is primary. Tie-break: longer coverage. NAIC Model COB Regulation § 6(D)(3); Catholic Diocese of Biloxi Supp. Med. Plan.

12. Dependent child — separated parents, no court order. Custodial parent's plan is primary; then custodial parent's spouse; then non-custodial parent; then non-custodial parent's spouse.

13. Medicaid present — payer of last resort. When Medicaid is one of the coverages, Medicaid is always the payer of last resort by federal law (Social Security Act § 1902(a)(25); 42 CFR 433 Subpart D). All other identified third-party payers must be exhausted before Medicaid pays. This is enforced via cost avoidance (provider bills third party first) or pay-and-chase (Medicaid pays then recovers).

14. Longer/shorter rule (fallback). If no higher-priority rule applies, the plan that has covered the patient longest is primary. NAIC Model COB Regulation § 6(D)(7). The engine flags any determination resting on this rule for analyst review.

## **6.2 Wisconsin overlay** {#6.2-wisconsin-overlay}

For any claim with accident state \= WI, the engine produces a Wisconsin overlay block alongside the primacy determination. The overlay does not change primacy; it surfaces Wisconsin-specific recovery constraints that the analyst, the Recovery Tracker, and the demand-letter generator all consume downstream.

* **Made-whole doctrine flag.** Every WI determination is annotated with the made-whole requirement (Rimes v. State Farm Mut. Auto. Ins. Co., 106 Wis. 2d 263 (1982); Vogt v. Schroeder, 129 Wis. 2d 3 (1986); Petta v. ABC Ins. Co., 692 N.W.2d 639 (2005)). Where a self-funded ERISA plan is present on the claim, the note shifts to recommend plan-language preemption review before relying on the doctrine.

* **Comparative-negligence calculator.** Where a Liability record carries the claimant's % fault, the engine computes recoverable medical damages \= total medical paid × (defendant's % fault), and zeroes the recoverable amount if the claimant is 51% or more at fault (Wis. Stat. § 895.045).

* **Common-fund pro-rata.** Where settlement and attorney-fee data are available, the engine computes the plan's pro-rata share of attorney fees as (plan recovery / total recovery) × attorney fee, and returns the net plan recovery.

* **Statutory framework.** Wis. Admin. Code § Ins 3.40 (the Wisconsin Coordination of Benefits administrative rule, codifying the NAIC-model order of benefit determination at § Ins 3.40(11)(b)), together with Wis. Stat. § 632.32 (auto policy regulation) and Wis. Stat. § 631.43 (subrogation rights under insurance contracts), are surfaced on every WI overlay.

* **Regulatory authority for the recovery workflow — § Ins 3.40(18).** Wis. Admin. Code § Ins 3.40(18) ("Coordination with noncomplying plans") is the explicit regulatory basis for COB Flow’s recovery workflow. Traditional automobile "fault" contracts are carved out from the "complying plan" framework, so when a Wisconsin health plan stands secondary to an auto carrier that does not engage in good-faith coordination, § Ins 3.40(18) authorizes the complying plan to advance benefits to the member and recover from the noncomplying plan through subrogation. The official note attached to § Ins 3.40(18) directs precisely this workflow: "the Complying Plan should assume the primary position in order to avoid undue claim delays and hardship to the insured. The Complying Plan may, through its subrogation rights, seek reimbursement for such payments." This is the legal anchor of the product’s positioning, not a marketing assertion; the Demand Letter Template cites it directly.

* **Known engine gaps against § Ins 3.40.** Three discrete gaps identified during the 2026-05-17 mapping pass against the full Ins 3.40 text are documented for Phase 2/3 closure: (a) continuation coverage rule, § Ins 3.40(11)(b)5m — COBRA / Wis. Stat. § 632.897(3)(a) continuation coverage gets its own primacy slot between active/inactive employee and longer/shorter, which the engine does not model as a discrete rule today; (b) 24-hour bridge rule, § Ins 3.40(11)(b)6m — when applying the longer/shorter fallback, two plans are treated as one continuous period if the second begins within 24 hours of the first ending, and benefit-scope or payer changes do not restart the clock; (c) COB benefits cap math, § Ins 3.40(12) — secondary may reduce benefits so total payments do not exceed allowable expenses, with credit-savings tracked across a claim determination period. The engine today does primacy ordering; (12) does dollar-level allowance calculation with running-balance state. The cap is the larger Phase 2/3 item.

Wisconsin is not a no-fault state, so the engine's mandatory no-fault PIP rule never fires for a WI claim. This is enforced by the decision tree, not by accident — Rule 1 above checks NO\_FAULT\_STATES membership before applying.

## **6.3 Engine output contract** {#6.3-engine-output-contract}

Every determination is emitted as a structured object so it can be persisted, displayed in the UI, and embedded in the generated demand package as an audit trail.

{   "primaryCoverageId": "cov\_auto\_allstate\_001",   "secondaryCoverageId": "cov\_health\_aetna\_002",   "rule": "MSP\_WORKING\_AGED",   "ruleVersion": "v0.2",   "ruleDescription": "Medicare Secondary Payer — working aged (employer size \>=20).",   "citations": \["42 U.S.C. § 1395y(b)(1)(A)", "42 C.F.R. § 411.170"\],   "confidence": "HIGH",   "rationale": "Member age 70, covered as active employee under Anthem PPO (employer assumed \>=20 staff). MSP working-aged rule applies; Anthem is primary, Medicare is secondary.",   "warnings": \[\],   "reviewRequired": false,   "determinationId": "det\_2026\_05\_16\_abc123",   "determinedAt": "2026-05-16T13:42:11Z",   "determinedBy": "engine" }

# **7\. AI-Assisted Document Parsing (Option B)** {#7.-ai-assisted-document-parsing-(option-b)}

Locating the COB clause inside an 80–150 page plan booklet, or transcribing fields from a payer-specific EOB, is the most time-consuming manual task in the analyst workflow today. COB Flow includes LLM-assisted extraction in MVP scope, with the explicit constraint that the LLM operates as a typist, never a judge.

## **7.1 Pipeline** {#7.1-pipeline}

15. Analyst uploads a plan document (PDF) or EOB image/PDF.

16. OCR runs on scanned content; native text PDFs are parsed directly.

17. LLM extraction prompt is scoped narrowly per document type: for plan docs, locate the Coordination of Benefits section and classify the COB clause type (escape, excess, non-conforming, NAIC-standard) plus capture verbatim the relevant paragraph; for EOBs, extract payer name, member ID, dates of service, billed and paid amounts, denial codes.

18. Extracted fields are surfaced in a Review pane with the source-document highlight visible side-by-side. Every field has a confirm/edit control.

19. Only confirmed fields flow into the structured Coverage and Claim records that feed the engine. Unconfirmed fields are persisted as draft, never visible to the engine.

20. Every confirm/edit action is logged with the original LLM output, the analyst's confirmed value, the document URI, and a timestamp — building the labeled dataset needed to harden the model over time.

## **7.2 Guardrails** {#7.2-guardrails}

* The engine treats LLM-extracted-but-unconfirmed values as missing inputs. It will never run a determination on an unconfirmed field.

* Free-text rationale generation is template-driven, not LLM-generated. Citations are pulled from a static rule-to-citation table, not produced by the model.

* Periodic random audits of confirmed extractions against source documents; reviewer disagreement rate is a Service Level Objective tracked in the admin dashboard.

# **8\. System Architecture** {#8.-system-architecture}

## **8.1 High-level components** {#8.1-high-level-components}

* **Web client —** React SPA, responsive (TailwindCSS), works on desktop and mobile browsers. Native wrapper deferred to Phase 3\.

* **API gateway —** REST \+ JSON. OAuth2 / SAML SSO for enterprise tenants. mTLS for carrier integrations.

* **Application service —** Stateless Node.js or Python FastAPI. Hosts intake, triage, the COB engine, document generation, and workflow state machine.

* **COB decision engine —** Pure function library, versioned independently. Outputs the structured determination contract above. Exposable as a standalone API in Phase 2\.

* **Extraction service —** OCR (tesseract or AWS Textract) plus an LLM pipeline (Anthropic Claude or OpenAI) for plan-doc and EOB field extraction. Strict prompt templates, structured-output mode, and per-field confidence.

* **Document service —** Templated DOCX/PDF generation (docx-js, WeasyPrint) with merge fields populated from claim \+ determination data.

* **Integration adapters —** Pluggable adapters per claims platform: SFTP/CSV in v1; X12 270/271/837/835 and FHIR R4 Coverage/Claim/EOB/Patient in Phase 2; native Facets/QNXT/HealthRules/HealthEdge/TriZetto adapters in Phase 3\.

* **Datastore —** Postgres with row-level security keyed on tenant\_id for shared cluster; schema-per-tenant for enterprise. S3 \+ KMS for source documents and generated PDFs.

* **Audit log —** Append-only, immutable. Every PHI read, every determination, every LLM extraction confirmation, every document send is logged with user, timestamp, and rule version.

## **8.2 Multi-tenancy** {#8.2-multi-tenancy}

Shared Postgres cluster with row-level security keyed on tenant\_id for smaller and mid-market customers. Schema-per-tenant or dedicated DB for large carriers, chosen at provisioning. Tenant context is derived from the auth token on every request; the application layer enforces cross-tenant rejection at the query layer.

# **9\. Data Model (core entities)** {#9.-data-model-(core-entities)}

| Entity | Key fields |
| :---- | :---- |
| Tenant | id, name, mode (carrier|vendor|service), config, data\_isolation (rls|schema|db) |
| Member | id, tenant\_id, external\_member\_id, dob, age, relationship\_to\_subscriber, qmcso (if any), parents\_separated (bool), hashed\_ssn\_last4 |
| Coverage | id, member\_id, payer\_name, plan\_type (HEALTH|AUTO\_PIP|AUTO\_MEDPAY|WC|LIABILITY|MEDICARE|MEDICAID|TRICARE|TFL|VA|BLACK\_LUNG), funding (SELF\_FUNDED|FULLY\_INSURED|N/A), basis (ACTIVE\_EMPLOYEE|SPOUSE\_OF\_ACTIVE|RETIREE|COBRA|DEPENDENT\_CHILD|DEPENDENT\_SPOUSE|SELF), employer\_size (XS\<20 | S\<100 | L\>=100), effective\_date, policy\_number, group\_number, plan\_document\_ref, has\_coordination\_clause, has\_escape\_clause, has\_excess\_clause, source (manual|file|cob\_smart|x12) |
| Claim | id, tenant\_id, member\_id, service\_dates, billed\_amount, paid\_amount, diagnosis\_codes\[\], place\_of\_service, accident\_indicator (Y|N|U), accident\_state, accident\_date, source\_system, esrd\_indicator |
| TriageResult | claim\_id, has\_subrogation\_potential, signals\[\], score, recommended\_action, trauma\_code\_edit\_hit (bool) |
| CobDetermination | claim\_id, primary\_coverage\_id, secondary\_coverage\_id, rule, rule\_version, citations\[\], confidence, rationale, warnings\[\], review\_required, reviewer\_id, reviewed\_at, override\_reason |
| Recovery | id, claim\_id, target\_carrier, status (IDENTIFIED|INVESTIGATING|DEMAND\_SENT|NEGOTIATING|SETTLED|CLOSED), demand\_amount, recovered\_amount, status\_history\[\], assigned\_to, opened\_date |
| ExtractionEvent | id, document\_id, field\_name, llm\_value, llm\_confidence, confirmed\_value, confirmed\_by, confirmed\_at — used both as audit and as training data |
| Document | id, recovery\_id, type (DEMAND|QUESTIONNAIRE|LIEN|SETTLEMENT|MEDICAID\_TPL\_NOTICE), template\_version, generated\_at, storage\_url, sent\_at, sent\_to |
| AuditEvent | id, tenant\_id, user\_id, action, target\_type, target\_id, payload\_hash, timestamp |

# **10\. Integrations** {#10.-integrations}

## **10.1 Standards-based & adapter integrations**

* **Tier 1 — File drop (MVP).** SFTP or web upload of CSV/JSON extracts on a daily/weekly cadence. Smallest integration burden.

* **Tier 2 — Standards-based (Phase 2).** X12 270/271 for eligibility (including COB Smart-style returns), 837 for claim ingest, 835 for remittance, FHIR R4 Coverage / Claim / EOB / Patient.

* **Tier 3 — Direct adapters (Phase 3).** Native adapters to Facets, QNXT, HealthRules Payer, HealthEdge, TriZetto. Real-time event ingest via webhooks or message bus.

* **CAQH COB Smart integration (Phase 2).** Consume COB Smart match files as a Coverage source. Annotate determinations whose inputs came via COB Smart. Position as "the recovery layer on top of COB Smart" in collateral.

## **10.2 Customer claim feeds (added v0.8)**

The customer claims feed is the inbound channel through which a customer’s claim data enters COB Flow for triage and recovery. Pass 1 deployments use a single feed per customer; pass 2+ supports multi-customer configurations in vendor mode. The feed lands as a daily batch (most common), a periodic batch on a customer-defined cadence, or an event-driven stream for customers with mature integration infrastructure.

Six logical stages structure the pipeline. (1) Customer feed — SFTP with key-based authentication is the small-TPA default; X12 837 batches for larger carriers; flat CSV or pipe-delimited extracts for less-integrated customers; API endpoints for modern payer platforms. (2) Ingest pipeline — validates the file, parses records, normalizes into the COB Flow internal claim model; logs per-file and per-record errors. (3) Triage — the existing decision-engine triage logic flags injury-related claims by ICD-10 S00–T98 diagnosis range, trauma indicators, and place-of-service signals. (4) Case grouping — multiple claims for the same member with overlapping injury-related diagnoses within a temporal window group into one case; manual grouping in pass 1, heuristic automation in pass 2\. (5) Case state lifecycle — GATHERING → READY → ASSIGNED → IN\_RECOVERY → CLOSED → REOPENED. (6) Monitoring — feed health, error queue, ingest configuration; lives in Admin → Integrations.

The late-arriving claims problem requires explicit handling. Healthcare claims for a single accident arrive over weeks to months from many providers — ER day 1, specialist visits weeks 1–12, imaging, surgery, PT/rehab, prescriptions. A new claim that matches an open case’s member auto-attaches to that case; a new claim that matches a closed case’s member routes to the Supervisor approval queue as a LATE\_ARRIVAL\_TO\_CLOSED\_CASE item (the tenth approval-queue type beyond the nine in Dashboard Spec §5). The Supervisor decides supplemental demand, full reopen, or write-off, with audit logging on the decision.

Ownership of the feed splits along role lines that mirror the four-role authority model. Admin owns the pipe: feed health, file arrival, parsing, errors, customer configuration. Supervisors and analysts own the content: manual case grouping, judgment on case readiness, late-arrival decisions. Managers own the operational view: average days from first-claim-received to case-ready, leakage estimate, customer SLA conformance.

Several feed-design decisions deliberately wait for customer discovery. Transport format (SFTP / X12 / API / FHIR), schema (which fields the customer’s extract includes), cadence (daily / weekly / real-time), volume (50/day vs 5,000/day), error-handling expectations (acknowledgment files? reconciliation cycles?), historical-backfill scope, and date-of-loss handling are all open questions for Phase 1 customer-discovery interviews. The architecture above describes how the feed flows through the platform without committing to a specific customer’s format.

The implementation-level detail of this architecture lives in COB\_Flow\_NextJS\_Conversion\_Handoff.md §11. Specific surfaces — the Customer Feeds list in Admin → Integrations, the case-state lifecycle pill on Case Detail, the LATE\_ARRIVAL\_TO\_CLOSED\_CASE approval queue rendering, the ingest event filter in Admin → Audit — are demonstrated in the working prototype (COB\_Flow\_MVP.html).

# **11\. Compliance, Security & Liability** {#11.-compliance,-security-&-liability}

* **HIPAA.** The platform handles PHI from day one. BAAs with all subprocessors; TLS 1.2+ in transit, AES-256 at rest; least-privilege access; audit logging of every PHI access; breach notification process.

* **SOC 2 Type II.** Required for enterprise carrier sales. Begin readiness work after MVP; expect 9–12 months including observation window.

* **State insurance regulation.** Some states regulate third-party recovery vendors (e.g., NY DFS, CA DOI). Legal review per state before contracting.

* **Data residency.** US-only hosting for v1. State-specific residency considerations at enterprise scale.

* **Right to audit.** Enterprise contracts will require third-party audit rights. Build SOC 2 plus a customer audit response process together.

## **11.1 Liability framing** {#11.1-liability-framing}

Every determination COB Flow emits carries a confidence band, a citation, and a structured rationale. In v1 every determination requires analyst sign-off before any demand letter is sent or a primacy call is posted to the source claims system. Customer contracts will include a standard disclaimer that COB Flow output is a decision-support recommendation, not legal advice, and that the licensed user remains responsible for the final determination. Errors-and-omissions coverage scaled to recovery dollar volume is part of the v1 commercial template. The graduation from decision-support to autonomous determination in Phase 3 will require a measured engine-accuracy baseline (publicly reported), updated contract language reflecting the change in responsibility allocation, and a corresponding insurance line.

## **11.2 Authority & Approval Model** {#11.2-authority-&-approval-model}

MVP ships with the canonical four-role model: Analyst, Supervisor, Manager, and Admin. Role controls workspace access and the action set a user is permitted to take. Roles are distinct from Job Levels (Trainee, Junior, Mid, Senior), which apply only to the Analyst role and carry default authority bands (settlement, demand, lien reduction, closure). The model below is now built in working form in the prototype; production hardening of approval routing, SLA enforcement, and dashboard analytics remains scoped for Phase 2\.

**v0.6 update — Dashboard Spec is now the authoritative companion.** The prototype now demonstrates this section’s model in working form. The implementation-level companion document is COB\_Flow\_Dashboard\_Spec\_v0.1.docx, which documents the four-role workspace architecture (Analyst, Supervisor, Manager, Admin), the Job Levels authority framework (Trainee / Junior / Mid / Senior with default authority bands), the per-analyst override flow (Supervisor proposes, Manager approves), the per-file authority elevation flow (Supervisor grants within ceiling; above ceiling routes to Manager), coaching notes (role-private, separate from audit), and the implementation issues that need resolution before pilot. Sections 11.2.1 through 11.2.6 below remain accurate at the strategic level but the Dashboard Spec carries the deeper detail and the open implementation questions.

v0.7 update — role-model cleanup. §11.2.1 Roles table, the §11.2 intro, §11.2.6 architectural guardrail, §13.2 Phase 2 framing, and the Appendix A Roles & Authority glossary entries have all been reconciled to the canonical four-role model (Analyst, Supervisor, Manager, Admin) with Job Levels (Trainee, Junior, Mid, Senior) applying only to the Analyst role. Senior Analyst is no longer a distinct role — it is a Job Level; the glossary keeps the term as an informal pointer. A new Role vs. Job Level note follows the §11.2.1 Roles table. §14 picks up three new deferred-design entries: AI document parsing lifecycle, Send-to-Work-Comp routing path, and closed-file archival.

### **11.2.1 Roles**

| Role | Typical responsibility | Default authority posture |
| :---- | :---- | :---- |
| Analyst | Owns claim file day-to-day; runs determinations; sends correspondence; negotiates routine recoveries. Job Level (Trainee, Junior, Mid, or Senior) sets default authority bands. | Bounded by Job-Level defaults plus any per-analyst override or per-file authority elevation. Cannot edit sent correspondence; lock override requires Supervisor. Cannot close recoveries above level threshold without approval. |
| Supervisor | Reviews and approves analyst escalations; performs QC sampling; manages team-level metrics and per-analyst authority overrides; sets team SLAs. | Approves recovery closures, lien reductions, and settlement acceptances above analyst authority. Grants per-file authority elevation within the supervisor ceiling; above ceiling escalates to Manager. Can override correspondence lock. Reassigns claims. |
| Manager | Owns customer relationship and customer-level outcomes; defines Job Levels and their default authority bands; approves per-analyst overrides and above-supervisor file-authority grants; reviews supervisor performance. | Defines Job Levels and authority bands. Approves authority overrides escalated from Supervisor. Approves new template publication. Views customer-wide dashboards. |
| Admin | System administration: customer settings, user provisioning, security policy, integration configuration, compliance attestations, audit log access. | Full system-configuration access. No claim-level workflow actions. Read-only on claim and recovery data. Role may be held in addition to Manager at small customers. |

Role vs. Job Level. The two are distinct dimensions. Role determines which workspace surfaces the user sees (Analyst workflow, Management workspace for Supervisors and Managers, Admin workspace for Admins) and which actions are permitted. Job Level applies only to the Analyst role and is the default authority band template — Trainee carries the most conservative defaults; Senior carries the broadest. Supervisors do not have a Job Level; they have a supervisor ceiling configured at the customer level. A user can hold more than one Role in production — Manager plus Admin is a common combination at small customers.

### **11.2.2 Per-analyst authority limits (configurable)**

Authority limits are set per-analyst at the customer level by the Manager role. They are dollar (or percentage) thresholds above which the analyst's intended action requires supervisor approval before completing. A typical Analyst at the Junior Job Level at a Wisconsin pilot might have settlement authority of $10,000 and lien-reduction authority of 20%; an Analyst at the Senior Job Level with five years of experience might be $50,000 and 35%. The numbers are illustrative; each customer sets its own bands.

* **Settlement acceptance authority.** Maximum dollar amount of a proposed settlement the analyst can accept and close without supervisor sign-off.

* **Demand authority.** Maximum demand dollar amount the analyst can send without supervisor review. (Above the limit, the draft is sent to the supervisor approval queue before it can transition to SENT.)

* **Lien reduction authority.** Maximum percentage reduction off the plan's gross recovery the analyst may offer in negotiation, after the automatic common-fund pro-rata. Beyond this, supervisor approval required.

* **Recovery closure authority.** Maximum dollar amount the analyst can close a recovery for. Closures above the threshold require supervisor closure-approval.

* **Letter-content override authority.** Whether the analyst may request edit on locked correspondence (always allowed) versus directly override the lock (Supervisor and above only).

* **Template publication authority.** Whether the user may create or modify customer-wide letter templates. Manager and above only by default.

### **11.2.3 Approval workflow**

21. Analyst attempts an action above their authority limit (send demand \> limit, accept settlement \> limit, close recovery \> limit, override locked letter).

22. System routes a structured Approval Request to the configured approver role (Supervisor by default; Manager for above-supervisor amounts) with the action context, the dollar amount, the proposed disposition, and a required analyst justification.

23. Approver sees the request in their queue. Approve, request revisions, or reject. Each decision carries a required justification field.

24. On approve: the action proceeds. The audit log records the originator, the approver, the dollar amount, and both justifications. On reject: the action is blocked and the rejection rationale is returned to the analyst.

25. Escalation policy: if a Supervisor request is not acted on within the customer-configured SLA (default 24 business hours), the request auto-escalates to the Manager queue.

### **11.2.4 Supervisor dashboard (Phase 2\)**

* Pending approvals queue — count and aging, sortable by dollar amount and SLA risk.

* Team workload — open claims per analyst, recovery-stage distribution, average days-to-resolution per analyst.

* QC sampling — random sample of N% of closed recoveries from prior week flagged for QC review; supervisor confirms or flags issues.

* Correspondence lock activity — every override edit in the last 30 days, with justifications, for compliance review.

* Authority-limit utilization — per analyst, how often they hit their ceiling. Signal for promotion or training.

### **11.2.5 Manager dashboard (Phase 2\)**

* Customer-wide KPIs — total recovery dollars MTD/YTD, average days-to-recovery, recovery-to-paid ratio, leakage estimate.

* Authority limits administration — set or modify per-analyst limits with effective dates; changes audit-logged.

* Escalations — supervisor approval queue overflow, claims aging past customer SLA, complaints.

* Supervisor performance — approval response time, QC sample disagreement rate, team output.

* Template governance — pending template changes, retired templates, version diff review.

### **11.2.6 What MVP must do correctly to not block Phase 2**

Three design choices in MVP keep Phase 2 from being a rewrite:

* Every meaningful state change (status transition, content edit, recovery closure, settlement acceptance) writes to an append-only audit log with actor, timestamp, target, and justification fields. The justification field is optional in MVP but the column exists.

* Action validation is centralized — there is one place where 'is this action allowed?' is evaluated. In MVP that one place returns true if the user is signed in. In Phase 2, it consults the authority-limits table. Calling code does not change.

* Roles are stored on the user record and consulted through a single helper. The full four-role set (Analyst, Supervisor, Manager, Admin) is wired in MVP. Adding future role variants or splitting an existing role means adding rows and updating the helper, not rewriting permission checks scattered through the UI.

### **11.2.7 Per-file authority elevation (added v0.6)**

The authority model gained a third layer of override in addition to Job-Level defaults (set by Manager) and per-analyst overrides (Supervisor proposes, Manager approves). The per-file authority elevation is granted on a single claim only, scoped to specific actions (settlement, demand, lien reduction, closure), and auto-expires when the claim closes. The Supervisor grants unilaterally up to the supervisor ceiling (a separate customer-level configuration); proposed ceilings above the supervisor ceiling route to Manager via the Approvals queue (FILE\_AUTHORITY\_GRANT type). This solves the case where an analyst owns the file and has the relationship but the dollars exceed their level’s authority. The prototype demonstrates the full lifecycle (grant, modify, revoke, expiry) on the Claim Detail header. See Dashboard Spec §4.3 for the implementation detail.

### **11.2.8 Coaching notes (added v0.6)**

Notes that Supervisors keep on their analysts (and that Managers keep on their supervisors) are explicitly NOT part of the audit log. They live in a separate role-private store. Supervisor notes about an analyst are visible to that analyst’s supervisor and to the manager; manager notes about a supervisor are visible to the manager only. This separation is deliberate: candid coaching observations should not be intermingled with formal decision records. If a note needs to become a formal audit-log entry, the supervisor or manager elevates it manually.

# **12\. Validation Before Heavy Build** {#12.-validation-before-heavy-build}

The work between today and the next round of engineering investment is customer development, not feature development. The MVP prototype already in the project folder is the demo asset for these conversations. Pilot state is Wisconsin (see Section 1.4 of the GTM Roadmap for rationale).

* Conduct 15–25 structured interviews across the three modes: 5–8 carrier in-house COB/subrogation leaders, 5–8 subrogation vendor analysts and managers, and 3–5 independent recovery vendors or self-funded employer admins.

* Capture per-interview: current FTE count on COB/recovery, average recoveries per analyst per month, average dollar per recovery, what % of recoveries are auto-related, what % use COB Smart already, what % use Rawlings/Optum/Phia, and stated willingness-to-pay across the three pricing models.

* Build an ROI calculator that uses interview averages to project annual savings for a prospective customer; this becomes the leave-behind for the sales conversation.

* Confirm or revise the competitor positioning in Section 1.3 from interview data.

* Decide pilot tenant and pilot state before further engineering investment.

# **13\. Roadmap** {#13.-roadmap}

## **13.1 Phase 1 — MVP (months 0–4)** {#13.1-phase-1-—-mvp-(months-0–4)}

* Decision-support mode only; analyst sign-off required for every determination.

* File-drop intake; manual upload; AI-assisted extraction with human confirm.

* Engine v0.3 (decision tree above, including Wisconsin overlay).

* Wisconsin pilot tenant; one carrier, TPA, or self-funded employer in WI.

* Covers workflow phases 1, 2, 3, 5, 6, and partial 8\.

## **13.2 Phase 2 — Workflow completion & production-ready (months 4–9)** {#13.2-phase-2-—-workflow-completion-&-production-ready-(months-4–9)}

* Phase 4 (Liability & Fault Investigation) module — structured liability inputs, comparative-negligence calculator surfaced as its own analyst workspace, recoverable-damages display feeding directly into the Recovery Tracker.

* Phase 7 (Ongoing Claim Management) module — reserves, supplemental EOB intake, duplicate-payment detection, ongoing-treatment monitoring with re-triage.

* Phase 8 negotiation workspace — lien tracking, made-whole evaluator, settlement document workflow, common-fund pro-rata applied at the moment of settlement.

* Phase 9 (Audit & Closure) module — closure verification checklist, audit-ready summary export, claim lock with reopen reason.

* Production hardening of the Authority & Approval Model (see Section 11.2). The four-role model and approval queue surfaces are in MVP; Phase 2 adds the centralized canPerform enforcement chokepoint, hard-routing of approvals to the analyst's team supervisor with SLA timers and out-of-office delegation, and bulk approve/reject operations.

* Supervisor surfaces production hardening. The Management workspace tabs (Approvals, QC, KPIs, Team) are in MVP at skeleton level; Phase 2 wires them to live data feeds, adds drill-down analytics on authority-limit utilization, and surfaces SLA-breach notifications.

* Manager surfaces production hardening. The Manager view of the Management workspace (customer-wide KPIs, Job Level definitions, supervisor performance, template governance, escalations) is in MVP at skeleton level; Phase 2 wires real KPI calculations against production data, formalizes template governance with version diff review, and adds the leakage estimator.

* SOC 2 readiness; HIPAA BAAs in production; monitoring/alerting.

* X12 and FHIR R4 adapters. CAQH COB Smart consumption.

* Configurable workflow per tenant; branded document templates; vendor white-label.

* Auto-determine high-confidence calls; route low-confidence and high-dollar to review queue.

## **13.3 Phase 3 — Scale & expand (months 9–18)** {#13.3-phase-3-—-scale-&-expand-(months-9–18)}

* Native Facets/QNXT/HealthRules adapters.

* Hospital revenue-cycle and law-firm tenant types onboarded.

* Predictive recovery-likelihood model trained on closed cases.

* Graduation tier: fully autonomous determination with sampled QA, premium price band.

* React Native or wrapped iOS/Android for field and management use.

* Expansion beyond Wisconsin: Illinois, Minnesota, Iowa (adjacent markets) and Michigan (no-fault state to exercise the dormant PIP path).

# **14\. Open Questions for Jim** {#14.-open-questions-for-jim}

* Pilot tenant identity — which Wisconsin target from the Phase 0 target list does the first conversation aim at? Common Ground Healthcare Cooperative is verified (2026-05-17) as the marketplace QHP issuer behind CareSource WI plans; Brookfield HQ remains to be confirmed but the WI market presence is no longer in question. Diversified Benefit Services (Hartland) is the leanest first-pilot candidate.

* Independent-vendor mode: does COB Flow handle billing/collections, or only identification \+ correspondence with payment flowing through your back office?

* White-label requirements for vendor tenants — full domain/logo/color rebranding, or co-branded acceptable for v1?

* Pricing preference for independent-vendor mode: pure contingency (20–30%), hybrid (low monthly \+ lower contingency), or tiered dollar bands?

* LLM provider: Anthropic Claude vs. OpenAI vs. on-prem fine-tune? BAA availability and enterprise procurement constraints will drive this.

* Wisconsin OCI: confirm via Phase 0 attorney scoping call whether the independent-vendor mode requires third-party recovery vendor registration.

* AI document parsing — lifecycle model (deferred from prototype scope). Spec §7 establishes the Option B parsing pipeline; Onboarding Playbook Phase 3 (¶53) locates the activity in the onboarding arc. Lifecycle around the extracted language: extraction is primarily a one-time activity per plan at customer onboarding (or when COB Flow represents a plan as vendor); the result becomes a shared plan-language library available to all analysts at the customer so per-claim work consults the library rather than re-extracting; re-extraction is triggered only on plan change; for legal preservation, the plan language used at the time of a determination is captured as an immutable snapshot attached to the claim file and retained with it regardless of subsequent library updates; the library itself is versioned with date/time stamps so prior interpretations remain accessible. Pair follow-up: expand Onboarding Playbook Phase 3 (¶53) to incorporate this lifecycle when the Playbook reaches v0.2.

* Send-to-Work-Comp routing path. Prototype surface implemented at the Claim Detail Overview tab — analyst redirects a claim out of COB scope when investigation reveals a work-vehicle or work-capacity scenario; action requires justification and writes to the audit log. Front-end triage should filter most of these cases at intake (logic deferred). Downstream behavior is deferred: where the redirected file routes (customer WC carrier, WC TPA, or carrier intake for re-routing), what state the claim holds after redirect (locked from further COB action, retained for legal preservation per the closed-file archival entry), and what data exchange occurs with the receiving WC system.

* Closed-file archival (deferred from prototype scope). Legal retention requirement: when a claim file closes — by COB completion, withdrawal, or Send-to-Work-Comp redirect — the file is retained rather than deleted. Retention period is open: Wisconsin record-retention rules, HIPAA minimums, and customer SOW obligations all factor in; the Phase 0 attorney scoping call should resolve. Archive state: read-only on the claim record, supporting documents, audit log, and any plan-language snapshots captured per the AI document parsing entry; archived files remain indexed and searchable; every read is itself an audit event. Open: access scope (file owner, Manager, Admin), reopen path (analyst-initiated reopen, litigation hold, audit response), and whether the prototype needs a 'Restore from archive' surface. Storage-tier choice (hot DB vs. colder storage) is implementation detail and not specified here.

# **15\. Research, Standards & References** {#15.-research,-standards-&-references}

## **15.1 Primary government and standards sources** {#15.1-primary-government-and-standards-sources}

**CMS National Training Program — Module 5: Coordination of Benefits (2015).** Source for the MSP working-aged, disability, and ESRD rules, the EGHP group-size thresholds, the TFL and TRICARE distinction, the conditional payment doctrine for no-fault and liability, and the COBRA-vs-Medicare hierarchy. Engine v0.2 reflects these. [\[link\]](https://www.cms.gov/outreach-and-education/training/cmsnationaltrainingprogram/downloads/coordination-of-benefits-workbook.pdf)

**CMS Coordination of Benefits and Third Party Liability (COB/TPL) in Medicaid — Handbook (2020).** Source for the Medicaid payer-of-last-resort doctrine, the cost-avoidance vs. pay-and-chase workflow distinction, the Diagnosis and Trauma Code Edits triage term we adopted, Ahlborn limitations on tort recovery, and Assignment of Rights mechanics. [\[link\]](https://www.medicaid.gov/medicaid/eligibility/downloads/cob-tpl-handbook.pdf)

**CAQH COB Smart Webinar (Capital BlueCross / CAQH, July 2016).** Source for the $800M industry COB-inefficiency figure, the 5.1% average COB discovery rate, and confirmation that COB Smart explicitly excludes subrogation — the wedge that justifies COB Flow as a complementary downstream product. [\[link\]](https://www.caqh.org/sites/default/files/solutions/cob-smart/cob-smart-webinar-CBC-final-072116.pdf)

**CMS Medicare Secondary Payer overview.** Official MSP guidance, conditional payment process, and current contractor (BCRC) details. [\[link\]](https://www.cms.gov/medicare/coordination-benefits-recovery/overview/secondary-payer)

**X12 EDI Standards.** Authoritative source for 270/271 eligibility, 837 claim, 835 remittance transactions we will support in Phase 2\. [\[link\]](https://x12.org/)

## **15.2 Industry players (research and partnership targets)** {#15.2-industry-players-(research-and-partnership-targets)}

**Cotiviti.** Payment integrity and claims intelligence. Adjacent to COB Flow; potential partner. [\[link\]](https://www.cotiviti.com/)

**Rawlings.** Healthcare recovery and subrogation services. Direct competitor on subrogation/recovery. [\[link\]](https://www.rawlingscompany.com/)

**Zelis.** Payments and claims workflow. Adjacent payment integrity. [\[link\]](https://www.zelis.com/)

**Optum.** Owns Equian. Direct competitor on subrogation and payment integrity at scale. [\[link\]](https://www.optum.com/)

**Healthcare Financial Management Association (HFMA).** Industry association; useful for hospital revenue-cycle audience research (Phase 3). [\[link\]](https://www.hfma.org/)

## **15.3 Case-law cited in the engine** {#15.3-case-law-cited-in-the-engine}

* Buckley v. American Medical Security, 973 F. Supp. 1005 — self-funded ERISA plan's coordination clause vs. auto excess clause, circuit-split outcome.

* McGurl v. Trucking Employees of N. Jersey Welfare Fund — escape clauses unenforceable under federal common law between two ERISA plans.

* PM Group Life Ins. Co. v. Western Growers Assurance Trust — reinforces McGurl on escape-clause unenforceability.

* Catholic Diocese of Biloxi Supplemental Medical Plan — birthday rule application and dependent-coverage primacy.

* Great-West Life & Annuity v. Knudson, 534 U.S. 204 — limits on plan recovery remedies under ERISA § 502(a)(3).

* Arkansas Dep't of Health & Human Servs. v. Ahlborn, 547 U.S. 268 — limits on Medicaid recovery from tort settlement proceeds.

## **15.4 Wisconsin-specific case-law and statutes (pilot state)** {#15.4-wisconsin-specific-case-law-and-statutes-(pilot-state)}

* Rimes v. State Farm Mut. Auto. Ins. Co., 106 Wis. 2d 263 (1982) — establishes the made-whole doctrine in Wisconsin.

* Vogt v. Schroeder, 129 Wis. 2d 3 (1986) — reinforces made-whole doctrine, including in workers' compensation context.

* Petta v. ABC Ins. Co., 692 N.W.2d 639 (2005) — applies made-whole doctrine; clarifies subrogation claim handling.

* Wis. Stat. § 632.32 — auto policy form and content; mandates UM coverage; establishes UIM framework; Med-Pay is optional.

* Wis. Stat. § 631.43 — subrogation rights under contracts of insurance; operates alongside made-whole doctrine.

* Wis. Stat. § 895.045 — modified comparative negligence with 51% bar; drives the recoverable-damages calculation.

* Wis. Admin. Code § Ins 3.40 — Wisconsin Coordination of Benefits administrative rule. Codifies the NAIC-model order of benefit determination at § Ins 3.40(11)(b). § Ins 3.40(18) (coordination with noncomplying plans) authorizes the recovery workflow when the auto carrier — carved out as a “traditional automobile ‘fault’ contract” — does not engage in good-faith coordination; this is the regulatory anchor for COB Flow’s recovery positioning. Full text saved in the project folder as Wisconsin Legislature\_ Ins 3.40(11)(a).pdf.

* Wis. Stat. § 632.897(3)(a) — state continuation coverage statute referenced by § Ins 3.40(11)(b)5m (continuation coverage rule); flagged as engine v1.1 gap.

## **15.5 Related project documents** {#15.5-related-project-documents}

* COB\_Flow\_WI\_Workflow\_v1.0.docx — canonical 9-phase analyst workflow for Wisconsin auto-related COB claims. Source of truth for what the platform must support across the full claim lifecycle.

* COB\_Flow\_GTM\_Roadmap.docx — 6-phase go-to-market plan for the Wisconsin pilot through first commercial tenant.

* COB\_Flow\_Target\_List\_53045.xlsx — Phase 0 target list anchored on Brookfield, WI, with status tracker and outreach log.

* COB\_Flow\_SPD\_Review\_Template.docx — one-page Word doc for marking up a prospect’s SPD/EOC during Phase 1 discovery. Captures plan type, COB language, subrogation provisions, recovery legal handling model (in-house counsel / outside firm / TPA), red flags, and tailored interview questions. Front-door instrument for the discovery-to-pilot pipeline.

* COB\_Flow\_Onboarding\_Playbook\_v0.1.docx — two-part doc combining an internal phase-by-phase playbook with a customer-facing implementation guide. Covers all three deployment modes (carrier in-house, vendor/TPA, independent vendor service) across seven phases from pre-contract through ongoing operations.

* **COB\_Flow\_Dashboard\_Spec\_v0.1.docx (added in v0.6)** — technical companion to this spec. Documents the four-role workspace model, Job Levels with default authority bands, per-analyst and per-file authority elevation flows, coaching notes architecture, approval queue types, audit architecture (system events vs. PHI audit vs. coaching notes), and the implementation issues that need resolution before pilot. Section 11.2 of this spec describes the strategic model; the Dashboard Spec carries the implementation-level detail and the open questions for the implementation plan.

* COB\_Flow\_Auto\_COB\_Syllabus.docx — 12-module learning syllabus for someone with general health-insurance knowledge moving into Auto COB. Sequential modules from auto-insurance basics through state law overlays, ERISA preemption, the PI attorney workflow, end-to-end recovery operations, documents and EDI standards, compliance, and continuing development.

* COB\_Flow\_Demand\_Letter\_Template.docx — fillable letter template with merge fields. Updated 2026-05-17 to cite Wis. Admin. Code § Ins 3.40, including § Ins 3.40(18) (coordination with noncomplying plans), as recovery authority in WI matters.

* Wisconsin Legislature\_ Ins 3.40(11)(a).pdf — saved PDF of the full Wis. Admin. Code § Ins 3.40 text (subs (3)(i) through (19) plus Appendix A model COB provision). The COB rule the engine implements; canonical citation reference.

# **Appendix A: Glossary** {#appendix-a:-glossary}

This appendix defines the key terms, abbreviations, and product-specific names used throughout this spec. Entries are organized by domain (legal, healthcare, product, technical) and listed alphabetically within each section. Where a term has a specific role in the COB Flow engine, workflow, or roadmap, the entry notes the role explicitly so the glossary doubles as a reference for new readers.

## **Coordination of Benefits — Core Concepts** {#coordination-of-benefits-—-core-concepts}

**Active-over-retiree** — COB primacy rule: when a member is covered by an active-employee plan and a retiree (or COBRA) plan, the active-employee plan pays first. Exception: ESRD during the 30-month coordination period.  
**Birthday rule** — COB primacy rule for dependent children covered under both parents' plans without a QMCSO. The plan of the parent whose birthday (month/day) falls earlier in the calendar year is primary. Tie-break: longer coverage. Source: NAIC Model COB Regulation § 6(D)(3).  
**COB** — Coordination of Benefits. The set of rules that decide which payer is primary, secondary, and tertiary when a patient has multiple sources of health coverage. Distinct from Coordination of Care (clinical workflow), which shares the abbreviation casually but is a different domain.  
**COB clause type** — The classification of a plan's coordination language: NAIC-standard, excess (plan pays only what other plans don't cover), escape ("this plan pays only what is not covered by any other plan"), or non-conforming. The classification drives which engine rule applies.  
**Common-fund doctrine** — Equitable doctrine requiring a subrogating insurer to pay its pro-rata share of attorney fees when the insurer benefits from a fund created by the claimant's counsel. In Wisconsin, established by Petta v. ABC Ins. Co. The engine computes (plan recovery / total recovery) × attorney fee.  
**Coordination of Care (CoC)** — Clinical workflow between providers (PCP, specialist, behavioral health). Not the same as Coordination of Benefits despite the shared "coordination" label. Anthem's CoC newsletters are not COB documents.  
**Cost avoidance** — Medicaid TPL workflow in which the provider bills the third-party payer first, before Medicaid pays anything. Contrast with pay-and-chase.  
**Employee-over-dependent** — COB primacy rule: when one plan covers the patient as an employee/subscriber and another covers the same patient as a dependent, the employee plan is primary. Source: NAIC Model COB Regulation § 6(D)(1).  
**Escape clause** — Plan language stating the plan pays only what is not covered by any other plan. Unenforceable as between two ERISA plans under federal common law (McGurl; PM Group Life).  
**Excess clause** — Plan language stating the plan pays only after all other coverage is exhausted. Common in auto policies; interacts with ERISA preemption analysis when paired with a self-funded health plan.  
**Longer/shorter rule** — Fallback COB rule when no higher-priority rule applies: the plan covering the patient longest is primary. Source: NAIC Model COB Regulation § 6(D)(7). The engine flags any determination resting on this rule for analyst review.  
**Made-whole doctrine** — Equitable doctrine that an insurer's subrogation right does not attach until the insured has been fully compensated for the loss. In Wisconsin, established by Rimes v. State Farm; reinforced in Vogt and Petta. ERISA-preempted as against properly drafted self-funded plan documents (Sereboff line).  
**NAIC Model COB Regulation** — National Association of Insurance Commissioners' model regulation that most states (including Wisconsin via § Ins 3.40) adopt or adapt for COB ordering rules.  
**Pay-and-chase** — Medicaid TPL workflow in which Medicaid pays first, then recovers from the third-party payer. Contrast with cost avoidance.  
**Payer of last resort** — Federal-law doctrine that Medicaid pays after all other identified third-party payers are exhausted. Source: Social Security Act § 1902(a)(25); 42 CFR 433 Subpart D.  
**QMCSO** — Qualified Medical Child Support Order. A court order designating a parent's health plan as primary for a child; overrides the birthday rule. Source: ERISA § 609(a); 29 U.S.C. § 1169\.  
**Separated-parents rule** — COB fallback for children of separated parents with no court order: custodial parent's plan first, then custodial parent's spouse, then non-custodial parent, then non-custodial parent's spouse.  
**Subrogation** — The legal right of an insurer (or other payer) to step into the insured's shoes and recover from a responsible third party for amounts the insurer paid on the insured's behalf.

## **Medicare & Federal Programs** {#medicare-&-federal-programs}

**BCRC** — Benefits Coordination & Recovery Center. The CMS contractor that handles Medicare Secondary Payer coordination and recovery from primary plans.  
**COBRA** — Consolidated Omnibus Budget Reconciliation Act. Federal law that lets former employees continue group health coverage at their own expense. In COB ordering, treated as continuation coverage; secondary to active-employee plans except during ESRD coordination period.  
**Conditional payment** — Medicare payment made when another payer (no-fault, liability, workers' comp) is expected to pay primary but has not paid within 120 days. Medicare recovers conditionally paid amounts at settlement. Source: Medicare Modernization Act of 2003, Title III § 301\.  
**EGHP** — Employer Group Health Plan. The employer-sponsored coverage that pays primary to Medicare under the MSP working-aged, disability, and ESRD rules when applicable group-size thresholds are met.  
**ESRD** — End-Stage Renal Disease. Qualifies a person for Medicare regardless of age; triggers the 30-month MSP coordination period during which EGHP/COBRA pays primary.  
**Medicaid TPL** — Medicaid's Third Party Liability program. The set of rules and workflows for identifying and recovering from other liable payers before or after Medicaid pays.  
**MSP** — Medicare Secondary Payer. The statutory regime (42 U.S.C. § 1395y(b)) under which Medicare is secondary to specified other payers, including EGHPs in working-aged, disability, and ESRD scenarios, plus liability, no-fault, and workers' comp.  
**TFL** — TRICARE for Life. Wraparound coverage for Medicare-eligible military retirees and dependents; Medicare is primary, TFL is secondary.  
**TPL** — Third Party Liability. General term for coverage by a non-Medicaid payer that should pay before Medicaid; used heavily in CMS Medicaid documentation.  
**TRICARE** — DoD-administered health program for service members and families. Distinct from TFL: regular TRICARE coordinates under EGHP rules for active-duty families, while TFL is the Medicare-wraparound product.  
**Working aged** — MSP scenario: member is 65 or older and covered as an active employee (or spouse of one) where the employer has 20+ employees. EGHP is primary, Medicare is secondary. Source: 42 U.S.C. § 1395y(b)(1)(A); 42 C.F.R. § 411.170.

## **ERISA & Federal Law** {#erisa-&-federal-law}

**Ahlborn limitation** — Constraint from Ark. Dep't of Health & Human Servs. v. Ahlborn, 547 U.S. 268, that limits a Medicaid agency's recovery from a tort settlement to the portion attributable to medical expenses, not the entire settlement.  
**Assignment of Rights** — Medicaid requirement that recipients assign their rights to medical support and third-party payments to the state, enabling Medicaid recovery.  
**ERISA** — Employee Retirement Income Security Act of 1974\. Federal law governing private-sector employee benefit plans, including self-funded health plans. Source of the preemption doctrine that defeats state COB and made-whole law against properly drafted plan documents.  
**ERISA § 502(a)(3)** — ERISA's civil enforcement provision, limiting plan-recovery remedies to equitable relief. Source of the Great-West v. Knudson holding constraining recovery against general assets.  
**ERISA § 514** — ERISA's preemption clause, the basis for state-law preemption of conflicting COB and subrogation rules as applied to ERISA plans.  
**Federal common law** — Body of case law developed by federal courts to fill ERISA gaps. Source of the escape-clause unenforceability rule between two ERISA plans (McGurl; PM Group Life).  
**Self-funded plan** — Employer health plan in which the employer bears the financial risk of claims, often administered by a TPA. Subject to ERISA but exempt from most state insurance regulation. Made-whole and common-fund doctrines preempted as against properly drafted plan documents (Sereboff line).  
**Sereboff line** — The line of Supreme Court ERISA-preemption cases (Sereboff v. Mid Atlantic Medical Services; Great-West v. Knudson; US Airways v. McCutchen) that defines the scope of plan recovery from settlement proceeds and limits state-law equitable defenses (made-whole, common-fund) when plan documents are drafted properly.

## **Wisconsin Legal Framework** {#wisconsin-legal-framework}

**Comparative negligence (Wisconsin, modified)** — Wisconsin's 51% bar comparative-negligence rule: a claimant who is 51% or more at fault recovers nothing; otherwise damages are reduced by the claimant's % fault. Source: Wis. Stat. § 895.045. Drives the engine's recoverable-damages calculator.  
**Complying plan** — Under Wis. Admin. Code § Ins 3.40, a plan that complies with the WI COB rule. Eligible to use § Ins 3.40(18) when paired with a noncomplying plan.  
**Noncomplying plan** — Under Wis. Admin. Code § Ins 3.40, a plan that does not comply with the WI COB rule. Traditional automobile "fault" contracts are carved out as noncomplying for these purposes, enabling the complying plan to advance benefits and recover via subrogation.  
**OCI** — Wisconsin Office of the Commissioner of Insurance. The state regulator of insurance carriers and certain third-party recovery vendors in Wisconsin.  
**§ Ins 3.40(11)(b)** — Provision of Wisconsin's COB rule that codifies the NAIC-model order of benefit determination. The engine's primacy rules track this subsection.  
**§ Ins 3.40(11)(b)5m** — Continuation coverage rule. COBRA and Wis. Stat. § 632.897(3)(a) continuation coverage get a distinct primacy slot between active/inactive employee and longer/shorter. Documented engine gap; Phase 2/3 scope.  
**§ Ins 3.40(11)(b)6m** — 24-hour bridge rule. When applying the longer/shorter fallback, two plans are treated as one continuous period if the second begins within 24 hours of the first ending. Documented engine gap; Phase 2/3 scope.  
**§ Ins 3.40(12)** — COB benefits cap math. Allows the secondary to reduce benefits so total payments do not exceed allowable expenses, with credit-savings tracked across a claim determination period. Documented engine gap; the larger Phase 2/3 item.  
**§ Ins 3.40(18)** — Coordination with noncomplying plans. Authorizes a complying plan to advance benefits and recover via subrogation when a noncomplying plan (including auto carriers carved out as "traditional automobile 'fault' contracts") does not engage in good-faith coordination. The regulatory anchor of COB Flow's recovery positioning.  
**Wis. Admin. Code § Ins 3.40** — Wisconsin Coordination of Benefits administrative rule. Codifies the NAIC-model order of benefit determination and provides the regulatory authority for the COB Flow recovery workflow.  
**Wis. Stat. § 631.43** — Subrogation rights under insurance contracts. Operates alongside the made-whole doctrine.  
**Wis. Stat. § 632.32** — Wisconsin's auto policy regulation. Mandates UM coverage, establishes the UIM framework, makes Med-Pay optional.  
**Wis. Stat. § 632.897(3)(a)** — Wisconsin's state continuation coverage statute, referenced by § Ins 3.40(11)(b)5m.  
**Wis. Stat. § 895.045** — Wisconsin's modified comparative negligence statute with the 51% bar.

## **Case Law** {#case-law}

**Ahlborn (Ark. v. Ahlborn)** — 547 U.S. 268 (2006). Limits a Medicaid agency's recovery from a tort settlement to the portion attributable to medical expenses.  
**Buckley v. American Medical Security** — 973 F. Supp. 1005\. Self-funded ERISA plan coordination clause vs. auto excess clause. Outcome is circuit-dependent.  
**Catholic Diocese of Biloxi Supplemental Medical Plan** — Birthday rule application and dependent-coverage primacy precedent.  
**Great-West Life & Annuity v. Knudson** — 534 U.S. 204\. Constrains plan-recovery remedies under ERISA § 502(a)(3) to equitable relief against identifiable funds, not general assets.  
**McGurl v. Trucking Employees of N. Jersey Welfare Fund** — Third Circuit. Escape clauses are unenforceable under federal common law as between two ERISA plans.  
**Petta v. ABC Ins. Co.** — 692 N.W.2d 639 (2005). Wisconsin Supreme Court application of the made-whole doctrine; also informs common-fund treatment.  
**PM Group Life Ins. Co. v. Western Growers Assurance Trust** — Reinforces McGurl on escape-clause unenforceability between ERISA plans.  
**Rimes v. State Farm Mut. Auto. Ins. Co.** — 106 Wis. 2d 263 (1982). Establishes the made-whole doctrine in Wisconsin.  
**Vogt v. Schroeder** — 129 Wis. 2d 3 (1986). Reinforces the made-whole doctrine, including in the workers' compensation context.

## **Auto Insurance & PIP** {#auto-insurance-&-pip}

**Auto med-pay** — Medical Payments coverage on an auto policy. Pays medical expenses for the insured and passengers regardless of fault, up to policy limits. Optional in Wisconsin.  
**Med-Pay** — Shorthand for auto medical-payments coverage. Optional in Wisconsin under Wis. Stat. § 632.32.  
**No-fault state** — A state in which auto insurance is required to provide PIP benefits regardless of fault for accident-related medical expenses. Examples: MI, FL, NY, NJ, PA, HI, KS, KY, MA, MN, ND, UT. Wisconsin is not a no-fault state.  
**PI attorney** — Personal-injury attorney. In COB Flow workflows, the claimant-side counterparty for lien negotiation once a member retains counsel. Distinct from reimbursement-side law firms (plans' subrogation counsel).  
**PIP** — Personal Injury Protection. Mandatory auto coverage in no-fault states for accident-related medical expenses regardless of fault. The engine's mandatory-PIP rule never fires for a Wisconsin claim.  
**Tort state** — A state, like Wisconsin, in which auto-accident liability is determined by fault under tort principles rather than by no-fault PIP rules.  
**UIM** — Underinsured Motorist coverage. Pays insured's damages when the at-fault driver has insufficient liability coverage. Framework established by Wis. Stat. § 632.32.  
**UM** — Uninsured Motorist coverage. Pays insured's damages caused by a driver with no liability insurance. Mandated in Wisconsin under Wis. Stat. § 632.32.

## **Healthcare & Claims Terminology** {#healthcare-&-claims-terminology}

**Demand letter** — Formal correspondence from a payer or recovery party to a responsible third party seeking reimbursement for amounts the payer has paid. In COB Flow, generated from the Demand Letter Template with engine-supplied citations.  
**Diagnosis and Trauma Code Edits** — Medicaid TPL term for ICD-based filters that flag claims with subrogation potential (e.g., diagnoses suggesting an external cause of injury). Used in COB Flow's automated triage.  
**EOB** — Explanation of Benefits. The payer-generated document that itemizes services, charges, what the plan paid, what the patient owes, and denial codes. A primary source for AI-assisted field extraction.  
**EOC** — Evidence of Coverage. A formal plan document describing benefits, exclusions, and member rights, common in HMO and Medicare Advantage products. Reviewed alongside SPDs during prospect discovery.  
**FTE** — Full-Time Equivalent. Standard labor-cost unit used in ROI calculations for COB analyst staffing.  
**Lien** — A claim against settlement proceeds asserted by a payer (or other party) for amounts paid on the insured's behalf. In healthcare recovery, the plan's lien is typically negotiated against the settlement before disbursement.  
**Lien reduction** — Negotiated decrease in the plan's stated lien, often based on made-whole, common-fund, or settlement realities. Subject to analyst authority limits in the COB Flow approval model.  
**QHP** — Qualified Health Plan. A health plan certified to be sold on an ACA marketplace; Common Ground is verified as a Wisconsin marketplace QHP issuer behind CareSource WI plans.  
**Recovery** — The total dollar amount actually collected by the plan or recovery vendor for a given claim or determination period.  
**SPD** — Summary Plan Description. ERISA-required summary of plan terms; reviewed during discovery with the SPD Review Template. Source of COB language and subrogation provisions used to assess fully-insured vs. self-funded posture.

## **COB Flow Product — Modules & Workflow** {#cob-flow-product-—-modules-&-workflow}

**9-phase analyst workflow** — Canonical end-to-end workflow defined in COB\_Flow\_WI\_Workflow\_v1.0.docx. Six phases are in MVP, Phase 8 is partial, and Phases 4, 7, and 9 are Phase 2 of the product roadmap.  
**Claim** — A single member medical event being processed. Core entity in the data model.  
**Claims & Triage** — MVP module covering intake, automated triage, and the new-claim button surface.  
**COB Analyzer** — MVP module that runs the decision engine and surfaces the primary/secondary determination, rationale, citations, and Wisconsin overlay.  
**COB Flow** — The product. Decision-support SaaS for healthcare COB primacy determination, auto med-pay/PIP recovery, claims triage, and recovery correspondence.  
**Communications** — Ad-hoc messaging surface on each claim. Three channels: Email, Fax, Text. Distinct from Correspondence; not a formal-letter workflow.  
**Confidence band** — The engine's expressed confidence in a determination (HIGH / MEDIUM / LOW). Used by the analyst review queue, triggers warnings, and graduates from Phase 2 to gate auto-determination.  
**Correspondence** — Formal letter workflow with a status lifecycle and lock-on-send semantics. Distinct from Communications. Includes Tier 1 (Demand, COB Questionnaire, Medical Records Request, Lien Notice, Member Recovery Notice) and Tier 2 (Follow-up Demand, Subrogation Hold, Lien Reduction Offer, Settlement Acknowledgment).  
**Customer** — The user-facing term in the COB Flow UI for what is implemented as a tenant. Three customer modes are configured: carrier in-house, vendor/TPA, independent vendor service.  
**Determination** — The structured output of the decision engine: primary coverage, secondary coverage, controlling rule, citations, confidence, rationale. Persisted as the canonical record for downstream workflow.  
**Engine output contract** — The fixed JSON shape every determination is emitted in. Includes determinationId, primaryCoverageId, secondaryCoverageId, rule, ruleVersion, citations, confidence, rationale, warnings, reviewRequired, determinedAt, determinedBy.  
**Engine v0.3** — The current rule version of the COB Primacy Decision Engine, including the Wisconsin overlay.  
**Locked correspondence** — Letter that has transitioned to SENT status. Cannot be edited by the originating analyst; Supervisor override with justification is required.  
**Lock override** — Action by Supervisor or above to edit a locked letter or unlock it, with a recorded justification logged to the audit log.  
**Records** — Surface for inbound supporting evidence (medical records, EOBs, plan documents) attached to a claim. Distinct from Correspondence (outbound formal letters).  
**Recovery Tracker** — MVP module for tracking the post-determination recovery cycle: demand status, response, lien tracking, settlement, closure.  
**Rule version** — The semver-style identifier of the decision-engine rule set in effect when a determination was made. Persisted on every determination for reproducibility.  
**Status lifecycle (Correspondence)** — Draft → Sent → Acknowledged → Superseded → Closed. Locked on Send; auto-archives PDF to Records.  
**Tier 1 letter templates** — The five MVP demand-and-notice templates: Demand, COB Questionnaire, Medical Records Request, Lien Notice, Member Recovery Notice.  
**Tier 2 letter templates** — The four follow-up and negotiation templates: Follow-up Demand, Subrogation Hold, Lien Reduction Offer, Settlement Acknowledgment.  
**Wisconsin overlay** — Engine output block emitted for any claim with accident state \= WI. Surfaces made-whole flag (with ERISA-preemption variant), comparative-negligence calculation, common-fund pro-rata, and statutory framework references. Does not change primacy.

## **COB Flow Product — Roles & Authority** {#cob-flow-product-—-roles-&-authority}

**Admin** — System administrator role in the canonical four-role model. Owns customer settings, user provisioning, security policy, integration configuration, compliance attestations, and audit log access. No claim-level workflow actions; read-only on claim and recovery data. Role may be held in addition to Manager at small customers.  
**Analyst** — Primary user role. Drafts and sends correspondence, runs determinations, signs off on engine recommendations. Subject to Job-Level defaults plus per-analyst authority limits and per-file authority elevation.  
**Approval Request** — Structured object routed when an analyst attempts an action above their authority limit. Includes action context, dollar amount, proposed disposition, and analyst justification.  
**Authority limit** — Dollar (or percentage) threshold above which an analyst's action requires supervisor approval before completing. Set per-analyst, per-customer by the Manager role; defaults follow the Analyst's Job Level.  
**Demand authority** — Per-analyst maximum demand dollar amount that can be sent without supervisor review.  
**Job Level** — Default authority band template applied to the Analyst role only. Four levels — Trainee, Junior, Mid, Senior — carry progressively broader defaults for settlement, demand, lien reduction, and closure. The Manager defines the levels and their bands at the customer level; the Supervisor proposes the level for each Analyst and the Manager approves. Supervisors do not have a Job Level; they have a supervisor ceiling configured at the customer level.  
**Letter-content override authority** — Whether the analyst may directly override a locked correspondence lock (Supervisor and above only by default).  
**Lien-reduction authority** — Per-analyst maximum percentage reduction off gross recovery the analyst may offer in negotiation after the automatic common-fund pro-rata.  
**Manager** — Role with customer-wide visibility in the canonical four-role model. Defines Job Levels and their default authority bands, approves per-analyst overrides and above-supervisor file-authority grants, reviews supervisor performance, and governs template publication. Sees customer-wide dashboards.  
**RBAC** — Role-Based Access Control. The canonical four-role model (Analyst, Supervisor, Manager, Admin) layered with Job Levels (Analyst only) and per-analyst plus per-file authority overrides. Role determines workspace access and the action set; Job Level plus overrides determine default authority bands.  
**Recovery-closure authority** — Per-analyst maximum dollar amount at which the analyst can close a recovery without supervisor approval.  
**Senior Analyst** — Informal term for an Analyst at the Senior Job Level. Not a distinct role. See Job Level.  
**Settlement-acceptance authority** — Per-analyst maximum dollar amount of a proposed settlement the analyst can accept and close without supervisor sign-off.  
**Supervisor** — Role in the canonical four-role model. Reviews and approves analyst escalations, performs QC sampling, manages team-level metrics and per-analyst authority overrides, grants per-file authority elevation within the supervisor ceiling, and can override correspondence locks.  
**Template publication authority** — Whether a user may create or modify customer-wide letter templates. Manager and above by default.

## **Deployment Modes & Customer Types** {#deployment-modes-&-customer-types}

**Carrier in-house** — Deployment mode in which a health carrier licenses COB Flow for its own COB and recovery team. Sample tenant in the prototype: Lakeshore Health Plan (WI).  
**Independent vendor service** — Deployment mode in which COB Flow is operated by a small independent recovery shop (e.g., a Brookfield operation) performing recovery on behalf of plan clients. Sample tenant in the prototype: COB Flow Recovery — Brookfield.  
**Multi-tenant** — Architecture in which a single application instance serves multiple isolated customers (tenants) with row-level security on tenant\_id, or schema-per-tenant for enterprise scale.  
**Self-funded employer** — Employer that bears its own health-plan financial risk, often administered by a TPA. ERISA-governed; Phase 1 discovery segment.  
**TPA** — Third Party Administrator. An organization that processes claims and runs benefit operations on behalf of a plan sponsor. A primary COB Flow deployment mode.  
**Vendor / TPA mode** — Deployment mode in which a subrogation vendor or TPA licenses COB Flow to run recovery on behalf of one or more carrier or self-funded clients. Sample tenant in the prototype: Badger State Subrogation Services.  
**White-label** — Vendor-tenant configuration in which the COB Flow product is rebranded under the vendor's name, logo, and domain. Scope (full vs. co-branded) is an open question in Section 14 of the spec.

## **Technical Architecture & Standards** {#technical-architecture-&-standards}

**270 / 271** — X12 healthcare eligibility request (270) and response (271). Used in real-time eligibility lookups, including COB Smart-style returns. Phase 2 integration.  
**835** — X12 healthcare remittance advice transaction. The remittance posted by a payer in response to an 837 claim. Phase 2 ingest.  
**837** — X12 healthcare claim submission transaction. The standard healthcare claim format. Phase 2 ingest.  
**AES-256** — Advanced Encryption Standard at 256-bit key length. The at-rest encryption standard for COB Flow data and source documents.  
**API gateway** — Entry point for REST/JSON traffic, with OAuth2 / SAML SSO for enterprise tenants and mTLS for carrier integrations.  
**Audit log** — Append-only, immutable record of every PHI access, every determination, every LLM extraction confirmation, and every document send. Required by HIPAA and a core compliance asset.  
**BAA** — Business Associate Agreement. HIPAA-required contract with any subprocessor that touches PHI. Required for the LLM provider, cloud hosting, document services, etc.  
**CSV** — Comma-Separated Values. One of the MVP intake formats for claims data.  
**DOCX** — Microsoft Word document format. Output of the document service.  
**docx-js** — JavaScript library for templated DOCX generation, used by the document service.  
**FastAPI** — Python web framework. Candidate application-service stack alongside Node.js.  
**FHIR R4** — Fast Healthcare Interoperability Resources, Release 4\. Healthcare data exchange standard. Phase 2 integration for Coverage, Claim, EOB, and Patient resources.  
**HIPAA** — Health Insurance Portability and Accountability Act. The federal law governing PHI protection. COB Flow handles PHI from day one.  
**JSON** — JavaScript Object Notation. Wire format for the REST API and the engine output contract.  
**JSX** — JavaScript syntax extension used by React. The prototype uses Babel-standalone to compile JSX at runtime.  
**KMS** — Key Management Service (typically AWS KMS). Manages encryption keys for at-rest data.  
**LLM** — Large Language Model. Used in AI-assisted document parsing (Option B) to extract candidate fields from plan docs and EOBs; never used to generate citations or rationale.  
**mTLS** — Mutual TLS. Bilateral certificate authentication used for carrier-integration endpoints.  
**OAuth2** — Authorization framework for the API gateway. SAML SSO is also supported for enterprise tenants.  
**OCR** — Optical Character Recognition. Used (Tesseract or AWS Textract) on scanned documents before LLM extraction.  
**Option B** — Designation for the AI-assisted document parsing approach: LLM extracts candidate fields, analyst confirms each before any value flows into the engine. Distinguished from a hypothetical Option A in which the LLM's output would flow directly into the engine.  
**PHI** — Protected Health Information. Patient data subject to HIPAA. Every PHI read is audit-logged.  
**Postgres** — The application's relational datastore. Configured with row-level security keyed on tenant\_id for shared-cluster customers.  
**REST** — Representational State Transfer. The API gateway's protocol style. Paired with JSON for payloads.  
**Row-level security** — Postgres feature that filters query results by a session-scoped tenant\_id; enforces multi-tenant isolation at the database layer.  
**S3** — Amazon Simple Storage Service. Source-document and generated-PDF storage, paired with KMS for at-rest encryption.  
**SaaS** — Software as a Service. The COB Flow delivery model.  
**SAML** — Security Assertion Markup Language. Enterprise SSO protocol supported at the API gateway.  
**Schema-per-tenant** — Multi-tenancy model in which each tenant gets its own database schema. Used for enterprise customers requiring stronger isolation than row-level security.  
**SFTP** — SSH File Transfer Protocol. The intake transport for daily/weekly CSV/JSON extracts.  
**SLA** — Service Level Agreement. Customer-contracted response or resolution times; surfaced in the Phase 2 approval-queue escalation policy.  
**SLO** — Service Level Objective. Internal target (e.g., reviewer-disagreement rate in periodic extraction audits).  
**SOC 2 Type II** — Service Organization Control 2, Type II. Operational-controls audit standard required for enterprise carrier sales. 9–12 month readiness window after MVP.  
**SPA** — Single-Page Application. The COB Flow web client is a React SPA with TailwindCSS, responsive across desktop and mobile browsers.  
**SSO** — Single Sign-On. Enterprise-tenant authentication via SAML.  
**TailwindCSS** — Utility-first CSS framework used by the web client.  
**Tenant** — Internal/architectural term for a customer. User-facing UI says "Customer."  
**Tesseract** — Open-source OCR engine. One of the two OCR options alongside AWS Textract.  
**TLS** — Transport Layer Security. TLS 1.2+ in transit is required by COB Flow's HIPAA stance.  
**WeasyPrint** — Python PDF generation library; an alternative to docx-js for templated output.  
**X12** — ASC X12 EDI standards body. Source of the 270/271/837/835 healthcare transactions.

## **Industry Organizations & Competitors** {#industry-organizations-&-competitors}

**Anthropic Claude** — One LLM provider option for the extraction service. Final selection subject to BAA availability and enterprise procurement constraints.  
**AWS Textract** — Amazon's OCR service; one option for COB Flow's extraction service.  
**CAQH** — Council for Affordable Quality Healthcare. Industry collaborative; runs COB Smart.  
**CAQH COB Smart** — Industry utility solving upstream commercial COB eligibility discovery at scale. Excludes subrogation, which is COB Flow's wedge. Phase 2 integration consumes COB Smart match files as a Coverage source.  
**CareSource WI** — Wisconsin marketplace QHP plans, with Common Ground Healthcare Cooperative as the verified marketplace issuer (2026-05-17).  
**CMS** — Centers for Medicare & Medicaid Services. Primary regulator and source of MSP and Medicaid TPL guidance.  
**Common Ground Healthcare Cooperative** — Wisconsin marketplace QHP issuer behind CareSource WI plans. Phase 0 pilot-tenant candidate. Brookfield HQ remains to be confirmed.  
**Cotiviti** — Payment integrity and claims intelligence vendor. Adjacent to COB Flow; potential partner.  
**Diversified Benefit Services** — Hartland-based TPA. The leanest Phase 0 pilot-tenant candidate.  
**Equian** — Healthcare recovery business; now part of Optum.  
**Facets** — TriZetto / Cognizant payer-administration platform. Phase 3 native-adapter target.  
**HealthEdge** — Payer-administration platform. Phase 3 native-adapter target.  
**HealthRules Payer** — HealthEdge's payer-administration product. Phase 3 native-adapter target.  
**HFMA** — Healthcare Financial Management Association. Industry association relevant to hospital revenue-cycle audience research in Phase 3\.  
**OpenAI** — One LLM provider option alongside Anthropic Claude. Procurement subject to BAA availability.  
**Optum** — Subrogation and payment-integrity competitor at scale; owns Equian.  
**ProHealth Care** — Waukesha-area health system. Phase 0 target on the WI 53045 target list.  
**QNXT** — Cognizant payer-administration platform. Phase 3 native-adapter target.  
**Quartz** — Wisconsin commercial carrier; potential carrier-in-house customer in subsequent target waves.  
**Rawlings** — Direct competitor in healthcare recovery and subrogation services.  
**TriZetto** — Cognizant payer-administration ecosystem (Facets, QNXT). Phase 3 native-adapter target.  
**Zelis** — Payments and claims workflow vendor. Adjacent payment integrity, partnership candidate.  
— end of v0.7 —