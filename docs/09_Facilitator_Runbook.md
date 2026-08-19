# 09 — Facilitator Runbook

How to actually run this in a room. Everything below assumes the library is rendered; if it
isn't, see `08_Master_Plan_and_Roadmap.md` §"Phase 4".

---

## 1. Two ways to run a class

### A. Live, from the browser — *recommended for most sessions*

```bash
./start_slide_app.sh
```

Opens the **deck randomiser**. Press **SPIN THE WHEEL** → it lands on a talk format → a brand-new
deck is compiled on the spot from the building-block library → press **Start presenting** and
drive it with the arrow keys.

**There is exactly one control on the opening screen.** Format, deck length and scenario are all
randomised the moment the page loads, so you can open the file in front of a room and press one
button. Everything else lives behind three icons in the top-right:

| Icon | What is behind it |
|---|---|
| **⌗** | The source repository on GitHub. MIT licensed — code *and* images. |
| **⚙** | Format · deck length · scenario · seed · session variety, then **Apply & rebuild deck**. |
| **?** | The rules, how to play, the opening card, the corner cue, coaching notes, the key map. |

#### The result card carries two things

The format, and the **shape** that format follows — *Name it → set the scene → show the numbers →
name the trouble → have the realisation → paint the promise → make the ask.* Read the shape out
to the performer. It is the only help they get, and it is what stops the talk becoming a list.

There is **no deck preview, for anybody, including you.** The old scenario block, beat table and
thumbnail strip are gone on purpose: you cannot spoil a slide you have not seen, and the person
driving the laptop reacting honestly is half the fun for the room.

#### One button does everything

There is no separate shuffle. **Every spin re-rolls the format, the premise and the slides
together.** Landing on the same format twice still deals a completely different deck. Spin
between performers; nothing else is needed.

#### Session variety

The app remembers what it has already dealt. Slides used in the **last 4 decks are held back**
from the next draw, so the same image does not turn up two rounds running. Without it, a 40-deck
session against the 180-slide library repeats about 21 slides in back-to-back decks; with it,
none. It relaxes automatically if a run is long enough to need it, is capped at 45% of the
library, and is **switched off whenever you type a seed** — a seed must reproduce its deck
exactly. *Reset session history* in **⚙** clears it when a new group takes over the laptop.

Best for: regular classes, jams, drop-ins. Zero prep, infinite decks, nothing written to disk.

| Key | Action |
|---|---|
| `←` `→` | previous / next slide |
| `F` | toggle fullscreen |
| `B` | beat overlay (facilitator only — shows the beat and the transition line) |
| `N` | compile a new deck, same theme |
| `Esc` | back to the lobby |
| `S` | **on the lobby** — show the scenario, for you to read aloud |
| `N` | **on the lobby** — spin the wheel |

#### What the performer sees on screen

Every deck is **opening card → 8–12 images → closing card**:

1. **The opening card**, always page 1 — the words *Presenter Name*, *Accomplishments &
   Credentials*, *Venue* and *Topic* over four blank lines. Nothing is filled in. The performer
   says who they are, what they have supposedly achieved, where they are and what they are about
   to talk about, inventing all four out loud. It gives them a running start before the first
   image lands, and it is the same card in every format.
2. **The corner cue** — one small word, bottom-right, on every slide: *Prelude*, *Context*,
   *Data*, *Problem*, *Insight*, *Pivot*, *Aspiration*, *Plan*, *Personal Story*,
   *Call to Action*. It tells them what the slide is **for**, never what it shows.
   **On by default.** Untick *Slide assistant text* on the spinner screen to hide it — and the
   on-screen key reminders with it — once the group no longer needs the safety net.
3. **The closing card**, always last — *Thank You for your Valuable Time* over an image the deck
   never used, so the performer signs off in front of a picture they never got to explain. Do not
   let them explain it. Take two audience questions in character instead; that is usually where
   the best material is.

There is no branded chrome bar, no slide numbers and no title text over the image. The picture
carries the screen and the performer carries the room.

The **scenario** is never on the card and never projected. Press **S** on the lobby to open it,
read it to the performer, and close it again. It is optional — plenty of groups prefer to have
the audience shout a premise out instead (**⚙ → Scenario → No scenario**). Picking one by hand
*pins* it, so later spins stop re-rolling it.

### B. Pre-built folders — for a planned workshop or a strange venue

```bash
./04_build_deck.sh --theme boardroom -n 7        # one deck  → output/boardroom/
./05_build_session.sh --performers 12 --slides 7 # a class   → output/SESSION-<date>-<time>/
```

