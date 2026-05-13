Visual Application State Protocol 0.1
=====================================

Status
------

This document defines VASP version 0.1. The terms MUST, SHOULD, and MAY are normative.

Document Model
--------------

A VASP document MUST be a JSON object with these top-level fields:

- `version`: protocol version string. Version 0.1 documents MUST use `"0.1"`.
- `state_id`: stable identifier for the visual state.
- `screen_type`: coarse classification of the screen.
- `confidence`: confidence level for the screen classification.
- `ui_tree`: ordered list of typed UI elements.
- `affordances`: optional list of precomputed user actions.
- `workflow_context`: optional short summary for logs and operators.
- `metadata`: optional extraction details.

State Identity
--------------

`state_id` MUST be stable for visually equivalent screens. Producers SHOULD use a perceptual image hash when the source image is available.

The canonical string form is:

```text
phash:<16 lowercase hex characters>
```

Screen Types
------------

The allowed `screen_type` values are:

- `error`
- `config`
- `terminal`
- `conversation`
- `ui`
- `unknown`

Producers SHOULD return `unknown` when no classifier reaches a useful confidence threshold.

Confidence
----------

The allowed `confidence` values are:

- `high`
- `medium`
- `low`
- `none`

UI Elements
-----------

Each `ui_tree` entry MUST include:

- `id`: stable element id within the document.
- `type`: typed element category.
- `text`: recognized text, or an empty string when unavailable.
- `cx`: center x coordinate in source pixels.
- `cy`: center y coordinate in source pixels.
- `w`: width in source pixels.
- `h`: height in source pixels.

The allowed element `type` values are:

- `text`
- `button`
- `input`
- `link`
- `checkbox`
- `radio`
- `select`
- `image`
- `icon`
- `unknown`

Coordinates
-----------

Coordinates MUST use source image pixels. Producers MUST NOT normalize coordinates unless a separate field explicitly states the coordinate space.

The point `(cx, cy)` is the center of the element bounding box. Width and height MUST be positive numbers.

Affordances
-----------

An affordance describes an action a workflow can attempt without a second grounding step. Each affordance SHOULD include:

- `action`: action name.
- `target_id`: id of the related UI element.
- `label`: operator-readable label.
- `cx`: click or focus x coordinate.
- `cy`: click or focus y coordinate.

Allowed action names are:

- `click`
- `type`
- `select`
- `toggle`
- `wait`

Diff Documents
--------------

A VASP diff document compares two VASP states and SHOULD include:

- `before_state_id`
- `after_state_id`
- `context_changed`
- `entries`
- `tokens_saved`

Each diff entry SHOULD identify whether an element `appeared`, `changed`, or `removed`.

Producer Requirements
---------------------

Producers MUST preserve source pixel coordinates.

Producers SHOULD sort `ui_tree` entries top-to-bottom, then left-to-right.

Producers SHOULD keep `workflow_context` short and factual.

Producers MUST NOT include secrets from local environment variables, shell history, or credential stores.

Consumer Requirements
---------------------

Consumers SHOULD prefer coordinates from affordances over inferred coordinates from text.

Consumers SHOULD treat `confidence: low` and `confidence: none` as advisory output.

Consumers MUST validate schema compatibility before executing high-impact actions.
