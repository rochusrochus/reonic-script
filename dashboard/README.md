# Kennzahlen-Dashboard – OH Voltaik Solaranlagenbau GmbH

Ein automatisch aktualisiertes Dashboard für das Büro-Display mit Angebots- & Auftragslage,
anstehenden Terminen und (nach Reonic-API-Anbindung) Umsatz, Gewinn und Team-Auslastung.

**Dashboard-URL:** https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35

## Wie es funktioniert

```
Claude-Routine „OH Voltaik Dashboard-Update“ (werktags stündlich, ca. 7–18 Uhr)
  → liest Reonic-Benachrichtigungsmails + Outlook-Kalender (Microsoft 365)
  → berechnet die Kennzahlen
  → befüllt dashboard/template.html und publiziert die Seite unter der festen URL
```

- Die genauen Schritte der Routine stehen in [`UPDATE.md`](UPDATE.md).
- Die Seite lädt sich selbst alle 15 Minuten neu, das Display zeigt also automatisch die neueste Version.
- Die Zahlen zur Angebots-/Auftragslage sind **Näherungswerte** aus den Reonic-Mails an
  hutterer@oh-solar.at – echte Umsatz-/Gewinnzahlen folgen mit der API-Anbindung
  (siehe [`reonic-api.md`](reonic-api.md)).

## Display im Büro einrichten

1. Dashboard-URL im Browser des Displays öffnen (beim ersten Mal mit dem Claude-Konto anmelden,
   oder das Artifact über das Teilen-Menü der Seite freigeben und den Freigabe-Link verwenden).
2. Browser in den Vollbild-/Kiosk-Modus schalten:
   - **Windows (Chrome/Edge):** `chrome.exe --kiosk "https://claude.ai/code/artifact/16e32cd8-4df8-40f8-a3a1-6dfa9018bb35"` bzw. F11
   - **Smart-TV/Fire-Stick:** Browser-App mit Startseite = Dashboard-URL
3. Bildschirm-Standby des Geräts deaktivieren.

## Routine verwalten

Einfach Claude im Chat bitten, z.B.:
- „Pausiere die Dashboard-Routine“ / „Aktiviere sie wieder“
- „Aktualisiere das Dashboard jetzt sofort“
- „Ändere den Rhythmus auf 1× täglich morgens“

Technisch: Die Routine heißt **„OH Voltaik Dashboard-Update“** (Cron `0 5-16 * * 1-5` UTC =
7–18 Uhr Wien im Sommer, 6–17 Uhr im Winter).

## Dateien

| Datei | Zweck |
|---|---|
| `template.html` | Die Dashboard-Seite; die Routine ersetzt nur den `DATA`-Block |
| `UPDATE.md` | Schritt-für-Schritt-Anleitung, die die Routine bei jedem Lauf ausführt |
| `reonic-api.md` | Anleitung: Reonic-V3-API-Schlüssel erzeugen und übergeben (für Umsatz & Auslastung) |

**Wichtig:** Niemals API-Schlüssel oder Passwörter in dieses Repository committen.
