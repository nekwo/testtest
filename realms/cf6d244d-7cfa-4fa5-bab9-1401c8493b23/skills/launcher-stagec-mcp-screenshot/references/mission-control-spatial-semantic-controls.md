# Mission Control spatial/terminal semantic control gap

## Trigger

Use this note when capturing or automating Mission Control, especially the spatial office/game surface, right-side Run Inspector, Live Agent Terminal, stage navigation, or proof image controls.

## Lesson from live Stage C proof

A human-visible Mission Control control can be visually clickable without being registered in the Stage C MCP semantic button registry. In the observed session, `get_buttons()` exposed shell navigation and shell scroll controls, but not the `Dev Agent` selector in the Mission Office/spatial surface. A coordinate click could select it, but that is only a debug fallback.

Tony corrected the screenshot workflow: when he asks for the terminal or another lower panel, **scroll down to the exact target** and capture that target. Do not substitute an overview, the Run Inspector header, or a resize-only workaround as if it were exact proof.

## Durable product/QA requirement

If a human can click or scroll a Mission Control surface, MCP should have a stable redaction-safe semantic action/state hook for it.

Recommended semantic controls:

- Agent selection:
  - `mission.agent.select.dev`
  - `mission.agent.select.qa`
  - `mission.agent.select.neko_supervisor`
- Right inspector / terminal scrolling:
  - `mission.inspector.scroll.up|down|top|bottom`
  - `mission.terminal.scroll.up|down|top|bottom`
- Stage-to-terminal jump controls:
  - `mission.terminal.jump.stage.<stage_id>`
- Proof inspection:
  - `mission.proof.inspect.<proof_id>`

Recommended state readbacks:

- selected agent/persona id
- selected run id
- terminal filter/jump target
- terminal scroll offset and max offset
- proof inspector open/closed + selected proof id

## Screenshot workflow rule

1. Navigate to Mission Control semantically.
2. Select the relevant agent/run semantically when available.
3. Scroll the exact relevant panel semantically (`mission.terminal.*` or `mission.inspector.*`), not only the outer shell.
4. Verify state-first: selected agent/run/terminal target and scroll offset.
5. Capture the PNG.
6. Vision-check or otherwise verify the PNG visibly shows the requested target before sending.

## Fallback classification

Coordinate clicks, window resizing, and overview screenshots are acceptable only as temporary exploration/debug fallbacks. They are not AAA proof for the durable QA lane. If used, report the missing semantic control gap explicitly and create/append a docs/TODO item.
