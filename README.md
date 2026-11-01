# Sandzeit (Arbeitstitel)

Ein textbasiertes Incremental-/Idle-Browserspiel im Stil von *Kittens Game*, erzählerisch angelehnt an Frank Herberts *Dune*.
Das Spiel rechnet Fortschritt auch im Hintergrund-Tab und bei geschlossenem Browser nach (Offline-Fortschritt).

> **Status:** Planungsphase – noch kein Code.

## Kurzfassung

- **Genre:** Ressourcen-Management-Incremental (Kittens Game) + narratives Entdeckungs-Incremental (A Dark Room, Universal Paperclips).
- **Kern:** Wasser ist die zentrale Knappheit, Gewürz die wertvollste und gefährlichste Ressource.
- **Ablauf:** Allein gestrandet → Sietch aufbauen → Gewürzwirtschaft → Politik im Imperium → Begrünung des Planeten → Gilde und Außenwelten → Entscheidung über das Schicksal des Planeten.
- **Prestige:** Generationenwechsel mit „Ahnenerinnerung“; zweite Ebene „Der schmale Pfad“.
- **Technik:** TypeScript, Vite, Preact, statisch gehostet auf GitHub Pages, kein Server.

## Planungsdokumente

| Dokument | Inhalt |
|---|---|
| [01 – Konzept & Mechaniken](docs/01-konzept-und-mechaniken.md) | Genre-Einordnung, Spielschleifen, Phasen, Ressourcen, Berufe, Gebäude, Forschung, Ereignisse, Handel, Ökologie, Prestige, Formeln, Balancing, UI |
| [02 – Geschichte & Erzählung](docs/02-story.md) | Rechtlicher Rahmen, Namensschicht, Prämisse, Figuren, Akte, Entscheidungen, Enden, Beispieltexte |
| [03 – Technik & Architektur](docs/03-technik.md) | Stack, Ordnerstruktur, Datenmodell, Spielschleife, Hintergrund-/Offline-Fortschritt, Speichern, Tests, Deploy |
| [04 – Roadmap](docs/04-roadmap.md) | Meilensteine mit Abnahmekriterien, MVP-Schnitt, Risiken, offene Fragen |

## Hinweis

Fan-Projekt ohne Verbindung zu den Rechteinhabern von *Dune*. Alle Figuren und Texte sind eigene Schöpfungen; Setting-Begriffe laufen über eine austauschbare Namensschicht (siehe [02-story.md](docs/02-story.md#1-rechtlicher-rahmen-und-kanon-nähe)).

---

## Nebenprojekt: Zinseszins-Zählwerk

[`vermoegen/index.html`](vermoegen/index.html) ist eine eigenständige Einzelseite, unabhängig vom Spielplan oben. Sie zeigt im Stil eines Incremental-Spiels, wie ein Vermögen bei durchschnittlich 6 % Rendite (einstellbar) im Sekundentakt wächst:

- Zählwerk mit Rollziffern bis auf Hundertstel-Cent, jede Sekunde ein Takt mit „+x €“-Anzeige; umschaltbar zwischen Gesamtvermögen und dem Zuwachs heute, diese Woche, diesen Monat oder dieses Jahr
- Zeiträume: Zuwachs seit Beginn von Tag/Woche/Monat/Jahr, Hochrechnung bis zum Ende des Zeitraums und Rückblick auf die letzten 24 Std., 7 Tage, den letzten Monat und 12 Monate
- Zeitraffer (1 Minute, 1 Stunde, 1 Tag, 1 Monat oder 1 Jahr pro Sekunde), Ansicht nominal oder als Kaufkraft heute (inflationsbereinigt)
- Kassenstand: Zinsen, Einzahlungen, Netto nach vereinfachter Steuer, 4-%-Entnahme
- Ertragsuhr (Zinsen pro Sekunde bis pro Jahr), Kaufkraft-Karten („alle 3 Std. 12 Min. ein Cappuccino“, Anzahl gezählt ab Stichtag 0:00 Uhr)
- Tresor als Münzraster (Einzahlungen und Zinsen), Meilensteine mit Datumsprognose (große Marken plus kleine Zinsstufen, im ersten Jahr etwa zwei pro Monat), Prognosediagramm und Tabelle
- Eingaben und erreichte Meilensteine bleiben im `localStorage` des Browsers; beim nächsten Öffnen zeigt die Seite den Zuwachs seit dem letzten Besuch

**Online (GitHub Pages):** https://kingjashugan.github.io/W-rfel/vermoegen/
Voraussetzung: In *Settings → Pages* ist unter *Build and deployment* die Quelle „Deploy from a branch“ mit dem Branch `ccr-0fd6f351-yoghm7` (oder später dem Hauptbranch) und dem Ordner `/ (root)` gewählt.

**Lokal:** Datei direkt im Browser öffnen. Kein Build, keine Abhängigkeiten. Nur die Schriften kommen von Google Fonts, ohne Netz greifen Systemschriften.

**Formel:** `V(t) = V₀·(1+r)^t + 12·s·((1+r)^t − 1)/ln(1+r)`: Rendite stetig verteilt, Sparrate als gleichmäßiger Zufluss.
