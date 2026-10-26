# 03 – Technik & Architektur

---

## 1. Technologie-Stack

| Bereich | Empfehlung | Begründung | Alternativen |
|---|---|---|---|
| Sprache | **TypeScript** | Viele verzahnte Datenstrukturen (Ressourcen, Gebäude, Effekte) → Typen verhindern Fehler | JavaScript |
| Build | **Vite** | Schnell, minimale Konfiguration, statischer Output | esbuild, Parcel |
| UI | **Preact + Signals** | Klein, React-ähnlich, feingranulare Aktualisierung (wichtig bei vielen sich ständig ändernden Zahlen) | Svelte 5, SolidJS, Vanilla + lit-html |
| Tests | **Vitest** | Gleiche Konfiguration wie Vite | Jest |
| Codequalität | ESLint + Prettier | Standard | Biome |
| Kompression Spielstand | **lz-string** | Klassiker bei Incremental Games, Export als Text | pako |
| Große Zahlen | **vorerst `number`** (bis ca. 1,8 × 10³⁰⁸) | Kittens-artige Spiele bleiben weit darunter | break_infinity.js / break_eternity.js, falls Prestige-Ebenen eskalieren |
| Hosting | **GitHub Pages** über GitHub Actions | Kostenlos, kein Server nötig | Netlify, Cloudflare Pages, itch.io |
| Backend | **keines** | Alles läuft im Browser | – |

**Zentrale Architekturentscheidung:** Die **Engine ist reines TypeScript ohne DOM-Zugriff**. Die UI liest nur den Zustand und ruft Aktionen auf. Dadurch kann die Engine headless getestet, im Balancing-Bot verwendet und später ggf. in einen Web Worker verschoben werden – unabhängig vom UI-Framework.

---

## 2. Ordnerstruktur

```
/
├─ index.html
├─ vite.config.ts                # base: '/W-rfel/' für GitHub Pages
├─ src/
│  ├─ main.ts                    # Start: Laden → Offline-Berechnung → Loop → UI
│  ├─ engine/                    # reine Spiellogik, kein DOM
│  │  ├─ state.ts                # GameState-Typen, Initialzustand
│  │  ├─ loop.ts                 # Tick-Steuerung, Catch-up
│  │  ├─ offline.ts              # Offline-Simulation, Zusammenfassung
│  │  ├─ rng.ts                  # deterministischer Zufall
│  │  ├─ formulas.ts             # Kosten, Raten, Boni
│  │  ├─ requirements.ts         # Auswertung von Freischaltbedingungen
│  │  ├─ effects.ts              # Anwenden von Effekten
│  │  └─ systems/                # calendar, resources, population, buildings,
│  │                             # research, crafting, events, expeditions, trade,
│  │                             # threat, legend, ecology, guild, story,
│  │                             # prestige, achievements, automation
│  ├─ content/                   # datengetriebene Definitionen
│  │  ├─ resources.ts  buildings.ts  jobs.ts  techs.ts  upgrades.ts
│  │  ├─ events.ts  factions.ts  expeditions.ts  perks.ts  challenges.ts
│  │  └─ story/                  # prolog.ts, act1.ts … epilog.ts
│  ├─ i18n/
│  │  ├─ de.json                 # alle Texte
│  │  └─ names/                  # neutral.json, canon.json (Namensschicht)
│  ├─ save/                      # serialize.ts, migrations.ts, storage.ts
│  └─ ui/                        # Komponenten, Tabs, Tooltips, Log, Dialoge
├─ scripts/
│  └─ simulate.ts                # Balancing-Bot (headless, Node)
├─ tests/
└─ .github/workflows/            # ci.yml (Lint, Test, Bot), deploy.yml (Pages)
```

---

## 3. Datenmodell (Skizze)

```ts
interface GameState {
  version: number;                 // Spielstand-Version für Migrationen
  rng: number;                     // Zustand des deterministischen Zufallsgenerators
  lastTick: number;                // Unix-Zeit (ms) des letzten Ticks
  calendar: { day: number; cycle: CycleId; year: number };
  resources: Record<ResourceId, { amount: number; unlocked: boolean }>;
  buildings: Record<BuildingId, { count: number; active: number }>;
  population: { total: number; jobs: Record<JobId, number>; growthTimer: number };
  techs: TechId[];
  upgrades: UpgradeId[];
  expeditions: { regionId: RegionId; endsAt: number }[];   // Endzeit als Spielzeit
  factions: Record<FactionId, { standing: number }>;
  threat: { value: number; tributePaidYear: number };
  legend: { accepted: boolean | null; invested: number };
  spiceRation: { level: 0 | 1 | 2 | 3; dependencyDays: number };
  ecology: { moisture: number; worms: number };
  story: {
    act: number;
    flags: Record<string, boolean | number | string>;
    seen: EventId[];
    pendingChoices: EventId[];
  };
  prestige: {
    generation: number;
    memoryTotal: number;            // insgesamt erhaltene AE
    memoryFree: number;             // nicht ausgegebene AE
    perks: Record<PerkId, number>;
    providence: number;             // Vorsehung (Ebene 2)
    endings: EndingId[];
    archivedFlags: Record<string, unknown>[];   // Flags früherer Generationen
  };
  automation: Record<AutomationId, { enabled: boolean; threshold: number }>;
  stats: { lifetime: Partial<Record<ResourceId, number>>; playTimeMs: number };
  log: LogEntry[];                  // begrenzt (z. B. 200 Einträge)
  settings: Settings;
}
```

