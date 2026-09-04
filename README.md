# UMZH Connect

**UMZH Connect** is a collaborative initiative to improve digital interoperability
between healthcare providers in the Zurich ecosystem — initially focusing on the
university hospitals and close partners, and designed to extend to further
participants and use cases over time.

Today, key processes such as referrals, transfers, and external orders (e.g. lab
or radiology requests) still require manual re-entry of clinical and administrative
information across systems, causing delays, inconsistencies, and avoidable
workload. UMZH Connect targets these friction points by enabling **"push-button"
and fully automated data exchange** across participants, driven by concrete,
high-value use cases that can be implemented quickly and measured in terms of
clinical and business benefit.

The solution is an **API framework and shared implementation approach** that lets
providers act as API producers and consumers over standardized, interoperable
interfaces (**FHIR** and **REST**). At its centre is a clearly defined *data
contract* — a FHIR Implementation Guide, based on the international
[Clinical Order Workflow (COW)](https://hl7.org/fhir/uv/cow/2025May/) and
customized for Swiss specifics — that supports read and write operations for
agreed workflows while remaining extensible for additional use cases and
participants.

This repository holds the **governance and cross-cutting concept documents** for
the initiative: the [operating model](governance-blueprint.md), the
[data-security and data-protection considerations](data-security.md), the
[production reference architecture](reference-architecture.md), and pointers to the
technical aids that help participants integrate.

---

## Who we are

UMZH Connect is an **initiative and stewarding organisation**, currently funded
and mandated by **UMZH (Universitäre Medizin Zürich)**. We develop and maintain
the shared standards and artifacts, **operate the shared services**, and **govern
the onboarding** of new participants and the further development of the ecosystem.

Governance runs through a lightweight but accountable structure — a **Steering
Committee**, a **Technical Office** (core team), topic-based **Working Groups**,
and a **Participant Assembly**. The full model is described in the
[operating model](governance-blueprint.md).

## What is our goal

- Enable **automated, standardized clinical order and referral workflows** between
  healthcare providers.
- Establish a **nation-wide standard** for digital clinical orders, built on Swiss
  base standards ([CH Core](https://fhir.ch/ig/ch-core/index.html),
  [CH eTOC](https://fhir.ch/ig/ch-etoc/index.html)) and the international COW IG.
- **Mid-term, hand over** the maintained artifacts and operated services to
  national initiatives, so the standard outlives its regional origin.

## What are our services

UMZH Connect maintains a set of shared **artifacts** and operates shared
**services** for the ecosystem:

| Service / Artifact | Description | Repository |
|---|---|---|
| **Implementation Guide** | The normative FHIR data contract: use cases, data structures, API operations and security. | [`umzhconnect-ig`](https://github.com/umzhconnect/umzhconnect-ig) |
| **mCSD Registry** | The public, read-only directory of `Organization`, `Endpoint` and `HealthcareService` records for endpoint discovery. | [`umzhconnect-registry`](https://github.com/umzhconnect/umzhconnect-registry) |
| **Authorization Server** | Keycloak-based OAuth 2.0 / SMART-on-FHIR authorization for machine-to-machine access. | [`umzhconnect-auth`](https://github.com/umzhconnect/umzhconnect-auth) |
| **Sandbox & Party Stack** | Reference implementations that help each participant comply with the IG and integrate with existing systems. | [`umzhconnect-sandbox`](https://github.com/umzhconnect/umzhconnect-sandbox), [`umzhconnect-cow`](https://github.com/umzhconnect/umzhconnect-cow) |

## How do we operate

We operate as a **standards-first, open, and interoperability-driven** steward:

- **Change management** — all evolution of the standard, services and artifacts
  flows through a single, transparent change-request process, with classification
  (editorial → breaking), semantic versioning and predictable release trains.
- **Prioritization & decisions** — an explicit value-over-effort prioritization
  model feeds a roadmap approved by the Steering Committee; decisions default to
  lazy consensus and are recorded.
- **Shared-service operations** — the registry and authorization server are run to
  documented service levels, with incident and change management.
- **Security & data protection by design** — built into the change and onboarding
  processes rather than bolted on.

See the [operating model](governance-blueprint.md) for the full detail.

## How to get involved

We welcome participants and contributors.

- **Become a participant** — healthcare providers (hospitals today; practices and
  labs later) can join through a governed onboarding pipeline: apply → conformance
  testing → security & data-protection review → registry entry and credentials →
  go-live. See
  [Participant Onboarding](governance-blueprint.md#8-participant-onboarding--offboarding).
- **Contribute to the standard** — raise a change request or join a Working Group.
  Contributions to the IG follow its
  [CONTRIBUTING guide](https://github.com/umzhconnect/umzhconnect-ig/blob/main/CONTRIBUTING.md).
- **Join the conversation** — the Participant Assembly and Working Groups are open
  to delegates from any participating organisation.

## Documentation

| Document | What it covers |
|---|---|
| [Governance — Operating Model](governance-blueprint.md) | Who governs and operates the ecosystem; bodies, roles, change management, prioritization, decision-taking, onboarding, funding, and handover. |
| [Data Security & Data Protection](data-security.md) | How sensitive health data is protected: legal basis, security-by-design, consent/authorization model, and the detailed assessments. |
| [Reference Architecture](reference-architecture.md) | The production-grade architecture for a single hospital participating in the ecosystem. |
| [Technical Aid](https://github.com/umzhconnect/umzhconnect-ig) | The IG, sandbox and party stack that help participants implement and integrate — see [`umzhconnect-sandbox`](https://github.com/umzhconnect/umzhconnect-sandbox) and [`umzhconnect-cow`](https://github.com/umzhconnect/umzhconnect-cow). |

## License

Specifications and artifacts are published under open licenses; the IG itself is
licensed under Creative Commons
[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/). See each repository
for its specific license.

HL7®, FHIR® and the FHIR logo are trademarks owned by Health Level Seven
International.
