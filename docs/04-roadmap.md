# 04 – Roadmap, Risiken, offene Fragen

---

## 1. Meilensteine

Aufwandsklassen (sehr grobe Schätzung für eine Person in Freizeit-Tempo, ohne Erfahrungswerte aus diesem Projekt):
**S** ≈ 1–3 Tage · **M** ≈ 1–2 Wochen · **L** ≈ 2–4 Wochen

| # | Meilenstein | Inhalt | Abnahmekriterien | Aufwand |
|---|---|---|---|---|
| M0 | Projektgerüst | Vite + TypeScript + Preact, ESLint, Prettier, Vitest, CI, Deploy auf GitHub Pages | Leere Seite ist auf GitHub Pages erreichbar; CI ist grün | S |
| M1 | Engine-Kern | Zustand, Ressourcen, Lager, Gebäudekosten, Tick-Schleife, Speichern/Laden, Export/Import, Offline-Berechnung, Zahlformat, Zufallsgenerator | Testgebäude produziert; nach 1 h Abwesenheit stimmt der Offline-Ertrag mit Online-Simulation überein (±2 %); Tests grün | M |
| M2 | **Vertical Slice: Prolog + Akt I** | Klick-Aktionen, Bevölkerung, Berufe, Moral, Gebäude Phase 0–1, Log, Story-Engine (Trigger, Flags, Entscheidungen), progressive UI | Ein neuer Spieler erreicht den Wasserschwur in 30–60 min ohne Anleitung | M–L |
| M3 | Forschung, Werkstatt, Kalender | Forschungsbaum Phase 1–2, Handwerk, Werkstatt-Upgrades, Zyklen und Wetter | Alle Inhalte von Phase 1 erreichbar; Bot-Lauf misst Zeiten | M |
| M4 | **Gewürz-Phase (Akt II)** | Gewürz, Destillanzüge als Berufsbedingung, Erntefabriken, Wurmzeichen, Gewürzeruption, Expeditionen, Handel mit 2–3 Fraktionen | Akt II durchspielbar; aktive und Offline-Auflösung der Ereignisse getestet | L |
| M5 | Imperium (Akt III) | Legende, Einfluss, Fraktionen vollständig, Bedrohung/Verteidigung, Tribut, Mentaten, Auto-Handwerk | Akt III inkl. „Nacht der Stürme“ in allen Varianten durchspielbar | L |
| M6 | **Prestige-Ebene 1** | Generationenwechsel, Ahnenerinnerung, Blutlinien-Merkmale, Ahnenstimmen, variierende Starttexte, Erfolge | Zweite Generation erreicht Akt III spürbar schneller (Bot-Messung) | M |
| M7 | Ökologie (Akt IV) | Feuchtigkeit, Wurmpopulation, Pflanzstation, Qanat, Dünenanker | Zielkonflikt Wasser ↔ Gewürz im Bot sichtbar | L |
| M8 | Gilde (Akt V) | Kontrakte, Außenwelt-Missionen, Druckmittel Gewürz | Akt V durchspielbar | L |
| M9 | Endgame | Enden A–D, Prestige-Ebene 2, Prüfungen, volle Automatisierung | Alle Enden erreichbar (per Bot oder Testspielstand) | L |
| M10 | Politur | Balancing-Durchgänge, Barrierefreiheit, PWA, Einstellungen, helles Thema, Texte überarbeiten | Playtest mit mindestens 3 Personen ohne blockierende Probleme | M |

