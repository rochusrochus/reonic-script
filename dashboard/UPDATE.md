# Anleitung für die stündliche Dashboard-Aktualisierung

Diese Anleitung wird von einer automatischen Claude-Routine ausgeführt (werktags stündlich).
Ziel: das Kennzahlen-Dashboard unter der festen Artifact-URL neu publizieren.

**Artifact-URL (immer dieselbe!):** `https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35`

## Datenquellen im Überblick

| Dashboard-Bereich | Quelle |
|---|---|
| Umsatz & Gewinn 2026 (KPIs + Monats-Chart) | Reonic-API |
| Ziele & Erfolge: „Aufträge signiert", Umsatz-Ziele | Reonic-API |
| Ziele & Erfolge: „Angebote versendet" | Reonic-Mails |
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
- **Antwortformat:** jede Antwort ist ein Umschlag `{"data": [...], "pagination": {...}}` – die Liste steht unter `data`, nicht unter `items`.
- **Rate-Limit beachten:** max. ~3 parallele Anfragen, bei HTTP 429 mit Backoff (2s/4s/8s…) wiederholen. Datumsfilter dürfen max. 365 Tage umspannen.

Abrufe:
1. **Gewonnene Aufträge:** `GET /residentialProjects?dealState=Won&itemsPerPage=200&page=N` (alle Seiten). Relevante Felder: `deal.decidedAt`, `id`.
2. **Je gewonnenem Projekt mit `decidedAt` im laufenden Jahr:** `GET /residentialProjects/{id}/variants` → Primärvariante (`isPrimary`):
   - **Umsatz** = `totalPrice.net`
   - **Rohertrag** = Summe `margin.total` über alle `systems.*.lineItems` (Positionen ohne `margin` überspringen)
   - Nach Monat von `decidedAt` gruppieren → Monats-Chart (Werte in T€, gerundet), Jahres-/Monats-KPIs, Ø pro Auftrag.
3. **Kachel „🎯 Ziele & Erfolge" (`DATA.ziele`):** vier Fortschrittsbalken mit goldener Soll-Linie.

   **Zielwerte (hier ändern, wenn der Chef neue Ziele vorgibt):**
   | Ziel | Wert |
   |---|---|
   | Angebote versendet pro Woche | **8** (aus Mails, laufende KW) |
   | Aufträge signiert pro Woche | **5** (API, `decidedAt` in laufender KW) |
   | Umsatz pro Monat | **250.000 €** netto (API, `decidedAt` im laufenden Monat) |
   | Umsatz Jahresziel | **3.000.000 €** netto (API, laufendes Jahr) |

   Berechnung je Balken:
   - `ist`/`ziel` numerisch, `anzeige`/`zielText` als Anzeige-Strings (deutsch formatiert, Mio/T€ kürzen).
   - **`soll` (goldene Linie)** = zeitanteiliger Sollwert: Woche = vergangene Arbeitstage inkl. heute ÷ 5; Monat = Tag ÷ Tage im Monat; Jahr = Tag im Jahr ÷ 365.
   - **`status`**: motivierender Kurztext mit Emoji. Ziel erreicht → `✅ Ziel erreicht – stark!`; vor der Linie → `🚀 Vor Plan!` (+Abstand); hinter der Linie → Differenz zur Linie freundlich benennen (`💪 Noch X bis zur goldenen Linie`). Nie vorwurfsvoll.
   - **`erfolge`**: genau 3 Chips aus echten Daten, z.B. Rekordmonat (höchster Monatsumsatz), Anlagen-Zähler des Jahres, stärkste Angebots-Woche. Bei neuen Rekorden aktualisieren – Rekorde nie stillschweigend verschlechtern.

Hinweis: Die API cached Antworten 1 Stunde – das passt zum Stundenrhythmus. Für die ~136+ Variantenabrufe gilt: geduldig sequenziell/gedrosselt arbeiten (dauert 1–2 Minuten).

### 3. Reonic-Mails (Wochen-Chart + übrige KPIs)

`outlook_email_search` mit `sender: "reonic"`, `afterDateTime: <Montag der ältesten anzuzeigenden KW>` (8 abgeschlossene Wochen + laufende Woche = 9 KWs). **Alle Seiten paginieren** (`offset` = `nextOffset`, solange `moreResults`).

