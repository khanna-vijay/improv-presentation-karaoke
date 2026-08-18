# 08 — Master Plan & Roadmap

The target: a **master repository of context-free slides, sized by coverage rather than by a
headline count**, that can be endlessly recombined into themed, story-shaped practice decks. The
expensive work (rendering) happens once; every deck for every class afterwards is a free local
draw.

> **The live numbers — how many slides are rendered *right now*, what stopped where, what to run
> next — are in `RESUME.md` at the repo root.** This document holds the plan; that one holds the
> state. Do not duplicate counts here; they go stale within a session.

---

## The two-folder architecture

This is the mental model the whole system rests on:

```
  BUILDING BLOCKS                                ASSEMBLED THEMES
  Slide_Assets/<CODE - Full Name>/               output/<theme>/
  ──────────────────────────────                 ─────────────────
  DV - Data and Visualisation/                   boardroom/
      PKS-DV-CD-001__kevin-donut__v1.png           01__PRE__PKS-AC-CD-001.png
  AC - Acronym/                                    02__CTX__PKS-MP-CD-001.png
      PKS-AC-CD-001__c-r-u-s-t__v1.png             03__DAT__PKS-DV-CD-001.png
  CT - Call To Action/                             …
      PKS-CT-WA-001__activate-geese__v1.png        RUNSHEET.md · manifest.json
  …    context-free, reusable slides             ashram/  fever_dream/  story_spine/
                                                 boardroom-2/ …
       ↑ permanent · append-only · never          ↑ disposable · regenerate anytime
         deleted, only versioned (_v1 → _v2)
```

- **Building blocks** are permanent assets. One slide, one file, organised by slide type. They
  know nothing about themes, scenarios or decks.
- **Theme folders** are assembled views: for each beat of a theme's narrative arc, one building
  block is drawn at random (weighted by register, difficulty and motif) and copied in with a
  `01`, `02`, `03` prefix so it plays in order.

Because the assembly is a *draw*, not a fixed mapping, the same blocks yield an effectively
unlimited number of distinct themed decks. Delete `output/` at any time; nothing is lost.

**Three ways to compile a new theme folder:**

| Way | Command | Use when |
|---|---|---|
| One deck | `./04_build_deck.sh --theme ashram -n 8` | You want a folder to carry to a venue |
| Whole class | `./05_build_session.sh --performers 12` | Teaching; global no-repeat across performers |
| **Live, no folder** | `./start_slide_app.sh` | In the room. Spin the wheel, present, re-roll |

---

## Phases

| Phase | Deliverable | Status |
|---|---|---|
| **0 — Scaffold & plan** | Folder notation, ID grammar, taxonomy, arcs, this docs set | ✅ Done |
| **1 — Seed database** | 60 slide exemplars (1 per type × register cell), 42 scenarios, chrome wordbank | ✅ Done |
| **2 — Pipeline** | 5 numbered stages + 4 utility drivers + orchestrator, catalog layer, deck builder, web app | ✅ Done |
| **3 — Sample proof** | Tranche elaborated and rendered end to end; theme folders built and verified | ✅ Done |
| **4 — Fill the matrix** | `./09_run_full_build.sh` → ≥2 rendered per cell (120 slides) | ▶ In progress |
| **5 — Curate** | Run classes; `--approve` what worked, `--retire` what became predictable | Pending |
| **6 — Deepen to 3/cell** | `./09_run_full_build.sh --cells 3` → 180 slides | Later |
| **7 — Expand to 300+** | Second ideation pass with anti-repetition history active | Later |

---

## Phase 4 — filling the matrix

One command, resumable, safe to kill and restart:

```bash
nohup ./09_run_full_build.sh > logs/fullbuild.log 2>&1 &
./04_build_deck.sh --stats          # watch it from another shell
```

It runs A (ideate to the cell floor) → B (elaborate) → C (render) → D (ingest drop-ins + rebuild
the app index), skipping everything already done. Levers: `--limit N` to render one tranche,
`--size 2K` to halve time/cost/disk, `--cells 3` to raise the floor.

If you would rather drive it by hand:

```bash
./00_setup_check.sh                        # ADC + config + coverage
./01_ideate_slides.sh --fill-cells 2       # one Flash call per deficient cell
./02_elaborate_prompts.sh                  # → render-ready prompts
./03_generate_slide_images.sh --limit 40   # tranche 1
./04_build_deck.sh --stats                 # check coverage before spending more
./03_generate_slide_images.sh              # the rest
./start_slide_app.sh --index-only          # refresh app/library.js and previews
```

### Coverage gates — check these before declaring the library done

| Gate | Target | Why |
|---|---|---|
| Matrix cells with ≥2 slides | 60/60 | Every type × register combination is drawable |
| Slides per beat | ≥12 | Any arc can be filled |
| Slides for `DAT`, `PRB`, `CTA` | ≥20 each | Every arc uses these; they starve first |
| `text_weight: none` slides | ≥25 | R7 needs a wordless slide in every deck; `slow_burn` needs 3 |
| Difficulty spread | no band <10% | Beginner and advanced classes both served |

