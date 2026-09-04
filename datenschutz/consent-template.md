# Placer-Freigabe — Konzept und technisches Mapping

> Ausführungsdokument zu [dsfa-blueprint.md](dsfa-blueprint.md) §6.2 P15.
>
> Beschreibt das **Freigabe-/Autorisierungskonzept** auf Placer-Seite:
> wie der zuweisende Arzt im Rahmen seines Behandlungsverhältnisses und
> seiner ärztlichen Verantwortung eine **technische Freigabe**
> (FHIR-`Consent`-Ressource) ausstellt, mit der das Partnerspital
> begrenzten Lesezugriff auf die zuweisungsrelevanten Daten erhält.
>
> Dieses Dokument ist **kein patientenseitiges Einwilligungsformular**.
> Die rechtliche Zulässigkeit der Bekanntgabe stützt sich — wie heute
> bei einer Zuweisung per Mail/Fax/HIN — auf das Behandlungsverhältnis
> und die berufliche Verantwortung des zuweisenden Arztes; das
> entsprechende Aufklärungsgespräch mit der Patientin / dem Patienten
> ist Teil der normalen klinischen Praxis und wird im KIS dokumentiert.
> Siehe [dsfa-blueprint.md §3.3](dsfa-blueprint.md) für die juristische
> Einordnung.
>
> Enthält **drei Teile**:
>
> 1. **Placer-seitiges Freigabe-Konzept** — Arbeitsanweisung für den
>    zuweisenden Arzt: was er entscheidet, was im UI bestätigt wird,
>    wie er Patient/innen informiert.
> 2. **Rechtliche Anforderungsmatrix** — Nachweis, dass die
>    Pflichtbestandteile nach IDG ZH / revFADP über das Behandlungs-
>    verhältnis + die technische Freigabe abgedeckt sind.
> 3. **Technisches Mapping** auf die FHIR `Consent`-Ressource —
>    feldgenaue Abbildung der Freigabe-Entscheidung.
>
> **Bezugsdokumente**:
>
> - [dsfa-blueprint.md](dsfa-blueprint.md)
> - [bra-blueprint.md](bra-blueprint.md) §7 (Kontrollen C-20 bis C-22)
> - [authorization-scenarios.md](authorization-scenarios.md)
> - [security-concept.md](security-concept.md) §Consent-centric authorization
> - HL7 FHIR R4 — [Consent](https://www.hl7.org/fhir/R4/consent.html)

---

## Teil 1 — Placer-seitiges Freigabe-Konzept

### 1.1 Grundverständnis

In der heutigen Praxis sendet ein zuweisender Arzt klinische Unterlagen
per Mail, Fax oder HIN an das weiterbehandelnde Spital. Er entscheidet
dabei:

- **was** er beilegt (Diagnosen, Befunde, Bildgebung, …),
- **an wen** er es sendet (konkretes Spital, konkrete Abteilung),
- **wie lange** die Information für den Empfänger relevant ist
  (implizit: bis zum Behandlungsabschluss),
- und er **informiert die Patientin/den Patienten** über die Zuweisung
  im normalen klinischen Gespräch.

Die Rechtsgrundlage dafür ist das Behandlungsverhältnis (IDG ZH § 8) in
Verbindung mit der Bekanntgabeerlaubnis nach IDG ZH § 17 Abs. 2 und
der ärztlichen Berufspflicht (Art. 321 StGB).

In der digitalen Architektur ändert sich daran **nichts**, ausser dass:

- der "Anhang an der Mail" durch eine **technische Freigabe** ersetzt
  wird, die maschinenlesbar im FHIR-Datenmodell hinterlegt ist
  (`Consent`-Ressource),
- der Empfänger **direkten, aber strikt begrenzten Lesezugriff** auf die
  freigegebenen Ressourcen erhält statt einer Kopie der Dokumente,
- die Freigabe technisch **enger, präziser und besser auditierbar**
  ist als der heutige Mail-Versand mit beigelegtem Stapel.

> **Verantwortung**: Die juristische Verantwortung für die Zulässigkeit
> der Bekanntgabe liegt — wie heute — beim Placer-Arzt. Das System
> unterstützt ihn dabei, übermässige Bekanntgaben zu vermeiden, kann
> aber seine ärztliche Beurteilung nicht ersetzen.

### 1.2 Schritte beim Erstellen einer Freigabe

Wenn ein Arzt im Placer-Spital eine Zuweisung an das Partnerspital
erstellt, durchläuft er folgende Schritte:

1. **Klinische Beurteilung**: Ist die Zuweisung indiziert? Welches
   Spital / welche Abteilung? Welche Informationen sind für die
   Übernahme der Behandlung relevant?
2. **Patientengespräch / Aufklärung**: Der Patient wird über die
   Zuweisung informiert — Empfänger, Grund, in groben Zügen über die
   übermittelten Informationen, sowie das Recht, sich gegen die
   Zuweisung auszusprechen. Dies geschieht im **normalen klinischen
   Gespräch** und wird im KIS dokumentiert.
3. **Erstellung des `ServiceRequest`** im Placer-System.
4. **Erstellung der technischen Freigabe (`Consent`)** mit folgenden,
   vom Arzt aktiv gesetzten Feldern:
   - **Empfänger**: konkretes Partnerspital (eine konkrete
     `Organization` aus dem gemeinsamen mCSD-Registry).
   - **Zweck**: Behandlung (`TREAT`). Andere Zwecke (Forschung,
     Statistik) sind in diesem Workflow ausgeschlossen.
   - **Erlaubte Ressourcenklassen** (`provision.class`): nur das, was
     für die Übernahme der Behandlung tatsächlich erforderlich ist.
     Standard-Default ist die **engste plausible Auswahl**, der Arzt
     kann erweitern. Voreinstellung "alle Klassen" ist
     server-seitig blockiert (BRA-Kontrolle C-22).
   - **Optionale Negativliste** (`provision.provision[]` mit
     `type=deny`): explizit ausgeschlossene Kategorien (z. B.
     psychiatrische Befunde, falls für die Zuweisung nicht relevant).
   - **Gültigkeitsperiode**: typischerweise 6 Monate, anpassbar.
5. **Explizite Bestätigung im UI**: Die Placer-Anwendung zeigt vor
   dem Speichern eine Klartext-Zusammenfassung der Freigabe (Empfänger,
   Klassen, Periode, Negativlisten); der Arzt bestätigt aktiv. Die
   konkrete Umsetzung dieser Bestätigung ist Sache des Placer-
   Spitals (UX, Anbindung KIS). Cross-Party-seitig wird ergänzend
   server-seitig validiert (BRA C-22: keine "ALL"-Defaults, breite
   Freigaben erfordern Begründungsfeld).
6. **Übergabe**: Der `Task` wird beim Fulfiller angelegt, der auf den
   `ServiceRequest` und die Freigabe-ID verweist.

### 1.3 Anpassung und Rückzug einer Freigabe

Eine erteilte Freigabe kann jederzeit durch eine der folgenden Stellen
**angepasst oder zurückgezogen** werden:

| Auslöser | Wer kann es initiieren | Technische Wirkung |
|---|---|---|
| Behandlungsabschluss / -abbruch | Placer-Arzt | `Consent.status = inactive` |
| Wunsch des Patienten ("Ich möchte das doch nicht") | Patient (über Placer-Arzt oder Patientenportal) → Placer setzt um | `Consent.status = inactive` |
| Sachlicher Fehler in der ursprünglichen Freigabe | Placer-Arzt | `Consent.status = entered-in-error` + neue korrigierte Freigabe |
| Einschränkung des Umfangs nachträglich | Placer-Arzt | neue Version der `Consent` mit engerer `provision.class` |
| **Erweiterung** des Umfangs nachträglich | Placer-Arzt mit Vier-Augen-Bestätigung (BRA-Kontrolle C-20) | neue Version der `Consent` mit erweiterter `provision.class` |
| Ablauf der Periode | (passiv) | OPA wertet automatisch zu `deny` |

Die Rückwirkung am externen Gateway erfolgt innerhalb von **≤ 60 s**
(DSFA-Massnahme P04). Diese Verzögerung ist klinisch und rechtlich
vertretbar — vergleichbar mit der Verzögerung beim "Rückruf" eines
bereits versendeten Fax, das aber noch nicht gelesen wurde.

### 1.4 Patienteninformation (Aushändigung, nicht Unterschrift)

Den Patientinnen und Patienten wird im Rahmen der Zuweisung eine
**Patienteninformation** ausgehändigt (Papier oder Patientenportal).
Dieses Informationsschreiben:

- erklärt, dass die Zuweisung an das {{Spital B}} digital erfolgt
  anstelle des bisherigen Mail-/Fax-Versands,
- benennt das Empfangsspital,
- benennt die übermittelten Informationskategorien in
  verständlicher Sprache,
- nennt die Geltungsdauer,
- erklärt das Recht, der digitalen Übermittlung zu widersprechen
  (Fallback: bisheriger Mail-/Fax-/HIN-Versand),
- nennt das Auskunftsrecht (IDG ZH § 20) und den Widerrufsweg.

Die Patienteninformation **wird nicht unterschrieben** — sie ist eine
Information, keine Einwilligungserklärung. Der Patient kann
selbstverständlich der Zuweisung als solcher widersprechen, dies
betrifft aber das Behandlungsverhältnis und nicht die technische
Freigabe.

Vorlage des Informationsschreibens siehe Anhang A unten.

### 1.5 Wer darf eine Freigabe ausstellen?

**Klinisch berechtigtes Personal** mit aktivem Behandlungsverhältnis
zur betroffenen Patientin / zum betroffenen Patienten — typischerweise
Ärzte, in delegierter Form auch klinisches Fachpersonal nach Anweisung.

Die **Steuerung, welche Personen diese Berechtigung erhalten und wie
sie sich anmelden**, ist Teil des **Sicherheitskonzepts des
Placer-Spitals** (interne RBAC, MFA, Schulung). Sie liegt ausserhalb
des Geltungsbereichs dieser Cross-Party-Architektur und der zugehörigen
DSFA/BRA.

**Cross-Party-relevante Pflichten** (in scope dieser Architektur,
durchgesetzt am Placer-Backend / FHIR-Server):

- Pflichtfeld `Consent.performer` mit gültiger
  `Practitioner`/`PractitionerRole`-Referenz (BRA-Kontrolle C-21);
  ein Datensatz ohne `performer` wird vom Placer-Backend abgelehnt.
- Sanity-Checks gegen offensichtlich überweite Freigaben (BRA C-22).
- Im Cross-Party-Token, das beim Fulfiller eintrifft, prüft das
  externe Gateway die **Organisationszugehörigkeit** des aufrufenden
  Clients (Token-Claim, ausgestellt vom gemeinsamen Keycloak) gegen
  `Consent.provision.actor`.

---

## Teil 2 — Rechtliche Anforderungsmatrix

Diese Matrix zeigt, **wo** jede Pflichtanforderung nach IDG ZH /
revFADP abgedeckt ist. Manche werden durch das bestehende
Behandlungsverhältnis und das Aufklärungsgespräch erfüllt
("klinischer Pfad"), andere durch die technische Freigabe und die
Patienteninformation ("technischer Pfad").

| Anforderung | Rechtsgrundlage | Wo / wie abgebildet |
|---|---|---|
| Identifikation des Verantwortlichen | IDG ZH § 8 | Patienteninformation (Anhang A); Placer-Spital ist Verantwortlicher |
| Identifikation der betroffenen Person | Sorgfaltspflicht | KIS-Stammdaten; `Consent.patient` |
| Klarer Zweck der Bearbeitung | IDG ZH § 8 lit. b; Art. 6 Abs. 3 revFADP | KIS-Dokumentation der Zuweisung; `Consent.category` = `TREAT` |
| Zweckbindung / Ausschluss Sekundärnutzung | IDG ZH § 8 lit. a | KIS, Vertragslage, Patienteninformation; technisch: `Consent.scope` = `patient-privacy`, `provision.purpose` = `TREAT` |
| Datenkategorien bestimmt | IDG ZH § 8 | `Consent.provision.class` (Whitelist) + ggf. `provision.provision[]` mit `type=deny` (Negativliste) |
| Bestimmter Empfänger benannt | IDG ZH § 17 Abs. 2 | `Consent.provision.actor[]` = konkrete Partner-Organisation |
| Befristung der Bekanntgabe | Verhältnismässigkeit | `Consent.provision.period` |
| Bekanntgabeerlaubnis | IDG ZH § 17 Abs. 2 lit. a / lit. b | Behandlungsverhältnis + ärztliche Beurteilung (Placer-Arzt) |
| Information der betroffenen Person | IDG ZH § 13 / Transparenz | klinisches Aufklärungsgespräch + Patienteninformationsschreiben (Anhang A) |
| Recht auf Widerspruch gegen die digitale Übermittlung | Verhältnismässigkeit | Patienteninformation; Fallback-Weg Mail/Fax/HIN bleibt verfügbar |
| Auskunftsrecht | IDG ZH § 20; Art. 25 revFADP | Patienteninformation; technisch: Audit-Log am Gateway, Auskunftsprozess (DSFA P09) |
| Berichtigungs-/Beschwerderecht | IDG ZH § 21; Art. 32 revFADP | Patienteninformation; Standardprozess je Spital |
| Nachvollziehbarkeit der konkreten Bekanntgabe-Entscheidung | IDG ZH § 10 (Datensicherheit) | `Consent`-Ressource selbst (versioniert, signiert, `Provenance`) — sie ist der **Beweis** der Freigabe-Entscheidung des Arztes |
| Schutz besonderer Personendaten | Art. 6 Abs. 7 revFADP | strikt begrenzter Empfänger; Gateway-Enforcement; technische Massnahmen DSFA M01–M15 |
| Aufbewahrung / Löschung | GesG ZH § 33 (20 J. KG) | DSFA Anhang B (Aufbewahrungskonzept P14) |
| Urteilsfähigkeit / Vertretung | ZGB Art. 16, KESG | bestehende KIS-Prozesse Placer-Spital |
| Sprachverständlichkeit | gute Praxis | Patienteninformation in {{DE, FR, IT, EN}} |

**Wichtig** im Vergleich zur Erstausgabe dieses Dokuments:

- Es entfällt die Anforderung, eine *separate Patientenunterschrift* auf
  einem Sharing-Formular einzuholen. Die Rechtsgrundlage ergibt sich
  aus dem Behandlungsverhältnis (wie heute beim Mail/Fax-Versand).
- Der Patient wird **informiert**, nicht zur Unterschrift gebeten.
- Die `Consent`-Ressource trägt **die Entscheidung des Placer-Arztes**,
  nicht die "Einwilligung des Patienten" in juristischem Sinn. Sie ist
  trotzdem ein wichtiges Dokument: sie ist der maschinenlesbare
  Nachweis, dass der Arzt die Bekanntgabe in genau diesem Umfang
  freigegeben hat.

---

## Teil 3 — Technisches Mapping auf FHIR `Consent`

Der Inhalt der Freigabe-Entscheidung wird in der FHIR-`Consent`-
Ressource hinterlegt. Diese Ressource wird auf dem **FHIR-Server des
Placer-Spitals** persistiert (jedes Spital betreibt seinen eigenen
FHIR-Server; Produktwahl ist Sache des jeweiligen Spitals) und ist
die einzige **autoritative Quelle für die Zugriffsentscheidung** am
externen Gateway (OPA-Prüfung).

### 3.1 Feld-Mapping

| Inhaltliches Feld | FHIR-Pfad | Beispielwert |
|---|---|---|
| Patient, dessen Daten freigegeben werden | `Consent.patient` | `{ reference: "Patient/PetraMeier", display: "Petra Meier" }` |
| Status der Freigabe | `Consent.status` | `active` &#124; `inactive` &#124; `entered-in-error` |
| Zeitpunkt der Freigabe-Erstellung | `Consent.dateTime` | `2026-05-21T14:30:00+02:00` |
| Kategorie / Zweck | `Consent.category[0].coding` | `{ system: "http://terminology.hl7.org/CodeSystem/v3-ActCode", code: "TREAT", display: "treatment" }` |
| Geltungsbereich | `Consent.scope.coding` | `{ system: "http://terminology.hl7.org/CodeSystem/consentscope", code: "patient-privacy" }` |
| Ausstellendes Spital (Placer) | `Consent.organization[0]` | `{ reference: "{{registry}}/Organization/HospitalP" }` (gemeinsames mCSD-Registry) |
| Ausstellender Arzt | `Consent.performer[0]` | `{ reference: "Practitioner/HansMuster" }` — **Pflichtfeld**, BRA-Kontrolle C-21 |
| Bezug zur konkreten Zuweisung | `Consent.sourceReference` | `{ reference: "ServiceRequest/REF-2026-04711" }` |
| Empfangendes Spital | `Consent.provision.actor[0]` | `{ role: { coding: [{ code: "IRCP" }] }, reference: { reference: "{{registry}}/Organization/HospitalF" } }` (gemeinsames mCSD-Registry) |
| Permit/Deny | `Consent.provision.type` | `permit` |
| Gültigkeitsperiode | `Consent.provision.period` | `{ start: "2026-05-21", end: "2026-11-21" }` |
| Erlaubte Ressourcenklassen (Whitelist) | `Consent.provision.class` | `[ ServiceRequest, Condition, Observation, MedicationStatement ]` |
| Ausschluss-Liste (z. B. psychiatrische Befunde) | `Consent.provision.provision[]` mit `type=deny` | siehe JSON-Beispiel |
| Versionierung / Nachvollziehbarkeit | `Consent.meta.versionId`, `Provenance` | jede Mutation versioniert, `Provenance.agent` benennt den verändernden Account |

Anmerkungen zum Vergleich mit der "patient consent"-Lesart:

- Es gibt **kein** `Consent.verification[]`-Feld mit einer
  Patienten-Signatur. Stattdessen identifiziert `Consent.performer` den
  Placer-Arzt, der die Freigabe ausgestellt hat.
- Es gibt **kein** `Consent.sourceAttachment` mit einem unterschriebenen
  PDF. Stattdessen ist die Patienteninformation generisch (kein
  Beweisstück eines Einzelfalls); die Aufklärungs-Dokumentation des
  Arztes erfolgt im KIS.

### 3.2 Vollständiges Beispiel (JSON)

```json
{
  "resourceType": "Consent",
  "id": "ConsentPetraMeier-Ortho-2026",
  "meta": {
    "security": [
      { "system": "http://terminology.hl7.org/CodeSystem/v3-Confidentiality", "code": "R", "display": "restricted" }
    ]
  },
  "status": "active",
  "scope": {
    "coding": [
      { "system": "http://terminology.hl7.org/CodeSystem/consentscope", "code": "patient-privacy" }
    ]
  },
  "category": [
    { "coding": [
      { "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode", "code": "TREAT", "display": "treatment" }
    ] }
  ],
  "patient": { "reference": "Patient/PetraMeier", "display": "Petra Meier" },
  "dateTime": "2026-05-21T14:30:00+02:00",
  "performer": [
    { "reference": "Practitioner/HansMuster", "display": "Dr. Hans Muster (zuweisender Arzt)" }
  ],
  "organization": [
    { "reference": "{{REGISTRY_URL}}/fhir/Organization/HospitalP" }
  ],
  "sourceReference": { "reference": "ServiceRequest/REF-2026-04711" },
  "provision": {
    "type": "permit",
    "period": { "start": "2026-05-21", "end": "2026-11-21" },
    "actor": [
      {
        "role": {
          "coding": [
            { "system": "http://terminology.hl7.org/CodeSystem/v3-ParticipationType", "code": "IRCP", "display": "information recipient" }
          ]
        },
        "reference": { "reference": "{{REGISTRY_URL}}/fhir/Organization/HospitalF" }
      }
    ],
    "purpose": [
      { "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason", "code": "TREAT" }
    ],
    "class": [
      { "system": "http://hl7.org/fhir/resource-types", "code": "ServiceRequest" },
      { "system": "http://hl7.org/fhir/resource-types", "code": "Condition" },
      { "system": "http://hl7.org/fhir/resource-types", "code": "Observation" },
      { "system": "http://hl7.org/fhir/resource-types", "code": "MedicationStatement" }
    ],
    "provision": [
      {
        "type": "deny",
        "class": [
          { "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode", "code": "PSY", "display": "psychiatry" }
        ]
      }
    ]
  }
}
```

### 3.3 Validierung am externen Gateway (OPA-Logik, Auszug)

Die OPA-Policy prüft beim eingehenden Cross-Party-Request **alle**
Bindungen zwischen Token, Freigabe und angefragter Ressource (BRA-
Kontrolle C-04). Diese Prüfung schützt sowohl gegen technische Angriffe
(Freigabe-Schmuggel B07) als auch gegen Folgen einer fehlerhaften
Placer-seitigen Freigabe-Ausstellung — soweit sie technisch erkennbar
sind.

```rego
allow if {
    consent := input.consent          # Gateway lädt anhand X-Consent-Id
    consent.status == "active"
    consent.provision.type == "permit"

    # Geltungsperiode
    today := time.now_ns() / 1e9
    consent.provision.period.start_ts <= today
    today <= consent.provision.period.end_ts

    # Empfänger-Org muss der aufrufenden Org entsprechen.
    # input.token.client_org_ref kommt aus einem Token-Claim, den der
    # GEMEINSAME Keycloak mit jedem M2M-Token mitliefert
    # (Organisationszugehörigkeit des Clients).
    some a
    consent.provision.actor[a].reference.reference == input.token.client_org_ref

    # Patient in der Freigabe = Subject der angefragten Ressource
    consent.patient.reference == input.request.target_subject_ref

    # Angefragte Ressourcen-Klasse muss in Whitelist sein
    input.request.resource_type in class_codes(consent.provision.class)

    # Negativlisten-Check
    not denied_class(input.request, consent)

    # Zweck im Token muss zum Freigabe-Zweck passen
    consent.provision.purpose[_].code == input.token.purpose
}
```

Vollständige Policy in `services/opa/policies/`.

### 3.4 Lebenszyklus der `Consent`-Ressource

| Ereignis | FHIR-Operation | Effekt |
|---|---|---|
| Placer-Arzt stellt Freigabe aus | `POST Consent` (FHIR-Server des Placer-Spitals) | Status `active`, `Provenance` mit `agent`=`Practitioner/...`, Audit-Eintrag |
| Behandlung abgeschlossen, Placer setzt inaktiv | `PATCH Consent` → `status=inactive` | Cache-Invalidierung am Gateway ≤ 60 s (DSFA P04), neue Anfragen abgewiesen |
| Patient widerspricht im klinischen Gespräch | Placer-Arzt setzt umgehend `status=inactive` | wie oben |
| Periode abgelaufen | (passiv) | OPA wertet automatisch zu `deny` |
| Korrektur (Eingabe-Fehler des Arztes) | `PATCH Consent` → `status=entered-in-error` + neue Freigabe-Version | alte als ungültig markiert, Audit-Trail bleibt |
| Einschränkung des Umfangs | neue Version der `Consent` mit engerer `provision.class` | sofort wirksam nach Cache-Invalidierung |
| **Erweiterung** des Umfangs nachträglich | neue Version mit erweiterter `provision.class`, **Vier-Augen-Pflicht** (BRA C-20) | bei besonders sensitiven Klassen + Step-up-Auth |
| Aufbewahrungsfrist (KG) | nach 20 Jahren (GesG ZH) | `$expunge` der `Consent` zusammen mit den zugehörigen Ressourcen |

---

## Anhang A — Muster Patienteninformation (zur Aushändigung, nicht Unterschrift)

> Aushändigbares Informationsschreiben (Papier oder elektronisch im
> Patientenportal), das die Patientin / den Patienten über die digitale
> Übermittlung im Rahmen einer Zuweisung informiert. Es wird nicht
> unterschrieben.

---

### Information zur digitalen Übermittlung Ihrer medizinischen Unterlagen im Rahmen einer Zuweisung

**Spital**: {{Spital A — z. B. Universitäres Medizinisches Zentrum Zürich}}
**Stand**: {{TT.MM.JJJJ}}

Sehr geehrte Patientin, sehr geehrter Patient

Ihre behandelnde Ärztin / Ihr behandelnder Arzt überweist Sie an ein
anderes Spital, das **{{Spital B}}**, zur weiteren Behandlung. Damit
das {{Spital B}} Sie gut versorgen kann, müssen wir ihm die für diese
Behandlung relevanten Informationen über Ihre Gesundheit übermitteln.

#### Wie die Informationen übermittelt werden

Bisher wurden solche Informationen per Mail, Fax oder Brief geschickt.
Neu erfolgt die Übermittlung **direkt digital** über eine gesicherte
Verbindung zwischen den Spitälern. Das {{Spital B}} erhält dabei
**Lesezugriff auf genau diejenigen Informationen, die Ihr behandelnder
Arzt für diese Zuweisung als nötig erachtet** — nicht mehr und nicht
weniger.

Konkret werden für diese Zuweisung folgende Kategorien Ihrer Daten
zugänglich gemacht:

- Identifikationsdaten (Name, Geburtsdatum, …)
- die Zuweisung selbst (klinische Fragestellung, Dringlichkeit)
- behandlungsrelevante Diagnosen
- behandlungsrelevante Befunde
- behandlungsrelevante Medikation

Nicht übermittelt werden Informationen, die mit dieser Zuweisung
nichts zu tun haben.

#### Wie lange

Der Lesezugriff besteht für **{{6 Monate}}** ab heute und endet
spätestens am **{{TT.MM.JJJJ}}**. Danach kann das {{Spital B}}
keine neuen Informationen mehr abrufen.

#### Wer dafür verantwortlich ist

Die Entscheidung, welche Informationen für Ihre Behandlung im
{{Spital B}} weitergegeben werden, trifft Ihre behandelnde Ärztin /
Ihr behandelnder Arzt im {{Spital A}} — wie bei einer Zuweisung per
Brief oder Fax. Sie unterliegt der ärztlichen Schweigepflicht.

#### Wenn Sie damit nicht einverstanden sind

Sie können der **digitalen Übermittlung** jederzeit widersprechen.
Wir übermitteln die Informationen dann auf dem **bisherigen Weg**
(z. B. verschlüsselte Mail, Fax, Brief). Ihre Versorgung ist dadurch
nicht beeinträchtigt.

Sie können der **Zuweisung als solcher** ebenfalls widersprechen.
Bitte sprechen Sie in beiden Fällen mit Ihrer behandelnden Ärztin /
Ihrem behandelnden Arzt.

#### Ihre Rechte

Sie haben jederzeit das Recht:

- **Auskunft** zu erhalten, welche Ihrer Daten wann an das {{Spital B}}
  übermittelt wurden (IDG ZH § 20).
- **Berichtigung** unrichtiger Daten zu verlangen.
- **Beschwerde** bei der Datenschutzbeauftragten des Kantons Zürich
  einzulegen (`datenschutz@dsb.zh.ch`).

**Kontakt für Fragen und Widerruf**:

- E-Mail: {{datenschutz@spital-a.ch}}
- Telefon: {{+41 ...}}
- Persönlich: {{Adresse / Stelle}}
- Patientenportal: {{URL}}

---

*Dieses Schreiben ist eine Information; eine Unterschrift ist nicht
erforderlich. Ein Exemplar dieses Schreibens wird Ihnen ausgehändigt
oder ist im Patientenportal abrufbar.*

---

## Anhang B — Edge Cases und offene Punkte

| Fall | Empfehlung |
|---|---|
| **Notfallzugang** (z. B. bewusstloser Patient, kein Aufklärungsgespräch möglich) | Separater Notfall-Workflow ("break the glass") mit nachgelagerter Information; **nicht** in dieser Vorlage abgedeckt — eigene DSFA-Iteration. |
| **Minderjährige zwischen 14 und 18** | Wenn urteilsfähig: Aufklärungsgespräch mit dem Minderjährigen. Bei Hochsensitivität: Einbezug der gesetzlichen Vertretung. |
| **Mehrfach-Zuweisungen** (mehrere Empfänger, z. B. Tumorboard) | Pro Empfänger eine separate `Consent`-Ressource. Sammel-Freigaben sind technisch nicht vorgesehen und klinisch zu vermeiden. |
| **Digitale Zuweisung über Patientenportal** | Patient nutzt das Portal nicht als Sharing-Tool, sondern als Informationsempfänger und ggf. Widerrufskanal. Die Freigabe-Entscheidung trifft weiterhin der Arzt. |
| **Übermittlung zusätzlicher Anhänge (DocumentReference / Binary)** | Wenn weitere Klassen aufgenommen werden, `Consent.provision.class` entsprechend erweitern; Vier-Augen-Pflicht (C-20). |
| **Korrektur eines aktiven Freigabe-Eintrags** | Nicht überschreiben — alten Eintrag `entered-in-error`, neuen Eintrag ausstellen; `Provenance` verlinkt. |
| **Patient fordert Widerruf gegenüber dem Fulfiller-Spital** | Fulfiller leitet den Widerrufswunsch ans Placer-Spital weiter (Placer ist Verantwortlicher der Freigabe). Verfahren in {{interner SLA}} festzulegen. |
| **Widerruf nach Versterben des Patienten** | Behandlungsverhältnis endet; aktive Freigaben bleiben für die laufende Nachbearbeitung gemäss spitalinternem Prozess gültig, dann `inactive`. Aufbewahrung der KG bleibt davon unberührt. |
| **Spätere Sekundärnutzung (Forschung, Statistik)** | Aus diesem Workflow heraus **nicht** möglich. Erfordert separate Rechtsgrundlage (Forschungseinwilligung HFG / BWG, neue DSFA). |

---

*Dieses Konzept ist mit der DSB ZH abzustimmen, idealerweise als
Beilage zur Vorabkontrolle gemäss [dsfa-blueprint.md §7](dsfa-blueprint.md).
Die Patienteninformation (Anhang A) ist vor produktivem Einsatz durch
eine unabhängige Lesbarkeitsprüfung sowie eine juristische
Schlussabnahme zu prüfen.*
