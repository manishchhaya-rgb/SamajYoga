# SamajYoga — Specification

| | |
|---|---|
| **Spec version** | 1.10 |
| **Date** | 2026-11-14 |
| **Matches code** | `index.html` in `manishchhaya-rgb/SamajYoga`, commit "v1.10: 20 poses — 14 new standing, kneeling, floor and seated poses" |
| **Live URL** | https://manishchhaya-rgb.github.io/SamajYoga/ |

**Purpose of this document:** anyone (or Claude) given only this file should be able to rebuild the app so it behaves the same. Every threshold, colour, message and timing that affects behaviour is listed here. It is updated with every change to the app, and the version number goes up each time.

---

## 1. Product summary

SamajYoga is a "yoga teacher" that runs in a web browser. It uses the device's front camera to track the user's body, and for the chosen pose it:

- draws the **target pose as a see-through outline** on the video, sized so the user steps into it and fits their body inside it;
- tells the user (on screen and out loud) whether to **face the camera or stand side-on**, and prompts them to turn if they're the wrong way round;
- draws a skeleton over the mirrored video, with each checked body line **red** until aligned and **green** when aligned;
- shows a checklist of alignment cues and a 0–100 **alignment score**;
- runs a **hold timer** that counts only while fully aligned;
- coaches the user through **My sequence** — a saved, editable list of poses with a hold time each (edited in a side menu; a default six-pose sequence is provided) from a library of **20 poses** — 10 done facing the camera and 10 side-on;
- **speaks** pose announcements, facing/placement prompts and corrections, then counts the time during a hold — one prompt at a time, never overlapping (on by default).

Everything runs on the device. No video or data leaves the machine; only the pose model and library are downloaded once.

## 2. Platform and constraints

- **One file:** `index.html` containing all HTML, CSS and JavaScript (`<script type="module">`). No build step, no server code.
- **Hosting:** GitHub Pages (repo root, `main` branch). Must be served over http(s). Chrome blocks the camera on `file://` pages.
- **Devices and browsers:** phone, iPad (portrait and landscape) and laptop — current Chrome (desktop and Android) and Safari (iPhone/iPad/Mac). The layout adapts to each (§4).
- **iOS home-screen app feel:** meta tags `apple-mobile-web-app-capable=yes`, status bar `default`, title "Yoga Teacher", `mobile-web-app-capable=yes`, `theme-color=#F3F1EC`. Viewport: `width=device-width, initial-scale=1.0, maximum-scale=1.0, viewport-fit=cover`.
- **Page title:** "Yoga Teacher — Pose Coach".

## 3. External dependencies

| What | Source |
|---|---|
| MediaPipe Tasks Vision library 0.10.12 (`FilesetResolver`, `PoseLandmarker`) | ES module import from `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.12` |
| WASM files | Try `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.12/wasm`, then fall back to `https://unpkg.com/@mediapipe/tasks-vision@0.10.12/wasm` |
| Pose model | `https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_lite/float16/1/pose_landmarker_lite.task` |
| Speech | Browser built-in Web Speech API (`speechSynthesis`) — no download |
| Fonts | Google Fonts: **Fraunces** (opsz 9..144, weights 400/500/600) for headings and numbers, **Manrope** (400–800) for everything else, via `fonts.googleapis.com` `css2` with `display=swap`. Fallbacks: Georgia/Times for Fraunces; the system sans stack for Manrope. |

**Model options:** `runningMode: "VIDEO"`, `numPoses: 1`, `minPoseDetectionConfidence`, `minPosePresenceConfidence`, `minTrackingConfidence` all `0.5`. Try `delegate: "GPU"` first; if that fails, try `"CPU"`.

## 4. Screen layout — "Studio Calm" design

A light, calm look: warm stone background, white cards, sage-green accent, serif headings. The app has **two screens** — **Home** and **Session** — plus a **side menu** (drawer). Only one screen shows at a time (`hidden` on the other); switching scrolls to the top.

**Design tokens (CSS variables):**

| Token | Value | Use |
|---|---|---|
| `--bg` | `#F3F1EC` | page background (warm stone) |
| `--card` | `#FFFFFF` | cards, pills, on-video overlays |
| `--ink` | `#1F2A24` | main text |
| `--muted` | `#5C665F` | secondary text |
| `--line` | `#DAD6CC` | borders, empty progress segments |
| `--line-soft` | `#E6E2D8` | dividers, empty bars, card shadow (0 1px 0) |
| `--sage` | `#4F6E5B` | accent: primary button, icons, links, progress, good score |
| `--sage-dark` | `#3A5444` | primary button hover |
| `--sage-mid` | `#9DB3A4` | current progress segment, focus ring |
| `--sage-soft` | `#E6ECE5` | icon tiles, "Now" row, good badges |
| `--good-text` | `#2F6B45` | "inside the outline", ✓ marks |
| `--amber` / `--amber-soft` | `#9A4F1E` / `#F6E6D8` | corrections and wrong-way prompts |
| `--camera` | `#46534B` | camera card behind the video |

Fonts: `--serif` Fraunces (headings, pose name, score, queue countdown, wordmark), `--sans` Manrope (everything else). Focus ring: 3px `--sage-mid`, offset 2px. Shared controls: **round button** 44×44, radius 22, 1px `--line` border, white; **pill button** 44 high, radius 22, 14px/600 with a 16px icon; **primary button** 56 high, radius 28, `--sage` fill, white 17px/700 text with a play icon; **small button** 34 high, radius 17, 13px/600; **text button** sage 14px/700, no border. Icons are inline line SVGs (stroke `currentColor`), never emoji.

### 4.1 Home screen

- **Top bar** (max-width 1280, padding 14px 20px, respects the top safe area): round menu button (☰ lines; opens the drawer), the wordmark "SamajYoga" (Fraunces 20px/600) in the middle, round voice button on the right (§10).
- **Content** (max-width 1080, centred, padding 8px 20px 32px):
  - Greeting: "{weekday} {morning|afternoon|evening}" (from the device clock: before 12 / before 18 / after; 14px muted) over the heading **"Your practice"** (Fraunces 34px; 42px from 820px width).
  - **Sequence card** (white, radius 24): "Today's sequence" (15px bold) + "Edit" text button (opens the drawer); a meta line "N poses · {total} of holding · one turn / N turns" (turns = the number of times facing changes between consecutive steps; left out if 0); then one row per step (min-height 52): a 42px sage-soft tile with the pose's outline icon (§7.7 icon, sage), the short name (pose name without " Pose"; 15px/600) over "Face the camera" or "Side-on" (12.5px muted), and "N s" on the right. Wherever the facing changes between two steps, a divider row is inserted: turn-arrow icon + "Then turn side-on" / "Then face the camera" (sage 12.5px/700) followed by a 1px line. Empty: "Your sequence is empty — tap Edit to add poses."
  - **"Begin practice"** primary button (disabled when the sequence is empty) — starts the sequence (§9a).
  - Status line (13px muted; amber for errors) — e.g. the file:// warning.
  - **"Practise one pose"** (14px bold), then two groups, each starting with a full-width group label (sage 12.5px/700 with a 15px icon): person icon + "Facing the camera", then two-arrows icon + "Side-on". Each group holds one tile per pose of that facing, in pose order (§7.8): outline icon 40px over the short name (12.5px/600, line-height 1.25, wraps; 11.5px on ≤ 380px), min-height 96px, padding 10px 4px, radius 18, 1px border, in a 3-column grid with 10px gaps. Tapping a tile selects that pose and opens the Session screen.
  - **Tips card**: three rows (34px sage-soft icon tile + bold title + muted text): "Head to toe — Prop your device so the camera sees your whole body." / "Step into the outline — Each pose appears as an outline — fit your body inside it." / "Listen for cues — The voice tells you which way to face and what to adjust."
- **Phone (< 820px):** one column in the order greeting → sequence card → Begin → status → pose tiles (the same 3-column grid) → tips.
- **iPad and laptop (≥ 820px):** two columns (`1.15fr` / `1fr`, gap 28): left = sequence card, Begin, status; right = pose tiles (3-column grid) and tips.

### 4.2 Session screen

