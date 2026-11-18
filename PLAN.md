# Hearthollow — Critter Village Task Game

> Working title. A cozy, mobile-friendly SPA where real-world home and creative tasks
> (taxes, a mural, garden posts, an oil change) become lots in a little critter village.
> You wander and tend the village as a fidget while looking at your list. Finishing real
> tasks grows buildings, earns coins, brings in critters, and brightens the weather.

---

## 0. Decisions so far

| Area          | Decision                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------- |
| Metaphor      | Critter village: each task is a lot, and each finished task is a building with a resident critter |
| Player        | A fixed **caretaker/mayor** character who walks the village                                       |
| Play          | Tend and care (water, sweep, treats, critter wants) plus free wandering                           |
| Rewards       | Permanent pieces, world mood/weather shifts, and coins                                            |
| Coins buy     | Decor to place, critter treats, village expansion                                                 |
| Task data     | Resources/links, steps/subtasks, recurring schedule, due dates, effort                            |
| Visual style  | Soft vector / storybook SVG, rendered through the DOM                                             |
| Stack         | **React** SPA, mobile-first, self-hosted                                                          |
| Data          | **Local-only first** (IndexedDB), with a **small Supabase backend** in a later phase              |
| Theme content | **To be supplied from a separate thread.** The engine must be theme-driven (see §2)               |

### Design principles
- **The list comes first, the game second.** The task UI is always one tap away and readable over the village.
- **Gentle, never punishing.** Neglect makes things scruffy or rainy. Nothing is destroyed or lost.
- **Real-world truth.** Only real task completions grow the village. Tending gives small rewards; tasks give large ones.
- **Theme is data.** Categories, buildings, critters, decor and copy come from a theme config, not code.

---

## 1. Architecture overview

```
src/
  app/                 # routes/shell, providers
  domain/              # pure TS: task model, recurrence, mood, rewards (no React)
    tasks.ts
    recurrence.ts
    mood.ts
    rewards.ts
    village.ts
  theme/               # theme loader + schema; default placeholder theme
    schema.ts
    default.theme.ts
  store/               # Zustand store slices + persistence adapter
    persistence/
      local.ts         # IndexedDB (idb-keyval or Dexie)
      supabase.ts      # Phase 7
  ui/                  # task drawer, forms, sheets, HUD
  world/               # SVG village: grid, lots, buildings, caretaker, critters, weather
  assets/              # SVG parts (from the theme)
```

**Suggested libraries** (all optional, kept light):
- Vite + React + TypeScript
- Zustand (store) + `persist` with a custom IndexedDB storage
- Dexie or idb-keyval (IndexedDB)
- Framer Motion (UI and SVG transitions)
- date-fns (due dates and recurrence math)
- Zod (theme and import validation)
- Vitest + Testing Library; Playwright for one or two mobile smoke tests

**Rendering approach:** one `<svg>` world with a `viewBox` camera (pan and zoom by changing the
viewBox). Lots and decor sit on a logical tile grid. Building stages, critters and weather are
SVG `<g>` components. The HTML UI (drawer, HUD, sheets) is layered on top.

---

## 2. Theme contract (placeholder until the theme thread lands)

The engine reads everything flavorful from a `Theme` object. Phase 1 ships a **placeholder
theme** (simple shapes and emoji) so mechanics can be built before the real theme arrives.

```ts
interface Theme {
  id: string;
  name: string;
  palette: Record<string, string>;            // CSS tokens, including dark mode
  categories: CategoryDef[];                  // taxes, art, garden, car, home...
  critters: CritterDef[];
  decor: DecorDef[];
  areas: AreaDef[];                           // expansions: pond, forest edge, hill
  weather: Record<MoodLevel, WeatherDef>;
  copy: Record<string, string>;               // flavor text, critter lines
}

interface CategoryDef {
  id: string; label: string; icon: string;
  building: { stages: SvgPartRef[]; dusty: SvgPartRef; footprint: [w: number, h: number] };
  critterPool: string[];                      // critter ids that may move in
  areaId?: string;                            // required area, if any
}

interface CritterDef {
  id: string; name: string; sprite: SvgPartRef;
  wants: WantDef[];                           // e.g. treat, pat, find lost item
  lines: { happy: string[]; waiting: string[] };
}

interface DecorDef { id: string; label: string; cost: number; sprite: SvgPartRef; footprint: [number, number] }
interface AreaDef  { id: string; label: string; cost: number; bounds: Rect; unlocksCategories?: string[] }
```

