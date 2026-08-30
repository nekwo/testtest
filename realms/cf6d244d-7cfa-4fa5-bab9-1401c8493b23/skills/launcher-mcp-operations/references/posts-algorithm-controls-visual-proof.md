# Posts algorithm controls visual proof and fallback hydration

Session lesson from Stage C Posts proof for task `task_d9ccc54c`.

## What happened

The Launcher Posts tab had semantic navigation and a visible window, but the requested proof was not valid until the exact right-rail `Algorithm Controls` panel was visible with sliders. Several intermediate screenshots were rejected correctly:

1. Stale running binary still showed the old right rail (`Trending`) even though source mounted `AlgorithmControlsPanel`.
2. Fresh rebuild showed `Loading algorithm controls…`, proving the panel mounted but the provider was not hydrated.
3. Backend returned `500`/HTML; the provider surfaced raw error text instead of degrading to fallback.
4. A first layout fix made the whole right rail disappear from the visible frame.
5. A second layout fix showed controls but had a Flutter yellow/black bottom overflow banner.
6. Final proof was accepted only after the panel showed budget/weight sliders and no overflow banner.

## Durable proof rules

- Do not accept a Posts tab screenshot as algorithm-controls proof unless the screenshot visibly shows `Algorithm Controls` and the relevant slider labels/values.
- If source code says the panel is mounted but pixels still show old rail content, suspect a stale Windows binary. Kill Launcher, rebuild `flutter build windows --debug --target lib/main_marionette.dart`, relaunch, then recapture.
- If the panel is stuck on `Loading algorithm controls…`, check whether provider hydration is blocked. A backend 500/HTML response should not render raw server HTML in the panel; read-only config providers can safely degrade to fallback config while admin PATCH remains strict.
- Treat Flutter overflow banners as proof failures, not cosmetic noise. Fix layout and recapture.
- A right rail that disappears after making it scrollable is also proof failure. Prefer bounding the controls panel with `Expanded` and putting `SingleChildScrollView` inside the panel content rather than a loose whole-rail scroll that can collapse/vanish.

## Good final evidence shape

A good screenshot should show, in the right rail:

- `Algorithm Controls`
- `For You source budgets`
- `Follow budget`, `Hashtag budget`, `Community budget`, `Recommendation budget`
- `Budget total: 1.000 / 1.000`
- `For You weights`
- No yellow/black Flutter overflow banner

Use vision/model inspection before attaching the proof if any of these are uncertain.
