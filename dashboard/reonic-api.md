# Reonic-API-Zugang einrichten (für Umsatz, Gewinn & Team-Auslastung)

Gute Nachricht: Laut der Reonic-Mail vom 08.07.2026 („Aktion erforderlich: Migration zur API V3")
nutzt euer Reonic-Konto die API bereits (noch V2) – und **den neuen V3-Schlüssel könnt ihr selbst
erzeugen**, ganz ohne Support-Anfrage.

## Schritt 1: V3-API-Schlüssel generieren

1. Bei Reonic anmelden (apps.reonic.de).
2. **Portaleinstellungen** öffnen.
3. Dort einen **neuen V3-API-Schlüssel generieren** (lt. Reonic-Mail „Migration in 3 Schritten", Schritt 1).

Hinweise aus der Reonic-Mail:
- Neue Basis-URL: `/rest/v3/` (statt `/rest/v2/`)
- `/clients/{clientId}` entfällt aus den Anfragepfaden
- Die alten V2-Endpunkte laufen mindestens bis **01.10.2026**
- Die vollständige API-Dokumentation ist in der Mail vom 08.07.2026 verlinkt („Zur Dokumentation")

Bei Fragen ist euer Ansprechpartner bei Reonic: **Niklas Cosmann** (Head of Partnerships),
niklas.cosmann@reonic.de – Referenz: Client-ID `47d9ba16-c5c7-4757-9c8d-ba923f27709b`.

## Schritt 2: Schlüssel sicher an Claude übergeben

**Den Schlüssel NIEMALS in dieses Repository committen oder per unverschlüsselter Mail verschicken.**

Empfohlener Weg: In den Einstellungen der Claude-Code-Umgebung (claude.ai/code → Environments →
Umgebung dieser Sessions) eine Umgebungsvariable anlegen:

```
REONIC_API_KEY=<der generierte V3-Schlüssel>
```

Danach Claude in einer neuen Session sagen: „Der Reonic-API-Schlüssel ist als REONIC_API_KEY
hinterlegt – bitte Phase B des Dashboards umsetzen."

## Was Claude dann freischaltet (Phase B)

- **Umsatz & Gewinn** – Monat, Quartal, Jahr (aus Auftrags-/Rechnungsdaten)
- **Auslastung Montage-Teams** – geplante Einsätze pro Team/Elektriker aus der
  Reonic-Einsatzplanung, nächste 2 Wochen (geplante Stunden ÷ verfügbare Stunden)
- **Angebots-Pipeline direkt aus der API** statt aus den Benachrichtigungsmails
  (genauer, inkl. Angebotswerten in Euro)

Die Update-Routine (`UPDATE.md`) wird dabei um die API-Abrufe erweitert; die beiden
Platzhalter-Kacheln im Dashboard verschwinden.