**Open item:** map the theme thread's structure onto this contract. Adjust the schema rather than
hard-code anything.

---

## 3. Core domain model

```ts
type Effort = 'S' | 'M' | 'L';
type TaskStatus = 'active' | 'done' | 'archived';

interface Task {
  id: string;
  title: string;
  categoryId: string;
  effort: Effort;
  due?: string;                                   // ISO date
  recurrence?: { every: number; unit: 'day' | 'week' | 'month' | 'year' };
  steps: { id: string; title: string; done: boolean }[];
  resources: { id: string; label: string; url?: string; note?: string }[];
  notes?: string;
  status: TaskStatus;
  completions: string[];                          // ISO timestamps (history and streaks)
  lotId: string;
  createdAt: string; updatedAt: string;
}

interface Lot {
  id: string; taskId: string; pos: [x: number, y: number];
  stage: number;                                  // derived from steps done / effort
  critterId?: string;                             // resident after the first completion
  condition: 'fresh' | 'dusty' | 'overgrown';     // derived for recurring tasks
}

interface VillageState {
  wallet: number;
  decor: { id: string; decorId: string; pos: [number, number] }[];
  unlockedAreas: string[];
  plants: { id: string; pos: [number, number]; lastWatered: string }[];
  litter: { id: string; pos: [number, number] }[];
  critters: { id: string; critterId: string; friendship: number; currentWant?: string }[];
  caretaker: { pos: [number, number]; outfit?: string };
}
```

**Derived values (pure functions, unit tested):**
- `buildingStage(task)`: the share of steps done mapped to the number of stages. A task with no steps is 0 → full on completion.
- `dueState(task, now)`: `ok | soon | due | overdue`.
- `lotCondition(task, now)`: a recurring task drifts to dusty, then overgrown, as the next due date nears or passes.
- `villageMood(tasks, now)`: `sunny | cloudy | drizzle | golden` (golden = recent streak).
- `rewardFor(event)`: coin payouts for a step, a completion and a recurring renewal, scaled by effort.

**Reward table (initial, tunable):**
| Event                                | Coins                 |
| ------------------------------------ | --------------------- |
| Step completed                       | 5                     |
| Task completed (S / M / L)           | 20 / 50 / 120         |
| Recurring task renewed on time       | +25% bonus            |
| Tending action (water, sweep, treat) | 1–2, with a daily cap |

---

## Phases

Each phase ends in something you can deploy and use.

### Phase 1 — Task core (no game yet)
**Goal:** a task manager that is useful on its own and works well on a phone.
- Vite + React + TS scaffold, lint/format, Vitest.
- Domain model, Zustand store, IndexedDB persistence.
- Task list: grouped by due state (overdue, soon, later, recurring), filter by category.
- Task editor sheet: title, category, effort, due date, recurrence, steps, resources (link/note), notes.
- Complete a step or task, with recurrence rollover (completion → next due date).
- JSON **export/import** (backup, and later migration to Supabase).
- Placeholder theme with categories only.

**Done when:** you can enter your real tasks (taxes, mural, garden posts, oil change) on mobile, check steps off, and they survive a reload.

