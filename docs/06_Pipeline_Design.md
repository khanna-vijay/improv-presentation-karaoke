# 06 — Pipeline Design

Five numbered stages, plus four utility drivers and an orchestrator. Every stage is
**idempotent** and **resumable**: it skips records already past its own `status` unless
`--force` is given. Nothing is ever overwritten — a re-render becomes `_v2`.

```
 seed/slide_types/*.md ──┐
 seed/show_types/*.md  ──┤        hand-added images ──┐
                         │                            │
        ┌────────────────▼────────────────────────────┼──────────────────┐
        │ 00_setup_check.sh   ADC pre-warm · config self-check · stats    │
        └────────────────┬────────────────────────────┼──────────────────┘
                         │                            │
   ┌─────────────────────▼──────────────────────┐     │  status: —      → idea
   │ 01_ideate_slides.sh                        │     │
   │   Gemini 3.7 Flash  ·  --fill-cells K      │     │  catalog/ideas/*.json
   └─────────────────────┬──────────────────────┘     │  catalog/slides_master.json
                         │                            │
   ┌─────────────────────▼──────────────────────┐     │  status: idea   → prompted
   │ 02_elaborate_prompts.sh                    │     │
   │   Claude Opus 5  ·  idea → image prompt    │     │  slides_master.json.image_prompt
   └─────────────────────┬──────────────────────┘     │
                         │                            │
   ┌─────────────────────▼──────────────────────┐  ┌──▼─────────────────────────┐
   │ 03_generate_slide_images.sh                │  │ 07_ingest_dropins.sh       │
   │   Gemini 3 Pro Image · 4K 16:9 PNG         │  │ adopt images you added by  │
   │   Slide_Assets/<CODE - Name>/…__v1.png     │  │ hand → real PKS records    │
   │   + .prompt.txt + .json sidecars           │  └──┬─────────────────────────┘
   └─────────────────────┬──────────────────────┘     │  status: → rendered
                         │                            │
                         ├────────────────────────────┘
                         │
   ┌─────────────────────▼──────────────────────┐   ┌────────────────────────────┐
   │ 04_build_deck.sh   1 theme folder          │   │ 08_build_images_from_      │
   │ 05_build_session.sh N decks + N shows      │   │   prompts.sh               │
   │   → output/<theme>/                        │   │ prompts .md → images,      │
   │   (no LLM calls — pure local assembly)     │   │ bypassing stages 1–2       │
   └─────────────────────┬──────────────────────┘   └────────────────────────────┘
                         │
   ┌─────────────────────▼──────────────────────┐
   │ start_slide_app.sh   index → serve → open  │   app/library.js · app/preview/
   │   the live randomiser, no files written    │   Slide_Show_App.html
   └────────────────────────────────────────────┘

   09_run_full_build.sh  =  A(01 --fill-cells) → B(02) → C(03) → D(07 + index)
                            one resumable command for the whole library
```

---

## Stage 0 — `00_setup_check.sh`

No LLM calls. Run this first, every time.

1. Pre-warms the Google ADC token (`gcloud auth application-default print-access-token`).
   WSL OAuth read-timeouts are the single most common failure in this environment; pre-warming
   removes them.
2. `python3 scripts/config.py` — prints resolved paths, model IDs, taxonomy sizes, and checks
   that every arc resolves at every length. Fails loudly on a missing prompt asset.
3. `python3 scripts/seed_parser.py` — re-parses every `.md` seed and reports slide/show counts,
   matrix cell coverage and the **ambiguity gate** (every exemplar must carry ≥3 readings).
4. `python3 scripts/4_build_deck.py --stats` — the live type × register coverage matrix **and the
   beat coverage counts**.

```bash
./00_setup_check.sh
```

A healthy system prints: ADC OK · 10 types × 6 registers · 9 beats / 10 arcs / 12 themes · all
arcs resolve · 60 slide seeds · 42 shows · ambiguity gate 60/60 · coverage matrix.

---

## Stage 1 — `01_ideate_slides.sh` · Gemini 3.7 Flash