**Parallel-Strang „Text“:** Story-Texte, Epigraphe und Ahnenstimmen werden pro Akt *vor* dem jeweiligen Technik-Meilenstein als Entwurf geschrieben (Umfang siehe [02-story.md, Abschnitt 10](02-story.md#10-umfang-schätzung-für-die-erste-fassung)).

### MVP-Schnitt

**MVP = M0 bis M4 + vereinfachtes M6** (Prestige ohne Ahnenstimmen):
Ein spielbares Spiel mit ca. 2–4 Tagen Inhalt, Offline-Fortschritt und einem ersten Reset. Alles danach baut darauf auf, ohne dass die Grundarchitektur geändert werden muss.

---

## 2. Risiken

| Risiko | Auswirkung | Gegenmaßnahme |
|---|---|---|
| **Umfang** (Kittens Game ist über Jahre gewachsen) | Projekt wird nie fertig | Harter MVP-Schnitt; jede Phase für sich spielbar |
| Balancing-Aufwand unterschätzt | Spiel zu zäh oder zu schnell | Balancing-Bot ab M3, Korridore in der CI |
| Rechte am *Dune*-Universum | Veröffentlichung problematisch | Eigene Figuren, Namensschicht, keine Zitate (siehe 02-story.md, Abschnitt 1) |
| Offline-Berechnung ungenau | Spieler verlieren oder „gewinnen“ Fortschritt | Offline-Äquivalenztest ab M1 |
| Spielstandverlust (Safari, gemeinsame Herkunft auf GitHub Pages) | Frust, Abbruch | Präfix, Backups, Export-Erinnerung, PWA |
| Story kollidiert mit Idle-Spielweise (Texte werden übersprungen) | Geschichte wirkt nicht | Kurze Log-Zeilen, Chronik, Entscheidungen mit spürbaren Folgen |
| Leistung bei vielen Ressourcen/Gebäuden | Ruckeln | Gecachte Modifikatoren, gedrosseltes Rendering |

---

## 3. Offene Fragen (Entscheidungen, die bei dir liegen)

| # | Frage | Optionen | Empfehlung |
|---|---|---|---|
| 1 | Soll das Spiel **öffentlich** erscheinen oder privat bleiben? | öffentlich / privat | Entscheidet über Variante A oder B (Namensschicht) |
| 2 | **Kanon-Begriffe** oder eigene Namen? | Kanon / neutral / umschaltbar | Umschaltbar, Standard neutral |
| 3 | **Kanon-Nähe der Handlung**: eigene Seitengeschichte oder Nacherzählung der Romanhandlung? | Seitengeschichte / Nacherzählung | Seitengeschichte – eine Nacherzählung lässt kaum Spielerentscheidungen zu und ist rechtlich heikler |
| 4 | **Ziel-Spieldauer** bis zum ersten Ende | Wochen / Monate | 2–3 Monate (siehe Balancing-Ziele) |
| 5 | **Sprache** | nur Deutsch / Deutsch + Englisch | Deutsch, Texte aber von Anfang an in `de.json` (Übersetzung später möglich) |
| 6 | **Plattform** | Desktop zuerst / mobil gleichwertig | Desktop zuerst, mobil nutzbar |
| 7 | **Repository**: Soll das Spiel hier in „W-rfel“ entstehen? Der Name deutet auf ein anderes Vorhaben („Würfel“) hin | hier / neues Repository | Klären, bevor Code entsteht (beeinflusst auch den Pfad auf GitHub Pages) |
| 8 | **Technik-Vorkenntnisse**: Kennst du TypeScript/React? | – | Bei React-Kenntnissen Preact; sonst ggf. Svelte prüfen |
| 9 | **Klickanteil** | viel / wenig | Klicken nur im Prolog und bei Bonus-Ereignissen |
| 10 | **Offline-Rate** | 100 % / reduziert | 100 % mit Obergrenze |
| 11 | **Grafik** | rein Text / Icons / Illustrationen | Rein Text, später Icons |
| 12 | **Kampf** | abstrakt / taktisch | Abstrakt (Bedrohung vs. Verteidigung) |

---

## 4. Nächste Schritte

1. Offene Fragen 1–3 und 7 klären (Rechte, Kanon-Nähe, Repository).
2. M0 umsetzen: Projektgerüst mit CI und Deploy.
3. M1 umsetzen: Engine-Kern inkl. Offline-Berechnung und Tests.
4. Prolog- und Akt-I-Texte als Entwurf schreiben.
5. M2 (Vertical Slice) – danach erster Playtest, bevor weitere Phasen gebaut werden.
