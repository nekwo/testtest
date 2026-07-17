# Mission Control terminal screenshot: semantic mounted but white pixels

Session lesson: Tony asked for a screenshot of the new Mission Control right-side live terminal. The app could be launched/navigated successfully via Stage C MCP with env pins (`hermes_profile`, `harness_runtime_root`, `hermes_home`) and MCP state reported `selected_tab=missionControl`, shell mounted, no loading overlay, and no error banner. However, both `screenshot_window`/PrintWindow and foreground `ImageGrab` captures produced a uniform white Launcher surface. A normal debug `main.dart` launch produced the same white surface.

Do not send a blank/white capture as terminal proof. Treat this as a visual render/capture blocker even when semantic MCP state is healthy.

Fallback attempted: a temporary Flutter golden/widget render produced a non-blank layout image, but text rendered as block glyphs due to `google_fonts` test/golden capture behavior. That artifact is useful as layout-only proof, not readable terminal proof.

Recommended future workflow:
1. Rebuild fresh Launcher target before proof.
2. Launch/open Mission Control with explicit Harness env pins.
3. Capture via `screenshot_window`; if blank, try foreground capture once.
4. Verify the PNG visually/key-frame before delivery.
5. If pixels are still white while semantic state is mounted, report `live Windows pixel proof blocked: white render surface` and do not overclaim.
6. If using widget/golden fallback, disable or properly mock GoogleFonts with real test assets; otherwise block-glyph text is not readable proof.
7. Remove temporary capture tests/artifacts from the product repo before final handoff.

Acceptance language:
- Valid live proof: readable PNG showing Run Inspector / Live Agent Terminal with terminal transcript lines.
- Partial proof: widget/golden layout image with readable text.
- Invalid proof: uniform white/blank window or block-glyph golden where terminal text cannot be read.
