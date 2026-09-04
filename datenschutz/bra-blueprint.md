# Bedrohungs- und Risikoanalyse (BRA) — UMZH Connect Produktivumgebung

> Anhang / vertiefende Analyse zu [dsfa-blueprint.md](dsfa-blueprint.md) §6.2 P12.
>
> Diese BRA erweitert die DSFA um die **sicherheitstechnische**
> Sicht: Bedrohungsakteure, Angriffsflächen, Asset-bezogene Bewertung,
> STRIDE-Mapping, OWASP-API/FHIR-spezifische Bedrohungen. Sie dient als
> Grundlage für die Vorabkontrolle bei der DSB ZH (IDG ZH § 10), den
> Penetrationstest (DSFA P07), das SIEM-Use-Case-Design (P05) und das
> Notfallkonzept (P08).
>
> **Bezugsdokumente**:
> - [dsfa-blueprint.md](dsfa-blueprint.md) — Datenschutzsicht
> - [security-concept.md](security-concept.md) — Kontrollkatalog
> - [sandbox-architecture-and-requirements.md](sandbox-architecture-and-requirements.md)
> - [authorization-scenarios.md](authorization-scenarios.md)
>
> **Methodik**: STRIDE pro Asset, kombiniert mit OWASP API Security
> Top 10 (2023) und FHIR-spezifischem Bedrohungskatalog (HL7
> Security WG). Bewertung Eintrittswahrscheinlichkeit × Schadenshöhe
> auf 4-stufiger Skala (1=tief, 4=sehr hoch); Risikowert = W × S.
>
> **Terminologie — "Konsent" / `Consent` in dieser BRA**:
>
> Die FHIR-`Consent`-Ressource ist in dieser Architektur eine
> **technische Freigabe-/Autorisierungsressource**, die der Placer-Arzt
> als Teil der Zuweisung ausstellt — funktional das digitale
> Äquivalent zur Auswahl der Mailanhänge / Faxbeilagen bei einer
> heutigen Zuweisung. Sie ist *kein* patientenseitig unterschriebenes
> Einwilligungsdokument. Die juristische Grundlage der Bekanntgabe
> bleibt das Behandlungsverhältnis Placer–Patient (IDG ZH § 8 lit. a /
> § 17 Abs. 2; siehe [dsfa-blueprint.md §3.3](dsfa-blueprint.md)).
>
> Für die Bedrohungsanalyse bedeutet das: Schutzziel ist die **Integrität
> und Durchsetzung der Placer-Freigabe-Entscheidung** — also dass der
> Empfänger genau das (und nur das) sieht, was der zuweisende Arzt
> freigegeben hat. Die Begriffe "Konsent", "Freigabe" und "Sharing-
> Autorisierung" werden im Folgenden synonym verwendet.

---

## 1. Geltungsbereich

### 1.1 Schutzgegenstand

Produktive Inbetriebnahme des UMZH-Connect-Datenaustauschs zwischen
{{Spital A}} und {{Spital B}}, fokussiert auf den **Cross-Party-Pfad
sowie die gemeinsam betriebenen Komponenten (Keycloak + mCSD-Registry)**:

**Spital-eigene Komponenten (in scope, je Spital separat betrieben)**:

- das **externe APISIX-Gateway** (eingehend + ausgehend Cross-Party)
- der **FHIR-Server** (klinische Daten)
- die **OPA** je externem Gateway mit Konsent-/Freigabe-Policy
- der **Audit-Pfad** vom externen Gateway in die spital-eigene
  SIEM-Infrastruktur (kein gemeinsamer SIEM-Eintrittspunkt)

**Gemeinsam betriebene Komponenten (in scope, zwei Komponenten)**:

- **Keycloak**: Autorisierungsserver (1 Realm, Spitäler als
  Organisationen mit Affiliation als Token-Claim) inkl. Signing-Keys,
  Client-Registrierung, Token-Ausstellung, JWKS-Verteilung
- **mCSD-Registry**: öffentlicher FHIR-Server mit `Organization`,
  `Endpoint`, `HealthcareService` als verbindlicher Partnerverzeichnis-
  Dienst; autoritative Quelle für die Organisations-Referenzen in
  Cross-Party-Ressourcen

**Logische Aspekte in scope**:

- die **Cross-Party-Referenzlogik** zwischen den FHIR-Servern beider
  Spitäler (Referenzen `basedOn` / `sourceReference` / `subject`
  sowie `provision.actor`/`organization` über Server-Grenzen hinweg —
  letztere via Registry aufgelöst)
- die **technische Freigabe (`Consent`)** als spitalübergreifend
  ausgewertetes Autorisierungsartefakt

*Hinweis Sandbox vs. Produktion: Die Sandbox bündelt aus
operativen Gründen mehrere Komponenten in einer einzigen
HAPI-Instanz mit drei Partitionen. Produktiv betreibt jedes Spital
seinen **eigenen FHIR-Server und sein eigenes externes Gateway**;
die zwei gemeinsam betriebenen Komponenten (Keycloak und
mCSD-Registry) werden auf einer separaten, gemeinsamen Infrastruktur
betrieben.*

**Ausserhalb des Geltungsbereichs (out of scope — bestehende
Spital-Sicherheitskonzepte)**:

- das **interne Gateway** und alle dahinterliegenden spitalinternen
  Komponenten (UI → internes Gateway → eigener FHIR-Server,
  internes Gateway → OPA, internes Gateway → Keycloak)
- die **klinische Webapp / KIS-Integration** an spitalseitigen
  Endgeräten (Browser-Sicherheit, CSRF/XSS, Session-Handling,
  Frontend-Supply-Chain, lokale Speicherung)
- die **interne rollenbasierte Zugriffssteuerung** (welche
  Spital-Mitarbeiter:innen die Webapp/KIS nutzen dürfen, MFA-Roll-out,
  Berechtigungsreviews)
- die **spitalseitige Plattformadministration** (FHIR-Server-Betrieb,
  DB-Zugriff, Backup, Patching, PAM für spitalinterne Plattform-Admins)
- die **physische und Netzwerksicherheit** des Rechenzentrums
- die **allgemeine Endpoint-Sicherheit** klinischer Arbeitsplätze

