# Current proof artifacts and env pinning

> **Rewritten 2026-10-02.** The video recipe this note carried — maximize the window, then
> FFmpeg `gdigrab` it by title — is retired under owner ruling
> OR-2026-10-02-qa-never-covers-the-screen: a window-title grab records whatever is on top
> of that screen region, so it only works with the QA Launcher over the owner's work. There
> is no in-app video lane yet. The stale-artifact rule and env pinning are the durable
> content.

## Trigger

Use this when the operator asks for Launcher visual proof, or when an artifact does not
show the same state as the current Launcher.

## Stale artifacts are not proof

A previously captured PNG or MP4 can be valid media but invalid proof if it predates the
current fix/state. If the operator says one artifact shows the right UI but another does
not, assume the older one is stale until proven otherwise. Do not defend or resend it.

Correct recovery:

1. Bring the Launcher to the claimed state through semantic controls, and read the state
   back.
2. Capture a fresh in-app frame (`screenshot_window`); the window is never raised.
3. Verify the new PNG shows the claimed state before sending it.

## Motion and timing defects

When the defect depends on motion, timing, transitions or flicker, there is no compliant
recording lane today. Do not record the desktop and do not foreground the window to make a
recording work. Capture an in-app frame sequence (several `screenshot_window` calls, each
paired with its state readback) and say plainly that video proof was not taken because the
QA lane has no in-app recorder.

## Env-pinned Launcher QA Smoke proof

Stage C proof is only trustworthy when the Launcher process is pinned to the same Harness
runtime the CLI is checking.

Required safe envelope signals for pinned proof:

- `app.hermes_profile: alice` or the intended profile
- `app.harness_runtime_root_configured: true`
- `app.hermes_home_configured: true`
- `navigation_state.selected_tab: <requested tab>`

The MCP path supports `hermes_profile`, `harness_runtime_root`, and `hermes_home`, and the
Stage C MCP launch manager forwards them to its internal launch helper.

Pinned requests must refuse unpinned live disk sessions with a bounded
`app_env_pin_mismatch` instead of silently attaching to a stale Launcher that may show an
empty or foreign runtime's state.

## Acceptance pattern

Valid visual proof has all of:

- an in-app frame captured after the fix/state change, not reused from an earlier smoke;
- the visible Launcher surface matching the CLI/Harness state it claims (roster rows, chat
  transcript, board cards, office scene, agent console);
- env-pinned launch metadata in the envelope when parity is the claim.
