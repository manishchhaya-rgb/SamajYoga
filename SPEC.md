# SamajYoga — Specification

| | |
|---|---|
| **Spec version** | 1.4 |
| **Date** | 2026-09-28 |
| **Matches code** | `index.html` in `manishchhaya-rgb/SamajYoga`, commit "Add Warrior I, Warrior III, Half Forward Fold and per-pose hold timer" |
| **Live URL** | https://manishchhaya-rgb.github.io/SamajYoga/ |

**Purpose of this document:** anyone (or Claude) given only this file should be able to rebuild the app so it behaves the same. Every threshold, colour, message and timing that affects behaviour is listed here. It is updated with every change to the app, and the version number goes up each time.

---

## 1. Product summary

SamajYoga is a "yoga teacher" that runs in a web browser. It uses the device's front camera to track the user's body, and for the chosen pose it:

- draws a skeleton over the mirrored video, with each checked body line **red** until aligned and **green** when aligned;
- shows a checklist of alignment cues and a 0–100 **alignment score**;
- runs a per-pose **hold timer** (default 10 s) that counts only while fully aligned;
- optionally **speaks** corrections and encouragement.

Everything runs on the device. No video or data leaves the machine; only the pose model and library are downloaded once.

## 2. Platform and constraints

- **One file:** `index.html` containing all HTML, CSS and JavaScript (`<script type="module">`). No build step, no server code.
- **Hosting:** GitHub Pages (repo root, `main` branch). Must be served over http(s). Chrome blocks the camera on `file://` pages.
- **Browsers:** current Chrome (desktop and Android) and Safari (iPhone/iPad). Must work at phone width.
- **iOS home-screen app feel:** meta tags `apple-mobile-web-app-capable=yes`, status bar `black-translucent`, title "Yoga Teacher", `mobile-web-app-capable=yes`, `theme-color=#0e1116`. Viewport: `width=device-width, initial-scale=1.0, maximum-scale=1.0, viewport-fit=cover`.
- **Page title:** "Yoga Teacher — Pose Coach".

## 3. External dependencies

| What | Source |
|---|---|
| MediaPipe Tasks Vision library 0.10.12 (`FilesetResolver`, `PoseLandmarker`) | ES module import from `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.12` |
| WASM files | Try `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.12/wasm`, then fall back to `https://unpkg.com/@mediapipe/tasks-vision@0.10.12/wasm` |
| Pose model | `https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/1/pose_landmarker_lite.task` |
| Speech | Browser built-in Web Speech API (`speechSynthesis`) — no download |

**Model options:** `runningMode: "VIDEO"`, `numPoses: 1`, `minPoseDetectionConfidence`, `minPosePresenceConfidence`, `minTrackingConfidence` all `0.5`. Try `delegate: "GPU"` first; if that fails, try `"CPU"`.

## 4. Screen layout

Dark theme. Two columns on wide screens (left `minmax(0,1.4fr)`, right `minmax(280px,0.9fr)`, gap 18px, padding 18px 20px); a single column below 860px width.

**Header:** "🧘 Yoga Teacher" (18px, weight 650) plus the muted tagline "Live posture coaching from your webcam — everything runs locally in your browser." Bottom border.

**Left column, top to bottom:**
1. **Stage:** black box, 4:3 aspect ratio, 14px rounded corners, 1px border. Contains the `<video>` (`playsinline muted`, `object-fit: cover`) and a `<canvas>` overlay filling it. **Both are mirrored with CSS `transform: scaleX(-1)`.** Before the camera starts, a centred placeholder shows 📷 and "Click **Start camera** and allow camera access. Stand back far enough that your whole body is visible."
2. **Colour legend:** three short 18×4px bars with labels: red "Needs adjusting", green "Aligned", grey "Not checked in this pose".
3. **Controls:** "Start camera" (primary, blue), "Stop" (disabled until running), "🔊 Voice coaching: off/on" (green outline when on).
4. **Pose picker:** one button per pose, in this order: Mountain Pose, Tree Pose, Warrior I, Warrior II, Warrior III, Half Forward Fold. The active pose has a blue border and background `#14263f`.
5. **Status line:** 13px, muted; turns orange for errors.
6. **Help text** (12.5px, muted) covering: framing tip and privacy; skeleton colours; hold timer; "Half Forward Fold: stand side-on to the camera"; voice coaching; and what to do if the model won't download (run `python3 -m http.server 8000` and open `http://localhost:8000/…`).

