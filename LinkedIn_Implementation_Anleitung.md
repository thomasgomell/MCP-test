# Implementierungs-Anleitung LinkedIn / meetAlfred – migRaven.Max

Zielgruppe: Campaign / Sales Ops Manager
Ziel: Technisch & operativ saubere Umsetzung + teilautomatisierte Optimierung.

## 1. Übersicht Workflow

1. Zielsegmente & Persona Clustering (Strategisch / Taktisch)
2. Sales Navigator Export & Rohdaten-Normalisierung
3. Anreicherung (Trigger, Pain, Branche, Quartalsfokus)
4. CSV Preparation für meetAlfred (Mapping Felder)
5. Sequenz-Setup (2 Varianten) + Branching Tags
6. Asset Landingpage & Tracking-Verknüpfung
7. Launch Pilot (Strategisch) → Review → Rollout Taktisch
8. Laufende Optimierung (A/B, KPI-Drift, Copy Iteration)
9. Reporting & Feedback Loop in Produkt-/Marketing-Team

## 2. Datenbeschaffung & Segmentierung

### 2.1 Kriterien Strategisch

- Titel enthält: CIO, "Chief Digital", "Chief Transformation", CFO, CISO, "Risk Officer"
- Unternehmensgröße: 500–5000 (Sales Nav Filter)
- Geografie: DACH primär

### 2.2 Kriterien Taktisch

- Titel enthält: "Enterprise Architect", "Head of Architecture", "PMO", "Program Manager", "IT Service", "Change Manager"

### 2.3 Export Normalisierung

Pflichtfelder in CSV: firstName, lastName, fullName, title, company, companySize, location, profileUrl
Custom Felder hinzufügen: trigger_primary, pain_secondary, industry_tag, focus_q, persona_cluster

## 3. Anreicherungsregeln (Heuristiken)

| Feld            | Quelle                      | Regel                                                                    |
| --------------- | --------------------------- | ------------------------------------------------------------------------ |
| trigger_primary | Manuell + LinkedIn Activity | Häufigstes Transformationsthema im letzten Post (z.B. Cloud, Cost, Risk) |
| pain_secondary  | Erfahrungswerte / Job Title | CIO→"Impact Transparenz", EA→"Dependency Drift"                          |
| industry_tag    | LI Company Industry         | 1:1 übernommen / vereinheitlicht (Manufacturing→MFG)                     |
| focus_q         | Aktuelles Quartal           | Q3_2025 etc.                                                             |
| persona_cluster | Mapping                     | Strategisch oder Taktisch                                                |

## 4. meetAlfred CSV Mapping

CSV Header → meetAlfred Felder:

- LIST_NAME: dynamisch generiert (z.B. Strategic_500-5000_CIO_Q3)
- CUSTOM_VAR_1 = trigger_primary
- CUSTOM_VAR_2 = pain_secondary
- CUSTOM_VAR_3 = industry_tag
- CUSTOM_VAR_4 = focus_q
- CUSTOM_VAR_5 = persona_cluster

Beispiel-Zeile:
Strategic_500-5000_CIO_Q3;Cloud Portfolio;Impact Transparenz;MFG;Q3_2025;Strategisch

## 5. Sequenz Setup (meetAlfred)

Erstelle zwei Sequenzen:

- SEQ_STRATEGIC_MAX_V1
- SEQ_TACTICAL_MAX_V1

Schritt-Konfiguration (Beispiel Strategic):
| Step | Delay | Kanal | Template Code | Branch Tags |
|------|-------|-------|---------------|-------------|
| 1 | 0d | Connect | CONNECT_STRAT_V1 | onAccept → move to Step2 |
| 2 | +2d | Message | WELCOME_STRAT_V1 | Like→tag:LIKED_STEP2 |
| 3 | +3d | Message | INSIGHT_Q_V1 | Reply→jump Step4 / tag:REPLIED_STEP3 |
| 4 | +3d | Message | ASSET_OFFER_V1 | Reply→jump Step6 |
| 5 | +4d | Message | SOCIAL_USECASE_V1 | - |
| 6 | +4d | Message | CALL_SPAR_V1 | Accept Call→tag:CALL_BOOKED |
| 7 | +5d | Message | BREAKUP_V1 | - |

