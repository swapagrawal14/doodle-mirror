# Doodle Mirror 🎃

**Your hand-drawn doodle twin, in costume, mirroring you live in the browser.**

Kiroween 2026 entry · Costume Contest · by Swapnil Agrawal

Stand in front of your webcam. A hand-drawn doodle character copies your pose, blinks when you blink and moves her mouth when you talk. Switch costumes between doodle, vampire and ghost, then record a 9:16 clip with your voice (or a haunted version of it) and post it.

> Live demo: _coming soon (Vercel)_

## Why it's different
Most webcam avatars are puppets: stretchy limbs and visible joints that wobble with every tracking glitch. Doodle Mirror animates **pose first**, the way hand-drawn cartoons are made. Every frame is a complete drawing. Your body only *chooses* which drawing to show (idle, wave, point, thumbs up, hands up, open arms, facepalm, dab), and a calm state machine switches between them with a little squash-and-settle. Eyes and mouth are drawn patches on top, so the doodle still blinks and lip-syncs live.

## Privacy
- **Everything runs on your device.** Face, hand and pose tracking use MediaPipe Tasks Vision in the browser (WebAssembly/WebGL).
- Video, audio and recordings never leave your computer. There is no server, no analytics and no account.
- Model files are served from this site itself; the app makes no third-party requests.
- Clips are saved only when you click download.

## How it works
1. **Track:** MediaPipe Face, Hand and Pose Landmarkers read the webcam frame.
2. **Understand:** landmarks become normalised features (arm angles, wrist height, finger states).
3. **Choose a pose:** a scorer rates all 8 drawn poses; hysteresis and a minimum hold stop flicker.
4. **Draw:** the chosen drawing lands on the beat with an eased squash-and-settle, plus breathing and sway on the whole drawing.
5. **Face:** eye-aspect ratio picks open/half/closed eyes; face blendshapes pick the mouth (A, E, O, M, smile, laugh).
6. **Costume:** each skin is a full pose pack plus face patches, described in `assets/manifest.json`.
7. **Scene and clip:** fog, bats, candle flicker and moon over a haunted background; MediaRecorder captures the 1080x1920 stage with mic audio, optionally through a WebAudio "haunt" voice.

Stack: Vite, TypeScript, Canvas2D, MediaPipe Tasks Vision, WebAudio, MediaRecorder. Static deploy on Vercel.

## Run locally
```bash
npm install
npm run fetch-models   # downloads MediaPipe models + WASM into public/
npm run dev
```

## Art
All character art is Swapnil's doodle character (bold black outlines, big round eyes, pink tulip, hot pink + sunny yellow). Poses and face patches live in `assets/skins/<skin>/`, all on one canvas size and floor line.

| Skin | Poses | Eye patches | Mouth patches |
|---|---|---|---|
| Doodle Swapnil (default) | 8 | open, half, closed | A, E, O, M, smile, laugh |
| Vampire | 8 | open, half, closed | A, E, O, M, smile, laugh (with fangs) |
| Ghost | 8 | open, half, closed | A, E, O, M, smile, laugh |

The art was prepared before the build (AI-assisted drawings of Swapnil's own doodle character from her brand references, keyed, normalised to one floor line and scale, face patches cut from one neutral head per skin). The app code is written in Kiro from the spec.

## Built with Kiro
<!-- TODO (Swapnil): fill this in while building, with screenshots. Judges score real Kiro usage. -->
- **Spec-driven development:** _TODO: how the requirements → design → tasks spec in `.kiro/specs/doodle-mirror/` guided the build; which tasks Kiro implemented; what you changed in the spec along the way._
- **Steering:** _TODO: how `product.md`, `tech.md`, `structure.md` and `animation.md` kept Kiro on the pose-first rule (no puppet limbs)._
- **Agent hooks:** _TODO: the manifest validator hook and the lint + test hook after every task; an example of a bug a hook caught._
- **Vibe coding moments:** _TODO: anything built conversationally (e.g. tuning the classifier with recorded landmark fixtures)._
- **What surprised you:** _TODO._

## License
MIT © 2026 Swapnil Agrawal. See [LICENSE](LICENSE).