Diese ausser-Scope-Themen sind durch die bestehenden ISMS / ISO 27001-
analogen Praxis der einzelnen Spitäler abgedeckt. Eine BRA pro Spital
liegt dort bereits vor und wird in dieser Cross-Party-BRA nicht
dupliziert. Bedrohungen, deren *Wirkung* aber den Cross-Party-Pfad
erreicht (z. B. kompromittierte Klinik-Session führt zur Ausstellung
einer ungewollten Freigabe), werden hier dennoch betrachtet — mit
expliziter Trennung von "Prävention" (Spital-Policy) und "Nachweis +
Schadensbegrenzung" (in dieser BRA).

### 1.2 Schutzziele (Priorisierung)

| Schutzziel | Priorität | Begründung |
|---|---|---|
| Vertraulichkeit | **sehr hoch** | Gesundheitsdaten = besondere Personendaten (IDG ZH § 3 Abs. 4) |
| Integrität | **sehr hoch** | Fehlerhafte klinische Daten = unmittelbares Patientensicherheitsrisiko; **manipulierte Placer-Freigaben** würden zu unautorisiertem Zugriff führen |
| Authentizität / Nachvollziehbarkeit | **sehr hoch** | Beweisbarkeit der Placer-Freigabe-Entscheidung, Auskunftsrecht IDG ZH § 20 |
| Verfügbarkeit | **hoch** | Klinischer Betrieb; Fallback Fax/HIN bleibt aber verfügbar |
| Nichtabstreitbarkeit | **hoch** | Cross-Party-Bekanntgabe muss beweisbar sein |

### 1.3 Abgrenzung — weiteres

Zusätzlich zu der in Ziffer 1.1 vorgenommenen Scope-Trennung sind
folgende Bereiche ebenfalls *out of Scope*:

- Klinikinformationssysteme (KIS) der einzelnen Spitäler
- Active Directory / Identity-Provider der einzelnen Spitäler
  (sofern nicht direkt am gemeinsamen Keycloak föderiert)
- Sekundärnutzung (Forschung, BI) — erfordert separate Rechtsgrundlage + DSFA

---

## 2. Bedrohungsakteure (Threat Model)

| ID | Akteur | Motivation | Fähigkeit | Realistisches Ziel |
|---|---|---|---|---|
| TA1 | **Externer Angreifer, opportunistisch** | finanzieller Gewinn, Ransomware | mittel | öffentlich erreichbare Endpunkte (externes Gateway, Webapp, Keycloak) |
| TA2 | **Externer Angreifer, gezielt** (APT) | Patientendaten Prominenter, Forschungs-IP, Erpressung | hoch | spezifische Patientenakten, Token-Diebstahl |
| TA3 | **Kompromittiertes Partnerspital** | unabsichtlich (Folge eines TA1/TA2 dort) | hoch (gültige Tokens, gültige Clients) | Datenabfluss über erweiterte Cross-Party-Anfragen |
| TA4 | **Insider — Klinikpersonal** *(Prävention spital-intern; in dieser BRA nur Wirkungspfad Cross-Party betrachtet)* | Neugier, Mobbing, Verkauf an Boulevard | tief–mittel | Ausstellung einer ungewollten Freigabe → Cross-Party-Bekanntgabe |
| TA5a | **Insider — Admin der gemeinsamen Komponenten** (Keycloak und/oder mCSD-Registry) | Erpressung, Selbstvermarktung, Rache | sehr hoch (Signing-Keys, Realm-Konfig, JWKS; Registry-Endpoint-Adressen) | gefälschte Tokens (KC), Umlenkung des Cross-Party-Traffics (Registry-Manipulation), beide Spitäler gleichzeitig betroffen |
| TA5b | **Insider — Spital-Plattform-Admin** *(spital-intern, out of scope; nur als Wirkungspfad: kann eigenen FHIR-Server / Audit-Logs manipulieren)* | wie TA5a | sehr hoch im eigenen Spital | spital-interne Spuren-/Datenmanipulation; Wirkung auf Cross-Party-Beweisbarkeit |
| TA6 | **Supply Chain** | Drittparteien-Kompromittierung (FHIR-Server-/Keycloak-Lib, OPA-/APISIX-Plugins, npm-Pakete) | hoch (transitives Vertrauen) | Backdoor, Datenexfiltration via Telemetrie |
| TA7 | **Patient / Patienten-Anwalt** (rechtlich legitim) | Auskunftsrecht, Berichtigung, Schadenersatz, Beanstandung der Placer-Freigabe | tief technisch, hoch juristisch | Beweis fehlerhafter Bekanntgabe → operatives Risiko bei mangelhaftem Audit oder bei Bekanntgabe ohne aktuelle Placer-Autorisierung |
| TA8 | **Aufsichtsbehörde** (DSB ZH) | Aufsicht, anlassbezogene Kontrolle | hoch (Rechtszugriff) | Auskunfts- und Auditierbarkeitsnachweis |

---

## 3. Assets

| ID | Asset | Datenklasse | Eigentümer | Schutzziele |
|---|---|---|---|---|
| A1 | **FHIR-Server Spital A** (eigene Instanz, Produktwahl spital-intern) — enthält klinische PD | besondere PD | Spital A | C, I, A |
| A2 | **FHIR-Server Spital B** (eigene Instanz, Produktwahl spital-intern) | besondere PD | Spital B | C, I, A |
| A3 | **Gemeinsames mCSD-Registry** (öffentlicher FHIR-Server für `Organization`/`Endpoint`/`HealthcareService`) — autoritativer Partnerverzeichnis-Dienst | öffentlich (Org-Stammdaten) | Betreiber gemeinsame Komponenten | **I sehr hoch**, A |
| A4 | DB-Backend der spitalseitigen FHIR-Server (z. B. Postgres) — Backup-Inhalt für Cross-Party-Audit relevant | besondere PD | je Spital | C, I, A |
| A5 | **Gemeinsamer Keycloak** (1 Realm, beide Spitäler als Organisationen; M2M-Clients, Users für Onboarding/Admin) | Identitätsdaten + Geheimnisse | Betreiber gemeinsame Komponenten | **C sehr hoch**, I, A |
| A6 | **Keycloak-Signing-Keys** (RS256) im gemeinsamen Keycloak — schützt das ganze Ökosystem | Geheimnis | Betreiber gemeinsame Komponenten | **C sehr hoch** |
| A8 | APISIX externes Gateway | Konfiguration + Tokens in flight + PD in flight | je Spital | C, I, A |
| A9 | OPA + Policies (am externen Gateway) | Code + Freigabe-Kontext (Placer-`Consent`) | je Spital | I, A |
| A11 | Audit-Logs am externen Gateway / Cross-Party-Audit | besondere PD (Pseudonyme + Bezugsdaten) | je Spital eigenständig (kein gemeinsamer SIEM-Eintritt) | **I sehr hoch**, C, A |
| A12 | Backups (eigener FHIR-Server + Cross-Party-Audit) | besondere PD | je Spital | C, I |
| A13 | Client-Private-Keys (Cross-Party-M2M) — Spital A und Spital B halten je ihren eigenen | Geheimnis | je Spital | **C sehr hoch** |