Taktisch gleiche Struktur, aber INSIGHT_Q_V1_TACT etc.

## 6. Template Tokens

In meetAlfred Platzhalter verwenden:
{{firstName}}, {{CUSTOM_VAR_1}}, {{CUSTOM_VAR_2}}, {{CUSTOM_VAR_3}}, {{CUSTOM_VAR_4}}

Beispiel Anpassung Insight:
"In >70% der {{CUSTOM_VAR_1}} Initiativen sehen wir {{CUSTOM_VAR_2}} als Grund für Verzögerungen – triggert das bei euch etwas?"

## 7. Branching / Automations

Regeln (Pseudokonfiguration):

- Wenn Like auf Step2 und keine Antwort 48h: Sende Variante Step3a (INSIGHT_Q_LIKE_FOLLOW)
- Wenn Antwort enthält 'schick' oder 'gern' (Regex: /schick|gern/i): Skip Step3 → Step4
- Wenn Antwort enthält 'kein Bedarf' (Regex: /(kein Bedarf|gerade nicht)/i): Set Tag NO_NEED + Schedule Step7 in 14d
- Wenn Tag CALL_BOOKED: Entferne aus Sequenz (Stop Automation)

## 8. Asset Landingpage

Minimaler Aufbau:

- Headline: "Impact Snapshot: Entscheidungs- & Change-Graph"
- 1 PNG Visual (<300KB) + 3 Kennzahlen (z.B. Time-to-Approval -23%)
- CTA Secondary: "15 Min Sparring sichern" (Calendly Link mit utm_source=linkedin_outreach&utm_campaign=li_graph_outreach)

Tracking Parameter: utm_source=linkedin_outreach, utm_medium=dm, utm_campaign=li_graph_outreach

## 9. Reporting & KPI Dashboard

Wöchentliche Tabelle (Sheets / BI Tool):
| KPI | Strategic | Tactical | Ziel | Abweichung | Trend |
|-----|-----------|----------|------|-----------|-------|
| Connections gesendet | | | | | |
| Accept Rate % | | | 40% | | |
| Replies % | | | 18% | | |
| Asset Requests % | | | 35% | | |
| Calls gebucht | | | | | |
| Call Conv % | | | 25% | | |
| Re-Engage Antworten | | | 5% | | |

## 10. A/B Test Betriebsmodell

1. Jede Woche nur 1 aktive Hypothese pro Sequenz
2. Mindestens 100 gesendete Nachrichten pro Variante bevor Entscheidung
3. Gewinner → Basis für V(n+1), Verlierer archivieren (Changelog pflegen)
4. Archiv-Datei: /outreach_tests/AB_LOG_Q3_2025.md (manuell oder automatisiert per MCP)

## 11. Fehler- & Qualitätskontrollen (Preflight Checklist)

- [ ] Keine Platzhalter-Leichen (Regex Check: /{{[A-Za-z0-9_]+}}/ nach Merge)
- [ ] Kein aggressiver Pitch-Vokabular (Regex Negativliste: /(Demo|verkauf|rabatt)/i)
- [ ] Tageslimit nicht überschritten (Monitoring Script)
- [ ] DSGVO Disclaimer intern reviewed
- [ ] Response Routing getestet (Testprofil)

## 12. Automatisierungsmöglichkeiten

### 12.1 Lokale Skripte (Python) + meetAlfred CSV Feed

- Script 1: salesnav_normalize.py (bereinigt Export, fügt persona_cluster)
- Script 2: enrich_rules.py (regelbasierte Zuordnung Trigger/Pain)
- Script 3: qc_validate.py (Regex Checks, KPI Forecast)
- Script 4: generate_csv_meetalfred.py (final Mapping + LIST_NAME Logik)

### 12.2 MCP Server Integration

