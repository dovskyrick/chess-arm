# Chapter 1 — How a robot arm is controlled

Each section is about 60 words. This chapter covers the basics of any arm. Chapter 2 covers how LeRobot uses them.

## 1.1 Links, joints, degrees of freedom
An arm is rigid links connected by motorised joints. Each joint that can move independently is one degree of freedom (DOF). The SO-101 has five arm joints: `shoulder_pan`, `shoulder_lift`, `elbow_flex`, `wrist_flex`, `wrist_roll`, plus a `gripper`. The gripper is often counted separately, because it does not move the hand through space. It only opens and closes.

## 1.2 Joint space versus task space
There are two ways to describe where the arm is. **Joint space** is the list of joint angles, which is what the motors understand. **Task space** (Cartesian space) is where the gripper is in the world: x, y, z plus orientation. Humans and chess boards think in task space ("go to e4"); motors only think in joint space. Most of arm control is translating between the two.

## 1.3 Forward kinematics
Forward kinematics (FK) answers: given these joint angles, where is the gripper? You chain each link's length and each joint's rotation, multiplying one transform per joint. The answer is always unique and cheap to compute. FK needs a geometric model of the arm, usually a **URDF** file: an XML description of links, joints, lengths and limits.

## 1.4 Inverse kinematics
Inverse kinematics (IK) answers the opposite question: which joint angles put the gripper at this point? It is harder:
- There may be several solutions (elbow up or down), or none if the point is out of reach.
- Near some poses ("singularities") tiny moves need huge joint motions.

Solvers usually iterate numerically from the current pose. LeRobot wraps the `placo` solver for this.

## 1.5 Five DOF is not six
A free object in space has six DOF: three for position and three for orientation. With only five joints, the SO-101 cannot reach every position with every orientation. For chess that is fine: you need position plus "gripper pointing down", and the wrist roll is free. Still, ask IK only for what you need, or it will fail or twist strangely.

## 1.6 Servos and position control
The SO-101 uses Feetech STS3215 serial bus servos. Each has a motor, a gearbox, a magnetic encoder (4096 steps per turn, about 0.09°) and a small controller running a PID loop. You do not drive current yourself. You write a **goal position** register, and the servo pushes toward it. LeRobot configures the PID gains and sends goals over one shared serial bus.

## 1.7 The control loop
Every robot program is a loop: read sensors, decide, send commands, wait, repeat. LeRobot typically runs this at about 30 Hz. Each tick reads the joint positions (and camera frames), computes the next target, and writes it. The rate matters. Too slow looks jerky and reacts late. Too fast overloads the serial bus and cameras. Timing jitter is a common real-world bug.

## 1.8 Trajectories, not jumps
If you send a far-away goal, the servo rushes there at full speed, which is dangerous and inaccurate. Instead, break motion into a **trajectory**: many small intermediate targets, interpolated over time, ideally easing in and out. For pick-and-place: move above the target, descend slowly, grip, lift straight up, travel high, descend, release. LeRobot's `max_relative_target` setting caps how far a single step can jump.

## 1.9 Calibration
A raw encoder number means nothing until you know which number corresponds to which physical angle. Calibration records each joint's middle position and its range of motion, then maps raw steps to a normalised value. LeRobot uses -100 to 100 by default, or degrees if you choose. Leader and follower arms are calibrated separately so that the same physical pose gives the same number on both.

## 1.10 Accuracy versus repeatability
**Repeatability:** if you command the same pose twice, does the arm land in the same place? **Accuracy:** does it land where your model *says* the pose is? Cheap arms are usually more repeatable than accurate. Gear backlash, link flex, sagging under weight, and small URDF errors all add up. This is why "teach the arm exact poses" often beats "compute poses from geometry".

## 1.11 Cameras and coordinate frames
A camera sees pixels; the arm lives in millimetres. Linking them needs:
- **Intrinsics:** lens focal length and distortion, measured once with a checkerboard.
- **Extrinsics:** where the camera sits relative to the board or arm base.

With a fixed camera this setup is called "eye-to-hand". ArUco markers make extrinsics easy: OpenCV finds them and returns their pose relative to the camera.

## 1.12 Open loop versus closed loop
**Open loop:** compute the target, move, and trust it. **Closed loop:** measure the result and correct. The servos already close the loop on joint angles. Visual closed loop ("visual servoing") also closes it on the real gripper or piece position seen by the camera. This can beat the arm's mechanical error, but it costs speed and complexity. Also, the arm often blocks the camera's view.

## 1.13 Grasping
Picking a piece is a contact problem, not just geometry. The gripper must arrive centred, close with enough force to hold but not knock, and the piece must not slip or rotate. Grasps fail because of the shape of the pieces, slippery materials, and neighbouring pieces. Good fingers and good pieces (tall, with a neck, weighted base) solve more grasping problems than good software.

## 1.14 Safety
Even small arms pinch fingers and break things. Basics:
- Limit torque and speed: LeRobot already limits the gripper's torque.
- Keep motion slow near the board.
- Disable torque on exit, and make sure you can always cut power.
- Never trust a new policy or script at full speed. Run it first with your hand near the switch.

At an exhibition, assume children will reach in mid-move.