`./04_build_deck.sh --stats` prints all of these. **`DAT` is the historical bottleneck** — chart
slides are the least generous beat-taggers, and `PLN` and `CTA` run close behind. If a build
fails with beat starvation it will name the beat; the fix is either
`./01_ideate_slides.sh --type DV DG MP CT` and re-render, or the much cheaper
`./04_build_deck.sh --rebeat <ID> <BEATS>` on slides that are under-tagged rather than bad.

### Cost shape

| Stage | Calls for 120 slides | Weight |
|---|---|---|
| 1 — Flash ideation | ≤60 (one per deficient cell) | negligible |
| 2 — Opus 5 elaboration | ~15 | modest |
| 3 — Gemini 3 Pro Image | 120, ~2 min each | **dominant** |
| 4/5 — deck assembly | 0 | free, forever |

Levers: `--limit N`, `--size 2K` (halves cost and disk, still above 1080p), `--type <TT>` to
build one type at a time, `--dry-run` to validate prompts before spending.

---

## Phase 5 — curation

Slides earn their place on stage, not in the catalog.

```bash
./04_build_deck.sh --approve PKS-DV-CD-001    # worked → 1.4× draw weight
./04_build_deck.sh --retire  PKS-QT-CC-011    # died / too on-the-nose → out of pool
./04_build_deck.sh --rebeat  PKS-AR-DT-001 INS,ASP,CTA,PVT   # re-tag placement
./03_generate_slide_images.sh --id PKS-AR-DT-001 --force     # re-render → v2
./start_slide_app.sh --index-only                            # refresh the app
```

**The retirement criterion that matters:** if three different performers gave roughly the same
justification for a slide, it is no longer ambiguous. Retire it. Ambiguity is the product.

Re-tagging beats is the cheapest, highest-leverage fix in the whole system: a slide that keeps
failing to place is usually under-tagged, not bad.

---

## Extending the system

Everything below is a Markdown edit plus a re-run. No code changes.

| To add… | Edit | Then |
|---|---|---|
| A scenario | `seed/show_types/SHOWS_<GGG>_*.md` | `./01_ideate_slides.sh --seed-only` |
| A slide exemplar (steers all future ideation) | `seed/slide_types/SEED_<TT>_*.md` | `./01_ideate_slides.sh --type <TT>` |
| Fake org names / codenames for deck chrome | `seed/00_Deck_Chrome_Wordbank.md` | nothing — read at build time |
| An image you made or found elsewhere | drop it in `Slide_Assets/<TYPE>/<REGISTER>/` | `./07_ingest_dropins.sh` |
| A one-off image from a prompt you wrote | `prompts/custom_image_prompts.md` | `./08_build_images_from_prompts.sh --file …` |
| Different ideation behaviour | `prompts/ideation_*.md` | `./01_ideate_slides.sh` |
| Different art-direction discipline | `prompts/elaboration_*.md` | `./02_elaborate_prompts.sh --force` |
| Standing renderer rules | `prompts/image_system.md` | `./03_generate_slide_images.sh --force` |

Code changes are only needed for genuinely new structure: a new **slide type** (`config.TYPES` +
a new `SEED_<TT>_*.md`), a new **register** (`config.REGISTERS` + `REGISTER_STYLE`), a new **arc**
(`config.ARCS`) or a new **theme** (`config.THEMES`). Adding an arc or theme is ~8 lines in
`config.py` and is **immediately available in both the CLI and the web app** — the app reads its
arc and theme tables out of `app/library.js`, which `6_export_library.py` generates from
`config.py`. Re-run `./start_slide_app.sh --index-only` and it is there.

Changing a type's *specification* (as the `AC` split-frame rule did) is not a code-only change:
existing renders predate the new rule, so follow with
`./02_elaborate_prompts.sh --type <TT> --force` and then
`./03_generate_slide_images.sh --id <the stale IDs> --force` to produce `_v2`s. Do not pass
`--type` to that second command unless you want to re-render the whole type, including the slides
that were already built to the new spec.

---

## Known limits

- **Beat starvation is the real constraint**, not slide count. A 200-slide library that cannot
  fill `PLN` still cannot build an `INQUIRY` deck. Watch the beat histogram, not the total.
- **Cell coverage and beat coverage are different gates** and both bite. A full 60/60 matrix can
  still starve a beat, and a healthy beat histogram can still leave a register unreachable.
- **Deck chrome is composited at build time**, so a chromed deck folder needs real file copies
  (`--link` requires `--no-chrome`). The web app draws chrome in CSS instead, so it is free there.
- **`gemini-3-pro-image`** is the served model ID at `location=global`; the `-preview` suffix
  404s on this project. Pinned in `config.py` and `sample.env`.
- **4K PNGs are 15–18 MB each.** A 120-slide library is ~2 GB. Use `--size 2K` if that matters.
  The browser never touches them — the app uses 2560px JPEG previews from `app/preview/`.
- **Session mode at 12 × 7 is the real stress test.** Global no-repeat drains the thinnest beat
  first, so it will surface a coverage hole that single-deck builds never hit.