**Nicht in scope** (Verweis auf bestehende Spital-Asset-Register):
APISIX internes Gateway, Webapp / React-Bundle ausgeliefert an Browser,
KIS-Stammdaten, Active Directory der Spitäler, interne Endgeräte etc.

---

## 4. Angriffsflächen / Schnittstellen

| ID | Schnittstelle | Trust-Übergang | Schutz |
|---|---|---|---|
| I5 | Spital → externes Partner-Gateway (ausgehende Cross-Party-Anfrage) | spitalintern → öffentlich → fremdes Spital | TLS, **2 JWTs** (Hospital-Token + Client-Token), Token-Forwarding |
| I6 | externes Gateway ← Partner-Gateway (eingehende Cross-Party-Anfrage) | öffentlich → spitalintern | TLS, JWT-Validierung (RS256 + JWKS gemeinsamer Keycloak), OPA, Rate-Limit, WAF |
| I7 | externes Gateway → eigener FHIR-Server (Lesepfad für Cross-Party) | spitalintern | TLS, strict scope-check |
| I8 | externes Gateway → OPA | spitalintern | TLS lokal, schmale REST-API |
| I9 | externes Gateway → **gemeinsamer Keycloak** (JWKS / Discovery) | spitalintern → gemeinsamer Auth-Server | TLS, JWKS-Cache mit Rotations-Overlap |
| I10 | externes Gateway / OPA → **gemeinsames mCSD-Registry** (Lookup `Organization`/`Endpoint`) | spitalintern → gemeinsamer Registry-Dienst | TLS, no-auth, signed cache; Lookup-Logs |
| I11 | FHIR-Server → DB-Backend (z. B. Postgres) | spitalintern | TLS, Account-Trennung |
| I12 | Audit-Pfad: externes Gateway → eigene SIEM | spitalintern | TLS, mTLS empfohlen, WORM |
| I13 | Admin-Konsolen der **gemeinsamen Komponenten** (Keycloak Admin, mCSD-Registry-Pflege) | priv. Netz Betreiber | MFA (WebAuthn), IP-Allowlist, PAM |
| I14 | CI/CD → gemeinsame Komponenten (Realm- und Registry-Konfig-Deployment) | extern (GitHub/GitLab) → Betreiber-Plattform | OIDC-Federation, signierte Artefakte; spital-eigene CI/CD-Pfade out of scope |
| I15 | Backup → Cold Storage (Cross-Party-relevante Daten) | spitalintern → Storage | clientseitige Verschlüsselung |

**Nicht in scope** (in Sicherheitskonzept der jeweiligen Spitäler abgedeckt):
Browser → internes Gateway (vormals I1), internes Gateway → eigene
FHIR-Server (I2), internes Gateway → Keycloak für eigene UI-Logins
(I3), internes Gateway → OPA (I4), spital-interne Admin-Konsolen.

**Hauptrisikofläche**: **I5/I6** (Cross-Party-Pfad) und **I9/I13**
(gemeinsamer Keycloak — Common-Mode-Risiko für das gesamte Ökosystem).

---

## 5. Bedrohungskatalog

### 5.1 STRIDE pro Asset (Auszug, vollständig in Anhang A)

| Asset | S poofing | T ampering | R epud. | I nfo. discl. | D oS | E oP |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| A1/A2 FHIR-Server (Cross-Party-Lesepfad) | T1 | T6 | T11 | **T15** | T20 | T25 |
| A5 Gemeinsamer Keycloak | T2 | **T7** | T12 | T16 | **T21** | T26 |
| A6 Signing Keys (im gemeinsamen KC) | — | **T8** | — | **T8** | — | **T8** |
| A8 externes Gateway | T4 | T10 | T14 | **T18** | T23 | T28 |
| A9 OPA + Policies | — | **T9b** | — | — | T24 | T29 |
| A11 Audit-Logs | — | **T_AUD** | **T_AUD** | T19 | — | — |

### 5.2 Konkrete Bedrohungen — UMZH-spezifisch