**Right column (panel):**
1. Pose title as "Name  ·  Sanskrit", and the pose description underneath.
2. **Score ring:** 76px circle drawn as a conic-gradient ring with the number inside. Label beside it is "Waiting…", "Keep adjusting" (<50), "Getting there…" (50–79) or "Great alignment!" (≥80), with "Alignment score" underneath. Ring colour is green ≥80, blue 50–79, orange <50. It shows "—" when idle.
3. **Hold timer box** (see §9): "⏱ Hold timer", − [10s] + buttons, progress bar, status text and a "Restart" button.
4. **Cue checklist:** one row per cue, with a round badge and the cue label, plus a one-line message under it. The badge is ✓ green, ! orange, or • grey (idle, message "Get into position…").

**Colours (CSS variables):** bg `#0e1116`, panel `#161b22`, panel-2 `#1c232d`, text `#e6edf3`, muted `#9aa7b4`, good `#3fb950`, bad (UI orange) `#f0883e`, accent `#58a6ff`, line `#2b3440`, seg-bad (skeleton red) `#f85149`. System font stack. Buttons have 10px radius, padding 10px 14px, and a blue border on hover.

## 5. Body landmarks used

MediaPipe Pose landmark indices: nose 0, left ear 7, right ear 8, shoulders 11/12, elbows 13/14, wrists 15/16, hips 23/24, knees 25/26, ankles 27/28 (left/right). Coordinates are normalised: x is 0–1 across the width, y is 0–1 down the height. Each landmark may have a `visibility` value between 0 and 1.

**Skeleton lines (12):** 11–12 shoulders, 11–13, 13–15 left arm, 12–14, 14–16 right arm, 11–23, 12–24 torso sides, 23–24 hips, 23–25, 25–27 left leg, 24–26, 26–28 right leg.

**Line groups** used by cues:
- ARM left = 11–13, 13–15; ARM right = 12–14, 14–16
- LEG left = 23–25, 25–27; LEG right = 24–26, 26–28
- SHOULDERS = 11–12; HIPS = 23–24
- TORSO = virtual "spine" line (mid-hips → mid-shoulders) + 11–23 + 12–24

## 6. Geometry (must be exact)

**Aspect correction:** x and y are normalised to different lengths, so before any angle maths, y differences are divided by the frame aspect `AR = videoWidth / videoHeight` (default 4/3 if unknown).

- `angle(a, b, c)`: the angle at point b in degrees (0–180) between vectors b→a and b→c, using aspect-corrected y. Returns 0 if either vector has zero length.
- `tiltFromVertical(a, b)`: `atan2(|dx|, |dy|)` in degrees, with dy aspect-corrected. Range 0–90, where **0 = vertical**. It must not depend on the direction of travel (shoulders being above hips must not read as 180°).
- `tiltFromHorizontal(a, b)`: `atan2(|dy|, |dx|)` in degrees. Range 0–90, where **0 = horizontal**.
- `mid(a, b)`: the midpoint of two points.
- `kneeAngles`: left is angle(hip 23, knee 25, ankle 27); right is angle(24, 26, 28).
- Level checks for shoulders and hips use the raw difference in y (normalised), not an angle.
- `within(v, lo, hi)`: true if lo ≤ v ≤ hi.

## 7. Poses and cues

Each cue returns: **ok** (pass/fail), **msg** (the text shown and spoken), and **marks**, which is a list of *{line group, ok}* saying which skeleton lines it controls. Where a cue checks left and right separately, each side gets its own mark, so one side can be green while the other is red. Messages are listed as *pass / fail*.

"Torso tilt" means `tiltFromVertical(mid-hips, mid-shoulders)`.

