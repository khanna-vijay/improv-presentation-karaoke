# 01 — Naming Convention

Every artefact in this project has a machine-parseable ID. Nothing is named by hand, nothing is
overwritten, and every ID round-trips: **filename → ID → catalog record → source seed**.

---

## 1. The three ID namespaces

| Prefix | Namespace | Example |
|---|---|---|
| `PKS` | **P**owerPoint **K**araoke **S**lide (an image asset) | `PKS-DV-WA-004` |
| `PKC` | **P**owerPoint **K**araoke **C**ontext (a show / scenario) | `PKC-COR-01` |
| `PKD` | **P**owerPoint **K**araoke **D**eck (an assembled practice run) | `PKD-20260816-1432-A7F3` |

The prefixes are distinct, so a code like `AC` can safely mean *Acronym* inside `PKS-…` and
*Academic* inside `PKC-…` without collision. Parsers switch on the prefix first.

---

## 2. Slide ID — `PKS-<TT>-<RR>-<NNN>`

```
PKS  -  DV  -  WA  -  004
 |       |      |      └── sequence, 001–999, unique within the TT×RR cell
 |       |      └───────── REGISTER  (tone / emotional key) — 2 letters
 |       └──────────────── TYPE      (slide artefact category) — 2 letters
 └──────────────────────── namespace
```

- **`TT` — Type** is *what kind of object* the slide is: a chart, a quote, a photo, a mandala.
  Ten values. See `03_Taxonomy_Slide_Types_and_Registers.md`.
- **`RR` — Register** is *what emotional key* it is played in: corporate-deadpan, whacky,
  comic, deep-thinking, spiritual-mystic, inspiring-epic. Six values.
- **`NNN`** restarts at `001` in every cell. So `PKS-DV-WA-004` and `PKS-QT-WA-004` are different
  slides and that is fine — the full ID is the key, never the number alone.

The type and register code sets are **disjoint by construction** (no 2-letter code appears in
both lists), so a malformed ID is detectable even without positional parsing.

### 2.1 Why register is in the ID and genre is not

A slide's register is intrinsic — a mandala is mystical no matter who presents it. A slide's
*genre affinity* is situational — the same mandala works for a cult orientation, a wellness
startup pitch, or a Ph.D. defence on sacred geometry. Genre affinity is therefore **metadata
(`affinity` tag list), never part of the ID**. Baking genre into the filename would quietly
destroy the mix-and-match property this whole project exists for.

---

## 3. Slide filenames

```
<PKS-ID>__<slug>__v<N>.png            the rendered 4K 16:9 image
<PKS-ID>__<slug>__v<N>.prompt.txt     the exact prompt that produced it (sidecar)
<PKS-ID>__<slug>__v<N>.json           the catalog record snapshot (sidecar)
```

Concretely:

```
Slide_Assets/DV - Data and Visualisation/PKS-DV-WA-004__impossible-pie-chart-340pct__v1.png
Slide_Assets/DV - Data and Visualisation/PKS-DV-WA-004__impossible-pie-chart-340pct__v1.prompt.txt
Slide_Assets/DV - Data and Visualisation/PKS-DV-WA-004__impossible-pie-chart-340pct__v1.json
Slide_Assets/SY - Symbol and Ritual/PKS-SY-SM-002__mandala-of-office-chairs__v1.png
```

**Field rules**

| Field | Rule |
|---|---|
| separator between fields | double underscore `__` |
| separator inside a field | single hyphen `-` |
| `slug` | lowercase `[a-z0-9-]`, max 48 chars, derived from the idea title, no stop-words |
| `v<N>` | version, starts at `v1`, **auto-increments — files are never overwritten** |

A re-render of the same slide produces `…__v2.png` alongside `…__v1.png`. The catalog's
`active_version` field decides which one decks actually use, so a bad re-roll costs nothing.
`config.version_path()` takes `max(existing)+1`, never the first gap, so a manually deleted `_v2`
can never be reclaimed by a later render and left shadowing the wrong sidecars.

`Slide_Assets/` is partitioned into one folder per **type**, named `<CODE> - <Full Name>`
(`DV - Data and Visualisation/`, `QT - Quote/`, …). Type is the coarsest, most stable axis;
register changes more often during curation, and is already in the ID. Each type folder also
holds one **drop-zone sub-folder per register** for hand-added images — see `02` §2.