| # | Bedrohung | STRIDE | OWASP API | Akteur | Auswirkung |
|---|---|---|---|---|---|
| **B01** | **Scope-Bypass via `_include`** — Angreifer ergänzt erlaubten `GET ServiceRequest/X` um `_include=ServiceRequest:subject:Patient&_include=…`, um Daten zu erhalten, die nicht vom Konsent gedeckt sind | I | API1 BOLA / API3 Object Property | TA3 | breite Datenabflüsse |
| **B02** | **Search-Parameter-Injection** — `_filter=`, FHIRPath-Engine, oder DB-Suchstring führen zu unkontrollierter Query | T, I | API8 | TA1, TA3 | Datenabfluss, ggf. DoS |
| **B03** | **Stale Placer-Freigabe** — Placer-Arzt zieht die Freigabe zurück, schränkt sie ein oder die Periode läuft ab; Cache am Gateway/OPA gibt weiter `permit` zurück | I, R | API1 | unbeabsichtigt; TA7 auf Folgenebene | Bekanntgabe ausserhalb des aktuellen Placer-Willens; Verletzung der Verhältnismässigkeit |
| **B04** | **Token-Replay** über Netzwerk-/Log-Diebstahl | S, I | API2 | TA1, TA5 | Identitätsübernahme |
| **B05** | **JWT-Algorithm-Confusion / `alg=none`** an einem Hop | S | API2 | TA1 | vollständiger AuthZ-Bypass |
| **B06** | **Stolen `client_secret` / Private Key** des Partner-M2M-Clients | S | API8 | TA2, TA5 | Identitätsübernahme Partnerspital |
| **B07** | **Freigabe-Schmuggel** — Angreifer setzt `X-Consent-Id`-Header auf eine fremde, formal gültige Placer-Freigabe (eines anderen Patienten / anderen Empfängers); OPA prüft nicht alle Bindungen (Patient ↔ Freigabe, Empfänger-Org ↔ Token-Org, Ressourcenklasse ↔ Whitelist) | I, E | API5 | TA3 | unautorisierte Bekanntgabe |
| **B08** | **Cross-Party-Reference Abuse** — manipulierte `subject.reference` / `basedOn.reference` über Server-Grenzen hinweg; Empfänger interpretiert Referenz auf einen Patienten, der dem Auftrag nicht entspricht | T, E | API3 | TA3 | Datenverwechslung |
| **B09** | **Registry-Cache-Poisoning** — Manipulation der zwischengespeicherten mCSD-Daten (Endpoint-Adresse, JWKS-URL) auf einem Spital-Gateway, sodass eine veraltete oder gefälschte Sicht auf das gemeinsame Registry verwendet wird | T, S | API8 | TA2, lokal | Umlenkung Datenströme, MitM |
| **B10** | **`$expunge` / Mass-Delete am FHIR-Server durch Insider** — Spuren-/Datenvernichtung; in dieser BRA *nur* sofern Cross-Party-relevantes Audit-Material betroffen wird (sonst out of scope, Spital-Policy) | T, R, E | API5 | TA5b | Beweismittelvernichtung gegenüber Partnerspital und DSB |
| **B11** | **Logging von PHI im Klartext** am externen Gateway / im Cross-Party-Audit (URL-Parameter, Stacktraces) → Logs werden zur sekundären Datenklasse | I | API8/9 | unbeabsichtigt | DSFA-Verstoss, Compliance |
| ~~B12~~ | ~~Webapp Supply Chain — kompromittiertes npm-Paket exfiltriert Tokens aus `localStorage`~~ | — | — | — | **Out of scope (Webapp wird durch Spital-Sicherheitskonzept abgedeckt).** Falls Wirkung auf Cross-Party (Tokenmissbrauch): siehe B04/B20. |
| ~~B13~~ | ~~CSRF / Click-jacking der internen Webapp~~ | — | — | — | **Out of scope (internes UI).** |
| **B14** | **Ransomware auf den FHIR-Server eines Spitals oder das Cross-Party-Audit-Backup** | D, T | — | TA1 | klinischer Ausfall des Cross-Party-Pfads, Erpressung; klinischer Fallback Mail/Fax bleibt |
| **B15** | **Patient-ID-Enumeration** an externem Gateway (timing-/error-basiert) | I | API2 | TA2, TA3 | Existenznachweis = Datenpunkt |
| **B16** | **OPA-Policy-Bug** — Edge Case (Konsent ohne `period.end`, "permit" mit leerem `purpose`) wird zu unkontrolliertem `allow` | I, E | API5 | unbeabsichtigt | systematische Fehlentscheide |
| **B17** | **MitM auf Cross-Party-Pfad** trotz TLS (CA-Kompromiss, Captive Portal) | S, I | — | TA2 | Token-/PHI-Mitschnitt |
| **B18** | **Audit-Log-Tampering** durch privilegierten Admin | T, R | — | TA5 | Beweismittelmanipulation |
| **B19** | **Denial of Service** an externem Gateway / FHIR-Server / gemeinsamem Keycloak | D | API4 | TA1 | klinischer Ausfall des Cross-Party-Pfads, Reputationsschaden |
| **B20** | **Insider-Phishing** auf Klinikpersonal → gestohlene Session führt zur Ausstellung einer ungewollten Freigabe (Cross-Party-Wirkung) | S, E | API2 | TA2 (vorgelagert), TA4 (Ziel) | Bekanntgabe ohne tatsächliche ärztliche Entscheidung. **Prävention durch Spital-Policy (MFA, Schulung); Schadensbegrenzung Cross-Party durch C-22, U-11, U-12.** |
| **B21** | **Backup-Diebstahl** (gestohlene Tape/Snapshot, ungeschützter S3-Bucket) | I | — | TA1, TA5 | Komplettdatenabfluss |
| ~~B22~~ | ~~Übermässige Rechte — `placer-user` darf Cross-Party-Schreibzugriff, der nicht benötigt wird~~ | — | — | — | **Out of scope (interne RBAC).** In dieser BRA wird stattdessen geprüft, dass das **externe Gateway** Schreibzugriffe vom Partnerspital generell ablehnt — vgl. M01/M03. |
| **B23** | **Manipulation der Placer-Freigabe nach Ausstellung** — Insider (TA4/TA5) ändert `Consent.provision.class` oder `provision.period.end` einer fremden Freigabe, um Lesezugriff über das ursprünglich vom Arzt Freigegebene hinaus zu ermöglichen | T, E | API5 | TA4, TA5 | unautorisierte Bekanntgabe, kompromittierter Beweisstand |
| **B24** | **Unautorisierte Ausstellung einer Placer-Freigabe** — Insider ohne klinische Rolle (oder mit übernommener Klinik-Session, vgl. B20) erstellt eine `Consent`, ohne dass ein zuweisender Arzt diese Entscheidung getroffen hat | S, E | API5 | TA4, TA5 (post B20) | Bekanntgabe ohne ärztliche Legitimation |
| **B25** | **Stille Freigabe-Ausweitung** — Placer-UI fügt ohne Wissen des Arztes weitere Ressourcenklassen / längere Geltungsdauer in `Consent.provision` ein (z. B. via kompromittierten Webapp-Code oder Default-Werte). Prävention im Webapp-Code = Spital-Policy; server-seitige Sanity-Checks am externen Gateway / im Placer-Backend bleiben hier in scope. | T | API5/8 | TA6, unbeabsichtigt | systematische Über-Freigabe |
| **B26** | **Kompromittierung des gemeinsamen Keycloak** — Common-Mode-Risiko: Diebstahl der Signing-Keys oder Manipulation der Realm-Konfiguration (z. B. Hinzufügen eines bösartigen Clients, Änderung der Token-Mapper, Deaktivierung der `private_key_jwt`-Anforderung). Wirkung: beide Spitäler gleichzeitig betroffen. | S, T, E | API2/8 | TA2, TA5a | flächendeckende Identitätsübernahme, gefälschte M2M-Tokens, langanhaltend unentdeckt |
| **B27** | **Manipulation des gemeinsamen mCSD-Registry-Inhalts** durch Insider beim Betreiber — falsche `Endpoint.address` / JWKS-Verweis für ein Spital eingetragen, Umlenkung des Cross-Party-Traffics | T, S | API8 | TA5a | MitM, Token-Fehlleitung, Phishing-ähnliche Effekte zwischen Spitälern |