### 7.1 Mountain Pose · Tadasana
Description: "Stand tall, feet grounded, arms relaxed at your sides, spine long."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Spine tall & vertical | torso tilt < 10° | TORSO | "Nicely stacked." / "Lengthen up — you're leaning to one side." |
| Shoulders level | \|y11 − y12\| < 0.04 | SHOULDERS | "Even and relaxed." / "Drop the raised shoulder to level them." |
| Hips level | \|y23 − y24\| < 0.04 | HIPS | "Weight is even." / "Balance your weight evenly on both feet." |
| Legs straight | each knee > 160° (per side) | LEG l / LEG r | "Strong, straight legs." / "Gently straighten your knees." |

### 7.2 Tree Pose · Vrksasana
Description: "Balance on one leg; place the other foot on your inner thigh or calf. Grow tall."
Standing leg = the side with the larger knee angle (ties go to left); the other is the lifted leg.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Standing on one leg | standing knee > 160° AND lifted knee < 130° | LEG stand (ok = stand > 160), LEG lift (ok = lift < 130) | "Good — one leg lifted." / "Lift one foot and rest it against the standing leg." |
| Standing leg strong | standing knee > 168° | LEG stand | "Rooted and steady." / "Press down and straighten the standing leg." |
| Hips level & balanced | \|y23 − y24\| < 0.06 | HIPS | "Steady hips." / "Level the hips — don't jut one out." |
| Arms lifted overhead | each wrist y < nose y (per side) | ARM l / ARM r | "Reaching up like branches." / "Raise both hands overhead (or to heart center)." |
| Torso upright | torso tilt < 14° | TORSO | "Tall and centered." / "Stack your chest over your hips." |

### 7.3 Warrior I · Virabhadrasana I (best side-on)
Description: "Best side-on to the camera. Lunge with the front knee toward 90°, back leg long, both arms reaching straight up, chest lifted."
Front leg = the side with the smaller knee angle (ties go to left); back leg = the side with the larger angle.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 120° | LEG front | "Strong front knee." / if > 120: "Bend the front knee more toward 90°." else "Ease up — keep the knee over the ankle." |
| Back leg straight | back knee > 150° | LEG back | "Back leg long and strong." / "Straighten the back leg and press through the back heel." |
| Arms reaching straight up | per side: wrist y < nose y AND tiltFromVertical(shoulder, wrist) < 25° | ARM l / ARM r | "Arms reaching high." / "Reach both arms straight up beside your ears." |
| Arms straight | per side: elbow angle (shoulder–elbow–wrist) > 150° | ARM l / ARM r | "Long through the fingertips." / "Straighten the elbows." |
| Torso upright | torso tilt < 15° | TORSO | "Chest lifted over the hips." / "Lift the chest — don't lean forward over the front knee." |

### 7.4 Warrior II · Virabhadrasana II
Description: "Wide stance, front knee bent toward 90°, back leg straight, arms reaching out level with the floor."
Front leg = smaller knee angle (ties go to left); back leg = larger (ties go to right).

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 120° | LEG front | "Deep, strong lunge." / if > 120: "Bend the front knee more toward 90°." else "Ease up — don't let the knee pass the ankle." |
| Back leg straight | back knee > 155° | LEG back | "Strong back leg." / "Straighten and energize the back leg." |
| Arms level with floor | per side: tiltFromHorizontal(shoulder, wrist) < 20° | ARM l / ARM r | "Arms floating, parallel to the floor." / "Lift both arms to shoulder height, reaching out." |
| Arms extended | per side: elbow angle > 155° | ARM l / ARM r | "Long through the fingertips." / "Straighten the elbows and reach wide." |
| Torso upright | torso tilt < 18° | TORSO | "Chest stacked over hips." / "Don't lean over the front leg — lift the torso." |

