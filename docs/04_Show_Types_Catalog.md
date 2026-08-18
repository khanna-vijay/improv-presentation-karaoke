# 04 — Show Types Catalog

**Axis A** of the system: the scenarios a performer must inhabit. Slides know nothing about
these; the pairing is a genuine collision every time (docs/00 §2).

**42 scenarios across 6 genres.** The source of truth is `seed/show_types/SHOWS_<GGG>_*.md`.

To add or change a scenario, edit that Markdown and run `./01_ideate_slides.sh --seed-only` (no
API calls) — it re-parses every show file into `catalog/shows_master.json`, which is what the
deck builder and the app read. **This document is a hand-maintained reading copy, not a generated
artefact**: nothing rewrites it, so update the relevant table below in the same commit or it will
drift. If the two ever disagree, the seed file wins.

---

## Genre codes

| Code | Genre | Count | What it trains |
|---|---|---|---|
| `COR` | Corporate & Professional | 7 | Institutional authority worn badly. The most universally legible genre — everyone has sat through a bad meeting. |
| `ACA` | Academic & Educational | 7 | Sounding like the smartest person in the room while holding nothing. Vocabulary, citation-voice, defensive posturing. |
| `SCI` | Sci-Fi, Fantasy & Bizarre | 7 | Treating impossible things as routine admin. Commitment to a wrong world. |
| `CIV` | Civic & Personal | 7 | Small stakes, enormous feelings. Vulnerability underneath comedy. |
| `SPR` | Spiritual, Ritual & Wellness | 7 | Unearned cosmic authority. Stillness, slowness, the courage to let silence run. |
| `MED` | Media, Sport & Entertainment | 7 | Performing inside the fiction. Permission to be big — the highest-energy genre. |

---

## `COR` — Corporate & Professional

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-COR-01` | **The Crisis Management Press Conference** | A PR representative who has not slept and has been told to "own the narrative" | 2 | `boardroom`, `corporate_collapse`, `mission_brief` |
| `PKC-COR-02` | **Mandatory HR Compliance Seminar** | A thoroughly exhausted HR representative on their fourth delivery today | 1 | `boardroom`, `slow_burn`, `saturday_morning` |
| `PKC-COR-03` | **The Disastrous Exit Interview** | A manager who has never fired anyone and has over-prepared | 3 | `boardroom`, `slow_burn`, `corporate_collapse` |
| `PKC-COR-04` | **Shark Tank Pitch For A Terrible Idea** | A founder with total, unearned, radiant conviction | 2 | `late_night_sale`, `ted_talk`, `fever_dream` |
| `PKC-COR-05` | **The All-Hands Reorganisation Announcement** | A newly-installed executive delivering news they clearly do not understand | 3 | `boardroom`, `ted_talk`, `corporate_collapse` |
| `PKC-COR-06` | **The Quarterly Review That Goes Wrong** | A department head presenting numbers they have not actually read | 4 | `corporate_collapse`, `boardroom` |
| `PKC-COR-07` | **Onboarding Day One For The New Hire** | An over-caffeinated culture lead who genuinely loves this company | 1 | `saturday_morning`, `boardroom`, `ashram` |

## `ACA` — Academic & Educational

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-ACA-01` | **The Ph.D. Thesis Defence** | A grad student who has been awake for two days and is defending nine years of … | 3 | `thesis_defence`, `slow_burn`, `corporate_collapse` |
| `PKC-ACA-02` | **The Uncomfortable Middle School Assembly** | A deeply out-of-touch guest speaker hired to be "relatable" | 2 | `saturday_morning`, `ted_talk`, `fever_dream` |
| `PKC-ACA-03` | **The Pretentious Art Gallery Tour** | A curator whose taste is a personality and whose personality is a weapon | 3 | `ashram`, `ted_talk`, `slow_burn` |
| `PKC-ACA-04` | **The PTA Curriculum Proposal** | An overzealous PTA president with a laminated agenda and a grievance | 2 | `boardroom`, `late_night_sale`, `saturday_morning` |
| `PKC-ACA-05` | **Substitute Teacher, No Lesson Plan** | A substitute who found this deck on the desktop and is committing to it | 2 | `mixed_roulette`, `saturday_morning`, `fever_dream` |
| `PKC-ACA-06` | **The Keynote At A Conference You Do Not Belong At** | An academic who accepted the invitation without checking the field | 4 | `ted_talk`, `thesis_defence`, `corporate_collapse` |
| `PKC-ACA-07` | **The Grant Renewal Panel** | A researcher whose lab dies in six weeks without this money | 4 | `thesis_defence`, `boardroom`, `ted_talk` |

