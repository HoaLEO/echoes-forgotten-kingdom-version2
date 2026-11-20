# World Architecture — Echoes of the Forgotten Kingdom (Version 2, Unity)

| | |
|---|---|
| Status | **Draft v0.1 — awaiting owner approval. No game code is written until this is approved.** |
| Engine | Unity (exact LTS version pinned in Milestone 0) |
| Scope | The permanent foundation of the world of Elvareth. Not a content plan. |
| Verification | Nothing in this document has been built or tested in Unity yet. It is a design. |

---

## 1. Purpose

Version 2 is a **new game** built in Unity: a new world and a new story. The story and lore of
version 1 (Three.js, single `index.html`) are not carried over. What carries over are the lessons
learned — above all that in version 1 every new region required edits to core code — and selected
gameplay concepts where they fit the new story (section 13).

Version 2 is built on a foundation that is designed once and then left alone, so the world can keep
growing for years.

This document separates two things and keeps them separate everywhere — in folders, in assemblies,
in save data and in how we work:

- **The World Architecture** — the foundation. Hierarchy, IDs, data model, loading, save format,
  the Living Sanctuary system. Changes only when genuinely necessary, and never in a way that
  breaks existing content or saves.
- **The World Content** — domains, regions, locations, quests, characters, puzzles, lore. Grows
  continuously. Adding content must not require changing the architecture.

## 2. Principles

These are binding. A design that violates one of them needs an explicit, written exception.

1. **World scalability.** No built-in limit on the number of Domains, Regional Hubs, Locations or
   Quests. Nothing in core code is sized, indexed or enumerated by content count.
2. **Modular architecture.** Every Domain, gameplay system and asset group can be developed, loaded
   and shipped independently.
3. **Data-driven world.** A new location is added mainly by creating data and content, not by
   editing core code.
4. **Hierarchical navigation.** The player moves from the world, to a domain, to a region, to a
   place. The same hierarchy organises the project.
5. **Progressive discovery.** The player only ever sees what they have discovered, the gates they
   have opened and the clues that lead onward. The full world is never shown at once.
6. **Backward compatibility.** Old content and old saves keep working when the game grows.

What "data-driven" honestly means for a 3D game: a handcrafted cave still needs handcrafted
geometry, lighting and sometimes location-specific scripts. The rule is not "no code in content".
The rule is: **content may contain its own code, but it plugs into the core through fixed
interfaces and never edits the core.** If a location needs a capability the core lacks (swimming,
for example), that capability is added to the core as a general system — not hidden inside the
location.

## 3. The Eternal Nexus — world hierarchy

The Eternal Nexus is the name for the whole connected structure of Elvareth. It has four levels.

```
ELVARETH  (the Eternal Nexus)
│
├── Level 1 · THE GRAND SANCTUARY          central hub — exactly one
│
└── Level 2 · GREAT DOMAINS                independent realms, reached through great gates
      │        e.g. The Great Ascent, and further realms defined by the new story
      │
      └── Level 3 · REGIONAL SANCTUARIES   hubs inside a domain: a city, temple, camp, waystation
            │
            └── Level 4 · LOCATIONS        the actual playable maps: forest, cave, ruin, vault…
```

| Level | Name | Role | Is it a playable space? |
|---|---|---|---|
| 1 | Grand Sanctuary | Where the journey is anchored. Characters, world map, domain gates, the World Tree. | Yes — one scene that grows (section 4). |
| 2 | Great Domain | A realm with its own theme, story arc and asset set. | No — it is a grouping. You arrive in one of its hubs. |
| 3 | Regional Sanctuary | A hub inside a domain. Connects to several locations. | Yes — a small safe scene. |
| 4 | Location | A map the player explores. | Yes. |

Capacity example (illustration only, not a target or a limit): 20 domains × 10 hubs × 10 locations
= 2,000 locations. The hierarchy makes 2,000 or 10,000 locations *organisable and navigable*. It
does **not** reduce what each location costs to build, store and optimise (section 12).

### 3.1 The hierarchy is for organisation; travel is a graph

The tree above answers "where does this belong?". It does not mean the player must return to a hub
to go anywhere. Locations also connect **directly** to each other through `Connection` records
(a mountain path that descends into a cave, a secret tunnel between two domains). So:

- **Tree** (parent/child) → project organisation, world map drill-down, loading groups.
- **Graph** (connections) → how the player actually walks through the world.

