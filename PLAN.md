# The Abbey of the Wrinkled Lord — a Pug Sanctuary task game

> A cozy, mobile-friendly React SPA. You play the sole caretaker of a misty island monastery
> run by lazy, snorting, deeply revered pugs. Your **real-world tasks** (taxes, a mural,
> garden posts, an oil change) arrive as solemn **Decrees** from the Wrinkled Lord. Doing them
> in real life raises sanctuary buildings, brings new pugs, earns Kibble, and builds
> **Devotion**. Devotion unlocks **Pilgrimages**: playful in-game set pieces that physically
> change the island (bridges clear, catacombs open, the weather changes).
> The sanctuary grows from a cramped cottage into a sprawling, absurd dog-empire.

---

## 0. Decisions

| Area        | Decision                                                                                                                            |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Theme       | **Pug Sanctuary: the Abbey of the Wrinkled Lord.** Cozy pastel plus gothic-monastic drama, played completely straight for the jokes |
| Player      | The **sole caretaker**: a fixed character who walks the island                                                                      |
| Real tasks  | Shown as **Decrees** (scrolls with wax seals) and tied to sanctuary **institutions** (buildings)                                    |
| Play        | Wander, tend the pugs (fidget layer), and play **Pilgrimages** (set-piece mini-quests that unlock the world)                        |
| Rewards     | Permanent buildings and pugs; climate/lighting shifts; **Kibble** (spendable); **Devotion** (lifetime, gates Pilgrimages)           |
| Kibble buys | Decor (shrines, statues, cushions, stained glass), pug treats, land expansion                                                       |
| Task data   | Resources/links, steps, recurring schedule, due dates, effort                                                                       |
| Art         | Soft vector **SVG**, in a hand-drawn pastel style with gothic touches: stained glass, fog, cobblestones, candlelight                |
| Stack       | **React** + TypeScript SPA, mobile-first, self-hosted                                                                               |
| Data        | **Local-only first** (IndexedDB), with a **small Supabase backend** in the last phase                                               |

### How real tasks and in-game quests fit together
The theme brief describes in-game quests (Snore Synchronization and so on), while the game's
purpose is real-world tasks. Both are kept, in separate layers:

- **Decrees are real tasks.** Only real completions grow buildings, bring pugs, and earn Devotion.
- **Pilgrimages are the in-game quests.** They're fun to play, but they **unlock only once real-world
  Devotion reaches a threshold**. Real progress opens the door, and the game makes going through it fun.
- **Tending is the fidget.** Small rewards, a daily cap, no pressure.

### Design principles
- **The Decree list comes first.** It's always one tap away and readable over the island.
- **Gentle, never punishing.** Overdue Decrees bring fog and sulking pugs. Nothing is lost.
- **Solemn absurdity.** Every bit of text treats pugs as medieval royalty. Monks treat an ear-scratch like a papal bull.
- **Theme is data.** Institutions, pugs, Pilgrimages, decor and copy live in a theme config, so content can be tuned without code changes.

---

## 1. The island

```
                    ☁  ☁        (sky — stained-glass pug clouds)
   ┌─────────────── WILD CLIFFS (foraging) ────────────────┐
   │                                                       │
   │  SLEEPING QUARTERS     CLOISTER COURTYARD   ══BRIDGE══╪══ FORBIDDEN APPLE ORCHARD
   │  (snoring pugs)        (weather vane)       (Lord Chonk)    (locked)
   │                                                       │
   │        CARETAKER'S COTTAGE (start)   INSTITUTION LOTS │
   └───────────────────────────────────────────────────────┘
          ▼ (locked trapdoor)
   DORMITORY CATACOMBS — underground layer (locked)
   Hot springs — hidden across the map (locked)
```

- **Start:** a cramped cottage, a muddy yard, the courtyard, the sleeping quarters, the cliff path.
  Lord Chonk blocks the bridge. Permanent **misty twilight with drizzle**.
- **Map layers:** `surface` and `catacombs` (Phase 5). The camera switches layers through stairs or a trapdoor.
- **Expansion plots** (bought with Kibble) extend the surface along the edges.

---

## 2. Decrees and institutions (real tasks)

Each task category belongs to an **institution**, which is the building its lot grows into.
Buildings go through stages: **shack → kennel → chapel → dog-palace**.