---

## 6. Risikobewertung

**Skala**: Eintrittswahrscheinlichkeit W ∈ {1=selten, 2=möglich, 3=wahrscheinlich, 4=fast sicher},
Schadenshöhe S ∈ {1=gering, 2=mittel, 3=hoch, 4=existenziell}.
Risikowert R = W × S. Kategorisierung:

- **R ≤ 4**: akzeptabel
- **5–8**: tolerierbar mit Kontrolle
- **9–12**: signifikant → Massnahme erforderlich
- **≥ 13**: kritisch → vor Go-Live zu adressieren

**Bewertung nach Umsetzung der DSFA-Massnahmen M01–M15** (Residualrisiko):

| # | Bedrohung | W | S | R | Klasse | Primäre Kontrolle (M/P) |
|---|---|:---:|:---:|:---:|---|---|
| B01 | Scope-Bypass via `_include` | 2 | 4 | 8 | tolerierbar | M03 OPA, M06 Scopes, **neu C-01**: `_include`-Whitelist |
| B02 | Search-Parameter-Injection | 2 | 3 | 6 | tolerierbar | M03, **C-02** strikte Param-Allowlist |
| B03 | Konsent-Cache-Stale | 2 | 4 | 8 | tolerierbar | **P04** Widerruf-Propagation |
| B04 | Token-Replay | 1 | 4 | 4 | akzeptabel | M07 kurze TTL, **C-03** `jti`-Replay-Cache |
| B05 | JWT alg-Confusion | 1 | 4 | 4 | akzeptabel | M02 strenge Validierung (JWKS, `alg`-Allowlist) |
| B06 | Stolen Private Key | 1 | 4 | 4 | akzeptabel | M08 `private_key_jwt`, **P01** HSM |
| B07 | Konsent-Schmuggel via Header | 2 | 4 | 8 | tolerierbar | M03, **C-04** Konsent-zu-Token-Binding |
| B08 | Cross-Party-Reference Abuse | 1 | 3 | 3 | akzeptabel | M04 (getrennte Server), M12 (keine spitalübergreifenden IDs) |
| B09 | Registry-Cache-Poisoning (lokal am Spital-Gateway) | 1 | 4 | 4 | akzeptabel | **C-05** signierte mCSD-Manifeste / TOFU + Pinning der Registry-JWKS |
| B10 | `$expunge` am FHIR-Server durch Insider (Cross-Party-Audit-Wirkung) | 2 | 4 | 8 | tolerierbar | **P06** PAM (für Cross-Party-relevante Daten), **C-06** Vier-Augen-Prinzip für `$expunge`; allgemeine Prävention via Spital-Policy |
| B11 | PHI in Logs (am externen Gateway / Cross-Party-Audit) | 3 | 3 | 9 | **signifikant** | **C-07** strukturierte Logs, PHI-Redaction-Filter, Schulung |
| ~~B12~~ | ~~Webapp Supply Chain~~ | — | — | — | Out of scope (Spital-Policy) |
| ~~B13~~ | ~~CSRF / Click-jacking~~ | — | — | — | Out of scope (internes UI) |
| B14 | Ransomware (FHIR-Server eines Spitals / Cross-Party-Audit-Backup) | 2 | 4 | 8 | tolerierbar | **P08** Backup-Konzept, **C-11** immutable / air-gapped Backups, Restore-Drill |
| B15 | Patient-ID-Enumeration | 2 | 2 | 4 | akzeptabel | **C-12** uniforme Antwortzeiten, generische 404/403 |
| B16 | OPA-Policy-Bug | 2 | 4 | 8 | tolerierbar | **C-13** Policy-Unit-Tests, Decision-Logs, Property-based Tests |
| B17 | MitM auf Cross-Party | 1 | 4 | 4 | akzeptabel | M10 TLS, **P02** mTLS, **C-14** Certificate Transparency Monitoring |
| B18 | Audit-Log-Tampering | 2 | 4 | 8 | tolerierbar | **P03** WORM, **C-15** Log-Signing / Hash-Chain |
| B19 | DoS | 3 | 2 | 6 | tolerierbar | **C-16** Rate-Limits am Edge, Circuit-Breaker, autoscaling |
| B20 | Insider-Phishing (Wirkung Cross-Party) | 2 | 3 | 6 | tolerierbar | Prävention out of scope (Spital-Policy: MFA, Schulung); Schadensbegrenzung Cross-Party via **C-22**, **U-11/U-12** |
| B21 | Backup-Diebstahl | 1 | 4 | 4 | akzeptabel | **C-18** clientseitige Verschlüsselung, Key-Separation |
| ~~B22~~ | ~~Übermässige Rechte~~ | — | — | — | Out of scope (interne RBAC); externes Gateway lehnt Cross-Party-Schreibzugriffe systematisch ab |
| B23 | Manipulation der Placer-Freigabe nach Ausstellung | 2 | 4 | 8 | tolerierbar | **C-20** `Consent`-Änderungen versionieren + signieren, Vier-Augen bei nachträglicher Erweiterung, Audit-Alarm |
| B24 | Unautorisierte Ausstellung einer Freigabe | 2 | 4 | 8 | tolerierbar | **C-21** `Consent.performer` muss aktiver klinischer Account sein (server-seitige Pflichtfeld-Prüfung im Placer-Backend); UI-Login-/Step-up-Anforderungen sind Spital-Policy |
| B25 | Stille Freigabe-Ausweitung durch UI/Defaults | 2 | 3 | 6 | tolerierbar | **C-22** Server-seitige Sanity-Checks (keine "ALL"-Defaults, Begründungsfeld bei breiter Freigabe); UI-Bestätigungsdialog ist Spital-Policy |
| B26 | Kompromittierung des gemeinsamen Keycloak | 1 | 4 | 4 | akzeptabel (mit Auflagen) | **P01** HSM-Schlüssel, **P11** Governance, **C-23** Realm-Konfig in Git mit Vier-Augen, **C-24** unabhängige Token-Anomalie-Detektion, jährlicher externer Audit |
| B27 | Manipulation des gemeinsamen mCSD-Registry-Inhalts | 1 | 4 | 4 | akzeptabel (mit Auflagen) | **C-05** signierte Manifeste, **C-23** Registry-Inhalt in Git + Drift-Detection, **P11b** Joint-Governance-Change-Board für Registry-Mutationen |

