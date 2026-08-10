# Reonic-API-Zugang einrichten (für Umsatz, Gewinn & Team-Auslastung)

> **Status 10.08.2026: ERLEDIGT.** V3-Schlüssel wurde generiert, die Netzwerkfreigabe gesetzt und
> Umsatz/Gewinn + Pipeline sind im Dashboard live. Die Anleitung unten bleibt als Referenz
> (z.B. falls der Schlüssel erneuert wird). Offen ist nur noch die **Team-Auslastung** –
> sie erfordert, dass die Einsatzplanung (Kalender) in Reonic tatsächlich gepflegt wird.
>
> Technische Eckdaten der Anbindung: Basis-URL `https://api.reonic.de/rest/v3`,
> Auth-Header `X-Authorization: <Key>`, Doku unter https://api.reonic.de/rest/v3/docs,
> Rate-Limit: bei HTTP 429 mit Backoff wiederholen, Datumsfilter max. 365 Tage.

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

## Schritt 2: Netzwerkzugriff auf reonic.de freischalten

Die Claude-Code-Umgebung blockiert standardmäßig fremde Domains. Damit Claude die Reonic-API
erreichen kann: claude.ai/code → **Environments** → eure Umgebung → **Network access** →
folgende Domains erlauben (oder Netzwerkzugriff auf „alle Domains" stellen):

- `reonic.de` inkl. Subdomains (`apps.reonic.de`, `docs.reonic.de`, ggf. `api.reonic.de`)

Die offizielle API-Doku liegt unter: https://docs.reonic.de/docs/integrations/rest/
(API-Schlüssel werden laut Doku unter **Einstellungen → Integrationen** generiert.)

## Schritt 3: Schlüssel sicher an Claude übergeben

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
