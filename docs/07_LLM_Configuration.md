# 07 — LLM Configuration

All models run on **Google Cloud Vertex AI** using local **Application Default Credentials**.
No API keys anywhere in this project.

Authoritative source: `C:\_VK_Code\____00_Re-Usable_Code_Assets\Models to be Used.txt`

```
project  = YOUR_GCP_PROJECT
location = global
auth     = gcloud ADC   (gcloud auth application-default login)
```

Override any value in `.env` (see `sample.env`). Code reads it via `scripts/config.py`.

---

## 1. Model assignment

| Stage | Role | Model ID | SDK |
|---|---|---|---|
| 1 — Ideate | Divergent volume: 120+ slide concepts | `gemini-3.7-flash` | `google-genai` (`vertexai=True`) |
| 2 — Elaborate | Convergent judgement: art direction, text discipline | `claude-opus-5` | `anthropic.AnthropicVertex` |
| 3 — Render | 4K 16:9 slide images with legible typography | `gemini-3-pro-image` | `google-genai` (`vertexai=True`) |

**Why this split.** Stage 1 is a breadth problem — hundreds of concepts, cheap, fast, high
temperature. Flash is the right tool. Stage 2 is a precision problem — enforce an 8-word cap,
detect and remove self-contained punchlines, produce art direction a renderer can actually
follow. That is Opus 5's strength and it runs on far fewer tokens. Stage 3 is the only model that
can render legible embedded text at 4K.

> **Image model ID.** At `location=global` the served ID is **`gemini-3-pro-image`**. The
> `-preview` suffix 404s on this project. Pinned in `scripts/config.py` and `sample.env`; if
> renders suddenly start 404-ing, check this first.

---

## 2. Stage 1 — Gemini 3.7 Flash

```python
model = "gemini-3.7-flash"

GenerateContentConfig(
    temperature       = 1.15,          # high: we want divergence, not correctness
    top_p             = 0.98,
    max_output_tokens = 65535,
    response_mime_type = "application/json",
    response_schema    = IDEA_BATCH_SCHEMA,      # structured output, no scraping
    system_instruction = prompts/ideation_system.md,
    thinking_config    = ThinkingConfig(thinking_level="MEDIUM"),
    safety_settings    = config.SAFETY_THRESHOLDS,   # NOT all-OFF — see below
)
```

- **No tools.** Google Search grounding is deliberately *disabled* here: grounding pulls ideas
  toward real, existing things, which is the opposite of what this stage needs.
- **Safety settings are ON**, per category, from `config.SAFETY_THRESHOLDS`. They were `OFF`
  on all four categories in the original build; that is no longer acceptable, because the library
  must stay child-friendly (`00` §4.1). `SEXUALLY_EXPLICIT` and `HATE_SPEECH` block at
  `BLOCK_MEDIUM_AND_ABOVE` — those are hard requirements and a false positive costs one retry.
  `DANGEROUS_CONTENT` and `HARASSMENT` sit at `BLOCK_ONLY_HIGH`, which is the loosening the
  absurdist content genuinely needs: stricter thresholds spuriously reject stage directions like
  "COMMENCE THE SHOUTING PHASE". Override per category in `.env`
  (`SAFETY_HATE_SPEECH=BLOCK_LOW_AND_ABOVE`, …).
- These thresholds are the **third** of four enforcement layers, not the first. The prompt files
  do the real work; this is the backstop. See `00` §4.1.
- **`thinking_level: MEDIUM`** — enough to honour the constraint list, not so much that 10 calls
  get slow.
- With `--fill-cells K` it is one call per *deficient cell*, not per type — so a full first
  run is up to 60 calls and every later run is only as large as the gap. `-n N --type TT` still
  does one call per type when you want a bulk top-up.

---

## 3. Stage 2 — Claude Opus 5

```python
AnthropicVertex(project_id="YOUR_GCP_PROJECT", region="global")

client.messages.stream(
    model        = "claude-opus-5",
    max_tokens   = 32000,
    thinking     = {"type": "adaptive"},
    output_config= {"effort": "high"},
    system       = prompts/elaboration_system.md,
    messages     = [{"role": "user", "content": <batch of 8 ideas>}],
)
```

- **Streaming is mandatory.** Non-streaming requests hit a 10-minute server timeout; with
  adaptive thinking on a batch of 8, that limit is reachable.
- **`effort: high`** — this stage is where quality is won or lost.
- Batch size 8 (`ELABORATION_BATCH`) → ~15 calls for a 120-slide library.
- **`effort` is clamped** to `{low, medium, high}` in `llm_clients.py`. A shell exporting
  `CLAUDE_EFFORT=xhigh` would otherwise be passed straight through and rejected by the API
  partway into a long unattended run.

---