**Top-Restrisiken** (vor Go-Live mit konkretem Massnahmen-Owner):

1. **B11 PHI in Logs** (R=9) — C-07 sofort umsetzen, Schulung verpflichtend
2. **B26 Kompromittierung gemeinsamer Keycloak** (R=4 mit Auflagen) — niedrige Wahrscheinlichkeit dank HSM und Governance, aber höchste Schadenshöhe (beide Spitäler gleichzeitig). C-23, C-24, C-25 + P01 + P11 müssen vor Go-Live umgesetzt sein; externer Audit halbjährlich.
3. **B01/B03/B07/B10/B14/B16/B18/B23/B24 — Cluster R=8** — vollständige Abarbeitung der C-01 bis C-25 vor Go-Live; Nachweis durch Penetrationstest (DSFA P07).
   Besondere Aufmerksamkeit auf den **Placer-Freigabe-Ausstellungspfad** (B23/B24): er ist der einzige Punkt, an dem eine ungültige Bekanntgabe entsteht, ohne dass das externe Gateway dies erkennen kann — denn das Gateway *vertraut* einer formal gültigen Freigabe.

**Aus der Wirkung in scope, Prävention out of scope** (in DSFA als
Restrisiken auf Spital-Verantwortung dokumentiert):

- B20 Insider-Phishing (Spital-Policy: MFA, Schulung); Cross-Party-
  Wirkung wird über C-22, C-24, U-11/U-12 detektiert
- Kompromittierte interne Webapp / Endgeräte (Spital-Policy)
- Über-/unter-privilegierte interne Nutzer:innen (Spital-RBAC)

---

## 7. Neue Kontrollen (C-01 bis C-19)

