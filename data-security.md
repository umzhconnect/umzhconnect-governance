# Data Security & Data Protection

**Scope:** How the UMZH Connect ecosystem protects the sensitive health data it
exchanges — the guiding approach, legal basis and responsibilities, before the
detailed assessments that follow.
**Status:** Overview / v0.1

---

## Why this matters

UMZH Connect enables the automated exchange of **clinical and administrative
patient data** — diagnoses, medication, reports, imaging and referral content —
between healthcare providers. This is among the most sensitive categories of
personal data, and the exchange crosses organisational boundaries. Protecting it
is therefore not an add-on but a **core design constraint** of the ecosystem: the
standard, the shared services and every participant's deployment are shaped by it.

This page gives the **high-level picture**. The detailed, formal assessments and
templates live in [`datenschutz/`](datenschutz/) and in the IG's security concept.

## Our approach in brief

1. **Security by design, privacy by design.** Data-protection and security review
   are built into the change and onboarding processes, not bolted on afterwards.
   See the governance
   [operating model](governance-blueprint.md#10-security-privacy--compliance-governance).
2. **Standards-based security.** The IG specifies OAuth 2.0 / OpenID Connect with
   SMART-on-FHIR system scopes and FAPI 2.0 hardening for machine-to-machine
   communication.
3. **Context-centric, fine-grained authorization.** Access is granted in the
   context of a specific workflow object (a `ServiceRequest` or `Task`) rather than
   as broad, standing access — the receiver sees exactly what the workflow
   requires, and no more.
4. **Placer-driven release.** In the clinical order workflow, the referring
   ("Placer") clinician issues a **technical release** (a FHIR `Consent` resource)
   as part of the referral — the digital equivalent of choosing which attachments
   to send with a referral today. The legal basis for disclosure remains the
   treatment relationship and the clinician's professional responsibility.
5. **Defence in depth.** WAF/TLS termination, gateways, policy enforcement (OPA),
   audit logging to each participant's SIEM, and key/credential lifecycle
   management — layered per the
   [reference architecture](reference-architecture.md).

## Legal basis & responsibilities

- **Applicable law.** The Zurich Information and Data Protection Act
  (**IDG ZH**, LS 170.4), with the revised Swiss Federal Act on Data Protection
  (**revFADP**, SR 235.1) applying subsidiarily.
- **Controllership.** Each participating provider is the **controller** of its own
  patient data. Where two hospitals exchange data, each is responsible for its own
  disclosure, and a **DSFA / DPIA** is generally prepared **per controller**.
- **UMZH Connect's role.** As operator of the shared services (registry and
  authorization server), UMZH Connect provides the trust infrastructure and
  governs the security model; it is not the controller of the clinical data flowing
  between participants. Roles are encoded in the participation agreement.
- **Prior consultation.** Where required, a data-protection impact assessment is
  submitted to the cantonal data-protection authority (DSB ZH) ahead of production
  go-live.

## Detailed documents

> The detailed assessments below are authored in **German**, aligned to the DSB
> Kanton Zürich templates and the Swiss legal context.

| Document | Purpose |
|---|---|
| [DSFA-Blueprint](datenschutz/dsfa-blueprint.md) | Data-Protection Impact Assessment (DSFA/DPIA) template for the production data exchange, per the DSB Kanton Zürich form. |
| [BRA-Blueprint](datenschutz/bra-blueprint.md) | Threat & Risk Analysis (Bedrohungs- und Risikoanalyse): threat actors, attack surface, STRIDE + OWASP API/FHIR threat mapping. |
| [Consent / Placer-Release Template](datenschutz/consent-template.md) | The Placer-release concept and its technical mapping onto the FHIR `Consent` resource. |
| [IG Security Concept](https://github.com/umzhconnect/umzhconnect-ig) | The normative security architecture (OAuth2/OIDC, SMART, context-centric authorization) in the Implementation Guide. |
| [Reference Architecture](reference-architecture.md) | The production security controls per component for a single participating hospital. |

## How it connects to governance

Data security and data protection are **cross-cutting gates** in the operating
model. The **Security & Data-Protection Working Group** reviews every change that
touches the trust/security model and signs off each participant's production
deployment during onboarding, and it holds a documented compliance veto. See the
[operating model, §10](governance-blueprint.md#10-security-privacy--compliance-governance).

---

*This is a living overview. It is a governed artifact; changes follow the change
process in the [operating model](governance-blueprint.md#6-change-management).*
