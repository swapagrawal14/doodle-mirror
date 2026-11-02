# Requirements Document

## Introduction

Doodle Mirror is a client-side web app in which Swapnil Agrawal's hand-drawn doodle character mirrors the person in front of the webcam. MediaPipe Tasks Vision tracks the face, hands and body on device. The body pose selects one of eight pre-drawn full-body poses; eye and mouth patches on top give live blinks and lip-sync. Halloween costume skins swap the whole pose pack. A spooky animated background, a 9:16 clip recorder with mic audio and an optional "haunt" voice complete the experience. It ships as a static site on Vercel.

## Glossary
- **Pose pack:** the 8 full-body drawings (idle, wave, point, thumbs_up, hands_up, open_arms, facepalm, dab) for one skin.
- **Skin:** a costume variant (default, vampire, ghost) with its own pose pack and face patches.
- **Head anchor:** `{x, y, size}` per pose in the manifest; where face patches go.
- **Viseme:** a mouth shape: A, E, O, M, smile, laugh.
- **Stage:** the 1080x1920 canvas the user sees and records.

## Requirements

### Requirement 1: Camera access and onboarding
**User Story:** As a visitor, I want a clear start screen, so that I understand what will happen before granting camera access.

#### Acceptance Criteria
1. WHEN the page loads THE SYSTEM SHALL show a start screen with the character in `idle`, a one-line privacy note ("Everything stays on your device") and a "Start mirror" button, without requesting camera permission.
2. WHEN the user presses "Start mirror" THE SYSTEM SHALL request webcam access (front camera, 640x480 ideal) and start tracking.
3. IF camera access is denied or unavailable THEN THE SYSTEM SHALL show a friendly message explaining how to allow the camera and keep the character animating in `idle` with auto-blink.
4. WHILE models are loading THE SYSTEM SHALL show a loading indicator and SHALL start mirroring within 5 seconds of models being cached on a mid-range laptop.

### Requirement 2: On-device tracking
**User Story:** As a privacy-conscious user, I want all tracking to run locally, so that my image never leaves my computer.

#### Acceptance Criteria
1. THE SYSTEM SHALL run FaceLandmarker (with blendshapes), HandLandmarker (up to 2 hands) and PoseLandmarker (1 pose) from `@mediapipe/tasks-vision` in the browser.
2. THE SYSTEM SHALL load model and WASM files only from the site's own origin.
3. THE SYSTEM SHALL NOT send video frames, audio, landmarks or recordings over the network.
4. WHEN the GPU delegate fails to initialise THE SYSTEM SHALL fall back to the CPU delegate.
5. WHILE tracking THE SYSTEM SHALL mirror left and right so the character behaves like a mirror image.

### Requirement 3: Pose classification
**User Story:** As a user, I want the character to copy my gesture, so that it feels like my doodle twin.

#### Acceptance Criteria
1. WHEN pose landmarks are available THE SYSTEM SHALL compute normalised features (relative to shoulder midpoint, scaled by shoulder width) and a score in 0..1 for each of the 8 poses.
2. WHEN one hand is raised above the shoulder with an open palm moving side to side THE SYSTEM SHALL score `wave` highest.
3. WHEN one arm is extended sideways or forward with the index finger extended and other fingers curled THE SYSTEM SHALL score `point` highest.
4. WHEN a fist with the thumb extended upward is detected THE SYSTEM SHALL score `thumbs_up` highest.
5. WHEN both wrists are above the head THE SYSTEM SHALL score `hands_up` highest.
6. WHEN both arms are spread wide at or below shoulder height THE SYSTEM SHALL score `open_arms` highest.
7. WHEN a hand covers the face region (wrist or palm within the face box) THE SYSTEM SHALL score `facepalm` highest.
8. WHEN one arm is raised diagonally and the other forearm is bent across the face THE SYSTEM SHALL score `dab` highest.
9. WHEN no other pose scores above the idle threshold THE SYSTEM SHALL choose `idle`.
10. THE SYSTEM SHALL use hand landmarks to disambiguate `point`, `thumbs_up` and `wave` when available, and fall back to pose landmarks alone otherwise.

### Requirement 4: Stable pose switching
**User Story:** As a viewer, I want the character to stay calm, so that tracking noise never makes it flicker.

#### Acceptance Criteria
1. THE SYSTEM SHALL switch to a candidate pose only after it beats the current pose's score by a configurable margin for a configurable number of consecutive frames (defaults 0.15 and 6).
2. AFTER a switch THE SYSTEM SHALL hold the new pose for at least a configurable minimum number of frames (default 9).
3. WHEN no person is detected for more than 0.5 s THE SYSTEM SHALL return to `idle`.
4. WHEN a held gesture is steady THE SYSTEM SHALL display the matching pose within 400 ms.

### Requirement 5: Pose-first rendering
**User Story:** As Swapnil, I want the character to look hand-animated, so that it matches my doodle style and never looks like a puppet.

#### Acceptance Criteria
1. THE SYSTEM SHALL always render a complete pre-drawn pose image and SHALL NOT rotate, stretch or reposition individual limbs.
2. WHEN the pose changes THE SYSTEM SHALL perform the configured transition (default: anticipation squash, cut on the beat, eased pose warp; optional crossfade or smear) within 250 ms.
3. WHILE a pose is held THE SYSTEM SHALL apply only whole-drawing secondary motion (breathing, gentle sway).
4. THE SYSTEM SHALL anchor every pose at a shared floor line and scale defined in the manifest.