Writes self-contained folders under `output/`: numbered 16:9 images that play in order in any OS
image viewer, plus a `RUNSHEET.md` per deck. No PowerPoint file, no Python on the venue machine,
no browser config. Best when the venue machine is unknown, when you want printed runsheets, or
when you want the *same* deck run by three performers for comparison (`--seed 20260816`).

A session folder holds one sub-folder per performer (`boardroom`, `boardroom-2`, …) and a
`SESSION_RUNSHEET.md` whose running-order table names each performer's folder. Print each
performer their **own** deck's `RUNSHEET.md`, and only the half above the tear line.

---

## 2. The 90-minute class

| Time | Segment | Notes |
|---|---|---|
| 0:00 | **Frame the game** | "You will present a deck you have never seen. You *will* know its shape." |
| 0:05 | **Teach the arc** | Draw `PRE → CTX → DAT → PRB → INS → ASP → CTA` on the board. Leave it up all session. |
| 0:12 | **Demo** | Facilitator runs one deck badly, then the same deck well. Difference = committing to the first idea. |
| 0:22 | **Round 1 — `saturday_morning`, 5 slides** | Easiest theme, difficulty 1. Everyone succeeds. |
| 0:45 | **Note-back** | One note per performer. Only about *justification* and *transitions*. |
| 0:55 | **Round 2 — `story_spine`, 7 slides** | Compulsory verbatim stems. Teaches causality; kills the seven-separate-jokes habit. Swap for `boardroom` / `ted_talk` if the group already builds one story. |
| 1:20 | **Round 3 (optional) — `fever_dream` or `corporate_collapse`** | Advanced only. Composure under collapse. |
| 1:30 | **Close** | Retire any slide that got the same reading three times. |

---

## 3. Choosing a theme

| The class needs… | Theme | Why |
|---|---|---|
| A safe first round | `saturday_morning` | Difficulty 1, warm cartoon register, 5–7 slides, the arc does the work |
| **To stop telling seven separate jokes** | **`story_spine`** | **The Disney story spine. Every slide must be justified as a *consequence* of the last — "and because of that…" is printed on the runsheet and they must say it.** |
| Straight comedy | `boardroom` | Everyone recognises a bad quarterly review |
| Status work | `ted_talk` | `HERO_JOURNEY` arc rewards vulnerability and self-mythology |
| To slow people down | `slow_burn` / `ashram` | Minimum three wordless slides. Trains stillness. |
| To break confident players | `fever_dream` | Nothing is stable, and difficulty ramps 1→5 |
| Composure under pressure | `corporate_collapse` | Difficulty ramps 1→5 across the deck. Starts credible, ends unhinged. |
| Big physical energy | `late_night_sale` | `INFOMERCIAL` arc, two problem beats back to back |
| Precision and hedging | `thesis_defence` | `INQUIRY` arc — method *before* data |

Full table: `05_Slide_Use_Method_Deck_Assembly.md` §4.

### Running a `story_spine` round

This is the one theme where the performer is handed **sentences**, not just a shape. The runsheet
prints the spine down the left-hand side:

```
  1  Once upon a time...          5  Until finally...
  2  And every day...             6  And ever since then...
  3  Until one day...             7  ...and so I'm asking you for one thing.
  4  And because of that...
```

Rules for the round:

1. **The stem is compulsory and verbatim.** Say it out loud, *then* look at the slide, then
   justify. Saying it first is what forces the causal link; saying it after turns it into
   narration.
2. **No new subjects.** "And because of that" forbids starting a fresh idea. Whatever is on the
   slide has to be a consequence of the previous slide, however hard that is. That difficulty is
   the entire exercise.
3. **`-n 9` gives the full eight-step spine** with all three "and because of that" escalations.
   `-n 7` (the default) is the classic seven-slide version and the right size for a first pass.
   Below 7, the middle escalations drop out first and it becomes a straight three-act shape.
4. **The note afterwards is always the same one:** where did they break the chain? Almost every
   performer breaks it exactly once, and they know precisely where.

Best follow-up: run `boardroom` immediately after and watch them keep the causality. That
transfer is the whole point of teaching the spine.

---

## 4. The rules that actually matter on the night

1. **Never open a deck folder on the projector before the set.** A thumbnail grid destroys the
   game. Preview on a laptop, present from the app or from slideshow mode.
2. **Give them a physical clicker.** A TV remote, a marker, anything. The physical click is a
   beat to breathe and transition — it is the single biggest quality improvement available.
