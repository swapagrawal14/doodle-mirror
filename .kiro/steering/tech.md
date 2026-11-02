---
inclusion: always
---

# Tech stack and constraints

- **Build:** Vite + TypeScript (strict mode). No UI framework; plain DOM for the small control bar.
- **Rendering:** a single Canvas2D stage at 1080x1920 logical pixels (9:16), scaled to fit the window with CSS. `devicePixelRatio` aware.
- **Tracking:** `@mediapipe/tasks-vision` — `FaceLandmarker` (with blendshapes), `HandLandmarker` (max 2 hands), `PoseLandmarker` (lite model, 1 pose). Run in `VIDEO` running mode with `detectForVideo` on a `requestVideoFrameCallback` (fallback `requestAnimationFrame`) loop. GPU delegate first, CPU fallback.
- **Model files and WASM:** self-host under `public/models/` and `public/wasm/` (downloaded by an npm script `fetch-models`) so the deployed site makes no third-party requests at runtime. Do not commit the binaries; `.gitignore` excludes them and the build step fetches them.
- **Audio:** `getUserMedia({ audio: true })` only when recording or haunt voice is on. WebAudio graph: mic → (optional haunt chain: pitch-down via playbackRate-free granular or a simple `DelayNode` modulation + low-pass + feedback echo) → `MediaStreamAudioDestinationNode` for the recorder; analyser node feeds mouth fallback.
- **Recording:** `canvas.captureStream(30)` + audio track → `MediaRecorder` (prefer `video/webm;codecs=vp9,opus`, fall back to `vp8,opus`, then `video/mp4` on Safari). Max clip 30 s. Download via object URL.
- **Testing:** Vitest for pure logic (classifier, hysteresis, viseme mapping, easing, manifest validation). ESLint (typescript-eslint, flat config). `npm run lint` and `npm test` must pass after every task (a Kiro hook runs them).
- **Deploy:** static `dist/` on Vercel. Add `vercel.json` with headers `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` only if needed by the WASM build; otherwise keep defaults.

## Hard rules
- **Everything on device.** No fetch/XHR to any server other than the site's own static files. No analytics, no fonts from CDNs.
- **No secrets, no env vars needed.**
- **Performance budget:** stage at 30 fps on a mid-range laptop. Run the pose landmarker every frame, hand landmarker every 2nd frame, face landmarker every frame; reuse last results otherwise. Never allocate per frame inside the render loop (reuse arrays).
- Keep pure logic (classification, smoothing, viseme mapping, manifest parsing) in modules with no DOM access so it is unit-testable.
- Assets are read only through `assets/manifest.json`; never hard-code pose file names or pixel coordinates in TS.
- Browsers: latest Chrome, Edge, Firefox, Safari. Mobile Chrome/Safari should work in portrait.