Zählung nach Betreff (`receivedDateTime` bestimmt den Tag):

| Betreff | Kennzahl |
|---|---|
| `Ihr Photovoltaik-Angebot steht bereit` | Angebote versendet |
| `Der PDF-Export deiner Checkliste ist fertig` | Montagen abgeschlossen (siehe Regeln) |
| `Ihre Anfrage bei OH Voltaik` | Anfragen abgelehnt |

Mails mit dem Betreff `Angebot unterzeichnet!` **nicht** als Kennzahl zählen – „Aufträge gewonnen"
und „Aufträge signiert" kommen ausschließlich aus der API (`deal.decidedAt`), sonst wird doppelt gezählt.

Regeln für Checklisten-Mails: Projektname steht in der Summary (z.B. `"Installation - Fila.pdf"`); gleicher Projektname am selben Tag nur 1× zählen; Sammel-Exporte (`"... und N weitere ..."`) nicht zählen; `Anlagendoku…` zählt als abgeschlossene Montage. Ignorieren: `Reonic Login`, Marketing-/HubSpot-Mails, Support-Konversationen.

KPI „Aufträge gewonnen" (letzte 30 Tage + Jahr) kommt aus der API (`deal.decidedAt`), nicht aus Mails.

### 4. Termine aus den Outlook-Kalendern (Reiter)

Die Termin-Kachel hat umschaltbare Reiter (`DATA.termineTabs`), je Reiter ein Outlook-Kalender.
Abruf jeweils mit `outlook_calendar_search`, `query: "*"`, `afterDateTime: heute`, `beforeDateTime: heute + 21 Tage`, `order: "oldest"`:

| Reiter | Abruf |
|---|---|
| `Montageteam A` | `calendarName: "Montageteam A"` |
| `Montageteam B` | `calendarName: "Montageteam B"` |
| `Büro` | ohne `calendarName` (Standardkalender von Hutterer@oh-solar.at) |

Formatierung:
- **Bereits vergangene Termine des heutigen Tages weglassen** – die Kachel heißt „Nächste Termine".
  Vorsicht beim Filtern: `afterDateTime` wird gegen dieselbe Zeitzone geprüft, in der die Tool-Antwort
  die Zeiten liefert (`timeZone`, meist UTC). Eine Wiener Uhrzeit als Filter schneidet daher 1–2 Stunden
  zu viel weg – entweder die aktuelle **UTC**-Zeit als Filter verwenden oder mit `afterDateTime: heute 00:00`
  abrufen und die vergangenen Termine anschließend selbst herausfiltern.
- Zeiten nach Europe/Vienna umrechnen (im Sommer UTC+2). Termine mit Uhrzeit: `Mo 17.08. · 08:30`. Ganztägige Mehrtages-Termine: `Mo 17.–Di 18.08.` (Achtung: `end` ist exklusiv – letzter Tag = end − 1 Tag).
- Montage-Termine: `what` = Kundenname + Ort (Ortsteil/PLZ aus `location`); Betreff „…Vorläufiger Montagetermin" → `flag: "vorläufig"`. `showAs: "tentative"` → `flag: "unter Vorbehalt"`.
- Max. 5 Einträge je Reiter (sonst wird die Karte abgeschnitten).
- `hinweis` **einzeilig halten** (max. ~55 Zeichen) – bricht er auf zwei Zeilen um, wird der 5. Termin abgeschnitten.
- `hinweis` der Montage-Reiter: **Auslastung** = Anzahl der durch Termine belegten Arbeitstage (Mo–Fr, Vereinigungsmenge der Termintage) in den nächsten 2 Wochen ÷ 10, Format `Montagen · 7 von 10 Arbeitstagen belegt (2 Wochen)`.
- `termineTabWechselSekunden` (Auto-Wechsel der Reiter) unverändert lassen.

Geplant: zusätzlicher Reiter für den **Elektriker-Kalender** – exakter Kalendername muss noch vom
Nutzer genannt werden (unter „Elektriker"/„Elektro" in Hutterers Postfach nicht auffindbar, Stand 10.08.2026).

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
