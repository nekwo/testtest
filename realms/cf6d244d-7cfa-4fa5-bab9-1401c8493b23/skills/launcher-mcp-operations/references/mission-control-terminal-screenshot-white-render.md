# White render surface: semantic state healthy, pixels blank

> **Refresh note (2026-08-28):** the original incident targeted the Run Inspector / Live
> Agent Terminal, which no longer exists. The white-render classification is the durable
> content and applies to any Launcher surface — office scene, agent console, chat
> transcript, board, roster.

Session lesson: the app could be launched/navigated successfully via Stage C MCP with env
pins (`hermes_profile`, `harness_runtime_root`, `hermes_home`), and MCP state reported the
requested `selected_tab`, shell mounted, no loading overlay, and no error banner. However,
both `screenshot_window`/PrintWindow and foreground `ImageGrab` captures produced a
uniform white Launcher surface. A normal debug `main.dart` launch produced the same white
surface.

Do not send a blank/white capture as proof. Treat this as a visual render/capture blocker
even when semantic MCP state is healthy.

Fallback attempted: a temporary Flutter golden/widget render produced a non-blank layout
image, but text rendered as block glyphs due to `google_fonts` test/golden capture
behavior. That artifact is useful as layout-only proof, not readable content proof.

## Recommended workflow

1. Rebuild a fresh Launcher marionette target before proof.
2. Launch/open the requested surface with explicit Harness env pins.
3. Capture via `screenshot_window`; if blank, try a foreground capture once.
4. Verify the PNG visually / by key frame before delivery.
5. If pixels are still white while semantic state is mounted, report
   `live Windows pixel proof blocked: white render surface` and do not overclaim.
6. Before reaching for a widget/golden fallback, try the Marionette internal Flutter
   screenshot path — see `stagec-marionette-internal-screenshot-fallback.md`. It captures
   Flutter's rendered scene and is stronger evidence than a golden.
7. If using a widget/golden fallback anyway, disable or properly mock GoogleFonts with real
   test assets; otherwise block-glyph text is not readable proof.
8. Remove temporary capture tests/artifacts from the product repo before final handoff.

## Acceptance language

- Valid live proof: a readable PNG showing the requested surface with its actual content
  (roster rows, transcript lines, board cards, console text).
- Partial proof: a widget/golden layout image with readable text.
- Invalid proof: a uniform white/blank window, or a block-glyph golden where the text
  cannot be read.
