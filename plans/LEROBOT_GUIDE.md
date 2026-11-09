# Learning LeRobot — a human guide (virtual first)

About 70 words per topic. Stages 1–7 need no physical arm. Command names change between LeRobot releases, so always check the official docs for the version you pinned.

## 1. What LeRobot actually is
LeRobot is a Python library from Hugging Face, built on PyTorch. It does four things: drives cheap arms like the SO-101, records demonstrations into a standard dataset format, trains policies (ACT, Diffusion, SmolVLA, pi0-style models) on those datasets, and runs them on a robot or in simulation. It is not ROS. There is no middleware to learn: just Python, config files, command-line scripts, and the Hugging Face Hub.

## 2. Set up a clean environment
Make a fresh Python environment for this repo (`uv` or conda), install LeRobot at a pinned version, and check that PyTorch sees your GPU if you have one. A CPU laptop is enough for everything except serious training. Add the simulation extras for the environments you want to try. Write every command that worked into `docs/lerobot-notes.md`. Installation pain is normal; solve it once, write it down.

## 3. Explore datasets before robots
Open the LeRobot dataset visualizer and browse community datasets on the Hub, especially SO-100/SO-101 pick-and-place ones. Look at what one episode contains: camera frames, joint positions, actions, timestamps. Then load one in Python with `LeRobotDataset` and print shapes. Understanding "observation in, action out" at this level is the foundation of everything else, including your long-term idea about learning from human video.

## 4. Run a pretrained policy in simulation
Install one simulation environment (PushT is the classic small one, ALOHA a richer arm one) and run a pretrained policy from the Hub inside it. Watch the rendered video. You have now done the full loop, observation to action to world, with zero hardware. Then deliberately break things: change the seed, the number of steps, the camera. Learn what the evaluation script logs and saves.

## 5. Train your first policy
Train a small policy, for example ACT or Diffusion on PushT, for a short run. Do not chase good results yet; aim to understand the training config, checkpoints, logs, and how to resume. Use Weights & Biases or plain logs to watch the loss. If your laptop is slow, rent a cloud GPU for an afternoon. Then evaluate your own checkpoint in simulation and compare it to the pretrained one.

## 6. Meet the SO-101 virtually
Download the SO-101 robot description (URDF/MJCF from TheRobotStudio's repository or MuJoCo models) and load it in a MuJoCo or PyBullet viewer. Move each joint, see its limits, and measure how far the gripper reaches. Place a virtual 20 cm board in front of it and check every corner. This becomes your reach test and your inverse-kinematics playground long before your own arm exists.

## 7. Scripted motion in simulation
Write a tiny script: given a target point above a square, compute joint angles with inverse kinematics, move there, descend, close the gripper, lift. This is the exact pick-and-place primitive the chess robot needs, built without hardware. Put it in this repo under `src/chess_arm/arm/` behind the `Arm` interface, with a simulation implementation. When the real arm arrives, only the driver underneath changes.

## 8. Your brother's arm: calibrate and teleoperate
When you get time with the real arm, follow the official SO-101 steps: find the motor ports, set motor IDs, calibrate leader and follower, then teleoperate. Feel how precise it really is: try to touch the centre of a 2.5 cm square repeatedly and note the error. Save the calibration files. Expect some confusion with ports and permissions on the first try; it is normal.

## 9. Record, push, replay
Attach a camera and record a few short episodes of picking up a chess piece with the leader arm. Push the dataset to a private Hugging Face repo, open it in the visualizer, and replay an episode on the follower arm. Now you know the whole data pipeline end to end. Name datasets clearly, for example `chess-arm-pick-e4-v1`, and log what each one contains.

## 10. Train and run on real hardware
With roughly fifty clean episodes of one simple task, train an ACT policy and run it on the arm. It will probably be wobbly; that is the lesson. Compare it honestly with your scripted mover from topic 7. For the chess MVP the script will likely win. The learned policy is your Phase 5 track and the seed of the startup idea.

## 11. Join the community
LeRobot has an active Discord, regular hackathons, and many SO-101 builders sharing datasets and fixes. Read the GitHub issues before fighting a bug alone; someone has usually hit it. Share your own small datasets publicly when they are clean; it builds reputation for the future company. Follow release notes, because breaking changes are frequent, and upgrade your pinned version deliberately, never by accident.
