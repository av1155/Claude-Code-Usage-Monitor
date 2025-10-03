Here’s a straight, evidence-based read on the repo + roadmap and whether it’s doable.

# What the ROADMAP claims — and reality check

1. “OpenCode Storage Location: `~/.local/share/opencode/storage/`”
   **Almost right, but incomplete.** OpenCode stores data under `~/.local/share/opencode/`. Session/message JSON lives under a `storage/` directory, but _where_ depends on version and whether you’re in a Git repo: current docs say `~/.local/share/opencode/project/<project-slug>/storage/` (or `.../global/storage/` when not in a repo). There are also recent issues showing paths like `~/.local/share/opencode/storage/message/...` on some setups. So your parser should check for both layouts. ([opencode.ai][1]) ([GitHub][2])

2. “Message file format includes `modelID`, `providerID`, `tokens` (input/output/reasoning/cache) and `cost`.”
   **Doable and confirmed.** A live issue shows the exact JSON the roadmap describes (with `tokens.input`, `tokens.output`, `tokens.reasoning`, `cache.{write,read}`, and `cost`). That’s exactly what you need to parse. ([GitHub][3])

3. “Multi-provider support (OpenAI/Anthropic/Google) with pricing + reasoning tokens (o1).”
   **Doable, with a static pricing table and a few caveats.**
   • Anthropic: official pages document per-MTok prices (e.g., Sonnet 4/4.5 at $3/$15 MTok), and note variations above 200K context; so a static map is viable. ([Anthropic][4]) ([Anthropic][5])
   • OpenAI: o-series models include **reasoning tokens** in usage, which you’ll see in usage metadata; pricing is published and can be mapped in code. ([OpenAI Platform][6]) ([OpenAI Platform][7]) ([OpenAI][8])
   • Google Gemini (Vertex AI): pricing is public; token billing exists per model family and GA notes show pricing adjustments—suitable for a static table you refresh occasionally. ([Google Cloud][9]) ([Google Cloud][10])

4. “Calculate costs when `message.cost = 0`.”
   **Required and doable.** There’s an open issue showing `cost: 0` in stored messages; you can recompute from token counts + your pricing table. ([GitHub][3])

5. “Detect provider/model (e.g., `providerID`, `modelID`) for pricing lookup.”
   **Doable and confirmed.** Multiple logs/issues show `providerID`/`modelID` flowing through OpenCode, so you can read them from the message/session JSON. ([GitHub][11]) ([GitHub][12]) ([GitHub][13])

6. “Platform auto-detection; CLI flag (`--platform opencode`).”
   **Doable.** You can inspect standard OpenCode locations (`~/.local/share/opencode/`) to detect presence; docs also pin credential/config locations that are reliable indicators. ([opencode.ai][1]) ([opencode.ai][14]) ([opencode.ai][15])

7. “Timestamp format: ms since epoch.”
   **Likely correct.** OpenCode stores rich JSON and logs in ISO/MS epoch styles; your example is ms-epoch and aligns with what we see in logs and stored JSON conventions. (And you’re parsing as ints anyway.) ([GitHub][16])

---

# Concrete examples you can implement today

## A) Find OpenCode sessions/messages on disk

Do both to cover versions/layouts (and pass `hours_back` later):

```python
# pseudo-logic
candidates = [
  "~/.local/share/opencode/project/*/storage/message/*.json",  # per-project
  "~/.local/share/opencode/global/storage/message/*.json",     # no git repo
  "~/.local/share/opencode/storage/message/*.json"             # older/alt layout seen in the wild
]
```

Why this works: Docs define the `project/…/storage/` layout, and issues show real-world installs writing to `~/.local/share/opencode/storage/message/…`. You’ll parse whatever exists. ([opencode.ai][1]) ([GitHub][2])

## B) Parse what OpenCode writes

The JSON below is from _real OpenCode output_ (trimmed), so your schema is right:

```json
{
    "id": "prt_x",
    "messageID": "msg_x",
    "sessionID": "ses_x",
    "type": "step-finish",
    "tokens": {
        "input": 26535,
        "output": 1322,
        "reasoning": 0,
        "cache": { "write": 0, "read": 0 }
    },
    "cost": 0
}
```

You’ll also see `providerID` and `modelID` in logs/records for mapping—e.g., `providerID:"anthropic", modelID:"claude-sonnet-4-20250514"`. ([GitHub][3]) ([GitHub][11])

## C) Compute cost when `cost` is missing/zero

Load a static table (refresh occasionally):

```python
PRICING = {
  "anthropic/claude-sonnet-4.5": {"in": 3, "out": 15},                   # $/MTok
  "anthropic/claude-sonnet-4":   {"in": 3, "out": 15},                   # docs/news
  "openai/o1-pro":               {"in": 150, "out": 600},                # reasoning model
  "openai/gpt-4.1":              {"in": 5, "out": 15},                   # example; confirm page
  "google/gemini-2.5-pro":       {"in": X, "out": Y},                    # fill from Vertex AI pricing
  "google/gemini-2.5-flash":     {"in": X2, "out": Y2}
}
```

Then:

```python
def dollars_per_mtok_to_per_token(v): return v / 1_000_000.0

def compute_cost(tokens, model_key):
    p = PRICING[model_key]
    cost = tokens["input"]  * dollars_per_mtok_to_per_token(p["in"])
    cost += tokens["output"] * dollars_per_mtok_to_per_token(p["out"])
    # Optional: add "reasoning" handling if provider charges differently (OpenAI o-series still bills via in/out; "reasoning" is a reported category)
    return round(cost, 6)
```