### Phase 2 — Village shell
**Goal:** the world exists and reflects your tasks.
- SVG world with a tile grid and a viewBox camera (drag to pan, pinch/wheel to zoom).
- Lot placement: a new task auto-places a lot (staked sign) in a free spot; you can move it.
- Building stage rendering, driven by `buildingStage`.
- Caretaker: tap-to-walk (simple grid pathfinding, A*) and WASD/arrow keys on desktop.
- Tap a lot → its noticeboard opens the task (resources, steps).
- Layout: the village fills the screen, with the task drawer as a bottom sheet (mobile) or side panel (desktop).

**Done when:** each task has a visible lot whose building grows as you complete steps, and you can walk to it and open it.

### Phase 3 — Feedback and mood
**Goal:** finishing real tasks *feels* good, and the village shows how things stand.
- Completion moment: construction animation, coin burst, a critter moves in (from the category's pool).
- Wallet HUD.
- Mood/weather system: sunny, cloudy, drizzle (localized over overdue lots), golden hour on streaks.
- Recurring lots get dusty or overgrown as they come due; renewing restores them, with a streak plaque.
- Critter idle wandering near home; "waiting on porch" when its task is overdue.

**Done when:** your current task situation can be read from the village at a glance.

### Phase 4 — Tend and care (the fidget layer)
**Goal:** a gentle reason to linger and play around the list.
- Plants need watering (soft daily timers), with a wilted state that never dies.
- Litter, leaves and snails spawn on paths; the caretaker sweeps or relocates them.
- Critter wants: treat, pat, find a lost item hidden somewhere in the village.
- Friendship hearts; best friends follow the caretaker or leave gifts.
- Daily coin cap on tending so that real tasks stay the main income.
- Context action button (water, sweep, talk), shown when the caretaker is next to something.

**Done when:** you can spend 2–5 relaxed minutes in the village with things to do and no pressure.

### Phase 5 — Shop, decor and expansion
**Goal:** coins have meaning, and the village becomes yours.
- Shop sheet: decor, treats, area unlocks (from the theme).
- Decor placement mode: grid snapping, move and rotate, store back in inventory.
- Area expansion: reveal new map regions (pond, forest edge, hill), which may open new category lots.
- Treat inventory, used in critter interactions.

**Done when:** you can earn, spend, place and expand, and the village looks personal.

### Phase 6 — Real theme integration and polish
**Goal:** swap the placeholder for the real theme from the other thread.
- Map the supplied theme structure onto the `Theme` schema (adjust the schema as needed).
- Replace placeholder SVG parts with the real building stages, critters, decor and weather.
- Sound (optional, off by default), haptics on mobile, reduced-motion support.
- Light/dark palettes; accessibility pass (focus order, labels, contrast; the task UI is fully usable without the game).
- PWA manifest and offline cache (optional, cheap with Vite PWA).

**Done when:** it looks and feels like the intended game and is pleasant on a phone.

### Phase 7 — Small Supabase backend
**Goal:** one village across devices.
- Supabase Auth (magic link).
- Tables: `tasks`, `village_state` (JSONB snapshot), `completions` (append-only log), with RLS per user.
- Persistence adapter swap: `local.ts` → `supabase.ts`, keeping IndexedDB as an offline cache.
- Sync strategy: tasks merge per row by `updatedAt`; village state is last-writer-wins on the snapshot.
- One-time migration: import from local or a JSON export.

**Done when:** a task finished on your phone shows up as a grown building on desktop.

### Later / maybe
- Shared household village (multiple caretakers, assigned tasks).
- Seasonal events tied to the calendar (autumn leaves, winter lights).
- Task templates with built-in resources ("Change oil": supplies list, video link, every 5k mi).
- Mileage-based recurrence for car tasks.
- Weekly "village newspaper" summary of what got done.

---

## Open questions
1. **Theme structure:** coming from the other thread. Needed by Phase 6 at the latest, and helpful by Phase 3.
2. **Map shape:** free-form grid, or fixed plots along a path? (This affects lot placement in Phase 2.)
3. **Task deletion:** what happens to a building when its task is deleted or archived? (Proposal: it stays as a "memory" landmark unless you remove it.)
4. **Name:** "Hearthollow" is a placeholder.
