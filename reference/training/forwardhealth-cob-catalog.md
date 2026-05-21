# ForwardHealth COB Training Catalog

External training materials from Wisconsin's ForwardHealth (Medicaid) program. ForwardHealth is the canonical source for Wisconsin Medicaid COB policy and operational guidance — directly relevant to the COB Flow build given the Wisconsin pilot.

Master catalog page: https://www.forwardhealth.wi.gov/WIPortal/cms/public/trainings/home

## In this directory

- `ForwardHealth_COB_Training_final.pdf` — master combined training deck (31 slides; Parts 1–3 of the modular catalog below)
- `ForwardHealth_all_coord_policy.pdf` — comprehensive ForwardHealth coordination policy reference

## Track 1 — Overview of ForwardHealth COB and the Commercial Insurance Process

General-audience training for all providers. Modules available as video + PDF.

| Module | Video | PDF |
|---|---|---|
| General Concepts of COB | https://vimeo.com/1185528687 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_cob_general_pdf |
| The COB Process | https://vimeo.com/1185522422 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_cob_process_pdf |
| Other COB Policy Reminders and Resources | https://vimeo.com/1185538490 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_cob_other_policy_pdf |

## Track 2 — Special Considerations for Behavioral Treatment Providers re: COB

Specialty track focused on behavioral treatment claims, but COB-specific.

| Module | Video | PDF |
|---|---|---|
| Key Concepts of BT Claims and Prior Authorization Requests | https://vimeo.com/1185557873 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_btp_pa_pdf |
| The COB Process Specific to Behavioral Treatment Services | https://vimeo.com/1185559099 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_btp_cob_pdf |
| Other Coordination of Benefits Processes to Consider | https://vimeo.com/1185571166 | https://www.forwardhealth.wi.gov/WIPortal/Subsystem/Public/ExternalPage.aspx?id=training_btp_other_pdf |

## Related materials

- **ForwardHealth Update 2022-54** (policy memo, not training): https://www.forwardhealth.wi.gov/kw/pdf/2022-54.pdf
- **2025 ForwardHealth Portal Other Insurance Claims Training (CLTS)**: https://vimeo.com/1109157337

## Authoritative ongoing reference — ForwardHealth Online Handbook

The training above codifies workflows; the Online Handbook is the canonical policy text and is the source of truth for any policy details the engine needs to track:

https://www.forwardhealth.wi.gov/WIPortal/Subsystem/KW/Display.aspx

Topics directly relevant to the COB engine and document automation:

| Topic # | Subject |
|---|---|
| 253 | Payer of Last Resort |
| 255 | Primary Payer / Secondary Payer |
| 538 | Cost Sharing |
| 596 | Exhausting Commercial Health Insurance Sources |
| 601 | Definition of Commercial Health Insurance |
| 603 | Services Not Requiring Commercial Health Insurance Billing |
| 604 | Non-Reimbursable Commercial Health Insurance Services |
| 605 | Other Insurance Indicators |
| 769 | Services Requiring Commercial Health Insurance Billing |
| 844 | Claims Denied by Commercial Health Insurance |
| 4942 | Reporting Discrepancies |
| 12877 | Real-Time Claim Submission Requirements for COB |
| 18497 | Explanation of Medical Benefits Form Requirement |

## Key operational concepts (from the master training deck)

- **Payer of last resort.** ForwardHealth pays only after all other third-party resources have met their legal obligation. Providers must make a reasonable effort to exhaust other coverage before submitting to ForwardHealth.
- **Provider-based billing.** ForwardHealth recoups paid claims when other coverage is discovered or made retroactive after payment. This is the post-payment subrogation pattern the COB Flow engine handles.
- **Third-party liability (TPL).** The legal obligation of third parties to pay part or all of medical assistance expenditures.
- **Submission mechanics.** 837 EDI, Portal DDE, PES software, or paper with the EOMB form (F-01234) required when other insurance was billed.
- **Discrepancy reporting.** Form F-01159 — relevant for audit trail when claim feed data contradicts EVS lookups.

## How to use this catalog

- Agents working on COB Flow features can reference these materials when the spec uses terms like "payer of last resort," "TPL," "EOMB," "Other Insurance Indicators."
- When the engine ports in Phase G, the operational concepts above should map cleanly to engine rules.
- The Online Handbook is the authoritative source for current policy text — fetch handbook topics by number when authoritative citation is needed.

Last updated: 2026-05-21.
