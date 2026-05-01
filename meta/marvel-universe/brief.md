# Marvel Universe — Research Brief

**Domain**: `marvel-universe`
**Output dir**: `meta/research/marvel-universe/`
**Created**: 2026-05-01

## Goal

To get a clear map of the Marvel universe.

The map should be a full historical view: not just a snapshot of current (2026) state, but the structural evolution — how cosmology, continuities, and major institutions have been retconned, expanded, or replaced over time.

## Inspirations & references

- Official Marvel references — Official Handbook of the Marvel Universe (OHOTMU, all editions), Marvel Encyclopedia (DK), Marvel Atlas, Marvel.com canon pages
- Fan wikis — Marvel Database / marvel.fandom.com and similar comprehensive fan-maintained references
- MCU-specific sources — MCU Wiki, Disney+ canon, official MCU companion books
- Comics scholarship & podcasts — sequential-art academia, Comic Book Herald, House to Astonish, Cerebro, Jay & Miles X-Plain the X-Men, etc.

## Constraints & scope

- Marvel IP only — no DC, Image, or other publishers
- Skip obscure alternate universes — keep only major, structurally significant realities (616, 1610, MCU/199999, key What If…? earths, 2099, Spider-Verse cornerstones)
- Stay structural — do not produce deep character biographies; characters appear only as nodes in the larger map

## Source types

For this project, source preference is: **Type 2 primary, Type 3 secondary; Type 1 not relevant** (best for tech / industry / cultural-canon research).

The three types are defined below regardless of priority. When citing a source, use the format and tag for its type. No quotas — cover whichever types the topic actually justifies, weighted by the priority above.

- **Type 1** — published research papers, peer-reviewed work, conference proceedings. Cite as: source + page + paragraph. Example: `[T1] Smith et al. 2024, p. 12, ¶3`.
- **Type 2** — official documentation, technical specs, design system docs, white papers, books, renowned domain blogs. For Marvel, this means OHOTMU, Marvel Encyclopedia (DK), Marvel.com canon entries, official companion books, Marvel Atlas, etc. Cite as: source + subsection (if any) + line/page number. Example: `[T2] OHOTMU Master Edition Vol. 4, Galactus entry, p. 12`.
- **Type 3** — Twitter/X, Hacker News, Reddit, small blogs, podcasts, conference videos, fan wikis. For Marvel, this includes Marvel Database/Fandom, podcasts, fan analysis pieces. Cite as: source + URL. Example: `[T3] Marvel Database, "Earth-1610" entry, https://...`.

Tag every inline citation with its type (`[T1]`, `[T2]`, `[T3]`) so the synthesis can spot coverage gaps relative to the stated preference.

## Conventions

- Each topic is researched by a parallel agent and saved to `meta/research/marvel-universe/{topic-slug}.md`.
- Each topic agent's primary mode is external web research — fresh sources, not training data.
- Synthesis lives at `meta/research/marvel-universe/synthesis.md` and is fully rewritten after each round.
- Rounds are appended below as `## Round 2`, `## Round 3`, etc. Earlier rounds and their topic files are never rewritten.

## Round 1

### Multiverse structure & cosmology

File: `multiverse-cosmology.md`

Map the structural architecture of the Marvel multiverse: distinctions between Multiverse, Megaverse, and Omniverse; the Earth-XXXXX designation system; planes of reality (mortal, divine, abstract); and key dimensional layers (Astral Plane, Negative Zone, Microverse, Mojoverse). Trace how this cosmology evolved historically — early "Counter-Earth" framings, the original Beyonder-era Secret Wars (1984), the Heroes Reborn pocket universe, the post-Secret Wars (2015) "All-New All-Different" multiverse reboot, and the current Multiverse Saga / Incursion model.

### Major continuities (Earth-616, Earth-1610, MCU, others)

File: `major-continuities.md`