Der Zustand ist **vollständig JSON-serialisierbar** (keine Klassen, keine `Set`/`Map`), damit Speichern, Export und Migrationen trivial bleiben.

---

## 4. Inhalte datengetrieben definieren

Inhalte sind **Daten**, keine Logik. Bedingungen und Effekte sind deklarative Objekte – dadurch sind sie serialisierbar, testbar und später auch per Tabelle/Editor pflegbar.

```ts
// content/buildings.ts
export const windtrap: BuildingDef = {
  id: 'windtrap',
  phase: 1,
  cost: { scrap: 50, rock: 20 },
  costRatio: 1.15,
  requires: [{ tech: 'windtraps' }],
  effects: [{ type: 'produce', resource: 'water', perSecond: 0.4 }],
  weather: { storm: 1.3 },
};
```

```ts
// content/story/act1.ts
export const waterOath: StoryEventDef = {
  id: 'act1.waterOath',
  trigger: { all: [{ flag: 'act1.started' }, { population: { gte: 8 } }] },
  text: 'story.act1.waterOath.text',
  epigraph: 'epigraph.rahila.water',
  choices: [
    {
      id: 'share',
      text: 'story.act1.waterOath.share',
      effects: [
        { type: 'addPopulation', amount: 3 },
        { type: 'setFlag', flag: 'act1.sharedWater', value: true },
        { type: 'standing', faction: 'sandfolk', delta: 20 },
      ],
    },
    {
      id: 'hoard',
      text: 'story.act1.waterOath.hoard',
      effects: [
        { type: 'storageBonus', resource: 'water', percent: 25 },
        { type: 'setFlag', flag: 'act1.sharedWater', value: false },
        { type: 'standing', faction: 'sandfolk', delta: -20 },
      ],
    },
  ],
};
```

---

## 5. Spielschleife

- **Fester Simulationsschritt:** 200 ms (5 Ticks/s). Die Simulation rechnet immer mit festen Schritten → reproduzierbar, unabhängig von der Bildwiederholrate.
- **Zeitbasis ist die Wanduhr** (`Date.now()`), nicht die Anzahl der Aufrufe. `performance.now()` läuft auf manchen Plattformen während des System-Ruhezustands nicht weiter und ist daher für lange Pausen ungeeignet.
- **Simulation per `setInterval`, Darstellung per `requestAnimationFrame`.** `requestAnimationFrame` wird in Hintergrund-Tabs komplett angehalten und darf deshalb die Simulation nicht antreiben.

```ts
const TICK_MS = 200;
const CATCHUP_THRESHOLD_MS = 60_000;   // größere Lücken → Offline-Pfad

let acc = 0;
let last = Date.now();

setInterval(() => {
  const now = Date.now();
  const elapsed = Math.max(0, now - last);   // Uhr zurückgestellt → 0
  last = now;

  if (elapsed > CATCHUP_THRESHOLD_MS) {
    showSummary(simulateOffline(state, elapsed));   // Abschnitt 6.3
    return;
  }
  acc += elapsed;
  while (acc >= TICK_MS) {
    engine.tick(state, TICK_MS / 1000);
    acc -= TICK_MS;
  }
}, TICK_MS);

function frame() {
  ui.render(state);            // UI höchstens ~10× pro Sekunde neu zeichnen
  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```

**Reihenfolge innerhalb eines Ticks:**

1. Kalender (Tag, Zyklus, Jahr)
2. Modifikatoren berechnen (Wetter, Moral, Boni) – gecacht, nur bei Änderungen neu
3. Produktion und Verbrauch anwenden, auf Lager begrenzen
4. Bevölkerung (Wachstum, Mangel)
5. Automatisierung (Auto-Handwerk usw.)
6. Expeditionen, Timer
7. Zufallsereignisse
8. Story-Trigger
9. Erfolge

---

## 6. Hintergrund- und Offline-Fortschritt

„Läuft im Hintergrund weiter“ umfasst drei technisch unterschiedliche Fälle:

| Fall | Was passiert im Browser | Lösung |
|---|---|---|
| **Tab im Hintergrund** | Timer werden gedrosselt. In Chromium-Browsern laufen Timer in versteckten Tabs höchstens etwa einmal pro Sekunde, nach einigen Minuten teils nur noch etwa einmal pro Minute. Andere Browser drosseln ähnlich. | Wanduhr-basierte Schleife (Abschnitt 5) holt die verpasste Zeit beim nächsten Aufruf nach |
| **Tab eingefroren oder verworfen** (z. B. Speicher-Sparmodus) | Seite wird angehalten oder beim Zurückkehren neu geladen | Beim Verstecken speichern (`visibilitychange`, `pagehide`); beim Laden Offline-Berechnung |
| **Browser geschlossen** | Kein Code läuft – auch kein Service Worker kann eine Spielschleife dauerhaft ausführen | Beim nächsten Start Offline-Berechnung aus dem gespeicherten Zeitstempel |

### 6.1 Wichtige Konsequenz

Das Spiel „läuft“ bei geschlossenem Browser nicht wirklich – es **rechnet beim nächsten Öffnen nach**, was in der Zwischenzeit passiert wäre. Das ist bei allen bekannten Browser-Incrementals so. Echte Hintergrundausführung bräuchte einen Server, der den Spielstand verwaltet; das ist für dieses Projekt unverhältnismäßig.

Ebenfalls nicht ohne Server möglich: **Benachrichtigungen bei geschlossenem Browser** („Deine Expedition ist zurück“). Push-Benachrichtigungen benötigen einen Push-Server. Für das MVP: keine Benachrichtigungen; Benachrichtigungen nur bei geöffnetem, aber verstecktem Tab (Notification API) als optionale Einstellung.

### 6.2 Offline-Regeln

| Regel | Wert / Verhalten |
|---|---|
| Offline-Obergrenze | 12 h zu Beginn, erweiterbar über das Merkmal „Wachsame Ahnen“ (+4 h je Stufe) |
| Offline-Ertragsrate | 100 % (Empfehlung; Alternative: reduzierte Rate, z. B. 50 %, als bewusster Anreiz für aktives Spiel) |
| Klickbare Ereignisse | Werden offline nicht erzeugt; stattdessen Erwartungswert-Auflösung (z. B. 25 % des Gewürzeruptions-Bonus) |
| Wurmzeichen, Überfälle | Werden offline per Zufall aufgelöst (deterministischer Zufallsgenerator) |
| Story-Ereignisse ohne Entscheidung | Treten ein und erscheinen in der Zusammenfassung |
| Story-Entscheidungen | Werden in die Warteschlange gestellt |
| Uhr zurückgestellt | Negative Zeitdifferenz → 0, keine Strafe |
| Uhr vorgestellt (Schummeln) | Wird durch die Offline-Obergrenze begrenzt; bei einem Einzelspieler-Spiel ohne Ranglisten nicht weiter verhindert |

### 6.3 Offline-Simulation

Einfaches Hochrechnen (`Rate × Zeit`) ist **falsch**, weil sich der Zustand währenddessen ändert: Lager laufen voll, Wasser geht aus, Bewohner wandern ab, Bevölkerung wächst. Deshalb wird dieselbe `tick`-Funktion mit **gröberen Schritten** ausgeführt:

```ts
function simulateOffline(state: GameState, elapsedMs: number): OfflineSummary {
  const capped = Math.min(elapsedMs, offlineCapMs(state));
  const MAX_STEPS = 2_000;
  const step = Math.max(TICK_MS, capped / MAX_STEPS);   // 12 h → ca. 22 s pro Schritt
  const before = snapshot(state);

  for (let t = 0; t < capped; t += step) {
    engine.tick(state, Math.min(step, capped - t) / 1000, { offline: true });
  }
  return diff(before, state);   // „Während du fort warst …“
}
```

- Begrenzung auf ca. 2 000 Schritte hält die Berechnung beim Laden deutlich unter einer Sekunde.
- **Test-Pflicht:** Offline-Ergebnis muss innerhalb einer Toleranz (z. B. ±2 %) mit einer Online-Simulation gleicher Dauer übereinstimmen (Abschnitt 11).
- Optional später: Engine in einen **Web Worker** verlegen, um Ruckler beim Nachrechnen zu vermeiden. Nicht für das MVP.

---

## 7. Speichern

| Aspekt | Umsetzung |
|---|---|
| Format | JSON → lz-string-Kompression |
| Speicherort | `localStorage`, Schlüssel mit Präfix `sandzeit.` |
| Autosave | alle 30 s, zusätzlich bei `visibilitychange` (versteckt) und `pagehide` (`beforeunload` ist auf Mobilgeräten unzuverlässig) |
| Backups | Rotierende Sicherungen der letzten 3 Spielstände (Schutz gegen beschädigte Spielstände) |
| Export / Import | Spielstand als Text (Base64) kopieren und einfügen, zusätzlich als Datei herunterladen |
| Versionierung | `version` im Spielstand; Migrationsfunktionen `migrations[v](alterStand) → neuerStand` |
| Persistenz anfragen | `navigator.storage.persist()` beim ersten Start |

