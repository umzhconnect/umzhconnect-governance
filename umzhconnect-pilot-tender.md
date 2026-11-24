# UMZH Connect Pilot Tender: Digital Referrals from General Practice

**Scope:** Call for technology providers to implement and pilot digital referrals according to the UMZH Connect Implementation Guide, with part-funding by UMZH Connect.
**Status:** Draft / v0.1 — for Steering Committee review
**Date:** 2026-11-17
**Issuer:** UMZH Connect Steering Committee · **Procedure owner:** Technical Office

> Items marked **[TBD]** are open decisions for the Steering Committee; see
> [§13 Open items](#13-open-items).

---

## Table of Contents

1. [Summary](#1-summary)
2. [Background](#2-background)
3. [Scope of the pilot](#3-scope-of-the-pilot)
4. [Who can apply](#4-who-can-apply)
5. [Funding](#5-funding)
6. [Approval requirements](#6-approval-requirements)
7. [Award criteria](#7-award-criteria)
8. [Selection procedure](#8-selection-procedure)
9. [Governance of the tender](#9-governance-of-the-tender)
10. [Compliance](#10-compliance)
11. [Obligations of funded pilots](#11-obligations-of-funded-pilots)
12. [Application package](#12-application-package)
13. [Open items](#13-open-items)

---

## 1. Summary

| | |
|---|---|
| **What** | Pilot implementations of digital referrals from general practice to UMZH Connect members, built on the [UMZH Connect Implementation Guide](https://build.fhir.org/ig/umzhconnect/umzhconnect-ig/) (Clinical Order Workflow). |
| **Who** | Technology providers, such as practice information systems (PIS), GP network platforms and integrators, applying together with at least one general practitioner or associated organization. |
| **Phase I** | **Referral by API:** the GP side sends referrals to a UMZH Connect member through the IG's APIs and the shared services. Live operation with real patient data. |
| **Phase II** | **Closed loop:** the GP side reads the outputs of the referral (appointments, clinical reports) as structured FHIR data. |
| **Funding** | Part-funding of **CHF 20,000 per phase** and pilot, paid against milestones. Phase II funding requires an opt-in at application and a completed Phase I. |
| **Budget** | Total **CHF 200,000**, so up to **5 pilots** can be funded (5 × CHF 40,000). |

> **On "general practice":** throughout this tender, *general practitioner / general practice* stands representatively for the wider range of **referrers** the standard serves — for example specialist practices, outpatient and ambulatory clinics, and other referring care providers such as rehabilitation clinics or nursing and home-care services. General practice is the pilot's deliberate starting point, as the highest-volume sending side; the IG and shared services are not limited to it.

---

## 2. Background

UMZH Connect replaces manual re-entry between healthcare providers with standardized, automated data exchange. One data contract, the FHIR Implementation Guide (IG) based on the international Clinical Order Workflow (COW), defines the use cases, data structures, API operations and security. Patient data stays with the treating provider. The shared services provide discovery (the mCSD registry) and machine-to-machine authorization (OAuth 2.0 / SMART on FHIR).

The university hospitals are on board as receivers of referrals. The pilot tender brings the **sending side, general practice,** onto the standard. It proves the workflow end to end in live operation and builds the installed base in the PIS market that a national standard needs.

---

## 3. Scope of the pilot

### 3.1 Phase I — Referral by API

The applicant implements the **Placer** role for the referral use case of the IG:

> **Data focus of Phase I:** the **patient master data** (demographics and administrative/referral context) and the **reason for referral** are the mandatory core. **Referenced clinical resources** (findings, reports, attachments) are **optional** and may be included where the GP system already holds them structured.

- Create and send referrals from the GP system to a UMZH Connect member (the **Fulfiller**) through the IG's API operations and data structures, built on CH Core and the related Swiss IGs.
- Use the **shared services**: find the receiver's endpoint in the mCSD registry, and authenticate through the UMZH Connect authorization server.
- Track the status of the referral on the shared task, as defined by the IG.
- Integrate into the GP's everyday workflow, push-button and without re-entering data.
- Operate the implementation **in production with real patient data** for the pilot period.

### 3.2 Phase II — Closed loop

The applicant extends the implementation to **read the outputs** of a referral back into the GP system as structured FHIR data, as defined by the IG:

- **Appointments** scheduled by the receiver.
- **Clinical reports**, such as the discharge or consultation report, as structured FHIR resources rather than PDF attachments only.
- Display the outputs and, where the GP system supports it, import them, linked to the original referral.

### 3.3 Out of scope

- Implementing the Fulfiller role, or changes to the receiving hospitals' systems.
- Proprietary extensions that bypass the IG or the shared services.
- Use cases other than referral, such as lab or radiology orders. These may follow in later calls.

---

## 4. Who can apply

- **Lead applicant:** a technology provider that develops or operates the sending system. This can be a PIS vendor, a GP network platform or an integrator.
- **Pilot partner (mandatory):** at least one **general practitioner or associated organization**, such as a group practice, health center or GP network, that commits to piloting in live operation.
- **Receiving member (mandatory):** at least one **UMZH Connect member**, including Insel Spital, that confirms in writing its readiness to receive the referrals in the pilot period.

One technology provider may submit one application. It may name several pilot partners and receiving members.

---

## 5. Funding

- **Amount:** CHF 20,000 per phase and funded pilot, so at most CHF 40,000 per pilot.
- **Total budget:** CHF 200,000, so **at most 5 pilots** are funded. Phase II funds of pilots that did not opt in, or did not complete Phase I, are not passed on to other applicants. They remain with the Steering Committee for a later call.
- **Part-funding:** the funding covers part of the implementation effort. The applicant discloses the total effort and bears the remainder. There is no legal claim to funding.
- **Milestone-based payment:**

| Phase | Milestone | Evidence | Share |
|---|---|---|---|
| I | M1 — Conformance | Sandbox integration and the conformance suite passed for the Placer role | 30% |
| I | M2 — Go-live | Security and data-protection review passed, registry entry and credentials issued, first live referral sent | 30% |
| I | M3 — Stable operation | At least **[TBD: e.g. 20]** live referrals over at least **[TBD: e.g. 8]** weeks, plus a pilot report | 40% |
| II | M4 — Conformance | Conformance suite passed for reading appointments and clinical reports | 40% |
| II | M5 — Closed loop live | At least **[TBD: e.g. 10]** completed loops in production, plus a pilot report | 60% |

- **Deadlines:** Phase I must reach M3 within **[TBD: e.g. 9]** months of the funding agreement. Phase II must reach M5 within **[TBD: e.g. 9]** months of its start. The Steering Committee may extend a deadline once if the delay was not caused by the applicant, for example if the receiver is not ready.
- **Phase II release:** Phase II funding is released only to pilots that opted in at application **and** completed Phase I.

---

## 6. Approval requirements

An application is approved only if it meets **all** of the following mandatory requirements. They are assessed pass/fail before scoring.

| # | Requirement | Evidence in the application |
|---|---|---|
| R1 | **Technical implementation that adheres to the IG** for the Placer role of the referral use case, with no proprietary deviations | Technical concept mapped to the IG's profiles and operations; commitment to pass the conformance suite |
| R2 | **Commitment to pilot with productive, real-life data** in Phase I | Signed letter of intent by the pilot partner(s), with a planned go-live date |
| R3 | **Referrals from a general practitioner** (or associated organization) **to a UMZH Connect member** (including Insel Spital) | Letters of intent by the pilot partner and the receiving member |
| R4 | **Usage of the shared services**: mCSD registry for discovery and the authorization server for access | Technical concept; acceptance of the onboarding pipeline |
| R5 | **Opt-in for Phase II funding**: a binding declaration whether the applicant applies for Phase II | Signed declaration; plan for Phase II if opted in |

We further recommend these additional mandatory requirements, to be confirmed by the Steering Committee:

| # | Requirement | Rationale |
|---|---|---|
| R6 | **Data protection and security:** a data-protection concept, a processing agreement with the pilot partner, hosting in Switzerland or a country with adequate protection, and compliance with the security baseline of the [reference architecture](reference-architecture.md) | Live patient data from day one; mandatory security review ([Governance Blueprint §10](governance-blueprint.md#10-security-privacy--compliance-governance)) |
| R7 | **Onboarding through the regular pipeline**, including the participation agreement ([Governance Blueprint §8](governance-blueprint.md#8-participant-onboarding--offboarding)) | Funded pilots get no shortcut to production |
| R8 | **Open contribution:** generic fixes and improvements to the shared artifacts are contributed back. The applicant's own product remains its property. | Openness principle ([Governance Blueprint §12](governance-blueprint.md#12-intellectual-property-licensing--openness)) |
| R9 | **Declarations:** no conflict of interest, no double funding of the same work, acceptance of the funding conditions | Integrity of the procedure |
| R10 | **Transparency of the future licensing/cost model:** the applicant discloses at application the intended pricing and licensing model for productive use after the pilot — e.g. licence fees, per-practice or per-transaction charges, and support/maintenance terms — so participants can assess total cost of ownership | No lock-in and predictable cost before the network scales ([Governance Blueprint §12](governance-blueprint.md#12-intellectual-property-licensing--openness)) |

---

## 7. Award criteria

Applications that meet all requirements are scored from 0 to 5 per criterion. If more than five applications qualify, the five with the highest weighted scores are funded. The next-ranked applications form a waiting list for the case that a funded pilot withdraws before M1.

| Criterion | Weight | What scores well |
|---|---|---|
| **Technical approach and IG adherence** | 30% | A clear mapping to the IG, a sound architecture, use of the sandbox and party stack, a realistic test approach |
| **Pilot readiness** | 25% | Committed partners on both sides, a concrete plan to go live, a credible timeline, an experienced team |
| **Reach and scalability** | 20% | Installed base among GPs, a path from the pilot to general availability in the product, the number of practices that can follow |
| **Data protection and security** | 15% | Quality of the data-protection concept, hosting, access control, incident handling |
| **Value for money** | 10% | Co-financing share, plausibility of the effort, commitment to Phase II, and a transparent, fair future licensing/cost model (R10) |

**Diversity of the portfolio:** when scores are close, the Steering Committee may prefer a mix of systems, vendors and receiving members over several pilots on the same stack. This rule is published with the call.

---

## 8. Selection procedure

| Step | What happens | Responsible | Indicative timing |
|---|---|---|---|
| 1. **Publication** | Call, criteria, templates and timeline are published on the UMZH Connect website and announced to PIS vendors, GP networks and associations | Technical Office | Week 0 |
| 2. **Information session and Q&A** | One online information session; written questions are collected, and all answers are published anonymously to all interested parties | Technical Office | Weeks 1–4 |
| 3. **Submission** | Applications are submitted electronically by the deadline; late submissions are not accepted | Applicants | Week 6 |
| 4. **Formal check** | Completeness and mandatory requirements (R1–R10) checked; one short deadline to fix purely formal gaps | Technical Office | Weeks 7–8 |
| 5. **Evaluation** | Each panel member scores independently, then the panel reaches a consensus score. The Governance & Data-Protection representative may veto an application on compliance grounds. | Evaluation panel | Weeks 8–10 |
| 6. **Technical presentation** | Shortlisted applicants demonstrate their approach and answer questions (optional, at the panel's discretion) | Evaluation panel | Week 10 |
| 7. **Award decision** | The Steering Committee decides on the panel's recommendation; the decision and its reasons are minuted | Steering Committee | Week 12 |
| 8. **Notification and agreement** | All applicants receive a written decision with brief reasons; funded pilots sign the funding agreement and the participation agreement | Technical Office | Weeks 12–14 |
| 9. **Kick-off and monitoring** | Pilots start onboarding; milestone evidence is reviewed by the Technical Office before each payment | Technical Office | From week 14 |

**Reconsideration:** unsuccessful applicants may request reconsideration once, in writing, within 10 days. Requests can only be based on procedural errors, not on the scoring. The Steering Committee decides finally.

---

## 9. Governance of the tender

The tender follows the existing governance structure ([Governance Blueprint §4](governance-blueprint.md#4-bodies--responsibilities)). No new bodies are created.

| Activity | Steering Committee | Technical Office | Evaluation panel | Governance & Data-Protection WG |
|---|---|---|---|---|
| Approve the call, criteria, budget and funding agreement | **A** | R | C | C |
| Run the procedure: publication, Q&A, formal check | I | **A/R** | I | I |
| Evaluate and recommend | I | R | **A** | R (veto) |
| Award decision | **A** | R | C | C |
| Review milestones and release payments | I | **A/R** | – | C (M2 review) |
| Stop or claw back funding | **A** | R | – | C |

**Evaluation panel:** 3–5 members appointed by the Steering Committee for this call:

- the Technical Office lead (chair, non-voting if needed for an odd number of votes);
- one representative of the Use-Case & IG WG;
- one representative of the Governance & Data-Protection WG, who holds the compliance veto;
- one clinical representative from general practice;
- optionally, one independent external expert, such as one from HL7 CH or eHealth Suisse.

**Conflicts of interest:**

- Every panel and Steering Committee member signs a declaration of interests for this call.
- Members with ties to an applicant, such as employment, consulting, shareholding or close personal relationships, recuse themselves from the evaluation and the decision on that application.
- Representatives of a receiving member may confirm its readiness but do not score applications naming their own institution.
- Recusals are minuted.

**Records:** the call, the published Q&A, the individual and consensus scores, the recommendation and the award decision are kept for at least 10 years. The decision is recorded like any other governance decision.

---

## 10. Compliance

### 10.1 Funding and procurement law

UMZH and its member institutions are public bodies. Whether the part-funding counts as a **subsidy** (Staatsbeitrag) or as a **public procurement** under the IVöB determines the applicable rules.

The design aims for a subsidy: the applicant co-finances the work, keeps the rights to its product and serves its own customers, and UMZH Connect buys no service for its own use. The amounts are below the thresholds for open procurement.

UMZH legal services must still **confirm this classification before publication [TBD]**. Either way, the call follows the principles of **transparency, equal treatment and non-discrimination**:

- The criteria and weights are published in advance.
- All applicants receive the same information.
- There is no exclusivity, and no funded vendor receives preferential treatment in onboarding or in the standard.

### 10.2 Data protection

- **Roles:** the GP is the controller of the patient data. The technology provider acts as a processor under a processing agreement with the GP. The receiving member is controller of its own data.
- **Applicable law:** the Federal Act on Data Protection (nDSG) for private practices and vendors. The cantonal data-protection laws apply to the public receiving institutions: IDG ZH for Zurich, KDSG for Bern (Insel).
- **Professional secrecy:** the technology provider is bound as an auxiliary person to medical professional secrecy (Art. 321 StGB) and binds its staff and subcontractors accordingly.
- **Consent and minimization:** patient information and consent follow the IG's consent model. Only the data the referral needs is shared.
- **DSFA:** the pilot partner and the provider complete a data-protection impact assessment, using the UMZH Connect DSFA blueprint in [`datenschutz/`](datenschutz/), before go-live.
- **Sandbox:** no production data is used in the sandbox or in conformance testing.

### 10.3 Information security

- The provider meets the security baseline of the [reference architecture](reference-architecture.md): client credentials and key management, encrypted transport, audit logging and patch management.
- The mandatory security review ([Governance Blueprint §10](governance-blueprint.md#10-security-privacy--compliance-governance)) must be passed before production credentials are issued.
- Security incidents and personal-data breaches are reported to the Technical Office without delay, in addition to statutory duties. The fast-track change process applies.

### 10.4 Regulatory

- The applicant self-assesses whether its implementation qualifies as a medical device (MepV / MDR) and remains responsible for that. Pure communication software usually does not qualify.
- The pilot does not replace or interfere with the electronic patient record (EPD). Where possible, alignment with eHealth Suisse exchange formats is welcome.

### 10.5 Funding conditions

- The funds may be used only for the described implementation and pilot.
- The Technical Office may review milestone evidence and, for cause, effort records.
- Funding is **reclaimed** in full or in part if:
  - statements in the application were false;
  - the pilot is abandoned after payment;
  - funded results are used to bypass the standard;
  - the same work is funded twice.
- Funded pilots may be named publicly. The pilot reports and lessons learned are published, after removing confidential business information.

---

## 11. Obligations of funded pilots

- Complete the onboarding pipeline and remain conformant to a supported IG version for at least **[TBD: e.g. 24]** months after go-live.
- Delegate a contact to the Use-Case & IG WG for the pilot period. Raise IG issues through the change-request process.
- Report pilot metrics monthly, without personal data:
  - number of referrals and completed loops;
  - error and rejection rates;
  - time to go-live;
  - feedback from users on both sides.
- Deliver a pilot report per phase: technical lessons learned, workflow impact and change requests for the IG.
- Take part in at least one interoperability event, such as the HL7 CH connectathon or the eHealth Suisse Projectathon, where one is scheduled during the pilot.
- Offer the integration to further GP customers on general availability, on non-discriminatory terms. **[TBD: binding or best effort]**

---

## 12. Application package

1. **Applicant profile:** organization, product(s), installed base among GPs, team and references.
2. **Partners:**
   - letter(s) of intent by the pilot partner(s) (R2, R3);
   - letter(s) of intent by the receiving member(s) (R3).
3. **Technical concept:** architecture, mapping to the IG for Phase I and, if opted in, Phase II, use of the shared services, test approach.
4. **Plan:** milestones M1–M5 with dates, dependencies and risks.
5. **Budget:** total effort, requested funding and co-financing share.
6. **Data protection and security:** data-protection concept, hosting, processing agreement template, security measures.
7. **Declarations:**
   - opt-in for Phase II (R5);
   - acceptance of the funding conditions and the onboarding obligations;
   - conflicts of interest;
   - no double funding.

Templates for the letters of intent, the budget and the declarations are provided with the call **[TBD]**.

> **Form of the application — keep it lean.** We expect a **compact slide deck** that covers the topics above (applicant profile, partners, technical concept and IG mapping, plan and milestones, budget and co-financing, data protection and security, and the future licensing/cost model), rather than a long prose document. The letters of intent, the budget and the declarations are attached separately on the provided templates. Clarity and conciseness count, not length.

---

## 13. Open items

| # | Item | Owner |
|---|---|---|
| 1 | Legal classification as a subsidy or a procurement, and the funding agreement template | UMZH legal services |
| 2 | Thresholds for the milestones: referral volume, operating period, deadlines | Steering Committee, on proposal by the Technical Office |
| 3 | Confirm the recommended requirements R6–R10 | Steering Committee |
| 4 | Readiness and capacity of the receiving members, including Insel Spital, to receive referrals and deliver Phase II outputs (appointments, structured reports) | Technical Office with the members |
| 5 | Status of the conformance suite for the Placer role and the Phase II outputs; the call should not open before it is available | Governance & Data-Protection WG |
| 6 | Whether funded vendors must hold HL7 CH membership, as participants must ([Governance Blueprint §8.1](governance-blueprint.md#81-membership-commitments)) | Steering Committee |
| 7 | Binding or best-effort roll-out to further GP customers after the pilot | Steering Committee |
| 8 | Composition of the evaluation panel | Steering Committee |