- **Top bar:** pill button "✕ End" (stops the camera and any sequence, returns Home); in the middle the progress label — "Pose N of M" during a sequence, "Sequence complete" when finished, "Practising one pose" otherwise — with, during/after a sequence, one 4px segment per step (max 26px each, row width min(200px, 46vw)): done = sage, current = sage-mid, upcoming = `--line`; on the right the round voice button.
- **Stage** (camera card): radius 28 (22 on ≤ 480px), background `--camera`, aspect ratio = the camera's (`videoWidth/videoHeight`, default 4:3), width `min(100%, stage-max-height × aspect)`, centred. It holds the `<video>` (`playsinline muted`, `object-fit: contain`) and the `<canvas>`; **both mirrored with `transform: scaleX(-1)`**. While the camera is off a centred placeholder shows a camera icon and a message ("Starting the camera…", an error message — light orange `#FFE2CC` for errors — or "Camera off."). Un-mirrored overlays (white, §11a): facing **chip** (top-left), **pose queue** (top-right), **facing card** (centre), **completion card** (centre), **placement hint** (bottom-centre).
- **Coaching panel:**
  - **Pose head:** short pose name (Fraunces 30px) over the Sanskrit name (13px italic muted); on the right the **score** "N%" (Fraunces 32px/600; "%" at 16px) over "alignment" (12px muted). Score colour: sage when ≥ 80, amber when < 50, muted otherwise; "—" when idle.
  - **Coach card** (white, radius 20): a 38px round badge with an icon + a bold 15.5px title over a 13px muted line. It always shows the single most useful thing to do now (first match):
    1. camera not running: camera icon, "Camera off" / "Press End to go back, then begin again." (while starting: "Starting the camera…" / "Allow camera access if your browser asks.");
    2. model not loaded: "Getting ready…" / "Loading the pose tracker. Step back so your whole body is in view.";
    3. no body: person icon, "Step into view" / "Your whole body should be visible, head to toe.";
    4. facing wrong (§11a): amber badge, turn icon, "Turn to face the camera" / "Turn side-on to the camera", "The outline shows the pose from this angle.";
    5. a cue failing: amber badge, target icon, the **first failing cue's message** as the title and its label as the sub-line;
    6. hold complete: sage badge, check icon, "Hold complete — well done" / "Moving on in a moment…" (sequence) or "Release the pose when you're ready.";
    7. otherwise: check icon, "Lovely — hold it there" / "All N checks are green."
  - **Hold** (§9): status text (left) and "E of T s" (right, 13px/600 muted; green when done), an 8px bar (sage on `--line-soft`), then tools: in single-pose practice "−  Ns  +"; always "Restart hold"; during a sequence "Skip pose ⏭".
  - **Description** (14px muted).
  - **Checks card** (white): "Checks · P of N aligned" (13px/700 muted; just "Checks" when idle), then one row per cue: a 20px round dot (✓ sage-soft/good-text, ! amber-soft/amber, • grey when idle with the message "Get into position…"), the cue label and its message (12.5px muted).
  - **Legend** (12px muted): outline swatch "Target outline", red "Adjust" (`#F0504F`), green "Aligned" (`#34C46A`), grey "Not checked".
  - **Status line** (13px; amber for errors).
- **Layouts:**
  - **Phone (< 700px):** one column — stage (max height 58vh), pose head, coach card, hold, description, checks, legend, status.
  - **iPad portrait / narrow windows (700–1023px, unless landscape ≥ 900px):** stage on top (max height 60vh); below it the pose head across the full width, then two columns: left = coach card, hold, description; right = checks, legend.
  - **Laptop and iPad landscape (≥ 1024px, or ≥ 900px in landscape):** two columns — stage on the left (max height `100vh − 110px`), coaching panel on the right (360px, sticky at top 12px) with everything stacked.
  - ≤ 480px: tighter side padding (12px), smaller chip and queue (§11a).

### 4.3 Side menu (drawer)

Fixed to the left edge, full height, width `min(400px, 92vw)`, background `--bg`, slides in from the left (0.25 s); a backdrop `rgba(31,42,36,.35)` covers the page. Opened from the Home menu button or "Edit"; closed by the ✕ round button, the backdrop or Esc. Contents: heading "Your sequence" (Fraunces 24px); sub-text "The poses you'll be coached through, in order, and how long to hold each. Saved on this device — change it any time."; the **step editor** (§9a); "+ Add pose" and "Reset to default" small buttons; the total line; a primary "Begin practice" button (closes the drawer and starts the sequence); a collapsible "Help & tips" section covering set-up and privacy, the outline, facing, skeleton colours, the hold timer, the sequence (default turns once), voice, and what to do if the model won't download (`python3 -m http.server 8000`, then `http://localhost:8000/index.html`).

**Pose outline icons** (sequence rows, tiles, step editor, queue, facing card) are small SVGs drawn from the outline data in §7.7: each bone a round-capped line in `currentColor` with stroke width 0.75 × the bone's thickness, the head a filled circle, in a square viewBox fitted to the pose's bounding box (+0.02 padding), y flipped so up is up.

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

"Torso tilt" means `tiltFromVertical(mid-hips, mid-shoulders)`. Every pose has a **facing** ("front" = face the camera, "side" = stand side-on) and a **spoken name** used by the voice.

### 7.1 Mountain Pose · Tadasana
Facing: **front**. Spoken name: "Mountain pose".
Description: "Stand tall, feet grounded, arms relaxed at your sides, spine long."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Spine tall & vertical | torso tilt < 10° | TORSO | "Nicely stacked." / "Lengthen up — you're leaning to one side." |
| Shoulders level | \|y11 − y12\| < 0.04 | SHOULDERS | "Even and relaxed." / "Drop the raised shoulder to level them." |
| Hips level | \|y23 − y24\| < 0.04 | HIPS | "Weight is even." / "Balance your weight evenly on both feet." |
| Legs straight | each knee > 160° (per side) | LEG l / LEG r | "Strong, straight legs." / "Gently straighten your knees." |

### 7.2 Tree Pose · Vrksasana
Facing: **front**. Spoken name: "Tree pose".
Description: "Balance on one leg; place the other foot on your inner thigh or calf. Raise your hands overhead and grow tall."
Standing leg = the side with the larger knee angle (ties go to left); the other is the lifted leg.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Standing on one leg | standing knee > 160° AND lifted knee < 130° | LEG stand (ok = stand > 160), LEG lift (ok = lift < 130) | "Good — one leg lifted." / "Lift one foot and rest it against the standing leg." |
| Standing leg strong | standing knee > 168° | LEG stand | "Rooted and steady." / "Press down and straighten the standing leg." |
| Hips level & balanced | \|y23 − y24\| < 0.06 | HIPS | "Steady hips." / "Level the hips — don't jut one out." |
| Arms lifted overhead | each wrist y < nose y (per side) | ARM l / ARM r | "Reaching up like branches." / "Raise both hands overhead (or to heart center)." |
| Torso upright | torso tilt < 14° | TORSO | "Tall and centered." / "Stack your chest over your hips." |

### 7.3 Warrior I · Virabhadrasana I
Facing: **side**. Spoken name: "Warrior one".
Description: "Lunge with the front knee toward 90°, back leg long, both arms reaching straight up, chest lifted."
Front leg = the side with the smaller knee angle (ties go to left); back leg = the side with the larger angle.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 120° | LEG front | "Strong front knee." / if > 120: "Bend the front knee more toward 90°." else "Ease up — keep the knee over the ankle." |
| Back leg straight | back knee > 150° | LEG back | "Back leg long and strong." / "Straighten the back leg and press through the back heel." |
| Arms reaching straight up | per side: wrist y < nose y AND tiltFromVertical(shoulder, wrist) < 25° | ARM l / ARM r | "Arms reaching high." / "Reach both arms straight up beside your ears." |
| Arms straight | per side: elbow angle (shoulder–elbow–wrist) > 150° | ARM l / ARM r | "Long through the fingertips." / "Straighten the elbows." |
| Torso upright | torso tilt < 15° | TORSO | "Chest lifted over the hips." / "Lift the chest — don't lean forward over the front knee." |

### 7.4 Warrior II · Virabhadrasana II
Facing: **front**. Spoken name: "Warrior two".
Description: "Wide stance, front knee bent toward 90°, back leg straight, arms reaching out level with the floor."
Front leg = smaller knee angle (ties go to left); back leg = larger (ties go to right).

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 120° | LEG front | "Deep, strong lunge." / if > 120: "Bend the front knee more toward 90°." else "Ease up — don't let the knee pass the ankle." |
| Back leg straight | back knee > 155° | LEG back | "Strong back leg." / "Straighten and energize the back leg." |
| Arms level with floor | per side: tiltFromHorizontal(shoulder, wrist) < 20° | ARM l / ARM r | "Arms floating, parallel to the floor." / "Lift both arms to shoulder height, reaching out." |
| Arms extended | per side: elbow angle > 155° | ARM l / ARM r | "Long through the fingertips." / "Straighten the elbows and reach wide." |
| Torso upright | torso tilt < 18° | TORSO | "Chest stacked over hips." / "Don't lean over the front leg — lift the torso." |

### 7.5 Warrior III · Virabhadrasana III
Facing: **side**. Spoken name: "Warrior three".
Description: "Balance on one straight leg, tip forward so torso and lifted leg form one level line, arms reaching forward."
Standing leg = the side whose ankle is lower on screen (larger y; ties go to left); the other is the lifted leg.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Standing leg straight | standing knee > 160° | LEG stand | "Rooted standing leg." / "Straighten the standing leg — a micro-bend is fine." |
| Lifted leg level & straight | tiltFromHorizontal(lifted hip, lifted ankle) < 20° AND lifted knee > 150° | LEG lift | "Back leg long and level." / if not level: "Lift the back leg up to hip height." else "Straighten the lifted leg and reach through the heel." |
| Torso level with floor | tiltFromHorizontal(mid-hips, mid-shoulders) < 20° | TORSO | "Torso level — a perfect T." / "Tip the chest forward until your torso is level with the floor." |
| Arms reaching forward | per side: tiltFromHorizontal(shoulder, wrist) < 25° AND elbow angle > 150° | ARM l / ARM r | "Reaching long through the fingertips." / "Reach both arms straight forward, in line with your torso." |