3. **Show the arc, hide the slides.** The performer sees the beat map before they start. That is
   the design: known skeleton, unknown flesh.
4. **Hold them to the ask.** Every arc ends on `CTA`. If a performer summarises instead of
   demanding something, that is the note.
5. **Character over cleverness.** If a justification dies, the *character's reaction* is the
   material. An arrogant tech-bro claims the confusing slide was their genius idea; a nervous
   intern apologises for the typo. Both work. Explaining does not.
6. **Do not rescue them.** The recovery is the lesson.

---

## 5. Coaching notes, in the order they usually come up

| What you see | The note |
|---|---|
| Reading the slide aloud | "We can all read. Tell us what it *means* to you." |
| Hedging — "I think this might be…" | "You made this deck. You know what it is." |
| Each slide is a separate joke | "Callback slide 2 on slide 5. Build one story." — then put them on `story_spine`, where the deck won't let them. |
| Running out of steam by slide 4 | "Plan two beats ahead. You know a problem slide is coming." |
| Trailing off at the end | "Land on an ask. Demand something from the room." |
| Panic at the pivot slide | "That's the gear-change. Let it change you, then continue." |
| Same energy for 7 slides | "Where's the quiet one? Give us a slow slide so the loud ones land." |

**The transition scaffolds** in every RUNSHEET (`"Which is exactly why what I'm about to show you
is so concerning."`) are a genuine drill. Run one round where performers must use every scaffold
verbatim. It is constraining, it is funny, and it produces the most coherent decks of the session.
`--theme story_spine` is that drill made mandatory: its scaffolds *are* the arc.

---

## 6. Scaling difficulty without changing the deck

Same library, harder game:

- `--difficulty 4` — biases toward slides that resist interpretation
- `--clash` — deliberately pairs the scenario with the *least* fitting slides
- `-n 10` — a longer arc; stamina and structure
- No scenario (`--no-show`) — the audience calls it live, after the first slide is already up
- **Two performers, one deck** — they alternate slides and must accept each other's premises
- **Silent round** — the performer may not speak; only gesture and the clicker
- `--theme story_spine -n 9` — the full nine-slot spine. Three consecutive "and because of that"
  beats is a genuinely hard causal chain to hold, and it is hard in a *constructive* way rather
  than a hostile one — the difficulty is structural, not in the slides.
- **Seeded repeat** — `--seed 20260816` run by three performers back to back. The most
  instructive twenty minutes in the whole exercise: same slides, three completely different decks.

---

## 7. After the session — curate

Slides earn their place on stage.

```bash
./04_build_deck.sh --approve PKS-DV-CD-001 PKS-SY-SM-001   # worked → 1.4× draw weight
./04_build_deck.sh --retire  PKS-QT-CC-011                 # died / became predictable
./04_build_deck.sh --rebeat  PKS-AR-DT-001 INS,ASP,CTA     # re-tag if it never places
./04_build_deck.sh --stats                                 # coverage before the next class
```

**The retirement criterion:** if three different performers gave roughly the same justification
for a slide, it is no longer ambiguous. Retire it. Ambiguity is the product.

After curating, refresh the web app index:

```bash
./start_slide_app.sh --index-only
```

---

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `beat starvation: no usable slide can serve PLN` | Library too thin for that beat | `./01_ideate_slides.sh --type DG MP AC` → `./02_elaborate_prompts.sh` → `./03_generate_slide_images.sh`. Cheaper first: `./04_build_deck.sh --rebeat <PKS-ID> PLN,INS` on under-tagged slides. |
| "Some variety rules were relaxed" warning | Small rendered pool | Render more slides; the deck is still usable |
| App shows "library.js is missing" | Index not built | `./start_slide_app.sh --index-only` |
| App loads but no images | Opened over `file://` from a different folder | Run `./start_slide_app.sh` (serves over http) |
| Renders 404 | Wrong model ID for the region | `gemini-3-pro-image` at `location=global` — no `-preview` suffix |
| ADC / OAuth timeouts in WSL | Cold token | `./00_setup_check.sh` pre-warms it; or `gcloud auth application-default login` |
| Deck folder is huge | 4K masters, 15–24 MB each | Deck copies are already downscaled to 2560px — check you did not pass `--full-res`. To make folders free: `./04_build_deck.sh --no-chrome --link` (hard-links; chrome must be off). To shrink the masters themselves: `./03_generate_slide_images.sh --size 2K` |