### 3.1 Hand-added images (drop-ins)

Images not produced by the pipeline may be dropped into
`Slide_Assets/<TYPE>/<REGISTER>/` under **any filename at all**. They have no ID, so the folder
tree supplies the taxonomy: type from the parent folder, register from the sub-folder.

`./07_ingest_dropins.sh` mints a real `PKS` ID and writes an editable `.meta.json` beside the
image. **It never moves, renames or deletes your file** — the ID lives in the catalog and the
sidecar, and the record simply points at wherever you put it. This is deliberate: an ingester
that rearranges a human's folder is an ingester nobody trusts with their only copy.

```
Slide_Assets/AC - Acronym/CD - Corporate Deadpan/scan_00417.png     ← you drop this
                    ↓  ./07_ingest_dropins.sh
                        …/scan_00417.png                            ← untouched
                        …/scan_00417.meta.json    ← new: id, title, beats, motifs, difficulty
                    ↓  edit the .meta.json, then
                       ./07_ingest_dropins.sh --rescan              ← catalog picks up the edits
```

Register resolution order, first hit wins: an existing `.meta.json` → the sub-folder name → a
register code in the filename (`mything_SM_2.png` → `SM`) → the type folder's
`_folder_defaults.md`. Everything else you leave blank also falls back to `_folder_defaults.md`.

Two behaviours worth knowing:

- **Relinking.** A file whose name *does* match the pipeline convention (`PKS-…__slug__v2.png`)
  but which is missing from that record's `renders[]` is adopted as that version rather than
  given a new ID. This is the recovery path if a catalog write is ever lost mid-render.
- **`--prune`** retires records whose image file has since vanished. It edits the catalog only;
  nothing on disk is deleted on your behalf, ever.

---

## 4. Show ID — `PKC-<GGG>-<NN>`

```
PKC  -  COR  -  01
 |       |       └── sequence within genre, 01–99
 |       └────────── GENRE, 3 letters
 └────────────────── namespace
```

Genres use **three** letters (shows) versus two for slide types (`TT`) — a deliberate visual tell
so you can never mistake one namespace for the other at a glance.

| Code | Genre |
|---|---|
| `COR` | Corporate & Professional |
| `ACA` | Academic & Educational |
| `SCI` | Sci-Fi, Fantasy & Bizarre |
| `CIV` | Civic & Personal |
| `SPR` | Spiritual, Ritual & Wellness |
| `MED` | Media, Sport & Entertainment |

Show source files: `seed/show_types/SHOWS_<GGG>_<genre_name>.md`.

---

## 5. Deck ID — `PKD-<YYYYMMDD>-<HHMM>-<XXXX>`

```
PKD-20260816-1432-A7F3
```

`XXXX` is a 4-hex-char digest of the RNG seed + the ordered slide-ID list, so **the ID is a
checksum of the deck's content**. Two decks with the same ID contain the same slides in the same
order. Passing `--seed 20260816` to the builder reproduces a deck exactly — which is how you run
the same deck against three different performers and compare.

**The deck ID is metadata, not a folder name.** It lives in `manifest.json`, at the top of
`RUNSHEET.md`, and in `catalog/history/deck_history.json`. The folder itself is named for the
**theme**, so a facilitator can find it by feel five minutes before a class:

```
output/fever_dream/
├── manifest.json                  deck_id, seed, theme, arc, show, motifs, chrome,
│                                  ordered slides, relaxations
├── RUNSHEET.md                    arc map, transition scaffolds, facilitator notes
├── 01__PRE__PKS-CT-CD-007.png     ← images sit directly in the folder, no slides/ level
├── 01__PRE__PKS-CT-CD-007.prompt.txt
├── 01__PRE__PKS-CT-CD-007.json
├── 02__CTX__PKS-MP-CD-001.png
└── …
```

A second deck of the same theme becomes `fever_dream-2`, `-3`, … — existing folders are never
overwritten. `--out DIR` puts a deck somewhere else entirely.

