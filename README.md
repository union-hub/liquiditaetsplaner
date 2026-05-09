# Persönlicher Liquiditätsplaner

Schlankes Single-File-Tool zur privaten Finanzplanung — läuft als Progressive Web App im Browser, speichert ausschließlich lokal, kein Backend, kein Tracking.

![Stack](https://img.shields.io/badge/stack-Vanilla%20JS-yellow) ![PWA](https://img.shields.io/badge/PWA-offline--ready-blue) ![Daten](https://img.shields.io/badge/Daten-100%25%20lokal-success)

## Funktionen

| Tab | Zweck |
|---|---|
| **Dashboard** | KPIs (Einnahmen/Ausgaben/Saldo/Kontostand), Notfallreserve-Widget, Budget-Ampel, anstehende Fälligkeiten |
| **Einnahmen / Ausgaben** | Plan-Einträge mit Wiederholung (monatlich/jährlich/quartalsweise/einmalig), Fälligkeitstag, Kategorie, Zeitraum |
| **Buchungen** | Tag-genaues Transaktionsjournal — geplante Posten abhaken (`✓ Buchen`) oder spontane Posten erfassen |
| **Sparziele** | Ziele mit Ist-Stand, Notfallreserve (auto = 3× Fixkosten oder manuell) |
| **Vorschau** | 12-Monats-Forecast mit Chart.js |
| **Sichern** | Kontostand-Snapshot, Kategorien & Budgets verwalten, CSV-Export, HTML-Export (Daten eingebettet) |

## Datenmodell

Drei Konzepte, die zusammen den Liquiditätsstand bilden:

- **Einnahmen / Ausgaben** = wiederkehrende Vorlagen (Plan/Forecast)
- **Buchungen** = tatsächlich ausgeführte Transaktionen mit Datum
- **Kategorie-Budgets** = optionale Limits pro Kategorie (monatlich/jährlich), Verbrauch via Buchungen

Verknüpfung: Eine Buchung kann via `planRef` an einen Plan-Eintrag gebunden sein. Dadurch wird der Plan-Eintrag im jeweiligen Monat **nicht doppelt** in KPIs und Budget-Ampel gezählt.

## Kontostand-Logik

Im Tab _Sichern_ wird ein Kontostand-Snapshot mit Datum eingetragen. Das Dashboard projiziert daraus:

- **Snapshot-Monat:** Stand zum Stichtag + alle Posten **strikt nach** dem Stichtag (Buchungs-Datum oder Plan-`faelligTag`)
- **Folgemonate:** zusätzlich der jeweilige Monatssaldo (Plan minus gebucht-via-`planRef` plus Buchungen)
- **Ansicht vor Snapshot:** zeigt den Snapshot-Wert (kein Backtracking)

## Datenspeicherung

- 100 % im `localStorage` des Browsers unter dem Schlüssel `liqplaner_v3`
- Keine API, kein Cloud-Sync, keine Telemetrie
- Einzige externe Ressource: `chart.js` von jsDelivr (wird vom Service Worker offline gecacht)

> **Wichtig:** Browser-Cache leeren / anderes Gerät = Daten weg. Regelmäßiger HTML-Export im Sichern-Tab empfohlen — die exportierte Datei enthält die Daten als JSON eingebettet und kann später wieder importiert werden.

## Installation & Nutzung

### Direkt im Browser
```bash
git clone https://github.com/union-hub/liquiditaetsplaner.git
cd liquiditaetsplaner
python -m http.server 8765
# http://localhost:8765/index.html im Browser öffnen
```

### Als PWA installieren
Im Chrome/Edge auf das Install-Icon in der Adressleiste klicken (oder Menü → "App installieren"). Auf iOS Safari: Teilen → "Zum Home-Bildschirm".

### GitHub Pages
Repository-Settings → Pages → Source: `main` Branch → Speichern. Tool ist dann unter `https://<user>.github.io/liquiditaetsplaner/` erreichbar.

## Tech-Stack

- **Vanilla JavaScript** — keine Build-Tools, keine Dependencies außer Chart.js
- **Chart.js 4.4.0** (CDN) für die 12-Monats-Vorschau
- **Service Worker** mit Versionierungs-Cache (`liqplaner-v4`) für Offline-Betrieb
- Alles in einer einzigen `index.html` — leicht zu lesen, leicht zu auditieren

## Projektstruktur

```
.
├── index.html      # gesamte App (HTML + CSS + JS)
├── sw.js           # Service Worker (Cache-Strategie)
├── manifest.json   # PWA-Manifest
└── README.md
```

## Datenmigration

Beim Laden wird automatisch von älteren Schema-Versionen migriert (additiv, ohne Datenverlust). Aktueller LocalStorage-Key: `liqplaner_v3`. Fallback liest auch `liqplaner_v2`.