### 7.5 Warrior III · Virabhadrasana III (side-on)
Description: "Side-on to the camera. Balance on one straight leg, tip forward so torso and lifted leg form one level line, arms reaching forward."
Standing leg = the side whose ankle is lower on screen (larger y; ties go to left); the other is the lifted leg.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Standing leg straight | standing knee > 160° | LEG stand | "Rooted standing leg." / "Straighten the standing leg — a micro-bend is fine." |
| Lifted leg level & straight | tiltFromHorizontal(lifted hip, lifted ankle) < 20° AND lifted knee > 150° | LEG lift | "Back leg long and level." / if not level: "Lift the back leg up to hip height." else "Straighten the lifted leg and reach through the heel." |
| Torso level with floor | tiltFromHorizontal(mid-hips, mid-shoulders) < 20° | TORSO | "Torso level — a perfect T." / "Tip the chest forward until your torso is level with the floor." |
| Arms reaching forward | per side: tiltFromHorizontal(shoulder, wrist) < 25° AND elbow angle > 150° | ARM l / ARM r | "Reaching long through the fingertips." / "Reach both arms straight forward, in line with your torso." |

### 7.6 Half Forward Fold · Ardha Uttanasana (side-on)
Description: "Stand side-on to the camera. Hinge forward from the hips with a long, flat back until the angle between legs and torso is less than 90°. Hands can rest on the floor or shins."
**Side selection:** add up visibility of hip + knee + ankle + shoulder for each side (missing visibility counts as 1); use the side with the larger total (ties go to left). **Head point** = that side's ear if its visibility > 0.4, otherwise the nose. Arms are **not** checked (hands may rest on the floor), so they show grey.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Hinged from the hips (< 90°) | angle(ankle, hip, head) < 90° — this value is also saved for the on-screen readout | TORSO | "Good deep hinge from the hips." / "Hinge further forward from the hips." |
| Back straight | angle(hip, shoulder, head) > 150° | TORSO | "Long, flat back." / "Lengthen your spine — reach the crown of your head forward, don't round." |
| Legs straight | each knee > 155° (per side) | LEG l / LEG r | "Legs long and steady." / "Straighten your legs — a soft bend in the knees is fine." |

## 8. Scoring and line colours (every frame with a detected body)

1. Run every cue of the current pose.
2. **Line state:** for each line named in any mark, the line is aligned only if **every** mark that touches it is ok (logical AND across cues and marks). Lines not named by any mark are "unchecked".
3. **Line smoothing:** each checked line keeps a value between 0 and 1: `new = old × 0.6 + (aligned ? 1 : 0) × 0.4` (the first frame takes 0 or 1 directly). It is drawn **green if ≥ 0.5, red otherwise**. Unchecked lines are grey `rgba(154,167,180,0.55)`. Reset on pose change and on Stop.
4. **Score:** raw = round(passed cues ÷ total cues × 100). Smoothed = `old × 0.7 + raw × 0.3`, kept as an unrounded number (so it can actually reach 100) and **rounded only for display**. Reset on pose change.
5. Update the checklist and ring, then the hold timer (§9), then voice (§10).
6. If no body is detected: status "No body detected — make sure you're fully in frame." and the canvas is cleared.

## 9. Hold timer

