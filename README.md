# VASP: Visual Agent State Protocol

Version: 1.0-draft | Status: Open RFC | Published: May 2026
Reference implementation: [farscry](https://github.com/teles-forge/farscry)

## The Problem

Every agent framework handles visual context differently.
Devin sends raw images. Claude Code re-processes every step.
No standard exists for how agents represent or share visual UI state.

Vision APIs describe. VASP gives coordinates.

## What VASP Is

A typed, deterministic, offline interchange format
for screenshot-derived visual context.
Works without app cooperation, without VLM, without GPU.

## Core Fields

```
vasp_version: "1.0"
state_id: phash:<16-char-hex>
screen_type: error | config | terminal | conversation | ui | unknown
confidence: high | medium | low | none
lang: string
agent_context: string  (one-line summary for the agent)
delta_from: phash:<hex> | null
context_similarity: 0.0-1.0 | null
context_changed: bool | null
```

## state_id Algorithm

1. Resize image to 32x32 (nearest-neighbor)
2. Convert to grayscale: 0.299R + 0.587G + 0.114B
3. Compute 32x32 2D DCT-II
4. Extract top-left 8x8 coefficients (64 values)
5. Exclude DCT[0][0] (DC component)
6. Compute mean of remaining 63 values
7. bit=1 if value > mean, else 0
8. Pack 64 bits -> [u8; 8]
9. Encode: `phash:<16-char-lowercase-hex>`

## screen_type Values

| Value | Description |
|---|---|
| `error` | Error messages, stack traces, failed states |
| `config` | Forms, settings panels, configuration screens |
| `terminal` | Shell output, command prompts, build logs |
| `conversation` | Chat interfaces, message threads |
| `ui` | General UI (buttons, nav, dashboards) |
| `unknown` | Unclassified |

## context_similarity Gate

Compute token overlap between before/after OCR outputs.
If overlap < 0.20: emit `context_changed: true`, skip diff.
Prevents diffing unrelated UIs.

## Output Format (compact text)

```
=== farscry visual context ===
source: screenshot.png
screen_type: config
state_id: phash:8f4a2c9d1e3b7f6a
confidence: high
lang: eng
agent_context: "Payment settings - Save available"
---
[top-center]    heading  "Payment Settings"
[middle-left]   label    "Max Value:"
[middle-center] input    value="1500"        editable:true
[middle-right]  button   "Save Changes"      enabled:true
[bottom]        error    "Value must be <= 10000"

affordances:
  click -> "Save Changes" at (400,300)  enabled:true
  type  -> "Max Value"    at (200,120)  current:"1500"
```

## Diff Output Format

```
=== farscry diff ===
delta_from: phash:8f4a2c9d1e3b7f6a
state_id:   phash:3d9b1e4f2a8c7b5e
context_similarity: 0.923
context_changed: false
---
appeared:  error   "Card declined"              at (20,350)
changed:   button  "Submit" -> "Processing..."  disabled:true
unchanged: [3 elements]
```

## Why Not Accessibility Trees?

33% of macOS apps have broken or missing AX trees.
VASP works on any screenshot, any app, any platform.

## Why Not Vision APIs?

Vision APIs return prose. Agents guess coordinates.
VASP returns typed coordinates. Agents act with precision.
~9x fewer tokens on 1080p. ~16x fewer on 4K. $0. Offline.

## Reference Implementation

[farscry](https://github.com/teles-forge/farscry)

## Status

Open RFC. Comments welcome.
Open an issue: https://github.com/vasp-protocol/spec/issues

## License

Apache 2.0
