---
inclusion: fileMatch
fileMatchPattern: "src/{render,pose,face}/**/*.ts"
---

# Animation rules (pose first, motion second)

Source: Swapnil's Rigpop Motion rules, https://github.com/swapagrawal14/rigpop-motion/blob/main/AGENTS.md

## The character is always a complete drawing
- At any moment the stage shows one full pre-drawn pose PNG from the active skin (plus, during a transition, the outgoing pose for at most the transition window).
- **Never** rotate, stretch, skew or IK individual limbs. **No visible joints**, no shoulder balls, no separately drawn arms. The tracked skeleton only *chooses* which drawing to show.
- If a gesture is not in the pose pack, map it to the nearest drawn pose (usually `idle`). Do not invent motion for it.

## Choosing the pose
- Landmarks → normalised features (divide by shoulder width, origin at the shoulder midpoint, mirror x for selfie view).
- Classifier returns a score 0..1 for every pose. Pick the best.
- **Hysteresis:** only switch when the candidate beats the current pose's score by `SWITCH_MARGIN` (default 0.15) for `CONFIRM_FRAMES` consecutive frames (default 6 at 30 fps ≈ 0.2 s).
- **Minimum hold:** after a switch, stay at least `MIN_HOLD_FRAMES` (default 9 ≈ 0.3 s), except for switching back to `idle` when tracking is lost for >0.5 s.
- When no person is detected, ease to `idle` and fade the face patches to `eyes_open` + `smile`.

## Moving between poses
- The swap lands as a **cut on the beat**, sold by a short **pose warp**: the incoming drawing starts at scale (1.04, 0.96) anchored at the feet and eases to (1, 1) over ~160 ms with `easeOutBack`; an anticipation squash of (0.97, 1.03) for ~60 ms precedes it.
- **Default transition mode is `cut`.** Swapnil's Rigpop rule found that cross-dissolving two painted poses reads as a ghosted double character. A `crossfade` mode (≤120 ms, `easeInOutCubic`) and a `smear` mode (outgoing pose motion-blurred by drawing it 3 times at decreasing alpha along the gesture direction for ≤80 ms) exist as options in `config.ts` and the debug panel, but stay off by default unless Swapnil chooses them after seeing them.
- Secondary motion only as small offsets on the whole current drawing: breathing (scale y ±0.8%, 3.2 s sine), sway (x ±4 px, 4.5 s), a tiny bob after a switch. Never per-limb.
- Everything eased. No linear moves.

## Face on top of the pose
- Every pose in the manifest has a `head` anchor `{x, y, size}` in pose-image pixels: x,y is the face centre (between the eyes and mouth), size is the face width.
- Eye and mouth patches are drawn at that anchor, scaled by `head.size / skin.face.refSize`, using the patch rects stored in the manifest (relative to the face centre).
- The mouth patch must cover the drawn mouth completely so only one mouth is ever visible. Same for eyes.
- Poses whose face is covered (`facepalm`) have `"face": "eyes-only"` or `"none"` in the manifest; honour it.
- Eyes: eye-aspect ratio (EAR) from FaceLandmarker eye landmarks, averaged over both eyes and smoothed (EMA α=0.5). EAR > 0.24 → open, 0.17–0.24 → half, < 0.17 → closed (tunable). Fall back to the `eyeBlinkLeft/Right` blendshapes if landmarks are missing. Add an idle auto-blink every 3–6 s when the face is not tracked.
- Mouth (visemes) from blendshapes, priority order: `jawOpen > 0.55` and smile → `laugh`; `mouthFunnel`/`mouthPucker > 0.35` → `O`; `jawOpen > 0.35` → `A`; `mouthStretch`/wide + small jaw → `E`; smile > 0.45 and jaw < 0.15 → `smile`; otherwise `M`. Hold each viseme at least 2 frames to avoid chatter. When the face is lost but mic is on, use mic RMS to alternate `M`/`A`/`E`.