- Per-pose target, **default 10 s**, adjusted with − / + in **5 s steps**, limited to **5–120 s**. Saved per pose in `localStorage` under key `samajyoga.holdTargets` as JSON `{poseKey: seconds}`. Every storage read and write is wrapped in try/catch, and the app must work without storage. Changing the target restarts the timer.
- "All ok" means every cue in the current pose passes on that frame.
- The timer starts the first time all cues pass. After that it is **active** while the last all-ok frame was less than **0.6 s** ago (short tracking wobbles don't pause it). Time is added only while active; each frame adds at most 0.25 s.
- **Voice** (only if voice coaching is on): once remaining time drops to 5 s or below while above 4 s, say "Five more seconds." once, but only if the target is ≥ 10 s. At 0 s, mark it done and say "Well done. Release the pose."
- **Panel status text:**
  - not started: "Get every line green to start the timer."
  - active: "Holding… Ns left"
  - paused: "Paused — realign to continue (Ns left)"
  - done: "Hold complete ✓ — nice work!" (green, bold; the bar turns green)
- **Progress bar:** 8px, fill width = elapsed ÷ target (blue, green when done).
- **Reset** (elapsed back to 0, not started, not done) happens on: pose change, Stop, the Restart button, and changing the target. Reset also clears the voice "praise given" flag.

## 10. Voice coaching

- Off by default. The toggle button text is "🔊 Voice coaching: off" / "🔊 Voice coaching: on". Turning it on says "Voice coaching on. I'll guide your alignment." (this click also unlocks audio on phones). Turning it off cancels speech. If the browser has no speech, show the error "This browser doesn't support speech. Try the latest Chrome."
- **Voice choice:** the first voice whose language matches en-US/GB/AU and whose name matches female|samantha|karen|serena|moira|tessa|google us english|zira; otherwise the first English voice; otherwise the first voice. Re-check when the browser's voice list changes.
- **Delivery:** rate **0.82**, pitch **0.78**, volume 1 (calm and deeper). Each new utterance cancels anything still queued.
- **Clean-up before speaking:** "°" becomes " degrees", "~" becomes "about ", brackets are removed, and whitespace is collapsed.
- **Corrections:** only while running. Speak the message of the **first failing cue** (in checklist order). A different cue waits at least **4 s** after the last speech; the same cue repeats no sooner than **8 s**.
- **Praise:** when all cues pass, the praise hasn't been given yet, and ≥ 1.5 s have passed since the last speech, pick one of: "Beautiful. Hold it there." / "That's it — lovely alignment." / "Perfect. Stay with the breath." / "Great posture. Hold and breathe." If the hold isn't finished, strip any trailing " Hold…" or " Stay…" sentence and append " Hold for N seconds." (N = remaining whole seconds, rounded up). Praise resets whenever a cue fails.
- On pose change while running with voice on, speak "{Pose name}. {description}".

## 11. Overlay drawing (canvas matches the video's pixel size; drawn in this order)

Line widths scale with the canvas width W (height H). Landmark x, y are multiplied by W, H.

1. **Centre guide:** dashed vertical line at W/2, dash [6, 8], white at 22% opacity, width max(1.5, W×0.0025).
2. **Skeleton:** base width = max(3, W×0.006), with round line caps. Each of the 12 lines is drawn in its colour (§8); checked lines are 1.3× the base width. Joints (the 13 landmarks in §5) are circles of radius max(4, W×0.008), coloured red if any checked line through them is red, green if they touch only green checked lines, and grey if they touch no checked lines.
3. **Level guides:** dashed [5, 6] horizontal lines at 60% opacity, width max(1.5, W×0.003), from x = 8% to 92% of the width, at the shoulder-midpoint height and the hip-midpoint height. Their colour is the colour of the shoulder line (11–12) and hip line (23–24) respectively, which means grey when the pose doesn't check them.
4. **Spine indicator:**
   - a dashed [4, 5] white line (40% opacity) straight up from mid-hips to shoulder height;
   - a solid line from mid-hips to mid-shoulders, width max(4, W×0.008), green or red. The colour follows the smoothed "spine" state if the pose checks the torso; otherwise it is green when torso tilt is under the pose's tolerance (Mountain 10, Tree 14, Warrior II 18, others 12);
   - yellow `#ffd166` dots at both ends, radius max(6, W×0.012);
   - a text readout in the same colour at the line's midpoint shifted left by W×0.06, bold, size max(14, W×0.022). It shows the torso tilt as "N°"; for Half Forward Fold it shows "hip N°" (the saved hip angle); for Warrior III it shows "N° from level" (90 − tilt).
5. **Hold countdown** (once the timer has started): centred at (W/2, H×0.1), bold, size max(36, W×0.08). It shows the remaining whole seconds in white while active, white at 45% opacity while paused, and a green "✓" when done.

**Mirrored text:** because the canvas is flipped by CSS, all text is drawn with a horizontal counter-flip (translate to the point, scale(−1, 1)), centred on both axes, so it reads correctly on screen.

## 12. Start-up, camera and errors

- **On load:** select Mountain. If the page is opened as `file:`, show the error "⚠ This page is opened as a local file, and Chrome blocks the camera on file:// pages (that's why no permission prompt appears). Run it from a local server instead — see the note below." If `navigator.mediaDevices.getUserMedia` is missing, show "This browser doesn't expose camera access here. Use Chrome, and open the page via http://localhost (see the note below)."
- **Start** (camera first, then the model):
  1. Disable Start and show "Requesting camera…". Request `video: {width ideal 960, height ideal 720, facingMode "user"}`, with no audio. Play the video, hide the placeholder, set running, enable Stop, and begin the frame loop.
  2. Camera errors (re-enable Start):
     - NotAllowed/Permission on `file:` → the file:// message above in camera form ("Camera blocked because this page is opened as a local file…")
     - NotAllowed/Permission otherwise → "Camera permission was denied and no prompt appeared. Two things to check: (1) click the camera icon in Chrome's address bar and set it to Allow; (2) on Mac, open System Settings ▸ Privacy & Security ▸ Camera and make sure Chrome is switched on. Then press Start again."
     - NotFound → "No camera was found…"
     - NotReadable → "The camera is busy — another app (Zoom, FaceTime, Photo Booth…) may be using it…"
     - anything else → "Couldn't open the camera: {message}"
  3. If the model isn't loaded yet, show "Downloading pose model (first time only)…" and load it (§3). If offline → "You appear to be offline. The pose model has to download once from the internet — reconnect and press Start again. (Your camera is working.)" If a download fails → "Camera works, but the pose model couldn't download — a network or firewall is blocking it…". Any other failure → "Camera works, but the pose model failed to load: {message}". On success, briefly show "Model ready (GPU|CPU).", then "Tracking… step back so your whole body is in frame." The camera keeps running even if the model fails.
- **Frame loop** (`requestAnimationFrame`): when the video has data, match the canvas size to the video. Only when `video.currentTime` has changed, run `detectForVideo(video, performance.now())`, clear the canvas, then evaluate (§8) and draw (§11).
- **Stop:** cancel speech, stop the camera tracks, show the placeholder, re-enable Start and disable Stop, clear the canvas and line smoothing, reset the hold timer, show "Stopped.", and reset the ring to "—" and "Waiting…".

## 13. Acceptance checks (to confirm a rebuild matches)

1. Standing straight in Mountain pose facing the camera → every line is green, the score climbs to 100, the timer counts 10 → ✓, and the spine readout is near 0° (not ~180°).
2. Raising one shoulder in Mountain → only the shoulder line and its guide turn red; the score drops to 75.
3. In Mountain, the arms stay grey the whole time.
4. Half Forward Fold side-on with a flat back and a hip angle under 90° → green, with "hip N°" shown. Rounding the back → the torso turns red and "Back straight" fails. Standing up → "Hinged" fails.
5. Warrior III with the lifted leg dropped → only that leg is red; the message is "Lift the back leg up to hip height."
6. Stepping out of a pose for about 2 s mid-hold → the timer shows "Paused…", then resumes from where it was. A 0.3 s tracking blip doesn't pause it.
7. Changing a pose's timer to 20 s, reloading the page → it's still 20 s for that pose and 10 s for the others.
8. With voice on, at full alignment → praise plus "Hold for N seconds", "Five more seconds", then "Well done. Release the pose."

## 14. Known limitations

- Thresholds are hand-picked and not yet tuned with real users; the side-on poses (Warrior I/III, Fold) need real-camera testing.
- The app is 2D only: poses judged side-on can't check things that are only visible from the front (e.g. hips square in Warrior I).
- Single person only; the first detected body is used.
- The help text still refers to `yoga-teacher.html` in the local-server tip (the hosted file is `index.html`).

## 15. Version history

| Ver | Date | Change |
|---|---|---|
| 1.0 | 2026-08-21 | Prototype: Mountain, Tree, Warrior II; cue checklist; score ring; robust camera/model start-up |
| 1.1 | 2026-08-21 | Voice coaching |
| 1.2 | 2026-09-27 | Hosted on GitHub Pages as SamajYoga; aspect-ratio correction in angle maths; centre, level and spine guides with degree readout; slower, deeper voice (0.82/0.78) |
| 1.3 | 2026-09-28 | Colour-coded skeleton (red/green/grey per line) + legend; tilt helpers made direction-independent (fixed ~180° spine reading) |
| 1.4 | 2026-09-28 | Warrior I, Warrior III, Half Forward Fold; per-pose hold timer (default 10 s); guides follow each pose's checks; score can reach 100 |