### Requirement 6: Eyes
**User Story:** As a user, I want the character to blink when I blink, so that it feels alive.

#### Acceptance Criteria
1. WHEN face landmarks are available THE SYSTEM SHALL compute a smoothed eye-aspect ratio and show the `open`, `half` or `closed` eye patch using configurable thresholds.
2. IF face landmarks are missing THEN THE SYSTEM SHALL use eye-blink blendshapes, or an automatic blink every 3–6 s when no face is tracked.
3. THE SYSTEM SHALL place eye patches at the pose's head anchor so that the drawn eyes are fully covered.

### Requirement 7: Mouth and lip-sync
**User Story:** As a user, I want the character's mouth to move with mine, so that clips look like it is talking.

#### Acceptance Criteria
1. WHEN face blendshapes are available THE SYSTEM SHALL map them to one of A, E, O, M, smile or laugh and show the matching mouth patch.
2. THE SYSTEM SHALL hold each viseme at least 2 frames to avoid chatter.
3. WHILE the mic is on and the face is not tracked THE SYSTEM SHALL drive the mouth from mic loudness.
4. WHERE a pose's manifest face mode is `eyes-only` or `none` THE SYSTEM SHALL skip the patches that mode excludes.

### Requirement 8: Costume skins
**User Story:** As a Kiroween visitor, I want to change costumes, so that my doodle twin can be a vampire or a ghost.

#### Acceptance Criteria
1. THE SYSTEM SHALL offer the skins listed in the manifest (default, vampire, ghost) in the control bar and via keys 1–3.
2. WHEN the user selects a skin THE SYSTEM SHALL swap the entire pose pack and face patches, keeping the current pose, with a short "poof" fog burst.
3. WHILE a skin's images are still loading THE SYSTEM SHALL keep showing the previous skin.
4. IF a skin's assets fail to load THEN THE SYSTEM SHALL fall back to the default skin and log a console warning.

### Requirement 9: Spooky background
**User Story:** As a viewer, I want a Halloween atmosphere, so that the clip feels festive.

#### Acceptance Criteria
1. THE SYSTEM SHALL draw the background plate from the manifest and animate on top of it: drifting fog layers, flying bats, candle flicker and a glowing moon.
2. THE SYSTEM SHALL keep the background animation deterministic per seed and within the frame budget (≤ 4 ms per frame on a mid-range laptop).
3. WHERE the user prefers reduced motion THE SYSTEM SHALL slow and reduce background animation.

### Requirement 10: Clip recording
**User Story:** As a creator, I want to record a vertical clip with my voice, so that I can post it.

#### Acceptance Criteria
1. WHEN the user presses Record THE SYSTEM SHALL request the microphone if not yet granted and record the stage at 1080x1920, 30 fps, with mic audio.
2. WHILE recording THE SYSTEM SHALL show a countdown timer and stop automatically at 30 s.
3. WHEN recording stops THE SYSTEM SHALL offer the clip for download as `doodle-mirror-<timestamp>.webm` (or `.mp4` where WebM is unsupported).
4. IF microphone access is denied THEN THE SYSTEM SHALL record video without audio and say so.
5. THE SYSTEM SHALL NOT include the raw webcam image in the recording.

### Requirement 11: Haunt voice
**User Story:** As a user, I want a spooky voice effect, so that my clip sounds haunted.

#### Acceptance Criteria
1. WHERE haunt voice is enabled THE SYSTEM SHALL process the mic through a WebAudio chain (pitch-down effect, low-pass, echo) and use the processed audio in the recording.
2. WHILE haunt voice is enabled and headphones are not detected THE SYSTEM SHALL keep live monitoring off by default to avoid feedback.

### Requirement 12: Asset manifest
**User Story:** As a developer, I want all art described in one manifest, so that new skins or poses need no code changes.

#### Acceptance Criteria
1. THE SYSTEM SHALL load `assets/manifest.json` and validate that every skin has all 8 poses, every pose has a head anchor with positive size inside the image bounds, and every skin has all 3 eye and 6 mouth patches.
2. IF validation fails THEN THE SYSTEM SHALL report the exact missing or invalid entries in the console and on a debug overlay.

### Requirement 13: Controls, accessibility and debug
**User Story:** As a user, I want simple controls, so that I can play without instructions.

#### Acceptance Criteria
1. THE SYSTEM SHALL provide a control bar with skin picker, Record, Haunt toggle and a "show camera" picture-in-picture toggle (off by default).
2. THE SYSTEM SHALL make all controls keyboard accessible with visible focus and ARIA labels.
3. WHEN the user presses `D` THE SYSTEM SHALL toggle a debug overlay showing landmarks, pose scores, current pose, EAR and viseme.

### Requirement 14: Deployment
**User Story:** As Swapnil, I want a one-click deploy, so that judges can try it from a link.

#### Acceptance Criteria
1. THE SYSTEM SHALL build to a static `dist/` folder with `npm run build` and deploy to Vercel with no server functions.
2. THE SYSTEM SHALL work over HTTPS on the latest Chrome, Edge, Firefox and Safari, including mobile portrait.