| Category           | Institution                   | Resident monk                         | Decree flavor                                                                       |
| ------------------ | ----------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------- |
| Finance / taxes    | **The Tithe Scriptorium**     | Brother Ledger (ink-stained, anxious) | "The Annual Reckoning of Coin shall be Filed, lest His Wrinkledness be Displeased." |
| Art / creative     | **The Stained Glass Atelier** | Sister Pigment                        | "A Mural of Great Glory shall Adorn the Wall."                                      |
| Garden / outdoor   | **The Herb Cloister**         | Brother Mulch                         | "Let Posts be Raised, that the Beans may Ascend."                                   |
| Car / vehicles     | **The Chariot Stables**       | Brother Axle                          | "The Chariot's Black Blood shall be Renewed."                                       |
| Home repair        | **The Masonry Guild**         | Sister Mortar                         | "The Crack in the Eastern Wall offends the Lord."                                   |
| Health / self-care | **The Nap Chapel**            | Brother Pillow                        | "The Caretaker shall Rest. It is Commanded."                                        |
| Other              | **The Miscellany Reliquary**  | Sister Odds                           | "A Matter of Uncertain Nature requires Attention."                                  |

**Decree presentation:** your task title stays exactly as written. The flavor line, wax seal and
monk come from the theme. Steps become "Articles of the Decree", resources become "Holy Texts".

**On completion:**
- The building goes up a stage (or straight to the top stage for a task with no steps), with a short ceremony: a choir chant, a monk bowing, confetti petals.
- A **new pug arrives**, with a generated title: *"Sir Snorbert the Damp, Keeper of the Lesser Cushion."*
- Kibble is paid out, and Devotion is added.

**Recurring Decrees** (oil change, taxes): as the next due date nears, the building gets mossy
and cobwebbed and its monk starts pacing. Renewing it restores the building, and a streak plaque
is hung by the door ("Thrice Renewed").

---

## 3. Pilgrimages (in-game set pieces)

Each one unlocks at a **Devotion** threshold, is played in-game (2–5 minutes), and **permanently
changes the island**. The player can take them in any order once they're unlocked.

### 3.1 The Great Snore Synchronization — unlocks at Devotion ≥ 5 (tunable)
- **Play:** take the antique brass microphone from the belfry, walk the sleeping quarters, and hold the mic next to sleeping pugs to collect snore samples (each pug has a pitch). Then use the **resonance organ**: three to four drag-sliders for the dampening baffles, keeping the combined waveform under the "glass crack" line while the stained glass trembles.
- **Unlock:** the **Dormitory Catacombs** layer, with heated sleeping crypts and **rare nocturnal pugs** (a new pug pool that only appears there).

### 3.2 The Sacred Treat Tribute — unlocks at Devotion ≥ 12
- **Play:** Lord Chonk (huge, ancient, immobile) is lying on the bridge. Forage three ingredients across the Wild Cliffs (gull-guarded parsnip, cliff honey, crunchy seaweed), then use the **tribute oven**: hit a temperature band and a crunchiness band at the same time (two dials, timing-based). If you get it wrong, Lord Chonk sighs theatrically and you try again.
- **Unlock:** Lord Chonk waddles aside and the **Forbidden Apple Orchard** opens. The **lighting** changes from misty twilight to a **golden autumn afternoon**.

### 3.3 The Tail-Curl Calibration — unlocks at Devotion ≥ 20
- **Play:** the courtyard weather vane reads the collective helix of the pugs' tails. Gently herd stubborn pugs into alignment circles (a soft, Sokoban-style puzzle: pugs move one tile at a time, some only when bribed, one only backwards).
- **Unlock:** the **weather engine** changes. The eternal drizzle stops and **Solar Spa Mode** begins: glowing hot springs bubble up across the map. Soaking in one grants **mobility upgrades**:
  - *Warm Paws:* walk faster.
  - *Steam Hop:* cross puddles and stepping stones (shortcuts open up).
  - *Pug Cart:* the best-friend pug pulls you (fast travel between springs).

### Later Pilgrimages (content slots)
The engine supports any number of them. Ideas: *The Wrinkle Census* (a counting puzzle that opens
the Library of Folds), *The Ear Flop Inquisition*, *The Midnight Zoomies Procession*.

