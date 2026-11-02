---
inclusion: always
---

# Project structure

```
doodle-mirror/
├─ .kiro/                  steering, specs and hooks (committed; this is how the app was built)
├─ assets/                 hand-drawn art, read via manifest.json (Vite serves it as static)
│  ├─ manifest.json        poses, head anchors, skins, face patch rects
│  ├─ skins/<skin>/poses/<pose>.png        full-body poses, transparent, one shared canvas size
│  ├─ skins/<skin>/face/eyes_<state>.png   eye patches   (open | half | closed)
│  ├─ skins/<skin>/face/mouth_<shape>.png  mouth patches (A | E | O | M | smile | laugh)
│  └─ bg/                  background plate(s)
├─ public/models, public/wasm   MediaPipe files fetched by `npm run fetch-models` (gitignored)
├─ src/
│  ├─ main.ts              bootstraps: permissions screen, loads manifest, starts loop
│  ├─ assets/manifest.ts   types + loader + validator for manifest.json
│  ├─ tracking/            tracker.ts (MediaPipe setup + loop), features.ts (landmarks → features)
│  ├─ pose/                classifier.ts (features → scores), hysteresis.ts (stable pose state)
│  ├─ face/                eyes.ts (EAR → open/half/closed), visemes.ts (blendshapes/audio → mouth)
│  ├─ render/              stage.ts, character.ts (pose + transitions + face), background.ts
│  ├─ audio/               mic.ts, haunt.ts
│  ├─ record/              recorder.ts
│  ├─ ui/                  controls.ts, styles.css
│  └─ util/                easing.ts, math.ts
├─ tests/                  Vitest specs mirroring src/
├─ scripts/fetch-models.mjs
├─ index.html, vite.config.ts, tsconfig.json, eslint.config.js, vercel.json
```

## Conventions
- File names kebab-case or single words; exported types PascalCase; functions camelCase.
- One responsibility per module. DOM and canvas code only in `render/`, `ui/`, `record/`, `audio/`, `main.ts`.
- Pose ids and skin ids are string unions generated from the manifest types: `PoseId = 'idle' | 'wave' | 'point' | 'thumbs_up' | 'hands_up' | 'open_arms' | 'facepalm' | 'dab'`, `SkinId = 'default' | 'vampire' | 'ghost'`.
- Every tunable number (thresholds, hold frames, fade ms) lives in `src/config.ts`, with a comment saying what it does.
- Tests go in `tests/<same path>.test.ts`.
