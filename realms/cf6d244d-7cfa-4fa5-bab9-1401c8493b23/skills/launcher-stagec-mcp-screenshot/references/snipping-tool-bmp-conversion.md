# Snipping Tool BMP Conversion for Launcher Chat Proof

Use this when Tony says a Launcher-chat proof artifact is a Snipping Tool screenshot saved as `.bmp`.

## Durable rule

Convert Snipping Tool BMP screenshots to **PNG** before attaching, archiving, or using them as visual QA/model proof.

Quick answer when Tony asks “PNG or JPEG?” for Snipping Tool / Launcher / chat screenshots: **PNG by default**. JPEG is the wrong default for UI proof because it can blur text, introduce ringing artifacts around edges, and weaken pixel/layout evidence.

- PNG is lossless, preserves UI text/edges exactly, compresses screenshots well, and is friendly to Telegram/native media + vision tooling.
- BMP is also lossless but often huge and less convenient for chat delivery, indexing, and downstream tooling.
- JPEG is acceptable for photo-like imagery only, for bandwidth-constrained delivery where loss is acceptable, or when Tony explicitly asks for smaller/lossy files. Avoid JPEG for UI, terminal, Mission Control, code, logs, or any screenshot where text and pixel-exact layout matter.

## Workflow

1. Keep the original BMP as raw source evidence if provenance matters.
2. Convert to PNG for delivery/proof.
3. If visual QA is needed, inspect/analyze the PNG, not a JPEG transcode.
4. Send the PNG with `MEDIA:<absolute-path>` and minimal text when Tony asks to see it.

## Conversion examples

Preferred if ImageMagick is available:

```bash
magick input.bmp output.png
```

Batch pattern:

```bash
for f in *.bmp; do magick "$f" "${f%.bmp}.png"; done
```

Python/Pillow fallback when available:

```python
from PIL import Image
from pathlib import Path
for src in Path('.').glob('*.bmp'):
    Image.open(src).save(src.with_suffix('.png'))
```

## Verification

- Output file extension is `.png`.
- File opens successfully and has non-trivial byte count.
- UI text remains crisp; no JPEG block/ringing artifacts.
- Final user-facing message attaches the PNG, not the BMP/JPEG, unless Tony requested otherwise.