### 7.6 Half Forward Fold · Ardha Uttanasana
Facing: **side**. Spoken name: "Half forward fold".
Description: "Hinge forward from the hips with a long, flat back until the angle between legs and torso is less than 90°. Hands can rest on your shins or the floor."
**Side selection:** add up visibility of hip + knee + ankle + shoulder for each side (missing visibility counts as 1); use the side with the larger total (ties go to left). **Head point** = that side's ear if its visibility > 0.4, otherwise the nose. Arms are **not** checked (hands may rest on the floor), so they show grey.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Hinged from the hips (< 90°) | angle(ankle, hip, head) < 90° — this value is also saved for the on-screen readout | TORSO | "Good deep hinge from the hips." / "Hinge further forward from the hips." |
| Back straight | angle(hip, shoulder, head) > 150° | TORSO | "Long, flat back." / "Lengthen your spine — reach the crown of your head forward, don't round." |
| Legs straight | each knee > 155° (per side) | LEG l / LEG r | "Legs long and steady." / "Straighten your legs — a soft bend in the knees is fine." |

### 7.7 Target pose outlines

Each pose has an outline ("template") of the target shape **as seen on screen** (the mirrored view). Units: 1.0 = standing body height; x to the right, y **up from the floor**. Pairs are [screen-left, screen-right]. Side-on poses face screen-right; asymmetric front poses have the bent/lifted leg on screen-right. (The outline is flipped at run time to match the user — §11a.)

| Pose | side? | head | shoulders | elbows | wrists | hips | knees | ankles |
|---|---|---|---|---|---|---|---|---|
| mountain | no | (0, .93) | (±.11, .82) | (±.13, .65) | (±.14, .48) | (±.07, .52) | (±.07, .28) | (±.07, .04) |
| tree | no | (0, .93) | (±.11, .82) | (±.15, .98) | (±.04, 1.12) | (±.07, .52) | L (−.07, .28), R (.25, .37) | L (−.07, .04), R (−.03, .32) |
| warrior2 | no | (0, .77) | (±.11, .66) | (±.28, .66) | (±.45, .66) | (±.07, .36) | L (−.235, .20), R (.31, .30) | L (−.40, .04), R (.32, .04) |
| warrior1 | yes | (.03, .79) | both (0, .68) | both (.01, .86) | both (.02, 1.02) | both (0, .38) | L (−.19, .22), R (.23, .30) | L (−.38, .06), R (.25, .04) |
| warrior3 | yes | (.40, .555) | both (.30, .54) | both (.47, .55) | both (.63, .555) | both (0, .52) | L (0, .28), R (−.24, .53) | L (0, .04), R (−.48, .54) |
| fold | yes | (.36, .41) | both (.27, .44) | both (.25, .27) | both (.22, .12) | both (−.02, .52) | both (0, .28) | both (0, .04) |

| salute | no | (0, .93) | (±.11, .82) | (±.12, .99) | (±.12, 1.16) | (±.07, .52) | (±.07, .28) | (±.07, .04) |
| sidebend | no | (.141, .905) | L (0, .84), R (.206, .764) | L (.085, .987), R (.291, .911) | L (.17, 1.134), R (.376, 1.058) | (±.07, .52) | (±.07, .28) | (±.07, .04) |
| star | no | (0, .87) | (±.11, .76) | (±.23, .88) | (±.35, 1.0) | (±.07, .46) | (±.185, .25) | (±.30, .04) |
| goddess | no | (0, .71) | (±.11, .60) | (±.28, .60) | (±.28, .77) | (±.08, .30) | (±.30, .28) | (±.30, .04) |
| triangle | no | (.336, .685) | L (.183, .712), R (.309, .532) | L (.183, .882), R (.309, .362) | L (.183, 1.052), R (.309, .192) | (±.07, .45) | (±.195, .245) | (±.32, .04) |
| sideangle | no | (.355, .565) | L (.205, .605), R (.315, .415) | L (.338, .712), R (.315, .245) | L (.47, .82), R (.315, .075) | (±.07, .36) | L (−.235, .20), R (.31, .30) | L (−.40, .04), R (.32, .04) |
| reverse | no | (−.174, .732) | L (−.227, .585), R (−.027, .679) | L (−.26, .42), R (−.085, .839) | L (−.30, .26), R (−.143, .999) | (±.07, .36) | L (−.235, .20), R (.31, .30) | L (−.40, .04), R (.32, .04) |
| chair | yes | (.155, .756) | both (.092, .666) | both (.164, .82) | both (.236, .974) | both (−.08, .42) | both (.12, .248) | both (0, .04) |
| standfold | yes | (.147, .151) | both (.10, .25) | both (.17, .12) | both (.25, .03) | both (−.03, .52) | both (0, .28) | both (0, .04) |
| lowlunge | yes | (.08, .71) | both (.07, .60) | both (.08, .77) | both (.09, .94) | both (.06, .30) | L (−.10, .08), R (.30, .28) | L (−.33, .03), R (.30, .04) |
| downdog | yes | (.24, .17) | both (.205, .252) | both (.333, .14) | both (.46, .028) | both (−.02, .45) | both (−.16, .245) | both (−.30, .04) |
| plank | yes | (.41, .29) | both (.304, .262) | both (.304, .14) | both (.304, .02) | both (.014, .184) | both (−.218, .122) | both (−.45, .06) |
| staff | yes | (.01, .48) | both (0, .37) | both (0, .22) | both (.02, .07) | both (0, .08) | both (.24, .07) | both (.48, .06) |
| boat | yes | (−.264, .394) | both (−.193, .31) | both (−.02, .31) | both (.15, .31) | both (0, .08) | both (.17, .25) | both (.34, .42) |

Default direction of the asymmetric front poses: Side Bend leans to screen-right; Triangle and Extended Side Angle have the front leg and the lean on screen-right; Reverse Warrior has the bent knee on screen-right and leans back to screen-left. Side-on poses face screen-right (Low Lunge: front leg on screen-right).