Why this works: the JSON exposes token counts; official price pages give per-MTok rates; and OpenAI’s o-series exposes “reasoning tokens” in usage metadata even though billing is still per input/output tokens. ([GitHub][3]) ([OpenAI Platform][6]) ([OpenAI Platform][7]) ([Anthropic][4]) ([Google Cloud][9])

## D) Show provider/model nicely in your UI

OpenCode consistently surfaces `providerID`/`modelID`—e.g., `"providerID=anthropic modelID=claude-sonnet-4-20250514"`. Normalize to `"{provider}/{model}"` and map friendly names (e.g., `Anthropic Claude Sonnet 4`). ([GitHub][11])

## E) Platform auto-detection

Detect OpenCode by presence of any of:

- `~/.local/share/opencode/`
- `~/.config/opencode/opencode.json`
- `~/.local/share/opencode/auth.json`
  Then route to the OpenCode reader; else fall back to Claude Code reader. ([opencode.ai][1]) ([opencode.ai][14]) ([opencode.ai][15])

---

# So…is the ROADMAP doable?

**Yes.** The pieces the roadmap relies on (storage paths, message JSON with token counts, provider/model identifiers) all exist today in OpenCode, and prices are documented by each provider. The only tweak: use a _robust_ storage path detection because OpenCode writes under `project/<slug>/storage/` (or `global/storage/`) per docs, and some environments use a top-level `storage/` dir. Also, be prepared to compute cost yourself since `cost` is often `0` in the stored messages. ([opencode.ai][1]) ([GitHub][2]) ([GitHub][3])

---

# “Do this → show that → update this/that” (end-to-end proving steps)

- **Do this:** Parse a single message JSON from `~/.local/share/opencode/**/storage/message/*.json`.
  **Show that:** Print a row like: `2025-10-03 12:34 — anthropic/claude-sonnet-4-20250514 — in:26,535 out:1,322 reason:0 — $computed_cost`. (Real fields observed in OpenCode output.) ([GitHub][3]) ([GitHub][11])

- **Do this:** Sum a session’s messages by `sessionID`.
  **Show that:** Display total tokens and cost per session; verify that sessions exist under `project/<slug>/storage` for repos and under `global/storage` otherwise. ([opencode.ai][1])

- **Do this:** Switch between providers in a mixed OpenCode session (e.g., Copilot and Anthropic).
  **Show that:** Your UI lists per-message provider/model and per-provider totals. Use the pricing table to compute totals even when `cost = 0` in the file. ([GitHub][3]) ([GitHub][12])

- **Do this:** Add `--platform opencode` and auto-detection.
  **Show that:** Running from a directory with OpenCode data auto-picks the OpenCode parser; otherwise falls back to Claude Code. Presence of `~/.config/opencode/opencode.json`/`auth.json` are strong indicators. ([opencode.ai][14]) ([opencode.ai][15])

---

## Bottom line

- **Feasibility:** ✅ Yes—parse the JSON OpenCode writes, normalize provider/model, compute costs from official pricing, and handle two storage layouts.
- **Corrections:** storage path nuance; cost recomputation.
- **Evidence:** Official OpenCode docs (storage/logs/paths), live JSON examples in issues, and official pricing docs from OpenAI/Anthropic/Google. ([opencode.ai][1]) ([GitHub][3]) ([OpenAI][8]) ([Anthropic][4]) ([Google Cloud][9])

[1]: https://opencode.ai/docs/troubleshooting/ "Troubleshooting | opencode"
[2]: https://github.com/sst/opencode/issues/2506?utm_source=chatgpt.com "Plan mode using Azure does not work #2506 - sst/opencode"
[3]: https://github.com/sst/opencode/issues/2891 "Cost in storage/session is always 0 · Issue #2891 · sst/opencode · GitHub"
[4]: https://www.anthropic.com/claude/sonnet?utm_source=chatgpt.com "Claude Sonnet 4.5"
[5]: https://www.anthropic.com/news/1m-context?utm_source=chatgpt.com "Claude Sonnet 4 now supports 1M tokens of context"
[6]: https://platform.openai.com/docs/models/compare?model=o1-pro&utm_source=chatgpt.com "Compare models - OpenAI API"
[7]: https://platform.openai.com/docs/guides/reasoning-best-practices?utm_source=chatgpt.com "Reasoning best practices - OpenAI API"
[8]: https://openai.com/api/pricing/?utm_source=chatgpt.com "API Pricing"
[9]: https://cloud.google.com/vertex-ai/generative-ai/pricing?utm_source=chatgpt.com "Vertex AI Pricing | Generative AI on ..."
[10]: https://cloud.google.com/vertex-ai/docs/release-notes?utm_source=chatgpt.com "Vertex AI release notes"
[11]: https://github.com/sst/opencode/issues/325?utm_source=chatgpt.com "Cannot find module @ai-sdk/anthropic · Issue #325"
[12]: https://github.com/sst/opencode/issues/2940?utm_source=chatgpt.com "[BUG] OpenCode just hangs randomly after receiving ..."
[13]: https://github.com/sst/opencode/issues/2873?utm_source=chatgpt.com "Error with model DeepSeek V3 Base (free) · Issue #2873"
[14]: https://opencode.ai/docs/config/?utm_source=chatgpt.com "Config"
[15]: https://opencode.ai/docs/providers/?utm_source=chatgpt.com "Providers"

[16]: https://github.com/sst/opencode/issues/409?utm_source=chatgpt.com "sst/opencode - Github copilot \"Failed to send message: 400\""
