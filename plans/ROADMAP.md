# Chess-Arm Roadmap (condensed)

**Goal:** At a small electronic-art exhibition, a visitor plays a physical chess game against an SO-ARM101. A camera watches the board, an engine picks a reply, the arm moves the piece. It may be clumsy. It must be honest and it must finish games.

**Rule above all others:** minimal viable product. Each phase ends with something that runs end to end. No phase starts until the previous one has been demoed.

---

## Core design decisions (MVP)

| Problem | MVP choice | Why | Later / stretch |
|---|---|---|---|
| Knowing the human's move | Fixed overhead camera + **occupancy diff** of 64 squares, before vs after, checked against legal moves (`python-chess`) | No piece recognition needed: the legal-move list resolves almost every ambiguity | Piece classifier, video-transformer move recognition |
| Knowing when the human is done | **Physical button** (chess-clock style) | Removes the hardest perception problem (hands over the board, half-made moves) | Hand detection / motion-settle detection |
| Board geometry | 4 **ArUco markers** on board corners → homography → square grid | Robust to the camera being bumped; one-line OpenCV | — |
| Choosing a reply | **Stockfish** via `python-chess`, limited `UCI_Elo` | Solved problem, adjustable difficulty for visitors | Personality / move commentary |
| Moving a piece | **Scripted pick-and-place**: square → (x,y,z) → inverse kinematics → joint targets, sent through LeRobot's SO-101 driver | Deterministic, debuggable, no training data needed | **Learned policy** (ACT / SmolVLA) trained on teleop demos in LeRobot |
| Captures | Remove captured piece to a fixed "graveyard" tray first, then move | Simple two-step | — |
| Special moves | Castling = two moves; en passant = capture elsewhere; promotion = always queen, robot asks human to swap the piece (screen message) | Comedy is allowed | — |
| Errors | If perception is unsure or illegal: screen says "I didn't see that, please fix the board" and re-scans | Honest failure beats silent failure | — |
| UI | One laptop screen: camera view + digital board + status text | Exhibition visitors need to know what the robot "thinks" | — |

**Hardware risk to test in week 1:** SO-101 reach vs board size. A standard board (5–5.7 cm squares, ~45 cm wide) is likely too big. Plan for a **small board (~3.5–4 cm squares, 28–32 cm wide)** with tall, weighted, easy-to-grip pieces, arm mounted at the middle of one side. Measure before buying anything.

---

## Phases

### Phase 0 — Repo and LeRobot familiarisation (both people)
- Monorepo skeleton, `uv` Python env, LeRobot installed and pinned.
- Brother's arm: calibrate, teleoperate leader→follower, record a short dataset, push it to a private Hugging Face Hub repo, replay it.
- **Done when:** both of you can run `lerobot` teleop and replay from this repo's scripts.

### Phase 1 — "Brains in a box" (no arm needed)
- `engine/`: Stockfish wrapper, game state, move → list of physical actions (pick square, place square, graveyard).
- `perception/`: ArUco homography, square crop, occupancy diff, legal-move resolver. Built and tested on **recorded videos/photos** of a real board.
- `game/`: orchestrator loop with a **mock arm** (prints moves / shows them on screen; a human moves the robot's pieces).
- **Done when:** you can play a full game against the laptop on a physical board, with the human doing the "robot" moves by hand. This is already an exhibition fallback.

### Phase 2 — Arm moves one piece
- `arm/`: square→world coordinates calibration (touch 4 corner squares with the arm, fit a plane), IK via the SO-101 URDF, pick/place primitive with safe hover height, gripper open/close.
- **Done when:** command `e2→e4` and the arm does it 9 times out of 10.

### Phase 3 — Full loop (MVP)
- Plug real arm into `game/`. Button → perception → engine → arm → perception verifies the arm's own move.
- Handle captures, castling, promotion, recovery messages.
- **Done when:** 5 complete games in a row with no laptop intervention beyond "please fix the board".

### Phase 4 — Exhibition hardening
- Fixed lighting (LED panel), rigid camera mount, board glued/clamped to the base, emergency stop, auto-restart, idle "attract mode" motion, difficulty selector.
- **Done when:** runs for 3 hours unattended in a test run at home.

### Phase 5 — Stretch: learning (bridge to the long-term vision)
- Record teleop demos of pick-and-place in LeRobot, train ACT / SmolVLA, compare against the scripted mover.
- Experiment: annotate videos of *human* hands moving pieces, try to use them as training signal.
- **Done when:** a learned policy moves pieces at least as reliably as the script on a subset of squares.

---

## Monorepo layout (target)

```
chess-arm/
├── plans/              # this roadmap, the guide, decisions log
├── docs/               # hardware build notes, wiring, BOM, photos
├── hardware/           # STL/STEP files: board mount, camera arm, gripper fingers, piece adapters
├── src/chess_arm/
│   ├── perception/     # camera, ArUco, occupancy, move detection
│   ├── engine/         # Stockfish wrapper, game state, move→actions
│   ├── arm/            # LeRobot SO-101 driver wrapper, IK, pick/place, MockArm
│   ├── game/           # orchestrator state machine, button input
│   └── ui/             # exhibition screen
├── configs/            # YAML: camera, board geometry, arm calibration, per-machine overrides
├── scripts/            # CLI entry points: calibrate, teleop, record, play, replay
├── tests/              # unit tests + fixtures (small images only)
├── data/               # gitignored: recordings, datasets
└── models/             # gitignored: checkpoints (real copies live on HF Hub)
```

- **Interfaces first:** `Camera`, `Arm`, `Engine` are small Python protocols with a real and a mock implementation. Everyone can develop without hardware.
- **Storage:** code in git; small test images in git; datasets and model checkpoints in **private Hugging Face Hub repos** (LeRobot pushes there natively); calibration files per arm committed under `configs/arms/<name>.yaml`.
- **Pin versions:** LeRobot changes fast. Pin a commit/version in `pyproject.toml`; upgrade on purpose.

---

## Out of scope (for now)
LLM / voice interaction, piece recognition from scratch, multi-arm play, cloud inference, a mobile app, anything that does not make a visitor's game work better.
