# 00 — Project Overview

**Project:** PowerPoint Karaoke — Slide Repository & Deck Roulette
**Location:** `31_All_Improv_Code/18_PowerPoint Karaoke`
**Source of truth for the brief:** `Seed_Ideas.md` (repo root, untouched)

---

## 1. What this is

PowerPoint Karaoke (a.k.a. *Slide Roulette*) is an improv exercise: a performer is handed a
**scenario** ("you are a Bond villain briefing your henchmen") and then must present a deck of
**slides they have never seen before**, justifying each one live as they click through.

This project builds the **asset factory** behind that exercise:

1. A **master repository of standalone slides** — each one deliberately ambiguous, visually
   authoritative, and *context-free* so it can land in any scenario. The library is sized by
   **coverage, not by a headline number**: the target is ≥2 rendered slides in every one of the
   60 type × register cells, which is **120 slides**. Raising the floor to 3 per cell gives 180.
2. A **deck assembler** that fills a declared **narrative arc** — Prelude → Context → Data →
   Problem → Insight → Aspiration → Plan → Call to Action — with 5–10 slides, so every deck tells
   a plausible story even though every slide in it is nonsense.
3. A **scenario catalog** of show types, drawn independently of the slides, so the pairing is
   always a genuine collision.
4. A **live web app** (`Slide_Show_App.html`) that does 2 and 3 in the browser, in the room, with
   no files written: spin a wheel for a theme, get a freshly compiled deck, present full screen.

The whole thing is driven by `.sh` scripts that read `.md` source files. Nothing is hand-placed.

---

## 2. The two independent axes

The single most important design decision in this project:

> **Slides know nothing about scenarios. Scenarios know nothing about slides.**

```
   AXIS A — SHOWS (the scenario a performer must inhabit)
   PKC-COR-01  The Crisis Management Press Conference
   PKC-SCI-03  Cult Initiation Orientation
   PKC-CIV-02  The Unprepared Best Man
                          ×
   AXIS B — SLIDES (the artefact they must justify)
   PKS-DV-WA-004  a pie chart totalling 340%
   PKS-SY-SM-002  a mandala made of office chairs
   PKS-CT-CD-007  "COMMENCE THE SHOUTING PHASE."
                          ↓
   A PRACTICE = 1 show  ×  1 theme  ×  5–10 slides on a narrative arc
```

Because the axes are independent, `42 shows × the slide pool × 12 themes` yields effectively
unlimited practices from a finite, reusable asset library. This is the "mix and match"
requirement.

---

## 3. Deliverables in this repo

| Layer | Artefact | Where |
|---|---|---|
| Plan | This docs set (`00`–`09`) | `docs/` |
| Seed DB — scenarios | 6 genre files, 42 shows | `seed/show_types/` |
| Seed DB — slides | 10 type files, 60 hand-authored exemplars (1 per type × register cell) | `seed/slide_types/` |
| Ideation | Gemini 3.7 Flash expands seeds until every cell holds ≥2 concepts | `catalog/ideas/` |
| Master DB | Merged, deduped, ID-stamped slide records | `catalog/slides_master.json` |
| Art direction | Opus 5 turns each idea into a render-ready image prompt | `catalog/slides_master.json` (`image_prompt`) |
| Images | Gemini 3 Pro Image → 4K 16:9 PNG + `.prompt.txt` + `.json` sidecars | `Slide_Assets/<CODE - Full Name>/` |
| Theme decks | 5–10 slides on an arc + chrome + runsheet, numbered `01__`, `02__`… | `output/<theme>/` |
| Live app | Spinner → theme → deck compiled in-browser → full-screen player | `Slide_Show_App.html` + `app/` |

---

## 4. Design principles (inherited from `Seed_Ideas.md`)

These are enforced in the prompts, not just aspirational:

1. **High Ambiguity** — every slide must support ≥3 wildly different readings. The ideation prompt
   requires each idea to ship with three sample justifications from three unrelated genres. If an
   idea can only mean one thing, it is rejected.
2. **Juxtaposition** — professional format, insane content. A McKinsey-grade quadrant chart whose
   axes are "Moisture" and "Regret".
3. **Vague Authority** — the slide must *look like it belongs* in a real deck. Standard fonts,
   confident layout, clean margins. The absurdity should ambush the performer.
4. **Minimal Text** — a 3-word phrase, not a paragraph. Stage time is for performing, not reading.
   Hard cap: **8 words of on-slide text**, enforced in the elaboration stage.
5. **No punchline on the slide** — the slide is a *prompt*, not a joke. The comedy is the
   performer's reaction. Slides that are already funny by themselves steal the performer's job.
6. **Absurd premises, impeccable logic** — the deck must look *authored*. A declared arc, a
   consistent footer, a repeating motif and an escalating build make seven unrelated images read
   as one coherent artefact. The audience should think "I don't understand this company, but it
   clearly understands itself," never "that was random." See `05_…Deck_Assembly.md` §5.
7. **Child-friendly, always** — this library runs in classrooms, school halls and corporate
   rooms, so **every slide must be suitable for a ten-year-old audience.** See §4.1.