## 4. Stage 3 — Gemini 3 Pro Image

```python
model = "gemini-3-pro-image"

GenerateContentConfig(
    temperature        = 1.0,
    top_p              = 0.95,
    max_output_tokens  = 32768,
    response_modalities= ["IMAGE"],
    system_instruction = prompts/image_system.md,
    image_config = ImageConfig(
        aspect_ratio     = "16:9",
        image_size       = "4K",
        output_mime_type = "image/png",
    ),
    safety_settings    = config.SAFETY_THRESHOLDS,   # NOT all-OFF — see sec 2
)
```

- **16:9 / 4K** — every projector is 16:9; 4K survives a large hall and future re-crops.
  `--size 2K` halves cost and disk and is still above 1080p.
- **`response_modalities: ["IMAGE"]`** only. We do not want a text commentary alongside.
- One call per slide → **120 calls** for a full library at 2 per cell. At roughly 2 minutes per
  image that is ~4 hours wall-clock. This is the expensive stage; use `--limit` to render in
  tranches and `nohup … &` to run it unattended.

### Getting legible on-slide text

Gemini 3 Pro Image renders text well *when told precisely what to render*. The Stage-2 prompt
therefore always specifies, verbatim:

> Render the following text exactly, spelled exactly as written, and no other words anywhere in
> the image: headline `"COMMENCE THE SHOUTING PHASE"` centred, 12% cap-height, white on red.

Free-form instructions like "a slide about shouting" produce garbled pseudo-text. The explicit
quoted-string pattern is the difference between a usable slide and a reject.

---

## 5. Retry & throttling policy

Identical across all three stages (`llm_clients.py`):

| Setting | Default | `.env` key |
|---|---|---|
| Max attempts | 12 | `MAX_RETRIES` |
| Base delay | 5 s | `BASE_DELAY` |
| Backoff | exponential ×2 with ±25% jitter | — |
| Delay cap | 300 s | `MAX_DELAY` |
| Inter-call pause | 4 s | `INTER_CALL_DELAY` |
| Min valid PNG | 40 KB | `MIN_IMAGE_SIZE_KB` |

Retry triggers: `429`, `RESOURCE_EXHAUSTED`, `quota`, `rate limit`, `throttl`, `503`,
`unavailable`, plus any transport error. `KeyboardInterrupt` is re-raised immediately, never
swallowed by the broad handler.

---

## 6. Cost envelope (full 120-slide build, order of magnitude)

| Stage | Calls | Notes |
|---|---|---|
| 1 — Flash ideation | ≤60 | One per deficient cell. Small in, small JSON out. Negligible. |
| 2 — Opus 5 elaboration | ~15 | Thinking-heavy but short outputs. Modest. |
| 3 — Gemini 3 Pro Image | 120 | **Dominant cost.** 4K images, ~2 min each. |
| 4/5 — Deck assembly | 0 | Local only. Free and instant, forever. |

The economics are the point of the architecture: the expensive stage runs **once**. After that,
every practice deck for every class is a free local draw from the same repository.

**Cost-control levers:** `--limit N` (render in tranches) · `--size 2K` · `--type <TT>` (build one
type at a time) · `--dry-run` (validate prompts before spending).

---

## 7. Authentication

```bash
gcloud auth application-default login       # once per machine
gcloud config set project YOUR_GCP_PROJECT
```

`00_setup_check.sh` pre-warms the token before any stage. Under WSL the first ADC call after
idling can hit an OAuth read-timeout; pre-warming eliminates it.

`llm_clients.py` explicitly `os.environ.pop`s `GOOGLE_API_KEY` and `GEMINI_API_KEY` at client
init, so a stray key in the shell can never silently switch the SDK off the Vertex path.

---

## 8. Prompt assets

Behaviour is tuned by editing Markdown, never Python:

| File | Controls |
|---|---|
| `prompts/ideation_system.md` | What makes a good karaoke slide; the 5 principles; the quality gate |
| `prompts/ideation_user.md` | Per-type template: `{{TYPE_*}}`, `{{SEEDS}}`, `{{AVOID}}`, `{{N}}`, `{{REGISTER_TARGETS}}` |
| `prompts/elaboration_system.md` | Art-direction discipline, 8-word cap, punchline removal, register style bible |
| `prompts/elaboration_user.md` | Batch template: `{{IDEAS_JSON}}` |
| `prompts/image_system.md` | Standing renderer rules: projector-safe contrast, exact spelling, no watermarks/signatures |
| `prompts/custom_image_prompts.md` | Hand-written prompt blocks for `08_build_images_from_prompts.sh` — not an LLM instruction, an input |

Every rendered request payload is archived to `prompts/runs/<ts>__<stage>__<model>.json`, so any
image in the library can be traced to the exact prompt and model settings that produced it.