Version 1 showed that the feeling of one continuous journey — walking from a mountain straight
down into a cave — is worth more than any menu, and it must be kept.

## 4. The Living Sanctuary

The Grand Sanctuary is not a menu and not a static building. It is an **ancient entity that
grows** in response to what the player does in the world. The hub is part of the story.

### 4.1 Kinds of growth

| Kind | What the player sees | Typical trigger |
|---|---|---|
| **Wing** | A dormant part of the Sanctuary awakens — lights, sound, new rooms, new characters. | A domain is completed. |
| **Gate** | An ancient gate materialises or unseals. | An Ancient Key is found. |
| **Corridor** | A hidden passage opens. | A set of history fragments is assembled. |
| **Tier** | An architectural level that did not exist before appears. | The world has grown enough to need it (see 4.4). |
| **World Tree branch** | The World Tree grows a branch reaching toward a new realm. | A new domain becomes known. |

### 4.2 Growth nodes

Every possible change to the Sanctuary is one data record, a **Sanctuary Growth Node**:

- a stable ID,
- a kind (table above),
- a **condition** (section 6.3) that awakens it,
- a **presentation**: which additive scene or prefab to enable, at which anchor, and which
  awakening sequence to play,
- optional **effects** when it awakens (reveal a domain, grant an item, set a flag).

Adding a new piece of Sanctuary growth is therefore data plus art. The Sanctuary's core scene and
code do not change.

### 4.3 Three states, so the Sanctuary hints before it reveals

| State | Meaning | Example |
|---|---|---|
| **Dormant** | Not visible at all. | A blank wall. |
| **Stirring** | A visible hint that something is here but not yet open. | A sealed arch with a keyhole; a bud on the World Tree. |
| **Awakened** | Fully present and usable. | The gate stands open; the branch has grown. |

"Stirring" is how progressive discovery works *inside the world* instead of through UI: the player
sees a sealed arch long before they hold the key for it.

**Rule (approved):** growth that is earned while the player is away is queued and performed when
they next enter the Sanctuary, so they witness it happen rather than finding it already done.

### 4.4 How one building can hold an unlimited world

A room cannot physically hold 2,000 doors. The hierarchy solves this:

- The Grand Sanctuary holds only **domain gates** — tens, not thousands.
- Each domain's Regional Sanctuaries hold the portals to their own locations.
- When the number of domains outgrows the gates a tier can hold, **a new tier awakens**. Tiers and
  World Tree branches place their gates at anchor slots generated by a layout rule, not placed by
  hand per domain.

