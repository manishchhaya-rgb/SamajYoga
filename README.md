# SamajYoga

A browser-based yoga teacher. It watches you through your device's camera, shows each pose as an outline to step into, checks your alignment, colours your skeleton red/green, times your holds, and coaches you through a saved sequence of poses. Everything runs on your device; no video leaves it.

**Open the app:** https://manishchhaya-rgb.github.io/SamajYoga/

## Files in this repo

| File | What it is |
|---|---|
| `index.html` | The whole app (HTML, CSS and JavaScript in one file). |
| `SPEC.md` | The full specification. Anyone given only this file can rebuild the app. Its version number and history match the code. |
| `README.md` | This page: status, how we work, and what's next. |

## Current version: 1.6

- 6 poses: Mountain, Tree, Warrior II (facing the camera); Warrior I, Warrior III, Half Forward Fold (side-on).
- **Pose outline:** each pose is drawn on the video as a see-through outline you step into; it turns green when you're inside it, and the app tells you to step back, come closer or move left/right.
- **Facing prompts:** on screen and out loud — "Stand facing the camera" / "Stand side-on" — plus a warning if you're turned the wrong way.
- **My sequence:** a default sequence of all six poses (front poses first, then side-on); edit it from the ☰ side menu.
- Colour-coded skeleton, alignment score, hold timer, voice coaching (on by default).

See `SPEC.md` §15 for the full version history.

## How we work

Every change is one commit that contains **both** the updated `index.html` and the updated `SPEC.md` (version number bumped, version-history row added), plus this README when the status or next steps change. The commit message names the spec version.

## Running it locally

Open it over http, not by double-clicking the file (Chrome blocks the camera on `file://`):

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Next steps

- Test v1.6 with a real camera: outline size/position, the facing check, and the side-on poses; tune thresholds.
- Ideas: several named sequences, rest time between poses, syncing sequences between devices.