Einsetzbare / mögliche MCP Server:
| Aufgabe | MCP Option | Beschreibung |
|---------|------------|--------------|
| Text-Qualität / Tonalität | LLM Server (z.B. OpenAI Proxy MCP) | Automatisches Scoring / Vorschläge |
| Regex QA / Platzhalterprüfung | Custom Python MCP | Endpunkt `validate_outreach` prüft CSV |
| KPI Aggregation | Analytics MCP (Custom) | Liest Export-CSV, berechnet Raten, speichert Node "Outreach*KPI*<Datum>" |
| AB-Test Log Persistenz | Neo4j Memory MCP | Schreibt Testergebnisse als Knoten (AB_Test) |
| Persona Matching | Embedding MCP | Ähnlichkeits-Check Titel → Persona Cluster |

### 12.3 Beispiel: Custom MCP Endpoint Spec (Pseudo)

Endpoint: `/enrich/contact`
Input: { title, last_post, industry }
Output: { trigger_primary, pain_secondary, persona_cluster, confidence }
Logik: Regel + Embedding-Semantik fallback.

### 12.4 Neo4j Nutzung

- Knoten: Outreach_Contact (id, list_name, trigger, pain, industry, persona, status, last_step, tags[])
- Beziehungen: (Outreach_Contact)-[:HAS_EVENT]->(Event {type, ts})
- KPI Query Beispiel:
  MATCH (c:Outreach_Contact)
  WITH count(c) AS total,
  sum(CASE WHEN 'ACCEPTED' IN c.tags THEN 1 ELSE 0 END) AS accepted
  RETURN total, accepted, (accepted\*1.0/total) AS accept_rate;

## 13. Sicherheits- & Compliance Aspekt

- Keine Speicherung sensibler personenbezogener Daten über öffentliches Profil hinaus
- Löschung auf Anfrage: Flag `delete_requested` → tägliches Cleanup-Skript
- Logging minimal (Event-Typ + Zeit), kein Volltext der privaten Nachricht im Klartext (optional Hash)

## 14. Betriebs-RACI

| Aktivität         | R                | A              | C                | I   |
| ----------------- | ---------------- | -------------- | ---------------- | --- |
| Segmentierung     | Campaign Manager | Head Marketing | Sales Lead       | CEO |
| Copy Updates      | Campaign Manager | Head Marketing | LLM Reviewer     | -   |
| KPI Reporting     | Ops Analyst      | Head Marketing | Sales            | CEO |
| A/B Governance    | Ops Analyst      | Head Marketing | Campaign Manager | -   |
| Compliance Review | Legal            | Head Marketing | Campaign Manager | CEO |

## 15. Rollout-Timeline (Beispiel)

| Woche | Meilenstein                               |
| ----- | ----------------------------------------- |
| -3    | Segmentierung + Enrichment Scripte fertig |
| -2    | Sequenz Setup + Pilot CSV import          |
| -1    | Pilot Strategic Launch (50)               |
| Go    | Taktisch Launch + KPI Review              |
| +1    | A/B Iteration 1 (Hook)                    |
| +2    | A/B Iteration 2 (Insight Format)          |
| +3    | Skalierung 500 weitere Kontakte           |

## 16. Eskalationspfade

| Problem              | Aktion                                         |
| -------------------- | ---------------------------------------------- |
| Accept Rate <30%     | Hook A/B sofort starten, Branchen-Split prüfen |
| Reply Rate <10%      | Frage vereinfachen, Pain-Schärfung             |
| Asset Rate <20%      | Asset Label ändern (Canvas vs Snapshot)        |
| Call Conversion <15% | CTA ändern (Workshop vs Sparring)              |
| Spam Warning         | Outreach pausieren 48h, Volumen -30%           |

## 17. Changelog (manuell pflegen)

Datei: /outreach_tests/CHANGELOG.md → Jede Iteration: Datum, Hypothese, Ergebnis, Entscheidung.

---

Letzte Aktualisierung: 10.08.2025