## `SCI` — Sci-Fi, Fantasy & Bizarre

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-SCI-01` | **The Supervillain's Master Plan Reveal** | A Bond-style villain who has waited eleven years for this meeting | 2 | `mission_brief`, `corporate_collapse`, `boardroom` |
| `PKC-SCI-02` | **Cult Initiation Orientation** | A charismatic leader with warm eyes and no blinking | 3 | `ashram`, `ted_talk`, `slow_burn` |
| `PKC-SCI-03` | **Secret Agent Mission Briefing** | A stern intelligence director who has already lost three teams to this target | 2 | `mission_brief`, `boardroom` |
| `PKC-SCI-04` | **The Time Traveller's Warning** | A panicked traveller from 2085 who has thirty minutes of battery left | 3 | `fever_dream`, `corporate_collapse`, `mission_brief` |
| `PKC-SCI-05` | **Interspecies Diplomatic Address** | Earth's hastily appointed ambassador, who was a regional sales manager on Mond… | 4 | `mixed_roulette`, `fever_dream`, `boardroom` |
| `PKC-SCI-06` | **The Haunted Property Disclosure** | An estate agent legally obliged to disclose, but commercially motivated not to | 2 | `boardroom`, `saturday_morning`, `late_night_sale` |
| `PKC-SCI-07` | **The Simulation Maintenance Update** | A mid-level technician from outside reality, delivering a scheduled service no… | 4 | `slow_burn`, `ashram`, `boardroom` |

## `CIV` — Civic & Personal

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-CIV-01` | **The Aggressive HOA Meeting** | A petty board member who has been documenting this for fourteen months | 1 | `boardroom`, `late_night_sale`, `corporate_collapse` |
| `PKC-CIV-02` | **The Unprepared Best Man** | A deeply emotional friend who is holding a microphone and a drink | 2 | `saturday_morning`, `ted_talk`, `corporate_collapse` |
| `PKC-CIV-03` | **City Council Open Mic Proposal** | A local eccentric who has waited nine weeks for these five minutes | 2 | `late_night_sale`, `mixed_roulette`, `ted_talk` |
| `PKC-CIV-04` | **The Family Intervention** | The sibling who organised this and printed the handouts | 3 | `ted_talk`, `slow_burn`, `boardroom` |
| `PKC-CIV-05` | **The Neighbourhood Watch Annual Report** | A volunteer coordinator with a hi-vis vest and unlimited time | 1 | `boardroom`, `saturday_morning`, `mission_brief` |
| `PKC-CIV-06` | **The Divorce Mediation Opening Statement** | A spouse who has hired a consultant to prepare their side of the asset split | 4 | `boardroom`, `slow_burn`, `corporate_collapse` |
| `PKC-CIV-07` | **The Retirement Send-Off Speech** | A colleague who did not know the leaver well and was asked this morning | 2 | `saturday_morning`, `ted_talk`, `boardroom` |

