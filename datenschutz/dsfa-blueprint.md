# DSFA-Blueprint — UMZH Connect Produktivumgebung

> Vorlage für die Datenschutz-Folgenabschätzung (DSFA) gemäss Formular der
> Datenschutzbeauftragten des Kantons Zürich (Stand 2024) für die produktive
> Inbetriebnahme des UMZH-Connect-Datenaustauschs zwischen zwei Spitälern
> auf Basis der Sandbox-Referenzarchitektur.
>
> **Rechtsgrundlagen**: IDG ZH (LS 170.4), insb. §§ 7, 8, 9, 10, 17;
> revFADP (SR 235.1) subsidiär; SMART on FHIR (HL7); UMZH Implementation Guide.
>
> **Konventionen**:
> - `{{Platzhalter}}` = vor Einreichung durch das jeweilige Spital zu ergänzen
> - Querverweise auf bestehende Dokumente: [https://build.fhir.org/ig/umzhconnect/umzhconnect-ig/security.html](UMZH Security Concept),
>   [reference-architecture.md](Referenz-Architektur)

---

## 1. Angaben zum verantwortlichen öffentlichen Organ

Eine DSFA wird pro verantwortliches öffentliches Organ erstellt. Bei zwei
beteiligten Spitälern sind grundsätzlich **zwei DSFA** auszufüllen (je
eine pro Spital als Verantwortlicher seiner eigenen Patientendaten). Die
nachfolgenden Inhalte sind für beide Spitäler weitgehend identisch und
können wechselseitig referenziert werden.

| Feld | Angabe |
|---|---|
| Verantwortliches Organ | {{Spital A}} und/oder {{Spital B}}|
| Kontaktperson | {{Vor- und Nachname, Funktion}} |
| Telefonnummer | {{+41 ...}} |
| E-Mail-Adresse | {{datenschutz@spital.ch}} |

**Hinweis**: Die DSB Kanton ZH erwartet einen benennbaren *natürlichen*
Ansprechpartner, nicht nur eine Funktionsmailbox.

---

## 2. Sachverhalt

> *Beschreiben Sie präzise den Sachverhalt des Projekts mit Fokus auf
> die neuen oder angepassten Datenbearbeitungen.*

Die beiden Spitäler {{Spital A}} und {{Spital B}} führen einen
**zuweisungsbasierten, konsentierten klinischen Datenaustausch** ein, der
es einem zuweisenden Arzt (Placer) erlaubt, einen Auftrag (z. B.
orthopädische Zuweisung) elektronisch an ein leistungserbringendes Spital
(Fulfiller) zu übermitteln und letzterem den lesenden Zugriff auf
genau die für den Auftrag relevanten Patientendaten zu gewähren.

Technisch erfolgt der Austausch über **HL7 FHIR** mit folgenden
Kernressourcen:

- `ServiceRequest` (Auftrag, beim Placer angelegt)
- `Consent` — **technische Freigabe-/Autorisierungsressource**, beim
  Placer angelegt, referenziert den `ServiceRequest`. Sie ist *kein*
  patientenseitig unterschriebenes Einwilligungsdokument, sondern eine
  vom zuweisenden Arzt ausgestellte, maschinenlesbare Erklärung "ich
  gebe folgende Daten zu folgendem Zweck an folgendes Spital für
  folgenden Zeitraum frei" — funktional analog zum heutigen Versand
  einer Zuweisung per Fax/HIN-Mail mit den klinischen Beilagen.
- `Task` (Bearbeitungsauftrag, beim Fulfiller angelegt, mit `basedOn`-Referenz auf den ServiceRequest und Freigabe-ID in `meta.security`)
- begleitende Ressourcen (`Patient`, `Practitioner`, `PractitionerRole`, `Condition`, `Observation` etc.) — nur im **jeweils eigenen FHIR-Server** des Spitals; Zugriff durch das Partnerspital ausschliesslich lesend, durch die Placer-Freigabe gedeckt und auftragsgebunden.

> Jedes Spital betreibt seinen
> eigenen FHIR-Server *und* sein eigenes externes Gateway**.
> **Gemeinsam betriebene Komponenten** sind ausschliesslich:
> (1) der **Autorisierungsserver (Keycloak)** und
> (2) das **öffentliche mCSD-Registry** (`Organization`/`Endpoint`/
> `HealthcareService`) als verbindliches Partnerverzeichnis. Alle
> anderen Komponenten (externes Gateway, FHIR-Server, OPA, Audit) sind
> spital-eigen.

Die Architektur folgt einem **Dual-Gateway-Pattern**,
mit **klar getrennten spital-eigenen und gemeinsam betriebenen
Komponenten**:

**Spital-eigene Komponenten** (jedes Spital betreibt selbst):

- ein **eigenes externes Gateway** für eingehende Partneranfragen —
  Gegenstand dieser DSFA,
- ein **eigenes internes Gateway** für die hauseigene Anwendung
  (**ausserhalb des Geltungsbereichs dieser DSFA**, siehe Ziffer 2.1),
- ein **eigener FHIR-Server** (klinische Daten); Produktwahl je Spital,
- eine **eigene Policy Engine (OPA)** je externem Gateway, die die
  vom Placer-Arzt ausgestellte Freigabe (`Consent`) bei jedem
  eingehenden Cross-Party-Request maschinell prüft,
- ein **eigener Audit-Pfad** in die spital-eigene SIEM-Infrastruktur
  (kein gemeinsamer SIEM-Eintrittspunkt).

**Gemeinsam betriebene Komponenten** (zwei, beide Gegenstand dieser DSFA):

- ein **gemeinsamer OAuth2-/OIDC-Autorisierungsserver** (Keycloak), der
  für alle Spitäler die Authentifizierung von M2M-Clients und die
  Ausgabe der für die Cross-Party-Kommunikation benötigten Tokens
  übernimmt (ein einziger Realm, Spitäler als Organisationen mit
  Organisationszugehörigkeit als Token-Claim);
- ein **öffentliches mCSD-Registry** (`Organization`, `Endpoint`,
  `HealthcareService`) als verbindliches Partnerverzeichnis. Die
  Cross-Party-Referenzen auf Organisationen (`Consent.organization`,
  `Consent.provision.actor.reference`) verweisen auf das Registry,
  *nicht* auf die spital-eigenen FHIR-Server — so wird sichergestellt,
  dass beide Spitäler dieselbe autoritative Sicht auf die
  Organisations-Stammdaten haben.

Neu gegenüber der heutigen Praxis (Fax, E-Mail, papierbasierte Zuweisung,
HIN) sind:

1. **Strukturierte, maschinenlesbare** Übermittlung medizinischer Daten,
2. **Direkter Online-Lesezugriff** des Partnerspitals auf einen
   begrenzten Datenausschnitt des Placers,
3. **Feingranulare Freigabe-Steuerung durch den zuweisenden Arzt** —
   anstelle des "ganzen Anhangs an der Mail" wählt der Placer pro
   Zuweisung patientenspezifisch aus, welche Ressourcen-Klassen das
   Partnerspital für welchen Zeitraum lesen darf. Diese Freigabe ist
   technisch in einer FHIR-`Consent`-Ressource hinterlegt und wird am
   externen Gateway maschinell durchgesetzt.

**Rechtsgrundlage und Verantwortlichkeitsmodell** (siehe Ziffer 3.3):

Die rechtliche Zulässigkeit der Bekanntgabe an das Partnerspital wird
**nicht durch eine separate Patientenunterschrift auf einem
Sharing-Formular** etabliert, sondern — wie heute beim Fax/Mail/HIN-
Versand einer Zuweisung — durch:

- das **bestehende Behandlungsverhältnis** zwischen Placer-Arzt und
  Patient (IDG ZH § 8 lit. a) und
- die **berufliche Verantwortung des zuweisenden Arztes**
  (Arztgeheimnis Art. 321 StGB, ärztliche Standespflichten), kombiniert
  mit der Aufklärung des Patienten über die Zuweisung im Rahmen des
  normalen klinischen Gesprächs.

Die FHIR-`Consent`-Ressource ist in diesem Modell das **technische
Äquivalent zur Auswahl der Mailanhänge** durch den Placer-Arzt: sie
dokumentiert maschinenlesbar, wer was für welchen Zweck wie lange sehen
darf, und ermöglicht die feingranulare Durchsetzung am Gateway. Die
*juristische Bindung* der Bekanntgabe bleibt in der Verantwortung des
Placer-Arztes.

Bestehende Bearbeitungen (Behandlungsdokumentation im eigenen KIS, Anlage
einer Zuweisung) ändern sich datenschutzrechtlich **nicht**; neu ist
ausschliesslich die strukturierte digitale Bekanntgabe an das
Partnerspital — bei unverändertem Verantwortungs- und
Rechtsgrundlagenmodell auf Placer-Seite.

### 2.1 Geltungsbereich der DSFA

Diese DSFA fokussiert auf die **neue, gemeinsam betriebene
Bearbeitungsebene**, also auf das, was sich gegenüber dem Status quo
inhaltlich oder technisch ändert:

**Im Geltungsbereich (in scope)**:

- die **externen Gateways** beider Spitäler (eingehende und
  ausgehende Cross-Party-Schnittstelle) — *jeweils vom Spital selbst
  betrieben*,
- der **gemeinsame Autorisierungsserver** (Keycloak) — gemeinsam
  betrieben — einschliesslich Realm-Konfiguration, Client-
  Registrierung, Token-Ausstellung, JWKS-Schlüsselverwaltung und
  Governance des Betreibers,
- das **gemeinsame mCSD-Registry** (`Organization`, `Endpoint`,
  `HealthcareService`) — gemeinsam betrieben — als verbindliches
  Partnerverzeichnis und autoritative Quelle für die
  Organisations-Referenzen in Cross-Party-Ressourcen,
- die **Konsent-/Freigabe-Validierung** durch die jeweilige OPA-Policy
  am externen Gateway,
- die **Cross-Party-Referenzlogik** zwischen den FHIR-Servern beider
  Spitäler (Referenzen `basedOn` / `sourceReference` / `subject` über
  Server-Grenzen hinweg, Organization-Referenzen ins gemeinsame
  Registry),
- der **Audit-Pfad** vom externen Gateway in die SIEM-Infrastruktur
  *(jeweils spitalseitig — kein gemeinsamer SIEM-Eintritt)*,
- die **technische Freigabe-Ressource (`Consent`)** als spitalübergreifend
  ausgewertetes Autorisierungsartefakt.

**Ausserhalb des Geltungsbereichs (out of scope — durch bestehende
Spital-Policies abgedeckt)**:

- das **interne Gateway** und alle dahinterliegenden
  spitalinternen Pfade (UI → internes Gateway → eigener FHIR-Server,
  internes Gateway → OPA, internes Gateway → Keycloak),
- die **klinische Webapp / KIS-Integration** auf Endgeräten der
  jeweiligen Spitäler (Browser-Sicherheit, CSRF/XSS, Session-Handling,
  Frontend-Supply-Chain, lokale Speicherung),
- die **interne rollenbasierte Zugriffssteuerung** (Keycloak-Rollen
  innerhalb der Spital-Organisation, Berechtigungsreviews für eigene
  Nutzer:innen, MFA-Roll-out),
- die **Plattformadministration je Spital** (HAPI-DB-Zugriff,
  Backup, Patching, PAM für interne Admins),
- die **physische und Netzwerksicherheit** des Rechenzentrums,
- die allgemeine **Endpoint-Sicherheit** klinischer Arbeitsplätze.

Diese Aspekte unterliegen bei jedem Spital den **bestehenden
Informations- und Datensicherheitskonzepten** sowie den
spitalinternen Vorgaben (ISMS, IT-Grundschutz / ISO 27001 / NIST-
analoge Praxis). Die DSFA verweist auf diese Konzepte, ohne sie zu
duplizieren.

**Konsequenz für Verantwortlichkeiten**:

- *Eigene externe Gateways, eigene FHIR-Server, eigene OPA, eigener
  Audit-Pfad*: jeweils in der **Eigenverantwortung des jeweiligen
  Spitals**, im Rahmen einer gemeinsam abgestimmten technischen
  Spezifikation (UMZH-IG) und der hier festgelegten Sicherheits-
  anforderungen.
- *Gemeinsame Komponenten (Keycloak + mCSD-Registry)*: Gegenstand
  einer **Betreibervereinbarung** zwischen den Spitälern und dem
  Betreiber; Verantwortlichkeitsmodell siehe DSFA-Massnahme P11.
- *Spital-internes*: Verantwortung verbleibt vollständig beim
  jeweiligen Spital; Nachweis durch Verweis auf das jeweilige
  Sicherheitskonzept.

---

## 3. Beschreibung der beabsichtigten Bearbeitung von Personendaten

### 3.1 Welche (besonderen) Personendaten sollen bearbeitet werden (Datenkategorien)?

Bearbeitet werden **besondere Personendaten gemäss IDG ZH § 3 Abs. 4
lit. a** (Daten über die Gesundheit). Konkret folgende
FHIR-Ressourcen-Kategorien:

| Kategorie (IDG ZH) | FHIR-Ressource | Inhalt (typisch) | Sensitivität |
|---|---|---|---|
| Identifikationsdaten | `Patient` | Name, Geburtsdatum, AHV-Nr. / Patienten-ID, Geschlecht, Adresse | hoch (besonders schützenswert nur indirekt) |
| Gesundheitsdaten | `ServiceRequest` | Indikation, klinische Fragestellung, Dringlichkeit | **besonders schützenswert** |
| Gesundheitsdaten | `Condition` | Diagnose (ICD-10, SNOMED CT) | **besonders schützenswert** |
| Gesundheitsdaten | `Observation` | Befunde, Vitalparameter, Laborwerte | **besonders schützenswert** |
| Freigabe-/Autorisierungsdaten | `Consent` (technisch) | Placer-seitige Sharing-Autorisierung: Empfänger, Zweck, erlaubte Ressourcenklassen, Gültigkeit. **Keine** Patientenunterschrift. | mittel (Nachweis der konkreten Freigabe-Entscheidung; die zugrunde liegende rechtliche Legitimation liegt im KIS / Behandlungsverhältnis) |
| Auftragsdaten | `Task` | Bearbeitungsstatus, Zuweisungs-Metadaten | mittel |
| Behandelnde Personen | `Practitioner`, `PractitionerRole` | Name, Funktion, Organisation des behandelnden Arztes | mittel (Personendaten des Personals) |
| Organisationsdaten (öffentlich, im gemeinsamen mCSD-Registry) | `Organization`, `Endpoint`, `HealthcareService` | Spital-Stammdaten, FHIR-Endpunkte, angebotene Leistungen — gemeinsam gepflegt im mCSD-Registry, jedes Spital A/R für die eigenen Einträge | **keine** Personendaten — öffentlich |

**Nicht bearbeitet** werden im Rahmen dieses Vorhabens: biometrische
Daten, genetische Daten, Daten über religiöse/weltanschauliche/politische
Ansichten, Daten über strafrechtliche Verfolgung.

### 3.2 Wie sollen Personendaten bearbeitet werden (Bearbeitungsvorgänge einschliesslich Datenbekanntgaben und Empfangende)?

**Bearbeitungsvorgänge** (IDG ZH § 3 Abs. 5):

| Vorgang | Ort | Auslöser |
|---|---|---|
| Erheben | Placer-KIS | Zuweisender Arzt erstellt `ServiceRequest` und `Consent` |
| Speichern | Placer FHIR fähiger store | Persistenz auf Spital-eigenen Servern |
| Bekanntgabe (lesend) | Externes Gateway Placer → externes Gateway Fulfiller | Anforderung durch berechtigten Fulfiller-Client mit gültigem Konsent |
| Speichern (Task) | Fulfiller FHIR fähiger store | Anlage `Task` als Bearbeitungsauftrag |
| Aufbewahren | Beide Spitäler | Gemäss spitalinternen Aufbewahrungsfristen (KVG, GesG ZH) |
| Vernichten / Anonymisieren | Beide Spitäler | Nach Ablauf der Aufbewahrungsfrist |

**Bekanntgabe ans Partnerspital** erfolgt ausschliesslich:

- **lesend** (kein Schreibzugriff durch das Partnerspital auf eigene Daten),
- **patientenspezifisch** (keine Bulk-Suchen),
- **auftragsgebunden** (nur Ressourcen, die über `basedOn` /
  `sourceReference` mit dem freigegebenen `ServiceRequest` verknüpft sind),
- **freigabe-validiert** (OPA-Policy verifiziert für jeden Request: die
  vom Placer-Arzt ausgestellte technische Freigabe (`Consent`) existiert,
  ist `active`, gilt für genau diesen Empfänger, innerhalb der
  Gültigkeitsperiode, Zweck passt zum OAuth-Scope, und die angefragte
  Ressourcenklasse ist in der Freigabe enthalten).

**Empfangende** (IDG ZH § 3 Abs. 7):

- {{Spital B als Empfänger der Placer-Daten}} bzw.
- {{Spital A als Empfänger der Fulfiller-Daten}} (Statusrückmeldungen).
- **Keine** Bekanntgabe an Dritte (insb. keine Cloud-Provider, keine
  Versicherer, keine Behörden im Rahmen dieses Vorhabens).
- **Betreiber der gemeinsamen Komponenten** (Keycloak + mCSD-Registry)
  verarbeitet ausschliesslich technische Daten (Client-IDs,
  Token-Metadaten, JWKS-Schlüssel, Organisations-Stammdaten,
  Endpoint-Adressen) — keine Patientendaten. Sein Status
  (Auftragsbearbeiter / gemeinsam Verantwortlicher) wird in der
  Betreibervereinbarung geregelt (Massnahme P11).
- *Innerhalb jedes Spitals*: die Steuerung, welche Personen im
  jeweiligen Spital die Cross-Party-Funktion nutzen dürfen, erfolgt
  durch die **bestehende interne Zugriffskontrolle** und liegt
  ausserhalb des Geltungsbereichs dieser DSFA (siehe Ziffer 2.1).

### 3.3 Zu welchem Zweck sollen Personendaten bearbeitet werden?

**Primärer Zweck** (gemäss IDG ZH § 8 Bindung an einen erkennbaren Zweck):

- Direkte medizinische Versorgung des Patienten ("Behandlungsauftrag")
  im Rahmen einer Zuweisung von einem Spital an ein anderes.

**Sekundärzwecke** ausgeschlossen:

- Keine Verwendung für Forschung, Statistik, Qualitätsmanagement im
  Sinne einer Zweckänderung; falls solche Verwendung später beabsichtigt
  wird, ist eine separate Rechtsgrundlage (Patienteneinwilligung oder
  Bewilligung nach HFG) und eine erneute DSFA erforderlich.
- Keine Profilerstellung, kein Scoring.
- Keine automatisierten Einzelentscheidungen (Art. 21 revDSG).

**Rechtsgrundlage der Bearbeitung und der Bekanntgabe**:

Die Bekanntgabe der Daten ans Partnerspital stützt sich auf:

- **Behandlungsverhältnis und ärztliche Zuweisung** (IDG ZH § 8 lit. a
  i.V.m. Spitalauftrag / Patientenvertrag) — wie heute bei einer
  Zuweisung per Mail/Fax/HIN. Der zuweisende Arzt ist berechtigt und
  verpflichtet, dem weiterbehandelnden Spital die behandlungsrelevanten
  Informationen zu übermitteln.
- **Berufliche Verantwortung des zuweisenden Arztes** (Arztgeheimnis
  Art. 321 StGB, Standespflichten FMH). Der Arzt entscheidet — im
  Rahmen der Aufklärung im normalen Patientengespräch und seiner
  beruflichen Pflichten —, welche Informationen er an wen weitergibt.
- **Bekanntgaberegelung** des IDG ZH § 17 Abs. 2 lit. a (Bekanntgabe
  ist zur Erfüllung der gesetzlichen Aufgabe erforderlich, hier:
  medizinische Versorgung) bzw. lit. b (Einwilligung der betroffenen
  Person), je nach konkreter Konstellation und kantonaler Auslegung.
- **Information der Patientin / des Patienten** über die Zuweisung als
  Teil der ärztlichen Aufklärung. Dies geschieht im klinischen Gespräch
  und wird im KIS dokumentiert — *nicht* durch eine separate
  Unterschrift auf einem Sharing-Formular.

**Stellung der FHIR-`Consent`-Ressource in diesem Modell**:

Die `Consent`-Ressource ist **nicht** der juristische Träger der
Einwilligung. Sie ist eine **technische Autorisierungsressource**, die
folgende Aufgaben erfüllt:

1. Sie dokumentiert die **konkrete Sharing-Entscheidung** des Placer-
   Arztes (Empfänger-Organisation, Zweck, erlaubte Ressourcenklassen,
   Gültigkeitsperiode).
2. Sie wird am externen Gateway maschinenlesbar zur **technischen
   Zugriffsentscheidung** herangezogen (siehe OPA-Policy, M03).
3. Sie ist das **digitale Pendant zur Auswahl der Anhänge** bei einer
   heutigen Mail-/Fax-Zuweisung.

Die juristische Verantwortung für die Zulässigkeit der Bekanntgabe
verbleibt beim zuweisenden Arzt / beim Placer-Spital — wie heute. Das
empfangende Fulfiller-Spital darf sich auf die vom Placer ausgestellte
Freigabe verlassen, soweit diese formal gültig ist (analog zur
heutigen Praxis: das empfangende Spital prüft nicht die einzelne
Patienteneinwilligung beim Placer, sondern stützt sich darauf, dass
ein qualifizierter Arzt die Zuweisung verfasst hat).

> **Hinweis zur Vorabkontrolle**: Dieses Modell entspricht der heute
> etablierten klinischen Praxis und ist datenschutzrechtlich
> äquivalent zur bestehenden Zuweisung per Mail/Fax/HIN. Der
> technische Fortschritt liegt in der **maschinell durchsetzbaren
> Begrenzung** des Empfänger-Zugriffs, nicht in einer Veränderung
> der Rechtsgrundlage.

### 3.4 In welchem Umfang sollen Personendaten bearbeitet werden (Anzahl Datensätze, betroffene Personen)?

**Mengengerüst** (siehe auch interne Kapazitätsplanung):

| Kennzahl | Wert | Quelle / Annahme |
|---|---|---|
| ServiceRequests pro Jahr (Cross-Party) | ~{{100'000}} | Hochrechnung Zuweisungsvolumen beider Spitäler |
| Tasks pro Jahr | ~{{100'000}} | 1 Task je SR (typisch) |
| Technische Freigaben (`Consent`-Ressourcen) pro Jahr | ~{{100'000}} | 1 Freigabe je SR, vom Placer-Arzt ausgestellt |
| Betroffene Personen pro Jahr (Patienten) | ~{{50'000 - 100'000}} | Mehrere SR pro Patient möglich |
| Geschätzter Datenbestand 5 J. je Spital-FHIR-Server (netto) | ~{{30–50 GB}} | siehe interne Sizing-Schätzung |
| Aufbewahrungsfristen | 10 Jahre (KVG); 20 Jahre für KG (GesG ZH § 33) | gesetzliche Mindestfristen |

**Skalierung der Risikoanalyse**: Die Verarbeitung erfolgt in einem
Umfang, der gemäss Merkblatt DSFA der DSB ZH als "grosser Umfang" gilt
→ siehe Ziffer 4.

---

## 4. Risikoanalyse

### 4.1 Identifizierte Risiken für Grundrechte (Privatsphäre, informationelle Selbstbestimmung)

| # | Risiko | Schutzgut | Quelle |
|---|---|---|---|
| R1 | **Unberechtigter Lesezugriff** durch das Partnerspital auf Patientendaten, die nicht von der Placer-Freigabe gedeckt sind (Scope-Bruch) | Vertraulichkeit | Cross-Party-Lesezugriff am externen Gateway |
| R2 | **Token-Diebstahl / Missbrauch** eines OAuth2-Access-Tokens, dadurch Identitätsübernahme zwischen Spitälern | Vertraulichkeit, Integrität | M2M-Tokens transportieren weitreichende Berechtigungen |
| R3 | **Kompromittierung der gemeinsamen Komponenten** (Keycloak und/oder mCSD-Registry) — Common-Mode-Risiko: ein Vorfall trifft beide Spitäler gleichzeitig (Keycloak: Signing-Key, Realm-Konfiguration, Client-Registrierung; Registry: Manipulation von `Endpoint.address` → Umlenkung des Cross-Party-Traffics) | Vertraulichkeit, Integrität, Verfügbarkeit | Single Point of Trust für das Ökosystem |
| R4 | **Ungewollte Verkettung** zwischen klinischen Daten beider Spitäler über Patienten-IDs / Partner-Discovery | Informationelle Selbstbestimmung | Cross-Party-Referenzen, föderierte Partnerentdeckung |
| R5 | **Veraltete oder zurückgezogene Placer-Freigabe** wird nicht durchgesetzt — Lesezugriff erfolgt, obwohl der Placer-Arzt die Freigabe inzwischen widerrufen, eingeschränkt oder ihre Geltung abgelaufen ist | Rechtmässigkeit | Asynchrone Invalidierung, Caching |
| R6 | **Verfügbarkeits-/Integritätsausfall des Cross-Party-Pfads** (eigenes externes Gateway, gemeinsamer Autorisierungsserver, gemeinsames Registry) — verzögerte Bekanntgabe; klinischer Fallback auf Mail/Fax bleibt aber stets verfügbar | Integrität, Verfügbarkeit | Cross-Party-Komponenten |
| R7 | **Mangelhafte Auditierbarkeit der Bekanntgaben** — Cross-Party-Zugriffe sind im Nachhinein nicht lückenlos rekonstruierbar (Auskunftsrecht / Beweisbarkeit gegenüber Patient und DSB) | Transparenz, Auskunftsrecht | Audit-Pipeline am externen Gateway |
| R8 | **Datenexfiltration / Manipulation durch privilegierte Administratoren der gemeinsamen Komponenten** (Betreiber von Keycloak und/oder mCSD-Registry) — abzugrenzen von spitalinternen Plattform-Admins (eigenes externes Gateway, eigener FHIR-Server, eigener Audit-Pfad), die durch die jeweiligen Spital-Policies abgedeckt sind | Vertraulichkeit, Integrität | Admin-Zugänge gemeinsamer Komponenten |

### 4.2 Besondere Risikofaktoren (Checkliste DSB ZH)

| ☑ / ☐ | Risikofaktor | Begründung |
|---|---|---|
| ☐ | automatisierte Einzelentscheidungen | Keine — alle medizinischen Entscheide trifft eine natürliche Person (Arzt / Ärztin). |
| ☐ | systematische Überwachung | Keine — Bearbeitung ist anlassbezogen je Zuweisung, kein Monitoring. |
| **☑** | **Bearbeitung von besonderen Personendaten** | **Ja** — Gesundheitsdaten gemäss IDG ZH § 3 Abs. 4 lit. a. |
| **☑** | **Personendaten in grossem Umfang** | **Ja** — ~100'000 Zuweisungen/Jahr, ~50–100k Patienten/Jahr. |
| **☑** | **Zusammenführen / Kombinieren von Personendaten aus unterschiedlichen Prozessen** | **Ja** — Cross-Party-Verknüpfung von Placer- und Fulfiller-Daten über `basedOn`/`sourceReference`. |
| **☑** | **Einsatz neuer Technologien** | **Ja** — OAuth2/OIDC, SMART on FHIR, OPA-basierte Konsent-Enforcement, mCSD-Partnerverzeichnis sind im klinischen Umfeld ZH neu. |
| ☐ | Zusammenarbeit von mehr als drei Amtsstellen | Nein — zwei Spitäler. |
| ☐ | Scoring / Profiling | Keines. |
| **☑** | **Online-Zugriffe / Abrufverfahren** | **Ja** — genau dies ist Gegenstand des Vorhabens (IDG ZH § 17 Abs. 2). |
| ☐ | andere Risikofaktoren | — |
| ☐ | keine besonderen Risikofaktoren vorhanden | — |

**Folge**: Mindestens vier besondere Risikofaktoren liegen vor.
→ **Vorabkontrolle durch die DSB Kanton ZH ist erforderlich**
(IDG ZH § 10 i.V.m. § 17; siehe Ziffer 7).

---

## 5. Bewertung von Risiken

### 5.1 Schwere des Eingriffs in die Grundrechte

| Risiko | gering | mittel | schwer | Begründung |
|---|:---:|:---:|:---:|---|
| R1 Unberechtigter Lesezugriff (Scope-Bruch) |   |   | ☑ | Gesundheitsdaten, ggf. Identifizierung, hohes Persönlichkeitsrechtsverletzungspotenzial |
| R2 Token-Diebstahl / Missbrauch |   |   | ☑ | Ermöglicht potentiell R1 in grossem Umfang |
| R3 Kompromittierung gemeinsamer Komponenten (Keycloak / Registry) |   |   | ☑ | Common-Mode: beide Spitäler gleichzeitig betroffen, ggf. Ausstellung gefälschter Tokens oder manipulierte Endpoint-Adressen |
| R4 Ungewollte Verkettung |   | ☑ |   | Pseudonymisierung im Registry, Patienten-IDs nicht spitalübergreifend |
| R5 Veraltete / zurückgezogene Placer-Freigabe |   | ☑ |   | Rechtsgrundlage der Bekanntgabe besteht weiterhin (Behandlungsverhältnis); jedoch über den vom Arzt aktuell gewollten Umfang hinaus → Verletzung der Verhältnismässigkeit, klare Aufsichtsrelevanz |
| R6 Verfügbarkeits-/Integritätsausfall des Cross-Party-Pfads |   | ☑ |   | Klinische Verzögerung möglich, aber Fallback (Fax/HIN) bleibt verfügbar |
| R7 Mangelhafte Auditierbarkeit |   | ☑ |   | Verletzt Auskunftsrecht; mit Mitigation gering |
| R8 Datenexfiltration / Manipulation durch Admins gemeinsamer Komponenten |   |   | ☑ | Privilegierter Zugriff auf Keycloak (Token-Fälschung) oder Registry (Endpoint-Umlenkung) hätte ökosystemweite Reichweite; enthält selbst aber keine Patientendaten |

### 5.2 Wahrscheinlichkeit des Eintretens

> Bewertung unter Berücksichtigung der **bereits getroffenen** Massnahmen
> (vgl. Ziffer 6.1). Ohne diese Massnahmen lägen alle Werte mindestens
> eine Stufe höher.

| Risiko | gering | mittel | schwer | Begründung |
|---|:---:|:---:|:---:|---|
| R1 Unberechtigter Lesezugriff (Scope-Bruch) | ☑ |   |   | OPA-Policy mit deny-by-default, Cross-Party-Negativtests Bestandteil der CI |
| R2 Token-Diebstahl / Missbrauch | ☑ |   |   | Kurzlebige Tokens (≤15 min), `private_key_jwt` (Level 2), keine Refresh-Tokens für M2M |
| R3 Kompromittierung gemeinsamer Komponenten (Keycloak / Registry) | ☑ |   |   | Mit HSM-geschützten Signing-Keys, geregeltem Betrieb, MFA für Admins, signierten Registry-Mutationen und Hash-Chain-Auditing gering, aber nicht null |
| R4 Ungewollte Verkettung | ☑ |   |   | Patienten-IDs lokal, keine spitalübergreifende ID |
| R5 Veraltete / zurückgezogene Placer-Freigabe |   | ☑ |   | Freigabe-Validierung pro Request, aber Cache-TTL ein Restrisiko |
| R6 Verfügbarkeits-/Integritätsausfall des Cross-Party-Pfads |   | ☑ |   | Übliche Bedrohungslage; HA-Setup und Mail/Fax-Fallback mitigieren |
| R7 Mangelhafte Auditierbarkeit | ☑ |   |   | Strukturiertes Audit-Log am Gateway, WORM-Storage geplant |
| R8 Datenexfiltration / Manipulation durch Admins gemeinsamer Komponenten |   | ☑ |   | Vier-Augen-Prinzip + PAM beim Betreiber mitigiert nicht vollständig; Schadensauswirkung beschränkt auf Token-Fälschung und Endpoint-Umlenkung (kein direkter PHI-Zugriff) |

---

## 6. Massnahmen zur Reduktion der Risiken

Die Massnahmen sind mit Querverweis auf die Referenzarchitektur und das
bestehende Sicherheitskonzept ([security-concept.md](security-concept.md))
hinterlegt.

### 6.1 Bereits getroffene Massnahmen

| # | Massnahme | Komponente | Dok. vorhanden | Adressiert Risiko |
|---|---|---|---|:---:|
| M01 | **Externes Gateway pro Spital** für eingehende Partneranfragen — getrennt vom internen Anwendungspfad; erzwingt JWT-Validierung, OPA-Prüfung und Rate-Limit | API-Gateway | ☑ | R1, R8 |
| M02 | **JWT-Validierung am externen Gateway** (RS256, JWKS des gemeinsamen Keycloak), strikte `alg`/`iss`/`aud`-Prüfung, Allowlist | Gateway `jwt-auth`-Plugin | ☑ | R2, R3 |
| M03 | **Freigabe-basierte Feinautorisierung** — OPA prüft je Request: die vom Placer-Arzt ausgestellte technische Freigabe (`Consent`) existiert, ist `active`, gilt für Empfänger, innerhalb Gültigkeit, Ressourcenklasse und Zweck passen | OPA-Policies `services/opa/policies/` | ☑ | R1, R5 |
| M04 | **Strikte Trennung der FHIR-Datenbestände beider Spitäler** — produktiv durch **getrennte FHIR-Server-Instanzen je Spital** (kein gemeinsamer Datenbank- oder Admin-Plane). Cross-Party-Zugriff ausschliesslich über das externe Gateway des Datenhalters. Organization-Referenzen über das gemeinsame mCSD-Registry aufgelöst. | je Spital eigener FHIR-Server | ☑ | R1, R4 |
| M05 | **Gemeinsames mCSD-Registry als autoritatives Partnerverzeichnis** — `Organization`, `Endpoint` und `HealthcareService` werden ausschliesslich im gemeinsamen Registry geführt; spital-eigene FHIR-Server enthalten keine duplizierten Organisations-Stammdaten der anderen Teilnehmer. Cross-Party-Referenzen verweisen über das Registry, nicht auf den Partner-FHIR-Server. | gemeinsames mCSD-Registry | ☑ | R3, R4 |
| M06 | **SMART on FHIR System-Scopes** (`system/Patient.rs`, `system/ServiceRequest.rs` etc.) — feingranular pro Ressource und Operation, deny-by-default | gemeinsamer Keycloak (Realm-Konfig) | ☑ | R1 |
| M07 | **Kurzlebige Access-Tokens** (≤ 15 min) für M2M, **keine Refresh-Tokens** im M2M-Flow | gemeinsamer Keycloak Client-Settings | ☑ | R2 |
| M08 | **`private_key_jwt`-Client-Authentifizierung** (Level 2 gemäss [security-concept §"Stepwise security up-leveling"](security-concept.md)) für alle Cross-Party-Clients; pro Spital eigene Client-Keys | gemeinsamer Keycloak | ☑ | R2 |
| M09 | **Organisationszugehörigkeit als Token-Claim** — der gemeinsame Keycloak gibt mit jedem M2M-Token einen Org-Claim aus (Spital A/B), den OPA gegen die `Consent.provision.actor` prüft | Realm-Mapper + OPA | ☑ | R1, R7 |
| M10 | **TLS überall im Cross-Party-Pfad** (TLS 1.2+, mTLS optional Level 3); zwischen externem Gateway und gemeinsamen Komponenten (Keycloak, Registry) ebenso | Produktiv-Deployment | ☐ (Deploy-Doku ausstehend) | R1, R2, R3 |
| M11 | **Audit-Log am externen Gateway** — jeder eingehende Cross-Party-Request mit Token-Claims (inkl. Org), Patient, Ressource, Placer-Freigabe-ID, OPA-Entscheid | API Gateway Access-Log + strukturierte Header | ☑ | R7 |
| M12 | **Patienten-IDs nicht spitalübergreifend** — kein gemeinsamer Master-Patient-Index, Verknüpfung erfolgt ausschliesslich auftragsgebunden über `ServiceRequest.subject` | FHIR-Datenmodell des IG | ☑ | R4 |
| M13 | **Integrations-Testabdeckung** — automatisierte Hurl-Tests für Cross-Party-Freigabe-Enforcement, Scope-Prüfung, Negativtests | `tests/hurl/05-cross-party-consent.hurl` etc. | ☑ | R1, R5 |
| M14 | **Code Review & Vier-Augen-Prinzip** für Änderungen an OPA-Policies, Keycloak-Realm-Konfiguration und externen Gateway-Routen (Pull-Request-Workflow) | Git / GitHub | ☑ | R3, R8 |
| M15 | **URL-Rewriting am externen Gateway** unterbindet Lecks interner FHIR store URLs in Antworten an den Partner | API Gateway `response-rewrite` | ☑ | R1, R4 |

### 6.2 Geplante Massnahmen

| # | Massnahme | Adressiert | geplant per |
|---|---|:---:|---|
| P01 | **Hardware-/HSM-gestützte Schlüsselverwaltung** für die Signing-Keys des **gemeinsamen Keycloak** sowie für die Cross-Party-Client-Keys (Level 2/3 PKI gemäss security-concept); kontrollierte Rotation mit Überlappung | R2, R3, R8 | {{TT.MM.JJJJ}} |
| P02 | **mTLS am externen Gateway** (Level 3, FAPI 2.0-alignment) für besonders sensitive Datenklassen (`Observation`, `Condition` etc.) | R1, R2 | {{TT.MM.JJJJ}} |
| P03 | **WORM-/Immutable-Audit-Storage** (z. B. append-only Object Lock) für die Gateway-Logs, getrennte Custody | R7, R8 | {{TT.MM.JJJJ}} |
| P04 | **Freigabe-Invalidierungs-Propagation** — aktive Invalidierung von Cache-Entries (OPA + Gateway) bei `Consent.status=inactive`/`entered-in-error` oder Periodenende; max. Verzögerung ≤ 60 s. Auslöser kann sein: Placer-Arzt zieht aktiv zurück, Patient widerruft im Patientengespräch (Placer-Workflow), Periode abgelaufen. | R5 | {{TT.MM.JJJJ}} |
| P05 | **SIEM-Integration** mit Use-Cases: ungewöhnliches Anfrage-Volumen, Token-Replay, Geo-Anomalien, OPA-Deny-Cluster | R1, R2, R8 | {{TT.MM.JJJJ}} |
| P06 | **Privileged-Access-Management** (PAM) für Admins der **gemeinsamen Komponenten** (Keycloak und mCSD-Registry): Just-in-Time, Session-Recording, MFA via WebAuthn, Vier-Augen für Mutationen am Registry. Spital-eigene Plattform-Administration (eigenes externes Gateway, eigener FHIR-Server, eigener Audit-Pfad) unterliegt den jeweiligen Spital-Policies und ist nicht Gegenstand dieser DSFA. | R8 | {{TT.MM.JJJJ}} |
| P07 | **Penetrationstest** durch unabhängigen Dritten (mind. OWASP API Security Top 10, FHIR-spezifische Angriffsmuster); Wiederholung jährlich | R1, R2, R3 | {{TT.MM.JJJJ — vor Go-Live}} |
| P08 | **Notfallkonzept & Wiederanlaufübung** — RTO/RPO definiert, Backup-Restore-Test halbjährlich, Ransomware-Drill | R6 | {{TT.MM.JJJJ}} |
| P09 | **Auskunfts-Workflow** (IDG ZH § 20) — technischer Prozess, Patienten ihre eigenen Cross-Party-Zugriffsspuren offenzulegen | R7 | {{TT.MM.JJJJ}} |
| P10 | **Schulung** des klinischen Personals (Placer-Verantwortung für die Freigabe-Entscheidung; Aufklärungsgespräch mit Patient; korrekter Umgang mit `Consent.provision.class` zur Begrenzung des Sharing-Umfangs; Webapp-Bedienung; Meldepflicht bei Verdacht auf Datenschutzverletzung) | R3 | {{TT.MM.JJJJ — vor Go-Live}} |
| P11 | **Betreibervereinbarung für die gemeinsamen Komponenten** (Keycloak + mCSD-Registry) zwischen den Spitälern und dem Betreiber; klärt Verantwortlichkeitsmodell (Auftragsbearbeitung gem. IDG ZH § 6 Abs. 2 / Art. 9 revFADP, oder gemeinsame Verantwortlichkeit Art. 5 lit. j revFADP), Datenkategorien (Identitäts-/Token-Metadaten, Organisations-/Endpoint-Stammdaten), SLA, Sicherheitsanforderungen, Audit-Rechte, Incident-Meldewege, Sub-Operator-Klausel | übergreifend | {{TT.MM.JJJJ — vor Go-Live}} |
| P11b | **Joint-Governance-Modell** für die gemeinsamen Komponenten: Change-Board für Keycloak (Realm-Konfiguration, Client-Onboarding, Scope-/Rollendefinitionen) **und** Registry (Organisations-/Endpoint-Einträge); mind. ein Vertreter pro angeschlossenem Spital, Pull-Request-basiert | R3, R8 | {{TT.MM.JJJJ}} |
| P11c | **Bilaterale technische Spezifikation** für die eigenbetriebenen Komponenten (externes Gateway, FHIR-Server, OPA-Policy, mCSD-Veröffentlichung) — gemeinsam festgelegte Mindestanforderungen (Token-Validierung, OPA-Policy-Schema, mCSD-Endpunkt-Format, Audit-Felder); Konformitätsnachweis durch jedes Spital vor Go-Live | übergreifend | {{TT.MM.JJJJ — vor Go-Live}} |
| P12 | **Bedrohungs- und Risikoanalyse (BRA)** auf Ebene der konkreten Deployment-Topologie je Spital, mit Bewertung pro Asset und Schnittstelle | übergreifend | {{TT.MM.JJJJ}} |
| P13 | **Datenschutz-Vorfalls-Meldekette** mit der DSB ZH (IDG ZH § 13a) und untereinander (Spital A ↔ Spital B), inkl. Übungslauf | R6, R8 | {{TT.MM.JJJJ}} |
| P14 | **Aufbewahrungs- und Löschkonzept** je FHIR-Ressourcentyp, inkl. dokumentierter HAPI-`$expunge`-Verfahren | übergreifend | {{TT.MM.JJJJ}} |
| P15 | **Freigabe- und Aufklärungskonzept Placer-seitig** — interne Arbeitsanweisung für zuweisende Ärzte: Aufklärung des Patienten über die Zuweisung im normalen klinischen Gespräch, Dokumentation im KIS, Verantwortung für die Auswahl des Freigabe-Umfangs (Ressourcenklassen, Geltungsdauer), Verfahren für Widerruf/Anpassung. Patienteninformation als Beilage (analog zur heutigen Patienten-Information bei einer Zuweisung). | R5 | {{TT.MM.JJJJ — vor Go-Live}} |

---

## 7. Notwendigkeit Vorabkontrolle

Aufgrund der in Ziffer 4.2 identifizierten besonderen Risikofaktoren —
insbesondere

- Bearbeitung von **besonderen Personendaten** (Gesundheitsdaten),
- **grosser Umfang** der Bearbeitung,
- **Online-Zugriff / Abrufverfahren** ans Partnerspital,
- **neue Technologien** (OAuth2/SMART on FHIR/OPA),
- **Zusammenführen** von Daten aus unterschiedlichen Prozessen —

ist die geplante Datenbearbeitung der **Datenschutzbeauftragten des
Kantons Zürich (DSB ZH) zur Vorabkontrolle** einzureichen (IDG ZH § 10
i.V.m. § 17 Abs. 2).

**Einreichung an**: `datenschutz@dsb.zh.ch` bzw. verschlüsseltes
Kontaktformular der DSB ZH.

**Beizulegende Unterlagen** (Empfehlung):

- Diese DSFA,
- [https://build.fhir.org/ig/umzhconnect/umzhconnect-ig/security.html](UMZH Security Concept),
- [reference-architecture.md](reference-architecture.md),
- Architekturdiagramm (Deployment-Topologie produktiv),
- Muster der Patienteneinwilligung,
- Auftragsbearbeitungsverträge (sofern Dritte involviert),
- Bedrohungs- und Risikoanalyse (BRA) gemäss P12.

---

## 8. Weiteres Vorgehen

| ☑ / ☐ | Option |
|:---:|---|
| **☑** | **Projekt wird der DSB zur Vorabkontrolle eingereicht** |
| ☐ | Vorabkontrolle nicht erforderlich |

**Geplante nächste Schritte** (intern):

1. Finalisierung dieser DSFA mit Vorstand / Datenschutzverantwortlichem je Spital — bis {{TT.MM.JJJJ}}
2. Abgestimmte Einreichung beider Spitäler bei der DSB ZH — bis {{TT.MM.JJJJ}}
3. Adressierung allfälliger Auflagen der DSB ZH — fortlaufend
4. Penetrationstest (P07) und Notfallübung (P08) **vor Go-Live**
5. Reguläre Re-Evaluation dieser DSFA mindestens **jährlich** sowie
   anlassbezogen bei wesentlicher Architekturänderung

---

## Anhang A — Mapping FHIR-Ressource ↔ Bearbeitungszweck ↔ Rechtsgrundlage

| FHIR-Ressource | Speicherort | Zweck der Bearbeitung | Rechtsgrundlage |
|---|---|---|---|
| `Patient` | Placer-FHIR-Server + Fulfiller-FHIR-Server (jeweils eigene Instanz) | Identifikation im eigenen Behandlungskontext | IDG ZH § 8 lit. a (Behandlungsverhältnis) |
| `ServiceRequest` | Placer | Zuweisung formulieren | IDG ZH § 8 lit. a |
| `Consent` (**technische Freigabe**) | Placer | Maschinenlesbare Dokumentation der durch den Placer-Arzt ausgestellten Sharing-Autorisierung (Empfänger, Zweck, Ressourcenklassen, Geltungsdauer). **Nicht** Träger der juristischen Einwilligung. | IDG ZH § 8 lit. a (Behandlungsverhältnis); IDG ZH § 17 Abs. 2 lit. a/b (Bekanntgabeerlaubnis), in Verantwortung des Placer-Arztes (Art. 321 StGB) |
| `Task` | Fulfiller | Bearbeitungsstatus dokumentieren | IDG ZH § 8 lit. a |
| `Condition`, `Observation` | Placer | Klinische Kontextinformation für die Zuweisung | IDG ZH § 8 lit. a; Bekanntgabe durch Behandlungsverhältnis + Placer-Freigabe gedeckt |
| `Organization`, `Endpoint` (Stammdaten) | gemeinsames mCSD-Registry | Partnerverzeichnis | öffentliche Information, keine Personendaten |
| Audit-Logs am Gateway | Spital-eigene SIEM-Infrastruktur | Nachweis Rechtmässigkeit, Auskunftsrecht, Incident Response | IDG ZH § 10 (Datensicherheit), § 20 (Auskunftsrecht) |

## Anhang B — Aufbewahrung und Löschung

| Datenart | Aufbewahrungsfrist | Grundlage | Löschverfahren |
|---|---|---|---|
| Krankengeschichte (inkl. SR, technische Freigabe `Consent`, Task) | 20 Jahre | GesG ZH § 33 | HAPI `$expunge` nach Fristablauf, Verfahren P14 |
| Audit-Logs | mind. 2 Jahre, max. 10 Jahre | IDG ZH § 10 i.V.m. Praxis DSB ZH | automatisierte Rotation auf WORM-Storage |
| Zurückgezogene Freigabe (`Consent.status=inactive`/`entered-in-error`) | wie zugehörige KG | Nachweis der ursprünglich erteilten und später widerrufenen Freigabe-Entscheidung | wie KG; Status- und Audit-Trail bleiben erhalten |
| Backups | gemäss Backup-Konzept (≤ Aufbewahrungsfrist Primärdaten) | Datensicherheit | Crypto-Shredding bei Schlüsselrotation |

## Anhang C — Verantwortlichkeiten (RACI, Auszug)

| Aufgabe | Spital A (Placer) | Spital B (Fulfiller) | Betreiber Gemeinsame Komponenten | DSB ZH |
|---|:---:|:---:|:---:|:---:|
| Patientenaufklärung über die Zuweisung (klinisches Gespräch) | **A/R** | I | — | I |
| Rechtliche Beurteilung, ob Bekanntgabe zulässig ist | **A/R** (Arzt) | C | — | I |
| Ausstellung der technischen Freigabe (`Consent`-Ressource) | **A/R** | I | — | I |
| Bestimmung des Freigabe-Umfangs (Ressourcenklassen, Geltungsdauer) | **A/R** (Arzt) | C | — | I |
| Anpassung/Rückzug der Freigabe (auch auf Patientenwunsch) | **A/R** | I | — | I |
| Spital-interne Zugriffskontrolle (wer darf Cross-Party-Funktion nutzen) | **A/R** | **A/R** | — | I |
| Bekanntgabe / Lesezugriff (technisch) | R (Bekanntgeber) | A (Empfänger) | — | I |
| **Eigenes externes Gateway**: Betrieb, Patching, Sicherheit | **A/R** | **A/R** | — | I |
| **Eigener FHIR-Server**: Betrieb und Sicherheit | **A/R** | **A/R** | — | I |
| Freigabe-Durchsetzung am eigenen externen Gateway (OPA) | **A/R** | **A/R** | — | I |
| **Gemeinsamer Keycloak**: Betrieb, Patching, Schlüsselverwaltung | C | C | **A/R** | I |
| **Gemeinsamer Keycloak**: Realm-/Scope-/Client-Konfiguration | C (im Change-Board) | C (im Change-Board) | **A/R** (Vollzug) | I |
| **Gemeinsames mCSD-Registry**: Betrieb, Patching | C | C | **A/R** | I |
| **Gemeinsames mCSD-Registry**: Inhaltspflege (`Organization`/`Endpoint`-Mutationen) | **A/R** für eigenen Eintrag (im Change-Board) | **A/R** für eigenen Eintrag (im Change-Board) | R (Vollzug + Vier-Augen) | I |
| Audit-Log am eigenen externen Gateway: Verwaltung | **A/R** | **A/R** | — | C |
| Patientenauskunft (IDG ZH § 20) — eigene Daten | A/R | A/R | — | C |
| Patientenauskunft über erfolgte Cross-Party-Zugriffe | **A/R** (über eigenes Audit-Log) | C (kann Spital A mit Auditdaten beliefern) | C (Token-Ausstellungs-Logs, Registry-Lookup-Logs) | C |
| Meldung Datenschutzverletzung (IDG ZH § 13a) | A/R | A/R | R (gegenüber Spitälern, falls KC oder Registry betroffen) | A (Empfänger) |

> A = Accountable, R = Responsible, C = Consulted, I = Informed
>
> **Hinweise**:
>
> - Die **Accountability für die Rechtmässigkeit der Bekanntgabe** liegt
>   beim Placer-Spital bzw. konkret beim zuweisenden Arzt. Das Fulfiller-
>   Spital darf sich auf eine formal gültige technische Freigabe verlassen
>   (analog zur heutigen Praxis).
> - **Gemeinsame Komponenten** sind Keycloak und das mCSD-Registry. Der
>   Betreiber verarbeitet keine Patientendaten, sondern Identitäts-,
>   Konfigurations- und Organisations-Stammdaten. Sein Status
>   (Auftragsbearbeiter gem. IDG ZH § 6 Abs. 2 oder gemeinsam
>   Verantwortlicher Art. 5 lit. j revFADP) wird in der
>   Betreibervereinbarung (P11) festgelegt.
> - **Alle anderen Komponenten** (externes Gateway, FHIR-Server, OPA,
>   Audit) betreibt jedes Spital eigenständig. Konformität zur
>   gemeinsamen technischen Spezifikation (P11c) ist Voraussetzung für
>   die Cross-Party-Anbindung.
> - **Spital-interne Aufgaben** (Webapp, internes Gateway, interne RBAC,
>   Endpoint-Sicherheit, allgemeines Plattform-Patching) sind nicht in
>   dieser RACI abgebildet — sie verbleiben vollständig beim jeweiligen
>   Spital und unterliegen dessen bestehenden Sicherheits- und
>   Datenschutzrichtlinien.

---

*Dieses Blueprint ist eine ausfüllbare Vorlage und ersetzt nicht die
abschliessende rechtliche und sicherheitstechnische Beurteilung durch
die Datenschutz-Verantwortlichen der jeweiligen Spitäler und die DSB ZH.*
