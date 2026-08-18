# 05 — Slide Use Method: Themes, Narrative Arcs & Deck Assembly

How a pile of standalone slides becomes a **5–10 slide practice that tells a story**.
This is the algorithmic heart of the project.

Two things run this algorithm, and they agree by construction: `scripts/4_build_deck.py`, which
writes a folder to `output/`, and `Slide_Show_App.html`, which does the same draw in the browser
and writes nothing at all. Everything below describes both.

---

## 1. The core principle: known skeleton, unknown flesh

A deck of purely random slides is a hazing ritual. A deck with a **predictable narrative shape**
is a training tool.

> **The performer knows the *shape* of the deck. They never know the *content*.**

Every deck in this system follows a declared **arc** — a sequence of narrative beats such as
*Prelude → Context → Data → Problem → Insight → Aspiration → Plan → Call to Action*. The performer
is told the arc before they begin (it's printed on the runsheet and, in class, on the wall).

This is what a real presenter has: they know a chart is coming, they know they must land on an
ask. What they do not have is *this* chart. Removing the structural uncertainty and keeping the
content uncertainty is what turns chaos into skill-building:

- The performer can **plan two slides ahead** instead of surviving one at a time.
- They learn **transitions** — "which is exactly why the numbers look like this…" — because they
  know a data slide is next.
- The **ending is guaranteed to land**, because every arc terminates in a Call to Action.
- Comparing performers becomes meaningful: same skeleton, different flesh.

```
ONE PRACTICE  =  1 performer
              +  1 SHOW    (scenario — who you are, drawn from shows_master.json)
              +  1 THEME   (register palette + narrative arc)
              +  1 DECK    (5–10 slides filling that arc's beats)
              +  3–6 minutes on stage
```

**Why 5–10?** Five is the shortest arc that still has a beginning, a turn and an ending. Ten is
the most a 5–6 minute set can carry before it becomes a slideshow read-out. Default is **7**.

---

## 2. The ten narrative beats

Every slide in the master catalog is tagged with the beats it can serve (`beats: ["DAT","INS"]`).
Most slides serve two or three. This is the vocabulary all arcs are built from.

The **`short`** column is the cue word printed very small in the bottom-right corner of the
projected slide — the only text the performer gets while presenting (`11` and `09` §5).

| Beat | `short` | Name | What the performer must do here | Natural slide types |
|---|---|---|---|---|
| `PRE` | Prelude | **Prelude / Title Card** | Name the thing. Establish who you are and why we're here. | `AC` `CT` `SY` `PH` |
| `CTX` | Context | **Context / Landscape** | "Here is the world as it stands." Set the scene. | `MP` `PH` `DG` `AR` |
| `DAT` | Data | **Data / Evidence** | "The numbers speak for themselves." Assert a measured fact. | `DV` `DG` `MP` |
| `PRB` | Problem | **Problem / Threat** | "But." Introduce the complication. Raise stakes. | `PH` `AR` `IL` `CT` `DV` |
| `INS` | Insight | **Insight / Revelation** | The turn. "And then we understood." Reframe everything. | `AR` `SY` `DG` `QT` |
| `PVT` | Pivot | **Pivot / Anomaly** | The tonal gear-change. Something that does not belong. | any — chosen for register distance |
| `ASP` | Aspiration | **Aspiration / Inspiration** | The uplift. Paint the promised land. | `QT` `PH` `SY` `IL` |
| `PLN` | Plan | **Plan / Method** | "Here is how." Process, phases, mechanism. | `DG` `MP` `AC` |
| `CTA` | Call to Action | **Call to Action** | The demand. Land the plane. Never a summary — an *ask*. | `CT` `QT` `SY` |
| `STY` | Personal Story | **Personal Story / Lived Experience** | Drop the authority. Something that happened to *you*. First person, past tense, small details. | any — see aliases below |

**Beat tagging is generous by design.** A mandala can serve `PRE`, `INS`, `ASP` or `CTA`
depending on the deck it lands in. That flexibility is what keeps the library able to
fill eleven different arcs.

### 2.1 Beat aliases (`config.BEAT_ALIASES`)

A beat is a *performance instruction*, not a property of the picture — the image is ambiguous and
the performer supplies the function. `STY` was added after the catalogue was already tagged and
rendered, so no slide carries it directly. Rather than re-tag 120 records, `STY` declares aliases:

```python
BEAT_ALIASES = {"STY": ("CTX", "PVT", "INS", "ASP")}
```

Any slide already tagged for scene-setting, a tonal swerve, a realisation or an uplift can carry a
personal story. `config.serves(rec_beats, beat)` is the single predicate for "may this slide be
drawn for this beat", and every pool filter, the starvation guard, rule R9 and the coverage
counters all go through it.

**This matters:** a beat that nothing can serve trips the beat-starvation guard in
`draw_deck()` and *no deck using that arc can be built at all*. Never add a beat to an arc
without either tagging slides for it or giving it aliases.

---

## 3. Arcs

An **arc** is an ordered beat sequence plus a **priority list** that tells the builder which beats
to drop first when the deck is short. One arc definition therefore scales cleanly from 5 to 10
slides without a separate template per length.

```
ARC: CLASSIC_PITCH
  sequence  PRE → CTX → DAT → PRB → INS → PVT → ASP → PLN → CTA → (DAT²)
  priority  PRE, CTA, DAT, ASP, PRB, CTX, INS, PLN, PVT, DAT²

  n=5   PRE ─ DAT ─ PRB ─ ASP ─ CTA
  n=6   PRE ─ CTX ─ DAT ─ PRB ─ ASP ─ CTA
  n=7   PRE ─ CTX ─ DAT ─ PRB ─ INS ─ ASP ─ CTA
  n=8   PRE ─ CTX ─ DAT ─ PRB ─ INS ─ ASP ─ PLN ─ CTA
  n=9   PRE ─ CTX ─ DAT ─ PRB ─ INS ─ PVT ─ ASP ─ PLN ─ CTA
  n=10  PRE ─ CTX ─ DAT ─ DAT ─ PRB ─ INS ─ PVT ─ ASP ─ PLN ─ CTA
```

Algorithm: take the first `n` entries of `priority`, then re-sort them into `sequence` order.
`PRE` and `CTA` are pinned first and last in every arc and are never dropped.

### Named slots (`stems`)

Most arcs describe a *shape*: the performer is told "a data slide is coming" and finds their own
words. One arc — `STORY_SPINE` — describes a *script*: each slot has a fixed spoken sentence the
performer must actually say. An arc that works this way carries a `stems` map in
`config.ARCS[...]["stems"]`, beat → sentence. Where stems exist they replace the generic
transition scaffolds (§5.5) on the runsheet and in the app, because on such an arc the stem *is*
the transition. Arcs without `stems` behave exactly as before.

### The eleven arcs

#### `CLASSIC_PITCH` — the business deck
`PRE → CTX → DAT → PRB → INS → PVT → ASP → PLN → CTA`
The default and the one every performer should learn first. It is the shape of every quarterly
review, product launch and consultancy readout on earth. Universally legible.

#### `HERO_JOURNEY` — the thought-leadership talk
`PRE → CTX → STY → PRB → DAT → INS → ASP → PVT → PLN → CTA`
Ordinary world, **what happened to me**, disruption, evidence of the disruption, the revelation,
transformation, the call. Rewards vulnerability and status play.

`STY` sits third in the priority list, right after the `PRE`/`CTA` bookends, so **even a
six-slide TED talk still has to get personal** — the anecdote is the point of the form, not a
garnish to be dropped when the deck shortens. At the default `-n 7` this arc resolves to
`PRE → CTX → STY → PRB → INS → ASP → CTA`.

#### `REVELATION` — the sermon
`PRE → CTX → INS → PVT → DAT → ASP → PLN → CTA`
Invocation, contemplation, the teaching, the paradox, scripture-as-data, the blessing, the
commission. The insight arrives *early* and everything after justifies it backwards — a genuinely
different rhetorical muscle.

#### `DESCENT` — the collapse
`PRE → DAT → CTX → PVT → PRB → PVT → ASP → CTA`
Starts credible, ends unhinged. **Difficulty ramps 1 → 5 across the deck** and two pivots are
mandatory. The performer must maintain composure while the deck betrays them. Advanced.

#### `CRISIS_BRIEF` — the mission briefing
`PRE → CTX → DAT → PRB → INS → PLN → PVT → CTA`
Classification, situation, intel, threat, the asset, the mission, the complication, execute.
Terse, high-status, low-warmth. Bond villains, spy handlers, emergency coordinators.

#### `ORIGIN_STORY` — the children's assembly
`PRE → CTX → PRB → INS → ASP → PVT → PLN → CTA`
Once upon a time, the world, the trouble, the friend who helped, the lesson, the rule.
Warm, simple, moral. Excellent for beginners — the shape does most of the work.

> **Not the same as `STORY_SPINE`.** `ORIGIN_STORY` is a *fairy-tale shape* — the performer is
> told the beats and finds their own words. `STORY_SPINE` is a *script* — the performer is given
> the sentences and must say them. Use `ORIGIN_STORY` to make a round feel gentle; use
> `STORY_SPINE` to force causality. They also differ structurally: `STORY_SPINE` has three
> consecutive consequence beats, `ORIGIN_STORY` has none.

#### `INQUIRY` — the thesis defence
`PRE → CTX → PLN → DAT → PVT → PRB → INS → CTA`
Question, literature, method, findings, the anomaly, the limitation, the implication, further
work. Method comes *before* data — the academic tell. Rewards hedging and precision.

#### `SLOW_BURN` — the contemplation
`PRE → CTX → INS → DAT → PVT → ASP → CTA`
Long pauses, minimal text, at least three `text_weight: none` slides. Trains stillness and the
courage to let a silence run. Shortest arc; caps at 7.

#### `INFOMERCIAL` — the hard sell
`PRE → PRB → PRB → DAT → INS → ASP → PLN → CTA`
Two problem beats back to back (the "but wait, there's worse" structure), then relief, then the
offer. Highest energy. Deliberately the only arc that front-loads the problem.

#### `PRODUCT_LAUNCH` — the keynote reveal
`PRE → CTX → PRB → INS → ASP → DAT → PVT → PLN → CTA`
The category as it stands today, what is wrong with it, **the reveal**, what it makes possible,
the specifications, *one more thing*, availability. The insight lands early — the whole second
half is justification for a decision already announced, which is a different muscle from
`CLASSIC_PITCH`, where the evidence precedes the conclusion.

`PVT` is deliberately placed *late*: that slot is "one more thing", the anomaly produced after
the talk has apparently already finished.

#### `STORY_SPINE` — the Disney story spine
`PRE → CTX → PRB → DAT → PVT → PLN → INS → ASP → CTA`

The eight-step spine used by Pixar's story department and every improv long-form teacher since.
It is the only arc in the system with **named slots**: the performer is handed the sentence, not
just the beat. Theme: `story_spine`. Caps at 9.

| # | Spine step | Beat | What it does |
|---|---|---|---|
| 1 | *Once upon a time…* | `PRE` | Baseline reality. Introduce the character. |
| 2 | *And every day…* | `CTX` | The routine, the habit, the status quo. |
| 3 | *Until one day…* | `PRB` | The inciting incident. The routine breaks. |
| 4 | *And because of that…* | `DAT` | First consequence — the one you can measure. |
| 5 | *And because of that…* | `PVT` | Escalation. Something compounds, and the tone shifts. |
| 6 | *And because of that…* | `PLN` | Complications force a method. The peak approaches. |
| 7 | *Until finally…* | `INS` | The climax. Confrontation, or transformation. |
| 8 | *And ever since then…* | `ASP` | The new normal. |
| — | *…and so I'm asking you for one thing.* | `CTA` | The moral, delivered as a demand. |

**Why nine beats for eight steps.** Step 8 does two jobs — establish the new normal *and* deliver
the moral — and this system requires every arc to land on an ask rather than a summary (§5.4).
Splitting it lets the performer paint the new world on `ASP` and then convert the takeaway into a
demand on `CTA`. The spine's moral survives; it just has to be *asked for*.

**Why the three "because of that" beats are `DAT → PVT → PLN`.** They are the engine of the
spine, and they must not be three of the same thing. A consequence you can *measure*, a
consequence that changes the *tone*, and a consequence that forces a *method* — that is an
escalation rather than a list. They also drop in that order as the deck shortens, so the chain
degrades gracefully:

```
  n=5   Once upon a time · Until one day · And because of that · Until finally · the ask
  n=6   + And ever since then
  n=7   + And every day                    ← the canonical seven-slide spine (default)
  n=8   + a second And because of that
  n=9   + a third   And because of that    ← the full eight-step spine
```

**What it trains.** Causality. On every other arc a performer can start a new idea at each slide
and get away with it. Here they cannot: "and because of that" forbids a new subject, so each
slide has to be justified as the *consequence* of the one before. It is the single best structure
for breaking the "seven separate jokes" habit (§9 of the runbook coaching table), and the
warm cartoon register palette keeps it accessible enough for beginners.

**Teaching note:** make them say the stem out loud, verbatim, before each slide. The sentence
does the work; the performer only has to finish it.

---

## 4. Themes = arc + register palette + length + difficulty curve

A **theme** bundles everything the facilitator needs into one word. Defined in
`scripts/config.py::THEMES`.

Register palettes are draw weights, not quotas — a low weight means *rare*, never *forbidden*.
Every theme keeps a non-zero weight on all six registers so no slide is ever unreachable.

Each theme also carries its **presentation names**, so the CLI, the runsheet and the web app all
say the same thing and no participant ever reads an internal key:

| Field | Purpose |
|---|---|
| `label` | Plain English, shown in the dropdown and on the result card |
| `short` | One or two words; must stay legible written on a spinner wedge |
| `pastel` | Wedge fill in the web app (`accent` stays the deep colour used by deck chrome) |
| `active` | `False` hides it from the wheel, the dropdown and the session spread |

### 4.1 The eight active themes

These are the only formats a participant is offered. `config.active_themes()` is the single
source of that list.

| Theme key | Shown as | Arc | Register palette | Length | Difficulty |
|---|---|---|---|---|---|
| `boardroom` | Board Room Presentation — Annual Report | `CLASSIC_PITCH` | CD 50 · WA 25 · DT 10 · IE 10 · CC 5 · SM 2 | 6–8 | flat 2 |
| `ted_talk` | TED Talk | `HERO_JOURNEY` | IE 35 · DT 30 · CD 20 · SM 10 · CC 5 · WA 3 | 6–10 | flat 3 |
| `ashram` | Motivational Keynote Speaker | `REVELATION` | SM 40 · DT 30 · IE 15 · CD 5 · CC 5 · WA 5 | 6–8 | flat 3 |
| `mission_brief` | Mission Brief | `CRISIS_BRIEF` | CD 40 · IE 25 · WA 20 · DT 15 · SM 8 · CC 4 | 6–8 | flat 3 |
| `thesis_defence` | PhD Thesis & Research Breakthrough | `INQUIRY` | CD 35 · DT 35 · WA 15 · SM 15 · CC 4 · IE 4 | 6–8 | flat 4 |
| `late_night_sale` | Shark Tank — Raising Funds | `INFOMERCIAL` | WA 35 · CC 30 · IE 25 · CD 10 · DT 4 · SM 4 | 6–8 | flat 2 |
| `story_spine` | Story (Disney Format) | `STORY_SPINE` | CC 32 · IE 25 · SM 15 · DT 12 · CD 9 · WA 7 | 6–9 | flat 2 |
| `product_launch` | Product Launch | `PRODUCT_LAUNCH` | IE 40 · CD 22 · WA 15 · CC 10 · DT 8 · SM 5 | 6–9 | flat 2 |

### 4.2 Retired themes (`active: False`)

Kept in `THEMES`, hidden from participants. `--theme <key>` still builds them, existing
`output/` folders still work, and flipping the flag restores them instantly.

| Theme key | Was shown as | Why it went |
|---|---|---|
| `mixed_roulette` | Surprise Me | The wheel already is the roulette |
| `corporate_collapse` | Everything Goes Wrong | Overlaps `boardroom` in practice |
| `fever_dream` | Total Chaos | Advanced-only; kept for experienced rooms |
| `saturday_morning` | Kids' Cartoon Show | Out of register for a corporate-comedy set |
| `slow_burn` | Slow And Thoughtful | Dies in a room that wants pace |

**Effect on the library:** none of the 60 type × register cells becomes unreachable. Simulating
3,200 decks across the eight active themes draws **every rendered slide at least once**. The only
measurable change is exposure: `WA` falls from ~15.5% to ~12.0% of draws and `CC` from ~13.5% to
~11.7%, because the two themes that favoured those registers are the ones retired. `IE` and `CD`
absorb the difference. Re-run that check after any further theme change.

Note that `fever_dream` and `corporate_collapse` share the `DESCENT` arc, which carries
`difficulty_ramp: True` — the ramp comes from the *arc*, so it overrides the theme's flat number
in both. The difference between them is the palette, not the curve.

Any pairing can be forced: `--theme ashram --arc DESCENT` overrides the bundled arc. The length
is clamped to the theme's range *and* the arc's `max`, so `--theme mixed_roulette --arc SLOW_BURN
-n 10` yields 7, not 10.

### Suggested theme ↔ show-genre pairings

Not enforced — just what tends to sing. Blind pairing remains the default.

| Show genre | Natural themes |
|---|---|
| `COR` Corporate | `boardroom` · `corporate_collapse` · `late_night_sale` |
| `ACA` Academic | `thesis_defence` · `ted_talk` · `slow_burn` |
| `SCI` Sci-Fi / Bizarre | `mission_brief` · `fever_dream` |
| `CIV` Civic / Personal | `saturday_morning` · `story_spine` · `boardroom` · `late_night_sale` |
| `SPR` Spiritual / Ritual | `ashram` · `slow_burn` |
| `MED` Media / Sport | `late_night_sale` · `ted_talk` · `saturday_morning` |

`story_spine` is the one theme that pairs well with *any* genre — the spine is a shape of
causality, not of subject matter, so "once upon a time" works as readily for a divorce mediation
as for a cult orientation. Use it when the class needs structure more than it needs tone.

---

## 5. Coherence devices — making nonsense feel inevitable

The arc gives a deck its **shape**. These five devices give it **texture** — the felt sense that
one person made this deck, on purpose, about one thing. Without them, seven ambiguous slides are
seven unrelated images. With them, the same seven slides read as a plausible corporate artefact
that merely happens to be insane.

> **The goal: absurd premises, impeccable logic.** The audience should never think "that's
> random." They should think "I don't understand this company, but it clearly understands
> itself." Nonsense delivered with grammar is comedy. Nonsense delivered as nonsense is noise.

### 5.1 Deck chrome — the single highest-leverage device

At **build time** (not render time) the builder composites a thin, consistent frame onto each
slide copy using Pillow:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│                    [ the slide image ]                         │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│ ◆ HALVERSON DYNAMIC          Project MERIDIAN · Confidential  4/7 │
└────────────────────────────────────────────────────────────────┘
     fake org mark              deck title · classification    slide n/N
```

A 4%-height footer bar in the theme's accent colour, carrying a fake organisation mark, a fake
project codename, a classification stamp and `n/N`. Drawn from
`seed/00_Deck_Chrome_Wordbank.md` — one consistent set per deck, chosen by the deck's RNG seed.

This is the device that does the most work for the least cost. Seven unrelated images with the
same footer, the same accent colour and a running page count **stop looking random immediately**.
It is free, local, instant, and the master assets in `Slide_Assets/` stay clean — chrome exists
only on the deck copies.

`--no-chrome` disables it. Default is on.

### 5.2 Motif threading — the callback engine

Every slide record carries `motifs: ["pigeon", "kevin", "teal", "hexagon"]` — concrete nouns,
recurring names, dominant colours, repeated shapes.

The builder picks **1–2 motifs** for the deck and applies a **1.8× draw weight** to slides
carrying them. Result: a pigeon appears on slide 2 and again on slide 6. Nobody planned it, but
the performer *will* notice, and the callback is the biggest laugh in the set.

This is a soft weight, never a filter — if the motif can only be honoured twice in seven slides,
that is the perfect amount. Three or more and it starts to look like a theme park.
`manifest.json.motifs[]` records which were threaded so the facilitator can point them out in
the note-back afterwards.

### 5.3 Register spine — one design language

The theme's **dominant register supplies ≥ 50% of the deck's slides**. Because every register
maps to a fixed visual system (`config.REGISTER_STYLE` — palette, typography, lighting), the
majority of slides in any deck already share a design language. The minority slides then read as
*deliberate variation* rather than as a different deck entirely.

This is why the `PVT` beat works. A mystic slide in a corporate-deadpan deck is only a shock if
the other six slides agree with each other first.

### 5.4 Escalation — the deck must build

Slide 7 must feel more consequential than slide 2 or the arc has no engine. Two mechanisms:

- **Difficulty curve.** Every theme defines one; `DESCENT` themes ramp 1 → 5 so the deck visibly
  deteriorates while the performer maintains composure.
- **Beat ordering.** Every arc terminates in `CTA` — a *demand*, never a summary. The performer
  is structurally forced to raise stakes rather than trail off. This is the most common failure
  mode in untrained PowerPoint Karaoke and the arc simply removes it.

### 5.5 Transition scaffolds — the connective tissue

`RUNSHEET.md` prints a connective phrase for each beat transition. The performer is not required
to use them; they exist so a beginner is never stranded between slides.

| Transition | Scaffold |
|---|---|
| `PRE → CTX` | "To understand where we are, you have to understand where we've been." |
| `CTX → DAT` | "Now. The numbers." |
| `DAT → PRB` | "Which is exactly why what I'm about to show you is so concerning." |
| `PRB → INS` | "For eight months we got this wrong. Then someone in accounts said something." |
| `INS → PVT` | "I want to pause here, because this next one is difficult for me." |
| `PVT → ASP` | "And that's when I understood what we're actually building." |
| `ASP → PLN` | "So — how." |
| `PLN → CTA` | "Which brings me to what I need from each of you in this room." |
| `→ CTA` (any) | "So I'll leave you with this." |

These are the load-bearing sentences of every real presentation. Drilling them is a transferable
public-speaking skill, smuggled inside a comedy game. **Teaching note:** run one round where
performers must use every scaffold verbatim. It is constraining, it is funny, and it produces
the most coherent decks of the session.

**On `STORY_SPINE` the scaffolds are replaced, not supplemented.** That arc names its own slots
(§3), and its stems — *"And because of that…"* — are already the connective tissue. Printing a
second, different transition beside them would give the performer two contradictory sentences to
say. `config.arc_stem()` wins wherever it returns a value; the table above fills every other
case. Making the whole scaffold round *mandatory* is exactly what `story_spine` is for.

### 5.6 What is deliberately *not* coherent

Coherence is applied to **form only**. Content stays wild:

- No slide is ever chosen for topical relatedness to another slide.
- The show scenario is still drawn blind (§7).
- Motifs are visual, not semantic — a shared colour, not a shared subject.

The performer supplies the meaning. The system supplies only the impression that meaning exists.
That gap is where the entire exercise lives.

---

## 6. Variety constraints (hard rules)

The arc decides *what beat* each slot needs. These rules decide *which slide* fills it. A
candidate deck is rejected and re-drawn (up to 200 attempts) unless all hold:

| # | Rule | Why |
|---|---|---|
| R1 | No two consecutive slides share a `type` | Two charts in a row and the performer just does the chart bit twice |
| R2 | ≥ 4 distinct types (≥3 for n=5) | Forces range: a chart, a face, a word, a shape |
| R3 | ≥ 3 distinct registers | Prevents a uniformly deadpan or uniformly whacky deck |
| R4 | No slide repeats within a session | Tracked in `catalog/history/deck_history.json` |
| R5 | ≤ 2 slides from any one type (≤3 for n≥9) | Stops chart-heavy decks |
| R6 | Mean `difficulty` within ±0.6 of the theme's curve | Beginner decks stay winnable |
| R7 | ≥ 1 slide with `text_weight: none` (≥3 for `slow_burn`) | At least one pure-image slide with nothing to read |
| R8 | ≤ 1 slide of difficulty 5 (exempt: `DESCENT` arcs, which require exactly one, last) | Two hostile slides is demoralising, not instructive |
| R9 | **Every slot's slide must be tagged with that slot's beat** | The arc is a promise; a `DAT` slot gets a slide that can carry data |

**Relaxation order** when the pool is too thin: R6 → R5 → R7 → R3 → R2. **R1, R4, R8 and R9 never
relax** — R9 especially, because breaking it breaks the mental model that is the whole point.
Every relaxation is printed to the console and recorded in `manifest.json.relaxed[]`. If R9
cannot be satisfied, the builder **fails loudly** and names the under-populated beat so you know
exactly what to go ideate more of.

---

## 7. Show pairing modes

| Mode | Flag | Behaviour |
|---|---|---|
| **Blind** *(default)* | — | Show and slides drawn independently. Zero content correlation. |
| **Affinity-biased** | `--affinity` | Slides whose `affinity` list contains the show's genre get **2× draw weight**. Never a hard filter — a `COR` deck must still be able to serve up a mandala. |
| **Anti-affinity** | `--clash` | Inverts the weighting. Maximum difficulty. Advanced only. |
| **Show-free** | `--no-show` | Deck only; the audience calls the scenario live. |

Blind stays the default. The moment slides are *filtered* by scenario, this stops being
PowerPoint Karaoke and becomes "present a deck about your topic." Note the arc gives structure
**without** giving away content — that is exactly the line this design walks.

---

## 8. The draw algorithm

```
INPUT  n (5–10), theme, arc, difficulty, seed, show_mode, session_exclusions

1  POOL   ← slides where status ∈ {rendered, approved}  and  id ∉ session_exclusions
2  RNG    ← Mersenne Twister seeded with `seed`                  # reproducible decks
3  BEATS  ← arc.resolve(n)                                       # ordered list of n beats
4  CURVE  ← theme.difficulty_curve(n)                            # flat, or 1→5 ramp for DESCENT
5  for i, beat in enumerate(BEATS):
       CAND ← { s ∈ POOL : beat ∈ s.beats }
       if CAND is empty → FAIL LOUDLY naming the beat
       WEIGHT(s) ← theme.register_weight[s.register]
                   × (2.0 if affinity-mode and show.genre ∈ s.affinity else 1.0)
                   × (1.4 if s.status == "approved" else 1.0)
                   × gaussian(s.difficulty, CURVE[i], σ=1.2)
                   × (0.0 if s.type == chosen[i-1].type else 1.0)      # R1 inline
       chosen[i] ← weighted_choice(CAND); POOL ← POOL − chosen[i]
6  CHECK  R2–R9. Fail → goto 5 (max 200 attempts, then relax per §6)
7  EMIT   output/<theme>/  ->  manifest.json · RUNSHEET.md
                               NN__<beat>__<PKS-ID>.png  + .prompt.txt + .json
```

Beats are filled **in order** so the R1 adjacency check is a simple inline weight of zero, and so
the difficulty ramp lands on the right slots.

Step 2 makes decks **reproducible**: `--seed 20260816` yields the same deck from the same catalog
every time. Running one deck against three performers and comparing their choices is the single
most useful teaching move this tool enables.

---

## 9. What the performer sees

The folder is named for the **theme**, so a facilitator finds it by feel. Slide files carry their
position and their beat in the filename, so any image viewer plays them in order and the arc is
readable straight off the folder listing:

```
output/boardroom/
├── manifest.json                      deck_id, seed, arc, show, motifs, chrome, slide list
├── RUNSHEET.md
├── 01__PRE__PKS-AC-CD-003.png         + .prompt.txt and .json beside every image
├── 02__CTX__PKS-MP-CD-001.png
├── 03__DAT__PKS-DV-WA-004.png
├── 04__PRB__PKS-PH-WA-002.png
├── 05__INS__PKS-AR-DT-005.png
├── 06__ASP__PKS-QT-IE-001.png
└── 07__CTA__PKS-CT-CD-007.png
```

A second `boardroom` deck becomes `output/boardroom-2/`. Nothing is ever overwritten, and the
whole of `output/` is disposable — delete it and rebuild from the same library any time.

`RUNSHEET.md` opens with the arc map the performer is shown before starting:

```
THEME: boardroom          ARC: CLASSIC_PITCH          7 SLIDES
1 PRELUDE → 2 CONTEXT → 3 DATA → 4 PROBLEM → 5 INSIGHT → 6 ASPIRATION → 7 CALL TO ACTION
"Name it → set the scene → show the numbers → name the trouble →
 have the realisation → paint the promise → make the ask."
```

It does **not** list slide titles. Beats only. The facilitator's copy (below the tear line) has
the full slide IDs for troubleshooting.

---

## 10. Session mode

`05_build_session.sh --performers 12 --slides 7 --theme boardroom` builds a whole class into one
timestamped folder:

```
output/SESSION-20260816-1432/
├── SESSION_RUNSHEET.md      running-order table + facilitator reminders
├── session_manifest.json    seed, every deck's manifest, the no-repeat record
├── boardroom/               performer 1   — its own RUNSHEET.md, print this one for them
├── boardroom-2/             performer 2
└── …                        performer 12
```

- 12 decks, each paired with a **distinct** show, cycling genres so none repeats until all six
  have been used.
- **Global no-repeat:** a slide used in deck 3 cannot appear in decks 4–12. With a 120-slide
  library and 12 × 7 = 84 draws this is tight but workable — and it is the real stress test of
  beat coverage, because the no-repeat rule drains the thinnest beat first. The builder aborts
  with a clear message naming the starved beat rather than silently reusing a slide.
  `--allow-repeat` lifts the constraint if the library genuinely cannot carry the class.
- `--theme-rotate` gives each performer a *different* theme, so the class collectively sees every
  arc — the fastest way to teach the whole vocabulary in one session.
- The running-order table names each performer's **actual folder**, which is what you read off
  when handing out runsheets. Show a performer only their own sheet, and only the top half.

---

## 11. Projection & playback

There are two ways to put a deck on a screen, and **no PowerPoint file**. `.pptx` output was
dropped from the design: it added a dependency, a build step and a file format, and neither of
the two paths below needs it.

1. **The live app** — `./start_slide_app.sh`. Spin for a theme, get a deck compiled in the
   browser, present full screen with the arrow keys, press `N` for a new one. Nothing is written
   to disk. This is the recommended path for most sessions, and the only one that needs no prep.
2. **A theme folder** — `output/<theme>/`. The `01__`, `02__` prefixes mean any OS image viewer
   in slideshow mode plays them in order. Zero dependencies, portable to any venue laptop, works
   with no Python and no browser config. The reliable fallback, and the right choice when the
   venue machine is unknown or you want printed runsheets.

Deck copies are downscaled to 2560px so a folder is quick to build and quick to carry;
`--full-res` copies the 4K masters instead if you are projecting into a very large hall.

**Facilitator rule:** never open the deck folder on the projector before the performance. Preview
on a laptop only. A thumbnail grid destroys the exercise.

---

## 12. Curation loop

Slides earn their place by performing:

```bash
./04_build_deck.sh --approve PKS-DV-WA-004 PKS-SY-SM-002   # worked on stage → 1.4× draw weight
./04_build_deck.sh --retire  PKS-QT-CC-011                 # died / too on-the-nose → out of pool
./04_build_deck.sh --rebeat  PKS-AR-DT-003 INS,ASP,CTA     # re-tag which beats it can serve
./04_build_deck.sh --stats                                 # coverage: type×register AND beat counts

# a re-roll is a Stage-3 job, and always writes a NEW version — v1 stays on disk
./03_generate_slide_images.sh --id PKS-AR-DT-003 --force   # → __v2.png, active_version = 2
./start_slide_app.sh --index-only                          # refresh the app after curating
```

`--stats` reports **beat coverage** as well as the type × register matrix, because a library can
look full and still be unable to build a deck if, say, only four slides can serve `PLN`.
Target: **≥ 12 slides tagged for every beat**, and ≥ 20 for `DAT`, `PRB` and `CTA`.

**The retirement criterion that matters:** if a slide produced roughly the same justification from
three different performers, it is no longer ambiguous. Retire it. Ambiguity is the product.