## `SPR` — Spiritual, Ritual & Wellness

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-SPR-01` | **The Silent Retreat Welcome Briefing** | A retreat facilitator delivering the final words anyone will hear for ten days | 3 | `ashram`, `slow_burn` |
| `PKC-SPR-02` | **The Wellness Startup Investor Deck** | A founder who has fused venture capital and enlightenment into one voice | 2 | `ted_talk`, `ashram`, `boardroom` |
| `PKC-SPR-03` | **The Ancestral Prophecy Unveiling** | The keeper of a lineage, revealing what the elders sealed | 3 | `ashram`, `mission_brief`, `slow_burn` |
| `PKC-SPR-04` | **The Corporate Mindfulness Mandatory Session** | An external wellness consultant hired after a very bad employee survey | 3 | `ashram`, `boardroom`, `corporate_collapse` |
| `PKC-SPR-05` | **The Guru's Annual Address To The Devoted** | A teacher whose following has grown faster than their teaching | 4 | `ashram`, `ted_talk` |
| `PKC-SPR-06` | **The Funeral Eulogy, Wrong Person** | A speaker who realises three slides in that they have prepared for a different… | 5 | `slow_burn`, `corporate_collapse`, `ashram` |
| `PKC-SPR-07` | **The Astrology App Product Launch** | A product lead who genuinely believes and also has KPIs | 2 | `late_night_sale`, `ted_talk`, `boardroom` |

## `MED` — Media, Sport & Entertainment

| ID | Scenario | You are | Diff | Natural themes |
|---|---|---|---|---|
| `PKC-MED-01` | **The Post-Match Press Conference After A Historic Defeat** | A manager whose team has just lost in a way that will be shown for decades | 2 | `boardroom`, `corporate_collapse`, `mission_brief` |
| `PKC-MED-02` | **The Reality Show Contestant Elimination Defence** | A contestant given one final chance to argue for their survival, with visual a… | 2 | `late_night_sale`, `ted_talk`, `fever_dream` |
| `PKC-MED-03` | **The Film Pitch To A Studio That Has Read Nothing** | A writer-director with one shot and nine years of unmade scripts | 2 | `late_night_sale`, `ted_talk`, `fever_dream` |
| `PKC-MED-04` | **The Fan Convention Lore Panel** | A franchise loremaster explaining canon to people who know it better | 3 | `thesis_defence`, `mission_brief`, `saturday_morning` |
| `PKC-MED-05` | **The Influencer Apology Video** | A creator in a grey hoodie who has never been more sincere | 2 | `ted_talk`, `corporate_collapse`, `late_night_sale` |
| `PKC-MED-06` | **The Olympic Bid Presentation** | A city's delegate lead, presenting the bid to the international committee | 3 | `ted_talk`, `boardroom`, `mission_brief` |
| `PKC-MED-07` | **The Late-Night Shopping Channel Segment** | A host who has been on air for six hours and believes completely | 1 | `late_night_sale`, `saturday_morning`, `fever_dream` |

---

## How scenarios are used

| Mode | Flag | Behaviour |
|---|---|---|
| **Blind** *(default)* | — | Show and slides drawn independently. Zero content correlation. |
| Affinity-biased | `--affinity` | Slides whose `affinity` list contains the show's genre get 2× draw weight. Never a hard filter. |
| Anti-affinity | `--clash` | Inverts the weighting. Maximum difficulty. |
| Show-free | `--no-show` | Deck only; the audience calls the scenario live. |

Blind is the default deliberately. The moment slides are *filtered* by scenario this stops
being PowerPoint Karaoke and becomes "present a deck about your topic".

**Session mode** (`./05_build_session.sh`) cycles genres so no genre repeats until all six
have been used — a class of 12 sees every genre twice, in a different order each run.

## Difficulty guide

| # | Scenario feels like |
|---|---|
| 1 | The frame does the work. Beginners land it. |
| 2 | One clear character choice carries the set. |
| 3 | Requires committed status work. |
| 4 | High-status bluffing under scrutiny. |
| 5 | A high-wire recovery premise. Advanced only. |

Scenario difficulty and *deck* difficulty are independent knobs. A difficulty-1 scenario with a
`fever_dream` deck is a perfectly good advanced exercise, and vice versa.
