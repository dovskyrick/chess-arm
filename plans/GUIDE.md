# How to progress — a human guide

Short notes, about 70 words each. Read top to bottom once, then use as a checklist.

## 1. The one rule
Every week, something must run end to end, even if it is ugly. A laptop that tells you its move while you move the robot's piece by hand is a real product. A perfect inverse-kinematics module with no game around it is not. When you are unsure what to build next, pick the smallest thing that makes a full game slightly better.

## 2. Who does what
Your brother has the working arm, so he owns hardware: assembly, calibration, wiring, reach tests, gripper fingers. You own the software that does not need the arm: perception, engine, game loop, screen. You meet in the middle at the `Arm` interface. When your arm arrives, you both have hardware and can split by feature instead. Keep a short weekly call to demo progress.

## 3. Learn LeRobot first
Before writing chess code, spend a few evenings with LeRobot itself. Calibrate the arm, teleoperate with the leader arm, record a tiny dataset, push it to a private Hugging Face repo, replay it. This teaches you the driver, the config files, and the dataset format you will need later for learning. Write down every command that worked in `docs/lerobot-notes.md`. Future you will forget.

## 4. Measure the reach early
The biggest physical risk is that the arm cannot reach the far corner of the board. Before buying a board or pieces, tape a square on the table and move the arm by hand to each corner, at grabbing height. If it struggles, shrink the board. Small boards with tall, weighted, grippable pieces are the expected answer. Print the board and its mount so the geometry never changes.

## 5. Seeing the board
Do not try to recognise pieces. Put four ArUco markers on the board corners, warp the image to a flat grid, and only ask: which squares changed? Compare that change against the list of legal moves; usually only one fits. Collect photos of real moves under exhibition-like lighting and turn them into tests. A fixed LED lamp will save you more time than any clever algorithm.

## 6. The button
A human pressing a button when their move is done removes the hardest problem: hands over the board and half-made moves. It also feels natural, like a chess clock. Use a big arcade button through a microcontroller or a USB keypad. Later you may replace it with motion detection, but only after the whole game works. Until then, the button is a feature, not a shortcut.

## 7. The engine
Stockfish through `python-chess` does all the chess thinking. Limit its strength so visitors can win sometimes; a difficulty knob is a nice exhibition touch. Your own code here is small: keep the game state, turn the engine's move into physical steps (remove captured piece, pick, place, second move for castling), and answer "is this move legal?" for perception. Unit-test this heavily; it is cheap and catches bugs.

## 8. Moving pieces
Start scripted, not learned. Calibrate by touching a few known squares with the gripper, fit a board plane, and compute every square's position. Use inverse kinematics from the SO-101 model to reach a hover point, descend, grip, lift, travel, descend, release. Always lift high before travelling. Test one move a hundred times before trying a game. Reliability matters more than speed or elegance.

## 9. Mocks keep you moving
Every hardware piece gets a fake twin: a camera that plays back recorded images, an arm that prints its moves or draws them on screen. The game loop should not know the difference. This lets you work on a train, run tests in CI, and keep developing while the arm is broken or at your brother's house. It is the single best architecture decision for a two-person hardware project.

## 10. Storing data and models
Code lives in git. Large things never do. Datasets and model checkpoints go to private Hugging Face Hub repositories, which LeRobot already supports. Keep a small local `data/` and `models/` folder, both gitignored. Calibration files are small and personal to each arm, so commit them under `configs/arms/`. Write down which dataset trained which model, so experiments stay reproducible.

## 11. Failing honestly
The robot will misread the board and drop pieces. Plan for it. When perception is unsure or sees an illegal move, the screen says so and asks the human to fix the board. When a grip fails, the robot apologises and asks for help. At an art exhibition, a clumsy robot that admits its mistakes is charming. A robot that silently plays the wrong game is broken.

## 12. Getting exhibition-ready
Exhibitions are long, loud, and bright. Bolt everything to one base: board, arm, camera mast, lamp. Add an emergency stop button that cuts motor power. Make the software restart itself after a crash and resume or reset the game. Add an idle animation to draw people in. Then do a three-hour unattended rehearsal at home. Whatever breaks there would have broken in public.

## 13. After the MVP
Only once full games work reliably, start the learning track. Record teleoperated pick-and-place demos, train an ACT or SmolVLA policy in LeRobot, and compare it fairly against the scripted mover. Then try the long-term idea: videos of human hands moving pieces as training signal. The working chess robot gives you a real benchmark and a demo, which is exactly what a future startup pitch needs.
