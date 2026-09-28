# SamajYoga

A browser-based yoga teacher. It watches you through your device's camera, checks your alignment in each pose, colours your skeleton red/green, times your holds, and can run a saved routine of poses. Everything runs on your device; no video leaves it.

**Open the app:** https://manishchhaya-rgb.github.io/SamajYoga/

## Files in this repo

| File | What it is |
|---|---|
| `index.html` | The whole app (HTML, CSS and JavaScript in one file). |
| `SPEC.md` | The full specification. Anyone given only this file can rebuild the app. Its version number and history match the code. |
| `README.md` | This page: status, how we work, and what's next. |

## Current version: 1.5

- 6 poses: Mountain, Tree, Warrior I, Warrior II, Warrior III, Half Forward Fold (Warrior I/III and the Fold are judged side-on).
- Colour-coded skeleton, alignment score, per-pose hold timer (default 10 s), voice coaching.
- **My routine:** a saved, editable list of poses and hold times that runs as a guided session.

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

- Test the side-on poses, hold timer and routine with a real camera; tune thresholds.
- Ideas: several named routines, rest time between poses, syncing routines between devices.