**Contract:** `seed/slide_types/SEED_<TT>_*.md` + anti-repetition history → new concepts merged
into `catalog/slides_master.json` with `status: "idea"`.

**Seed Markdown grammar** (parsed by `scripts/seed_parser.py`; identical in every seed file):

```markdown
### PKS-DV-CD-001 — The Kevin Donut
- **Register:** CD
- **Concept:** A polished 3D corporate donut chart with four slices…
- **On-Slide Text:** Q3 SEGMENTATION | Kevin
- **Visual Style:** Sleek dark-mode corporate deck, neon accents, thin sans-serif
- **Ambiguity:** COR: an account that ate the business | SPR: the ego consuming the self | CIV: who took the parking spaces
- **Affinity:** COR, ACA, SCI
- **Beats:** DAT, PRB, CTX
- **Motifs:** kevin, donut, teal
- **Difficulty:** 2
- **Text Weight:** minimal
```

`###` opens a record. `- **Field:** value` sets a field. `|` separates list items. Anything else
in the file (prose, tables, `##` section headings) is ignored, so the seed files stay readable as
documents.

**The `.md` exemplars are imported first, on every run**, before any API call. They are the floor
of the library, and `--seed-only` stops there.

### Two ways to ask for concepts

| Mode | Command | Shape |
|---|---|---|
| **Fill cells** *(the one we use)* | `--fill-cells 2` | Counts what each type × register cell already holds and asks only for the shortfall. One targeted Flash call **per deficient cell**. |
| Per type | `-n 15 --type DV QT` | One Flash call per type, N concepts each, register mix left to the model. |

`--fill-cells K` is how the library is actually built, and it is why the target is expressed as
"≥K per cell" rather than a slide count. It is self-limiting — once every cell is full, the whole
stage is a no-op costing zero calls — which is what makes `09_run_full_build.sh` safe to re-run
after any interruption.

Each call receives: the type's full definition and comedy lever (from `config.TYPES`), the
hand-authored exemplars from that type's seed file, the register target, the last
`HISTORY_LOOKBACK` (default 200) idea titles as an explicit **do-not-repeat** list, and the
design principles from `Seed_Ideas.md` as hard constraints.

**Structured output.** `response_mime_type: application/json` + an explicit `response_schema`, so
parsing never depends on fenced-code-block scraping.

**Quality gate.** An idea is rejected before it reaches the catalog if it:
- has fewer than 3 `ambiguity` readings, or they name fewer than 3 distinct genres;
- has `on_slide_text` totalling more than 8 words;
- duplicates an existing title (normalised similarity ≥ 0.88 against every catalog title);
- contains a self-contained punchline (checked in Stage 2 by Opus, which has better judgement).

Rejections are written to `catalog/ideas/<ts>__rejected__<TT>.json` with reasons — never silently
dropped.

```bash
./01_ideate_slides.sh --fill-cells 2      # top every cell up to 2 — the normal call
./01_ideate_slides.sh --seed-only         # re-import the .md exemplars only, no API calls
./01_ideate_slides.sh --type DV QT        # only these types
./01_ideate_slides.sh -n 12               # 12 per type
./01_ideate_slides.sh --dry-run           # parse seeds, build prompts, make no API calls
```

Shows are re-parsed into `catalog/shows_master.json` on every run too, so the deck builder always
has a current scenario catalog.

---

## Stage 2 — `02_elaborate_prompts.sh` · Claude Opus 5

**Contract:** every record with `status: "idea"` → a render-ready `image_prompt`, `status:
"prompted"`.

Opus 5 is used here rather than Flash because this stage is **judgement, not volume**. It must:

1. Convert a one-line concept into a precise art direction: composition, camera, palette,
   lighting, typography, layout grid, aspect framing.
2. **Enforce the ≤8-word text rule** and specify the exact strings to render, in quotes, with
   their positions — this is what makes Gemini 3 Pro Image produce legible, correctly-spelled
   on-slide text instead of typographic soup.