| # | Kontrolle | Schicht | Verifikation | Adressiert |
|---|---|---|---|---|
| C-01 | `_include`-Allowlist am externen Gateway: Nur `ServiceRequest:subject:Patient`, `ServiceRequest:requester:Practitioner`. Alle anderen Includes → 403. | APISIX Plugin | Test in `tests/hurl/` | B01 |
| C-02 | Strikte FHIR-Search-Param-Allowlist je Ressource am externen Gateway; Ablehnung unbekannter Parameter | APISIX Plugin | Negativ-Test | B02 |
| C-03 | JWT-`jti` Replay-Cache (Redis, TTL = Token-Lebensdauer) | APISIX / OPA | Lasttest mit wiederholtem Token | B04 |
| C-04 | Freigabe-zu-Token-Binding: `X-Consent-Id` muss in *allen* relevanten Dimensionen zur Anfrage passen — Empfänger-Org in `Consent.provision.actor` = Org des aufrufenden M2M-Clients (OAuth-`client_id`/Token-Org-Claim); Patient in `Consent.patient` = Subject der angefragten Ressource; Ressourcen-Typ in `Consent.provision.class`-Whitelist. Sämtliche Prüfungen in OPA-Rego (keine Verlagerung an die Anwendung). | OPA | Policy-Unit-Test | B07 |
| C-05 | **Signierte mCSD-Manifeste** (JWS) oder TOFU + Pinning der Registry-JWKS am Spital-Gateway; lokale Caches sind signiert; Abweichungen vom autoritativen Registry → Alarm | gemeinsames Registry + externes Gateway / OPA-Konfig | manueller Verifikationstest | B09, B27 |
| C-06 | `$expunge` und Mass-Delete am spitalseitigen FHIR-Server erfordern Vier-Augen-Token (zwei unabhängige Admin-Sessions), wenn Cross-Party-relevantes Audit-Material betroffen ist; konkrete Umsetzung je nach gewähltem FHIR-Produkt durch das Spital | FHIR-Server / PAM | Audit-Review | B10 |
| C-07 | Strukturiertes JSON-Logging mit PHI-Redaction-Filter (`Patient.name`, `birthDate`, `identifier` etc. masked); URLs nie mit Body-Inhalt loggen | App / Gateway | Stichproben in SIEM | B11 |
| ~~C-08~~ | SBOM/SRI/CSP für Webapp | Webapp | — | **Out of scope** (Spital-Policy je Webapp-Auslieferer) |
| C-08b | SBOM (CycloneDX) für die **gemeinsamen Komponenten** (Keycloak, mCSD-Registry), automatisierter Vuln-Scan (Trivy/Grype) durch den Betreiber. SBOM/Vuln-Management für spital-eigene Komponenten (externes Gateway, FHIR-Server, OPA) ist Teil der jeweiligen Spital-Policy. | CI/CD Betreiber | wöchentlicher Report | B26 |
| ~~C-09~~ | Token-Speicherung in der Webapp | — | — | **Out of scope** |
| ~~C-10~~ | Cookie-/Frame-Header der Webapp | — | — | **Out of scope** |
| C-11 | Immutable Backups (Object Lock), zusätzlich offline / air-gapped, halbjährlicher Restore-Drill | Backup-Stack | Drill-Protokoll | B14 |
| C-12 | Uniforme Antwortzeiten / generische `403`/`404` am externen Gateway; keine "resource not found"-vs-"forbidden"-Unterscheidung | APISIX | Timing-Test | B15 |
| C-13 | OPA-Policies in Git, Pull-Request-Pflicht, Property-based Tests (z. B. Rego coverage ≥ 90 %), Decision-Logs ans SIEM | OPA-Pipeline | CI-Report | B16 |
| C-14 | Certificate Transparency Monitoring + Pinning der Partner-Endpoint-Zertifikate | Operativ | Alarmierung im SIEM | B17 |
| C-15 | Hash-Chain (Merkle) auf Audit-Logs, periodische Signatur, externes Notarisieren | SIEM-Pipeline | Verifikations-Tool | B18 |
| C-16 | Rate-Limits am eigenen externen Gateway (per Client und per Patient-ID-Hash), Circuit-Breaker, Skalierungs-/HA-Konzept der spitalseitigen Komponenten; HA-Konzept der gemeinsamen Komponenten (Keycloak + mCSD-Registry) auf Betreiberseite | APISIX / Plattform | Lasttest | B19 |
| ~~C-17~~ | MFA für Webapp-Nutzer | — | — | **Out of scope** (Spital-Policy); für Admins der gemeinsamen Komponenten (Keycloak + Registry) jedoch verbindlich, siehe C-25 |
| C-18 | Backup-Verschlüsselung clientseitig, Schlüssel im HSM (separater Trust-Anchor), Key-Custody dokumentiert | Backup-Stack | Audit | B21 |
| C-19 | Jährlicher Berechtigungsreview (Keycloak-Rollen, OPA-Datenfeeds), Just-in-Time-Privilegieren für Admin-Aufgaben | Operativ + PAM | Review-Bericht | B22 |
| C-20 | **Placer-Freigabe-Integrität**: jede `Consent`-Mutation am spitalseitigen FHIR-Server versioniert (`Consent.meta.versionId`), `Provenance` mit `who`/`agent` ans Audit; nachträgliche Erweiterung (`class` hinzufügen, `period.end` verlängern) erfordert Bestätigung durch zweiten klinischen Account oder dokumentierten Auftrag des ursprünglichen `performer`. Konkrete Umsetzung ist Produktwahl/Konfiguration des Placer-FHIR-Servers. | spitalseitiger FHIR-Server + Audit | Property-Test, Audit-Stichprobe | B23 |
| C-21 | **Authentische Freigabe-Ausstellung (server-seitig)**: Das Placer-FHIR-Server-/Vorvalidierungs-Layer lehnt `POST Consent` ohne gesetztes `Consent.performer` ab, prüft, dass der Performer eine aktive `Practitioner`/`PractitionerRole` ist, und dass der Token-`sub` zum Performer in einer dokumentierten Beziehung steht. UI-Login-Pflicht und Step-up-Auth sind Sache der Spital-Policy. | spitalseitiger FHIR-Server / Vorvalidierung | Negativtest mit Service-Account | B24 |
| C-22 | **Server-seitige Sanity-Checks für die Freigabe**: Placer-Backend / Gateway lehnt offensichtlich überweite Freigaben (z. B. `provision.class` enthält die maximale Menge, `period` > 1 Jahr) ohne explizites Begründungsfeld ab; bei nachträglicher Erweiterung Pflicht zur expliziten neuen Version (kein in-place Update). UI-seitige Bestätigungsdialoge sind Spital-Policy. | Placer-Backend | Property-Test | B25 |
| C-23 | **Konfiguration der gemeinsamen Komponenten in Git** mit Pull-Request-Pflicht und Vier-Augen-Review: Keycloak-Realm (Clients, Scopes, Mapper, Authentication-Flows) **und** mCSD-Registry-Inhalt (`Organization`, `Endpoint`, `HealthcareService`) versioniert und nachvollziehbar; Drift-Detection (laufender Stand vs. Git) als wöchentlicher Job | Betreiber / GitOps | Drift-Report | B26, B27 |
| C-24 | **Unabhängige Token-Anomalie-Detektion**: am externen Gateway zusätzlich zu KC-Logs eine eigene Heuristik (z. B. Anstieg `iss`-Mismatch, neue `kid` ohne Voranmeldung, sprunghafte Verschiebung Token-Volumen pro Client), Alarm bei Auffälligkeit unabhängig von KC | APISIX / SIEM | Lasttest, Detection-Drill | B26 |
| C-25 | **MFA-Pflicht für Admins der gemeinsamen Komponenten** (Keycloak und mCSD-Registry), WebAuthn / Passkey bevorzugt; Just-in-Time-Aktivierung; alle Admin-Aktionen ans SIEM des Betreibers, gespiegelt zu allen angeschlossenen Spitälern | Betreiber | Audit | B26, B27, R8 |

---

## 8. SIEM-Use-Cases (Detektion)

Zur Operationalisierung von DSFA-Massnahme **P05**. Mindestkatalog:

| # | Use Case | Datenquelle | Schwelle / Logik |
|---|---|---|---|
| U-01 | OPA-Deny-Spike | OPA Decision Log | > N Denies/Min für gleichen Client → Alarm |
| U-02 | Token-Anomalie | Keycloak Event-Log | Token-Ausstellung ohne vorheriges Login (M2M only) ausserhalb Standardzeiten |
| U-03 | Cross-Party-Volumen | externes Gateway Access-Log | Anstieg > X-fache Baseline pro Stunde |
| U-04 | Patient-ID-Enumeration | externes Gateway | > N 404 für `Patient/*` von gleichem Client in Y Min |
| U-05 | Konsent-Replay | OPA / Gateway | gleiche `X-Consent-Id` aus ≥ 2 unterschiedlichen Client-IDs |
| U-06 | `$expunge` / Mass-Delete | FHIR-Server Access-Log | jede solche Operation → Alarm + Ticket |
| U-07 | Privileged Login | Keycloak / PAM | Admin-Login ausserhalb Wartungsfenster |
| U-08 | Geo-Anomalie | Keycloak / Gateway | Login von neuer Geografie / Impossible Travel |
| U-09 | Backup-Zugriff | Storage-Audit | unerwarteter Lesezugriff auf Backup-Bucket |
| U-10 | Log-Chain-Bruch | Audit-Pipeline | Hash-Kette inkonsistent → unmittelbarer P1-Alarm |
| U-11 | Auffällige Freigabe-Muster (Placer) | FHIR-Server-Audit + Webapp-Log | Ungewöhnlich breite `Consent.provision.class`, ungewöhnlich lange `period.end`, ungewöhnlich viele Freigaben pro Arzt/Tag |
| U-12 | Nachträgliche Freigabe-Erweiterung | FHIR-Server-Versionen + `Provenance` | `Consent`-Update mit zusätzlichen Klassen oder verlängerter Periode → Review-Workflow + Audit-Alarm |
| U-13 | Anomalien am gemeinsamen Keycloak | Keycloak Admin-Event-Log + Drift-Detection | unangekündigte Realm-Mutation, neuer Client ausserhalb Onboarding-Fensters, neue `kid` ohne Voranmeldung → unmittelbarer P1-Alarm an alle angeschlossenen Spitäler |
| U-14 | Geänderter Endpoint im gemeinsamen mCSD-Registry | Registry-Audit + Drift-Detection | Mutation eines `Endpoint.address` ohne dokumentierten Change-Request → Sperre + manuelle Bestätigung der angeschlossenen Spitäler |