---

## 4. Climate and mood

Two separate systems:

| System                  | What drives it       | Effect                                                                                                                                                                                                              |
| ----------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Climate (permanent)** | Pilgrimage unlocks   | Lighting: twilight → golden autumn. Precipitation: drizzle → clear, with springs                                                                                                                                    |
| **Mood (live)**         | Current Decree state | All on track: candles lit, pugs sploot happily. Some due soon: light fog. Overdue: a fog bank over that institution, its monk wrings their hands, its pugs sulk. Streak: rays through stained glass and a choir hum |

Mood sits on top of climate as an overlay, so the island always shows how things stand without ever punishing.

---

## 5. Tending (the fidget layer)

The caretaker walks up to something and the context button changes:
- **Belly rub / ear scratch:** friendship hearts. The monk nearby gasps at the honor.
- **Wrinkle cleaning:** a little wipe gesture on a pug with grubby folds.
- **Light candles** on shrines, which go out slowly.
- **Sweep** fallen leaves and the occasional suspicious puddle.
- **Feed treats** (from inventory).
- **Pug wants:** a speech bubble with an icon (wants a cushion, wants to be carried, has lost a squeaky relic somewhere on the map).
- **Best friends** follow the caretaker in a waddling procession.

Tending pays 1–2 Kibble per action with a **daily cap**, and **never adds Devotion**.

---

## 6. Economy

| Event                            | Kibble              | Devotion  |
| -------------------------------- | ------------------- | --------- |
| Decree Article (step) done       | 5                   | 0.5       |
| Decree done (S / M / L)          | 20 / 50 / 120       | 1 / 2 / 4 |
| Recurring Decree renewed on time | +25%                | +1        |
| Tending action                   | 1–2 (daily cap ~30) | 0         |
| Pilgrimage completed             | 100                 | 0         |

**Shop (Kibble):** pug statues, stained-glass panels, velvet cushions, candle shrines, topiary pugs,
treats (bacon wafer, cheese relic, cloud biscuit), expansion plots.

All numbers live in the theme config and can be tuned.

---

## 7. Architecture

```
src/
  app/                 # shell, providers, layout (map + Decree drawer)
  domain/              # pure TS, no React — unit tested
    decrees.ts         # task model, step/complete, recurrence rollover
    recurrence.ts
    mood.ts            # live mood from Decree state
    climate.ts         # permanent climate from unlock flags
    economy.ts         # Kibble/Devotion payouts, caps
    pilgrimages.ts     # thresholds, unlock effects
    pugs.ts            # pug generation (names, titles, traits)
  theme/
    schema.ts          # Zod schema
    abbey.theme.ts     # the Pug Sanctuary content
  store/               # Zustand slices + persistence adapter
    persistence/local.ts      # IndexedDB
    persistence/supabase.ts   # Phase 7
  ui/                  # Decree drawer, Decree editor, HUD, shop, ceremony overlay
  world/               # SVG island: layers, tiles, lots, buildings, caretaker, pugs, monks, weather/fog
  pilgrimages/         # one folder per set piece (snore/, chonk/, tailcurl/)
  assets/svg/
```

**Libraries:** Vite, React, TypeScript, Zustand, Dexie/idb-keyval, Framer Motion, date-fns, Zod,
Vitest + Testing Library, Playwright (mobile smoke tests). Optional: Howler for sound in Phase 6.

**Rendering:** one `<svg>` per layer with a viewBox camera (pan and zoom). Logical tile grid for
lots, decor and pathfinding (A*). Fog, rain and light are SVG filters or overlays with gradient
masks. Pilgrimage mini-games are React overlays that each have their own small state machine.

