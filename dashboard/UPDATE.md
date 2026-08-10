# Anleitung für die stündliche Dashboard-Aktualisierung

Diese Anleitung wird von einer automatischen Claude-Routine ausgeführt (werktags stündlich).
Ziel: das Kennzahlen-Dashboard unter der festen Artifact-URL neu publizieren.

**Artifact-URL (immer dieselbe!):** `https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35`

## Datenquellen im Überblick

| Dashboard-Bereich | Quelle |
|---|---|
| Umsatz & Gewinn 2026 (KPIs + Monats-Chart) | Reonic-API |
| Pipeline & Montage | Reonic-API |
| KPI „Aufträge gewonnen" | Reonic-API |
| Übrige KPIs + Wochen-Chart (Angebote, Montagen, Anfragen) | Reonic-Benachrichtigungsmails |
| Nächste Termine | Outlook-Kalender |

## Ablauf

### 1. Zeitbezug

Alle Datums-/Zeitangaben in **Europe/Vienna** (`TZ=Europe/Vienna date`). Wochen laufen Montag–Sonntag (ISO-Kalenderwochen).

### 2. Reonic-API (Umsatz, Gewinn, Pipeline)

- **Basis-URL:** `https://api.reonic.de/rest/v3` · **Auth-Header:** `X-Authorization: <REONIC_API_KEY>`
  (Schlüssel aus der Umgebungsvariable `REONIC_API_KEY`; falls sie fehlt, diese Bereiche mit den zuletzt publizierten Werten unverändert lassen und am Ende kurz darauf hinweisen)
- **Doku:** https://api.reonic.de/rest/v3/docs (OpenAPI: `/rest/v3/openapi`)
- **Rate-Limit beachten:** max. ~3 parallele Anfragen, bei HTTP 429 mit Backoff (2s/4s/8s…) wiederholen. Datumsfilter dürfen max. 365 Tage umspannen.

Abrufe:
1. **Gewonnene Aufträge:** `GET /residentialProjects?dealState=Won&itemsPerPage=200&page=N` (alle Seiten). Relevante Felder: `deal.decidedAt`, `id`.
2. **Je gewonnenem Projekt mit `decidedAt` im laufenden Jahr:** `GET /residentialProjects/{id}/variants` → Primärvariante (`isPrimary`):
   - **Umsatz** = `totalPrice.net`
   - **Rohertrag** = Summe `margin.total` über alle `systems.*.lineItems` (Positionen ohne `margin` überspringen)
   - Nach Monat von `decidedAt` gruppieren → Monats-Chart (Werte in T€, gerundet), Jahres-/Monats-KPIs, Ø pro Auftrag.
3. **Pipeline:** 
   - Offene Angebote: `GET /residentialProjects?stage=offer&dealState=Open&itemsPerPage=1` → `pagination.total`
   - Verkauft, Montage nicht gestartet: Won-Projekte (aktiv, `archivedAt` leer) mit `stage=offer` zählen
   - In Montagephase: Won-Projekte (aktiv) mit `stage=installation` zählen

Hinweis: Die API cached Antworten 1 Stunde – das passt zum Stundenrhythmus. Für die ~136+ Variantenabrufe gilt: geduldig sequenziell/gedrosselt arbeiten (dauert 1–2 Minuten).

### 3. Reonic-Mails (Wochen-Chart + übrige KPIs)

`outlook_email_search` mit `sender: "reonic"`, `afterDateTime: <Montag der ältesten anzuzeigenden KW>` (8 abgeschlossene Wochen + laufende Woche = 9 KWs). **Alle Seiten paginieren** (`offset` = `nextOffset`, solange `moreResults`).

Zählung nach Betreff (`receivedDateTime` bestimmt den Tag):

| Betreff | Kennzahl |
|---|---|
| `Ihr Photovoltaik-Angebot steht bereit` | Angebote versendet |
| `Der PDF-Export deiner Checkliste ist fertig` | Montagen abgeschlossen (siehe Regeln) |
| `Ihre Anfrage bei OH Voltaik` | Anfragen abgelehnt |

Regeln für Checklisten-Mails: Projektname steht in der Summary (z.B. `"Installation - Fila.pdf"`); gleicher Projektname am selben Tag nur 1× zählen; Sammel-Exporte (`"... und N weitere ..."`) nicht zählen; `Anlagendoku…` zählt als abgeschlossene Montage. Ignorieren: `Reonic Login`, Marketing-/HubSpot-Mails, Support-Konversationen.

KPI „Aufträge gewonnen" (letzte 30 Tage + Jahr) kommt aus der API (`deal.decidedAt`), nicht aus Mails.

### 4. Termine aus dem Outlook-Kalender

`outlook_calendar_search` mit `query: "*"`, `afterDateTime: heute`, `beforeDateTime: heute + 14 Tage`, `order: "oldest"`.
Zeiten nach Europe/Vienna umrechnen, Format `Mo 17.08. · 08:30`; `showAs: "tentative"` → `flag: "unter Vorbehalt"`; max. 6 Einträge; `termineHinweis` sinnvoll setzen.

### 5. Seite rendern

1. `dashboard/template.html` aus diesem Repo nehmen.
2. **Nur** den Block zwischen `/* ==== DATA` und `/* ==== ENDE DATA ==== */` ersetzen (gleiche Struktur, neue Werte; `stand` = Zeitstempel Wien, `heute` = Datum ausgeschrieben; Geldbeträge deutsch formatiert, z.B. `1.792.771 €`).
3. Als Datei im Scratchpad speichern.

### 6. Publizieren

Artifact-Tool: `file_path` = gerenderte Datei, `url` = die feste Artifact-URL oben, `favicon` = `☀️` (nie ändern).

### 7. Abschluss

- Bei Erfolg: **still beenden** – keine Nachricht, kein Commit, kein Push.
- Bei Fehlern (API/Postfach nicht erreichbar): einmal warten und erneut versuchen; wenn es weiter fehlschlägt, mit kurzer Fehlermeldung enden. **Niemals** das Artifact mit leeren/unvollständigen Daten überschreiben.

## Ausbaustufe: Team-Auslastung

Die Kachel „Pipeline & Montage" zeigt einen Hinweis, dass die Team-/Elektriker-Auslastung folgt,
sobald die **Einsatzplanung (Kalender) in Reonic gepflegt wird** (Stand 10.08.2026: nur 6 Termine im Jahr).
Sobald dort echte Montagetermine stehen: `GET /appointments` (nächste 14 Tage) je Kalender
(`GET /calendars` → Zuordnung zu Nutzern/Teams), geplante Stunden ÷ verfügbare Stunden (8 h × Arbeitstage)
pro Team/Monteur berechnen und die Kachel erweitern.