### 4.1 Content rules — absolute

Nothing in this library may contain:

| Forbidden | Note |
|---|---|
| **Adult or sexual content** | No nudity, underwear, suggestive posing, innuendo or double entendre. |
| **Religion** | No real faith's symbols, dress, buildings, scripture, deities or clergy. The `SM` Spiritual-Mystic register means **invented** cosmic geometry — unnamed sigils, abstract mandalas, starfields, made-up alphabets — never Christianity, Islam, Judaism, Hinduism, Buddhism, Sikhism or any real tradition. |
| **Ethnic, racial, national or caste content** | Ethnicity is never the subject, the joke or the anomaly. No traditional dress as costume, no national stereotypes. |
| **Violence, gore, weapons, injury, death, self-harm, cruelty** | Including firearms and knives-as-threats. |
| **Drugs, smoking, alcohol abuse** | — |
| **Real people, politicians, celebrities, real brands** | — |
| **Visible distress** | Nobody crying or in pain. Mild bewilderment and deadpan boredom are the emotional range. |

**This is not a constraint on the comedy — it is a description of it.** The humour here comes
from *misplaced institutional confidence about harmless things*: a quadrant chart whose axes are
Moisture and Regret, a five-phase rollout plan for biscuits, an acronym nobody can expand, a
mandala made of office chairs. Shock is not available and is not needed. **Ambiguity is the
product**, and ambiguity survives every one of these restrictions intact.

Enforced redundantly, in four places, because any single layer can miss:

1. `prompts/ideation_system.md` — offending concepts are never generated
2. `prompts/elaboration_system.md` — Opus **rewrites** anything borderline rather than dropping
   it, keeping the slide's type, register, beats and ambiguity and swapping only the offending
   element (a cathedral becomes an invented civic institution; a weapon becomes an unexplained
   piece of office equipment)
3. `config.SAFETY_THRESHOLDS` — model-level blocking. Sexual content and hate speech block at
   `BLOCK_MEDIUM_AND_ABOVE`; dangerous content and harassment sit at `BLOCK_ONLY_HIGH` because
   stricter settings spuriously reject absurdist stage directions like "COMMENCE THE SHOUTING
   PHASE"
4. `./06_audit_content.sh` — screens the whole catalog against `config.BANNED_TERMS` and can
   `--quarantine` or `--requeue` any hit. Run it after every generation batch.

> **Known limit:** layer 4 is a keyword net over *text*. It cannot see an image. A slide whose
> prompt is clean can still render something unsuitable, so spot-check renders by eye.

---

## 5. Model roles

Per `C:\_VK_Code\____00_Re-Usable_Code_Assets\Models to be Used.txt`, all via Vertex AI + ADC
(`project=YOUR_GCP_PROJECT`, `location=global`). No API keys.

| Stage | Model | Why |
|---|---|---|
| 1 — Ideate | **Gemini 3.7 Flash** (`gemini-3.7-flash`) | High-volume divergent generation; cheap and fast for 120+ concepts |
| 2 — Elaborate | **Claude Opus 5** (`claude-opus-5`) | Precise art direction, layout discipline, on-slide text ≤8 words, no-punchline enforcement |
| 3 — Render | **Gemini 3 Pro Image** (`gemini-3-pro-image`) | 4K 16:9 PNG with legible embedded typography |

> At `location=global` the served image model ID is **`gemini-3-pro-image`**. The `-preview`
> suffix 404s on this project. Pinned in `scripts/config.py` and `sample.env`.

Full parameters in `07_LLM_Configuration.md`.

---

## 6. Reading order

| Doc | Read it for |
|---|---|
| `01_Naming_Convention.md` | The ID grammar for every slide, show and deck |
| `02_Folder_Structure.md` | Where everything lives and why |
| `03_Taxonomy_Slide_Types_and_Registers.md` | The 10 × 6 matrix |
| `04_Show_Types_Catalog.md` | The 42 scenarios and their genre codes |
| `05_Slide_Use_Method_Deck_Assembly.md` | **Beats, arcs, themes and coherence — the rules that turn a pile of slides into a story** |
| `06_Pipeline_Design.md` | Stage-by-stage script contracts |
| `07_LLM_Configuration.md` | Exact model params, retry policy, cost envelope |
| `08_Master_Plan_and_Roadmap.md` | Phases, targets, what's done, what's next |
| `09_Facilitator_Runbook.md` | How to actually run this in a class |

---

## 7. Status

Phases 0–3 are complete: the plan, the 60-exemplar seed database, the full pipeline (including
the drop-in ingester, the reusable prompt→image builder and the web app), and a proven sample
tranche of rendered slides and assembled theme folders. Phase 4 — rendering the library out to
≥2 slides per cell — is the current work.

`08_Master_Plan_and_Roadmap.md` holds the phase table. **`RESUME.md` at the repo root holds the
live counts** (how many slides are rendered right now, what stopped where, what to run next) and
is the file to read first when picking the project back up.
