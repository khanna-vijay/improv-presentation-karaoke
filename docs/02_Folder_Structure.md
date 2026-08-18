# 02 — Folder Structure

The layout mirrors the proven sibling pipeline `31_All_Improv_Code/01_Improv_InfoGraphics`
(numbered `.sh` drivers at root · `scripts/` · `prompts/` · `docs/` · asset dir with `.prompt.txt`
sidecars · `logs/`), extended with two things that pipeline does not have: a **catalog layer**,
because this project maintains a persistent queryable database rather than a one-shot batch, and
a **two-folder split** between permanent building blocks and disposable assembled decks.

---

## 1. The two-folder architecture

This is the mental model everything else hangs off:

```
  BUILDING BLOCKS  (permanent)                 ASSEMBLED THEMES  (disposable)
  Slide_Assets/<CODE - Full Name>/             output/<theme>/
  ──────────────────────────────               ────────────────
  AC - Acronym/                                boardroom/
    PKS-AC-CD-001__c-r-u-s-t__v1.png             01__PRE__PKS-SY-DT-001.png
  DV - Data and Visualisation/                   02__CTX__PKS-MP-WA-001.png
    PKS-DV-CD-001__kevin-donut__v1.png           03__DAT__PKS-DV-CD-001.png
  …                                              …  + .prompt.txt / .json beside each
  append-only · versioned _v1 → _v2              RUNSHEET.md · manifest.json
  never deleted                                ashram/  fever_dream/  …
                                                 delete any time; nothing is lost
```

Building blocks know nothing about themes, scenarios or decks. Theme folders are a **weighted
random draw** — one block per narrative beat — copied in with an ordering prefix. Because
assembly is a draw and not a mapping, the same library yields unlimited distinct decks.

---

## 2. The tree

