# Learning track: random-placement curriculum (Phase 5 idea)

## Why the task fits RL well
Moving one piece from a random square to another random square is an ideal first task: cheap to generate, endlessly varied, objectively checkable. Our Phase 1 perception already checks the result: source square empty, target occupied, nothing else changed. So we skip the hardest part of SERL, learning a reward classifier from images, and get a reliable reward almost for free.

## The arm resets itself
Real-world RL usually stalls on resets, because a human must restore the scene after every episode. Here the end of one episode is a valid start for the next: just sample a new target square. The arm practises unattended. A human is needed only when a piece falls over or leaves the board, and perception can detect that too and pause.

## Start from the scripted mover, not from scratch
A success-only reward is too sparse for RL from random motions. Instead, the Phase 2 scripted mover generates hundreds of automatic demonstrations. Train ACT or SmolVLA on those by imitation, then refine with RL (HIL-SERL style) for the hard cases: off-centre pieces, crowded squares, millimetre-precise placement. Shape the reward by distance from the square's centre.

## The curriculum
Stage 1: one piece, random source and target. Stage 2: two pieces, so the arm must avoid knocking a neighbour. Stage 3: N pieces arranged into a random target layout, where planning the order of moves matters. Advance a stage only after~90% success over 50 attempts. The target square must be an input to the policy: coordinates, a highlighted image, or a language instruction.

## Where it sits in our plan
This is Phase 5, deliberately off the exhibition's critical path. The scripted MVP ships first; learning runs in parallel on a second arm. Watch out for servo wear and overheating during long sessions. Log every episode as a LeRobot dataset: it doubles as the benchmark for the long-term video-learning idea.