So the fantasy ("floors that never existed begin to appear") and the engineering ("no fixed gate
count") are the same mechanism.

### 4.5 Art direction (from the owner's reference images — direction only)

The nine reference images are mood and composition references. They are not assets, must not be
embedded in the game, and are not stored in this repository.

- A colossal **World Tree** on a floating island above a sea of clouds; glowing runes spiralling
  around the trunk; water falling from the island's edges; smaller floating rocks and standing
  stones around it.
- A **cosmic sky**: galaxies, nebulae, slow-moving planets — the Sanctuary sits outside ordinary
  geography.
- **Monumental, library-like architecture**: tiered galleries, spiral staircases, warm gold light
  against deep blue space.
- **Great circular arches** that open onto sky and other worlds — the visual language for gates.
- A **circular plaza** with the tree at its centre on a carved base, ringed by glowing rune
  circles in the floor; statues and lanterns around the rim.
- A **radiant sun-disc hall**: a single overwhelming light source at the end of an axis.

Elvareth stays an original world. Motifs that resemble existing mythologies (a world tree linking
realms) are used as archetypes only; names, symbols and stories are our own.

## 5. Navigation and discovery

### 5.1 Discovery states

Every Domain, Regional Sanctuary and Location has one discovery state per save. It only ever moves
forward.

`Unknown → Rumored → Revealed → Visited → Completed`

| State | The player… | World map shows |
|---|---|---|
| Unknown | has no idea it exists | nothing |
| Rumored | has heard of it or found a clue | an unnamed or vaguely named hint |
| Revealed | knows where it is and how to reach it | name and position |
| Visited | has been there | full entry |
| Completed | has finished its main objective | full entry, marked complete |

### 5.2 World map

The world map (a physical object in the Sanctuary, also available as UI) drills down
World → Domain → Regional Sanctuary → Location and lists only entries that are at least Rumored.
With 10 locations discovered the map shows 10, whether the game ships 40 or 4,000.

### 5.3 Guidance

Version 1 playtesting found that players could not find entrances to new regions. Guidance is
therefore an architecture feature, not polish: every Location declares its entrances, and the
compass/objective system can always answer "where is the next known step?" from data.

## 6. Data model

All world structure is data, authored as Unity **ScriptableObject** assets. Runtime code reads
them; it never hard-codes a specific domain, location or gate.

### 6.1 Definitions

| Definition | Key fields |
|---|---|
| `WorldDefinition` | id, starting location, list of root growth nodes |
| `DomainDefinition` | id, display name key, theme, reveal condition, gate condition, completion condition, World Tree branch node |
| `RegionalHubDefinition` | id, parent domain, scene address, reveal condition |
| `LocationDefinition` | id, parent domain, parent hub, scene address (and sub-scene addresses), entrances/spawn points, reveal condition, completion condition, quality budget tier |
| `ConnectionDefinition` | id, from (location + exit), to (location + spawn point), condition, one-way or two-way |
| `ItemDefinition` | id, kind (relic, Ancient Key, tool, collectible), display data |
| `LoreFragmentDefinition` | id, set id, text key — the "pieces of history" |
| `QuestDefinition` | id, steps; each step has a condition and effects |
| `SanctuaryGrowthNode` | id, kind, condition, presentation, effects (section 4.2) |

Definitions refer to each other **by ID**, and to scenes and heavy assets **by address**, never by
direct asset reference from core code. This keeps domains independently loadable.

### 6.2 World state: one store of facts

All persistent progress is stored as **facts**: a map from a stable ID to an integer.

| Fact | Example key | Value |
|---|---|---|
| Flag | `flag.sanctuary.first_gate_opened` | 0 or 1 |
| Discovery state | `loc.great_ascent.north_face` | 0–4 (section 5.1) |
| Inventory | `item.key.ancient_01` | count |
| Sanctuary growth | `sanctuary.gate.great_ascent` | 0 dormant, 1 stirring, 2 awakened |
| Quest progress | `quest.main.prologue` | step index |
| Interactable state | `obj.great_ascent.base_camp.rope_winch` | state value |

(All IDs in this document are illustrations; real IDs are assigned when content is designed.)

One uniform store means conditions, saves, migrations and debugging tools all work on the same
simple thing.

### 6.3 Conditions and effects

Every unlock in the game is a **condition** written as data, evaluated against the facts:

- combinators: `All`, `Any`, `Not`
- tests: fact equals / at least, has item, discovery state at least, domain completed, lore set
  complete

Every consequence is an **effect** written as data: set fact, grant item, advance discovery state,
awaken growth node, start quest.

The list of condition and effect *types* is small and lives in the core. It is **append-only**: new
types may be added, existing types never change meaning. That is what keeps old content valid.

Rule: no `if (chapter == 3)` or `if (location == "cavern")` anywhere in core code. If the core
needs to branch on content, that is a missing condition type.

## 7. Stable IDs

IDs are the backbone of backward compatibility.

- Format: lowercase segments of `a–z`, `0–9`, `_`, separated by dots.
  `domain.great_ascent` · `hub.great_ascent.base_camp` ·
  `loc.great_ascent.north_face` · `item.key.ancient_01` · `lore.sanctuary.founding_03`
- An ID is **assigned once and never changed, never reused** — even if the display name, the parent
  hub or the scene changes.
- The prefix is a naming convention for humans. Code treats an ID as an opaque string and never
  parses meaning out of it.
- Removed content keeps its ID, marked deprecated. If something truly must be replaced, an
  **ID redirect table** maps old → new; saves are redirected on load.
- An editor validator (also run in CI) fails the build on duplicate, malformed or dangling IDs.

Version 1 identified interactions by order of registration, which made every insertion a risk to
saves. Version 2 removes that class of problem.

## 8. Scenes and loading

```
Bootstrap scene (always loaded)
  └─ core services: world state, save, registry, scene loader, input, audio, UI root

+ exactly one "place" at a time, loaded additively:
     the Grand Sanctuary   — base scene + awakened growth scenes
  or a Regional Sanctuary
  or a Location            — optionally split into sub-scenes for very large maps
```

- Scenes and heavy assets are loaded through **Addressables**, by address taken from the
  definitions.
- **One place in memory at a time.** Memory use depends on the place the player is in, not on how
  large the world has become. This is what lets the world grow without the game getting heavier.
- Very large locations (a vast cave system, for example) are split into sub-scenes that load and unload by
  position — the Unity equivalent of the region-visibility system that fixed performance in
  version 1.
- Transitions follow one path for every gate, portal and connection: save facts → fade → unload →
  load → place player at spawn point → fade in.
- **World index.** An editor tool generates a lightweight index (ID → parent, address, name key).
  The runtime loads this index and only loads full definitions for the domain the player is in, so
  start-up time does not grow with the size of the world.

## 9. Save system

- Format: versioned JSON. Human-readable, diffable, easy to migrate.
- Content: `schemaVersion`, game version, timestamps, the player's place and spawn point, and the
  **facts** (section 6.2).
- **Forward-tolerant:** facts with IDs the running build does not recognise are preserved, never
  dropped.
- **Migrations:** each schema change ships a migration step (`n → n+1`). Loading runs the chain.
  A migration never silently deletes progress.
- **Safe writes:** write to a temporary file, then replace; keep a rolling backup of the previous
  save.
- Version 1 browser saves are not importable. Compatibility is guaranteed from the first Unity
  release onward.

## 10. Project layout

```
Assets/
├── _Core/                         THE WORLD ARCHITECTURE — rarely changes
│   ├── Runtime/
│   │   ├── World/                 definitions, registry, world index, IDs
│   │   ├── State/                 facts, conditions, effects
│   │   ├── Save/                  save format, migrations
│   │   ├── Loading/               scene loader, transitions
│   │   ├── Sanctuary/             growth system
│   │   ├── Discovery/             discovery states, map, guidance
│   │   ├── Interaction/           interactables, pickups, doors
│   │   ├── Player/                first-person controller, input (keyboard/mouse + gamepad)
│   │   └── UI/  Audio/
│   ├── Editor/                    ID validator, world index generator, authoring tools
│   └── Tests/                     edit-mode and play-mode tests
│
├── Systems/                       optional gameplay systems, each its own assembly
│   ├── Climbing/                  controlled routes: ropes, ledges, handholds
│   ├── LightPuzzles/              crystals, mirrors, sun shafts
│   └── …
│
└── Content/                       THE WORLD CONTENT — grows continuously
    ├── Sanctuary/                 base scene, growth scenes, growth nodes
    ├── GreatAscent/               one folder per domain:
    │   ├── Data/                  definitions for this domain
    │   ├── Hubs/  Locations/      scenes
    │   ├── Art/  Audio/
    │   └── Scripts/               location-specific code, own assembly
    └── <NextDomain>/
```

Assembly dependency direction is enforced by Assembly Definitions:

`Content → Systems → _Core`  — never the other way.

`_Core` cannot reference any specific domain, so it cannot accidentally depend on one.

## 11. Adding content — the pipeline

**A new Location**

1. Create the scene from the Location template (spawn points, exits, required markers).
2. Create its `LocationDefinition`; assign a new ID; set parent hub, address, conditions.
3. Add `ConnectionDefinition`s to and from neighbouring places.
4. Build the content: geometry, lighting, interactables, puzzles.
5. Run the validator and regenerate the world index.
6. Play it from the nearest hub.

No file under `_Core/` is touched.

**A new Domain** — the same, plus a `DomainDefinition`, a domain gate growth node, a World Tree
branch growth node, and at least one Regional Sanctuary.

**A new piece of Sanctuary growth** — a `SanctuaryGrowthNode` plus its scene or prefab.

**When the core *does* change** — only when a new general capability is needed (a new condition
type, a new system). The change must be additive, must ship with tests, and must not alter the
meaning of existing data.

## 12. Scale, performance and honesty about limits

- The architecture supports an unlimited number of places. It does not make them free. Every
  location still costs design time, art, storage and optimisation. 2,000 quality maps is a
  multi-year production goal, not something the architecture produces.
- **Platforms:** PC first. The first release targets **Windows**. macOS and Linux desktop builds
  are planned later; Unity can produce them, but a signed, notarised macOS build normally needs a
  Mac. Mobile and web are not targets; nothing in the core may assume they will never come, but no
  work is spent on them now.
- Performance target: about 60 FPS at 1080p on a reasonably capable desktop GPU, with meaningful
  Low / Medium / High / Ultra presets. Weak integrated GPUs get a best-effort Low preset.
- **Development machine:** the Unity Editor runs on the owner's computer, which has a weak
  integrated GPU. Consequences: greybox first; keep Editor scenes light (sub-scenes, no huge
  always-loaded geometry); prefer realtime or cheap lighting during early milestones and schedule
  heavy light baking deliberately. Editor frame rate on this machine is not a measure of the
  game's performance on target hardware.
- Render pipeline: URP (scales down to weak hardware; HDRP would be too heavy for the development
  machine).
- Each location declares a **budget tier**; budgets are measured, not assumed. No performance claim
  is made without a real measurement on real hardware, and the hardware is named.
- Build size grows with content. Addressables groups per domain allow content to be shipped and
  updated as separate packs when the base build becomes too large.
- Input: Unity Input System with one set of gameplay actions. Bindings: keyboard and mouse, and
  gamepad. Other devices can be added later as new bindings without touching gameplay code.

## 13. Relationship to version 1

Version 2 has a **new story**, unrelated to version 1. No characters, lore, chapters, endings or
item names are carried over.

What does carry over:

| From version 1 | In version 2 |
|---|---|
| Lesson: new regions forced core code changes | Architecture vs Content separation (section 2) |
| Lesson: saves keyed by registration order were fragile | Stable IDs and versioned saves (sections 7, 9) |
| Lesson: players could not find new entrances | Discovery states and data-driven guidance (section 5) |
| Lesson: rendering everything at once killed performance | One place in memory, sub-scenes (section 8) |
| Concept: **The Great Ascent** (mountain, controlled climbing routes) | **Approved:** becomes its own Great Domain in the new world. Its name, place and story role are designed with the new story. |
| Other concepts (a vast cave inspired by real limestone caves, light-reflection puzzles, rune sequences, pressure plates) | Candidates only. Each returns only if it fits the new story, and is redesigned as new content. |

## 14. Milestones

Each milestone ends with a report: what was built, what was tested and how, and what is still
unverified. Play-testing happens in the Unity Editor on the owner's computer; anything not actually
run there is labelled untested.

| # | Milestone | Result |
|---|---|---|
| M0 | Project setup | Unity LTS version pinned, URP, Input System, Addressables, Git LFS, folder and assembly skeleton. Opens with zero errors on the owner's machine. |
| M1 | Core skeleton | IDs, definitions, facts, conditions/effects, save + migration, registry, validator. Covered by edit-mode tests. No visuals. |
| M2 | Vertical slice | Greybox Grand Sanctuary → one domain gate → one Regional Sanctuary → one Location and back. Fast travel, save, load, discovery map. |
| M3 | Living Sanctuary | Growth nodes with Dormant / Stirring / Awakened and queued awakenings; one wing, one gate, one corridor, one World Tree branch, driven only by data. |
| M4 | Proof of the principles | Add a second Location and a second Domain **without editing `_Core`**. If the core has to change, the architecture is fixed before any real content is built. |
| M5+ | Content | The new story's first domains, built as content, starting with the vertical-slice domain. |

In parallel with M0–M4 (no code): **Story bible** — the new world, its history, the Grand
Sanctuary and World Tree in the story, the first domains including the Great Ascent, and the
player's goal. Approved by the owner before M5.

M4 is the real test of this document.

## 15. Decisions and open questions

### Decided (2026-11-20)

| Topic | Decision |
|---|---|
| Story | A completely new story, unrelated to version 1. |
| The Great Ascent | Its own Great Domain, placed somewhere in the new world. |
| Fast travel | Yes — between visited Regional Sanctuaries and the Grand Sanctuary. |
| Growth while away | Queued and performed when the player returns (section 4.3). |
| Platforms | PC. Windows first; macOS and Linux desktop later. |
| Development machine | The owner's computer runs the Unity Editor. |
| "Celestial Sanctuary" name clash | Resolved by the new story — version 1 regions do not exist in version 2. |

### Open

1. **Where does the game begin?** In the Grand Sanctuary, or in a first domain where the player
   discovers the Sanctuary later? (The answer received was incomplete.)
2. **Title.** Does the game stay "Echoes of the Forgotten Kingdom" with a new story, or will the new
   story get a new title? Is the world still called Elvareth?
3. **Story bible.** Should Claude draft a first story outline for the owner to react to, or does the
   owner write the core idea first?
