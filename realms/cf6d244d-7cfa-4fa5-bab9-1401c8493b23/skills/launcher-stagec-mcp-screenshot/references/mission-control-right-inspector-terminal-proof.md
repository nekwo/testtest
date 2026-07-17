# Mission Control right-inspector terminal/proof visual proof

When Tony asks to "show me how it looks" after Mission Control Run Inspector / Live Agent Terminal changes, the screenshot must visibly show the exact changed surface, not only the top-level Mission Control overview.

## Lesson

The generic `shell.scroll` semantic controls scroll the active page/content column, but they may not expose lower content inside the right-side Run Inspector panel. In the observed case, `shell.scroll.bottom` reached `y=max_y`, yet the screenshot still showed only the top of the right Run Inspector: readable run metadata and `Skills Used: Agent Runtime Harness (agent-runtime-harness)`, while `Live Agent Terminal` and proof-image inspect controls remained below the panel fold.

## Correct workflow

1. Rebuild the Marionette target before visual proof after Mission Control UI code changes.
2. Open `missionControl` through Stage C MCP with explicit `hermes_profile`, `harness_runtime_root`, and `hermes_home` pins.
3. If `PrintWindow` returns blank/uniform pixels after semantic gates pass, use the Marionette internal Flutter screenshot fallback and label it as such.
4. Use vision/sanity-check before sending. If the screenshot does not visibly show the requested exact surface, do not overclaim it.
5. Try semantic `shell.scroll` controls for below-fold page content and verify offset changes.
6. If the right Run Inspector / Live Agent Terminal is still not visible after `shell.scroll.bottom`, report this as a page-local/right-inspector semantic-scroll gap. Do not coordinate-scroll or claim exact terminal proof.
7. For a future fix, add semantic controls or an explicit QA action for the Run Inspector panel itself, e.g. `mission_control.inspector.scroll.*`, `mission_control.agent.select.<id>`, and proof-image inspect/open controls, so screenshots can target terminal/proof areas deterministically.

## Reporting language

Good:
- "This PNG proves the readable-label Run Inspector header. Exact terminal/proof-inspector pixels are still blocked by missing right-inspector scroll/selection semantics."

Bad:
- "Here is the terminal/proof view" when the screenshot only shows the top metadata region.

## Artifact expectations

A valid terminal/proof screenshot should visibly include at least one of:
- `Live Agent Terminal`
- `Terminal index`
- transcript rows such as `Redaction-safe Live Transcript`
- `Inspect proof image <proof_id>`
- `Proof Image Inspector`

If none are visible, classify it as partial visual proof only.