Deck slide filenames carry three things: the **position** (`01__`) so any file browser or
projector app plays them in order, the **beat** (`PRE__`) so the facilitator can read the arc
straight off the folder listing, and the original **`PKS` ID** so every slide traces back to the
master repository. Deck copies also carry composited **deck chrome** (footer bar, fake org mark,
`n/N`) and are downscaled to 2560px; the 4K masters in `Slide_Assets/` stay clean and untouched.

> **No `.pptx`.** PowerPoint output was dropped from the design. A numbered folder of 16:9 images
> plays in order in any OS image viewer with zero dependencies, and the live web app
> (`Slide_Show_App.html`) covers everything a projector actually needs. See `05` §11.

---

## 6. Catalog record shape

One JSON object per slide in `catalog/slides_master.json`:

```jsonc
{
  "id": "PKS-DV-WA-004",
  "slug": "impossible-pie-chart-340pct",
  "type": "DV",                      // Data & Visualisation
  "register": "WA",                  // Whacky-Absurd
  "title": "The 340% Pie",
  "concept": "A polished 3D pie chart whose eight slices sum to 340%...",
  "on_slide_text": ["Q3 ALLOCATION", "Kevin — 140%"],   // ≤ 8 words total
  "visual_style": "Sleek corporate deck, dark-mode, neon accents",
  "ambiguity": [                     // ≥3 readings from unrelated genres — the quality gate
    "COR: over-committed engineering capacity",
    "SPR: the soul is more than the sum of its parts",
    "CIV: how the HOA divided the parking lot"
  ],
  "affinity": ["COR", "ACA", "SCI"], // soft bias only, never a filter
  "beats": ["DAT", "PRB"],           // narrative slots this slide can fill
  "motifs": ["kevin", "donut", "teal"],   // for callback threading across a deck
  "difficulty": 2,                   // 1 easy justify … 5 brutal
  "source": "seed:SEED_DV_data_viz.md#L42",
  "origin": "seed",                  // seed | gemini-3.7-flash | manual
  "image_prompt": "…",               // filled by Stage 2 (Opus 5)
  "elaboration_notes": "…",          // what Opus changed and why (e.g. punchline removed)
  "renders": [ { "version": 1,
                 "path": "Slide_Assets/DV - Data and Visualisation/…__v1.png",
                 "model": "gemini-3-pro-image" } ],
  "active_version": 1,
  "status": "rendered"               // idea | prompted | rendered | approved | retired
}
```

`status` is the pipeline's state machine. Every stage advances it and refuses to re-process
records already past its own stage unless `--force` is passed.

**Pipeline fields are protected from seed re-imports.** `status`, `renders`, `active_version`,
`image_prompt`, `elaboration_notes`, `elaborated_by` and `retired_reason` are listed in
`catalog.PIPELINE_FIELDS` and are never touched by a `.md` re-parse. Only a genuine change to a
**creative** field (`title`, `concept`, `on_slide_text`, `visual_style`) rewinds a record to
`"idea"` — because that is the only case where the existing image no longer depicts the record.
Without this split, re-running Stage 1 would silently mark the whole rendered library as unbuilt.

---

## 7. Slug generation rules

Deterministic, so the same idea title always yields the same slug:

1. lowercase; 2. strip anything not `[a-z0-9 -]`; 3. drop stop-words
   (`a an the of and or to in on for with`); 4. collapse whitespace to `-`;
5. truncate to 48 chars at a hyphen boundary; 6. on collision within a cell, append `-2`, `-3`, …

Implemented once in `scripts/config.py::slugify()` and used everywhere. Never re-implement it.

---

## 8. Run archives

Every LLM stage writes a timestamped archive so no generation is ever lost:

```
prompts/runs/<YYYYMMDD-HHMMSS>__<stage>__<model>.json    exact request payload
catalog/ideas/<YYYYMMDD-HHMMSS>__ideas__<TT>.json        raw stage-1 output, per type
logs/<YYYYMMDD-HHMMSS>__<stage>.log                      console transcript
```

`catalog/history/idea_history.json` keeps a rolling list of every idea title ever produced and is
fed back into the ideation prompt as a **do-not-repeat list** — the same anti-repetition pattern
proven in `01_Improv_InfoGraphics`.