3. **Kill self-contained punchlines.** If the idea is already funny without a performer, Opus
   rewrites it to be *strange* rather than *funny* and notes the change in `elaboration_notes`.
4. Bind the register to a concrete visual system (see `config.REGISTER_STYLE`) so a `CD` slide is
   recognisably the same design language every time.
5. Enforce type-specific hard rules — most notably the `AC` split-frame and full-stop rules
   (`03` §1), which Opus applies retroactively to already-written concepts.
6. Emit the final `beats`, `motifs`, `difficulty` and `text_weight` — Opus overrides Flash's
   guesses, having seen the finished art direction. **Beat tagging is the load-bearing one:** a
   slide tagged for too few beats is dead weight the deck builder can never place.

Batched at `ELABORATION_BATCH` (default 8) ideas per call to control cost, with `thinking:
adaptive` and `effort: high`.

```bash
./02_elaborate_prompts.sh
./02_elaborate_prompts.sh --type SY --force      # re-elaborate one type from scratch
./02_elaborate_prompts.sh --id PKS-AR-DT-003
./02_elaborate_prompts.sh --limit 20 --batch 4   # cost control
```

> `--force` rewrites `image_prompt` but **leaves `status: "rendered"` alone** — an existing image
> is still an image. To actually get the new art direction on disk you must follow with
> `./03_generate_slide_images.sh --id … --force`, which writes `_v2`. This is why re-specifying a
> type is a two-command job.

---

## Stage 3 — `03_generate_slide_images.sh` · Gemini 3 Pro Image

**Contract:** every record with `status: "prompted"` → a PNG on disk, `status: "rendered"`.

- `response_modalities: ["IMAGE"]`, `aspect_ratio 16:9`, `image_size 4K`, `temperature 1.0`.
- `system_instruction` from `prompts/image_system.md`: the standing rules — projector-safe
  contrast, no watermarks, no gibberish text, no signature, spelling of embedded words must be
  exact, safe margins for 16:9 projection.
- Retries with exponential backoff + jitter (12 attempts, 5s → 300s cap), detecting 429 /
  RESOURCE_EXHAUSTED / 503 — the pattern proven in `01_Improv_InfoGraphics/scripts/llm_clients.py`.
- Writes `.png`, `.prompt.txt` (exact prompt) and `.json` (record snapshot) together into
  `Slide_Assets/<CODE - Full Name>/`.
- **Never overwrites**: `version_path()` bumps `_v1 → _v2 → …` from `max(existing)+1`.
- Rejects and retries a render under `MIN_IMAGE_SIZE_KB` (default 40 KB) — the signature of a
  blank or failed generation.

```bash
./03_generate_slide_images.sh                    # everything pending
./03_generate_slide_images.sh --type AC          # one type
./03_generate_slide_images.sh --register CD SM   # one or more registers
./03_generate_slide_images.sh --limit 20         # first 20 pending (cost control)
./03_generate_slide_images.sh --size 2K          # cheaper / smaller
./03_generate_slide_images.sh --id PKS-DV-WA-004 --force   # re-render → v2
```

This is the only expensive stage: roughly **2 minutes and one image call per slide**. Budget
~3 hours for a 94-slide tranche, and run it with `nohup … &`.

---

## Stage 4 — `04_build_deck.sh`

No LLM calls. Pure local assembly per the algorithm in `05_Slide_Use_Method_Deck_Assembly.md`.
Emits `output/<theme>/` containing `manifest.json`, `RUNSHEET.md` and
`NN__<BEAT>__<PKS-ID>.png` with `.prompt.txt` / `.json` beside each. Deck copies carry composited
chrome and are downscaled to 2560px. A second deck of the same theme becomes `<theme>-2`.