### Theme schema (key parts)
```ts
interface Theme {
  institutions: InstitutionDef[];   // category → building stages, monk, decree copy templates
  pugPools: { id: string; layer: LayerId; traits: string[] }[];
  pugNames: { first: string[]; epithet: string[]; office: string[] };
  pilgrimages: PilgrimageDef[];      // threshold, minigame id, unlock effects
  climates: ClimateDef[];            // lighting + precipitation presets
  decor: DecorDef[];
  expansions: ExpansionDef[];
  economy: EconomyConfig;
  copy: Record<string, string[]>;    // ceremonies, monk lines, sulking lines
}

interface PilgrimageDef {
  id: 'snore-sync' | 'treat-tribute' | 'tail-curl' | string;
  devotionRequired: number;
  minigame: string;
  effects: (
    | { type: 'unlockLayer'; layer: LayerId }
    | { type: 'unlockArea'; area: string }
    | { type: 'removeBlocker'; blocker: string }
    | { type: 'setClimate'; lighting?: string; precipitation?: string }
    | { type: 'spawnFeatures'; feature: 'hotSpring'; count: number }
    | { type: 'addPugPool'; pool: string }
    | { type: 'grantAbility'; ability: string }
  )[];
}
```

### Domain model
```ts
interface Decree {                       // = real task
  id: string; title: string; categoryId: string;
  effort: 'S' | 'M' | 'L'; due?: string;
  recurrence?: { every: number; unit: 'day' | 'week' | 'month' | 'year' };
  steps: { id: string; title: string; done: boolean }[];
  resources: { id: string; label: string; url?: string; note?: string }[];
  notes?: string; status: 'active' | 'done' | 'archived';
  completions: string[]; lotId: string; createdAt: string; updatedAt: string;
}

interface SanctuaryState {
  kibble: number; devotion: number;
  pilgrimages: Record<string, 'locked' | 'available' | 'done'>;
  unlockedLayers: LayerId[]; unlockedAreas: string[]; blockersRemoved: string[];
  climate: { lighting: string; precipitation: string };
  abilities: string[];
  lots: { id: string; decreeId: string; pos: [number, number]; stage: number }[];
  pugs: { id: string; name: string; title: string; poolId: string; homeLotId?: string;
          friendship: number; want?: string }[];
  decor: { id: string; decorId: string; layer: LayerId; pos: [number, number] }[];
  shrines: { id: string; litAt?: string }[];
  caretaker: { layer: LayerId; pos: [number, number] };
  tendingToday: { date: string; kibble: number };
}
```

---

## Phases

Each phase ends in something you can deploy and use.

### Phase 1 — Decree core (task manager, no game yet)
- Vite + React + TS scaffold, lint/format, Vitest.
- Decree model, store, IndexedDB persistence, JSON export/import.
- CSV import from a spreadsheet (format below; template at `templates/decrees.csv`).
- Decree list grouped as Overdue / Due Soon / Later / Recurring, with an institution filter.
- Decree editor sheet: title, institution (category), effort, due date, recurrence, Articles (steps), Holy Texts (resources), notes.
- Completing a step or Decree, with recurrence rollover.
- Light theming: parchment cards, wax-seal checkbox, a generated flavor line per Decree.

**Done when:** your real tasks are entered on a phone, survive a reload, and the copy already makes you smile.

#### CSV import format
One row per Decree. Fill it in Google Sheets/Excel, then export as CSV. Only `title` is required.

| Column         | Values                                                                         | Maps to                    |
| -------------- | ------------------------------------------------------------------------------ | -------------------------- |
| `title`        | Free text, kept exactly as written                                             | `title`                    |
| `category`     | `finance`, `art`, `garden`, `car`, `home`, `health`, `other` (blank → `other`) | `categoryId`               |
| `effort`       | `S`, `M`, `L` (blank → `M`)                                                    | `effort`                   |
| `due`          | `YYYY-MM-DD`, or blank                                                         | `due`                      |
| `repeat_every` | Whole number, or blank for one-off                                             | `recurrence.every`         |
| `repeat_unit`  | `day`, `week`, `month`, `year` (required if `repeat_every` is set)             | `recurrence.unit`          |
| `steps`        | Items separated by ` \| `, in order                                            | `steps[]` (all not done)   |
| `resources`    | Items separated by ` \| `; each is a label, optionally ending in a URL         | `resources[].label`, `url` |
| `notes`        | Free text                                                                      | `notes`                    |

Import creates new Decrees (status `active`, fresh ids) and reports any rows it skipped and why.