Catalog the structurally important Marvel continuities and how they relate: Earth-616 (main), Earth-1610 (original Ultimate line + 2024 relaunch), Earth-199999 (MCU), Earth-2149 (Marvel Zombies), Earth-65 (Spider-Gwen / Ghost-Spider), Earth-928 (2099), Earth-92131 (X-Men: The Animated Series and X-Men '97), the Heroes Reborn pocket, and major What If? branch points. For each: founding year, defining differences from 616, current status (active / dormant / destroyed / merged), key books or media. Show how they sit in the multiversal map.

### Cosmic hierarchy of beings

File: `cosmic-hierarchy.md`

Map Marvel's cosmic hierarchy top-to-bottom: One-Above-All, Living Tribunal, Beyonders, Celestials, the abstract entities (Eternity, Infinity, Death, Oblivion, Galactus tier), Watchers, the Stranger, the In-Betweener, Order/Chaos. Document power tiers, jurisdictions (multiversal vs. universal vs. galactic), and historical retcons — including the Beyonder/Beyonders distinction, Galactus's role across eras, and the post-Secret Wars (2015) restructuring.

### Pantheons & god-tier groups

File: `pantheons.md`

Catalog Marvel's pantheons: Asgardians, Olympians, Heliopolitans (Egyptian), Vodu (Loa), Hindu Devas, Daevas (Persian), Annunaki, Tuatha de Danaan, Shinto Amatsu-Kami, and the umbrella Council of Godheads / Council of Skyfathers. For each: home dimension, key members, current and historical status (Ragnaroks, dimensional shifts), and inter-pantheon relationships. Reference the Thor: Ages of Thunder eras and recent reshuffles in current Thor / Immortal Thor runs.

### Major teams & factions

File: `teams-factions.md`

Map the major teams and factions: Avengers (and roster eras — original, West Coast, New, Mighty, Dark, Uncanny, current), X-Men and offshoots (X-Force, X-Factor, New Mutants, Excalibur, Krakoan Quiet Council), Fantastic Four, Guardians of the Galaxy (Original 31st-century vs. modern), Inhumans, Defenders, Illuminati, Cabal, Hellfire Club, Thunderbolts, A.I.M., Hydra, S.H.I.E.L.D., S.W.O.R.D. For each: formation, era-defining arcs, current and historical status.

### Major species & races

File: `species-races.md`

Catalog the structurally significant sentient species in Marvel: Homo sapiens superior (Mutants), Inhumans, Eternals, Deviants, Skrulls, Kree, Shi'ar, Brood, Asgardians, Atlanteans, Lemurians, Symbiotes, Watchers, Elders of the Universe. Cover origins (often Celestial-engineered), home worlds or dimensions, defining traits, and key historical events (Kree-Skrull War, Inhumans vs. X-Men, Annihilation, Empyre, Judgment Day).

### Earth-616 geography

File: `earth-616-geography.md`

Map the fictional and altered geography of Earth-616: sovereign nations (Wakanda, Latveria, Genosha, Symkaria, Madripoor, Atlantis, Lemuria, Sokovia, Carbombya), hidden civilizations (Savage Land, K'un-Lun, Attilan in its various locations, Krakoa, Arakko, Pangea), and major altered cities. For each: location on Earth, sovereignty/leadership, current status, and historical role across major arcs (e.g., Genosha post-Decimation, Krakoa post-Fall of X).

### Magic & mystic dimensions

File: `magic-mystic-dimensions.md`

Map Marvel's magic system and mystic dimensions: the Sorcerer Supreme lineage from Agamotto onward, magical patron hierarchies (Vishanti, Octessence, Faltine), key mystic dimensions (Dark Dimension, Limbo — Belasco's vs. Immortus's "Otherplace", the Faltine realm, the various Hells and Hell-lords like Mephisto, Dormammu, Nightmare). Cover the structural rules of magic (white/black/grey, hermetic vs. dimensional sourcing) and major retcons.

### Key crossover events timeline

File: `crossover-events.md`

Build a chronological timeline of Marvel's major line-wide crossovers from Contest of Champions (1982) and Secret Wars I (1984) through to events current as of 2026. For each major event (Inferno, Atlantis Attacks, Infinity Gauntlet, Onslaught, Civil War, House of M, Secret Invasion, Siege, Avengers vs. X-Men, Original Sin, Secret Wars 2015, Civil War II, Empyre, King in Black, Krakoan-era events including X of Swords / Hellfire Gala / Judgment Day / Fall of X, Multiverse Saga events): year, premise, lasting impact on continuity.

### MCU phases & Multiverse Saga

File: `mcu-phases-multiverse-saga.md`

Map the MCU's structural arc: Infinity Saga (Phases 1-3), Multiverse Saga (Phases 4-6). For each phase: defining films and Disney+ series, narrative arc, key character introductions and exits, and the multiverse mechanics introduced (Sacred Timeline, branching, variants, Incursions, the TVA's role). Cover Disney+ canon integration (WandaVision, Loki, What If…?, Ms. Marvel, etc.) and current state heading into Avengers: Doomsday / Secret Wars.

### Power scaling & classification systems

File: `power-scaling-classification.md`

Document Marvel's in-universe power classification systems: the OHOTMU Power Grid (intelligence, strength, speed, durability, energy projection, fighting skill — 1-7 scale and its evolution), threat indices, the Mutant power classification (Alpha, Beta, Gamma, Delta, Epsilon, Omega — including the inconsistent history of these labels and the Krakoan-era "Omega" formalization), and cosmic tier descriptions (Skyfather-class, Abstract-class, Multiversal). Cover how these systems shifted across eras and where they break down.

### Reality-altering artifacts & MacGuffins

File: `reality-altering-artifacts.md`

Catalog reality-altering artifacts: Infinity Stones/Gems (six in 616 with their pre-2010 names, the seventh "Ego Gem" retcon, the MCU variants), Cosmic Cubes / Tesseract / Kobik, the M'Kraan Crystal, the Heart of the Universe, the Ultimate Nullifier, the Wand of Watoomb, the Eye of Agamotto, the Casket of Ancient Winters, the Reality Gems of the Ultraverse-merger era. For each: origin, capabilities, current location/status, key historical uses, and which continuity/ies it appears in.

## Round 2

### Time travel & alternate-timeline mechanics

File: `time-travel-mechanics.md`

Map Marvel's time-travel and timeline systems: the Kang/Immortus/Rama-Tut/Iron Lad continuum, the Time Variance Authority (comics introduction in *Thor* vol. 1 #281, 1979 vs. MCU *Loki*), Cable's askani/2YL futures, *Days of Future Past* (Earth-811) and the proliferation of branched dystopian futures, Sacred Timeline mechanics, and the structural distinction between branching, parallel-reality, and single-timeline models across eras.

### Cosmic empires & galactic geopolitics

File: `cosmic-empires.md`

Map Marvel's interstellar empires structurally: Kree Stellar Empire (Hala, Supreme Intelligence), Skrull Empire and post-Annihilation diaspora, Shi'ar Imperium and Imperial Guard, Spartoi, Brood hives, Badoon, Skrull-Kree-Shi'ar treaty geometry, the Galactic Council (formed during *Infinity*), Annihilation Wave aftermath, *Empyre* unified Kree-Skrull state under Hulkling. Cover home worlds, ruling structures, and the long-arc consolidation pattern from 1971 *Kree-Skrull War* to 2020 *Empyre*.

### Knull / King in Black symbiote cosmology

File: `knull-symbiote-cosmology.md`

Trace the 2018–21 retcon (*Venom* vol. 4, Cates/Stegman; *King in Black* 2020–21) that reframed symbiotes as primordial Celestial-era substance and Knull as a god of the void predating the light, with Grendel and All-Black the Necrosword as its first weapon. Reconcile with Hickman's First Firmament cosmology, the post-2015 Eighth Cosmos, and prior Klyntar-as-collective-symbiote-paradise framing. Cover Eddie Brock's elevation to Captain Universe / King in Black successor.

### Major weapons & artifacts of cosmic war

File: `weapons-cosmic-war.md`

Catalog structurally significant weapons: Mjolnir lineage (original, Jane Foster's, Mjolnir as Beta Ray Bill's, Throg), Stormbreaker, Jarnbjorn (Aaron's Thanos-killer), the Necrosword / All-Black and its God-Butcher-era wielders, Captain America's shield (vibranium/proto-adamantium variants, Sam Wilson's, U.S. Agent's), Mandarin's Ten Rings (comics origin vs. MCU), the Ebony Blade (Black Knight lineage), the Twilight Sword, Odinsword/Ragnarok, Quantum Bands, Nega-Bands, Power Cosmic conduits, Adamantium and Vibranium as material-class artifacts.

### AI & cybernetic entities

File: `ai-cybernetic-entities.md`

Map Marvel's silicon-life lineage: Ultron and his progeny (Vision, Jocasta, Victor Mancha), Machine Man (Aaron Stack), Nimrod and Sentinel evolution (Mk-I through Sentinel City and Krakoan-era Nimrod), the Phalanx (Technarchy connection), Arnim Zola's body-mind transfer, Doombots, Iron Legion, Awesome Android, the Sentry's Void as informational entity, and the Krakoan-era Orchis/Mother Mold/Sentinel resurgence. Treat as a structural category with its own evolutionary axis.

### Demographics & population structures

File: `demographics-populations.md`

Tabulate the headcount axis that recurs across the corpus: mutant population (pre-M-Day estimates, post-*House of M* "198", Krakoan-era resurrection-driven growth, *Fall of X* numbers), Inhuman population (Attilan's small enclave vs. post-Terrigen-cloud NuHumans wave, Inhumans vs. X-Men *IvX* 2017), Eternal numbers and the Excluded, Asgardian census across Ragnaroks, Kree-Skrull diaspora numbers post-Annihilation/Empyre, civilian super-population (Initiative-era "fifty state initiative" numbers), and the heroic roster headcount across eras.

### Fictional corporations & industrial cosmology

File: `corporations-industry.md`

Catalog Marvel's industrial layer: Stark Industries (Iron Man, Resilient, the post-*Civil War II* arcs), Roxxon Energy Corporation (Brand Corporation roots, Dario Agger), Oscorp Industries (Norman Osborn), Pym Technologies, Hammer Industries (Justin Hammer, *Iron Man 2*), Fisk Industries, Damage Control (Marvel/Disney+ canon), Rand Corporation, Alchemax (2099 founder of the future), Worthington Industries, Cross Technological Enterprises (Yellowjacket/Ant-Man), and the SHIELD industrial complex. Cover ownership transitions and how they map to characters/arcs.
