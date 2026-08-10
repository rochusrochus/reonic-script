# Anleitung für die stündliche Dashboard-Aktualisierung

Diese Anleitung wird von einer automatischen Claude-Routine ausgeführt (werktags stündlich).
Ziel: das Kennzahlen-Dashboard unter der festen Artifact-URL neu publizieren.

**Artifact-URL (immer dieselbe!):** `https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35`

## Ablauf

### 1. Zeitbezug

Alle Datums-/Zeitangaben in **Europe/Vienna** (`TZ=Europe/Vienna date`). Wochen laufen Montag–Sonntag (Kalenderwochen nach ISO).

### 2. Daten aus dem Postfach ziehen (Microsoft 365 MCP)

`outlook_email_search` mit `sender: "reonic"`, `afterDateTime: <Montag der ältesten anzuzeigenden KW>` (8 abgeschlossene Wochen + laufende Woche = 9 KWs). **Alle Seiten paginieren** (`offset` = `nextOffset`, bis `moreResults` fehlt).

Zählung nach Betreff (`receivedDateTime` bestimmt den Tag):

| Betreff | Kennzahl |
|---|---|
| `Ihr Photovoltaik-Angebot steht bereit` | Angebote versendet |
| `Angebot unterzeichnet!` | Aufträge unterzeichnet |
| `Der PDF-Export deiner Checkliste ist fertig` | Montagen abgeschlossen (siehe Regeln) |
| `Ihre Anfrage bei OH Voltaik` | Anfragen abgelehnt |

Regeln für Checklisten-Mails:
- Der Projektname steht in der Summary (z.B. `"Installation - Fila.pdf"`). Mehrere Mails mit **demselben Projektnamen am selben Tag nur 1×** zählen (Doppel-Exporte).
- Sammel-Exporte (`"... und N weitere ..."`) **nicht** zählen.
- `Anlagendoku…`-PDFs zählen ebenfalls als abgeschlossene Montage.
- Ignorieren: `Reonic Login`, Marketing-/Newsletter-Mails (Absender `hs-send.com`/HubSpot), Support-Konversationen.

Daraus berechnen:
- **KPI-Kacheln** (letzte 30 Tage) mit passendem Vergleichswert im `hint` (z.B. „Juli gesamt: 28", „seit 15.06.: 5").
- **Wochenwerte** `[Angebote, Montagen]` je KW, laufende KW mit `laufend: true`.

### 3. Termine aus dem Kalender

`outlook_calendar_search` mit `query: "*"`, `afterDateTime: heute`, `beforeDateTime: heute + 14 Tage`, `order: "oldest"`.
- Zeiten von UTC nach Europe/Vienna umrechnen.
- Format: `Mo 17.08. · 08:30`.
- `showAs: "tentative"` → `flag: "unter Vorbehalt"`; abgesagte Termine weglassen.
- Max. 6 Einträge; `termineHinweis` sinnvoll setzen (z.B. wenn die laufende Woche leer ist).

### 4. Seite rendern

1. `dashboard/template.html` aus diesem Repo nehmen.
2. **Nur** den Block zwischen `/* ==== DATA` und `/* ==== ENDE DATA ==== */` ersetzen (gleiche Struktur, neue Werte; `stand` = aktueller Zeitstempel Wien, `heute` = aktuelles Datum ausgeschrieben).
3. Als Datei im Scratchpad speichern (z.B. `oh-voltaik-dashboard.html`).

### 5. Publizieren

Artifact-Tool aufrufen mit:
- `file_path`: die gerenderte Datei
- `url`: `https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35`
- `favicon`: `☀️` (nie ändern)

### 6. Abschluss

- Bei Erfolg: **still beenden** – keine Nachricht, kein Commit, kein Push.
- Bei Fehlern (z.B. Microsoft 365 nicht erreichbar): einmal kurz warten und erneut versuchen; wenn es weiter fehlschlägt, mit kurzer Fehlermeldung enden (nicht das Artifact mit leeren Daten überschreiben!).

## Phase B – sobald der Reonic-API-Schlüssel vorhanden ist

Wenn die Umgebungsvariable `REONIC_API_KEY` gesetzt ist (siehe `dashboard/reonic-api.md`):
zusätzlich Umsatz/Gewinn und Team-Auslastung aus der Reonic-API (V3, Endpunkte unter `/rest/v3/`) abrufen
und die beiden Platzhalter-Kacheln im Template durch echte Werte ersetzen.
Die genauen Endpunkte werden bei der Einrichtung von Phase B hier dokumentiert.