**Feet:** Warrior III uses explicit toe points L (.07, .01), R (−.49, .47); Low Lunge L (−.40, .02), R (.37, .01); Plank both (−.43, .01); Staff both (.50, .13) (toes pointing up). Otherwise: side-on poses put each toe at ankle + (.07, −.03); front poses at ankle + (±.03, −.03), pointing away from the centre (sign of the ankle's x; x = 0 counts as +).

**Bones and thickness** (thickness in body units; drawn as round-capped lines): per side shoulder→elbow 0.07, elbow→wrist 0.06, hip→knee 0.11, knee→ankle 0.08, ankle→toe 0.05; torso mid-hips→mid-shoulders 0.20 (front) / 0.15 (side); neck mid-shoulders→head 0.06; front poses only: shoulder→shoulder 0.08 and hip→hip 0.11. Head: circle, radius 0.065.

**Bounding box:** over all bone end points ± half their thickness and the head circle; the bottom is never above y = 0.

### 7.8 Helpers for the poses added in v1.10

- **Torso tilt** as above. **Elbow angle** (side s) = angle(shoulder, elbow, wrist).
- **Stance ratio** = |x27 − x28| ÷ |x11 − x12| (feet apart compared with shoulder width, both across the picture; shoulder span floored at 0.001).
- **Front leg** = the side with the smaller knee angle (ties go to left); **back leg** = the other side.
- **Visible side** (side-on poses) = the Fold's side selection (§7.6), returning that side's hip, knee, ankle, shoulder, elbow, wrist and ear; **head point** as in §7.6.
- **Per-side** checks give one mark per side (ARM l/r or LEG l/r) and pass only if both sides pass.
- **Arms up** = per side: wrist y < nose y AND elbow angle > 150°.

**Pose order** (step-editor menus and Home tiles): Mountain, Upward Salute, Tree, Standing Side Bend, Five-Pointed Star, Goddess, Warrior II, Reverse Warrior, Extended Side Angle, Triangle (facing the camera); Chair, Standing Forward Fold, Half Forward Fold, Warrior I, Low Lunge, Warrior III, Downward-Facing Dog, Plank, Staff, Boat (side-on). Keys: `mountain, salute, tree, sidebend, star, goddess, warrior2, reverse, sideangle, triangle, chair, standfold, fold, warrior1, lowlunge, warrior3, downdog, plank, staff, boat`.

### 7.9 Upward Salute · Urdhva Hastasana (`salute`)
Facing **front**. Spoken "Upward salute". Description: "Stand tall with your feet together and sweep both arms straight up overhead, palms facing, shoulders relaxed."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Arms straight up | per side: wrist y < nose y AND tiltFromVertical(shoulder, wrist) < 20° | ARM l / r | "Arms reaching straight up." / "Reach both arms straight up beside your ears." |
| Elbows straight | per side: elbow > 155° | ARM l / r | "Long arms." / "Straighten your elbows." |
| Spine tall & vertical | torso tilt < 10° | TORSO | "Tall and lifted." / "Stand up straight — don't lean to one side." |
| Legs straight | per side: knee > 160° | LEG l / r | "Strong legs." / "Gently straighten your knees." |

### 7.10 Standing Side Bend · Parsva Tadasana (`sidebend`)
Facing **front**. Spoken "Standing side bend". Description: "Feet together and arms overhead, lean your upper body over to one side in a long curve. Keep both feet grounded."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Arms overhead | per side: wrist y < nose y | ARM l / r | "Arms reaching overhead." / "Keep both arms reaching up overhead." |
| Side bend (12–40°) | 12° ≤ torso tilt ≤ 40° | TORSO | "Lovely long side bend." / if < 12: "Lean further over to one side." else "Come up a little — keep the bend long, not collapsed." |
| Hips level | \|y23 − y24\| < 0.05 | HIPS | "Hips steady." / "Keep your hips level — bend from the waist." |
| Legs straight | per side: knee > 160° | LEG l / r | "Feet grounded, legs strong." / "Straighten your legs." |

### 7.11 Five-Pointed Star · Utthita Tadasana (`star`)
Facing **front**. Spoken "Five pointed star". Description: "Step your feet wide, legs straight, and reach your arms out and up in a wide V, like a star."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Feet wide apart | stance ratio > 1.6 | LEG l + LEG r (same result) | "Nice wide stance." / "Step your feet wider apart." |
| Legs straight | per side: knee > 160° | LEG l / r | "Strong, straight legs." / "Straighten your knees." |
| Arms up and out in a V | per side: wrist y < shoulder y AND 15° ≤ tiltFromHorizontal(shoulder, wrist) ≤ 65° AND elbow > 150° | ARM l / r | "Arms reaching wide." / "Reach your arms up and out in a wide V." |
| Torso upright | torso tilt < 10° | TORSO | "Tall through the centre." / "Stack your chest over your hips." |

### 7.12 Goddess Pose · Utkata Konasana (`goddess`)
Facing **front**. Spoken "Goddess pose". Description: "Step wide with toes turned out, bend both knees to sink your hips, keep your chest upright, and bend your elbows to make cactus arms."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Both knees bent (~90°) | per side: 80° ≤ knee ≤ 130° | LEG l / r | "Deep, strong squat." / if either knee > 130: "Bend both knees and sink your hips lower." else "Come up a little — keep the knees over the ankles." |
| Feet wide apart | stance ratio > 1.8 | LEG l + LEG r | "Wide, steady base." / "Step your feet wider apart." |
| Cactus arms | per side: 60° ≤ elbow ≤ 120° AND wrist y < elbow y AND tiltFromHorizontal(shoulder, elbow) < 25° | ARM l / r | "Strong cactus arms." / "Lift your elbows to shoulder height, hands pointing up." |
| Torso upright | torso tilt < 12° | TORSO | "Chest lifted." / "Keep your chest upright over your hips." |

### 7.13 Reverse Warrior · Viparita Virabhadrasana (`reverse`)
Facing **front**. Spoken "Reverse warrior". Description: "From Warrior II, keep the front knee bent, slide your back hand down the back leg and sweep the front arm up and back overhead."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 125° | LEG front | "Front knee strong." / if > 125: "Keep the front knee bent toward 90°." else "Ease up — keep the knee over the ankle." |
| Back leg straight | back knee > 150° | LEG back | "Back leg long." / "Straighten the back leg." |
| Front arm up and back | front-side wrist y < nose y AND front-side elbow > 150° | ARM front | "Front arm reaching up and back." / "Sweep your front arm up overhead." |
| Leaning back over the back leg | the shoulder midpoint is on the same side of the hip midpoint (in x) as the back ankle, AND 8° ≤ torso tilt ≤ 45° | TORSO | "Lovely lean over the back leg." / if leaning the right way but > 45: "Ease up — keep the lean gentle." else "Lean your torso back over the back leg." |

### 7.14 Extended Side Angle · Utthita Parsvakonasana (`sideangle`)
Facing **front**. Spoken "Extended side angle". Description: "From Warrior II, lean your torso over the bent front knee, bring the lower hand down inside the front foot, and reach the top arm over your ear in one line with the back leg." The lower arm is not checked (grey).

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 80° ≤ front knee ≤ 120° | LEG front | "Strong front knee." / if > 120: "Bend the front knee more toward 90°." else "Ease up — keep the knee over the ankle." |
| Back leg straight | back knee > 155° | LEG back | "Back leg long and strong." / "Straighten the back leg and press through the outer foot." |
| Torso over the front thigh | 40° ≤ torso tilt ≤ 80° | TORSO | "Torso long over the front leg." / if < 40: "Lean your torso further over the front knee." else "Lift your chest a little — don't collapse onto the thigh." |
| Top arm reaching over your ear | top arm = the side whose wrist is higher (smaller y; ties go to right): wrist y < shoulder y AND elbow > 150° AND 20° ≤ tiltFromVertical(shoulder, wrist) ≤ 75° | ARM top | "Long line from back foot to fingertips." / "Reach the top arm up and over your ear." |

### 7.15 Triangle Pose · Trikonasana (`triangle`)
Facing **front**. Spoken "Triangle pose". Description: "Wide stance, both legs straight. Reach over your front leg, then tip your torso sideways — lower hand to your shin, top arm straight up, arms in one line."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Legs straight | per side: knee > 160° | LEG l / r | "Both legs long and straight." / "Straighten both legs — soft knees, not bent." |
| Feet wide apart | stance ratio > 1.8 | LEG l + LEG r | "Wide, steady base." / "Step your feet wider apart." |
| Torso tipped sideways | 35° ≤ torso tilt ≤ 80° | TORSO | "Long sideways reach." / if < 35: "Tip your torso further over the front leg." else "Lift your chest a little — keep both sides of the waist long." |
| Arms in one vertical line | per side: tiltFromVertical(shoulder, wrist) < 25° AND elbow > 150° | ARM l / r | "Arms in one long line." / "Stack your arms — one reaching down, one straight up." |

### 7.16 Chair Pose · Utkatasana (`chair`)
Facing **side**. Spoken "Chair pose". Description: "Feet together, bend your knees and sit your hips back as if into a chair, chest lifted, arms reaching up beside your ears."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Knees bent | 80° ≤ average knee angle ≤ 135° | LEG l + LEG r | "Nice deep chair." / if > 135: "Bend your knees more and sit your hips back." else "Come up a little — keep your knees above your toes." |
| Torso leaning forward a little | 15° ≤ torso tilt ≤ 50° | TORSO | "Chest lifted, torso long." / if < 15: "Sit your hips back and lean forward slightly." else "Lift your chest — don't fold forward." |
| Arms reaching up | arms up (§7.8) | ARM l / r | "Arms reaching up." / "Reach both arms up beside your ears." |

### 7.17 Standing Forward Fold · Uttanasana (`standfold`)
Facing **side**. Spoken "Standing forward fold". Description: "Fold forward from the hips with your legs straight and let your head and arms hang down toward the floor. Bend your knees a little if you need to." Uses the visible side and head point. Arms not checked.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Deep fold (< 50°) | angle(ankle, hip, head) < 50° — saved for the "hip N°" readout | TORSO | "Deep, relaxed fold." / "Fold further — let your chest come toward your thighs." |
| Head below the hips | head y > hip y + 0.05 | TORSO | "Head hanging heavy." / "Let your head drop below your hips." |
| Legs straight | per side: knee > 150° | LEG l / r | "Legs long." / "Straighten your legs as much as is comfortable." |

### 7.18 Low Lunge · Anjaneyasana (`lowlunge`)
Facing **side**. Spoken "Low lunge". Description: "Step one foot forward with the knee over the ankle, lower your back knee to the floor, lift your chest and reach both arms up."

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Front knee bent (~90°) | 75° ≤ front knee ≤ 115° | LEG front | "Front knee over the ankle." / if > 115: "Bend the front knee more." else "Ease forward less — keep the knee over the ankle." |
| Back knee down | back knee y > front knee y + 0.08 | LEG back | "Back knee resting down." / "Lower your back knee to the floor." |
| Chest lifted | torso tilt < 20° | TORSO | "Chest lifted over the hips." / "Lift your chest up over your hips." |
| Arms reaching up | arms up (§7.8) | ARM l / r | "Arms reaching high." / "Reach both arms up overhead." |

### 7.19 Downward-Facing Dog · Adho Mukha Svanasana (`downdog`)
Facing **side**. Spoken "Downward dog". Description: "Hands and feet on the floor, lift your hips high to make an upside-down V. Arms and legs long, head between your arms." Uses the visible side.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Hips high (an upside-down V) | 50° ≤ angle(ankle, hip, shoulder) ≤ 110° AND hip y < shoulder y | TORSO | "Hips lifted high — a strong V." / "Lift your hips up and back toward the ceiling." |
| Long back, arms in line | angle(hip, shoulder, wrist) > 145° | TORSO | "Long line from hands to hips." / "Press your chest toward your thighs to lengthen your back." |
| Arms straight | per side: elbow > 150° | ARM l / r | "Strong arms." / "Straighten your arms and press the floor away." |
| Legs straight | per side: knee > 150° | LEG l / r | "Legs long, heels reaching down." / "Straighten your legs — bend them only if your back rounds." |

### 7.20 Plank Pose · Phalakasana (`plank`)
Facing **side**. Spoken "Plank pose". Description: "Hands under your shoulders, legs straight back, body in one straight line from head to heels." Uses the visible side.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Body low and long | tiltFromHorizontal(ankle, shoulder) < 35° | TORSO | "Body long, close to level." / "Come down onto your hands — body nearly level with the floor." |
| One straight line | angle(shoulder, hip, ankle) > 160° | TORSO | "One long, straight line." / if the hip is below the shoulder–ankle line (larger y than the line at the hip's x): "Lift your hips — don't let them sag." else "Lower your hips into one straight line." |
| Shoulders over wrists | per side: tiltFromVertical(shoulder, wrist) < 20° AND elbow > 150° | ARM l / r | "Shoulders stacked over wrists." / "Stack your shoulders over your wrists, arms straight." |
| Legs straight | per side: knee > 160° | LEG l / r | "Legs strong." / "Straighten your legs and press back through your heels." |

### 7.21 Staff Pose · Dandasana (`staff`)
Facing **side**. Spoken "Staff pose". Description: "Sit with your legs straight out in front, feet flexed, spine tall, hands on the floor beside your hips." Arms not checked.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Right angle at the hips | 70° ≤ angle(shoulder, hip, ankle) ≤ 110° (visible side) | TORSO | "Sitting at a right angle." / "Sit down with your legs straight out in front." |
| Sitting tall | torso tilt < 15° | TORSO | "Spine tall." / "Sit up tall — stack your shoulders over your hips." |
| Legs straight | per side: knee > 160° | LEG l / r | "Legs long and active." / "Straighten your legs and flex your feet." |

### 7.22 Boat Pose · Navasana (`boat`)
Facing **side**. Spoken "Boat pose". Description: "Sit, lean back a little and lift your legs so your body makes a V. Keep your chest open and reach your arms forward. Bent knees are fine." Knees are not required to be straight.

| Cue | Passes when | Lines | Messages |
|---|---|---|---|
| Legs lifted | per side: ankle y < hip y AND 20° ≤ tiltFromHorizontal(hip, ankle) ≤ 70° | LEG l / r | "Legs lifted." / "Lift your feet off the floor, shins up." |
| A V shape at the hips | 55° ≤ angle(shoulder, hip, ankle) ≤ 110° (visible side) | TORSO | "Strong V shape." / "Lean back a little and lift your legs to make a V." |
| Chest lifted | torso tilt < 55° | TORSO | "Chest open and lifted." / "Lift your chest — don't round back." |
| Arms reaching forward | per side: tiltFromHorizontal(shoulder, wrist) < 25° AND elbow > 150° | ARM l / r | "Arms reaching forward." / "Reach both arms forward, level with the floor." |

## 8. Scoring and line colours (every frame with a detected body)

1. Run every cue of the current pose.
2. **Line state:** for each line named in any mark, the line is aligned only if **every** mark that touches it is ok (logical AND across cues and marks). Lines not named by any mark are "unchecked".
3. **Line smoothing:** each checked line keeps a value between 0 and 1: `new = old × 0.6 + (aligned ? 1 : 0) × 0.4` (the first frame takes 0 or 1 directly). It is drawn **green if ≥ 0.5, red otherwise**. Unchecked lines are white at 55% `rgba(255,255,255,0.55)`. Reset on pose change and on Stop.
4. **Score:** raw = round(passed cues ÷ total cues × 100). Smoothed = `old × 0.7 + raw × 0.3`, kept as an unrounded number (so it can actually reach 100) and **rounded only for display**. Reset on pose change.
5. Update the checklist and ring, then the hold timer (§9), then voice (§10).
6. If no body is detected: status "No body detected — make sure you're fully in frame." and the canvas is cleared.

## 9. Hold timer

- **Outside a sequence (single-pose practice):** per-pose target, **default 10 s**, adjusted with − / + in **5 s steps**, limited to **5–120 s**. Saved per pose in `localStorage` under key `samajyoga.holdTargets` as JSON `{poseKey: seconds}`. Every storage read and write is wrapped in try/catch, and the app must work without storage. Changing the target restarts the timer.
- **During a sequence:** the target is the current step's seconds (the − / + control is hidden).
- "All ok" means every cue in the current pose passes on that frame.
- The timer starts the first time all cues pass. After that it is **active** while the last all-ok frame was less than **0.6 s** ago (short tracking wobbles don't pause it). Time is added only while active; each frame adds at most 0.25 s.
- **Voice:** the time read-outs during a hold (start, every 10 s, "Five more seconds.", resume after a pause) are in §10. At 0 s, mark it done and say "Well done. Release the pose." (announcement priority, §10).
- **Panel text** (§4.2): not started "Hold · starts when every line is green"; active "Holding — keep breathing"; paused "Hold · paused until all green"; done "Hold complete ✓" (green). Right-hand count "E of T s" (E = whole seconds held, capped at T); "T s" when done.
- **Progress bar:** 8px, fill width = elapsed ÷ target, sage.
- **Single-pose practice** shows the − / + target control; during a sequence it is hidden and "Skip pose ⏭" is shown instead.
- **Reset** (elapsed back to 0, not started, not done) happens on: pose change, Stop, the "Restart hold" button, and changing the target. Reset also clears the record of which time read-outs have been said and any pending "seconds to go".
- When a hold completes, it notifies the sequence (§9a), which moves on if a sequence is running.

## 9a. My sequence

**Data:** an ordered list of steps `[{pose, secs}]`. `pose` is a key of the pose table (the 20 keys in §7.8). `secs` is a whole number from **5 to 300**; anything else is rounded and clamped, and non-numbers become 10. Saved as JSON in `localStorage` under **`samajyoga.sequence`** after every change; every read and write is wrapped in try/catch. (The v1.5 key `samajyoga.routine` is no longer read, so everyone starts from the new default.)

**Default sequence** (facing-the-camera poses first, then the side-on poses, so the user turns only once):

| # | Pose | Facing | Hold |
|---|---|---|---|
| 1 | Mountain Pose | front | 15 s |
| 2 | Tree Pose | front | 20 s |
| 3 | Warrior II | front | 20 s |
| 4 | Warrior I | side | 20 s |
| 5 | Warrior III | side | 15 s |
| 6 | Half Forward Fold | side | 20 s |

Total "6 poses · 1 min 50 s of holding".

**Loading:** if storage holds a valid list, use it, dropping any step whose pose no longer exists and clamping seconds. An **empty list stays empty**. If nothing is stored or the data is unreadable, use the default sequence.

**Pose choices** come straight from the pose table, so a new pose automatically appears in the step editor's menus and the single-pose picker.

**Step editor (in the drawer):** one white card per step (radius 16, padding 10), a grid with two rows: row 1 = step number (13px/700 muted) · 40px sage-soft tile with the outline icon · pose dropdown (options are the pose names, with " (side-on)" added for side poses); row 2 (under the dropdown) = seconds control (− button, number input 5–300 in steps of 5, "s", + button; ±5 s) on the left and ↑ ↓ ✕ on the right (36×36 buttons, radius 10). Inputs use the page background, 1px `--line` border, radius 10, 15px text. ↑ is disabled on the first card and ↓ on the last. During a run the current step has a sage border and sage-soft background, finished steps are at 55% opacity, and **every edit control is disabled**. Empty list: "Your sequence is empty — press “+ Add pose” to start building it."
- "+ Add pose" appends a step with the same pose as the last step (or the first pose if empty) at 10 s.
- "Reset to default" asks "Replace your sequence with the default one (six poses — facing the camera first, then side-on)?" and, if confirmed, restores the default.
- Add and Reset are disabled during a run; the drawer's "Begin practice" is disabled during a run or when the list is empty.
- **Total line:** "N pose(s) · {total} of holding", the time as "X s", "X min" or "X min Y s"; blank when empty.
- Every change re-renders the Home sequence card (§4.1), the session progress bar and the pose queue.

**While a sequence runs** the Session top bar shows "Pose N of M" with segments, the pose queue shows "Now" and what's next (§11a), and the hold area shows "Skip pose ⏭".

**Running:**
- **Start** (Home "Begin practice" or the drawer's "Begin practice"): unlock speech (§10), mark active, clear the outline's floor position, hide the completion card, show the Session screen, go to step 1, and start the camera if it isn't running.
- **Go to step i:** select that step's pose (resets the hold timer to the step's seconds and announces the pose — §10), then refresh the editor, Home card, progress bar and queue.
- When a hold completes (the voice says "Well done. Release the pose."), wait **3 s**, then go to the next step. After the last step: say "Sequence complete. Great work.", mark inactive and finished, reset the hold timer, unlock editing, set the progress label to "Sequence complete" with every segment done, and show the **completion card** on the stage: a check-circle icon, "Practice complete", "N poses · {total} of holding" and a primary "Finish" button (returns Home). While it shows, the canvas shows only the video (no outline, skeleton or evaluation) and no voice prompts are spoken.
- **Skip pose ⏭:** next step immediately (or finish on the last).
- **End** (Session top bar) or **Finish**: stop the camera (which stops any sequence), clear the finished state, hide the completion card and return Home.
- Choosing a single pose (Home tile) stops any sequence first. If the camera fails to start, the sequence is stopped (the Session screen stays, showing the error on the stage; End returns Home).

## 10. Voice coaching

- **On by default.** The choice is saved in `localStorage` key `samajyoga.voice` ("on"/"off"; anything else = on). If the browser has no speech it is off. There is a round **voice button** in the top bar of both screens: a sage speaker-with-waves icon when on, a muted speaker-with-✕ when off; its `aria-label`/`title` is "Voice coaching on — tap to turn off" / "Voice coaching off — tap to turn on". Turning it on says "Voice coaching on. I'll guide your alignment."; turning it off cancels speech. No speech support → error "This browser doesn't support speech. Try the latest Chrome."
- **Unlocking on phones/tablets:** tapping "Begin practice" (Home or drawer) or a pose tile immediately speaks a silent single-space utterance (volume 0) inside the tap, so later speech is allowed.
- **Voice choice:** the first voice whose language matches en-US/GB/AU and whose name matches female|samantha|karen|serena|moira|tessa|google us english|zira; otherwise the first English voice; otherwise the first voice. Re-check when the voice list changes.
- **Delivery:** rate **0.82**, pitch **0.78**, volume 1. All text is cleaned first: "°" → " degrees", "~" → "about ", brackets removed, whitespace collapsed.

### 10.1 One voice channel and priorities

There is only ever **one prompt playing**. Every prompt has a priority:

| Priority | Prompts |
|---|---|
| 1 — cue | placement and alignment corrections |
| 2 — facing | "Turn to face the camera." / "Turn side-on to the camera." |
| 3 — time | hold read-outs (§10.3) |
| 4 — announcement | pose announcement, "Well done. Release the pose.", "Sequence complete. Great work.", "Voice coaching on. I'll guide your alignment." |

- **Is something playing?** The app tracks the current utterance itself: it is "speaking" from the moment a prompt is started until the utterance's end/error event — or, if a browser never fires that event, until an estimate of **1 s + 0.5 s per word** has passed. When it ends, the "last spoke" time is set.
- **Rules when something is already playing:** an **announcement** stops a playing correction/facing/time prompt (cancel, then speak) or, if another announcement is playing, is queued straight after it. **Anything else is not spoken** — corrections are dropped (they will be retried by the normal rules), time read-outs are retried on the next frame until they go out of date (§10.3).
- Ending the session, Stop and turning voice off cancel all speech.

### 10.2 Pose announcement and corrections

- **Pose announcement** (on every pose change, and when the camera starts): shows the facing card for **4 s** (§11a). If voice is on and the camera is running, say (announcement priority) "{spoken name}. {Stand facing the camera. | Stand side-on to the camera.} {description} Step into the outline."
- **Corrections** are periodic, not continuous. The candidate each frame is the first that applies:
  1. **Facing wrong** (§11a): "Turn to face the camera." / "Turn side-on to the camera." (facing priority)
  2. **Placement**, at most **2 times per pose**, when the smoothed size ratio r > 1.25 → "Step back a little, so you fit the outline."; r < 0.78 → "Come a little closer, so you fill the outline."; otherwise |dx| > 0.8 → dx > 0 "Move a little to your left." else "Move a little to your right." (cue priority)
  3. **The first failing cue's message** (checklist order). (cue priority)
- A candidate is spoken only when **all** of these hold:
  - the **hold timer is not active and not done** — while holding, only time read-outs speak (alignment is already all green while the timer runs, apart from the 0.6 s wobble allowance);
  - the **same** candidate has been the top one continuously for **1.5 s** (brief tracking flickers never speak);
  - **nothing is playing** and at least **5 s** have passed since the last prompt ended;
  - it isn't the same line spoken less than **12 s** ago.
- There is no separate praise prompt; the start-of-hold read-out carries the encouragement (§10.3).

### 10.3 Time read-outs (time comes before alignment)

During a hold (time priority), with T = the hold target and "left" = T − elapsed:

| When | Says |
|---|---|
| The first time the hold becomes active (and left > 6 s) | one of "Lovely." / "Beautiful." / "That's it." / "Well aligned." + " Hold for N seconds." (N = left, rounded up) |
| Holds over 20 s: each multiple of 10 from the largest one ≤ T − 5 down to 10, as left reaches it | "N seconds left." |
| Holds of 10 s or more: left reaches 5 | "Five more seconds." |
| The hold resumes after being paused **≥ 2 s** (and left > 6 s) | "Good. N seconds to go." |
| left reaches 0 | "Well done. Release the pose." (announcement priority, §9) |

Examples: 15 s hold → "Hold for 15 seconds." … "Five more seconds." … "Well done. Release the pose." 40 s hold → "Hold for 40 seconds.", "30 seconds left.", "20 seconds left.", "10 seconds left.", "Five more seconds.", "Well done…".

- If a read-out can't be spoken because something is playing (e.g. the pose announcement is still going when the hold starts), it is tried again every frame. A mark read-out goes out of date once left is more than 2 s past its mark; the start and resume read-outs are dropped if left ≤ 6 s. Out-of-date read-outs are skipped silently.
- Whenever a read-out is actually spoken, any mark due within the next **3 s** is marked as said, so two time prompts never come back to back.
- Each read-out is said at most once per hold; the record is cleared when the hold resets (§9).
- *Planned, not built:* for long holds and experienced practitioners, an option to let alignment corrections speak during the hold as well.

## 11. Overlay drawing (canvas matches the video's pixel size)

The canvas is redrawn on every new video frame once the model is loaded (and on every animation frame while the model is still loading, so the outline is visible straight away). Order: clear → **target outline** → (if a body is detected) skeleton, level guides, spine indicator, hold countdown. Line widths scale with the canvas width W (height H); landmark x, y are multiplied by W, H. Because the canvas is shown mirrored, a point at screen position X is drawn at canvas x = W − X.

1. **Target outline** (§7.7 data, laid out as in §11a): the silhouette is painted on an off-screen canvas (every bone as a round-capped line of its thickness, head as a filled circle); a second off-screen copy is hollowed out by repainting the same silhouette with every width/radius reduced by an **edge** of max(3, W×0.007) on each side using `destination-out`, leaving only the outer outline. The fill is drawn at 28% opacity (34% when inside). The outline is drawn at full opacity twice: first with a dark shadow (`rgba(0,0,0,0.85)`, blur max(4, W×0.012)) so it stands out on light or busy backgrounds, then again without the shadow. Colour white `#FFFFFF`, or green `#34C46A` when the user is **inside** (smoothed fit ≥ 0.8 with a body detected).
2. **Skeleton:** base width = max(3, W×0.006), round caps. Each of the 12 lines in its colour (§8: aligned `#34C46A`, needs adjusting `#F0504F`, not checked white at 55%); checked lines 1.3× base width. Joints (the 13 landmarks of §5) are circles of radius max(4, W×0.008): red if any checked line through them is red, green if they touch only green checked lines, grey otherwise.
3. **Level guides:** dashed [5, 6] horizontal lines at 60% opacity, width max(1.5, W×0.003), from 8% to 92% of the width, at the shoulder-midpoint and hip-midpoint heights, coloured like the shoulder line (11–12) and hip line (23–24) — grey when not checked. (The v1.5 centre guide line is removed; the outline replaces it.)
4. **Spine indicator:** a dashed [4, 5] white line (40%) straight up from mid-hips to shoulder height; a solid line mid-hips → mid-shoulders, width max(4, W×0.008), coloured by the smoothed "spine" state if the pose checks the torso, otherwise green when torso tilt is under the pose's tolerance (Mountain 10, Tree 14, Warrior II 18, others 12); yellow `#ffd166` dots at both ends, radius max(6, W×0.012); a bold readout in the same colour at the line's midpoint shifted by W×0.06 (size max(14, W×0.022)): "N°" torso tilt; Half Forward Fold and Standing Forward Fold "hip N°"; Warrior III "N° from level" (90 − tilt).
5. (No canvas countdown — the hold countdown is shown on the current row of the pose queue, §4.)

**Mirrored text:** all canvas text is drawn with a horizontal counter-flip (translate to the point, scale(−1, 1)), centred on both axes.

## 11a. Outline placement, fit, facing and on-stage prompts

All measurements here use canvas pixels (so they are true proportions). "Screen x" of a landmark is (1 − x).

**Scale** s (pixels per body unit): the smallest of 0.92·W / box width and 0.86·H / box height over a set of poses — **every pose in the sequence** while a sequence runs (so the user can stay in one spot for the whole sequence), otherwise just the current pose.

**Position:** the outline's bounding box is centred horizontally on the screen. Vertically its floor (y = 0) is at **floorY**: default 0.97·H. Each frame with a body, take the **lowest visible key point** — the largest y among landmarks 11–16 and 23–28 with visibility > 0.5. If there is one and it is above the bottom 0.5% of the frame, the target is (that point's y·H + the outline's lowest key point height · s), where the outline's lowest key point height is the smallest y among its shoulders, elbows, wrists, hips, knees and ankles (e.g. 0.04 for standing poses = the ankles; 0.06 for Staff; 0.028 for Downward Dog = the wrists). Smoothed `old × 0.9 + new × 0.1`. So the outline stands on the same floor as the user in standing, kneeling, floor and seated poses. floorY is then clamped so the feet stay on screen (≤ H + box bottom·s − 2) and, taking priority, the head stays on screen (≥ box top·s + 0.01·H). floorY is cleared on Stop and when a sequence starts.

**Flip (which way the outline faces):** each frame with a body, a desired flip is computed in screen terms: Tree — the lifted (smaller knee angle) knee is left of mid-hips; Warrior II — the bent knee is left of mid-hips; Warrior I — the front (more bent) leg's ankle is left of mid-hips; Warrior III, Half Forward Fold, Standing Forward Fold and Plank — the nose is left of mid-hips; Extended Side Angle and Reverse Warrior — the bent (smaller knee angle) knee is left of mid-hips; Low Lunge — the front leg's ankle is left of mid-hips; Triangle — the lower wrist (larger y) is left of mid-hips; Standing Side Bend — the shoulder midpoint is left of mid-hips; Chair — the knee midpoint is left of mid-hips; Downward Dog — the wrist midpoint is left of mid-hips; Staff and Boat — the ankle midpoint is left of mid-hips; Mountain, Upward Salute, Star and Goddess — never. The outline flips (x → −x) only after **13 consecutive frames** disagree with its current direction. Reset to unflipped on pose change.

**Fit:** for each of 12 landmarks (11–16, 23–28) take the distance to the outline: the smallest of (distance to each bone's centre line − half its thickness) and (distance to the head centre − head radius); it is inside if ≤ 0.045·s. The nose counts too (against the head circle only). fit = inside ÷ 13, smoothed `old × 0.8 + new × 0.2` (reset to 0 on pose change).

**Size and side-to-side position:** user torso = distance mid-hips → mid-shoulders; outline torso likewise. r = user ÷ outline; dx = (user mid-hip screen x − outline mid-hip screen x) ÷ outline torso (positive = user is to the right on screen). Both smoothed `old × 0.85 + new × 0.15`; reset on pose change.

**Facing check:** ratio = shoulder width ÷ torso length, smoothed `old × 0.85 + new × 0.15`. Wrong way round if the pose wants **side** and ratio > 0.50, or wants **front** and ratio < 0.32. It must stay wrong for **0.9 s** before "facing wrong" is set; it clears as soon as the ratio is fine, or when no body is detected. Reset on pose change.

**On-stage overlays** (HTML, not mirrored; white backgrounds; updated only when their content changes; hidden when the camera stops):
- **Facing chip** (top-left 14px; 32px high pill, 12.5px/700; max width = stage width − 160px, ellipsis): person icon + "Face the camera" or two-arrows icon + "Stand side-on"; when facing is wrong it turns amber and reads "Turn to face the camera" / "Turn side-on to the camera". Hidden while the completion card shows.
- **Pose queue** (top-right 14px, width 124px, radius 18, padding 6): a **"Now"** row (sage-soft background, 26px outline icon, bold "Now", and the live hold countdown in Fraunces 22px sage) and, during a sequence, the following steps as rows labelled "Next" then "Then" with their seconds (20px icons, 12px/600 muted) — up to 3 after the current one (2 on ≤ 480px) — then "+K more" if there are more. Countdown: before the hold starts the target as "Ns" (13px muted sans); while active the remaining whole seconds; paused at 50% opacity; done = green "✓". On ≤ 480px the queue is 108px wide at 10px from the corner and the countdown is 19px.
- **Facing card** (centred, white, 2px sage border, radius 22, soft shadow): shown for 4 s after each pose announcement, or while facing is wrong (amber border, amber icon and title). It shows the pose's outline icon (72px, flipped like the outline), a Fraunces 22px title ("Stand facing the camera" / "Stand side-on to the camera", or "Turn to face the camera" / "Turn side-on to the camera") and a 13.5px muted line ("{short name} — step into the outline." or "The outline shows the pose from this angle.").
- **Completion card** (same style): see §9a.
- **Placement hint** (bottom-centre 14px pill, 13px/700; hidden while the facing or completion card shows). First match wins: model not loaded "Loading the pose tracker…"; no body "Step into view — your whole body should be visible"; r > 1.18 "Step back a little — you're bigger than the outline"; r < 0.82 "Come a little closer — you're smaller than the outline"; dx > 0.6 "Move a little to your left"; dx < −0.6 "Move a little to your right"; fit ≥ 0.8 a check icon + "You're inside the outline" in `--good-text`; otherwise "Fit your body inside the outline".

The outline and placement are guidance only: the score and hold timer still come from the cues (§8, §9).

## 12. Start-up, camera and errors

- **On load:** build the step editor, Home sequence card and pose tiles, select Mountain (single-pose), set the greeting, show the Home screen. If the page is opened as `file:`, show the error "⚠ This page is opened as a local file, and Chrome blocks the camera on file:// pages (that's why no permission prompt appears). Run it from a local server instead — see Help & tips in the menu." If `navigator.mediaDevices.getUserMedia` is missing, show "This browser doesn't expose camera access here. Use Chrome or Safari, and open the page via http(s) (see Help & tips in the menu)." Status messages appear on both screens' status lines, and on the stage placeholder while the camera is off.
- **Start** (camera first, then the model; ignored if already running or starting):
  1. Show the stage placeholder "Starting the camera…" and status "Requesting camera…". Request `video: {width ideal 960, height ideal 720, facingMode "user"}`, no audio. Play the video, hide the placeholder, set running, size the stage to the camera's aspect ratio, begin the frame loop, and announce the current pose (§10).
  2. Camera errors (stop any sequence; the message shows on the stage in light orange):
     - NotAllowed/Permission on `file:` → "Camera blocked because this page is opened as a local file. Chrome won't show a prompt on file:// pages. Open it via http://localhost instead — see Help & tips in the menu."
     - NotAllowed/Permission otherwise → "Camera permission was denied and no prompt appeared. Two things to check: (1) click the camera icon in the browser's address bar and set it to Allow; (2) on Mac, open System Settings ▸ Privacy & Security ▸ Camera and make sure your browser is switched on. Then try again."
     - NotFound → "No camera was found. Check that a webcam is connected and not in use by another app."
     - NotReadable → "The camera is busy — another app (Zoom, FaceTime, Photo Booth…) may be using it. Close that app and try again."
     - anything else → "Couldn't open the camera: {message}"
  3. If the model isn't loaded yet, show "Downloading pose model (first time only)…" and load it (§3). If offline → "You appear to be offline. The pose model has to download once from the internet — reconnect and try again. (Your camera is working.)" If a download fails → "Camera works, but the pose model couldn't download — a network or firewall is blocking it. Try a different network, or see Help & tips in the menu about running a tiny local server." Any other failure → "Camera works, but the pose model failed to load: {message}". On success the status is cleared. The camera keeps running even if the model fails.
- **Frame loop** (`requestAnimationFrame`): when the video has data, match the canvas size to the video (and re-size the stage). If the model is loaded: only when `video.currentTime` has changed, run `detectForVideo(video, performance.now())`; with a body, update flip, facing, placement and floor (§11a), the fit, then evaluate (§8); then draw (§11) and update the overlays and coach card. If no body: status "No body detected — make sure you're fully in frame." (set once; cleared when a body returns) and the checks return to idle. If the model isn't loaded yet: draw the outline every frame. While the completion card shows, only clear the canvas.
- **Stop** (End / Finish): stop any running sequence, cancel speech, stop the camera tracks, show the placeholder "Camera off.", clear the canvas, line smoothing and outline floor, hide the stage overlays, reset the hold timer and the checks/score, and clear the status.

## 13. Acceptance checks (to confirm a rebuild matches)

1. Standing straight in Mountain pose facing the camera → every line is green, the score climbs to 100, the timer counts 10 → ✓, and the spine readout is near 0° (not ~180°).
2. Raising one shoulder in Mountain → only the shoulder line and its guide turn red; the score drops to 75.
3. In Mountain, the arms stay grey the whole time.
4. Half Forward Fold side-on with a flat back and a hip angle under 90° → green, with "hip N°" shown. Rounding the back → the torso turns red and "Back straight" fails. Standing up → "Hinged" fails.
5. Warrior III with the lifted leg dropped → only that leg is red; the message is "Lift the back leg up to hip height."
6. Stepping out of a pose for about 2 s mid-hold → the timer shows "Paused…", then resumes from where it was. A 0.3 s tracking blip doesn't pause it.
7. Changing a pose's timer to 20 s, reloading the page → it's still 20 s for that pose and 10 s for the others.
8. With voice on, a 15 s hold at full alignment → "Lovely. Hold for 15 seconds." (or another opener), "Five more seconds", then "Well done. Release the pose." A 40 s hold adds "30 / 20 / 10 seconds left." Two prompts never play over each other, and no correction is spoken while the timer is running.
21. Out of position (not holding) → a correction is spoken only after the problem has lasted 1.5 s, at least 5 s after the previous prompt finished; the same correction doesn't repeat within 12 s. A 0.3 s tracking flicker mid-hold says nothing. Dropping out of the pose for 3 s mid-hold and coming back → "Good. N seconds to go."
9. First visit: the Home screen shows "Your practice" and the default sequence (Mountain 15 s, Tree 20 s, Warrior II 20 s, a "Then turn side-on" divider, Warrior I 20 s, Warrior III 15 s, Half Forward Fold 20 s; "6 poses · 1 min 50 s of holding · one turn"). The menu button or "Edit" opens the side menu; edit the sequence there (seconds, reorder, delete, add), reload → the edits are still there and the Home card matches.
10. Typing 999 seconds → it becomes 300; typing 2 → it becomes 5.
11. "Begin practice" → the Session screen opens with "Pose 1 of N" and segments, editing is locked, the − / + control is hidden and the timer uses that step's seconds. When the hold completes → 3 s later the app moves to step 2. "Skip pose" works. After the last step → the "Practice complete" card; "Finish" returns Home and editing unlocks.
12. Tapping a pose tile on Home → Session screen with "Practising one pose", the − / + control visible and the pose's own hold time; "End" returns Home.
13. Starting any pose → the outline of that pose appears on the video (before the model has even loaded), the facing card shows "Stand facing the camera" or "Stand side-on to the camera" for 4 s, and (voice on) the app says the pose name, which way to face, the description and "Step into the outline."
14. Standing too close → "Step back a little — you're bigger than the outline"; too far → "Come a little closer…"; off to one side → "Move a little to your left/right". Once inside, the outline turns green and the hint reads "✓ You're inside the outline".
15. In Warrior I/III or the Fold while facing the camera → after about 1 s an orange "Turn side-on to the camera" card appears and the voice says it; turning side-on clears it. In Mountain/Tree/Warrior II while side-on → "Turn to face the camera".
16. Doing Warrior I (or III, or the Fold) facing screen-left → within about half a second the outline flips to face left. Tree with the other leg lifted / Warrior II with the other knee bent → the outline flips too.
17. Layouts: phone = one column (camera, then coaching); iPad portrait = camera on top, coaching in two columns below; iPad landscape and laptop = camera left, coaching right. Home is one column on phones and two on iPad/laptop. No horizontal scrolling at 390px wide.
18. During a sequence the queue in the top-right of the camera view shows "Now" with the countdown (target "20s" → counting down → "✓") and the next poses ("Next", "Then") with their times. In single-pose practice it shows only "Now".
20. The coach card always shows one thing to do: "Step into view", the facing prompt, the first failing cue's message, or "Lovely — hold it there" when everything is green.
19. The outline is clearly visible on both dark and bright backgrounds (thick light edge with a dark shadow).
22. Home shows 20 pose tiles in two groups — "Facing the camera" (10) and "Side-on" (10) — and the step editor's pose menu lists all 20 in the same order.
23. Each pose's own outline, turned into landmarks, passes all of that pose's checks, both unflipped and flipped, and the flip rule picks the matching direction (checked in code for all 20).
24. Seated and floor poses (Staff, Boat, Plank, Downward Dog, Low Lunge) → the outline sits on the same floor as the user rather than floating (its lowest point lines up with the user's lowest point).

## 14. Known limitations

- Thresholds are hand-picked and not yet tuned with real users; the side-on poses and the 14 poses added in v1.10 need real-camera testing.
- Floor and seated poses (Plank, Downward Dog, Staff, Boat, Low Lunge) need the camera low and far enough back to see the whole body lengthways; on a phone in portrait the user has to be further away.
- Some poses share checks that a 2D view can't separate: e.g. a full forward fold also passes the Half Forward Fold checks, and Mountain doesn't check the arms or stance (so Upward Salute and Star also pass it). The facing check separates Staff (side-on) from Mountain (front).
- The app is 2D only: poses judged side-on can't check things that are only visible from the front (e.g. hips square in Warrior I).
- Single person only; the first detected body is used.
- One sequence per device; sequences aren't synced between devices (e.g. phone and laptop).
- The design's fonts load from Google Fonts; offline, the app falls back to Georgia and the system sans font.
- The outline uses fixed average body proportions; people with different proportions may not fit it exactly — it's a guide, while the score and timer come from the angle checks.
- The facing check (shoulder width vs torso length) can be fooled by loose clothing or partial views; thresholds need real-camera tuning.
- Voice prompts during a hold are time-only; corrections wait until the hold pauses. Alignment-first coaching for long holds / experienced practitioners is planned but not built.

## 15. Version history

| Ver | Date | Change |
|---|---|---|
| 1.0 | 2026-08-21 | Prototype: Mountain, Tree, Warrior II; cue checklist; score ring; robust camera/model start-up |
| 1.1 | 2026-08-21 | Voice coaching |
| 1.2 | 2026-09-27 | Hosted on GitHub Pages as SamajYoga; aspect-ratio correction in angle maths; centre, level and spine guides with degree readout; slower, deeper voice (0.82/0.78) |
| 1.3 | 2026-09-28 | Colour-coded skeleton (red/green/grey per line) + legend; tilt helpers made direction-independent (fixed ~180° spine reading) |
| 1.4 | 2026-09-28 | Warrior I, Warrior III, Half Forward Fold; per-pose hold timer (default 10 s); guides follow each pose's checks; score can reach 100 |
| 1.5 | 2026-09-28 | My routine: saved, editable list of poses + hold times (default: every pose at 10 s), guided run with auto-advance, skip and stop; help text points to `index.html` |
| 1.6 | 2026-10-05 | Simpler main screen: sequence editor and help moved to a ☰ side menu, read-only sequence strip + "Start sequence" on the main screen, coaching panel under the camera on phones; new default sequence (front poses then side-on, 15–20 s holds, new storage key); see-through target-pose outline on the video that the user steps into (scaled per sequence, stands on the user's floor, flips to match facing, turns green when inside) with step back/closer/left/right hints; facing-the-camera vs side-on prompts (card + voice + wrong-way detection); voice on by default and saved; pose icons; stage matches camera aspect; countdown moved to top-right; centre guide removed |
| 1.7 | 2026-10-05 | Bolder outline (thicker edge, stronger fill, dark shadow, brighter colour); pose queue moved into the top-right corner of the camera view with the live hold countdown on the current pose (replaces the main-screen strip and the canvas countdown); compact queue on phones |
| 1.8 | 2026-10-05 | "Studio Calm" redesign (chosen from three mock-ups): light stone/white/sage look with Fraunces + Manrope; separate Home screen (greeting, sequence card with turn divider, Begin practice, pose tiles, tips) and Session screen (End, progress segments, camera card, coach card with the one thing to do now, hold bar, checks); layouts for phone, iPad portrait, iPad landscape and laptop; white on-video overlays; completion card; icon voice button; updated skeleton colours |
| 1.9 | 2026-11-14 | Calmer voice coach: one voice channel with priorities (corrections < facing < time < announcements) so prompts never overlap; corrections are periodic (must persist 1.5 s, 5 s quiet gap, 12 s before repeating) and silent while the hold timer runs; time read-outs take priority (start "Hold for N seconds", every 10 s on holds over 20 s, "Five more seconds", "N seconds to go" after a pause); separate praise prompt removed; help text updated |
| 1.10 | 2026-11-14 | 20 poses: added Upward Salute, Standing Side Bend, Five-Pointed Star, Goddess, Reverse Warrior, Extended Side Angle, Triangle (facing the camera) and Chair, Standing Forward Fold, Low Lunge, Downward-Facing Dog, Plank, Staff, Boat (side-on), each with checks, outline, flip rule and voice; outline floor now follows the user's lowest visible point (for seated/floor poses); Home pose tiles grouped by facing in a grid; default sequence unchanged |