---

## 9. Restrisiken und Risikoakzeptanz

Nach Umsetzung aller Massnahmen aus DSFA (M01–M15, P01–P15) und BRA
(C-01 bis C-19) verbleiben folgende **akzeptierte Restrisiken**:

| Restrisiko | Begründung der Akzeptanz | Akzeptanz durch |
|---|---|---|
| Insider-Missbrauch durch klinisches Personal mit legitimem Behandlungsbezug | Kontrolle nur durch Audit/Stichprobe möglich, ohne klinischen Workflow zu blockieren | {{CISO / DSV}} |
| Restwahrscheinlichkeit Zero-Day in FHIR-Server-Produkten / Keycloak / APISIX | Industrieller Stand, Patching-Prozess vorhanden bei jeweiligem Betreiber | {{CISO}} |
| Freigabe-Invalidierungsverzögerung ≤ 60 s (Placer zieht zurück → Gateway verweigert) | klinisch und rechtlich vertretbar (Praxisstandard; vergleichbar mit der Verzögerung beim Rückruf eines bereits versendeten Fax) | {{DSV}} |
| Restrisiko, dass ein klinisch berechtigter Placer-Arzt eine inhaltlich falsche Freigabe-Entscheidung trifft | wie bei jeder Mail/Fax-Zuweisung — Teil der ärztlichen Verantwortung; durch Schulung (P10), UI-Bestätigung (C-22) und Audit (U-11/U-12) reduziert, nicht eliminierbar | {{Ärztliche Direktion + DSV}} |
| Fehlerhafte klinische Daten aufgrund menschlicher Eingabe | ausserhalb der Plattformverantwortung | {{Ärztliche Direktion}} |

---

## 10. Wiederholung und Pflege

| Anlass | Massnahme |
|---|---|
| **jährlich** | Vollreview dieser BRA, Aktualisierung Bedrohungskatalog |
| **bei wesentlicher Architekturänderung** (neue Schnittstelle, neuer Akteur, neuer Datenklasse) | Delta-BRA mit Re-Bewertung der betroffenen Bedrohungen |
| **nach Sicherheitsvorfall** | Lessons Learned in B-Katalog überführen, neue Kontrolle prüfen |
| **nach Penetrationstest** (P07) | Findings als neue Bedrohungen aufnehmen, Re-Test der Kontrollen |
| **vor Aufnahme eines weiteren Spitals** | Aktualisierung Akteur TA3, Re-Scoping, vermutlich neue DSFA-Iteration |

---

## Anhang A — STRIDE-Vollmatrix (kompakt)

| STRIDE | Bedrohung (Beispiel) | UMZH-spezifischer Vektor |
|---|---|---|
| **S**poofing | Identität vortäuschen | gestohlene Tokens, gestohlene Client-Keys, gestohlene Webapp-Session |
| **T**ampering | Daten/Code/Config manipulieren | Manipulation `Consent.status`, Policy-Bypass, Registry-Eintrag |
| **R**epudiation | Aktion abstreiten | fehlendes Audit, manipulierbares Log |
| **I**nformation Disclosure | unautorisiert lesen | Scope-Bypass, `_include`-Abuse, PHI in Logs, MitM |
| **D**enial of Service | Verfügbarkeit angreifen | DDoS gegen externes Gateway, Ressourcen-Exhaustion FHIR-Queries |
| **E**levation of Privilege | Rechte ausweiten | OPA-Lücke, ungeschützte Admin-API, Container-Escape |

## Anhang B — Mapping auf OWASP API Security Top 10 (2023)

| OWASP | Adressierte Bedrohungen |
|---|---|
| API1 Broken Object Level Authorization | B01, B03, B07, B08 |
| API2 Broken Authentication | B04, B05, B15, B20 |
| API3 Broken Object Property Level Authorization | B01, B08 |
| API4 Unrestricted Resource Consumption | B19 |
| API5 Broken Function Level Authorization | B07, B10, B16, B22 |
| API6 Unrestricted Access to Sensitive Business Flows | B10 |
| API7 Server-Side Request Forgery | (low) |
| API8 Security Misconfiguration | B02, B06, B09 |
| API9 Improper Inventory Management | C-08 (SBOM) |
| API10 Unsafe Consumption of APIs | B12, B17 |

## Anhang C — Verweis auf existierende Bausteine

| Bedrohung / Kontrolle | Fundstelle |
|---|---|
| Dual-Gateway-Pattern | [sandbox-architecture-and-requirements.md](sandbox-architecture-and-requirements.md) §Components — API management |
| OAuth-Level-Modell (1/2/3) | [security-concept.md](security-concept.md) — Stepwise security up-leveling |
| Konsent-Modell und OPA-Policies | [authorization-scenarios.md](authorization-scenarios.md) + `services/opa/policies/` |
| Sandbox-Tests Cross-Party | `tests/hurl/05-cross-party-consent.hurl` |
| mCSD-Datenmodell (Organization/Endpoint) | `services/seed/bundles/registry-bundle.json` — Inhalt des gemeinsamen mCSD-Registry |

---

*Diese BRA ist ein lebendiges Dokument. Sie wird im Zuge des
Penetrationstests (DSFA P07), nach jeder substanziellen
Architekturänderung sowie mindestens jährlich überarbeitet. Die
Owner-Spalten und Datumsfelder werden im Zuge der finalen Abstimmung
zwischen {{Spital A}} und {{Spital B}} gefüllt.*