```
18_PowerPoint Karaoke/
│
├── Seed_Ideas.md                    ← the original brief. READ-ONLY. Never edited by scripts.
├── README.md                        ← quickstart
├── RESUME.md                        ← live session state: what is done, what is next
├── requirements.txt
├── sample.env  /  .env              ← Vertex project, model overrides, tuning knobs
│
├── 00_setup_check.sh                ← ADC pre-warm + config self-check + coverage report
├── 01_ideate_slides.sh              ← STAGE 1  Gemini 3.7 Flash: seeds .md → concepts
├── 02_elaborate_prompts.sh          ← STAGE 2  Opus 5: idea → render-ready image prompt
├── 03_generate_slide_images.sh      ← STAGE 3  Gemini 3 Pro Image: prompt → 4K PNG
├── 04_build_deck.sh                 ← STAGE 4  assemble ONE theme folder (5–10, on an arc)
├── 05_build_session.sh              ← STAGE 5  assemble a whole class (N decks + N shows)
├── 07_ingest_dropins.sh             ← adopt hand-added images into the catalog
├── 08_build_images_from_prompts.sh  ← reusable prompt → image builder (variants, one-offs)
├── 09_run_full_build.sh             ← resumable A→D orchestration of the whole library
├── start_slide_app.sh               ← build the web index, serve it, open the app
│
├── Slide_Show_App.html              ← THE LIVE RANDOMISER. Spin → theme → deck → full screen.
├── app/
│   ├── library.js                   ← generated index the app reads (never hand-edited)
│   └── preview/                     ← 2560px JPEG previews; the browser never loads a 4K master
│
├── docs/                            ← THE PLAN (this folder)
│   ├── 00_Project_Overview.md
│   ├── 01_Naming_Convention.md
│   ├── 02_Folder_Structure.md
│   ├── 03_Taxonomy_Slide_Types_and_Registers.md
│   ├── 04_Show_Types_Catalog.md
│   ├── 05_Slide_Use_Method_Deck_Assembly.md
│   ├── 06_Pipeline_Design.md
│   ├── 07_LLM_Configuration.md
│   ├── 08_Master_Plan_and_Roadmap.md
│   └── 09_Facilitator_Runbook.md
│
├── seed/                            ← THE SEED DATABASE. Hand-authored .md. The scripts' input.
│   ├── 00_Seed_Master_Index.md
│   ├── 00_Deck_Chrome_Wordbank.md   ← fake org names / codenames / stamps for deck chrome
│   ├── show_types/                  ← AXIS A: 42 scenarios
│   │   ├── SHOWS_COR_corporate.md
│   │   ├── SHOWS_ACA_academic.md
│   │   ├── SHOWS_SCI_scifi_bizarre.md
│   │   ├── SHOWS_CIV_civic_personal.md
│   │   ├── SHOWS_SPR_spiritual_ritual.md
│   │   └── SHOWS_MED_media_entertainment.md
│   └── slide_types/                 ← AXIS B: 60 slide exemplars, 1 per type × register cell
│       ├── SEED_DV_data_viz.md      SEED_QT_quote.md      SEED_AC_acronym.md
│       ├── SEED_PH_photo.md         SEED_AR_abstract.md   SEED_CT_call_to_action.md
│       ├── SEED_DG_diagram.md       SEED_IL_illustration_comic.md
│       └── SEED_SY_symbol_ritual.md SEED_MP_map_timeline.md
│
├── prompts/                         ← LLM instruction assets (editable without touching code)
│   ├── ideation_system.md           Stage 1 system prompt
│   ├── ideation_user.md             Stage 1 user template  ({{TYPE}}, {{SEEDS}}, {{AVOID}} …)
│   ├── elaboration_system.md        Stage 2 system prompt
│   ├── elaboration_user.md          Stage 2 user template
│   ├── image_system.md              Stage 3 system instruction for the renderer
│   ├── custom_image_prompts.md      hand-written prompts for 08_build_images_from_prompts.sh
│   └── runs/                        every request payload, archived by timestamp
│
├── catalog/                         ← THE MASTER DATABASE (machine-owned; do not hand-edit)
│   ├── slides_master.json           every slide record — single source of truth
│   ├── shows_master.json            parsed from seed/show_types/*.md
│   ├── ideas/                       raw Stage-1 output, one file per run per type
│   └── history/
│       ├── idea_history.json        every idea title ever → anti-repetition memory
│       └── deck_history.json        every deck built → per-session no-repeat memory
│
├── Slide_Assets/                    ← BUILDING BLOCKS, partitioned by TYPE
│   ├── AC - Acronym/                  ← renders land here, at the type-folder root
│   │   ├── PKS-AC-CD-001__c-r-u-s-t__v1.png
│   │   ├── PKS-AC-CD-001__c-r-u-s-t__v1.prompt.txt
│   │   ├── PKS-AC-CD-001__c-r-u-s-t__v1.json
│   │   ├── _folder_defaults.md        ← editable defaults applied to drop-ins here
│   │   ├── CD - Corporate Deadpan/    ← DROP ZONE: hand-added images, any filename
│   │   ├── WA - Whacky Absurd/        ← register is read from the sub-folder name
│   │   └── CC · DT · SM · IE …
│   ├── DV - Data and Visualisation/   QT - Quote/       PH - Photographic/
│   ├── AR - Abstract Art/             CT - Call To Action/  DG - Diagram/
│   ├── IL - Illustration and Comic/   SY - Symbol and Ritual/  MP - Map and Timeline/
│   └── _rejected/                     curated-out renders, kept for reference, never drawn
│
├── output/                          ← ASSEMBLED THEME FOLDERS (disposable, regenerable)
│   └── <theme>/                       e.g. boardroom/  ashram/  fever_dream/
│       ├── manifest.json              seed · theme · arc · motifs · chrome · ordered slides
│       ├── RUNSHEET.md                arc map + transition scaffolds
│       └── NN__<BEAT>__<PKS-ID>.png   + .prompt.txt and .json beside each
│
├── scripts/                         ← Python implementation
│   ├── config.py                    paths, model IDs, taxonomy tables, slugify, self-check
│   ├── llm_clients.py               Vertex ADC clients: Gemini Flash · Opus 5 · Gemini Image
│   ├── seed_parser.py               .md → structured records (the "read from .md" contract)
│   ├── catalog.py                   load/save/merge/dedupe/ID-assign for slides_master.json
│   ├── deck_chrome.py               Pillow compositor: footer bar, org mark, n/N
│   ├── 1_ideate_slides.py           4_build_deck.py       7_ingest_dropins.py
│   ├── 2_elaborate_prompts.py       5_build_session.py    8_build_from_prompts.py
│   └── 3_generate_images.py         6_export_library.py
│
└── logs/                            ← timestamped console transcripts per stage
                                       fullbuild.log is the long unattended-run log
```

### Why renders sit at the type-folder root, not in the register sub-folder

