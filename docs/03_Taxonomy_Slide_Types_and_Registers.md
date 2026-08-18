# 03 — Taxonomy: Slide Types × Registers

The master repository is organised as a **10 × 6 matrix**. Every slide occupies exactly one cell.

```
                CD        WA        CC        DT        SM        IE
              Corp-     Whacky-   Comic-    Deep-     Spirit-   Inspir-
              Deadpan   Absurd    Cartoon   Thinking  Mystic    Epic
        ┌───────────────────────────────────────────────────────────────┐
   DV   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Data & Visualisation
   QT   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Quote
   AC   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Acronym
   PH   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Photographic
   AR   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Abstract Art
   CT   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Call To Action
   DG   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Diagram
   IL   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Illustration / Comic
   SY   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Symbol & Ritual
   MP   │   ▪         ▪         ▪         ▪         ▪         ▪        │  Map / Timeline
        └───────────────────────────────────────────────────────────────┘
              60 cells  ·  1 hand-authored seed each  ·  target >=2 rendered each = 120
```

**Type answers "what is this object?" Register answers "what key is it played in?"**
The same idea in two registers is two genuinely different slides — a growth curve rendered
corporate-deadpan and the same growth curve rendered as a sacred ascension diagram will pull
completely different performances out of the same performer.

---

## 1. The ten TYPES

### `DV` — Data & Visualisation
Charts that assert something with total confidence and mean nothing. Pie, donut, bar, line,
scatter, radar, waterfall, funnel, sankey, gauge, heatmap, stacked area.
*The comedy lever:* axes, legends and units. Label the Y-axis with a smell.
**Ambiguity source:** the performer must invent what was measured.
Seeds: `seed/slide_types/SEED_DV_data_viz.md`

### `QT` — Quote
A single sentence set as profound wisdom, with attribution. Full-bleed photo backgrounds,
minimalist serif on white, chalkboard, illuminated manuscript.
*The comedy lever:* grammatical seriousness applied to semantic nonsense.
**Ambiguity source:** who said it, and what on earth they meant.
Seeds: `SEED_QT_quote.md`

### `AC` — Acronym
Letters presented as an established, capitalised, trademarked framework — with the expansion
**withheld**. The performer must invent what each letter stands for, live.
*The comedy lever:* the ® symbol. Total institutional confidence in a nonsense word.
**Ambiguity source:** the meaning is absent by design *and* there is an unrelated image to
account for. Two problems the performer must connect live. Highest-yield type in the deck.
Seeds: `SEED_AC_acronym.md`

> **Two hard rules, enforced in `config.TYPES["AC"]`, the seed file and both prompt files:**
>
> 1. **A full stop between every character and after the last one** — `C.R.U.S.T.®`, never
>    `CRUST`. The punctuation is what makes it read as an institution rather than a word.
> 2. **Split the frame.** Letters occupy one side at ~55–60% width; the other side carries one
>    whacky, beautifully-executed image that is **completely disconnected** from the letters. The
>    image must never illustrate the acronym — a picture of a mountain beside `S.P.I.R.E.` hands
>    the performer the answer and kills the slide.
>
> Rule 2 was added after the first tranche was rendered. Anything rendered before it is
> re-elaborated and re-rendered as `_v2`; `active_version` promotes the new one.

### `PH` — Photographic
Photoreal staged scenes. Corporate stock photography, documentary, archival black-and-white,
surveillance still, product shot. The realism is the joke's straight man.
*The comedy lever:* one impossible element in an otherwise mundane, perfectly lit scene.
**Ambiguity source:** what happened immediately before and after this photo.
Seeds: `SEED_PH_photo.md`

### `AR` — Abstract Art
No figures, no text, or almost none. Colour fields, textures, generative forms, fluid dynamics,
macro material shots, unidentifiable objects.
*The comedy lever:* the performer must project *everything*. A pure Rorschach.
**Ambiguity source:** total. The most difficult and most rewarding type.
Seeds: `SEED_AR_abstract.md`