```bash
./04_build_deck.sh                                     # 7 slides, mixed_roulette, CLASSIC_PITCH
./04_build_deck.sh -n 10 --theme fever_dream
./04_build_deck.sh --theme story_spine -n 9            # the full Disney story spine
./04_build_deck.sh --theme ashram --arc DESCENT        # force a non-default arc
./04_build_deck.sh --show PKC-SCI-03 --affinity
./04_build_deck.sh --seed 20260816                     # reproducible
./04_build_deck.sh --count 8                           # eight folders in one go
./04_build_deck.sh --difficulty 4 --no-show --no-chrome
./04_build_deck.sh --full-res                          # 4K copies instead of 2560px
./04_build_deck.sh --out /media/usb/tonight            # write somewhere else
./04_build_deck.sh --stats                             # coverage report, builds nothing
```

It also owns curation, because curation is a catalog edit and the catalog is this script's
domain: `--approve`, `--retire`, `--rebeat <ID> <BEATS>`.

---

## Stage 5 — `05_build_session.sh`

Wraps Stage 4 for a whole class into one `output/SESSION-<date>-<time>/` folder, with global
no-repeat, genre-cycling show assignment and optional theme rotation so the class collectively
sees every arc. Emits `SESSION_RUNSHEET.md` (running order, naming each performer's real folder)
and `session_manifest.json`.

```bash
./05_build_session.sh --performers 12 --slides 7
./05_build_session.sh --performers 8 --theme boardroom --difficulty 2
./05_build_session.sh --performers 12 --theme-rotate   # every performer gets a different arc
./05_build_session.sh --performers 12 --allow-repeat   # if the library can't carry the class
```

---

## `07_ingest_dropins.sh` — adopt hand-added images

Not a stage; a side door into the same catalog. Drop images with **any filename** into
`Slide_Assets/<TYPE>/<REGISTER>/` and this mints real `PKS` records for them so the deck builder
and the app can draw them exactly like generated slides.

- Type from the parent folder; register from an existing `.meta.json` → the sub-folder → a code
  in the filename → `_folder_defaults.md`.
- Writes an editable `.meta.json` beside each image; `--rescan` re-applies your edits.
- **Never moves, renames or deletes your files.**
- Also **relinks** pipeline-named files missing from the catalog — the recovery path if a catalog
  write is lost mid-render.
- `--prune` retires records whose file has since disappeared (catalog only; nothing on disk).

```bash
./07_ingest_dropins.sh            # adopt anything new
./07_ingest_dropins.sh --rescan   # after editing a .meta.json
./07_ingest_dropins.sh --dry-run
```

---

## `06_audit_content.sh` — the content safety screen

The library must stay suitable for a ten-year-old audience (`00` §4.1). Three layers enforce
that before an image exists — the ideation prompt, the elaboration prompt, and the model-level
safety thresholds. This is the fourth, and the only one that can inspect what is **already in the
catalog**, including records generated before the rules existed.

It scans every record's title, concept, on-slide text, visual style and image prompt against
`config.BANNED_TERMS` (adult, religious, ethnic, violent, drug-related). The match is a blunt
word-boundary scan: it over-reports on purpose, because a human decides and the cost of a false
positive is reading one line.

```bash
./06_audit_content.sh                  # report only  (exit 1 if anything is flagged)
./06_audit_content.sh --field prompt   # scan the image prompt only
./06_audit_content.sh --quarantine     # status -> retired: out of the draw pool, files kept
./06_audit_content.sh --requeue        # status -> idea: re-elaborate under the current rules
./06_audit_content.sh --id PKS-IL-IE-002 --requeue    # one record
```

`--requeue` is usually the right action: Stage 2 rewrites the prompt keeping the slide's type,
register, beats and ambiguity, and Stage 3 renders a new version. Nothing is ever deleted.

> **Two limits worth knowing.** It is a keyword net over *text* — it cannot see an image, so a
> clean prompt can still render something unsuitable; spot-check by eye. And `--requeue` rewrites
> `image_prompt`, not `concept` — if the offending element is in the source concept it will come
> back on the next re-elaboration, so fix the concept too.

Run it after every generation batch, and before any class.

---

## `08_build_images_from_prompts.sh` — the reusable prompt → image builder

Renders images straight from prompts, bypassing Stages 1–2 entirely. Use it when you already know
exactly what you want, or to generate variants of an existing slide.

```bash
./08_build_images_from_prompts.sh --file prompts/custom_image_prompts.md   # a .md of prompt blocks
./08_build_images_from_prompts.sh --id PKS-AC-CD-001 --variants 3          # 3 re-rolls of one slide
./08_build_images_from_prompts.sh --prompt "…" --title "…" --type SY --register SM
./08_build_images_from_prompts.sh --file … --activate   # promote the new render to active_version
```

Same never-overwrite guarantee: every run adds a version, never replaces one.

---

## `start_slide_app.sh` — the live randomiser

```bash
./start_slide_app.sh                  # index + serve + open  → http://localhost:8777
./start_slide_app.sh --index-only     # just rebuild app/library.js and app/preview/
./start_slide_app.sh --port 9000
```

`scripts/6_export_library.py` writes `app/library.js` — the slide index plus `BEATS`, `ARCS`,
`THEMES` and `TRANSITIONS` straight out of `config.py`, so **a new arc or theme appears in the app
with no JavaScript change** — and 2560px JPEG previews into `app/preview/`. The browser never
loads a 4K master. Re-run `--index-only` after rendering or curating.

---

## `09_run_full_build.sh` — the orchestrator

One resumable command that takes the library from wherever it is to "K rendered per cell".

```bash
nohup ./09_run_full_build.sh > logs/fullbuild.log 2>&1 &   # the long unattended run
./09_run_full_build.sh --limit 30                          # render 30 this pass
./09_run_full_build.sh --size 2K                           # cheaper / faster / smaller
./09_run_full_build.sh --cells 3                           # aim for 3 per cell (180 slides)
```

| Phase | What | Cost |
|---|---|---|
| A — ideate | `01 --fill-cells K`; a no-op once every cell is full | one Flash call per deficient cell |
| B — elaborate | `02`; every remaining `idea` | ~1 Opus call per 8 records |
| C — render | `03`; every `prompted` record | **1 image call each, ~2 min each** |
| D — index | `07_ingest_dropins.sh` then `6_export_library.py` | free |

It checks ADC before starting and prints `--stats` at the end. Every phase filters on `status`,
so killing it and re-running loses nothing but the in-flight record.

---

## Cross-cutting

| Concern | Implementation |
|---|---|
| **Idempotence** | Every stage filters on `status`; re-running is a no-op without `--force`. |
| **Crash safety** | `slides_master.json` is written atomically (temp file + `os.replace`) after every record, so a mid-run crash loses at most one record. |
| **Resumability** | Kill Stage 3 at slide 60 of 94; re-run and it starts at 61. |
| **Never overwrite** | `version_path()` takes `max(existing)+1`. No render, prompt or sidecar is ever replaced or deleted; `active_version` decides what gets used. |
| **Seed re-import safety** | `PIPELINE_FIELDS` keeps a `.md` re-parse from clobbering `status`/`renders`. Only a real change to creative text rewinds a record. See `02` §4. |
| **Auditability** | Every request archived to `prompts/runs/`, every console session to `logs/`, the exact prompt beside every image. |
| **Anti-repetition** | `catalog/history/idea_history.json` grows monotonically and is fed back into Stage 1. |
| **Beat starvation** | Stage 4 fails loudly and names the starved beat rather than silently breaking a deck's arc. `--stats` shows beat counts before you find out the hard way. |
| **No API keys** | All three models via Vertex AI + local ADC. `GOOGLE_API_KEY` / `GEMINI_API_KEY` are actively unset in `llm_clients.py` to prevent accidental key-path fallback. |
| **Effort clamping** | `CLAUDE_EFFORT` is clamped to `{low, medium, high}` in `llm_clients.py`; a shell-level `xhigh` would otherwise be rejected by the API mid-run. |
| **Dry runs** | Every LLM stage supports `--dry-run`: builds and archives the exact payload, makes no call. |
| **Killing a background run** | Use `pgrep -f <pattern>` then `kill <pid>`. `pkill -f` has matched and killed the calling shell here, leaving orphaned Python still writing to the catalog. |
