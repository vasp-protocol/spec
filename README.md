VASP
====

Visual Application State Protocol defines a compact, deterministic document for visual UI state.

The protocol is designed for local screenshot extraction, repeatable UI automation, and state comparison. A VASP document gives consumers the screen type, stable state identity, recognized UI elements, available actions, and optional workflow context without requiring another visual interpretation pass.

Repository layout
-----------------

`SPEC.md` contains the normative protocol definition.

`schema/vasp.schema.json` contains the JSON Schema for version 0.1 documents.

`examples/payment-error.json` shows a complete extraction result.

Design goals
------------

- Stable coordinates over natural language captions.
- Local-first processing with no required network dependency.
- Typed UI elements suitable for click, type, wait, and verify loops.
- Deterministic diffs between before and after states.
- Small enough output for high-frequency automation workflows.
