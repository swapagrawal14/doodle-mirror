---
inclusion: always
---

# Product: Doodle Mirror

Doodle Mirror is a browser toy for Kiroween 2026 (Costume Contest category). Stand in front of your webcam and a hand-drawn doodle character, Swapnil's own cartoon self, mirrors you: your pose, your blinks and your mouth as you talk. Pick a Halloween costume (default, vampire, ghost) and the whole character changes outfit. Record a 9:16 clip with your voice and post it.

## Who it is for
- Kiroween judges and visitors who want a delightful 30-second "wow" in the browser.
- Creators who want a short vertical clip of a talking doodle without any editing software.

## What must feel great
1. **It reads as hand-drawn animation, not a puppet.** The character always shows a complete pre-drawn pose. It never bends limbs, never shows joints, never stretches arms to follow a wrist.
2. **It responds right away.** A held gesture switches the pose within about 0.4 s. Blinks and mouth shapes follow live.
3. **It is calm.** Noisy tracking must never make the character flicker between poses.
4. **It is private.** Video and audio never leave the device. No uploads, no analytics, no accounts.
5. **It is spooky but cute.** Fog, bats, candle flicker, a moon. Hot pink + sunny yellow brand palette stays visible through the Halloween palette.

## Core features (v1)
- Live webcam mirror with face, hand and pose tracking (MediaPipe Tasks Vision, on device).
- Pose classifier choosing among 8 drawn poses: idle, wave, point, thumbs_up, hands_up, open_arms, facepalm, dab.
- Face patches on top: eyes open/half/closed, mouths A/E/O/M/smile/laugh.
- Costume skins: default, vampire, ghost (each a full pose pack + face patches).
- Animated spooky background.
- Record button: 9:16 WebM clip of the stage with mic audio, downloadable.
- Optional "haunt" voice: WebAudio effect on the mic (pitch-down + echo) that also goes into the recording.

## Out of scope (v1)
- Server code of any kind, user accounts, cloud storage, sharing to social networks from inside the app.
- Multiple people at once (track the most prominent person only).
- Free-form limb rigging or IK.

## Tone of UI copy
Short, playful, a little spooky. Example: "Strike a pose, mortal." Never scold the user. Camera-denied state explains how to allow it in one sentence.