### Phase 2 — The island shell
- SVG surface map: cottage, courtyard, sleeping quarters, cliff path, bridge with a placeholder Lord Chonk, and fog.
- Tile grid, viewBox camera (drag/pinch/wheel), A* tap-to-walk, and WASD on desktop.
- Institution lots auto-placed per Decree, with building stages (shack → dog-palace), using simple placeholder SVG.
- Tap a lot → its Decree opens in the drawer.
- Layout: the island fills the screen, with the Decree drawer as a bottom sheet (mobile) or side panel (desktop).

**Done when:** every Decree has a visible institution that grows as you finish steps, and you can walk there.

### Phase 3 — Ceremony, pugs and mood
- Completion ceremony (chant, bow, petals), Kibble and Devotion payout, HUD.
- Pug generation (name, epithet, office) and arrival animation; pugs idle, sploot and wander near home.
- Monks: one per institution, with idle lines from the copy pool.
- Live mood overlay: candles, fog banks over overdue institutions, sulking pugs, stained-glass rays on streaks.
- Recurring Decrees: moss and cobwebs as they come due, a pacing monk, and a streak plaque on renewal.

**Done when:** how your real-world tasks stand can be read from the island at a glance, and finishing one feels like a coronation.

### Phase 4 — Tending
- Context action system (proximity-based button).
- Belly rubs, ear scratches, wrinkle wiping, candle lighting, sweeping, treat feeding.
- Pug wants and lost relics; friendship hearts; a best-friend procession.
- Daily Kibble cap for tending.

**Done when:** 2–5 relaxed minutes of fidgeting around your Decree list feels good.

### Phase 5 — Pilgrimages and world unlocks
- Pilgrimage framework: Devotion thresholds, an "available" notice (a monk runs up with a scroll), an effects runner, and save state.
- Multi-layer map support (surface and catacombs) and blocker removal.
- Climate system: lighting presets and precipitation, with transitions.
- **3.1 Snore Synchronization** → Catacombs layer, nocturnal pug pool.
- **3.2 Sacred Treat Tribute** → Lord Chonk moves, Orchard opens, golden autumn lighting.
- **3.3 Tail-Curl Calibration** → drizzle stops, hot springs appear, mobility abilities.

**Done when:** all three Pilgrimages can be played and each one visibly and permanently changes the island.

### Phase 6 — Shop, decor and expansion
- Shop sheet (Kibble): decor, treats, expansion plots.
- Decor placement mode: grid snap, move, store back in inventory; works on any unlocked layer.
- Expansion plots extend the surface.

**Done when:** you can earn and spend Kibble, and the sanctuary feels like your own absurd empire.

### Phase 7 — Art and audio polish
- Final pastel-gothic SVG art: stained glass, robed monks, Lord Chonk, building stages, fog and light.
- Sound (off by default): snorts, a choir chant, bells, rain. Haptics on mobile.
- Reduced-motion support, an accessibility pass (the Decree UI is fully usable without the game), light/dark palettes.
- Optional PWA manifest and offline cache.

### Phase 8 — Small Supabase backend
- Magic-link auth; tables `decrees`, `sanctuary_state` (JSONB), `completions` (append-only log); RLS per user.
- Swap in the persistence adapter, keeping IndexedDB as an offline cache.
- Sync: Decrees merge per row by `updatedAt`; sanctuary state is last-writer-wins on the snapshot.
- Migrate from local data or a JSON export.

**Done when:** a Decree finished on your phone shows up as a grander dog-palace on desktop.

### Later / maybe
- More Pilgrimages (Wrinkle Census, Ear Flop Inquisition, Midnight Zoomies Procession).
- Decree templates with built-in Holy Texts ("Renew the Chariot's Black Blood": supply list, video, every 5k mi).
- Seasonal festivals (the Feast of the Squashed Face).
- A weekly "Abbey Chronicle" illuminated-manuscript summary of what got done.
- A shared sanctuary with multiple caretakers.

---

## Open questions
1. **Devotion thresholds** (5 / 12 / 20): tune these to how many real tasks she's likely to finish per week, so the first Pilgrimage arrives within roughly the first week.
2. **Institution list:** do these categories match her real tasks, or should some be renamed or added?
3. **Archived/deleted Decrees:** proposal: the building stays as a "Relic Ruin" landmark unless removed.
4. **Pilgrimage replay:** one-time only, or replayable from a Pilgrim's Journal for fun (no further rewards)?