The register sub-folders (`AC - Acronym/CD - Corporate Deadpan/`) exist for **humans**, not the
pipeline. A rendered slide already carries its register in its ID (`PKS-AC-**CD**-001`), so
burying it one level deeper adds nothing and makes globbing a type slower. A hand-added image has
no ID and no metadata, so the folder it is dropped into is the *only* signal available — hence
type from the parent folder, register from the sub-folder. `07_ingest_dropins.sh` reads both,
mints a real `PKS` ID and writes an editable `.meta.json` beside the file. It leaves the file
itself exactly where and as it is; the catalog record points at the original path.

Folders are named `<CODE> - <Full Name>` throughout so the repository is legible to somebody
browsing it in Explorer who has never read these docs.

---

## 3. Ownership rules

| Path | Owner | Rule |
|---|---|---|
| `Seed_Ideas.md` | human | **Read-only.** The origin brief. |
| `seed/**` | human | Hand-authored. Scripts read, never write. Add ideas here to steer generation. |
| `prompts/*.md` | human | Tune LLM behaviour here, not in Python. |
| `docs/**` | human | The plan. |
| `RESUME.md` | human | Live session state. Update it when you stop mid-run. |
| `Slide_Assets/<TYPE>/<REGISTER>/` | human | Drop zone. Any filename. `07_ingest_dropins.sh` adopts them. |
| `Slide_Assets/<TYPE>/_folder_defaults.md` | human | Defaults (register, beats, difficulty) applied to drop-ins. |
| `catalog/**` | machine | Do not hand-edit — you will lose it on the next merge. Curate via `--approve` / `--retire`. |
| `Slide_Assets/<TYPE>/*.png` | machine | Append-only. Re-renders bump `_v<N>`; nothing is overwritten. |
| `app/library.js`, `app/preview/` | machine | Generated by `6_export_library.py`. Regenerable. |
| `output/**` | machine | Disposable. Regenerable from `manifest.json` at any time. |
| `logs/`, `prompts/runs/` | machine | Append-only archives. Safe to prune. |

---

## 4. The `.md` → assets contract

The user requirement is that `.sh` scripts read `.md` source files. That contract is honoured in
`scripts/seed_parser.py`:

1. `parse_slide_seeds()` reads `seed/slide_types/SEED_<TT>_*.md` → seed exemplar records.
2. `parse_show_types()` reads `seed/show_types/SHOWS_<GGG>_*.md` → scenario records.
3. `parse_chrome_wordbank()` reads `seed/00_Deck_Chrome_Wordbank.md` → the fake org names,
   project codenames and classification stamps used for per-deck chrome.

…and at two more points outside it:

4. `7_ingest_dropins.py` reads `Slide_Assets/<TYPE>/_folder_defaults.md` → metadata defaults for
   hand-added images.
5. `8_build_from_prompts.py` reads `prompts/custom_image_prompts.md` → prompt blocks to render
   directly, bypassing Stages 1–2 entirely.

All parse the same fixed Markdown grammar (`###` heading = one record, `- **Field:** value` =
fields), documented at the top of every seed file and in `06_Pipeline_Design.md`. To change what
the library contains, **edit Markdown, re-run the `.sh`**. No code change required.

### Seed re-import never destroys pipeline state

`catalog.merge_records()` splits every record's fields into **creative** (title, concept,
on-slide text, visual style) and **pipeline** (`status`, `renders`, `active_version`,
`image_prompt`, …). A seed re-import updates creative fields only; it can never overwrite
`status` back to `"idea"` and orphan a finished render. A record is rewound to `"idea"` — and so
re-elaborated and re-rendered as a new version — **only** when the seed's creative text has
genuinely changed. That is the one case where the existing image no longer depicts the record.

---

## 5. Disk footprint

| Item | Size |
|---|---|
| One 4K 16:9 PNG master | ≈ 15–18 MB |
| 120-slide library | ≈ **1.8–2.2 GB** |
| One 2560px web preview JPEG | ≈ 0.6–1.0 MB |
| Whole `app/preview/` set (120) | ≈ 100 MB |
| One theme folder (7 slides, downscaled + sidecars) | ≈ 6–12 MB |

Deck copies are **downscaled to 2560px wide by default** — still comfortably above 1080p
projection, ~25× smaller than the master, and instant to copy to a venue laptop. `--full-res`
copies the 4K masters 1:1 instead. Chrome is composited onto the copy, so a chromed deck needs
real files; `--no-chrome --link` hard-links instead and makes theme folders effectively free.
`03_generate_slide_images.sh --size 2K` halves the master library footprint if disk is tight.