### `CT` — Call To Action
An imperative in enormous type. Phase declarations, warnings, ultimatums, closing slides.
*The comedy lever:* the gap between the mildness of the format and the violence of the demand.
**Ambiguity source:** what the action *is* and why it's necessary.
Beats: the natural `CTA`, and a strong `PRE` when used as a title card.
Seeds: `SEED_CT_call_to_action.md`

### `DG` — Diagram
Structure without data. Flowcharts, org charts, 2×2 quadrants, Venn diagrams, cycles, pyramids,
funnels, maturity models, swim lanes, mind maps, exploded technical schematics.
*The comedy lever:* the implied logic. Arrows that go somewhere terrible.
**Ambiguity source:** the relationship the diagram claims exists.
Seeds: `SEED_DG_diagram.md`

### `IL` — Illustration / Comic
Drawn, not photographed. Single-panel cartoons, mascots, children's-book art, retro advertising,
manga, woodcut, safety-pictogram sequences.
*The comedy lever:* a friendly visual language delivering an unfriendly message.
**Ambiguity source:** what the character wants.
Seeds: `SEED_IL_illustration_comic.md`

### `SY` — Symbol & Ritual
Emblems, crests, sigils, mandalas, sacred geometry, tarot-like cards, ceremonial objects,
alchemical plates, corporate logos that feel like cults.
*The comedy lever:* implied doctrine. Everyone in the room already knows what this means —
except the performer.
**Ambiguity source:** the belief system it belongs to.
Seeds: `SEED_SY_symbol_ritual.md`

### `MP` — Map / Timeline
Anything with a spatial or temporal axis. World maps, floor plans, roadmaps, Gantt charts,
subway diagrams, blueprints, seating charts, evacuation plans, family trees, geological strata.
*The comedy lever:* the legend, and the one region marked with something inexplicable.
**Ambiguity source:** where/when we are, and what happens at the marked point.
Seeds: `SEED_MP_map_timeline.md`

---

## 2. The six REGISTERS

### `CD` — Corporate-Deadpan
The house style of PowerPoint Karaoke and the single most important register. Sleek, credible,
utterly straight-faced. Helvetica/Inter, generous whitespace, a two-colour brand palette, a
faint footer with a page number and a fake company mark. **Nothing signals a joke.**
The slide looks like it was approved by a steering committee.
*Target share of the library: ~30%.*

### `WA` — Whacky-Absurd
Loud, chaotic, maximalist. Clip-art collisions, WordArt, gradient meshes, too many exclamation
marks, 2003 shareware energy, colours that fight. The absurdity is on the surface.
*Target share: ~20%.*

### `CC` — Comic-Cartoon
Playful and illustrated. Bright flat colour, thick outlines, mascots, speech bubbles, meme
grammar, children's-book warmth. Approachable — which makes the content land harder.
*Target share: ~15%.*

### `DT` — Deep-Thinking
Contemplative and weighty. Muted palette, enormous negative space, small text, monochrome or
duotone photography, a single object in a void. Slow, quiet, philosophical.
Invites the performer to *slow down* — a valuable rhythm break in a fast comedy exercise.
*Target share: ~12%.*

### `SM` — Spiritual-Mystic
Sacred, cosmic, ceremonial. Gold leaf, deep indigo, radial symmetry, nebulae, candlelight,
illuminated manuscript, incense-smoke depth. Reverent and a little unnerving.
*Target share: ~12%.*

### `IE` — Inspiring-Epic
Motivational-poster grandeur. Golden-hour summits, cinematic wide shots, lens flare, rising
orchestral energy, heroic silhouettes. Uplifting to the point of menace.
*Target share: ~11%.*

---

## 3. Cross-cutting metadata

Beyond type and register, every slide record carries:

| Field | Values | Used for |
|---|---|---|
| `beats` | any of `PRE` `CTX` `DAT` `PRB` `INS` `PVT` `ASP` `PLN` `CTA` `STY` | **Which narrative slots this slide can fill** (`05` §2) |
| `motifs` | concrete nouns / colours / shapes | Callback threading across a deck (`05` §5.2) |
| `difficulty` | `1`–`5` | Scaling to beginner vs advanced classes |
| `affinity` | list of show genre codes | Soft bias only, never a hard filter |
| `text_weight` | `none` · `minimal` · `phrase` | Enforcing the ≤8-word rule and pacing |
| `ambiguity` | ≥3 readings from unrelated genres | **Quality gate — an idea without 3 fails** |

### Beat tagging

Beats are the **narrative slots** a slide can occupy — see `05_…Deck_Assembly.md` §2 for the full
definitions. Tag **generously**: most slides should carry 2–3 beats, because the library has to
fill eleven different arcs. A mandala can plausibly serve `PRE`, `INS`, `ASP`
*or* `CTA` depending on the deck it lands in; tag all four.

`STY` (Personal Story) is servable through **aliases** rather than direct tags — see `05` §2.1.
Newly ideated slides may be tagged `STY` directly and are preferred automatically when they are,
because a direct tag also matches.

Rough type → beat gravity (a default, not a rule):

| Type | Usual beats |
|---|---|
| `DV` Data-Viz | `DAT` `PRB` `CTX` |
| `QT` Quote | `ASP` `INS` `CTA` `PRE` |
| `AC` Acronym | `PRE` `PLN` `CTA` |
| `PH` Photo | `CTX` `PRB` `ASP` `PRE` |
| `AR` Abstract | `INS` `PVT` `CTX` |
| `CT` Call-to-Action | `CTA` `PRE` `PRB` |
| `DG` Diagram | `PLN` `INS` `DAT` `CTX` |
| `IL` Illustration | `PRB` `ASP` `PVT` |
| `SY` Symbol | `PRE` `INS` `ASP` `CTA` |
| `MP` Map / Timeline | `CTX` `PLN` `DAT` |

**Coverage floor:** ≥12 slides tagged for every beat; ≥20 for `DAT`, `PRB` and `CTA`, which every
arc uses. `--stats` reports this.

### `difficulty` scale

| # | Meaning |
|---|---|
| 1 | Obvious hooks. A beginner can justify it in two seconds. |
| 2 | One clear hook plus an oddity. |
| 3 | Genuinely ambiguous; requires a committed choice. |
| 4 | Actively resists interpretation. |
| 5 | Hostile. Two unrelated impossible elements. Advanced performers only. |

---

## 4. Coverage targets

The library is sized by **cell coverage, not by a headline slide count**. A 200-slide library
with three empty cells is worse than a 120-slide library with none: the builder can only draw
what exists, and an empty cell is a register the deck can never reach.

| Milestone | Rule | Slides |
|---|---|---|
| Seed *(done)* | exactly 1 hand-authored exemplar per cell | 60 |
| **Working library** | **≥2 rendered per cell** — `./09_run_full_build.sh` | **120** |
| Comfortable | ≥3 per cell — `./09_run_full_build.sh --cells 3` | 180 |
| Stretch | second ideation pass with anti-repetition history active | 300+ |

**Hard floor for a healthy library:** ≥2 rendered slides in *every* one of the 60 cells, plus the
beat coverage floor above (≥12 per beat; ≥20 for `DAT`, `PRB`, `CTA`). Cell coverage and beat
coverage are different constraints and both bite — a full matrix can still be unable to build an
`INQUIRY` deck if nothing is tagged `PLN`. The deck builder **fails loudly** and names the
starved beat rather than quietly building a deck that breaks its own arc.

The target is set by `--fill-cells K`, not by "N ideas per type": Stage 1 counts what each cell
already holds and asks only for the shortfall, so re-running it is cheap and self-limiting.

Run `./04_build_deck.sh --stats` at any time for live matrix and beat coverage. `RESUME.md`
records where the last run stopped.
