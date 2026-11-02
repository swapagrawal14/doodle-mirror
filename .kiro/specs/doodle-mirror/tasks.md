# Implementation Plan

- [ ] 1. Scaffold the Vite + TypeScript project
  - `npm create vite@latest . -- --template vanilla-ts` (keep existing `assets/`, `.kiro/`, README, LICENSE), strict `tsconfig.json`, folders from `structure.md`
  - Add ESLint flat config (typescript-eslint) and Vitest; scripts `dev`, `build`, `preview`, `lint`, `test`, `fetch-models`
  - Serve `assets/` as static (Vite `publicDir` stays `public/`; copy or alias `assets/` so `/assets/manifest.json` resolves in dev and build)
  - _Requirements: 14.1, 12.1_

- [ ] 2. Add `src/config.ts` with every tunable number
  - Hysteresis margin/frames, idle threshold, EAR thresholds, viseme thresholds, transition mode (default `cut`) and timings, breathing/sway amplitudes
  - _Requirements: 4.1, 4.2, 5.2, 6.1, 7.2_

- [ ] 3. Manifest types, loader and validator
  - Implement `src/assets/manifest.ts` per design; validator returns precise error strings
  - Unit tests: the committed `assets/manifest.json` passes; removing a pose, a mouth, or giving a head anchor outside the image each fails with a clear message
  - _Requirements: 12.1, 12.2_

- [ ] 4. Easing and math utilities with tests
  - `easeOutBack`, `easeInOutCubic`, `smoothstep`, `lerp`, EMA, seeded PRNG
  - _Requirements: 5.2, 9.2_

- [ ] 5. Model fetch script and tracker
  - `scripts/fetch-models.mjs` downloads face_landmarker.task, hand_landmarker.task, pose_landmarker_lite.task and the tasks-vision WASM into `public/` (gitignored)
  - `src/tracking/tracker.ts`: create the three landmarkers (GPU → CPU fallback), `VIDEO` mode, hands every 2nd frame, mirrored x, reused result object
  - _Requirements: 2.1, 2.2, 2.4, 2.5_

- [ ] 6. Start screen and camera permission flow
  - Start screen with idle character, privacy line and "Start mirror"; camera only on click; denied/unavailable message; loading indicator
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.3_

- [ ] 7. Feature extraction with tests
  - `src/tracking/features.ts` per design (normalised by shoulder width, wrist-speed ring buffer, finger extension from hand landmarks)
  - Add a debug "copy landmarks" button (wired in task 18) and seed `tests/fixtures/poses/` with synthetic fixtures for now
  - _Requirements: 3.1, 3.10_

- [ ] 8. Pose classifier with tests
  - One scoring function per pose using smoothstep ramps; idle fallback
  - Tests: each fixture scores its own pose highest
  - _Requirements: 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10_

- [ ] 9. Hysteresis state machine with tests
  - Switch margin + confirm frames, minimum hold, lost-person → idle after 0.5 s
  - Tests: jittery scores never flip; steady candidate switches after N frames; response within 400 ms at 30 fps
  - _Requirements: 4.1, 4.2, 4.3, 4.4_

- [ ] 10. Stage and asset store
  - `render/stage.ts`: 1080x1920 canvas, CSS fit, DPR aware; `AssetStore` decodes a skin's images to `ImageBitmap`s via the manifest
  - _Requirements: 5.4, 8.3, 12.1_

- [ ] 11. Character renderer: poses on one floor line
  - Draw the current pose with shared floor line and scale; secondary motion (breath, sway) on the whole drawing only
  - _Requirements: 5.1, 5.3, 5.4_

- [ ] 12. Pose transitions
  - `cut` default (anticipation squash → cut → eased warp), plus optional `crossfade` and `smear` modes selectable in config and debug panel
  - _Requirements: 5.2_

- [ ] 13. Eyes: EAR, smoothing, fallbacks, with tests
  - `face/eyes.ts`; draw eye patches at the head anchor; honour face mode
  - _Requirements: 6.1, 6.2, 6.3, 7.4_

- [ ] 14. Visemes and lip-sync, with tests
  - `face/visemes.ts` blendshape mapping, 2-frame hold, mic-RMS fallback; draw mouth patches at the head anchor
  - Check visually that the mouth patch fully covers the drawn mouth on every pose and skin (adjust manifest rects if not)
  - _Requirements: 7.1, 7.2, 7.3, 7.4_

- [ ] 15. Costume skins
  - Skin chips + keys 1–3; swap pose pack and face patches keeping the current pose; fog "poof"; keep old skin until new one is decoded; fallback to default on error
  - _Requirements: 8.1, 8.2, 8.3, 8.4_

- [ ] 16. Spooky background
  - Plate + parallax fog, bats, candle flicker, moon glow; seeded; reduced-motion support; measure frame cost
  - _Requirements: 9.1, 9.2, 9.3_

- [ ] 17. Mic, haunt voice and recorder
  - `audio/mic.ts`, `audio/haunt.ts` (delay-line pitch-down, low-pass, echo), `record/recorder.ts` (captureStream + audio, mime fallback, 30 s cap, countdown, download name)
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5, 11.1, 11.2_

- [ ] 18. Controls, accessibility and debug overlay
  - Control bar (skin, Record, Haunt, camera PiP), keyboard shortcuts, ARIA labels, focus styles; `D` debug overlay with landmarks, scores, pose, EAR, viseme, transition-mode switch and "copy landmarks"
  - _Requirements: 13.1, 13.2, 13.3_

- [ ] 19. Replace synthetic fixtures with recorded ones and tune
  - Record real landmark fixtures for all 8 gestures via the debug overlay, update tests, tune classifier weights and thresholds in `config.ts`
  - _Requirements: 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 4.4_

- [ ] 20. Privacy and performance pass
  - Confirm zero network requests besides own static files (DevTools Network); hold 30 fps; no per-frame allocations in the render loop
  - _Requirements: 2.2, 2.3, 9.2_

- [ ] 21. Deploy to Vercel
  - `vercel.json`, build runs `fetch-models` then `vite build`; test the HTTPS URL on desktop and mobile; add the live link to README
  - _Requirements: 14.1, 14.2_
