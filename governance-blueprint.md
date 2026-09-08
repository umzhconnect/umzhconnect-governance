# Governance Blueprint: UMZH Connect

**Scope:** The organizational setup that develops, maintains, operates and governs the UMZH Connect ecosystem — its standards, specifications, shared services and integration artifacts.
**Status:** Blueprint / v0.3
**Date:** 2026-09-07

> This is the **normative core** — the rules. The reasoning behind the model, the
> diagrams, the scaling path and the handover narrative live in the companion
> [Rationale & Background](governance-rationale.md). Where the two differ, this
> document governs.

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Guiding Principles](#2-guiding-principles)
3. [What Is Governed](#3-what-is-governed)
4. [Bodies & Responsibilities](#4-bodies--responsibilities)
5. [Decision-Making](#5-decision-making)
6. [Change Management](#6-change-management)
7. [Prioritization & Roadmap](#7-prioritization--roadmap)
8. [Participant Onboarding & Offboarding](#8-participant-onboarding--offboarding)
9. [Operations of Shared Services](#9-operations-of-shared-services)
10. [Security, Privacy & Compliance Governance](#10-security-privacy--compliance-governance)
11. [Funding & Sustainability](#11-funding--sustainability)
12. [Intellectual Property, Licensing & Openness](#12-intellectual-property-licensing--openness)
13. [Handover to National Initiatives](#13-handover-to-national-initiatives)
14. [Glossary](#14-glossary)

---

## 1. Purpose & Scope

UMZH Connect standardizes and automates **clinical orders** (referrals and external service orders such as lab orders) across a network of healthcare providers. Data exchange is **API- and FHIR-based**, governed by a dedicated Implementation Guide (IG) derived from the international *Clinical Order Workflow IG*.

To make the ecosystem work, UMZH Connect maintains **shared artifacts** (the IG, reference code, test suites) and operates **shared services** (an mCSD registry and an authorization server).

This blueprint defines **how that work is organized and decided**: who owns the artifacts, who operates the services, how participants are onboarded, and how the standard evolves. It is not a technical specification and references the technical repositories rather than restating them.

**In scope:** the bodies, roles and responsibilities that own the ecosystem; change management, prioritization and decision-taking for all governed artifacts; participant onboarding and ongoing governance; operation and service levels of the shared services; funding, sustainability, and the mid-term handover to national initiatives.

**Out of scope:** internal IT governance of individual participants; clinical governance of the underlying medical processes; the detailed technical content of the IG and the services (owned by their repositories).

**Strategic aim:** establish a **nation-wide standard** for digital clinical orders and, mid-term, **hand over** the maintained artifacts and operated services to national initiatives ([§13](#13-handover-to-national-initiatives)).

---

## 2. Guiding Principles

These are the tie-breakers for every decision. The reasoning is in the [Rationale](governance-rationale.md#2-principles--the-reasoning).

1. **Use-case driven.** Each clinical use case defines its structural (data structures, profiles) and workflow (API operations, interaction patterns) requirements, is prioritized by business value, and is documented as examples in the IG. Nothing enters the standard without a driving use case.
2. **Standards-first, not product-first.** The IG and its conformance criteria are the primary asset; services and code exist to realize and validate it.
3. **Interoperability over local optimization.** A change that serves one participant at the expense of network-wide interoperability is rejected or generalized.
4. **Backward compatibility by default.** Breaking changes are exceptional — explicitly classified, announced with a deprecation window and a migration path.
5. **Open and transparent.** Specifications, decisions and roadmaps are public by default; deliberations are minuted; artifacts carry open licenses ([§12](#12-intellectual-property-licensing--openness)).
6. **HL7 CH aligned.** The IG is built on CH Core and related Swiss IGs (CH eTOC, CH ORF, CH ELM), and is **balloted through HL7 CH** ([§5.4](#54-ig-changes-hl7-ch-balloting)); the intended path is its adoption as an HL7 CH IG, with the shared services handed to a national operator ([§13](#13-handover-to-national-initiatives)).
7. **Privacy and security by design.** Data-protection and security review are built into the change and onboarding processes ([§10](#10-security-privacy--compliance-governance)).
8. **Lightweight but accountable.** The smallest governance that works — clear owners, short paths, written decisions; bodies are added or split only when workload demands it ([§4.4](#44-pilot-configuration)).

---

## 3. What Is Governed

Each asset has a **maintaining working group** and a **designated maintainer** accountable for its backlog and releases.

| Asset | Repository / Location | Type | Primary Working Group |
|---|---|---|---|
| **Implementation Guide** (profiles, use cases, API operations, security) | `umzhconnect-ig` | Specification | IG WG |
| **Use-case definitions** (structural & workflow requirements, acceptance criteria) | `umzhconnect-ig` (`input/use-cases/`) | Specification input | Clinical & Use-Case WG |
| **mCSD Registry** (Organizations, Endpoints, HealthcareServices) | `umzhconnect-registry` | Shared service | Platform & Shared Services WG |
| **Authorization Server** (Keycloak, SMART/OAuth2 M2M) | `umzhconnect-auth` | Shared service | Platform & Shared Services WG |
| **Reference / Sandbox** (two-party reference implementation) | `umzhconnect-sandbox` | Reference artifact | Platform & Shared Services WG |
| **Party Integration Stack** ("cow" — single-hospital node) | `umzhconnect-cow` | Reference artifact | Platform & Shared Services WG |
| **Reference Architecture** (production single-hospital) | `umzhconnect-governance/reference-architecture.md` | Guidance | Platform & Shared Services WG |
| **Data-Protection Blueprints** (BRA, DSFA/DPIA, consent template) | `umzhconnect-governance/datenschutz/` | Compliance artifact | Security & Data-Protection WG |
| **Governance Blueprint** (this document + rationale) | `umzhconnect-governance/` | Governance | Steering Committee |
| **Conformance & test suites** | per-repo `tests/` | Quality artifact | owning WG + Onboarding & Conformance WG |

**Two governance regimes** run in parallel and are kept in step:

- **Specification governance** — the IG and its conformance criteria (the *contract*). UMZH Connect-maintained and **balloted through HL7 CH**, on a path to adoption as an HL7 CH IG ([§5.4](#54-ig-changes-hl7-ch-balloting)).
- **Service & artifact governance** — the running shared services and reference code that *implement* the contract. UMZH Connect-internal, governed by the process in [§6](#6-change-management).

A contract change that affects the services triggers a coordinated release across both ([§6.3](#63-versioning--release-policy)).

---

## 4. Bodies & Responsibilities

### 4.1 The bodies

| Body | Purpose | Decides | Cadence |
|---|---|---|---|
| **Steering Committee** (SC) | Strategic authority; escalation point of last resort; accountable for the handover. | Annual roadmap and release-train dates; major/breaking changes to the services and reference artifacts (IG content goes through [§5.4](#54-ig-changes-hl7-ch-balloting)); changes to the security or trust model; onboarding of new *classes* of participant and eligibility/conformance rules; budget and funding model; handover plan; chartering/merging/dissolving WGs. | Quarterly + extraordinary |
| **Technical Office** (Core Team) | The standing, (partly) funded team that does and coordinates day-to-day work: maintains artifacts, **operates the shared services**, runs releases, staffs onboarding. | Editorial changes; CR triage and classification; release-train execution; operational decisions within agreed SLAs. | Weekly sync |
| **Working Groups** (WGs) | Where analysis, design and most recommendations happen. Each has a chair, a charter and an open backlog. Membership is open to delegates from any participant plus co-opted experts. | Minor/non-breaking changes (lazy consensus); recommendations on major/breaking changes; backlog prioritization for their topic. | Bi-weekly to monthly |
| **Participant Assembly** | The forum of all onboarded participants — the community voice. Each participant nominates delegates. | Raises and endorses change requests and roadmap items; elects participant representatives to the SC; is consulted, with a defined comment period, on breaking changes (for IG content, via the HL7 CH comment period — [§5.4](#54-ig-changes-hl7-ch-balloting)) and onboarding-rule changes. | Semi-annual + always-open intake |

Funding sits **outside** the governance bodies and provides mandate and budget. SC composition (chair, sponsor representatives, Technical Office lead, elected participant representatives, the Security & Data-Protection WG chair, and a non-voting national-alignment liaison once identified) is set out in the [Rationale](governance-rationale.md#3-organizational-structure--diagram--notes); the SC fixes and publishes its exact voting-seat roster.

### 4.2 Standing Working Groups

| Working Group | Mandate |
|---|---|
| **Clinical & Use-Case WG** | Identify, prioritize (by business value) and specify clinical use cases; for each, derive structural and workflow requirements and ensure clinical validity. Feeds requirements to the IG WG. |
| **IG WG** | Own the IG: profiles, data structures, API operations, value sets, FSH, conformance rules, publication. Runs its **HL7 CH comment cycle** and prepares it for adoption as an HL7 CH IG ([§5.4](#54-ig-changes-hl7-ch-balloting)); keeps it built on CH Core and related Swiss IGs. |
| **Platform & Shared Services WG** | Design, build and operate the registry, auth server, sandbox and party stack; deployment and reference architecture. |
| **Security & Data-Protection WG** | Trust/security model, threat & risk assessment, DSFA/DPIA, consent model, incident response. |
| **Onboarding & Conformance WG** | Onboarding process, conformance/certification suite, registry admission, participant support. |

WGs may also be **time-boxed task forces** chartered by the SC for a specific deliverable, then dissolved. During the pilot the standing WGs run in a merged configuration ([§4.4](#44-pilot-configuration)).

### 4.3 RACI

*A = Accountable (single owner), R = Responsible, C = Consulted, I = Informed.*

| Activity | SC | Technical Office | Relevant WG | Participant Assembly |
|---|---|---|---|---|
| Set strategy & annual roadmap | **A** | R | C | C |
| Specify a use case | I | R | **A** (Clinical) | C |
| IG non-substantive change (technical correction) | I | R | **A** (IG WG) | I |
| IG substantive change — scope & publication slot | **A** | R | C (IG WG) | I |
| IG substantive change — content & reconciliation | I | R | **A** (IG WG, per HL7 CH rules) | C (via own HL7 CH membership) |
| Operate shared services (registry/auth) | I | **A/R** | C (Platform) | I |
| Approve a release train | **A** | R | C | I |
| Prioritize the backlog | C | R | **A** (per topic) | C |
| Onboard an individual participant | I | R | **A** (Onboarding) | I |
| Onboard a new *class* of participant | **A** | R | C | C |
| Security / trust-model change | **A** | R | **R** (Security) | C |
| Data-protection sign-off (DSFA) | I | C | **A** (Security) | I |
| Funding & budget | **A** | C | I | I |
| Handover (IG → HL7 CH; services → national operator) | **A** | R | C (IG WG for the IG) | C |

### 4.4 Pilot configuration

While the network is small (regional pilot, UMZH-funded, first university hospitals and close partners), the full structure runs in a reduced configuration:

- **Two working groups**, not five: a **Standards & Clinical WG** (merging Clinical & Use-Case + IG WG) and a **Platform, Security & Onboarding WG** (merging Platform & Shared Services + Security & Data-Protection + Onboarding & Conformance). The Security & Data-Protection compliance veto ([§5.2](#52-process-quorum--voting)) still applies within the merged group. The Standards & Clinical WG carries the IG WG's HL7 CH mandate, including the IG's HL7 CH ballot ([§5.4](#54-ig-changes-hl7-ch-balloting)).
- **Steering Committee and Technical Office** may meet jointly, provided decisions are minuted under the correct body and the SC voting rules are respected for major/breaking/strategic items.
- **Participant Assembly** convenes once enough participants exist to elect representatives (target: ≥ 4 onboarded participants); until then, participant feedback is gathered directly by the Technical Office and consultation on breaking changes is bilateral.

The SC dissolves these arrangements — splitting WGs and standing up the Assembly — when workload or participant count makes the full structure necessary. Triggers and the growth path are in the [Rationale](governance-rationale.md#8-scaling-the-governance).

---

## 5. Decision-Making

### 5.1 Decision classes

Every change is assigned a class. The class sets the decision authority, who is consulted, the compatibility expectation, the versioning impact and the SLA — it is the single lever that drives the rest of the process ([§6](#6-change-management)).

| Class | Examples | Authority | Consultation | Compatibility | Version impact |
|---|---|---|---|---|---|
| **Editorial** | Typos, clarifications, non-normative examples | Maintainer (Technical Office) | none | no behavior change | patch / none |
| **Minor / non-breaking** | New optional element, additive value-set entry, backward-compatible service change | Owning WG (lazy consensus) | WG | backward-compatible | minor |
| **Major / normative** | New API operation, new mandatory element in a service contract, new use case | Owning WG **recommends**, SC **approves** | WG + SC | backward-compatible, normative | minor (feature) |
| **Breaking** | Removing/renaming a service API element, incompatible auth or security change | SC (escalation path [§5.3](#53-escalation)) | WG + SC + **Assembly** | **not** backward-compatible | **major** + deprecation window |
| **Strategic** | Roadmap, funding, new participant class, handover, licensing | SC | SC | n/a | n/a |
| **Emergency** | Security fix, service-affecting incident | Technical Office acts, SC ratifies after the fact | post-hoc | varies | patch / hotfix ([§6.4](#64-emergency--security-changes)) |

**IG content is out of scope for the rows above** — normative and breaking IG changes follow [§5.4](#54-ig-changes-hl7-ch-balloting). The table governs the shared services, reference artifacts and all non-IG decisions.

### 5.2 Process, quorum & voting

**Default mode: lazy consensus.** A proposal circulated with a defined review window (5–10 working days) and no sustained objection is **adopted**.

**When consensus fails or the class requires a vote:**

- **Working Group:** simple majority of active members; the chair breaks ties. Quorum = 3 members or half of active members, whichever is greater.
- **Steering Committee:** simple majority; **major / breaking / strategic** decisions require a **two-thirds majority of voting seats**. Quorum = a majority of voting seats including the Chair. The Chair breaks ties except on breaking/strategic items, which need the two-thirds threshold rather than a casting vote. The non-voting national-alignment liaison counts toward neither quorum nor result.
- **Compliance veto:** the Security & Data-Protection WG may **block** a change on documented compliance/risk grounds. The block is overridden only by an SC two-thirds vote **and** a recorded risk acceptance.

**Every non-editorial decision is recorded** as a short **Architecture/Decision Record (ADR)** in the owning repository: context, options, decision, consequences, responsible body.

### 5.3 Escalation

Escalate when: consensus fails, the change is major/breaking/strategic, it spans multiple WGs, or it touches the security/trust model, funding, or participant eligibility.

Path: **Change request / issue → owning WG →** (unresolved or major) **→ Technical Office triage →** (cross-cutting or breaking) **→ Steering Committee →** (breaking / onboarding-rule) **→ Participant Assembly comment period → back to SC → decision recorded (ADR).** Breaking **IG content** takes the HL7 CH comment period instead of the Assembly one ([§5.4](#54-ig-changes-hl7-ch-balloting)). Diagram in the [Rationale](governance-rationale.md#4-decision-making--escalation-diagram).

### 5.4 IG changes: HL7 CH balloting

The IG is maintained by UMZH Connect (IG WG) and builds on CH Core and related Swiss IGs. It is **currently balloted through HL7 CH** — HL7 CH runs the standards process (public comment, reconciliation) — but it is **not yet published or owned by HL7 CH**. Adoption as an HL7 CH IG is the remaining step ([§13](#13-handover-to-national-initiatives), [Rationale open item 11](governance-rationale.md#10-open-items)). Decisions on **normative IG content** follow that process rather than the internal path in [§5.1](#51-decision-classes):

- **Non-substantive** IG changes — a typo, clarification or fix that does not change conformance expectations — are decided by IG WG lazy consensus and published as a technical correction.
- **Substantive** IG changes — new or removed profile, changed cardinality or binding strength, new mandatory element, changed interaction — go through **HL7 CH's published standards process**: published draft → public comment period → recorded reconciliation → new versioned release, per HL7 CH's current rules.
- **Urgent security-relevant IG corrections** cannot wait for a comment slot: the IG WG and Security & Data-Protection WG publish an out-of-band technical correction, the SC **ratifies** after the fact (as in [§6.4](#64-emergency--security-changes)), and it is carried into the next comment cycle for confirmation.

The **Steering Committee governs the envelope, not the content**: it approves the IG's scope and its roadmap slot, but does not adjudicate profile-level content or the comment reconciliation — that sits with the IG WG under HL7 CH rules. Everything else keeps the [§5.1](#51-decision-classes) path.

On **adoption as an HL7 CH IG**, HL7 CH becomes publisher/owner and a **dedicated working group (Arbeitsgruppe)** is set up for the IG under the HL7 CH Technical Committee; the full transition is in the [Rationale](governance-rationale.md#9-handover-to-national-initiatives--transition-model).

---

## 6. Change Management

All evolution of governed assets — IG, shared service, or reference artifact — enters through one intake and triage process, building on existing repository conventions (per-repo `CONTRIBUTING.md`, `BACKLOG.md`). Decide and release then diverge: substantive IG changes run the HL7 CH comment cycle ([§5.4](#54-ig-changes-hl7-ch-balloting)); everything else follows the class authority in [§5.1](#51-decision-classes).

### 6.1 Change request lifecycle

1. **Submit** — anyone opens a CR using the standard template (problem, affected asset, use case, proposed change, impact, urgency). Registry/onboarding requests follow the existing `requests/` convention.
2. **Triage** — Technical Office confirms completeness, routes to the owning WG, links duplicates.
3. **Classify** — assign a change class ([§5.1](#51-decision-classes)); this sets authority, consultation, versioning impact and SLA.
4. **Prioritize** — WG scores the item ([§7](#7-prioritization--roadmap)).
5. **Design & spec** — WG produces the concrete change (FSH edits, service design, conformance impact).
6. **Decide** — approve / reject / defer per the class; record an ADR.
7. **Implement & test** — PR against the repo, example instances validated, conformance suites green.
8. **Release** — bundled into the next release train; changelog updated, versions bumped ([§6.3](#63-versioning--release-policy)).
9. **Communicate** — release notes to the Assembly; migration guidance for major/breaking changes.

Every CR is public with a visible status; rejection or deferral carries a recorded rationale. For a substantive IG change, steps 6 (Decide) and 8 (Release) run through [§5.4](#54-ig-changes-hl7-ch-balloting) instead.

### 6.2 Change classification

Triage assigns a provisional class from the [§5.1](#51-decision-classes) table; the owning WG confirms it during design. For the IG, the **substantive / non-substantive** split ([§5.4](#54-ig-changes-hl7-ch-balloting)) applies on top — a breaking IG change is always substantive.

### 6.3 Versioning & release policy

- **Semantic versioning** (`MAJOR.MINOR.PATCH`) for the IG and each service; a breaking / substantive change increments MAJOR. STU (Trial Use) vs. Normative is a *status*, not a version; comment-cycle builds carry a `-ballot` suffix ([§5.4](#54-ig-changes-hl7-ch-balloting)).
- **Contract/implementation coupling:** each IG version declares the compatible ranges of the shared services and reference stack; a breaking IG change is released only alongside compatible service releases.
- **Release trains:** services quarterly, IG releases on HL7 CH's calendar; if a slipped IG cycle blocks a coupled service release, the **SC decides** whether to hold it or ship against the last published IG version (ADR). Emergency releases are out-of-band.
- **Deprecation policy:** breaking changes ship with a deprecation window (default ≥ 1 release train; target ≥ 6 months for elements participants integrate against), a migration guide, and where feasible a coexistence period.
- **Versioned conformance:** the conformance/certification suite is tagged to the IG version a participant is certified against ([§8](#8-participant-onboarding--offboarding)).

### 6.4 Emergency & security changes

For security vulnerabilities or service-affecting incidents the Technical Office may **act first and ratify later**:

1. Triage severity; if it endangers data, trust or availability, invoke the fast-track.
2. Apply the fix (hotfix release / service mitigation), Security & Data-Protection WG lead informed.
3. Notify affected participants promptly; handle any statutory data-protection breach notification per [§10](#10-security-privacy--compliance-governance).
4. SC **ratifies** at the next or an extraordinary session; a post-incident review and ADR are produced.

---

## 7. Prioritization & Roadmap

Prioritization is explicit and repeatable so the roadmap reflects value, not volume of voices.

**Scoring (WSJF-style):** each candidate is scored on clinical / participant value, interoperability / standardization impact, compliance / risk reduction, and national-alignment fit, divided by effort / cost. Higher = sooner. The Clinical & Use-Case WG owns clinical value; the IG and Platform WGs own effort and interoperability estimates; the Security & Data-Protection WG owns risk scoring. The concrete scale and weights are in the [Rationale](governance-rationale.md#6-prioritization--the-scoring-model).

**Roadmap process:** WGs maintain prioritized backlogs (`BACKLOG.md`) and review them monthly → the Technical Office consolidates a draft annual roadmap with release-train dates → the Participant Assembly reviews and comments → the SC **approves** the roadmap and any in-year re-prioritization that shifts a release train.

**Tie-breakers:** the guiding principles ([§2](#2-guiding-principles)) — standardization and interoperability win over local optimization.

---

## 8. Participant Onboarding & Offboarding

Membership is more than technical access — it is a commitment to the shared standard and its upkeep.

### 8.1 Membership commitments

Formalized in the **participation agreement** signed at onboarding ([§8.2](#82-onboarding--offboarding-pipeline), stage 2). By becoming a member, a participant commits to:

- **Governance participation.** Delegate members to the relevant Working Groups, take part in the Participant Assembly, and serve on the Steering Committee where elected.
- **HL7 CH membership.** Hold **HL7 CH membership** — directly or through a parent/partner member organization — for the duration of participation; this carries voting rights in the IG's HL7 CH balloting ([§5.4](#54-ig-changes-hl7-ch-balloting)).
- **Conformant implementation & operation.** Implement **and operate** IG-conformant APIs for the participant's selected roles (Placer and/or Fulfiller) and use cases: pass conformance testing, run the APIs to agreed service levels, and keep them certified against a supported IG version.
- **Open-source contribution.** Contribute fixes, improvements and generally-useful extensions back to the shared artifacts under the project's contributor terms ([§12](#12-intellectual-property-licensing--openness)); no private forks.
- **Financial commitment.** *To be defined* — a member contribution (fee and/or in-kind) is anticipated as funding broadens; the model is set by the SC ([§11](#11-funding--sustainability)) and applied transparently and proportionately.
- **Data protection & security.** Act as **controller** of one's own patient data; maintain the required DSFA/DPIA and consent handling ([§10](#10-security-privacy--compliance-governance)), meet the security baseline and reference architecture, patch promptly, and honour breach-reporting duties.
- **Operational reliability & directory accuracy.** Meet agreed availability/SLA obligations, keep mCSD registry records and endpoints current, and practise credential/key hygiene.
- **Conformance upkeep.** Adopt breaking changes within the published deprecation window ([§6.3](#63-versioning--release-policy)), re-certify on each new normative IG version, and join interoperability/regression testing when asked.
- **Good-faith collaboration.** Respect recorded decisions (ADRs) and the change process, share operational feedback, and avoid unilateral non-standard behaviour.
- **Named contacts & orderly exit.** Designate a technical and a governance contact; give advance notice before leaving and meet remaining data-protection obligations on exit.

Commitments **scale with involvement**: the role/use-case selection a member declares at onboarding sets the scope of its obligations.

### 8.2 Onboarding & offboarding pipeline

A governed, staged pipeline owned by the Onboarding & Conformance WG and executed by the Technical Office. Admitting a *new class* of participant is an SC decision; admitting an *individual* participant of an approved class is operational.

1. **Apply** — organization declares intent, contacts, target use cases and role(s).
2. **Eligibility & agreement** — meets participation criteria (including HL7 CH membership, [§8.1](#81-membership-commitments)); signs the participation agreement (obligations, data-protection roles, SLAs, IP/licensing acceptance, security baseline).
3. **Sandbox integration** — integrate against `umzhconnect-sandbox` / `umzhconnect-cow`; no production data.
4. **Conformance testing** — pass the versioned conformance suite for the target IG version; participation in the HL7 CH FHIR connectathon / eHealth Suisse Projectathon counts toward this where scheduled.
5. **Security & data-protection review** — deployment reviewed against the reference architecture and the DSFA/consent blueprints; risk sign-off by the Security & Data-Protection WG.
6. **Registry entry & credentials** — Technical Office creates the participant's `Organization`/`Endpoint`/`HealthcareService` records and provisions auth-server client(s), following the `requests/` and id-convention rules.
7. **Production go-live** — coordinated cutover; monitoring in place.
8. **Ongoing conformance** — re-certification is required on adopting a new normative IG version or on material integration changes.

**Offboarding:** a defined process to revoke auth-server credentials, remove or deactivate registry records, and handle data-protection obligations (retention, deletion) on exit — whether voluntary or for cause (sustained non-conformance, security breach). Offboarding for cause is an SC decision.

**Conformance & certification** is the trust anchor: a participant is certified against a specific IG version, and that status is the precondition for a live registry entry and production credentials.

---

## 9. Operations of Shared Services

UMZH Connect **runs** the shared services; participants depend on them, so they are governed to explicit service levels.

**Operated:** the **mCSD registry** (`umzhconnect-registry`) and the **authorization server** (`umzhconnect-auth`). The **sandbox** and **party stack** are reference artifacts (best-effort availability).

**Service-management commitments** (targets finalized per funding — see [Rationale §10](governance-rationale.md#10-open-items)):

- **Availability targets (SLA/SLO)** per shared service, with published maintenance windows.
- **Incident management** — severity classes, response/restoration targets, a single status/communication channel, post-incident reviews.
- **Change/upgrade management** — service changes follow the CR process ([§6](#6-change-management)); registry and auth-server upgrades are announced and, where they affect the contract, coupled to an IG-compatible release.
- **Backup, DR & data integrity** — for the registry directory and auth-server configuration/keys, with tested recovery.
- **Key & trust management** — lifecycle of signing keys, client credentials and JWKS, rotation policy, and the trust list. Trust-model changes are security-class decisions.
- **Observability & audit** — operational metrics and security-relevant audit logging, retained per the data-protection blueprints.
- **Registry data governance** — the directory is authoritative for discovery; entries change only through the onboarding/change process and obey the in-repo id conventions.

Operational runbooks live with each service repository; this blueprint governs **who** commits to **what levels** and **how changes are authorized**.

---

## 10. Security, Privacy & Compliance Governance

Security and data protection are **cross-cutting gates**, owned by the Security & Data-Protection WG and embedded in the change and onboarding processes.

- **Data-protection artifacts** live in `umzhconnect-governance/datenschutz/`: the **BRA** (Bearbeitungsreglement), the **DSFA/DPIA blueprint**, and the **consent template**. These are governed artifacts with their own change process.
- **Roles under data-protection law:** participants are **controllers** of their clinical data. The shared services carry directory and machine-to-machine authorization data, not clinical payloads; UMZH Connect's controller/processor position for that data is stated in the BRA and encoded in the participation agreement.
- **Consent model:** the ecosystem is consent-centric (fine-grained, policy-enforced authorization). Consent-model changes are **security-class** decisions with WG sign-off.
- **Mandatory security review** for any change touching the trust/security model, the auth server, the registry's exposure, or a participant's production deployment. The WG holds the compliance veto ([§5.2](#52-process-quorum--voting)).
- **Threat & risk management:** maintain a living threat model and risk register; feed high-severity items into prioritization ([§7](#7-prioritization--roadmap)).
- **Incident & breach handling:** security incidents use the emergency-change fast-track ([§6.4](#64-emergency--security-changes)); personal-data breaches additionally follow statutory notification duties, coordinated with the DPO function.
- **Auditability:** ADRs, registry changes, credential issuance and conformance results are traceable — the evidence base for operational trust and eventual handover.

---

## 11. Funding & Sustainability

**Today:** funding is provided by **UMZH (Universitäre Medizin Zürich)**, which holds the initial mandate and appoints the Steering Committee Chair.

**Broadening (planned):** the model accepts additional funding sources without changing the governance skeleton — participant contributions (fees or in-kind WG staffing, calibrated not to deter adoption), cantonal / public-health co-funding, and national-programme funding as handover approaches ([§13](#13-handover-to-national-initiatives)).

**Sustainability principles:**

- **Separate "standard" from "service" costs.** The IG is a public good; running the services has ongoing operational cost. Funding covers both, budgeted distinctly so the standard survives even if service operation is later transferred. The initiative's own HL7 CH membership and process costs sit under the "standard" line (participants' own HL7 CH membership is separate — [§8.1](#81-membership-commitments)).
- **No lock-in through funding.** A single funder does not gain disproportionate control; strategic authority stays with the SC under its voting rules.
- **Transparent budget.** The SC approves and publishes an annual budget and a multi-year sustainability plan, including the runway to a viable handover.

---

## 12. Intellectual Property, Licensing & Openness

- **Open by default.** Specifications and code are published under open licenses; the IG is licensed CC0 1.0. New artifacts adopt a compatible open license by default.
- **Contributor terms.** Contributions are accepted under a clear inbound license / contributor agreement so the artifacts are relicensing-compatible and **transferable to a national body** without per-contributor renegotiation.
- **Trademark / naming.** "UMZH Connect" naming and any conformance mark are governed by the SC; use of a conformance mark is tied to certification status ([§8](#8-participant-onboarding--offboarding)).
- **Standards alignment.** The IG builds on the international *Clinical Order Workflow IG* and Swiss base standards (CH Core and related); it is FHIR-native and reuses IHE resource models where useful (mCSD for the registry). Upstream licensing and attribution are respected, and changes of general value are offered upstream where appropriate.
- **License alignment (open item).** The IG's license must be reconciled across three constraints: its CC0 intent, the HL7 International license on the upstream *Clinical Order Workflow IG*, and HL7 CH's IG-licensing conventions — which the current balloting through HL7 CH may already impose, so this needs checking now rather than at adoption. Tracked in the [Rationale](governance-rationale.md#10-open-items).

---

## 13. Handover to National Initiatives

A core strategic aim is a **nation-wide standard** and, **mid-term, handover** of the maintained artifacts and operated services. The two regimes hand over to **different targets**:

- **The IG → HL7 CH.** Balloting through HL7 CH is already underway ([§5.4](#54-ig-changes-hl7-ch-balloting)); the remaining step is adoption of the IG as an HL7 CH IG, after which it is maintained under HL7 CH rules and the specification handover is largely done.
- **The shared services → a national operator** (e.g. eHealth Suisse, a cantonal or federal body), with the readiness criteria below.

**Readiness criteria** (handover does not happen before all hold):

- Stable, versioned IG with a healthy conformance suite and multiple certified participants, adopted as an HL7 CH IG.
- Shared services operated to documented SLAs with runbooks, DR and audit in place.
- Open, transferable licensing and contributor terms ([§12](#12-intellectual-property-licensing--openness)).
- A named national counterpart with the mandate and capacity to take ownership of the services.
- A funding path on the national side for continued operation.

**Continuity guarantees:** participants certified under UMZH Connect remain valid across the handover; the IG version and registry/credentials are preserved or migrated with notice. Routing the IG to HL7 CH removes the "no counterpart" risk for the specification; the SC owns and periodically reviews the handover plan, including the fallback if no national **service** operator emerges. The phased transition model is in the [Rationale](governance-rationale.md#9-handover-to-national-initiatives--transition-model).

---

## 14. Glossary

| Term | Meaning |
|---|---|
| **IG** | Implementation Guide — the normative FHIR specification (`umzhconnect-ig`). |
| **HL7 CH** | HL7 Switzerland — the Swiss HL7 affiliate that publishes the Swiss FHIR IGs. |
| **HL7 CH Technical Committee** | HL7 CH's committee for implementation-guide work; each Swiss FHIR IG is assigned to a **working group (Arbeitsgruppe)** under it. |
| **CH Core / CH eTOC / CH ORF / CH ELM** | Swiss FHIR base and domain IGs (core; transition of care; orders & referrals; laboratory) the IG builds on. |
| **Comment cycle / ballot** | HL7 CH's review of a draft standard: a public comment period, recorded reconciliation, then publication. Mechanics per HL7 CH's current rules. |
| **STU / Normative** | *Normative* means either an IG's binding conformance content (not its examples/prose) or the maturity status after *STU (Trial Use)* — the IG is at STU. Neither is the package version. |
| **Substantive change** | A change that alters conformance expectations; requires a new comment cycle. Non-substantive changes (typos, clarifications) ship as technical corrections. |
| **Connectathon / Projectathon** | Scheduled interoperability test events (HL7 CH connectathon; the IHE Suisse / eHealth Suisse Projectathon). |
| **mCSD** | *Mobile Care Services Discovery* — the IHE profile behind the registry of Organizations/Endpoints/HealthcareServices. |
| **Shared services** | The centrally operated registry and authorization server. |
| **Party / Participant** | An onboarded healthcare provider (hospital; later practice, lab) acting as Placer and/or Fulfiller. |
| **Placer / Fulfiller** | The ordering party and the servicing party in a clinical order workflow. |
| **CR** | Change Request. |
| **ADR** | Architecture/Decision Record — the durable written record of a decision. |
| **WG** | Working Group. |
| **SC** | Steering Committee. |
| **WSJF** | *Weighted Shortest Job First* — the value-over-effort prioritization heuristic. |
| **DSFA / DPIA** | Data-Protection Impact Assessment (*Datenschutz-Folgenabschätzung*). |
| **BRA** | *Bearbeitungsreglement* — records/rules of personal-data processing. |
| **Conformance / certification** | Verified compliance of a participant's implementation with a specific IG version. |
| **UMZH** | Universitäre Medizin Zürich — the current funding body and mandate holder. |

---

*This is a living blueprint and itself a governed artifact ([§3](#3-what-is-governed)); changes follow the process in [§6](#6-change-management) and are approved by the Steering Committee. Non-normative background: [Rationale & Background](governance-rationale.md).*