**Blinde Flecken beim Speichern:**

1. **Gemeinsame Herkunft auf GitHub Pages:** Alle GitHub-Pages-Projekte eines Kontos laufen unter derselben Herkunft (`<konto>.github.io`) und teilen sich daher `localStorage`. Ein anderes Projekt könnte gleichnamige Schlüssel überschreiben → eindeutiges Präfix verwenden; alternativ eigene Domain.
2. **Safari löscht Daten:** Safari kann per Skript geschriebene Speicherdaten von Websites löschen, die einige Tage nicht besucht wurden (Tracking-Schutz). Für ein Idle-Spiel mit längeren Pausen kritisch → regelmäßige Export-Erinnerung, Installation als PWA anbieten, `navigator.storage.persist()`.
3. **Privates Surfen:** Speicherung kann fehlen oder beim Schließen gelöscht werden → Hinweis im Spiel.

---

## 8. Zufall

- **Deterministischer, gesäter Zufallsgenerator** (z. B. mulberry32 oder sfc32), dessen Zustand im Spielstand liegt.
- Vorteile: reproduzierbare Tests und Bot-Läufe; kein „Neu laden, bis das Ereignis gut ausgeht“.

---

## 9. Zahlen und Formatierung

- `Intl.NumberFormat('de-DE')` für Dezimalkomma und Tausenderpunkte.
- Kurzschreibweise ab 10 000: 12,3 Tsd. · 4,56 Mio. · 7,89 Mrd. · 1,23 Bio. …
- Einstellbar: wissenschaftliche Schreibweise (1,23e15).
- Raten immer „/s“ anzeigen, intern pro Tick rechnen.

---

## 10. Darstellung und Leistung

- UI-Aktualisierung höchstens ca. 10× pro Sekunde, nicht pro Tick.
- Nur geänderte Werte neu zeichnen (Signals).
- Modifikatoren cachen und nur bei Kauf/Forschung/Ereignis neu berechnen.
- Log auf 200 Einträge begrenzen; vollständige Story-Texte liegen in der Chronik.

**Barrierefreiheit:**

- Echte `<button>`-Elemente, vollständig per Tastatur bedienbar.
- `aria-live="polite"` nur für Story-Einträge im Log, nicht für jede Zahlenänderung (sonst unbrauchbar für Screenreader).
- Ausreichender Kontrast in beiden Farbthemen; `prefers-reduced-motion` respektieren.
- Kosten direkt am Knopf sichtbar (Touch-Geräte haben kein Hover).

**Mobil:** Responsives Layout; auf schmalen Bildschirmen Ressourcen und Log als ein- und ausklappbare Leisten.

---

## 11. Tests und Balancing-Bot

| Testart | Inhalt |
|---|---|
| Unit-Tests | Formeln (Kosten, Summen, Boni), Bedingungsauswertung, Effekte |
| Spielstand-Tests | Speichern → Laden ergibt identischen Zustand; jede Migration mit Beispielspielstand |
| **Offline-Äquivalenz** | Online-Simulation über X Stunden vs. Offline-Simulation → Abweichung ≤ Toleranz |
| Inhaltsvalidierung | Jede referenzierte ID existiert; jeder Text-Schlüssel existiert in `de.json`; keine unerreichbaren Inhalte |
| **Balancing-Bot** | `scripts/simulate.ts` spielt headless mit einer einfachen Strategie (günstigstes sinnvolles Gebäude kaufen, Berufe nach Engpass verteilen, Entscheidung nach fester Regel) und schreibt die Zeit bis zu jedem Meilenstein als CSV |

Der Bot läuft in der CI und schlägt fehl, wenn Meilenstein-Zeiten außerhalb definierter Korridore liegen. Damit fällt auf, wenn eine Inhaltsänderung das Balancing zerstört.

---

## 12. Build und Veröffentlichung

- **CI (`ci.yml`):** bei jedem Push: Lint, Typprüfung, Tests, Bot-Lauf.
- **Deploy (`deploy.yml`):** bei Push auf den Hauptbranch: `vite build` → GitHub Pages.
- `vite.config.ts`: `base: '/W-rfel/'` (Repository-Name), solange keine eigene Domain genutzt wird.
- Optional: **PWA** (Manifest + Service Worker) für Installation und Offline-Spielbarkeit. Der Service Worker cached nur die Dateien – er führt die Spielschleife nicht aus (siehe Abschnitt 6).
